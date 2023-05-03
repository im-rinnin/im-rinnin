---
title: paxos 学习与思考
date:  2023-4-23 14:15:59
tags:
- paxos
- raft
- consensus
- 技术
---

这里并不会完整描述paxos，只是记录重点和我的思考过程

## 共识问题

多个节点对某个值达成一致

解决方案的核心思想：如何选定的数值在达到多数节点后，不再改变，至于如何达到多数不重要
上述问题等同于：在发现数值在达到多数节点后，proposer只发起这个数值proposal
分解为两个问题

1. 如何读到这个数值并作为proposal的数值
2. 由于乱序，acceptor可能会接受到之前的proposal，如何解决

问题1

通过多数读，保证能读到，但是也可能读到另外没有达到多数的数值，这个问题可以通过prepare 阶段解决，可以通过反证法，如果 a 达到多数，b是少数，且 b id>a id,那么在prepare阶段，a和b一定有重叠acceptor接受 b id然后再接受a id，违背了prepare 忽视id小于当前正在进行proposal的id这个行为。

问题2
acceptor忽略proposal id小于当前接受数值的id

实际上，id唯一加上prepare阶段，proposer读取多数节点选择id最大这两个行为，形成一种id偏序关系，这种偏序关系，使得如果 proposal a id > b id,则 a 的取值有可能是通过prepare阶段读到的b，而不可能反过来。这种关系保证集群的信息不会因为乱序倒退，出现a先达到多数，然后又变成其他值的情况。从而保证算法的正确性，算法的推进性（progress）则需要通过选举成功保证

另外需要注意的是,这里的多数指的是某次proposal形成了多数，而不是两次proposal，使用同一个数值形成的多数，wiki上有个例子 [wiki paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science)#Basic_Paxos) Basic Paxos where a multi-identifier majority is insufficient

同理，raft里要求来自不同term的log entry达到多数不能算达到多数也是一样的原因（raft 论文 5.4.2 Committing entries from previous terms）

### 核心规则

1. proposal id 唯一
2. acceptor不接受比当前prepare 阶段的id小或者比当前取值的proposal id小的proposal
3. proposer先读取多数节点，选取id最大的proposal的取值（或者没有，则可以选用任意取值），作为第二阶段的proposal的取值

考虑到老爷子说过世界上只有一个共识算法，所以说所有共识算法都是上述步骤的演化

### 参考资料

paxos-simple 论文，感觉写的不是特别好
[wiki paxos]( https://en.wikipedia.org/wiki/Paxos_(computer_science))从开头到basic paxos部分，这里对paxos的描述写的比论文清洗
