# paxos
目的：多节点之间如何就某个值（提案 Value）达成共识

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
 1. 我们用prepare请求来阻塞掉老的提议
 2. 我们用prepare请求来找到可能已经被选定的值

## 2. Multi-Paxos工作流程
参考[paxos.pdf](https://ongardie.net/static/raft/userstudy/paxos.pdf)第18页
 1. client希望状态机能执行一个command，它把这个请求发送给server的共识模块（consensus module）
 2. server的共识模块（consensus module）使用paxos，与其他server相互通信，并选择出这个log entry应该是什么值
 3. 一旦这个值被选定，这个值就可以输入到状态机中
 4. 状态机处理完这个command，进入新的状态，并返回处理结果给client

## 3. Multi-Paxos要解决的问题
 1. 当一个client发起请求时，我们要怎样为此请求选择一个log entry来存放这个命令？
 > 一台服务器收到一个客户端的请求后，就会从自己最小的未知是否选定的位置开始，跑basic paxos，这个位置不成功的话就再找下一个未知是否选定的位置，直到成功为止。
 2. 怎么解决Basic-Paxos性能问题，每个Instance都会执行Prepare、Accept两次RPC请求？
 > 如果proposer上次提交的值被选定，那proposer复用proposal number；如果proposer上次提交的值没有被选定，那根据paxos的步骤，proposer是必然要变更自己的proposal number
 3. 怎么确保一个完整的log副本被确定下来，并且让所有的server都知道这一份完整的log副本？
 4. 与client间的协议？
 5. 配置变更后，是否符合安全？

## 4. log的状态
只有chosen状态的log才能传给状态机
 1. accept状态（只有acceptor知道log被accept）
 2. chosen状态（只有proposer知道log被chosen）

## 5. 提案编号
1. 把服务器id作为提议号的低bit部分。这就保证了其他服务器肯定不可能生成一样的号出来。
2. 而提议号的高bit部分是一个round number。每个server都保存了自己至今为止所看到或用到过的最大的round number，设这个值为maxRound。
3. 要生成一个新的提议号时，server用maxRound+1来作为round number，拼接上自己的server id，就得到了一个提议号。
4. 为了确保一个proposer在crash后重启，不会碰巧使用了之前用过的提议号，proposer每次更新maxRound时，必须马上把maxRound永久存储在磁盘里。

```c++
bool operator >= (const BallotNumber & other) const
{
    if (m_llProposalID == other.m_llProposalID)
    {
        return m_llNodeID >= other.m_llNodeID;
    }
    else
    {
        return m_llProposalID >= other.m_llProposalID;
    }
}
```

