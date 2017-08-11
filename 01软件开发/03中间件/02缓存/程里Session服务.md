程里的Session使用服务的方式暴露出来


sessionserver

选择CRedis的客户端

Redis->JDedis->CRedis自己封装->SessionServer

Jedis等对命令模式的使用，真心多


Factory, Provider, Default等等



public class Jedis extends BinaryJedis implements JedisCommands, MultiKeyCommands,
    AdvancedJedisCommands, ScriptingCommands, BasicCommands, ClusterCommands, SentinelCommands {

  protected Pool<Jedis> dataSource = null;}

最中还是ip + port



 private T runWithRetries(byte[] key, int redirections, boolean tryRandomNode, boolean asking) {
        if (redirections <= 0) {
            throw new JedisClusterMaxRedirectionsException("Too many Cluster redirections?");
        }

        Jedis connection = null;
        try {

            if (asking) {
                // TODO: Pipeline asking with the original command to make it
                // faster....
                connection = askConnection.get();
                connection.asking();

                // if asking success, reset asking flag
                asking = false;
            } else {
                if (tryRandomNode) {
                    connection = connectionHandler.getConnection();
                } else {
                    connection = connectionHandler.getConnectionFromSlot(JedisClusterCRC16.getSlot(key));
                }
            }

            return execute(connection);
        } catch (JedisConnectionException jce) {
            if (tryRandomNode) {
                // maybe all connection is down
                throw jce;
            }

            // release current connection before recursion
            releaseConnection(connection);
            connection = null;

            // retry with random connection
            return runWithRetries(key, redirections - 1, true, asking);
        } catch (JedisRedirectionException jre) {
            // if MOVED redirection occurred,
            if (jre instanceof JedisMovedDataException) {
                // it rebuilds cluster's slot cache
                // recommended by Redis cluster specification
                this.connectionHandler.renewSlotCache(connection);
            }

            // release current connection before recursion or renewing
            releaseConnection(connection);
            connection = null;

            if (jre instanceof JedisAskDataException) {
                asking = true;
                askConnection.set(this.connectionHandler.getConnectionFromNode(jre.getTargetNode()));
            } else if (jre instanceof JedisMovedDataException) {
            } else {
                throw new JedisClusterException(jre);
            }

            return runWithRetries(key, redirections - 1, false, asking);
        } finally {
            releaseConnection(connection);
        }
    }


connection = connectionHandler.getConnectionFromSlot(JedisClusterCRC16.getSlot(key));

LOOKUP_TABLE

	  public static int getSlot(byte[] key) {
	    int s = -1;
	    int e = -1;
	    boolean sFound = false;
	    for (int i = 0; i < key.length; i++) {
	      if (key[i] == '{' && !sFound) {
	        s = i;
	        sFound = true;
	      }
	      if (key[i] == '}' && sFound) {
	        e = i;
	        break;
	      }
	    }
	    if (s > -1 && e > -1 && e != s + 1) {
	      return getCRC16(key, s + 1, e) & (16384 - 1);
	    }
	    return getCRC16(key) & (16384 - 1);
	  }


    @Override
    public Jedis getConnectionFromSlot(int slot) {
        JedisPool connectionPool = cache.getSlotPool(slot);
        if (connectionPool != null) {
            // It can't guaranteed to get valid connection because of node
            // assignment
            return connectionPool.getResource();
        } else {
            return getConnection();
        }
    }



CRC即循环冗余校验码（Cyclic Redundancy Check[1]  ）：是数据通信领域中最常用的一种查错校验码，其特征是信息字段和校验字段的长度可以任意选定。循环冗余检查（CRC）是一种数据传输检错功能，对数据进行多项式计算，并将得到的结果附在帧的后面，接收设备也执行类似的算法，以保证数据传输的正确性和完整性。


[REDIS的使用](http://www.cnblogs.com/honger/p/5852005.html)



    public T run(int keyCount, String... keys) {
        if (keys == null || keys.length == 0) {
            throw new JedisClusterException("No way to dispatch this command to Redis Cluster.");
        }

        // For multiple keys, only execute if they all share the
        // same connection slot.
        if (keys.length > 1) {
            int slot = JedisClusterCRC16.getSlot(keys[0]);
            for (int i = 1; i < keyCount; i++) {
                int nextSlot = JedisClusterCRC16.getSlot(keys[i]);
                if (slot != nextSlot) {
                    throw new JedisClusterException("No way to dispatch this command to Redis Cluster "
                            + "because keys have different slots.");
                }
            }
        }

        return runWithRetries(SafeEncoder.encode(keys[0]), this.redirections, false, false);
    }


        return runWithRetries(SafeEncoder.encode(keys[0]), this.redirections, false, false);
这儿有slot，但比如10台机器Redis怎么办？如何下发？


connectionHandler.getConnectionFromSlot(JedisClusterCRC16.getSlot(key));
        JedisPool connectionPool = cache.getSlotPool(slot);

JedisClusterInfoCache

  private Map<String, JedisPool> nodes = new HashMap<String, JedisPool>();
  private Map<Integer, JedisPool> slots = new HashMap<Integer, JedisPool>();

  private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
  private final Lock r = rwl.readLock();
  private final Lock w = rwl.writeLock();
  private final GenericObjectPoolConfig poolConfig;


key->slotid -> 具体的RedisConnection
	
	  public JedisPool getSlotPool(int slot) {
	    r.lock();
	    try {
	      return slots.get(slot);
	    } finally {
	      r.unlock();
	    }
	  }




public void discoverClusterNodesAndSlots(Jedis jedis) {
    w.lock();

    try {
      this.nodes.clear();
      this.slots.clear();

      List<Object> slots = jedis.clusterSlots();

      for (Object slotInfoObj : slots) {
        List<Object> slotInfo = (List<Object>) slotInfoObj;

        if (slotInfo.size() <= MASTER_NODE_INDEX) {
          continue;
        }

        List<Integer> slotNums = getAssignedSlotArray(slotInfo);

        // hostInfos
        int size = slotInfo.size();
        for (int i = MASTER_NODE_INDEX; i < size; i++) {
          List<Object> hostInfos = (List<Object>) slotInfo.get(i);
          if (hostInfos.size() <= 0) {
            continue;
          }

          HostAndPort targetNode = generateHostAndPort(hostInfos);
          setNodeIfNotExist(targetNode);
          if (i == MASTER_NODE_INDEX) {
            assignSlotsToNode(slotNums, targetNode);
          }
        }
      }
    } finally {
      w.unlock();
    }
  }

先理解到这，之后再加强，山子