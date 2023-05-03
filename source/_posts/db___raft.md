---
title: raft学习笔记
date:  2023-4-22 14:15:59
tags:
- raft
- consensus
- 技术
---

## 核心

Replicated state machine模型+ 优化的paxos+详细工程实现+简单理解
- ## 术语
  
  leader: 负责发起新的log写入，term下唯一，单个写入节点从而决定写入顺序，了解集群最新信息
  follower：写入信息复制
  candidate：leader失效后选举新leader的主动参选者
  term:递增的逻辑时间id，可以和paxos的proposal id进行对应
  log entry：数值的变动，entry形成一个有序的log，entry本身对应paxos的proposal
  index：entry 在log的序号
  committed: log entry复制达到多数节点，即完成共识
  ![屏幕快照 2023-04-23 下午3.30.56.png](/images/屏幕快照_2023-04-23_下午3.30.56_1682235075997_0.png)
## 关键流程
### 选举

通过超时机制，保证term下最多有一个leader
限制条件，只会投一次票，只给log比自己新的candidate投票
### log复制

log 由leader 复制到每个follower
leader的log只会追加不会修改
复制过程：leader找到最后一条相同的log ，从这里开始进行复制
### memership 转移

通过加入一个中间阶段配置作为过度
### log compaction

避免log太长，使用snapshot进行压缩，把多个log压缩成一个状态，去掉冗余的log
不是算法的核心，这里省略
## 特性

committed 的entry不会再改变（共识算法的核心定义）,通过选举的条件和leader不会修改自己的entry来保证
committed 的entry之前的entry不会再改变,实现同上
相同index和term的log entry相同，log复制的机制来保证
## 和paxos对比

term就是proposal id,唯一
log entry就是proposal,follow不接受比当前小的proposal
选举的限制条件，只给比自己log更新的candidate投票，也是paxos prepare阶段，获取最大的proposal id并用它的数值作为后续proposal的数值

所以raft 的内核还是逃不了paxos那一套
## 感想

文中提到为了方便理解，很多设计都偏向简单，比如超时实现leader选举，单一leader写入数据，简单数据流向（client->leader->follower）,leader数据不会修改
理解paxos 是理解raft的基础，知道为什么要这么做，那些步骤可以优化而不会破坏paxos的核心规则
文中提到作者思考过很多其他方案，感觉看论文就是这样，只能看到最后一个结果，但是背后的动机，想要正在搞清楚需要花很多时间去研究相关的背景材料。光是看一篇只能达到能用但是不一定能完全理解的程度
## 参考资料

raft paper In Search of an Understandable Consensus Algorithm
raft phd论文 CONSENSUS: BRIDGING THEORY AND PRACTICE 有一些tla证明
paxos 论文