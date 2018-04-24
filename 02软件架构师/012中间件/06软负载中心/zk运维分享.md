https://www.jianshu.com/p/4a16d9f36b61


那么在打算运维一套Zookeeper集群之前，我们先了解一些Zookeeper的基本原理。
集群里分三种角色: Leader, Follower和Observer。Leader和Follower参与投票，Observer只会『听』投票的结果，不参与投票。

一个集群容忍挂掉的节点数的等式为 N = 2F + 1，N为投票集群节点数，F为能同时容忍失败节点数。比如一个三节点集群，可以挂掉一个节点，5节点集群可以挂掉两个...

一个写操作需要半数以上的节点ack，所以集群节点数越多，整个集群可以抗挂点的节点数越多(越可靠)，但是吞吐量越差。

Zookeeper里所有节点以及节点的数据都会放在内存里，形成一棵树的数据结构。并且定时的dump snapshot到磁盘。

Zookeeper的Client与Zookeeper之间维持的是长连接，并且保持心跳，Client会与Zookeeper之间协商出一个Session超时时间出来(其实就是Zookeeper Server里配置了最小值，最大值，如果client的值在这两个值之间则采用client的，小于最小值就是最小值，大于最大值就用最大值)，如果在Session超时时间内没有收到心跳，则该Session过期。

Client可以watch Zookeeper那个树形数据结构里的某个节点或数据，当有变化的时候会得到通知。







