    private ZookeeperValue getValue(String key, String group) throws Exception {
        String path = ZookeeperUtils.getConfigPath(key, group);
        String timestampPath = ZookeeperUtils.getTimestampPath(path);
        ZookeeperValue zkValue = null;

        MonitorTransaction t = monitor.newTransaction("lion", "get");
        t.addData(key);
        try {
            byte[] data = zookeeperOperation.getDataWatched(path);

            if (data != null) {
                zkValue = new ZookeeperValue();
                zkValue.setValue(new String(data, Constants.CHARSET));
                data = zookeeperOperation.getData(timestampPath);
                if (data != null) {
                    Long timestamp = CodecUtils.getLong(data);
                    zkValue.setTimestamp(timestamp);
                }
            }
            t.setStatus(Monitor.SUCCESS);
            return zkValue;
        } catch (Exception e) {
            t.setStatus(e);
            throw e;
        } finally {
            t.complete();
        }
    }

    public byte[] getData(String path) throws Exception {
        try {
            byte[] data = curatorClient.getData().forPath(path);
            return data;
        } catch (NoNodeException e) {
            return null;
        }
    }



    watch。。。和http长轮询？


    ZooKeeper代码版本中，提供了分布式独享锁、选举、队列的接口，代码在zookeeper-3.4.3\src\recipes。其中分布锁和队列有Java和C两个版本，选举只有Java版本。

ZooKeeper的基本运转流程：
1、选举Leader。
2、同步数据。
3、选举Leader过程中算法有很多，但要达到的选举标准是一致的。
4、Leader要具有最高的执行ID，类似root权限。
5、集群中大多数的机器得到响应并接受选出的Leader。







