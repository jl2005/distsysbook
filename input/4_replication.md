# %chapter_number%. 复制

[TOC]

<!--
# %chapter_number%. Replication
-->

<!--
The replication problem is one of many problems in distributed systems. I've chosen to focus on it over other problems such as leader election, failure detection, mutual exclusion, consensus and global snapshots because it is often the part that people are most interested in. One way in which parallel databases are differentiated is in terms of their replication features, for example. Furthermore, replication provides a context for many subproblems, such as leader election, failure detection, consensus and atomic broadcast.
-->

复制问题是分布式系统中的许多问题之一。我选择专注于复制而不是其他问题，例如 leader 选举、失败检测、互斥，共识和全局快照，因为它通常是人们对复制最感兴趣。区分并行数据库的一种方式他们的复制功能。此外，复制为许多子问题提供了上下文，例如领导者选举，故障检测，共识和原子广播。

<!--
Replication is a group communication problem. What arrangement and communication pattern gives us the performance and availability characteristics we desire? How can we ensure fault tolerance, durability and non-divergence in the face of network partitions and simultaneous node failure?
-->

复制是一种群组通信问题。什么样的协议和通信模式可以为我们提供所需的性能和可用性特征？我们如何确保面对网络分区和同时节点故障时的容错、持久性和无分歧？

<!--
Again, there are many ways to approach replication. The approach I'll take here just looks at high level patterns that are possible for a system with replication. Looking at this visually helps keep the discussion focused on the overall pattern rather than the specific messaging involved. My goal here is to explore the design space rather than to explain the specifics of each algorithm.
-->

同样，有很多方法可以实现复制。我将在这里采用的方法只关注具有复制功能的系统可能采用的高级模式。从表面上看这一点有助于使讨论集中在整体模式而不是所涉及的特定消息传递上。我的目标是探索设计空间，而不是解释每个算法的细节。

<!--
Let's first define what replication looks like. We assume that we have some initial database, and that clients make requests which change the state of the database.
-->

让我们首先定义复制。 我们假设我们有一些初始数据库，并且客户端发出更改数据库状态的请求。

<img src="images/replication-both.png" alt="replication" style="height: 340px;">

<!--
The arrangement and communication pattern can then be divided into several stages:

1. (Request) The client sends a request to a server
3. (Sync) The synchronous portion of the replication takes place
4. (Response) A response is returned to the client
5. (Async) The asynchronous portion of the replication takes place
-->

然后可以将协议和通信模式分为几个阶段：

1. （请求）客户端向服务器发送请求
2. （同步）发生复制的同步部分
3. （响应）响应返回给客户端
4. （异步）发生复制的异步部分

<!--
This model is loosely based on [this article](https://www.google.com/search?q=understanding+replication+in+databases+and+distributed+systems). Note that the pattern of messages exchanged in each portion of the task depends on the specific algorithm: I am intentionally trying to get by without discussing the specific algorithm.

Given these stages, what kind of communication patterns can we create? And what are the performance and availability implications of the patterns we choose?
--> 

这个模型基于[这篇文章](https://www.google.com/search?q=understanding+replication+in+databases+and+distributed+systems)。请注意，在任务的每个部分中交换的消息模式取决于特定的算法：我有意在不讨论特定算法的情况下说明消息模式。

鉴于这些阶段，我们可以创建什么样的消息模式？我们选择的模式的性能和可用性含义是什么？

<!--
## Synchronous replication

The first pattern is synchronous replication (also known as active, or eager, or push, or pessimistic replication). Let's draw what that looks like:
--> 

## 同步复制

第一种模式是同步复制（也称为主动、或活跃、推送或悲观复制）。让我们画出来，看看像什么： 

<img src="images/replication-sync.png" alt="replication" style="height: 340px;">

<!--
Here, we can see three distinct stages: first, the client sends the request. Next, what we called the synchronous portion of replication takes place. The term refers to the fact that the client is blocked - waiting for a reply from the system.

During the synchronous phase, the first server contacts the two other servers and waits until it has received replies from all the other servers. Finally, it sends a response to the client informing it of the result (e.g. success or failure).
-->

在这里，我们可以看到三个不同的阶段：首先，客户端发送请求。接下来，我们称之为复制的同步部分。该术语指的是客户端被阻止——等待系统的回复。

在同步阶段，第一台服务器联系其他两台服务器并等待，直到收到所有其他服务器的回复。最后，它向客户发送响应，通知结果（例如成功或失败）。

<!--
All this seems straightforward. What can we say of this specific arrangement of communication patterns, without discussing the details of the algorithm during the synchronous phase? First, observe that this is a write N - of - N approach: before a response is returned, it has to be seen and acknowledged by every server in the system.

From a performance perspective, this means that the system will be as fast as the slowest server in it. The system will also be very sensitive to changes in network latency, since it requires every server to reply before proceeding.
->

这一切似乎都很简单。在没有讨论同步阶段的算法细节的情况下，我们可以看看通信模式的协议是什么样子的？首先，发现这是一种写 N-of-N 方法：在返回响应之前，系统中的每个服务器必须可见和确认。

从性能角度来看，这意味着系统将与其中最慢的服务器一样快。 系统对网络延迟的变化也非常敏感，因为它要求每个服务器在继续之前进行回复。

<!--
Given the N-of-N approach, the system cannot tolerate the loss of any servers. When a server is lost, the system can no longer write to all the nodes, and so it cannot proceed. It might be able to provide read-only access to the data, but modifications are not allowed after a node has failed in this design.

This arrangement can provide very strong durability guarantees: the client can be certain that all N servers have received, stored and acknowledged the request when the response is returned. In order to lose an accepted update, all N copies would need to be lost, which is about as good a guarantee as you can make.
-->

鉴于 N-of-N 方法，系统无法容忍任何服务器的失败。 当服务器失败时，系统无法再写入所有节点，因此无法继续。它可能能够提供对数据的只读访问权限，但在此设计中节点出现故障后，不允许进行修改。

这种协议可以提供非常强大的持久性保证：客户端可以确定所有N个服务器在返回响应时已经接收、存储和确认了请求。如果想要丢失已接受的更新，则需要所有N份副本都丢失，这是您可以做的最好的保证。

<!--
## Asynchronous replication

Let's contrast this with the second pattern - asynchronous replication (a.k.a. passive replication, or pull replication, or lazy replication). As you may have guessed, this is the opposite of synchronous replication:
-->

## 异步复制

让我们将其与第二种模式进行对比——异步复制（也被成为：被动复制、拉取复制或延迟复制）。您可能已经猜到，这与同步复制相反：

<img src="images/replication-async.png" alt="replication" style="height: 340px;">

<!--
Here, the master (/leader / coordinator) immediately sends back a response to the client. It might at best store the update locally, but it will not do any significant work synchronously and the client is not forced to wait for more rounds of communication to occur between the servers.

At some later stage, the asynchronous portion of the replication task takes place. Here, the master contacts the other servers using some communication pattern, and the other servers update their copies of the data. The specifics depend on the algorithm in use.
-->

这里，主（leader / coordinator）立即向客户端发回响应。它最多可以在本地存储更新，但它不会同步执行任何重要工作，并且客户端不会被迫等待服务器之间进行更多轮通信。

在稍后阶段，将发生复制任务的异步部分。这里，主设备使用某种通信模式联系其他服务器，其他服务器更新其数据副本。具体取决于使用的算法。

<!--
What can we say of this specific arrangement without getting into the details of the algorithm? Well, this is a write 1 - of - N approach: a response is returned immediately and update propagation occurs sometime later.

From a performance perspective, this means that the system is fast: the client does not need to spend any additional time waiting for the internals of the system to do their work. The system is also more tolerant of network latency, since fluctuations in internal latency do not cause additional waiting on the client side.

This arrangement can only provide weak, or probabilistic durability guarantees. If nothing goes wrong, the data is eventually replicated to all N machines. However, if the only server containing the data is lost before this can take place, the data is permanently lost.
-->

在不深入了解算法细节的情况下，我们可以看看具体协议是什么样子？好吧，这是一种写入 1-of-N 的方法：立即返回响应，并在稍后的某个时间发生更新传播。

从性能角度来看，这意味着系统速度很快：客户端不需要花费任何额外的时间来等待系统的内部工作。系统也更容忍网络延迟，因为内部延迟的波动不会导致客户端的额外等待。

这种协议只能提供弱的或概率的持久性保证。如果没有出错，数据最终会复制到所有N台机器。但是，如果包含数据的唯一服务器在此之前丢失，则数据将永久丢失。

<!--
Given the 1-of-N approach, the system can remain available as long as at least one node is up (at least in theory, though in practice the load will probably be too high). A purely lazy approach like this provides no durability or consistency guarantees; you may be allowed to write to the system, but there are no guarantees that you can read back what you wrote if any faults occur.

Finally, it's worth noting that passive replication cannot ensure that all nodes in the system always contain the same state. If you accept writes at multiple locations and do not require that those nodes synchronously agree, then you will run the risk of divergence: reads may return different results from different locations (particularly after nodes fail and recover), and global constraints (which require communicating with everyone) cannot be enforced.
-->

给定 1-of-N 方法，只要至少一个节点启动，系统就可以保持可用（至少在理论上，尽管在实践中负载可能太高）。像这样的纯粹的惰性方法没有提供持久性或一致性保证；您可能被允许写入系统，但无法保证您可以在发生任何故障时读到您所写的内容。

最后，值得注意的是，被动复制无法确保系统中的所有节点始终包含相同的状态。如果您接受多个位置的写入并且不要求这些节点同步确认，那么您将面临分歧的风险：读取可能会返回不同位置的不同结果（特别是在节点发生故障和恢复之后），以及全局约束（这需要与每一个都通信）无法执行。

<!--
I haven't really mentioned the communication patterns during a read (rather than a write), because the pattern of reads really follows from the pattern of writes: during a read, you want to contact as few nodes as possible. We'll discuss this a bit more in the context of quorums.

We've only discussed two basic arrangements and none of the specific algorithms. Yet we've been able to figure out quite a bit of about the possible communication patterns as well as their performance, durability guarantees and availability characteristics.
-->

我没有在读取（而不是写入）期间真正提到通信模式，因为读取模式实际上是从写入模式开始的：在读取期间，您希望尽可能少地联系节点。我们将在法定人数的背景下进一步讨论这个问题。

我们只讨论了两种基本协议，而不是讨论具体的算法。然而，我们已经能够弄清楚可能的通信模式以及它们的性能、持久性保证和可用性特征。

<!--
## An overview of major replication approaches

Having discussed the two basic replication approaches: synchronous and asynchronous replication, let's have a look at the major replication algorithms.

There are many, many different ways to categorize replication techniques. The second distinction (after sync vs. async) I'd like to introduce is between:

- Replication methods that prevent divergence (single copy systems) and
- Replication methods that risk divergence (multi-master systems)
-->

## 主要复制方法的概述

已经讨论了两种基本的复制方法：同步和异步复制，让我们来看看主要的复制算法。

有许多不同的方法可以对复制技术进行分类。我要介绍的第二个区别（在同步与异步之后）是：

- 防止分歧的复制方法（单拷贝系统）和
- 风险分歧的复制方法（多主系统）

<!--
The first group of methods has the property that they "behave like a single system". In particular, when partial failures occur, the system ensures that only a single copy of the system is active. Furthermore, the system ensures that the replicas are always in agreement. This is known as the consensus problem.

Several processes (or computers) achieve consensus if they all agree on some value. More formally:

1. Agreement: Every correct process must agree on the same value.
2. Integrity: Every correct process decides at most one value, and if it decides some value, then it must have been proposed by some process.
3. Termination: All processes eventually reach a decision.
4. Validity: If all correct processes propose the same value V, then all correct processes decide V.
-->

第一组方法具有“行为类似于单个系统”的属性。特别是，当发生部分故障时，系统确保只有一个系统副本处于活动状态。此外，系统确保副本始终保持一致。这被称为共识问题。

如果几个进程（或计算机）都同意某些值，那么它们就会达成共识。更正式的：

1. 协议：每个正确的进程必须就相同的值达成一致。
2. 完整性：每个正确的进程最多决定一个值，如果它决定某个值，那么它必须由某个进程提出。
3. 终止：所有进程最终都会做出决定。
4. 有效性：如果所有正确的进程提出相同的值V，那么所有正确的进程决定V.

<!--
Mutual exclusion, leader election, multicast and atomic broadcast are all instances of the more general problem of consensus. Replicated systems that maintain single copy consistency need to solve the consensus problem in some way.

The replication algorithms that maintain single-copy consistency include:

- 1n messages (asynchronous primary/backup)
- 2n messages (synchronous primary/backup)
- 4n messages (2-phase commit, Multi-Paxos)
- 6n messages (3-phase commit, Paxos with repeated leader election)
-->

互斥、leader 选举、多播和原子广播都是更普遍的共识问题的实例。保持单一副本一致性的复制系统需要以某种方式解决共识问题。

保持单一副本一致性的复制算法包括：

- 1n 条消息（异步主/从）
- 2n 条消息（同步主/从）
- 4n 条消息（2阶段提交，Multi-Paxos）
- 6n 条消息（三阶段提交，Paxos重复领导者选举）

<!--
These algorithms vary in their fault tolerance (e.g. the types of faults they can tolerate). I've classified these simply by the number of messages exchanged during an execution of the algorithm, because I think it is interesting to try to find an answer to the question "what are we buying with the added message exchanges?"

The diagram below, adapted from Ryan Barret at [Google](https://snarfed.org/transactions_across_datacenters_io.html), describes some of the aspects of the different options:
-->

这些算法的容错性不同（例如，它们可以容忍的故障类型）。我只是根据执行算法期间交换的消息数量对这些进行了分类，因为我认为有必要找到一个问题的答案“我们用增加的消息交换得到了什么？”

下图，改编自 [Google](https://snarfed.org/transactions_across_datacenters_io.html) 的 Ryan Barret，描述了不同选项的一些方面：

![Comparison of replication methods, from http://www.google.com/events/io/2009/sessions/TransactionsAcrossDatacenters.html](images/google-transact09.png)

<!--
The consistency, latency, throughput, data loss and failover characteristics in the diagram above can really be traced back to the two different replication methods: synchronous replication (e.g. waiting before responding) and asynchronous replication. When you wait, you get worse performance but stronger guarantees. The throughput difference between 2PC and quorum systems will become apparent when we discuss partition (and latency) tolerance.

In that diagram, algorithms enforcing weak (/eventual) consistency are lumped up into one category ("gossip"). However, I will discuss replication methods for weak consistency - gossip and (partial) quorum systems - in more detail. The "transactions" row really refers more to global predicate evaluation, which is not supported in systems with weak consistency (though local predicate evaluation can be supported).
-->

上图中的一致性、延迟、吞吐量、数据丢失和故障转移特性实际上可以追溯到两种不同的复制方法：同步复制（例如，在响应之前等待）和异步复制。当你等待时，你会得到更糟糕的性能，但更好的保障。当我们讨论分区（和延迟）容错时，2PC和仲裁系统之间的吞吐量差异将变得明显。

在该图中，执行弱（或最终）一致性的算法被归为一类（“gossip”）。但是，我将更详细地讨论弱一致性的复制方法—— gossip 和（部分）quorum system （仲裁系统）。 “transactions”行实际上更多地指的是全局断言评估(global predicate evaluation)，在一致性较弱的系统中不支持（尽管可以支持本地断言评估）。

<!--
It is worth noting that systems enforcing weak consistency requirements have fewer generic algorithms, and more techniques that can be selectively applied. Since systems that do not enforce single-copy consistency are free to act like distributed systems consisting of multiple nodes, there are fewer obvious objectives to fix and the focus is more on giving people a way to reason about the characteristics of the system that they have.

For example:

- Client-centric consistency models attempt to provide more intelligible consistency guarantees while allowing for divergence.
- CRDTs (convergent and commutative replicated datatypes) exploit semilattice properties (associativity, commutativity, idempotency) of certain state and operation-based data types.
- Confluence analysis (as in the Bloom language) uses information regarding the monotonicity of computations to maximally exploit disorder.
- PBS (probabilistically bounded staleness) uses simulation and information collected from a real world system to characterize the expected behavior of partial quorum systems.

I'll talk about all of these a bit  further on, first; let's look at the replication algorithms that maintain single-copy consistency.
-->

值得注意的是，执行弱一致性要求的系统具有较少的通用算法，并且可以选择性地应用更多技术。由于不强制执行单一副本一致性的系统可以自由地像由多个节点组成的分布式系统，因此需要修复的目标较少，而重点更多的是让人们了解他们拥有的系统特征。 。

例如：

- 以客户为中心的一致性模型试图在允许分歧的同时提供更可理解的一致性保证。
- CRDT（convergent and commutative replicated datatypes）利用某些状态和基于操作的数据类型的半格特性（关联性、可交换性、幂等性）。
- 汇流分析（如Bloom语言中）使用有关计算单调性的信息来最大限度地利用无序。
- PBS（概率上有界的陈旧性）使用从现实世界系统收集的模拟和信息来表征部分法定系统的预期行为。

首先，我将进一步讨论所有这些问题；让我们看一下维护单拷贝一致性的复制算法。

<!--
## Primary/backup replication

Primary/backup replication (also known as primary copy replication master-slave replication or log shipping) is perhaps the most commonly used replication method, and the most basic algorithm. All updates are performed on the primary, and a log of operations (or alternatively, changes) is shipped across the network to the backup replicas. There are two variants:

- asynchronous primary/backup replication and
- synchronous primary/backup replication
-->

## 主/备份复制(Primary/backup replication)

主/备份复制（也称为主副本复制、主从复制或日志传送）可能是最常用的复制方法，也是最基本的算法。所有更新都在主服务器上执行，操作日志（或者更改）通过网络传送到备份副本。有两种变体：

- 异步主/备份复制和
- 同步主/备份复制

<!--
The synchronous version requires two messages ("update" + "acknowledge receipt") while the asynchronous version could run with just one ("update").

P/B is very common. For example, by default MySQL replication uses the asynchronous variant. MongoDB also uses P/B (with some additional procedures for failover). All operations are performed on one master server, which serializes them to a local log, which is then replicated asynchronously to the backup servers.

As we discussed earlier in the context of asynchronous replication, any asynchronous replication algorithm can only provide weak durability guarantees. In MySQL replication this manifests as replication lag: the asynchronous backups are always at least one operation behind the primary. If the primary fails, then the updates that have not yet been sent to the backups are lost.
-->

同步版本需要两条消息（"update" + "acknowledge receipt"），而异步版本只能运行一条（"update"）。

P/B很常见。例如，默认情况下，MySQL复制使用异步变体。 MongoDB也使用P/B（带有一些额外的故障转移程序）。所有操作都在一个主服务器上执行，主服务器将它们序列化为本地日志，然后将其异步复制到备份服务器。

正如我们之前在异步复制环节中讨论的那样，任何异步复制算法都只能提供弱持久性保证。在MySQL复制中，这表现为复制滞后：异步备份始终至少在主的一次操作后面执行。如果主服务器出现故障，则尚未发送到备份的更新将丢失。

<!--
The synchronous variant of primary/backup replication ensures that writes have been stored on other nodes before returning back to the client - at the cost of waiting for responses from other replicas. However, it is worth noting that even this variant can only offer weak guarantees. Consider the following simple failure scenario:

- the primary receives a write and sends it to the backup
- the backup persists and ACKs the write
- and then primary fails before sending ACK to the client
-->

主/备份复制的同步变体确保在返回到客户端之前已将写入存储在其他节点上——代价是等待来自其他副本的响应。然而，值得注意的是，即使这种变体也只能提供弱保证。请考虑以下简单故障情形：

- 主服务器接收写入并将其发送到备份
- 备份持久化并确认写入
- 然后在向客户端发送ACK之前主失败

<!--
The client now assumes that the commit failed, but the backup committed it; if the backup is promoted to primary, it will be incorrect. Manual cleanup may be needed to reconcile the failed primary or divergent backups.

I am simplifying here of course. While all primary/backup replication algorithms follow the same general messaging pattern, they differ in their handling of failover, replicas being offline for extended periods and so on. However, it is not possible to be resilient to inopportune failures of the primary in this scheme.
-->

客户端现在假定提交失败，但备份已提交；如果备份被提升为主，则不正确。可能需要手动清理以协调发生故障的主或发散备份。

我当然在简化这里。虽然所有主/备复制算法都遵循相同的通用消息传递模式，但它们在故障转移处理方面存在差异，副本在长时间内处于脱机状态等等。但是，在该方案中，不可能适应主的不合时宜的故障。

<!--
What is key in the log-shipping / primary/backup based schemes is that they can only offer a best-effort guarantee (e.g. they are susceptible to lost updates or incorrect updates if nodes fail at inopportune times). Furthermore, P/B schemes are susceptible to split-brain, where the failover to a backup kicks in due to a temporary network issue and causes both the primary and backup to be active at the same time.

To prevent inopportune failures from causing consistency guarantees to be violated; we need to add another round of messaging, which gets us the two phase commit protocol (2PC).
-->

基于日志传送或主/备的方案的关键在于它们只能提供尽力而为的保证（例如，如果节点在不合适的时间失败，则它们容易丢失更新或不正确的更新）。此外，P/B方案容易受到裂脑影响，其中由于临时网络问题而导致备份故障转移并导致主和备同时处于活动状态。

防止不合时宜的故障导致违反一致性保证；我们需要添加另一轮消息传递，这将使我们获得两阶段提交协议（2PC）。

<!--
## Two phase commit (2PC)

[Two phase commit](http://en.wikipedia.org/wiki/Two-phase_commit_protocol) (2PC) is a protocol used in many classic relational databases. For example, MySQL Cluster (not to be confused with the regular MySQL) provides synchronous replication using 2PC. The diagram below illustrates the message flow:
-->

## 两阶段提交（2PC）

[两阶段提交](http://en.wikipedia.org/wiki/Two-phase_commit_protocol)（2PC）是许多经典关系数据库中使用的协议。例如，MySQL Cluster（不要与常规MySQL混淆）使用2PC提供同步复制。下图说明了消息流：

    [ Coordinator ] -> OK to commit?     [ Peers ]
                    <- Yes / No

    [ Coordinator ] -> Commit / Rollback [ Peers ]
                    <- ACK

<!--
In the first phase (voting), the coordinator sends the update to all the participants. Each participant processes the update and votes whether to commit or abort. When voting to commit, the participants store the update onto a temporary area (the write-ahead log). Until the second phase completes, the update is considered temporary.

In the second phase (decision), the coordinator decides the outcome and informs every participant about it. If all participants voted to commit, then the update is taken from the temporary area and made permanent.

Having a second phase in place before the commit is considered permanent is useful, because it allows the system to roll back an update when a node fails. In contrast, in primary/backup ("1PC"), there is no step for rolling back an operation that has failed on some nodes and succeeded on others, and hence the replicas could diverge.
-->

在第一阶段（投票）中，协调员（coordinator）将更新发送给所有参与者。每个参与者处理更新并投票决定是提交还是中止。投票提交时，参与者将更新存储到临时区域（预写日志）。在第二阶段完成之前，更新被视为临时更新。

在第二阶段（决定），协调员决定结果并通知每个参与者。如果所有参与者都投票提交，则更新将从临时区域获取并永久更新。

在提交被持久化之前具有第二阶段是有用的，因为它允许系统在节点失败时会回滚更新。相反，在主/备系统（"1PC"）中，没有用于回滚在某些节点上失败并在其他节点上成功的操作的步骤，因此副本可能会发散。

<!--
2PC is prone to blocking, since a single node failure (participant or coordinator) blocks progress until the node has recovered. Recovery is often possible thanks to the second phase, during which other nodes are informed about the system state. Note that 2PC assumes that the data in stable storage at each node is never lost and that no node crashes forever. Data loss is still possible if the data in the stable storage is corrupted in a crash.

The details of the recovery procedures during node failures are quite complicated so I won't get into the specifics. The major tasks are ensuring that writes to disk are durable (e.g. flushed to disk rather than cached) and making sure that the right recovery decisions are made (e.g. learning the outcome of the round and then redoing or undoing an update locally).
-->

2PC易于阻塞，因为单个节点故障（参与者或协调器）阻止进度直到节点恢复。由于第二阶段，所以是可恢复的，在第二阶段期间，其他节点被告知系统状态。请注意，2PC假定每个节点的稳定存储中的数据永远不会丢失，并且没有节点永远崩溃。如果稳定存储中的数据在崩溃中损坏，则仍可能丢失数据。

节点故障期间恢复过程的细节非常复杂，因此我不会详细介绍。主要任务是确保对磁盘的写入是持久的（例如刷新到磁盘而不是缓存）并确保做出正确的恢复决策（例如，学习轮次的结果，然后在本地重做或撤消更新）。

<!--
As we learned in the chapter regarding CAP, 2PC is a CA - it is not partition tolerant. The failure model that 2PC addresses does not include network partitions; the prescribed way to recover from a node failure is to wait until the network partition heals. There is no safe way to promote a new coordinator if one fails; rather a manual intervention is required. 2PC is also fairly latency-sensitive, since it is a write N-of-N approach in which writes cannot proceed until the slowest node acknowledges them.

2PC strikes a decent balance between performance and fault tolerance, which is why it has been popular in relational databases. However, newer systems often use a partition tolerant consensus algorithm, since such an algorithm can provide automatic recovery from temporary network partitions as well as more graceful handling of increased between-node latency.

Let's look at partition tolerant consensus algorithms next.
-->

正如我们在关于CAP的章节中所了解的那样，2PC是一个CA —— 它不是分区容忍的。 2PC地址的故障模型不包括网络分区；从节点故障中恢复的规定方法是等到网络分区恢复。如果失败，就没有安全的方法来促进新的协调员；而是需要手动干预。2PC也具有相当的延迟敏感性，因为它是一种写入 N-of-N 的方法，在最慢的节点确认它们之前，写入不能进行。

2PC在性能和容错之间取得了不错的平衡，这就是它在关系数据库中流行的原因。然而，较新的系统通常使用分区容忍一致性算法，因为这样的算法可以提供从临时网络分区的自动恢复以及更加优雅地处理增加的节点间延迟。

接下来让我们看一下容忍分区的一致性算法。

<!--
## Partition tolerant consensus algorithms

Partition tolerant consensus algorithms are as far as we're going to go in terms of fault-tolerant algorithms that maintain single-copy consistency. There is a further class of fault tolerant algorithms: algorithms that tolerate [arbitrary (Byzantine) faults](http://en.wikipedia.org/wiki/Byzantine_fault_tolerance); these include nodes that fail by acting maliciously. Such algorithms are rarely used in commercial systems, because they are more expensive to run and more complicated to implement - and hence I will leave them out.

When it comes to partition tolerant consensus algorithms, the most well-known algorithm is the Paxos algorithm. It is, however, notoriously difficult to implement and explain, so I will focus on Raft, a recent (~early 2013) algorithm designed to be easier to teach and implement. Let's first take a look at network partitions and the general characteristics of partition tolerant consensus algorithms.
-->

## 分区容忍一致性算法

就保持单拷贝一致性的容错算法而言，分区容忍一致性算法是我们要做的。还有一类容错算法：容忍任意（拜占庭）故障的算法; 这些包括因恶意行为而失败的节点。 这种算法在商业系统中很少使用，因为它们运行起来更昂贵并且实现起来更复杂 —— 因此我将它们排除在外。

当涉及分区容忍一致性算法时，最着名的算法是Paxos算法。然而，这是非常难以实现和解释的，所以我将重点关注Raft，这是一种最近（〜2013年初）算法，旨在更容易教学和实施。我们首先来看一下网络分区和分区容错一致性算法的一般特征。

<!--
### What is a network partition?

A network partition is the failure of a network link to one or several nodes. The nodes themselves continue to stay active, and they may even be able to receive requests from clients on their side of the network partition. As we learned earlier - during the discussion of the CAP theorem - network partitions do occur and not all systems handle them gracefully.

Network partitions are tricky because during a network partition, it is not possible to distinguish between a failed remote node and the node being unreachable. If a network partition occurs but no nodes fail, then the system is divided into two partitions which are simultaneously active. The two diagrams below illustrate how a network partition can look similar to a node failure.

A system of 2 nodes, with a failure vs. a network partition:
-->

### 什么是网络分区？

网络分区是指到一个或多个节点的网络链路故障。节点本身继续保持活动状态，甚至可以从网络分区一侧的客户端接收请求。正如我们之前所知 —— 在CAP定理的讨论中 —— 确实发生了网络分区，并非所有系统都能优雅地处理它们。

网络分区很棘手，因为在网络分区期间，无法区分节点是发生故障还是无法访问。 如果发生网络分区但没有节点发生故障，则系统将分为两个同时处于活动状态的分区。 下面的两个图说明了网络分区如何看起来与节点故障类似。

一个2节点的系统，具有故障与网络分区：

<img src="images/system-of-2.png" alt="replication" style="max-height: 100px;">

<!--
A system of 3 nodes, with a failure vs. a network partition:
-->

一个3节点的系统，具有故障与网络分区：

<img src="images/system-of-3.png" alt="replication"  style="max-height: 130px;">

<!--
A system that enforces single-copy consistency must have some method to break symmetry: otherwise, it will split into two separate systems, which can diverge from each other and can no longer maintain the illusion of a single copy.

Network partition tolerance for systems that enforce single-copy consistency requires that during a network partition, only one partition of the system remains active since during a network partition it is not possible to prevent divergence (e.g. CAP theorem).
-->

强制执行单一副本一致性的系统必须有一些方法来破坏对称性：否则，它将分成两个独立的系统，这些系统可以彼此分离，不再能够保持单个副本的假设。

对于实施单拷贝一致性的系统的网络分区容差要求在网络分区期间，只有系统的一个分区保持活动，因为在网络分区期间不可能防止分歧（例如CAP定理）。

<!--
### Majority decisions

This is why partition tolerant consensus algorithms rely on a majority vote. Requiring a majority of nodes - rather than all of the nodes (as in 2PC) - to agree on updates allows a minority of the nodes to be down, or slow, or unreachable due to a network partition. As long as `(N/2 + 1)-of-N` nodes are up and accessible, the system can continue to operate.

Partition tolerant consensus algorithms use an odd number of nodes (e.g. 3, 5 or 7). With just two nodes, it is not possible to have a clear majority after a failure. For example, if the number of nodes is three, then the system is resilient to one node failure; with five nodes the system is resilient to two node failures.
-->

### 多数决定

这就是分区容忍共识算法依赖于多数投票的原因。要求大多数节点 —— 而不是所有节点（如2PC中） —— 同意更新允许少数节点由于网络分区而关闭、缓慢或无法访问。只要N 个节点中的 N/2+1 个节点启动并可访问，系统就可以继续运行。

分区容忍一致性算法使用奇数个节点（例如3、5或7）。如果只有两个节点，故障后不可能有明显的多数。例如，如果节点数为3，则系统对一个节点故障具有弹性；通过五个节点，系统可以适应两个节点故障。

<!--
When a network partition occurs, the partitions behave asymmetrically. One partition will contain the majority of the nodes. Minority partitions will stop processing operations to prevent divergence during a network partition, but the majority partition can remain active. This ensures that only a single copy of the system state remains active.

Majorities are also useful because they can tolerate disagreement: if there is a perturbation or failure, the nodes may vote differently. However, since there can be only one majority decision, a temporary disagreement can at most block the protocol from proceeding (giving up liveness) but it cannot violate the single-copy consistency criterion (safety property).
-->

发生网络分区时，分区的行为不对称。一个分区将包含大多数节点。少数分区将停止处理操作以防止在网络分区期间出现分歧，但多数分区可以保持活动状态。这可确保只有系统状态的单个副本保持活动状态。

多数也很有用，因为它们可以容忍分歧：如果存在波动或失败，节点可能会以不同的方式进行投票。但是，由于只有一个多数决定，暂时的分歧最多会阻止协议继续进行（放弃活跃），但不能违反单拷贝一致性标准（安全属性）。

<!--
### Roles

There are two ways one might structure a system: all nodes may have the same responsibilities, or nodes may have separate, distinct roles.

Consensus algorithms for replication generally opt for having distinct roles for each node. Having a single fixed leader or master server is an optimization that makes the system more efficient, since we know that all updates must pass through that server. Nodes that are not the leader just need to forward their requests to the leader.

Note that having distinct roles does not preclude the system from recovering from the failure of the leader (or any other role). Just because roles are fixed during normal operation doesn't mean that one cannot recover from failure by reassigning the roles after a failure (e.g. via a leader election phase). Nodes can reuse the result of a leader election until node failures and/or network partitions occur.

Both Paxos and Raft make use of distinct node roles. In particular, they have a leader node ("proposer" in Paxos) that is responsible for coordination during normal operation. During normal operation, the rest of the nodes are followers ("acceptors" or "voters" in Paxos).
-->

### 角色

可以通过两种方式构建系统：所有节点可以具有相同的职责，或者节点可以具有单独的不同角色。

用于复制的共识算法通常选择为每个节点设置不同的角色。拥有一个固定的领导者或主服务器是一种优化，使系统更有效率，因为我们知道所有更新必须通过该服务器。不是领导者的节点只需要将他们的请求转发给领导者。

请注意，具有不同的角色并不妨碍系统从领导者（或任何其他角色）的失败中恢复。仅仅因为在正常操作期间角色是固定的并不意味着通过在失败之后重新分配角色（例如通过领导者选举阶段）而无法从失败中恢复。节点可以重用领导者选举的结果，直到发生节点故障和/或网络分区。

Paxos和Raft都使用不同的节点角色。特别是，他们有一个领导者节点（Paxos中的“proposer”），负责在正常操作期间进行协调。在正常操作期间，其余节点是跟随者（Paxos中的“acceptors”或“voters”）。

<!--
### Epochs

Each period of normal operation in both Paxos and Raft is called an epoch ("term" in Raft). During each epoch only one node is the designated leader (a similar system is [used in Japan](http://en.wikipedia.org/wiki/Japanese_era_name) where era names change upon imperial succession).
-->

### 时代(Epochs)

Paxos和Raft中的每个正常操作时段称为时期（Raft中的“term”）。在每个时代期间，只有一个节点是指定的领导者（在[日本](http://en.wikipedia.org/wiki/Japanese_era_name)使用类似的系统，其中时代名称在皇家继承时改变）。

<img src="images/epoch.png" alt="replication"  style="max-height: 130px;">

<!--
After a successful election, the same leader coordinates until the end of the epoch. As shown in the diagram above (from the Raft paper), some elections may fail, causing the epoch to end immediately.

Epochs act as a logical clock, allowing other nodes to identify when an outdated node starts communicating - nodes that were partitioned or out of operation will have a smaller epoch number than the current one, and their commands are ignored.
-->

选举成功后，同一个领导者会协调，直到这个时代结束。如上图所示（来自Raft论文），一些选举可能会失败，导致纪元立即结束。

Epoch充当逻辑时钟，允许其他节点识别过时节点何时开始通信 —— 分区或不运行的节点将具有比当前节点更小的纪元号，并且忽略它们的命令。

<!--
### Leader changes via duels

During normal operation, a partition-tolerant consensus algorithm is rather simple. As we've seen earlier, if we didn't care about fault tolerance, we could just use 2PC. Most of the complexity really arises from ensuring that once a consensus decision has been made, it will not be lost and the protocol can handle leader changes as a result of a network or node failure.

All nodes start as followers; one node is elected to be a leader at the start. During normal operation, the leader maintains a heartbeat which allows the followers to detect if the leader fails or becomes partitioned.

When a node detects that a leader has become non-responsive (or, in the initial case, that no leader exists), it switches to an intermediate state (called "candidate" in Raft) where it increments the term/epoch value by one, initiates a leader election and competes to become the new leader.

In order to be elected a leader, a node must receive a majority of the votes. One way to assign votes is to simply assign them on a first-come-first-served basis; this way, a leader will eventually be elected. Adding a random amount of waiting time between attempts at getting elected will reduce the number of nodes that are simultaneously attempting to get elected.
-->

### 领导者(Leader)通过决斗改变

在正常操作期间，分区容忍一致性算法相当简单。正如我们之前看到的，如果我们不关心容错，我们可以使用2PC。大多数复杂性确实来自于确保一旦做出共识决定，它就不会丢失，并且协议可以处理由于网络或节点故障导致的领导者更改。

所有节点都以关注者(followers)身份开始；一个节点在一开始就被选为领导者。在正常操作期间，领导者保持心跳，允许追随者检测领导者是否失败或被分割。

当一个节点检测到一个领导者已经变得没有响应时（或者，在最初的情况下，没有领导者存在），它会切换到一个中间状态（在Raft中称为“candidate”），它将术语/纪元值增加一个，发起领导人选举并竞争成为新的领导者。

为了当选为领导者，节点必须获得大多数选票。分配投票的一种方式是以先到先得的方式分配投票；这样，领导者最终将当选。在尝试选举之间添加随机的等待时间将减少同时尝试当选的节点数量。

<!--
### Numbered proposals within an epoch

During each epoch, the leader proposes one value at a time to be voted upon. Within each epoch, each proposal is numbered with a unique strictly increasing number. The followers (voters / acceptors) accept the first proposal they receive for a particular proposal number.
-->

### 一个时代内的编号提案

在每个时代，领导者一次提出一个值进行投票。在每个时期内，每个提案都以唯一严格增加的数字编号。追随者（选民/接受者）接受他们收到的特定提案编号的第一个提案。

<!--
### Normal operation

During normal operation, all proposals go through the leader node. When a client submits a proposal (e.g. an update operation), the leader contacts all nodes in the quorum. If no competing proposals exist (based on the responses from the followers), the leader proposes the value. If a majority of the followers accept the value, then the value is considered to be accepted.

Since it is possible that another node is also attempting to act as a leader, we need to ensure that once a single proposal has been accepted, its value can never change. Otherwise a proposal that has already been accepted might for example be reverted by a competing leader. Lamport states this as:
-->

### 普通操作

在正常操作期间，所有提议都通过领导节点。当客户端提交提议（例如，更新操作）时，领导者联系仲裁中的所有节点。如果不存在竞争提议（基于来自追随者的回复），领导者会提出价值。如果大多数粉丝接受该值，则认为该值被接受。

由于另一个节点可能也试图充当领导者，我们需要确保一旦接受单个提案，其值就永远不会改变。否则，已经被接受的提议可能例如由竞争领导者恢复。Lamport称这为：

<!--
> P2: If a proposal with value `v` is chosen, then every higher-numbered proposal that is chosen has value `v`.

Ensuring that this property holds requires that both followers and proposers are constrained by the algorithm from ever changing a value that has been accepted by a majority. Note that "the value can never change" refers to the value of a single execution (or run / instance / decision) of the protocol. A typical replication algorithm will run multiple executions of the algorithm, but most discussions of the algorithm focus on a single run to keep things simple. We want to prevent the decision history from being altered or overwritten.

In order to enforce this property, the proposers must first ask the followers for their (highest numbered) accepted proposal and value. If the proposer finds out that a proposal already exists, then it must simply complete this execution of the protocol, rather than making its own proposal. Lamport states this as:
-->

> P2：如果选择了值为v的提案，则所选的每个编号较高的提案都具有值v。

确保此属性保持不变要求跟随者和提议者都受到算法的限制，不会更改已被多数人接受的值。请注意，“值永远不会更改”是指协议的单个执行（或运行/实例/决策）的值。典型的复制算法将运行算法的多次执行，但算法的大多数讨论都集中在单次运行以保持简单。我们希望防止更改或覆盖决策历史记录。

为了强制执行此属性，提议者必须首先向关注者询问他们（编号最高）的提议和价值。如果提议者发现提案已经存在，那么它必须简单地完成协议的执行，而不是提出自己的提议。Lamport称这为：

<!--
> P2b. If a proposal with value `v` is chosen, then every higher-numbered proposal issued by any proposer has value `v`.

More specifically:

> P2c. For any `v` and `n`, if a proposal with value `v` and number `n` is issued [by a leader], then there is a set `S` consisting of a majority of acceptors [followers] such that either (a) no acceptor in `S` has accepted any proposal numbered less than `n`, or (b) `v` is the value of the highest-numbered proposal among all proposals numbered less than `n` accepted by the followers in `S`.
-->

> P2b：如果选择了具有值 `v` 的提议，则由任何提议者发布的每个更高编号的提议具有值 `v`。

进一步来说：

> P2c：对于任何`v`和`n`，如果[由领导者]发布具有值`v`和数字`n`的提议，则存在由大多数接受者[跟随者]组成的集合`S`，使得 a) `S`中的接受者都没有接受任何编号小于`n`的提案，或 b) `v`是所有提案中编号最大的提案的值，其编号小于`n`中的关注者所接受的提议。

<!--
This is the core of the Paxos algorithm, as well as algorithms derived from it. The value to be proposed is not chosen until the second phase of the protocol. Proposers must sometimes simply retransmit a previously made decision to ensure safety (e.g. clause b in P2c) until they reach a point where they know that they are free to impose their own proposal value (e.g. clause a).

If multiple previous proposals exist, then the highest-numbered proposal value is proposed. Proposers may only attempt to impose their own value if there are no competing proposals at all.

To ensure that no competing proposals emerge between the time the proposer asks each acceptor about its most recent value, the proposer asks the followers not to accept proposals with lower proposal numbers than the current one.

Putting the pieces together, reaching a decision using Paxos requires two rounds of communication:
-->

这是Paxos算法以及从中派生的算法的核心。直到协议的第二阶段才选择要提出的值。提议者有时必须简单地转发先前做出的确保安全的决定（例如P2c中的条款b），直到他们知道他们可以自由施加他们自己的提案价值（例如条款a）。

如果存在多个先前提议，则建议使用编号最高的提议值。如果根本没有竞争性提案，提案人可能只会尝试强加自己的价值。

为了确保在提议者询问每个接受者关于其最新值的时间之间没有出现竞争提议，提议者要求关注者不接受提议编号低于当前提案的提案。

将各个部分放在一起，使用Paxos做出决定需要两轮沟通：

    [ Proposer ] -> Prepare(n)                                [ Followers ]
                 <- Promise(n; previous proposal number
                    and previous value if accepted a
                    proposal in the past)

    [ Proposer ] -> AcceptRequest(n, own value or the value   [ Followers ]
                    associated with the highest proposal number
                    reported by the followers)
                    <- Accepted(n, value)

<!--
The prepare stage allows the proposer to learn of any competing or previous proposals. The second phase is where either a new value or a previously accepted value is proposed. In some cases - such as if two proposers are active at the same time (dueling); if messages are lost; or if a majority of the nodes have failed - then no proposal is accepted by a majority. But this is acceptable, since the decision rule for what value to propose converges towards a single value (the one with the highest proposal number in the previous attempt).

Indeed, according to the FLP impossibility result, this is the best we can do: algorithms that solve the consensus problem must either give up safety or liveness when the guarantees regarding bounds on message delivery do not hold. Paxos gives up liveness: it may have to delay decisions indefinitely until a point in time where there are no competing leaders, and a majority of nodes accept a proposal. This is preferable to violating the safety guarantees.
-->

准备阶段允许提议者了解任何竞争或以前的提议。第二阶段是建议新值或先前接受的值。在某些情况下 —— 例如，如果两个提议者同时活跃（决斗）；如果消息丢失；或者如果大多数节点发生故障 —— 那么大多数人都不会接受任何提案。但这是可以接受的，因为提议的值的决策规则会收敛到单个值（前一次尝试中具有最高提案编号的值）。

实际上，根据FLP不可能性结果，这是我们能做的最好的事情：解决共识问题的算法必须在信息传递边界的保证不成立时放弃安全性或活跃性。Paxos放弃了活跃性：它可能必须无限期地推迟决策，直到没有竞争领导者的时间点，并且大多数节点接受提议。这比违反安全保障更可取。

<!--
Of course, implementing this algorithm is much harder than it sounds. There are many small concerns which add up to a fairly significant amount of code even in the hands of experts. These are issues such as:

- practical optimizations:
  - avoiding repeated leader election via leadership leases (rather than heartbeats)
  - avoiding repeated propose messages when in a stable state where the leader identity does not change
- ensuring that followers and proposers do not lose items in stable storage and that results stored in stable storage are not subtly corrupted (e.g. disk corruption)
- enabling cluster membership to change in a safe manner (e.g. base Paxos depends on the fact that majorities always intersect in one node, which does not hold if the membership can change arbitrarily)
- procedures for bringing a new replica up to date in a safe and efficient manner after a crash, disk loss or when a new node is provisioned
- procedures for snapshotting and garbage collecting the data required to guarantee safety after some reasonable period (e.g. balancing storage requirements and fault tolerance requirements)

Google's [Paxos Made Live](http://research.google.com/archive/paxos_made_live.html) paper details some of these challenges.
-->

当然，实现这个算法比听起来要困难得多。即使在专家手中，也存在许多小问题，这些问题相当于相当多的代码。这些问题包括：

- 实际优化：
  - 避免通过领导租约（而不是心跳）重复领导人选举
  - 避免在领导者身份不变的稳定状态下重复建议消息
- 确保关注者和提议者不会丢失稳定存储中的项目，并且存储在稳定存储中的结果不会轻微损坏（例如磁盘损坏）
- 使集群成员资格能够以安全的方式发生变化（例如，基础Paxos取决于多数总是在一个节点中相交的事实，如果成员资格可以任意改变则不成立）
- 在崩溃，磁盘丢失或配置新节点后以安全有效的方式更新新副本的过程
- 快照和垃圾收集程序，以确保在一段合理时间后保证安全所需的数据（例如，平衡存储要求和容错要求）

Google的[Paxos Made Live](http://research.google.com/archive/paxos_made_live.html)文章详细介绍了其中的一些挑战。

<!--
## Partition-tolerant consensus algorithms: Paxos, Raft, ZAB

Hopefully, this has given you a sense of how a partition-tolerant consensus algorithm works. I encourage you to read one of the papers in the further reading section to get a grasp of the specifics of the different algorithms.

*Paxos*. Paxos is one of the most important algorithms when writing strongly consistent partition tolerant replicated systems. It is used in many of Google's systems, including the [Chubby lock manager](http://research.google.com/archive/chubby.html) used by [BigTable](http://research.google.com/archive/bigtable.html)/[Megastore](http://research.google.com/pubs/pub36971.html), the Google File System as well as [Spanner](http://research.google.com/archive/spanner.html).
-->

## 分区容忍一致性算法：Paxos，Raft，ZAB

希望这能让您了解分区容错一致性算法的工作原理。我鼓励您阅读下一部分中的一篇论文，以掌握不同算法的具体内容。

*Paxos* Paxos是编写强一致的分区容错复制系统时最重要的算法之一。它在Google的许多系统中使用，包括[BigTable](http://research.google.com/archive/bigtable.html) / [Megastore](http://research.google.com/pubs/pub36971.html)使用的[Chubby锁管理器](http://research.google.com/archive/chubby.html)，Google文件系统以及[Spanner](http://research.google.com/archive/spanner.html)。

<!--
Paxos is named after the Greek island of Paxos, and was originally presented by Leslie Lamport in a paper called "The Part-Time Parliament" in 1998. It is often considered to be difficult to implement, and there have been a series of papers from companies with considerable distributed systems expertise explaining further practical details (see the further reading). You might want to read Lamport's commentary on this issue [here](http://research.microsoft.com/en-us/um/people/lamport/pubs/pubs.html#lamport-paxos) and [here](http://research.microsoft.com/en-us/um/people/lamport/pubs/pubs.html#paxos-simple).

The issues mostly relate to the fact that Paxos is described in terms of a single round of consensus decision making, but an actual working implementation usually wants to run multiple rounds of consensus efficiently. This has led to the development of many [extensions on the core protocol](http://en.wikipedia.org/wiki/Paxos_algorithm) that anyone interested in building a Paxos-based system still needs to digest. Furthermore, there are additional practical challenges such as how to facilitate cluster membership change.
-->

Paxos以希腊的Paxos岛命名，最初由Leslie Lamport于1998年在一篇名为 "The Part-Time Parliament" 的论文中提出。它通常被认为难以实施，并且有一系列论文来自具有相当分布式系统专业知识的公司解释了更多实际细节（参见进一步阅读）你可能想在[这里](http://research.microsoft.com/en-us/um/people/lamport/pubs/pubs.html#lamport-paxos)和[这里](http://research.microsoft.com/en-us/um/people/lamport/pubs/pubs.html#paxos-simple)阅读Lamport对这个问题的评论。

这些问题主要涉及Paxos是根据一轮共识决策进行描述的事实，但实际工作实施通常希望有效地进行多轮共识。这导致了对核心协议的[许多扩展](http://en.wikipedia.org/wiki/Paxos_algorithm)的开发，任何对构建基于Paxos的系统感兴趣的人仍然需要消化。此外，还存在其他实际挑战，例如如何促进集群成员资格变更。

<!--
*ZAB*. ZAB - the Zookeeper Atomic Broadcast protocol is used in Apache Zookeeper. Zookeeper is a system which provides coordination primitives for distributed systems, and is used by many Hadoop-centric distributed systems for coordination (e.g. [HBase](http://hbase.apache.org/), [Storm](http://storm-project.net/), [Kafka](http://kafka.apache.org/)). Zookeeper is basically the open source community's version of Chubby. Technically speaking atomic broadcast is a problem different from pure consensus, but it still falls under the category of partition tolerant algorithms that ensure strong consistency.

*Raft*. Raft is a recent (2013) addition to this family of algorithms. It is designed to be easier to teach than Paxos, while providing the same guarantees. In particular, the different parts of the algorithm are more clearly separated and the paper also describes a mechanism for cluster membership change. It has recently seen adoption in [etcd](https://github.com/coreos/etcd) inspired by ZooKeeper.
-->

*ZAB* ZAB —— Zookeeper Atomic Broadcast协议用于Apache Zookeeper。Zookeeper是一个为分布式系统提供协调原语的系统，并被许多以Hadoop为中心的分布式系统用于协调（例如[HBase](http://hbase.apache.org/)、[Storm](http://storm-project.net/)、[Kafka](http://kafka.apache.org/)）。 Zookeeper基本上是开源社区的Chubby版本。从技术上讲，原子广播是一个与纯粹共识不同的问题，但它仍然属于确保强一致性的分区容错算法类别。

*Raft* Raft是最近（2013年）这一系列算法的补充。它的设计比Paxos更容易教学，同时提供相同的保证。特别是，算法的不同部分更加清晰，文章还描述了集群成员变更的机制。它最近出现在受到ZooKeeper启发的[etcd](https://github.com/coreos/etcd)中。

<!--
## Replication methods with strong consistency

In this chapter, we took a look at replication methods that enforce strong consistency. Starting with a contrast between synchronous work and asynchronous work, we worked our way up to algorithms that are tolerant of increasingly complex failures. Here are some of the key characteristics of each of the algorithms:
-->

## 具有强一致性的复制方法

在本章中，我们了解了强制一致性的复制方法。从同步工作和异步工作之间的对比开始，我们逐步采用能够容忍日益复杂的故障的算法。以下是每种算法的一些关键特征：

<!--
#### Primary/Backup

- Single, static master
- Replicated log, slaves are not involved in executing operations
- No bounds on replication delay
- Not partition tolerant
- Manual/ad-hoc failover, not fault tolerant, "hot backup"
-->

#### 主/备

- 单，静态主
- 复制日志，从属不参与执行操作
- 复制延迟没有限制
- 无分区容忍
- 手动/临时故障转移，而不是容错，“热备份”

<!--
#### 2PC

- Unanimous vote: commit or abort
- Static master
- 2PC cannot survive simultaneous failure of the coordinator and a node during a commit
- Not partition tolerant, tail latency sensitive

#### Paxos

- Majority vote
- Dynamic master
- Robust to n/2-1 simultaneous failures as part of protocol
- Less sensitive to tail latency
-->

#### 2PC

- 一致投票：提交或中止
- 静态主
- 在提交期间，2PC无法在协调器和节点同时发生故障
- 不分区容忍，尾部延迟敏感

#### Paxos

- 多数票
- 动态大师
- 作为协议的一部分，强大到 n/2 - 1 同时失败

---

## Further reading

#### Primary-backup and 2PC

- [Replication techniques for availability](http://scholar.google.com/scholar?q=Replication+techniques+for+availability) - Robbert van Renesse & Rachid Guerraoui, 2010
- [Concurrency Control and Recovery in Database Systems](http://research.microsoft.com/en-us/people/philbe/ccontrol.aspx)

#### Paxos

- [The Part-Time Parliament](http://research.microsoft.com/users/lamport/pubs/lamport-paxos.pdf) - Leslie Lamport
- [Paxos Made Simple](http://research.microsoft.com/users/lamport/pubs/paxos-simple.pdf) - Leslie Lamport, 2001
- [Paxos Made Live - An Engineering Perspective](http://research.google.com/archive/paxos_made_live.html) - Chandra et al
- [Paxos Made Practical](http://scholar.google.com/scholar?q=Paxos+Made+Practical) - Mazieres, 2007
- [Revisiting the Paxos Algorithm](http://groups.csail.mit.edu/tds/paxos.html) - Lynch et al
- [How to build a highly available system with consensus](http://research.microsoft.com/lampson/58-Consensus/Acrobat.pdf) - Butler Lampson
- [Reconfiguring a State Machine](http://research.microsoft.com/en-us/um/people/lamport/pubs/reconfiguration-tutorial.pdf) - Lamport et al - changing cluster membership
- [Implementing Fault-Tolerant Services Using the State Machine Approach: a Tutorial](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.20.4762) - Fred Schneider

#### Raft and ZAB

- [In Search of an Understandable Consensus Algorithm](https://ramcloud.stanford.edu/wiki/download/attachments/11370504/raft.pdf), Diego Ongaro, John Ousterhout, 2013
- [Raft Lecture - User Study](http://www.youtube.com/watch?v=YbZ3zDzDnrw)
- [A simple totally ordered broadcast protocol](http://labs.yahoo.com/publication/a-simple-totally-ordered-broadcast-protocol/) - Junqueira, Reed, 2008
- [ZooKeeper Atomic Broadcast](http://labs.yahoo.com/publication/zab-high-performance-broadcast-for-primary-backup-systems/) - Reed, 2011
