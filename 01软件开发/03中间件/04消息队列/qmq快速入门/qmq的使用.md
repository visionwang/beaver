消息分类与选择
 
消息生产者按发送消息可靠性分为：事务持久型消息生成者，（非事务）持久型消息生产者，非持久型生产者和非可靠消息生产者
**事务持久型消息生产者**：
可靠性由qmq保证，非常可靠，可与业务绑定在一个事务中，业务处理成功消息发送，业务处理失败业务回滚消息不发送，但并发量不高
事务消息的发送和业务操作在同一个事务里，业务操作成功，消息就一定会发送，业务操作失败消息就一定不发送(事务消息不仅仅是保证消息的可靠性，更多的是保证消息的一致性
**持久型消息生产者**：
可靠性由qmq保证，非常可靠，业务处理和发消息不在一个事务中，可能会导致业务处理失败但消息发送出去了，但并发量不高
持久消息的含义是，有的时候我们并不需要那么高的可靠性(因为可靠性也意味着更高的成本)，我们容许部分失败的情况，那我们可以选择持久型消息，持久型的消息只要sendMessage执行成功消息就一定会发出去
**非持久型消息生产者**：
消息在client端不持久化也不保证事务，在server端持久化，可靠性中等，client端可通过捕获异常和响应监听实现重发提高可靠性，并发量中等
**非可靠消息生产者**：
消息在client端和server端都不持久化，而且消息消费者消费失败也不会重推消息，可靠性最低，但是并发量大，性能高，一般用在非可靠通知场景。

消息生产者按消息投递时间分为：及时消息生产者和延迟（或定时）消息生产者
消息在client端发送过程会经历4个步骤：**校验、存储、队列缓冲和RPC远程调用**。


4种生产者的场景.......


	@Override
	public void send() {
	    base.message().removeProperty(BaseMessage.keys.qmq_registry);
	
	    if (state.compareAndSet(INIT, QUEUED)) {
	        tries.incrementAndGet();
	        //当未指定消息存储时，使用同步发送，发送失败会抛出异常
	        if (store == null && syncSend) {
	            queue.send(this);
	            base.addTimeAnnotation("sync enter queue");
	        } else if (queue.offer(this)) {
	            base.addTimeAnnotation("async enter queue");
	            QmqLogger.log(getBase(), "进入发送队列.");
	        } else if (store != null) {
	            base.addTimeAnnotation("enter queue failed, wait for task");
	            QmqLogger.log(getBase(), "内存发送队列已满! 此消息将暂时丢弃,等待task处理");
	        } else {
	            if (ReliabilityLevel.isLow(getBase())) {
	                base.addTimeAnnotation("enter queue failed, throw");
	                QmqLogger.log(getBase(), "内存发送队列已满! 非可靠消息，此消息被丢弃.");
	                return;
	            }
	            QmqLogger.log(getBase(), "内存发送队列已满! 此消息在用户进程阻塞,等待队列激活.");
	
	            if (queue.offer(this, WAIT_ENQUEUE_TIMEOUT)) {
	                base.addTimeAnnotation("enter queue succ");
	                QmqLogger.log(getBase(), "重新入队时成功进入发送队列.");
	            } else {
	                base.addTimeAnnotation("enter queue failed");
	                QmqLogger.log(getBase(), "由于无法入队,发送失败！取消发送!");
	            }
	        }
	    } else
	        throw new IllegalStateException("同一条消息不能被入队两次.");
	}

//1.插库
//2.rpc发送消息，基于dubbo,基于http

    @Override
    public void insertNew(ProduceMessage message) {
        try {
            StatementParameters parameters = new StatementParameters();
            parameters.set(1, message.getMessageId());
            parameters.set(2, new Timestamp(System.currentTimeMillis()));
            parameters.set(3, SerializerFactory.create().serialize(message.getBase()));

            RouteKey routeKey = computeRouteKey();
            DalClient dalClient = get(routeKey.logicDbName);

            DalHints dalHints = new DalHints();
            routeKey.setShardId(dalHints);
            routeKey.setRealDbName(dalHints);

            dalClient.update(SqlConstant.insertSQL, parameters, dalHints);
            message.setStoreKey(routeKey);
        } catch (SQLException e) {
            String error = e.getMessage();
            if (error == null) throw new RuntimeException(e);
            if (error.indexOf("qmq_msg_queue") > 0) {
                throw new RuntimeException("插入消息失败: " + dbName + " 这个db里没有qmq_msg_queue这个表，请在qmq管理后台提交申请: http://qmq.ctripcorp.com/application/create.do(fat,uat,pro在各自环境申请)");
            }
            throw new RuntimeException(e);
        }
    }


    @Override
    public void insertNew(ProduceMessage message) {
        try {
            StatementParameters parameters = new StatementParameters();
            parameters.set(1, message.getMessageId());
            parameters.set(2, new Timestamp(System.currentTimeMillis()));
            parameters.set(3, SerializerFactory.create().serialize(message.getBase()));

            RouteKey routeKey = computeRouteKey();
            DalClient dalClient = get(routeKey.logicDbName);

            DalHints dalHints = new DalHints();
            routeKey.setShardId(dalHints);
            routeKey.setRealDbName(dalHints);

            dalClient.update(SqlConstant.insertSQL, parameters, dalHints);
            message.setStoreKey(routeKey);
        } catch (SQLException e) {
            String error = e.getMessage();
            if (error == null) throw new RuntimeException(e);
            if (error.indexOf("qmq_msg_queue") > 0) {
                throw new RuntimeException("插入消息失败: " + dbName + " 这个db里没有qmq_msg_queue这个表，请在qmq管理后台提交申请: http://qmq.ctripcorp.com/application/create.do(fat,uat,pro在各自环境申请)");
            }
            throw new RuntimeException(e);
        }
    }


    static {
        Corporation corporation = ServerManager.getInstance().getCorporation();
        TABLE_NAME = corporation == Corporation.QUNAR ? DATABASE_NAME + "." + "qmq_msg_queue" : "qmq_msg_queue";
        insertSQL = String.format("INSERT INTO %s (message_id,create_time,content) VALUES(?,?,?)", TABLE_NAME);
        errorSQL = String.format("UPDATE %s SET status=?,error=error+1,update_time=? WHERE message_id=?", TABLE_NAME);
        finishSQL = String.format("DELETE FROM %s WHERE message_id=?", TABLE_NAME);
    }