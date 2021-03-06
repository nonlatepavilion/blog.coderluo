---
title: 'Zookeeper深度学习1-Paxos算法详解'
tags:
  - Paxos
  - Zookeeper
date: 2019-09-05 19:45:25
categories: Zookeeper
img: http://media.coderluo.top/img/20190928015012.png
---



*本片文章将开启对分布式协调服务zk的学习，目前规划是从理论基础开始逐步到源码解析，深入学习这个在分布式系统中起着至关作用的组件。*

对于 zk 理论的学习，最重要也是最难的知识点就是 Paxos 算法。所以我们首先学习 Paxos 算法。

-------------

## 算法简介

>Paxos 算法是莱斯利·兰伯特(Leslie Lamport)1990 年提出的一种基于消息传递的、具有高 容错性的一致性算法。Google Chubby 的作者 Mike Burrows 说过，世上只有一种一致性算法， 那就是 Paxos，所有其他一致性算法都是 Paxos 算法的不完整版。Paxos 算法是一种公认的晦 涩难懂的算法，并且工程实现上也具有很大难度。较有名的 Paxos 工程实现有 Google Chubby、 ZAB、微信的 PhxPaxos 等

Paxos 算法是用于解决什么问题的呢? Paxos 算法要解决的问题是，在分布式系统中如何 就某个决议达成一致。

## Paxos与拜占庭将军问题



> 拜占庭将军问题是由 Paxos 算法作者莱斯利·兰伯特提出的点对点通信中的基本问题。该 问题要说明的含义是，<font color='red'>在不可靠信道上试图通过消息传递的方式达到一致性是不可能的</font>。所 以，Paxos 算法的前提是<font color='red'>不存在拜占庭将军问题</font>，即信道是安全的、可靠的，集群节点间传 递的消息是不会被篡改的。

一般情况下，分布式系统中各个节点间采用两种通讯模型:**共享内存(Shared Memory)**、**消息传递(Messages Passing)**。而 Paxos 是基于消息传递通讯模型的。

## 算法描述

### 三种角色

在 Paxos 算法中有三种角色，分别具有三种不同的行为。但很多时候，一个进程可能同 时充当着多种角色。 

- Proposer:提案者，提出提案（Proposal）；
- Acceptor:表决者；
- Learner:学习者(同步者，即Proposer决议形成，将所有形成的决议发送给Learners)

### Paxos算法一致性

Paxos 算法的一致性主要体现在以下几点:

- 每个提案者在提出提案时都会首先获取到一个具有全局唯一性的、递增的提案编号N，即在整个集群中是唯一的编号 N，然后将该编号赋予其要提出的提案。
- 每个表决者在accept某提案后，会将该提案的编号N记录在本地，这样每个表决者中保存的已经被 accept 的提案中会存在一个编号最大的提案，其编号假设为 maxN。每个表决者仅会 accept 编号大于自己本地 maxN 的提案。
- 在众多提案中最终只能有一个提案被选定。
- 一旦一个提案被选定，则其它服务器会主动同步(Learn)该提案到本地。
- 没有提案被提出则不会有提案被选定。



### Paxos算法流程



Paxos 算法的执行过程划分为两个阶段:**准备阶段 prepare** 与**接受阶段 accept**。Ps:Learn阶段之前决议已经形成。

由于Paxos算法是晦涩难懂的，这里我将以自己的理解来做整个描述，虽然可能在严谨性上会差强人意，但是可读性会提高，希望可以给大家更轻松的阅读体验。

#### A、Prepare阶段



1. 提案者(Proposer)准备提交一个编号为 N 的提议，于是其首先向所有表决者(Acceptor)发 送 prepare(N)请求，用于试探集群是否支持该编号的提议。 如果这里不好理解我们可以试图理解为提议者拿着钞票去贿赂“表决者（Accept）”
2. 每个表决者（Acceptor）都保存者当前贿赂自己的最大金额数，即（maxN），当每个表决者接收到贿赂自己的提议时，会比较贿赂金额与maxN的值。有以下几种情况：
   1. 若 N 小于 maxN，则说明该提议已过时（钱少不接受），当前表决者采取不回应或回应 Error 的方 式来拒绝该 prepare 请求；
   2. 若 N 大于 maxN，则说明该提议是可以接受的（毕竟谁给的钱多听谁的），当前表决者会首先将该 N（当前贿赂金额） 记录下来， 并将其曾经已经 accept 的编号最大的提案 Proposal(myid,maxN,value)反馈给提案者， 以向提案者展示自己支持的提案意愿。其中第一个参数 myid 表示表决者 Acceptor 的标识 id，第二个参数表示其曾接受的提案的最大编号 maxN（前任领导贿赂的金额），第三个参数表示该 提案的真正内容 value（前任领导提议的内容）。当然，若当前表决者还未曾 accept 过任何提议，则会将 Proposal(myid,null,null)反馈给提案者。
   3. 在 prepare 阶段 N 不可能等于 maxN。这是由 N 的生成机制决定的。要获得 N 的值， 其必定会在原来数值的基础上采用同步锁方式增一。 



![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7ee1de1blj30qo0hg757.jpg)



> 这里需要说明下，为什么在表决者（Acceptor）判断贿赂金额大于当前保存的最大金额时会将前任已经保存的金额和提案内容返回给提案者，这是因为提案者（Proposer），在收到表决者的答复后，需要判断谁是最有钱的提案者，便推举它为领袖 (修改自己的提案)。

#### B、Accept阶段

1. 当提案者(Proposer)发出 prepare(N)后，若收到了超过半数的表决者(Accepter)的反馈， 那么该提案者就会将其真正的提案 Proposal(myid,N,value)发送给所有的表决者。
2. 当表决者(Acceptor)接收到提案者发送的 Proposal(myid,N,value)提案后，会再次拿出自己 曾经 accept 过的提议中的最大编号 maxN，或曾经记录下的 prepare 的最大编号，让 N 与它们进行比较，若 N **大于等于**这两个编号，则当前表决者 accept 该提案，并反馈给 提案者。若 N 小于这两个编号，则表决者采取不回应或回应 Error 的方式来拒绝该提议。
3. 若提案者没有接收到超过半数的表决者的 accept 反馈（中间有别人以更多的金额贿赂了它），则有两种可能的结果产生。一 是放弃该提案，不再提出；二是重新进入 prepare 阶段，递增提案号，重新提出 prepare 请求。
4. 若提案者接收到的反馈数量超过了半数，则其会向外广播两类信息：
   1. 向曾 accept 其提案的表决者发送“可执行数据同步信号”，即让它们执行其曾接收到的提案；
   2. 向未曾向其发送 accept 反馈的表决者发送“提案 + 可执行数据同步信号”，即让 它们接受到该提案后马上执行。



![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7ee1h8hxej30j80ihmym.jpg)



上述的过程中，如果某一个提议收到了大多数的表决者（Acceptor）的响应后（提案者（Proposal）中的N必须大于当前maxN才会响应），则提案通过，向所有表决者以及leaner发送同步数据，达成数据一致性。



当然上面只是简单的描述，真是的算法场景更复杂，所有提议者，决策者身份信息都是交叉的，如果提议者、接受者的数量是4个,5个。。。但是你按照上面的思路进行推演，最终会发现最终是唯一一个提议获取多数票而胜出，从而其他提议者和决策者同步此提议。

## 活锁问题



上面的流程可能会引发活锁问题，那么什么是活锁呢？  

> 活锁指的是任务或者执行者没有被阻塞，由于某些条件没有满足，导致一直重复尝试—失败—尝试—失败的过程。处于活锁的实体是在不断的改变状态，活锁有可能自行解开。



那么上面的行为是怎么会引发活锁呢？接下来我们一起来分析下：

在整个选举过程中，每个人谁先来谁后到，“接受者”什么时间能够接到“提议者”的信息，是完全不可控的；

假设，第一个提案者A（Proposal）已经成功过了prepare阶段，准备向Acceptors广播发送Accept时，有一个更有钱土豪提案者B也向决议者（Acceptors）广播了prepare请求并在A的accept请求到之前发送给了决议者，这时毫无疑问，决议者会接收该请求，并记录在册。这时候，A的accept请求姗姗来迟，决议者对比此proposal的贿赂金额已经小于当前记录的prepare最大编号，因此不响应给提议者A，则提议者A收到的响应为过半，此提案废弃。这时它又用大于Proposer A的贿赂金额重新发起 preapre广播请求，这时提议者B的accept请求还没有到达决议者（Acceptors），因此Acceptor也接受了该prepare请求，将其记录在案，在之后提议者B发出的accept请求到达，决议者发现贿赂金额已经小于当前prepare的最大贿赂金额，因此拒绝响应，这样就会形成活锁问题。



## 总结

本篇文章故事讲到这里就基本上结束了，下面我们来总结一下，其实Paxos算法主要包括两个阶段：

1. prepare阶段，这个阶段主要是准备一个编号为N的提案，首先向所有决议者（Acceptor）发送prepare请求，用于试探是否支持该编号的提议。
2. accept阶段，当一阶段提议收到了超过半数的响应，则开始正式下发提案内容proposal，如果过半则提案提交成功，广播给所有learner。

注意：编号（贿赂金额）很重要，无论在哪个阶段，编号（贿赂金额）小的，都会被鄙视（被拒绝）；



今天的分享就到这里了，这也是我学习Paxos算法过程中的一些心得，希望对初学者有些启发。

后续我们对继续深入学习Zookeeper之**Zab协议（Paxos算法的工业实现）**，今天提到的活锁问题，也会得到相应的解决，敬请期待。

系列文章：
- [Zookeeper深度学习2-Zab协议详解](http://coderluo.top/2019/09/08/zookeeper/zookeeper-shen-du-xue-xi-2-zab-xie-yi-xiang-jie/)
- [Zookeeper深度学习3-源码分析-Leader选举](http://coderluo.top/2019/09/08/zookeeper/zookeeper-shen-du-xue-xi-3-yuan-ma-fen-xi-leader-xuan-ju/)



















