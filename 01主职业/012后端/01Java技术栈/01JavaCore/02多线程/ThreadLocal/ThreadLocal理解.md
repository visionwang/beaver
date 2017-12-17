
ThreadLocal是一个非常简单的

　　一.对ThreadLocal的理解

　　二.深入解析ThreadLocal类

　　三.ThreadLocal的应用场景


　那么大家来仔细分析一下这个问题，这地方到底需不需要将connect变量进行共享？事实上，是不需要的。假如每个线程中都有一个connect变量，各个线程之间对connect变量的访问实际上是没有依赖关系的，即一个线程不需要关心其他线程是否对这个connect进行了修改的。

　　到这里，可能会有朋友想到，既然不需要在线程之间共享这个变量，可以直接这样处理，在每个需要使用数据库连接的方法中具体使用时才创建数据库链接，然后在方法调用完毕再释放这个连接。比如下面这样：

　这样处理确实也没有任何问题，由于每次都是在方法内部创建的连接，那么线程之间自然不存在线程安全问题。但是这样会有一个致命的影响：导致服务器压力非常大，并且严重影响程序执行性能。由于在方法中需要频繁地开启和关闭数据库连接，这样不尽严重影响程序执行效率，还可能导致服务器压力巨大。

　　那么这种情况下使用ThreadLocal是再适合不过的了，因为ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。

　　但是要注意，虽然ThreadLocal能够解决上面说的问题，但是由于在每个线程中都创建了副本，所以要考虑它对资源的消耗，比如内存的占用会比不使用ThreadLocal要大。



　　至此，可能大部分朋友已经明白了ThreadLocal是如何为每个线程创建变量的副本的：

　　首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。

　　初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。

　　然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

[Java并发编程：深入剖析ThreadLocal](http://www.cnblogs.com/dolphin0520/p/3920407.html)

从这段代码的输出结果可以看出，在main线程中和thread1线程中，longLocal保存的副本值和stringLocal保存的副本值都不一样。最后一次在main线程再次打印副本值是为了证明在main线程中和thread1线程中的副本值确实是不同的。

　　总结一下：

　　1）实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的；

　　2）为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，因为每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal；

　　3）在进行get之前，必须先set，否则会报空指针异常；

　　    如果想在get之前不需要调用set就能正常访问的话，必须重写initialValue()方法。



三.ThreadLocal的应用场景

	private static ThreadLocal<Connection> connectionHolder
	= new ThreadLocal<Connection>() {
	public Connection initialValue() {
	    return DriverManager.getConnection(DB_URL);
	}
	};
	 
	public static Connection getConnection() {
	return connectionHolder.get();
	}






　http://www.iteye.com/topic/103804


	private static final ThreadLocal threadSession = new ThreadLocal();
	 
	public static Session getSession() throws InfrastructureException {
	    Session s = (Session) threadSession.get();
	    try {
	        if (s == null) {
	            s = getSessionFactory().openSession();
	            threadSession.set(s);
	        }
	    } catch (HibernateException ex) {
	        throw new InfrastructureException(ex);
	    }
	    return s;
	}

什么是?

　可以看到ThreadLocalMap的Entry继承了WeakReference，并且使用ThreadLocal作为键值。



一个共享变量的一部分是线程独享的




    public DecryptResponseType decrypt(DecryptRequestType request)
                                    throws Exception {
        return super.invoke("decrypt", request, DecryptResponseType.class);
    }

    public <TReq, TResp> TResp invoke(final String operation, final TReq request, final Class<TResp> responseClass) throws IOException {
        return this.invoke(operation, request, null, responseClass);
    }


	 public <TReq, TResp> TResp invoke(final String operation, final TReq request, final Func<TResp> fallbackProvider, final Class<TResp> responseClass)
	            throws IOException {
	        if (StringValues.isNullOrWhitespace(operation))
	            throw new IllegalArgumentException("Argument 'operation' can not be null or white space");
	        if (request == null)
	            throw new IllegalArgumentException("Argument 'request' can not be null");
	
	        Span span = null;
	        if (_tracer.isTracing())
	            span = _tracer.startSpan(operation, getClass().getSimpleName(), SpanType.WEB_SERVICE);
	        ClientCatTransaction clientCatTransaction = null;
	        try {
	            final ExecutionContext<TResp> executionContext = createExecutionContext(operation, request, responseClass, span);
	            clientCatTransaction = new ClientCatTransaction(executionContext);
	            clientCatTransaction.startTransaction();
	            if (_hystrixEnabled) {
	                final ClientCatTransaction finalClientCatTransaction = clientCatTransaction;
	                return CHystrixCommandExecutor.execute(_domain, _groupKey, _chystrixKeyPrefix + operation.toLowerCase(), executionContext.getHost(),
	                        new Func<TResp>() {
	                            @Override
	                            public TResp execute() {
	                                try {
	                                    return invokeInternal(executionContext, finalClientCatTransaction);
	                                } catch (RuntimeException ex) {
	                                    throw ex;
	                                } catch (IOException ex) {
	                                    throw new TempRuntimeException(ex);
	                                }
	                            }
	                        }, fallbackProvider);
	            } else
	                return invokeInternal(executionContext, clientCatTransaction);
	        } catch (TempRuntimeException tempEx) {
	            if (clientCatTransaction != null)
	                clientCatTransaction.markFailure(tempEx.getCause());
	            throw (IOException) tempEx.getCause();
	        } catch (Throwable ex) {
	            if (clientCatTransaction != null)
	                clientCatTransaction.markFailure(ex);
	            throw ex;
	        } finally {
	            if (span != null)
	                span.stop();
	
	            if (clientCatTransaction != null)
	                clientCatTransaction.endTransaction();
	        }
	    }


