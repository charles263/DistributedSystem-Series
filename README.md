[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![license: CC BY-NC-SA 4.0](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-lightgrey.svg)][license-url]

<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/wx-chevalier/DistributedSystem-Series">
    <img src="header.svg" alt="Logo" style="width: 100vw;height: 400px" />
  </a>

  <p align="center">
    <a href="https://ng-tech.icu/DistributedSystem-Series"><strong>在线阅读 >> </strong></a>
    <br />
    <br />
    <a href="https://github.com/wx-chevalier/Awesome-CheatSheets">速览手册</a>
    ·
    <a href="https://github.com/wx-chevalier">代码实践</a>
    ·
    <a href="https://github.com/wx-chevalier/Awesome-Lists">参考资料</a>

  </p>
</p>

# Distributed System Series（分布式系统·实践笔记）

深入浅出分布式基础架构是笔者归档自己，在学习与实践软件分布式架构过程中的，笔记与代码的仓库；主要包含分布式计算、分布式系统、数据存储、虚拟化、网络、操作系统等几个部分。所谓的分布式系统，其主要由网络、分布式存储与分布式计算等部分构成，分布式存储侧重于数据的读写存取及一致性等方面，而分布式计算则侧重于资源、任务的编排调度。

![A Unified Data Infrastructure Architecture](https://s1.ax1x.com/2020/10/18/0XOno9.png)

# Introduction | 前言

![分布式系统题图](https://i.postimg.cc/2SVpd63d/image.png)

## 无处不在的分布式系统

过去数十年间，信息技术的浪潮深刻地改变了这个社会的通信、交流与协作模式，我们熟知的互联网也经历了基于流量点击赢利的单方面信息发布的 Web 1.0 业务模式，转变为由用户主导而生成内容的 Web 2.0 业务模式；在可见的将来随着 3D 相关技术的落地，互联网应用系统所需处理的访问量和数据量必然会再次爆发性增长。正如凯文．凯利 2016 年在《失控》一书中指出，分布式系统具有四个突出特点，即没有强制性的中心控制、次级单位具有自治的特质、次级单位之间彼此高度链接、点对点之间的影响通过网络形成了非线性因果关系。凯文．凯利进一步指出，与其说一个分布式、去中心化的网络是一个物体，还不如说它是一个过程。

## 不可靠的分布式系统

在 [《网络与时钟》](https://github.com/wx-chevalier/DistributedSystem-Series)中我们讨论了为何分布式系统是不可靠的；在[《高可用架构》](https://github.com/wx-chevalier/HA-Series)中我们会深入地讨论高可用分布式系统的特性，以及如何去保障系统的高可用。

## 一致性、共识与分布式事务

分布式系统的不可靠性是其内在属性，为了应对这种不可靠性，我们必然会进入到一致性、共识及分布式事务的领域。在分布式系统与数据库等技术领域中，一致性都会频繁地出现，但是在不同的语境和上下文中，它其实代表着不同的东西：

- 在事务的上下文中，比如 ACID 里的 C，指的就是通常的一致性（Consistency），即对数据的一组特定陈述必须始终成立。即不变量（invariants）。具体到分布式事务的上下文中这个不变量是：所有参与事务的节点状态保持一致：要么全部成功提交，要么全部失败回滚，不会出现一些节点成功一些节点失败的情况。

- 在分布式系统的上下文中，例如 CAP 里的 C，实际指的是线性一致性（Linearizability），即多副本的系统能够对外表现地像只有单个副本一样（系统保证从任何副本读取到的值都是最新的），且所有操作都以原子的方式生效（一旦某个新值被任一客户端读取到，后续任意读取不会再返回旧值）。

- “一致性哈希”，“最终一致性”这些名词里的“一致性”也有不同的涵义。

在分布式系统中，我们常说的一致性模型有线性一致性、因果一致性、最终一致性等。线性一致性能使多副本数据看起来好像只有一个副本一样，并使其上所有操作都原子性地生效，它使数据库表现的好像单线程程序中的一个变量一样；但它有着速度缓慢的缺点，特别是在网络延迟很大的环境中。因果性对系统中的事件施加了顺序（什么发生在什么之前，基于因与果）。与线性一致不同，线性一致性将所有操作放在单一的全序时间线中，因果一致性为我们提供了一个较弱的一致性模型：某些事件可以是并发的，所以版本历史就像是一条不断分叉与合并的时间线。因果一致性没有线性一致性的协调开销，而且对网络问题的敏感性要低得多。

但即使捕获到因果顺序（例如使用兰伯特时间戳），我们发现有些事情也不能通过这种方式实现：我们需要确保用户名是唯一的，并拒绝同一用户名的其他并发注册；如果一个节点要通过注册，则需要知道其他的节点没有在并发抢注同一用户名的过程中。这个问题引领我们走向共识。

达成共识意味着以这样一种方式决定某件事：所有节点一致同意所做决定，且这一决定不可撤销。共识问题通常形式化如下：一个或多个节点可以提议（propose）某些值，而共识算法决定采用其中的某个值。在保证分布式事务一致性的场景中，每个节点可以投票提议，并对谁是新的协调者达成共识。譬如 Raft 算法解决了全序广播问题，维护多副本日志间的一致性，其实就是让所有节点对同全局操作顺序达成一致，也其实就是让日志系统具有线性一致性。因而解决了共识问题。

通过深入挖掘，结果我们发现很广泛的一系列问题实际上都可以归结为共识问题，并且彼此等价（从这个意义上来讲，如果你有其中之一的解决方案，就可以轻易将它转换为其他问题的解决方案）。这些等价的问题包括：

- 线性一致性的 CAS 寄存器：寄存器需要基于当前值是否等于操作给出的参数，原子地决定是否设置新值。
- 原子事务提交：数据库必须决定是否提交或中止分布式事务。
- 全序广播：即保证消息不丢失，且消息以相同的顺序传递给每个节点。
- 锁和租约：当几个客户端争抢锁或租约时，由锁来决定哪个客户端成功获得锁。
- 成员/协调服务：给定某种故障检测器（例如超时），系统必须决定哪些节点活着，哪些节点因为会话超时需要被宣告死亡。
- 唯一性约束：当多个事务同时尝试使用相同的键创建冲突记录时，约束必须决定哪一个被允许，哪些因为违反约束而失败。

在分布式系统中，我们常常同时讨论分布式事务与共识，这是因为分布式事务本身的一致性是通过协调者内部的原子操作与多阶段提交协议保证的，不需要共识；但解决分布式事务一致性带来的可用性问题需要用到共识。为了保证分布式事务的一致性，分布式事务通常需要一个协调者（Coordinator）/事务管理器（Transaction Manager）来决定事务的最终提交状态。但无论 2PC 还是 3PC，都无法应对协调者失效的问题，而且具有扩大故障的趋势。这就牺牲了可靠性、可维护性与可扩展性。为了让分布式事务真正可用，就需要在协调者挂点的时候能赶快选举出一个新的协调者来解决分歧，这就需要所有节点对谁是领导者达成共识（Consensus）。

在实际的系统演化过程中，最初的时候我们只有单节点，或者我们能够人为地去控制仅有的几个节点。但如果该领导者失效，或者如果网络中断导致领导者不可达，这样的系统就无法取得任何进展。应对这种情况可以有三种方法：

- 等待领导者恢复，接受系统将在这段时间阻塞的事实。许多 XA/JTA 事务协调者选择这个选项。这种方法并不能完全达成共识，因为它不能满足终止属性的要求：如果领导者续命失败，系统可能会永久阻塞。
- 人工故障切换，让人类选择一个新的领导者节点，并重新配置系统使之生效，许多关系型数据库都采用这种方方式。这是一种来自“天意”的共识，由计算机系统之外的运维人员做出决定。故障切换的速度受到人类行动速度的限制，通常要比计算机慢（得多）。
- 使用算法自动选择一个新的领导者。这种方法需要一种共识算法，使用成熟的算法来正确处理恶劣的网络条件是明智之举。

尽管单领导者数据库可以提供线性一致性，且无需对每个写操作都执行共识算法，但共识对于保持及变更领导权仍然是必须的。因此从某种意义上说，使用单个领导者不过是“缓兵之计”：共识仍然是需要的，只是在另一个地方，而且没那么频繁。像 ZooKeeper 这样的工具为应用提供了“外包”的共识、故障检测和成员服务。它们扮演了重要的角色，虽说使用不易，但总比自己去开发一个能经受所有问题考验的算法要好得多。如果你发现自己想要解决的问题可以归结为共识，并且希望它能容错，使用一个类似 ZooKeeper 的东西是明智之举。

# Nav | 导读

- 如果你想了解数据库相关，可以参阅 [Database-Series](https://github.com/wx-chevalier/Database-Series)。

- 如果你想了解虚拟化与云计算相关，可以参阅 [Cloud-Series](https://github.com/wx-chevalier/Cloud-Series)。

- 如果你想了解 Linux 与操作系统相关，可以参阅 [Linux-Series](https://github.com/wx-chevalier/Linux-Series)。

# About

## Copyright & More | 延伸阅读

笔者所有文章遵循 [知识共享 署名-非商业性使用-禁止演绎 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)，欢迎转载，尊重版权。您还可以前往 [NGTE Books](https://ng-tech.icu/books/) 主页浏览包含知识体系、编程语言、软件工程、模式与架构、Web 与大前端、服务端开发实践与工程架构、分布式基础架构、人工智能与深度学习、产品运营与创业等多类目的书籍列表：

[![NGTE Books](https://s2.ax1x.com/2020/01/18/19uXtI.png)](https://ng-tech.icu/books/)

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->

[contributors-shield]: https://img.shields.io/github/contributors/wx-chevalier/DistributedSystem-Series.svg?style=flat-square
[contributors-url]: https://github.com/wx-chevalier/DistributedSystem-Series/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/wx-chevalier/DistributedSystem-Series.svg?style=flat-square
[forks-url]: https://github.com/wx-chevalier/DistributedSystem-Series/network/members
[stars-shield]: https://img.shields.io/github/stars/wx-chevalier/DistributedSystem-Series.svg?style=flat-square
[stars-url]: https://github.com/wx-chevalier/DistributedSystem-Series/stargazers
[issues-shield]: https://img.shields.io/github/issues/wx-chevalier/DistributedSystem-Series.svg?style=flat-square
[issues-url]: https://github.com/wx-chevalier/DistributedSystem-Series/issues
[license-shield]: https://img.shields.io/github/license/wx-chevalier/DistributedSystem-Series.svg?style=flat-square
[license-url]: https://github.com/wx-chevalier/DistributedSystem-Series/blob/master/LICENSE.txt
