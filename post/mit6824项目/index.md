
##MapReduce
### Map/Reduce input and output
主要是关于怎么做Map，怎么做Reduce；
Map过程主要的思想就是利用一个哈希函数将任务分类，以分别分配给不同的机器，并且保证任务之间没有重复也没有漏掉的。
### Distributing MapReduce tasks
1. 进程间通信 RPC
2. worker如何向master注册，master又如何为worker安排任务
### Handling worker failures
1. 如果某个worker挂了，要能够将它的工作安排给其他worker。
### WordCount/Inverted Index
1. MapReduce思想的应用

## Raft
### 实现LeaderElection和Heartbeats
1. 选出leader而且当网络中没有错误时，可以保证leader不变
2. 当leader挂掉时，保证选出新的leader
3. 当旧的leader重新连接时，保证旧leader不会影响新leader：主要是通过逻辑时钟来完成。
### 实现Leader和Follower之间的日志追加
### 实现持久化

## Fault-tolerant Key/Value Service
### Key/value服务
1. 在客户端实现Put/Append/Get方法
2. 在服务器端实现Put/Append/Get方法；