# paxos
> Once a value has been chosen, future proposals must propose the same value.

```
1. 有一台或多台服务会提议（propose）一些值。
2. 系统必须通过一定机制从提议（propose）的值中选定（chose）一个值。
3. 只有一个值能被选定（chosen）。即不允许系统先选定了值A，后面又改选定值B。
``` 

1. [用paxos实现多副本日志系统--basic paxos部分](https://cloud.tencent.com/developer/article/1147420)
2. [用paxos实现多副本日志系统--multi paxos部分](https://cloud.tencent.com/developer/article/1158799)
3. [Implementing Replicated Logs with Paxos](https://ongardie.net/static/raft/userstudy/paxos.pdf)
4. [图解超难理解的 Paxos 算法（含伪代码）](https://xie.infoq.cn/article/e53cbcd0e723e3a6ce4be3b8c)
5. [Paxos & Raft lecture, Diego Ongaro](https://www.bilibili.com/video/BV1WW411a77S?from=search&seid=9258539723484618240&spm_id_from=333.337.0.0)
6. [Paxos 的变种（一）：Multi-Paxos 是如何劝退大家去选择 Raft 的](https://xie.infoq.cn/article/92f6b1a031594da8164645459)

## 1. 为什么需要Prepare请求
 * 我们用prepare请求来阻塞掉老的提议
 * 我们用prepare请求来找到可能已经被选定的值
