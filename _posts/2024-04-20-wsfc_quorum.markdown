---
layout: default
title:  "【WSFC 学习笔记】如何理解 Windows Server 故障转移群集 的 Quorum"
tags: wsfc windows server zh-cn
---

# 前言

最近一个月一直在学高可用的东西，其中 WSFC 是其中一个重要的组成部分。在学习 WSFC 的过程中，我发现了一个很重要的概念：Quorum，这个概念网上一找全是照着微软官方文档写的 [仲裁配置选项](https://learn.microsoft.com/en-us/windows-server/failover-clustering/manage-cluster-quorum#quorum-configuration-options)。问有什么模型的时候头头是道，什么节点多数无见证啊、仅磁盘见证啊什么的，一问啥是仲裁（Quorum）和见证（Witness）的时候傻眼了。所以我决定写一篇文章来总结一下我对 Quorum 的理解。

# 仲裁（Quorum）是什么

仲裁（Quorum）是 WSFC 中的一个重要概念，它是用来保证群集中的节点之间能够达成一致的一个机制。在 WSFC 中，仲裁是通过仲裁资源来实现的，这个资源可以是磁盘、文件共享、或者是其他的仲裁资源。在 WSFC 中，仲裁资源的作用是用来保证群集中的节点之间能够达成一致，从而保证群集的正常运行。

依我看，`Quorum` 其实应该翻译成 `有效参与仲裁的设备（数）`，因为 Quorum 本来就翻译成 [法定人数/出席会议最小人数](https://dictionary.cambridge.org/dictionary/english/quorum)，不应该直接叫 仲裁。

现在涉及到 `仲裁` 这个法律概念，所以我们可以引入法律属于去解释。在法律中，`仲裁` 是指一种解决纠纷的方式，当纠纷发生时，双方可以通过仲裁的方式来解决纠纷。在法律仲裁中，仲裁员是一个独立的第三方，他们会根据法律和事实来做出裁决。但在 WSFC 中，仲裁员（节点/服务器/虚拟机）既是当事人（所有节点），又参与仲裁庭，它们会根据法律和事实（仲裁模型）来做出裁决，是用来保证群集中的节点之间能够达成一致的。

节点多数，无见证：由节点组织“合议制仲裁庭”，当事人（所有参与群集的节点/服务器）约定由奇数名仲裁员（节点）组成仲裁庭（偶数节点就使其中一个节点下线），其中一名为首席仲裁员（主节点/服务器）；如果是双节点就采取“独任制仲裁庭”，只推举一名仲裁员（一个节点做主服务器，另外一个下线）。

磁盘、文件共享见证：见证人是证人之外知道案件情况的当事人（所以磁盘、文件共享见证需要对所有节点可见，在注册表中节点文件是Cluster，见证文件是0.Cluster，可以证明见证确实是“当事人”）。见证人不参与仲裁庭，但是可以提供证据（见证文件）。

总结来说：

1. 节点多数，无见证：
    - 合议庭：集群的所有活动节点。
    - 仲裁庭成员：每个节点都可以投票决定集群状态，若节点总数为偶数，可能需要让一个节点下线以避免平票。
    - 首席仲裁员（主节点）：在实际的WSFC中，通常没有固定的“首席仲裁员”或主节点，所有节点理论上是平等的，但在实践中，某些节点可能因为资源位置或网络优势暂时承担更多责任。

2. 双节点集群的“独任制”：
    - 这种情况下，通常需要额外的见证（磁盘或文件共享见证），因为单纯的两个节点在一个节点失效时无法决定集群状态。如果不使用见证，确实可能会推举一个节点为主导，另一个则在主节点活跃时处于待命状态。

3. 磁盘、文件共享见证：
    - 见证人：在这种情况下，见证（文件共享或磁盘）充当了“知情人”，它存储关于集群配置的关键信息，确保在节点间的意见不一致时提供“证据”来帮助做出决策。见证的存在特别在节点数为偶数时非常关键，以避免平票问题。

但是大多数人对法律仲裁制度不了解，而且法律仲裁制度和 WSFC 中的仲裁机制有很大的不同，所以我觉得这个比喻不太合适。

所以我决定引入日常生活，比如双节点集群就像情侣之间决定晚餐吃什么，引入磁盘见证相当于三口之家中孩子的存在，这样比喻起来更加贴近生活，也更容易理解。

原文如下：

| Mode | Description |
| ---- | ----------- |
| Node majority (no witness) <br> <b>节点多数</b> | Only nodes have votes. No quorum witness is configured. The cluster quorum is the majority of voting nodes in the active cluster membership. |
| Node majority with witness (disk or file share) <br> <b>节点多数+见证</b> | Nodes have votes. In addition, a quorum witness has a vote. The cluster quorum is the majority of voting nodes in the active cluster membership plus a witness vote. A quorum witness can be a designated disk witness or a designated file share witness. |
| No majority (disk witness only) <br> <b>仅磁盘见证</b> | No nodes have votes. Only a disk witness has a vote. <br> The cluster quorum is determined by the state of the disk witness. Generally, this mode is not recommended, and it should not be selected because it creates a single point of failure for the cluster. |

我们可以这样理解：

| 仲裁模型 | 描述 |
| ---- | ----------- |
| 男生女生二人决定晚餐 <br> <b>节点多数</b> | 【双节点】男生让渡（即其中一节点下线，有一票投票权但动态见证干预其不投票），让女生（主节点/服务器/虚拟机 Owner Node）来选择点什么外卖或者出去吃什么餐馆。<s>你就负责买单就行。</s> <br> 【三节点及以上】女生带闺蜜来了，女生和闺蜜手拉手胳膊挽胳膊掌握晚餐选择主动权（即具有集群的控制权，在一个三节点集群中，通常需要至少两个节点在线并相互通信，以维持集群的正常操作）。象征性问你一下想吃什么。<s>所以你还是负责买单的小丑。</s> |
| 核心家庭（三口之家）或扩展家庭（爸妈或者和爷爷奶奶外公外婆一起为节点，孩子为见证）<br> <b>节点多数+见证</b> | <s>爸妈想吃啥就做啥，你的意见仅供参考</s> <br> 因为爷爷奶奶外公外婆说孩子想吃这个那个，所以爸妈做饭就做了大家都爱吃的东西。（换句话说见证也参与，而不仅仅是节点多数） |
| 你过生日那天 <br> <b>仅磁盘见证</b> | <s>“妈，我生日想吃开封菜。” “不行，油炸食品不健康。”</s> （别看上面的见证是你，但是这一条就不是你了） |

<img src="http://icing.fun/img/post/2024/04/20/1.png">

<img src="http://icing.fun/img/post/2024/04/20/2.png">

# 总结

给我写乐了。宿舍停水了，舍友用我的博客洗完了澡。
