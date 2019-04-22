# %chapter_number%. 上下的抽象层次

<!--
# %chapter_number%. Up and down the level of abstraction
-->

<!--
In this chapter, we'll travel up and down the level of abstraction, look at some impossibility results (CAP and FLP), and then travel back down for the sake of performance.
-->

在本章中，我们将遍历不同的抽象层次，查看一些不可能性结果（CAP和FLP），然后为了提高性能而向下移动。

<!--
If you've done any programming, the idea of levels of abstraction is probably familiar to you. You'll always work at some level of abstraction, interface with a lower level layer through some API, and probably provide some higher-level API or user interface to your users. The seven-layer [OSI model of computer networking](http://en.wikipedia.org/wiki/OSI_model) is a good example of this.
-->

如果已经有编程基础，那么抽象级别的概念可能对你来说很熟悉。你将始终在某种抽象级别工作，通过某些API与较低级别层接口，并可能为你的用户提供一些更高级别的API或用户界面。计算机网络的七层[OSI模型](http://en.wikipedia.org/wiki/OSI_model)就是一个很好的例子。

<!--
Distributed programming is, I'd assert, in large part dealing with consequences of distribution (duh!). That is, there is a tension between the reality that there are many nodes and with our desire for systems that "work like a single system". That means finding a good abstraction that balances what is possible with what is understandable and performant.
-->

我确信，分布式编程在很大程度上处理了分发的结果（呃！）。也就是说，存在许多节点的现实与我们对“像单一系统一样工作”的系统的需求之间存在着紧张关系。这意味着要找到一个好的抽象，以平衡可能与可理解和高效的东西。

<!--
What do we mean when say X is more abstract than Y? First, that X does not introduce anything new or fundamentally different from Y. In fact, X may remove some aspects of Y or present them in a way that makes them more manageable.
Second, that X is in some sense easier to grasp than Y, assuming that the things that X removed from Y are not important to the matter at hand.
-->

当说X比Y更抽象时，我们的意思是什么？首先，X不会引入任何新的或与Y基本不同的东西。事实上，X可能会删除Y的某些方面，或以一种使它们更易于管理的方式呈现它们。其次，X在某种意义上比Y更容易掌握，假设X从Y中移除的东西对于手头的事情并不重要。

<!--
As [Nietzsche](http://oregonstate.edu/instruct/phl201/modules/Philosophers/Nietzsche/Truth_and_Lie_in_an_Extra-Moral_Sense.htm) wrote:

> Every concept originates through our equating what is unequal. No leaf ever wholly equals another, and the concept "leaf" is formed through an arbitrary abstraction from these individual differences, through forgetting the distinctions; and now it gives rise to the idea that in nature there might be something besides the leaves which would be "leaf" - some kind of original form after which all leaves have been woven, marked, copied, colored, curled, and painted, but by unskilled hands, so that no copy turned out to be a correct, reliable, and faithful image of the original form.
-->

正如[尼采](http://oregonstate.edu/instruct/phl201/modules/Philosophers/Nietzsche/Truth_and_Lie_in_an_Extra-Moral_Sense.htm)写道：

> 每个概念都源于我们将不平等的东西等同起来。没有叶子完全等于另一个叶子，“叶子”的概念是通过对这些个体差异的任意抽象，通过忘记区别而形成的;现在它产生了这样的想法：在自然界中可能存在除了叶子之外还有“叶子”的东西 - 某种原始形式，之后所有的叶子都被编织，标记，复制，着色，卷曲和涂漆，但是由不熟练的手，所以没有副本证明是原始形式的正确，可靠和忠实的形象。

<!--
Abstractions, fundamentally, are fake. Every situation is unique, as is every node. But abstractions make the world manageable: simpler problem statements - free of reality - are much more analytically tractable and provided that we did not ignore anything essential, the solutions are widely applicable.
-->

从根本上说，抽象是假的。每种情况都是独特的，每个节点都是如此。但是抽象使得世界变得易于管理：更简单的问题陈述——没有现实——更易于分析，只要我们不忽视任何必要的东西，解决方案就可以广泛应用。

<!--
Indeed, if the things that we kept around are essential, then the results we can derive will be widely applicable. This is why impossibility results are so important: they take the simplest possible formulation of a problem, and demonstrate that it is impossible to solve within some set of constraints or assumptions.
-->

实际上，如果我们保留的东西是必不可少的，那么我们可以得出的结论将是广泛适用的。这就是不可能性结果如此重要的原因：它们采用最简单的问题形式，并证明在某些约束或假设中无法解决。

<!--
All abstractions ignore something in favor of equating things that are in reality unique. The trick is to get rid of everything that is not essential. How do you know what is essential? Well, you probably won't know a priori.

Every time we exclude some aspect of a system from our specification of the system, we risk introducing a source of error and/or a performance issue. That's why sometimes we need to go in the other direction, and selectively introduce some aspects of real hardware and the real-world problem back. It may be sufficient to reintroduce some specific hardware characteristics (e.g. physical sequentiality) or other physical characteristics to get a system that performs well enough.

With this in mind, what is the least amount of reality we can keep around while still working with something that is still recognizable as a distributed system? A system model is a specification of the characteristics we consider important; having specified one, we can then take a look at some impossibility results and challenges.
-->

所有抽象都忽略了一些有利于将事物等同于事物的东西。诀窍是摆脱一切不重要的东西。你怎么知道什么是必要的？好吧，你可能不知道先验。

每次我们从系统规范中排除系统的某些方面时，我们都会冒险引入错误源和/或性能问题。这就是为什么有时我们需要走向另一个方向，并有选择地介绍真实硬件和现实世界问题的某些方面。重新引入一些特定的硬件特性（例如物理顺序性）或其他物理特性以获得足够好的系统可能就足够了。

考虑到这一点，在仍然可以使用仍然可以识别为分布式系统的东西的同时，我们可以保留的最少量的现实是什么？系统模型是我们认为重要的特征的规范；如果指定了一个，我们就可以看看一些不可能的结果和挑战。


<!--
## A system model

A key property of distributed systems is distribution. More specifically, programs in a distributed system:

- run concurrently on independent nodes ...
- are connected by a network that may introduce nondeterminism and message loss ...
- and have no shared memory or shared clock.
-->

## 系统模型

分布式系统的关键属性是分布。更具体地说，分布式系统中的程序：

- 在独立节点上并发运行...
- 使用具有不确定性和消息丢失的网络连接......
- 没有共享内存或共享时钟。

<!--
There are many implications:

- each node executes a program concurrently
- knowledge is local: nodes have fast access only to their local state, and any information about global state is potentially out of date
- nodes can fail and recover from failure independently
- messages can be delayed or lost (independent of node failure; it is not easy to distinguish network failure and node failure)
- and clocks are not synchronized across nodes (local timestamps do not correspond to the global real time order, which cannot be easily observed)

A system model enumerates the many assumptions associated with a particular system design.
-->

这有很多含义：

- 每个节点同时执行一个程序
- 知识是本地的：节点只能快速访问其本地状态，任何有关全局状态的信息都可能过时
- 节点可能会失败并独立地从故障中恢复
- 消息可能会延迟或丢失（与节点故障无关；区分网络故障和节点故障并不容易）
- 并且时钟不跨节点同步（本地时间戳与全局实时顺序不对应，不容易观察到）

系统模型列举了与特定系统设计相关的许多假设。

<!--
<dl>
  <dt>System model</dt>
  <dd>a set of assumptions about the environment and facilities on which a distributed system is implemented</dd>
</dl>
-->

<dl>
  <dt>系统模型</dt>
  <dd>关于实现分布式系统的环境和设施的一组假设</dd>
</dl>


<!--
System models vary in their assumptions about the environment and facilities. These assumptions include:

- what capabilities the nodes have and how they may fail
- how communication links operate and how they may fail and
- properties of the overall system, such as assumptions about time and order
-->

系统模型对环境和设施的假设各不相同。 这些假设包括：

- 节点具有哪些功能以及它们可能如何失败
- 通信链路如何运作以及它们如何失效
- 整个系统的属性，例如关于时间和顺序的假设

<!--
A robust system model is one that makes the weakest assumptions: any algorithm written for such a system is very tolerant of different environments, since it makes very few and very weak assumptions.

On the other hand, we can create a system model that is easy to reason about by making strong assumptions. For example, assuming that nodes do not fail means that our algorithm does not need to handle node failures. However, such a system model is unrealistic and hence hard to apply into practice.

Let's look at the properties of nodes, links and time and order in more detail.
-->

一个健壮的系统模型是做出最弱假设的模型：为这样的系统编写的任何算法都非常容忍不同的环境，因为它做出非常少且非常弱的假设。

另一方面，我们可以通过做出强有力的假设来创建一个易于推理的系统模型。例如，假设节点没有失败意味着我们的算法不需要处理节点故障。 然而，这种系统模型是不现实的，因此难以应用于实践中。

让我们更详细地看一下节点、链接以及时间和顺序的属性。

<!--
### Nodes in our system model

Nodes serve as hosts for computation and storage. They have:

- the ability to execute a program
- the ability to store data into volatile memory (which can be lost upon failure) and into stable state (which can be read after a failure)
- a clock (which may or may not be assumed to be accurate)
-->

### 我们系统模型中的节点

节点作为计算和存储的主机。它们有：

- 执行程序的能力
- 能够将数据存储到易失性存储器（可能在发生故障时丢失）和固定存储（可在故障恢复后读取）
- 时钟（可能会或可能不会被认为是准确的）

<!--
Nodes execute deterministic algorithms: the local computation, the local state after the computation, and the messages sent are determined uniquely by the message received and local state when the message was received.
-->

节点执行确定性算法：本地计算，计算后的本地状态以及发送的消息由接收到的消息和接收消息时的本地状态唯一确定。

<!--
There are many possible failure models which describe the ways in which nodes can fail. In practice, most systems assume a crash-recovery failure model: that is, nodes can only fail by crashing, and can (possibly) recover after crashing at some later point.

Another alternative is to assume that nodes can fail by misbehaving in any arbitrary way. This is known as [Byzantine fault tolerance](http://en.wikipedia.org/wiki/Byzantine_fault_tolerance). Byzantine faults are rarely handled in real world commercial systems, because algorithms resilient to arbitrary faults are more expensive to run and more complex to implement. I will not discuss them here.
-->

有许多可能的故障模型描述了节点失败的方式。在实践中，大多数系统都采用崩溃恢复故障模型：即，节点只能通过崩溃而失败，并且能够（可能）在稍后崩溃后恢复。

另一种选择是假设节点可能以任意方式行为不端而失败。这被称为[拜占庭容错](http://en.wikipedia.org/wiki/Byzantine_fault_tolerance)。在现实世界的商业系统中很少处理拜占庭故障，因为对任意故障具有弹性的算法运行起来更昂贵并且实现起来更复杂。我不会在这里讨论它们。

<!--
### Communication links in our system model

Communication links connect individual nodes to each other, and allow messages to be sent in either direction. Many books that discuss distributed algorithms assume that there are individual links between each pair of nodes, that the links provide FIFO (first in, first out) order for messages, that they can only deliver messages that were sent, and that sent messages can be lost.

Some algorithms assume that the network is reliable: that messages are never lost and never delayed indefinitely. This may be a reasonable assumption for some real-world settings, but in general it is preferable to consider the network to be unreliable and subject to message loss and delays.
-->

### 系统模型中的通信链路

通信链路将各个节点相互连接，并允许以任一方向发送消息。讨论分布式算法的许多书籍假设每对节点之间存在单独的连接，连接为消息提供FIFO（先进先出）顺序，它们只能传递已发送的消息，并且发送的消息可以是丢失。

一些算法假设网络是可靠的：消息永远不会丢失，永远不会无限延迟。对于某些真实世界的设置，这可能是合理的假设，但一般来说，最好考虑网络不可靠并受到消息丢失和延迟的影响。

<!--
A network partition occurs when the network fails while the nodes themselves remain operational. When this occurs, messages may be lost or delayed until the network partition is repaired. Partitioned nodes may be accessible by some clients, and so must be treated differently from crashed nodes. The diagram below illustrates a node failure vs. a network partition:
-->

网络分区是指节点本身仍然可以运行，但是网络发生故障。发生这种情况时，消息可能会丢失或延迟，直到网络分区修复。某些客户端可以访问分区节点，因此必须与崩溃节点区别对待。下图说明了节点故障与网络分区的关系：

<img src="images/system-of-2.png" alt="replication" style="max-height: 100px;">

<!--
It is rare to make further assumptions about communication links. We could assume that links only work in one direction, or we could introduce different communication costs (e.g. latency due to physical distance) for different links. However, these are rarely concerns in commercial environments except for long-distance links (WAN latency) and so I will not discuss them here; a more detailed model of costs and topology allows for better optimization at the cost of complexity.
-->

很少对通信链路做出进一步的假设。我们可以假设链路仅在一个方向上工作，或者我们可以为不同的链路引入不同的通信成本（例如，由于物理距离引起的延迟）。但是，除了长距离链路（WAN延迟）之外，这些在商业环境中很少受到关注，所以我不会在这里讨论它们。更详细的成本和拓扑模型允许以复杂性为代价进行更好的优化。

<!--
### Timing / ordering assumptions

One of the consequences of physical distribution is that each node experiences the world in a unique manner. This is inescapable, because information can only travel at the speed of light. If nodes are at different distances from each other, then any messages sent from one node to the others will arrive at a different time and potentially in a different order at the other nodes.

Timing assumptions are a convenient shorthand for capturing assumptions about the extent to which we take this reality into account. The two main alternatives are:
-->

### 时间/顺序假设

物理分布的结果之一是每个节点以独特的方式体验世界。这是不可避免的，因为信息只能以光速传播。如果节点之间距离不同，则从一个节点发送到其他节点的任何消息将在不同的时间到达，并且可能在其他节点处以不同的顺序到达。

时间假设是一种方便的简写，用于捕捉关于我们将这一现实考虑在内的程度的假设。两个主要的替代方案是：

<!--
<dl>
  <dt>Synchronous system model</dt>
  <dd>Processes execute in lock-step; there is a known upper bound on message transmission delay; each process has an accurate clock</dd>
  <dt>Asynchronous system model</dt>
  <dd>No timing assumptions - e.g. processes execute at independent rates; there is no bound on message transmission delay; useful clocks do not exist</dd>
</dl>
-->

<dl>
  <dt>同步系统模型</dt>
  <dd>进程以锁步执行; 消息传输延迟有一个已知的上限; 每个过程都有一个准确的时钟</dd>
  <dt>异步系统模型</dt>
  <dd>没有时间假设——例如：流程以独立的速率执行; 消息传输延迟没有限制; 不存在可用的时钟</dd>
</dl>

<!--
The synchronous system model imposes many constraints on time and order. It essentially assumes that the nodes have the same experience: that messages that are sent are always received within a particular maximum transmission delay, and that processes execute in lock-step. This is convenient, because it allows you as the system designer to make assumptions about time and order, while the asynchronous system model doesn't.

Asynchronicity is a non-assumption: it just assumes that you can't rely on timing (or a "time sensor").
-->

同步系统模型对时间和顺序施加了许多约束。它基本上假定节点具有相同的体验：发送的消息总是在特定的最大传输延迟内接收，并且该进程在锁定步骤中执行。这很方便，因为它允许你作为系统设计者对时间和顺序进行假设，而异步系统模型则不然。

异步性是一种非假设：它只是假设你不能依赖于时间（或“时间传感器”）。

<!--
It is easier to solve problems in the synchronous system model, because assumptions about execution speeds, maximum message transmission delays and clock accuracy all help in solving problems since you can make inferences based on those assumptions and rule out inconvenient failure scenarios by assuming they never occur.

Of course, assuming the synchronous system model is not particularly realistic. Real-world networks are subject to failures and there are no hard bounds on message delay. Real world systems are at best partially synchronous: they may occasionally work correctly and provide some upper bounds, but there will be times where messages are delayed indefinitely and clocks are out of sync. I won't really discuss algorithms for synchronous systems here, but you will probably run into them in many other introductory books because they are analytically easier (but unrealistic).
-->

解决同步系统模型中的问题更容易，因为关于执行速度，最大消息传输延迟和时钟精度的假设都有助于解决问题，因为你可以根据这些假设做出推断，并通过假设它们永远不会发生来排除不方便的故障情况。

当然，假设同步系统模型不是特别现实。现实世界的网络容易出现故障，并且消息延迟没有硬性限制。真实世界的系统最多是部分同步的：它们偶尔可以正常工作并提供一些上限，但有时候消息会无限延迟并且时钟不同步。我不会在这里讨论同步系统的算法，但是你可能会在许多其他介绍性书籍中遇到它们，因为它们在分析上更容易（但不切实际）。

<!--
### The consensus problem

During the rest of this text, we'll vary the parameters of the system model. Next, we'll look at how varying two system properties:

- whether or not network partitions are included in the failure model, and
- synchronous vs. asynchronous timing assumptions

influence the system design choices by discussing two impossibility results (FLP and CAP).
-->

### 共识问题

在本文的其余部分，我们将改变系统模型的参数。接下来，我们将看看如何改变两个系统属性：

- 网络分区是否包含在故障模型中，以及
- 同步与异步时序假设

通过讨论两个不可能性结果（FLP和CAP）来影响系统设计选择。

<!--
Of course, in order to have a discussion, we also need to introduce a problem to solve. The problem I'm going to discuss is the [consensus problem](http://en.wikipedia.org/wiki/Consensus_%28computer_science%29).

Several computers (or nodes) achieve consensus if they all agree on some value. More formally:

1. Agreement: Every correct process must agree on the same value.
2. Integrity: Every correct process decides at most one value, and if it decides some value, then it must have been proposed by some process.
3. Termination: All processes eventually reach a decision.
4. Validity: If all correct processes propose the same value V, then all correct processes decide V.
-->

当然，为了进行讨论，我们还需要解决一个问题。我要讨论的问题是[共识问题](http://en.wikipedia.org/wiki/Consensus_%28computer_science%29)。

如果几个计算机（或节点）都同意一些值，那么它们就会达成共识。更正式的：

1. 协议：每个正确的程序必须就相同的值达成一致。
2. 完整性：每个正确的程序最多决定一个值，如果它决定某个值，那么它必须由某个程序提出。
3. 终止：所有程序最终都会做出决定。
4. 有效性：如果所有正确的程序提出相同的值V，那么所有正确的程序决定V.

<!--
The consensus problem is at the core of many commercial distributed systems. After all, we want the reliability and performance of a distributed system without having to deal with the consequences of distribution (e.g. disagreements / divergence between nodes), and solving the consensus problem makes it possible to solve several related, more advanced problems such as atomic broadcast and atomic commit.
-->

共识问题是许多商业分布式系统的核心。毕竟，我们需要分布式系统的可靠性和性能，而不必处理分布的后果（例如节点之间的分歧/发散），并且解决共识问题可以解决几个相关的，更高级的问题，例如原子广播和原子提交。

<!--
### Two impossibility results

The first impossibility result, known as the FLP impossibility result, is an impossibility result that is particularly relevant to people who design distributed algorithms. The second - the CAP theorem - is a related result that is more relevant to practitioners; people who need to choose between different system designs but who are not directly concerned with the design of algorithms.
-->

### 两个不可能

第一个不可能性结果，称为 FLP 不可能，与设计分布式算法的人特别相关。 第二个——CAP定理——是一个与从业者更相关的相关结果；需要在不同系统设计之间进行选择但不直接关注算法设计的人相关。

<!--
## The FLP impossibility result

I will only briefly summarize the [FLP impossibility result](http://en.wikipedia.org/wiki/Consensus_%28computer_science%29#Solvability_results_for_some_agreement_problems), though it is considered to be [more important](http://en.wikipedia.org/wiki/Dijkstra_Prize) in academic circles. The FLP impossibility result (named after the authors, Fischer, Lynch and Patterson) examines the consensus problem under the asynchronous system model (technically, the agreement problem, which is a very weak form of the consensus problem). It is assumed that nodes can only fail by crashing; that the network is reliable, and that the typical timing assumptions of the asynchronous system model hold: e.g. there are no bounds on message delay.
-->

## FLP不可能

我只会简要总结一下[FLP的不可能](http://en.wikipedia.org/wiki/Consensus_%28computer_science%29#Solvability_results_for_some_agreement_problems)，尽管它在学术界被认为[更为重要](http://en.wikipedia.org/wiki/Dijkstra_Prize)。 FLP不可能（以作者，Fischer、Lynch和Patterson名字命名）检查异步系统模型下的共识问题（技术上，协议问题，这是共识问题的一种非常弱的形式）。假设节点只能通过崩溃来失败；网络是可靠的，并且异步系统模型的典型时序假设成立：例如消息延迟没有限制。

<!--
Under these assumptions, the FLP result states that "there does not exist a (deterministic) algorithm for the consensus problem in an asynchronous system subject to failures, even if messages can never be lost, at most one process may fail, and it can only fail by crashing (stopping executing)".

This result means that there is no way to solve the consensus problem under a very minimal system model in a way that cannot be delayed forever.  The argument is that if such an algorithm existed, then one could devise an execution of that algorithm in which it would remain undecided ("bivalent") for an arbitrary amount of time by delaying message delivery - which is allowed in the asynchronous system model. Thus, such an algorithm cannot exist.
-->

在这些假设下，FLP结果表明“在存在故障的异步系统中，对于共识问题不存在（确定性）算法，即使消息永远不会丢失，最多一个进程可能会失败，并且它只能崩溃失败（停止执行）”。

这个结果意味着没有办法在一个非常小的系统模型下以一种不能永远延迟的方式解决共识问题。争论的焦点是，如果存在这样的算法，那么可以通过延迟消息传递（在异步系统模型中允许）来设计该算法的执行，其中它将在任意时间量内保持未定（“二价”）。因此，这种算法不可能存在。

<!--
This impossibility result is important because it highlights that assuming the asynchronous system model leads to a tradeoff: algorithms that solve the consensus problem must either give up safety or liveness when the guarantees regarding bounds on message delivery do not hold.

This insight is particularly relevant to people who design algorithms, because it imposes a hard constraint on the problems that we know are solvable in the asynchronous system model. The CAP theorem is a related theorem that is more relevant to practitioners: it makes slightly different assumptions (network failures rather than node failures), and has more clear implications for practitioners choosing between system designs.
-->

这种不可能性结果很重要，因为它突出了假设异步系统模型导致权衡：解决共识问题的算法必须在有关消息传递边界的保证不成立时放弃安全性或活跃性。

这种洞察力与设计算法的人特别相关，因为它对我们知道在异步系统模型中可解决的问题施加了严格的约束。 CAP定理是一个与从业者更相关的相关定理：它做出略微不同的假设（网络故障而非节点故障），并且对于在系统设计之间进行选择的从业者具有更明确的含义。

<!--
## The CAP theorem

The CAP theorem was initially a conjecture made by computer scientist Eric Brewer. It's a popular and fairly useful way to think about tradeoffs in the guarantees that a system design makes. It even has a [formal proof](http://www.comp.nus.edu.sg/~gilbert/pubs/BrewersConjecture-SigAct.pdf) by [Gilbert](http://www.comp.nus.edu.sg/~gilbert/biblio.html) and [Lynch](http://en.wikipedia.org/wiki/Nancy_Lynch) and no, [Nathan Marz](http://nathanmarz.com/) didn't debunk it, in spite of what [a particular discussion site](http://news.ycombinator.com/) thinks.
-->

## CAP定理

CAP定理最初是由计算机科学家Eric Brewer提出的猜想。在系统设计的保证中考虑权衡是一种流行且相当有用的方法。它甚至还有[Gilbert](http://www.comp.nus.edu.sg/~gilbert/biblio.html) 和 [Lynch](http://en.wikipedia.org/wiki/Nancy_Lynch)的[正式证明](http://www.comp.nus.edu.sg/~gilbert/pubs/BrewersConjecture-SigAct.pdf)，不然，尽管[特定的讨论网站](http://news.ycombinator.com/)认为，[Nathan Marz](http://nathanmarz.com/)并未对其进行揭穿。

<!--
The theorem states that of these three properties:

- Consistency: all nodes see the same data at the same time.
- Availability: node failures do not prevent survivors from continuing to operate.
- Partition tolerance: the system continues to operate despite message loss due to network and/or node failure

only two can be satisfied simultaneously. We can even draw this as a pretty diagram, picking two properties out of three gives us three types of systems that correspond to different intersections:
-->

该定理指出了这三个属性：

- 一致性：所有节点同时看到相同的数据。
- 可用性：节点故障不会阻止幸存者继续运行。
- 分区容错性：尽管由于网络和/或节点故障导致消息丢失，系统仍继续运行

只有两个可以同时满足。我们甚至可以将它绘制成一个漂亮的图表，从三个中选择两个属性为我们提供了三种类型的系统，它们对应于不同的交叉点：

![CAP theorem](images/CAP.png)

<!--
Note that the theorem states that the middle piece (having all three properties) is not achievable. Then we get three different system types:

- CA (consistency + availability). Examples include full strict quorum protocols, such as two-phase commit.
- CP (consistency + partition tolerance). Examples include majority quorum protocols in which minority partitions are unavailable such as Paxos.
- AP (availability + partition tolerance). Examples include protocols using conflict resolution, such as Dynamo.
-->

注意，该定理表明中间件（具有所有三个属性）是不可实现的。然后我们得到三种不同的系统类型：

- CA（一致性+可用性）：例如包括完全严格的仲裁协议，比如两阶段提交。
- CP（一致性+分区容错性）：例如包括多数分区协议，其中少数分区不可用，比如Paxos。
- AP（可用性+分区容错性）：例如包括使用冲突解决的协议，比如Dynamo。

<!--
The CA and CP system designs both offer the same consistency model: strong consistency. The only difference is that a CA system cannot tolerate any node failures; a CP system can tolerate up to `f` faults given `2f+1` nodes in a non-Byzantine failure model (in other words, it can tolerate the failure of a minority `f` of the nodes as long as majority `f+1` stays up). The reason is simple:

- A CA system does not distinguish between node failures and network failures, and hence must stop accepting writes everywhere to avoid introducing divergence (multiple copies). It cannot tell whether a remote node is down, or whether just the network connection is down: so the only safe thing is to stop accepting writes.
- A CP system prevents divergence (e.g. maintains single-copy consistency) by forcing asymmetric behavior on the two sides of the partition. It only keeps the majority partition around, and requires the minority partition to become unavailable (e.g. stop accepting writes), which retains a degree of availability (the majority partition) and still ensures single-copy consistency.
-->

CA 和 CP 系统设计都提供相同的一致性模型：强一致性。唯一的区别是 CA 系统不能容忍任何节点故障；在非拜占庭故障模型中给定`2f + 1`个节点的 CP 系统可以容忍多达 `f` 个节点故障（换句话说，只要多数 `f + 1` 保持不变，它就可以容忍少数 `f` 节点的故障）。原因很简单：

- CA系统不区分节点故障和网络故障，因此必须停止在任何地方接受写入以避免引入分歧（多个副本）。它无法判断远程节点是否已关闭，或者网络连接是否已关闭：因此唯一安全的做法是停止接受写入。
- CP系统通过在分区的两侧强制不对称行为来防止分歧（例如，维持单拷贝一致性）。它只保留多数分区，并要求少数分区变得不可用（例如，停止接受写入），这保留了一定程度的可用性（多数分区）并仍然确保单拷贝一致性。

<!--
I'll discuss this in more detail in the chapter on replication when I discuss Paxos. The important thing is that CP systems incorporate network partitions into their failure model and distinguish between a majority partition and a minority partition using an algorithm like Paxos, Raft or viewstamped replication. CA systems are not partition-aware, and are historically more common: they often use the two-phase commit algorithm and are common in traditional distributed relational databases.

Assuming that a partition occurs, the theorem reduces to a binary choice between availability and consistency.
-->

当我讨论 Paxos 时，我将在关于复制的章节中更详细地讨论这个问题。重要的是 CP 系统将网络分区合并到其故障模型中，并使用 Paxos 、Raft 或 viewstamped replication 等算法区分多数分区和少数分区。CA 系统不支持分区，并且在历史上更常见：它们通常使用两阶段提交算法，并且在传统的分布式关系数据库中很常见。

假设发生了分区，该定理简化为可用性和一致性之间的二元选择。

![Based on http://blog.mikiobraun.de/2013/03/misconceptions-about-cap-theorem.html](images/CAP_choice.png)

<!--
I think there are four conclusions that should be drawn from the CAP theorem:

First, that *many system designs used in early distributed relational database systems did not take into account partition tolerance* (e.g. they were CA designs). Partition tolerance is an important property for modern systems, since network partitions become much more likely if the system is geographically distributed (as many large systems are).

Second, that *there is a tension between strong consistency and high availability during network partitions*. The CAP theorem is an illustration of the tradeoffs that occur between strong guarantees and distributed computation.

In some sense, it is quite crazy to promise that a distributed system consisting of independent nodes connected by an unpredictable network "behaves in a way that is indistinguishable from a non-distributed system".
-->

我认为应该从 CAP 定理中得出四个结论：

首先，*在早期分布式关系数据库系统中使用的许多系统设计没有考虑分区容错性*（也就是说它们是CA设计）。分区容错性是现代系统的一个重要特性，因为如果系统在地理上分布（如同许多大型系统那样），则网络分区变得更加可能。

其次，*在网络分区期间，强一致性和高可用性之间存在紧张关系*。 CAP 定理说明了强一致和分布式计算之间的权衡。

从某种意义上说，承诺由不稳定网络连接的独立节点组成的分布式系统“以一种与非分布式系统无法区分的方式运行”是非常疯狂的。

![From the Simpsons episode Trash of the Titans](images/news_120.jpg)

<!--
Strong consistency guarantees require us to give up availability during a partition. This is because one cannot prevent divergence between two replicas that cannot communicate with each other while continuing to accept writes on both sides of the partition.

How can we work around this? By strengthening the assumptions (assume no partitions) or by weakening the guarantees. Consistency can be traded off against availability (and the related capabilities of offline accessibility and low latency). If "consistency" is defined as something less than "all nodes see the same data at the same time" then we can have both availability and some (weaker) consistency guarantee.

Third, that *there is a tension between strong consistency and performance in normal operation*.

Strong consistency / single-copy consistency requires that nodes communicate and agree on every operation. This results in high latency during normal operation.
-->

强一致性保证要求我们在分区期间放弃可用性。这是因为在继续接受分区两侧的写入的同时，无法防止两个不能彼此通信的副本之间的分歧。

我们如何解决这个问题？通过加强假设（假设没有分区）或弱化担保。可以根据可用性（以及离线可访问性和低延迟的相关功能）来交换一致性。如果“一致性”被定义为小于“所有节点同时看到相同数据”的东西，那么我们可以同时具有可用性和一些（弱）一致性保证。

第三，*在正常操作中强一致性和性能之间存在紧张关系*。

强一致性/单拷贝一致性要求节点每个操作都需要通信并达成一致。这导致正常操作期间的高延迟。

<!--
If you can live with a consistency model other than the classic one, a consistency model that allows replicas to lag or to diverge, then you can reduce latency during normal operation and maintain availability in the presence of partitions.

When fewer messages and fewer nodes are involved, an operation can complete faster. But the only way to accomplish that is to relax the guarantees: let some of the nodes be contacted less frequently, which means that nodes can contain old data.

This also makes it possible for anomalies to occur. You are no longer guaranteed to get the most recent value. Depending on what kinds of guarantees are made, you might read a value that is older than expected, or even lose some updates.
-->

如果你可以使用除经典模型之外的一致性模型，允许副本滞后或发散的一致性模型，那么你可以减少正常操作期间的延迟并在存在分区时保持可用性。

当涉及的消息更少且节点更少时，操作可以更快地完成。但实现这一目标的唯一方法是放宽保证：让一些节点的联系频率降低，这意味着节点可以包含旧数据。

这也使异常发生成为可能。你不再保证获得最新价值。根据所提供的保证类型，你可能会读取比预期更早的值，甚至会丢失一些更新。

<!--
Fourth - and somewhat indirectly - that *if we do not want to give up availability during a network partition, then we need to explore whether consistency models other than strong consistency are workable for our purposes*.

For example, even if user data is georeplicated to multiple datacenters, and the link between those two datacenters is temporarily out of order, in many cases we'll still want to allow the user to use the website / service. This means reconciling two divergent sets of data later on, which is both a technical challenge and a business risk. But often both the technical challenge and the business risk are manageable, and so it is preferable to provide high availability.

Consistency and availability are not really binary choices, unless you limit yourself to strong consistency. But strong consistency is just one consistency model: the one where you, by necessity, need to give up availability in order to prevent more than a single copy of the data from being active. As [Brewer himself points out](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed), the "2 out of 3" interpretation is misleading.

If you take away just one idea from this discussion, let it be this: "consistency" is not a singular, unambiguous property. Remember:
-->

第四 —— 有点间接—— *如果我们不想在网络分区期间放弃可用性，那么我们需要探索除了强一致性之外的一致性模型是否可用于我们的目的*。

例如，即使用户数据被地理复制到多个数据中心，并且这两个数据中心之间的链接暂时出现故障，在许多情况下我们仍然希望允许用户使用网站/服务。 这意味着稍后协调两组不同的数据，这既是技术挑战，也是业务风险。 但是，技术挑战和业务风险通常都是可管理的，因此最好提供高可用性。

一致性和可用性并非真正的二元选择，除非你将自己局限于强一致性。但强一致性只是一种一致性模型：你必须放弃可用性以防止多个数据副本处于活动状态。正如[Brewer 自己](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed)指出的那样，“三分之二”的解释具有误导性。

如果你从这个讨论中只拿走一个想法，就这样说：“一致性”不是一个单一的，明确的属性。记住：

<blockquote>
  <p>
   [ACID](http://en.wikipedia.org/wiki/ACID) consistency != <br>
   [CAP](http://en.wikipedia.org/wiki/CAP_theorem) consistency != <br>
   [Oatmeal](http://en.wikipedia.org/wiki/Oatmeal) consistency
  </p>
</blockquote>

<!--
Instead, a consistency model is a guarantee - any guarantee - that a data store gives to programs that use it.

<dl>
  <dt>Consistency model</dt>
  <dd>a contract between programmer and system, wherein the system guarantees that if the programmer follows some specific rules, the results of operations on the data store will be predictable</dd>
</dl>

The "C" in CAP is "strong consistency", but "consistency" is not a synonym for "strong consistency".

Let's take a look at some alternative consistency models.
-->

相反，一致性模型是数据存储为使用它的程序提供的保证 —— 任何保证。

<dl>
  <dt>一致性模型</dt>
  <dd>程序员和系统之间的契约，其中系统保证如果程序员遵循某些特定规则，数据存储上的操作结果将是可预测的</dd>
</dl>

CAP中的 "C" 是 “强一致性”，但 “一致性” 不是 “强一致性” 的同义词。

我们来看看一些替代的一致性模型。

<!--
## Strong consistency vs. other consistency models

Consistency models can be categorized into two types: strong and weak consistency models:

- Strong consistency models (capable of maintaining a single copy)
  - Linearizable consistency
  - Sequential consistency
- Weak consistency models (not strong)
  - Client-centric consistency models
  - Causal consistency: strongest model available
  - Eventual consistency models
-->

## 强一致模型 VS 其他一致性模型

一致性模型可以分为两类：强一致性模型和弱一致性模型：

- 强一致性模型（能够维护单个副本）
  - 可线性一致性
  - 顺序一致性
- 弱一致性模型（不强）
  - 以客户为中心的一致性模型
  - 因果一致性：最强的模型可用
  - 最终的一致性模型

<!--
Strong consistency models guarantee that the apparent order and visibility of updates is equivalent to a non-replicated system. Weak consistency models, on the other hand, do not make such guarantees.

Note that this is by no means an exhaustive list. Again, consistency models are just arbitrary contracts between the programmer and system, so they can be almost anything.
-->

强一致性模型可确保更新的明显顺序和可见性等同于非复制系统。另一方面，弱一致性模型不能做出这样的保证。

请注意，这绝不是一个详尽的清单。同样，一致性模型只是程序员和系统之间的任意契约，因此它们几乎可以是任何东西。

<!--
### Strong consistency models

Strong consistency models can further be divided into two similar, but slightly different consistency models:

- *Linearizable consistency*: Under linearizable consistency, all operations **appear** to have executed atomically in an order that is consistent with the global real-time ordering of operations. (Herlihy & Wing, 1991)
- *Sequential consistency*: Under sequential consistency, all operations **appear** to have executed atomically in some order that is consistent with the order seen at individual nodes and that is equal at all nodes. (Lamport, 1979)
-->

### 强一致性模型

强一致性模型可以进一步划分为两个相似但略有不同的一致性模型：

- *可线性化一致性*：在线性化一致性下，所有操作**似乎**都按照与全局实时操作顺序一致的顺序原子执行。（Herlihy＆Wing，1991）
- *顺序一致性*：在顺序一致性下，所有操作似乎都以某种顺序原子执行，这与在各个节点上看到的顺序一致，并且在所有节点上都是相等的。（Lamport，1979）

<!--
The key difference is that linearizable consistency requires that the order in which operations take effect is equal to the actual real-time ordering of operations. Sequential consistency allows for operations to be reordered as long as the order observed on each node remains consistent. The only way someone can distinguish between the two is if they can observe all the inputs and timings going into the system; from the perspective of a client interacting with a node, the two are equivalent.

The difference seems immaterial, but it is worth noting that sequential consistency does not compose.

Strong consistency models allow you as a programmer to replace a single server with a cluster of distributed nodes and not run into any problems.
-->

关键的区别在于可线性化的一致性要求操作生效的顺序等于操作的实际实时排序。只要在每个节点上观察到的顺序保持一致，顺序一致性就允许对操作进行重新排序。有人可以区分两者的唯一方法是，他们是否可以观察进入系统的所有输入和时间；从客户端与节点交互的角度来看，两者是等价的。

差异似乎并不重要，但值得注意的是顺序一致性并不构成。

强一致性模型允许你作为程序员用分布式节点集群替换单个服务器，而不会遇到任何问题。

<!--
All the other consistency models have anomalies (compared to a system that guarantees strong consistency), because they behave in a way that is distinguishable from a non-replicated system. But often these anomalies are acceptable, either because we don't care about occasional issues or because we've written code that deals with inconsistencies after they have occurred in some way.

Note that there really aren't any universal typologies for weak consistency models, because "not a strong consistency model" (e.g. "is distinguishable from a non-replicated system in some way") can be almost anything.
-->

所有其他一致性模型都有异常（与保证强一致性的系统相比），因为它们的行为方式可以与非复制系统区分开来。但通常这些异常是可以接受的，要么是因为我们不关心偶然的问题，要么是因为我们编写的代码在以某种方式发生后处理不一致。

请注意，弱一致性模型确实没有任何通用类型，因为“不是一个强一致性模型”（例如“在某种程度上可以与非复制系统区分开”）几乎可以是任何东西。

<!--
### Client-centric consistency models

*Client-centric consistency models* are consistency models that involve the notion of a client or session in some way. For example, a client-centric consistency model might guarantee that a client will never see older versions of a data item. This is often implemented by building additional caching into the client library, so that if a client moves to a replica node that contains old data, then the client library returns its cached value rather than the old value from the replica.

Clients may still see older versions of the data, if the replica node they are on does not contain the latest version, but they will never see anomalies where an older version of a value resurfaces (e.g. because they connected to a different replica). Note that there are many kinds of consistency models that are client-centric.
-->

### 以客户为中心的一致性模型

以客户为中心的一致性模型是一致性模型，它以某种方式涉及客户端或会话的概念。例如，以客户端为中心的一致性模型可能保证客户端永远不会看到数据项的旧版本。这通常通过在客户端库中构建其他缓存来实现，这样，如果客户端移动到包含旧数据的副本节点，则客户端库将返回其缓存值，而不是复制副本中的旧值。

客户端仍然可以看到旧版本的数据，如果它们所在的副本节点不包含最新版本，但是它们永远不会看到旧版本的值重新出现的异常（例如，因为它们连接到不同的副本）。请注意，有许多种以客户为中心的一致性模型。

<!--
### Eventual consistency

The *eventual consistency* model says that if you stop changing values, then after some undefined amount of time all replicas will agree on the same value. It is implied that before that time results between replicas are inconsistent in some undefined manner. Since it is [trivially satisfiable](http://www.bailis.org/blog/safety-and-liveness-eventual-consistency-is-not-safe/) (liveness property only), it is useless without supplemental information.

Saying something is merely eventually consistent is like saying "people are eventually dead". It's a very weak constraint, and we'd probably want to have at least some more specific characterization of two things:
-->

### 最终的一致性

最终的一致性模型表明，如果你停止更改值，则在一些未定义的时间后，所有副本将同意相同的值。暗示在此之前复制品之间的结果在某种不确定的方式上是不一致的。由于它是微不足道的（只有活力属性），没有补充信息就没用了。

说一些事情只是最终是一致的就像说“人们最终死了”。这是一个非常弱的约束，我们可能想要至少对两件事进行更具体的表征：

<!--
First, how long is "eventually"? It would be useful to have a strict lower bound, or at least some idea of how long it typically takes for the system to converge to the same value.

Second, how do the replicas agree on a value? A system that always returns "42" is eventually consistent: all replicas agree on the same value. It just doesn't converge to a useful value since it just keeps returning the same fixed value. Instead, we'd like to have a better idea of the method. For example, one way to decide is to have the value with the largest timestamp always win.

So when vendors say "eventual consistency", what they mean is some more precise term, such as "eventually last-writer-wins, and read-the-latest-observed-value in the meantime" consistency. The "how?" matters, because a bad method can lead to writes being lost - for example, if the clock on one node is set incorrectly and timestamps are used.

I will look into these two questions in more detail in the chapter on replication methods for weak consistency models.
-->

首先，“最终”有多长？具有严格的下限，或者至少对于系统收敛到相同值通常需要多长时间的一些想法是有用的。

其次，复制品如何就价值达成一致？始终返回 "42" 的系统最终是一致的：所有副本都同意相同的值。它只是没有收敛到有用的值，因为它只是保持返回相同的固定值。相反，我们希望更好地了解该方法。例如，一种决定方法是使具有最大时间戳的值始终获胜。

因此，当供应商说“最终一致性”时，他们的意思是一些更精确的术语，例如“最终最后作者获胜，同时读取最新观察值”的一致性。“怎么样？”很重要，因为糟糕的方法会导致写入丢失——例如，如果一个节点上的时钟设置不正确并且使用了时间戳。

我将在弱一致性模型的复制方法一章中更详细地研究这两个问题。

---

## Further reading

- [Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services](http://www.comp.nus.edu.sg/~gilbert/pubs/BrewersConjecture-SigAct.pdf) - Gilbert & Lynch, 2002
- [Impossibility of distributed consensus with one faulty process](http://scholar.google.com/scholar?q=Impossibility+of+distributed+consensus+with+one+faulty+process) - Fischer, Lynch and Patterson, 1985
- [Perspectives on the CAP Theorem](http://scholar.google.com/scholar?q=Perspectives+on+the+CAP+Theorem) - Gilbert & Lynch, 2012
- [CAP Twelve Years Later: How the "Rules" Have Changed](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed) - Brewer, 2012
- [Uniform consensus is harder than consensus](http://scholar.google.com/scholar?q=Uniform+consensus+is+harder+than+consensus) - Charron-Bost & Schiper, 2000
- [Replicated Data Consistency Explained Through Baseball](http://pages.cs.wisc.edu/~remzi/Classes/739/Papers/Bart/ConsistencyAndBaseballReport.pdf) - Terry, 2011
- [Life Beyond Distributed Transactions: an Apostate's Opinion](http://scholar.google.com/scholar?q=Life+Beyond+Distributed+Transactions%3A+an+Apostate%27s+Opinion) - Helland, 2007
- [If you have too much data, then 'good enough' is good enough](http://dl.acm.org/citation.cfm?id=1953140) - Helland, 2011
- [Building on Quicksand](http://scholar.google.com/scholar?q=Building+on+Quicksand) - Helland & Campbell, 2009
