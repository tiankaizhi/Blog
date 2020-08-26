
所有事务请求由 Leader 节点负责将其转换成一个事务 Proposal，并将该 Proposal 广播发送给集群中所有的 Follower 节点，然后 Leader 节点需要等待所有的 Follower 节点的 ACK 反馈，一旦超过半数的 Follower 节点返回了 ACK 后（Quorum 选举机制），Leader 就会再次向所有的 Follower 节点广播发送 Commit 消息，要求所有的 Follower 节点将收到的 Proposal 进行提交。消息广播过程如下图：
