---
title: 阶段总结7-区块链与D2D内容缓存/计算卸载
date: 2020-04-03
lastmod: 2020-04-15
tags: [科研记录]
math : true
lightgallery: true
categories: [研究生的区块链学习之路]
slug: blockchain for D2D cache or computing offload
typora-root-url: ..\..\..\static
---

## 1. 概念引入

论文[^mach_mobile_2017]提到，一些邻近的用户设备（User Equipment, UE）组合其计算能力，在本地服务于具有更高性能需求的应用可以叫做 ad-hoc cloud。实际应用需要解决几个关键的问题：

1. 在保证处理后的数据能够返回到源 UE 的同时，在附近找到合适的计算 UE
2. 尽管没有用于促进可靠计算的控制信道，但必须启用计算 UE 间的协调
3. 考虑到电池消耗和额外的数据传输约束，必须激励计算 UE 向其它设备提供其计算能力
4. 安全和隐私问题

[^mach_mobile_2017]: MACH P, BECVAR Z. Mobile Edge Computing: A Survey on Architecture and Computation Offloading[J]. IEEE Communications Surveys & Tutorials, 2017, 19(3): 1628–1656. DOI:[10.1109/COMST.2017.2682318](https://doi.org/10.1109/COMST.2017.2682318).

## 2. 背景调查

如第一部分所述，我们主要谈论的问题被称作 ad-hoc cloud，在论文[^yaqoob_mobile_2016]中，将该方向的研究正式命名为移动自组织云（Mobile ad hoc cloud，MAC），这篇综述对2016年及之前的该方向论文进行了总结，我们总结一些可能有用的内容。

[^yaqoob_mobile_2016]:YAQOOB I, AHMED E, GANI A, 等. Mobile Ad Hoc Cloud: A Survey: Mobile Ad Hoc Cloud[J]. Wireless Communications and Mobile Computing, 2016, 16(16): 2572–2589. DOI:[10.1002/wcm.2709](https://doi.org/10.1002/wcm.2709).

本地的计算密集型应用（如游戏和高清视频）需要大量的服务与资源，利用云提供这些资源的方式叫做移动云计算（Mobile Cloud Computing， MCC）。但是，考虑到网络连接的带宽和质量，这种方式不总是可行的，这种情况下，人们提出了 MAC，利用一组移动设备的共享资源，来实现一个共同的目标。MAC 不是用来取代原有技术的，而是一种补充，当网络质量不佳、附近基站不可用或其它情况时，使用这种方式。

以实时会议（conference）出席的例子来说明：移动用户拍摄会议参与者的快照，并将捕获的人脸和存储的照片进行匹配，从而确定谁在参加哪个会话（session）；此外，出于会议议程的规划，会议组织者希望获取不同 session 中参与者的最新情况。在该场景中，移动用户可能没有足够的资源在单个设备上执行该任务，因此选择使用 MAC 满足任务执行的实时性需求。假设每个 session 有50个参与者，其中10个愿意参与 MAC，任务被分解为如下步骤

1. 检测快照中的人脸数量，并为每个人脸裁剪一个小图像
2. 分别提交每个人脸进行识别
3. 将与会者列表返回给用户

假设第1步需要15秒，第2步需要10秒，第3步需要6秒，不使用 MAC 移动用户总共需要 15+10*50+6 = 512s，如果使用 MAC，任务2被分配给 10 个 MAC 参与者（provider），时间将大幅减少，假设与所有 provider 的通信开销是3s，那么总时间为 15+3 +10 * 5+6 = 74s。

实际的过程更加复杂，面对的问题包括：[构成 MAC 的移动设备的发现]^(MAC formation)、[任务卸载]^(task offloading)，任务调度与分配，安全与隐私，[移动性和激励]^(mobility and incentives)，资源管理。按这种分类总结相关解决方案如下图

![分类解决方案](/images/总结-区块链与D2D结合背景调查/分类解决方案.png "分类解决方案")

由于通常参与 MAC 的移动节点是随机选择的，因此安全性和隐私性是一个重要问题。在 MAC 中加入任何恶意节点都会增加总体任务执行时间，从而导致性能下降。此外，位置的隐私性也是共享设备的用户的一个重要的问题。为了解决这些问题，需要合适的认证与授权机制，保护位置隐私及防止恶意节点参与 MAC。论文[^lacuesta_spontaneous_2014]提出一种可信算法，基于AES 加密保证节点间的安全通信，同时也可以管理节点的加入和退出。论文[^hammam_trust_2013]则提出一种信任管理系统，监视参与 ad hoc 网络的节点，计算其信任值并存储，从而其它移动设备可验证其信任值，信任值越高越可靠，最终达到阻止恶意节点参与的目的。除此之外，有的方案建立和分发密钥来保护通信，有的方案利用 SaaS 或 PaaS 检测异常行为。

[^lacuesta_spontaneous_2014]:Lacuesta R, Lloret J, Sendra S, Peñalver L. Spontaneous ad hoc mobile cloud computing network. The Scientific World Journal 2014; 2014: 1–19. DOI: 10.1155/2014/232419
[^hammam_trust_2013]:Hammam A, Senbel S. A trust management system for ad-hoc mobile clouds. In Computer Engineering & Systems (ICCES), 2013 8th International Conference on, IEEE, 2013

## 3. 引入区块链的例子

这一部分取自论文[^qiao2019blockchain]，是一个利用区块链提供不可信 D2D 节点间信任的例子。

[^qiao2019blockchain]:Qiao G, Leng S, Chai H, et. al. Blockchain Empowered Resource Trading in Mobile Edge Computing and Networks[C]. ICC 2019 - 2019 IEEE International Conference on Communications (ICC). Shanghai, China: IEEE, 2019: 1–6. DOI:[10.1109/ICC.2019.8761664](https://doi.org/10.1109/ICC.2019.8761664).

首先，随着智能手机和 IoT 设备的增多，移动通信已成为日常生活的一部分，传统的云计算不能满足实时性的需要，D2D-ECN(device-to-device edge computing and network)被提出，目的是共享移动设备的计算和通信资源提供一个灵活的边缘计算平台。这种情况下假设希望在一个期望的时间内执行一个计算密集型的任务，可以将计算任务卸载到有可用计算资源的移动设备上执行，数据传输延迟也可以通过购买空闲通信资源减少

![本地的资源交换和任务分配](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8753818/8761046/8761664/qiao1-p6-qiao-small.gif "资源交换和任务分配")

区块链被引入用来解决分布式和点对点网络中不可信用户间的安全和可靠性问题。由于现有研究大部分使用 PoW 共识，在资源有限的 IoT 环境中不具备可行性，作者选择了一种基于信誉的共识（reputation-based consensus）[^gai2018proof]来协调设备和验证交易。整个系统的安全性靠该共识和一个信誉评估系统保证。

[^gai2018proof]:F. Gai et al., "Proof of Reputation: A Reputation-based Consensus Protocol for Peer-to-Peer Networks", *Proc. 2018 Int. Con. Dat. Sys. Adv. App. DASFAA*, Feb. 2018. ↩

作者认为现有研究没有考虑计算卸载规则对基于区块链的边缘网络的性能的影响，整个计算卸载过程可以划分为资源交易和任务分配两部分，资源交易的目的是提供高效的资源定价和分配策略，从而平衡任务所有者的[效用]^(utility)和资源所有者的收益，任务分配则是在保证较低的执行时间和系统开销。论文实际做的事有两件

1. 提出并使用智能合约实现了一个分布式的资源交易和任务分配方案，平衡资源定价与收益，最小化执行时间和系统开销；
2. 提出一个信誉评估系统并和共识配合使用保证计算卸载环境的可靠性

论文引入资源币激励用户参与，一个计算卸载过程的工作流如下图所示。设备首先在LBS中注册并被授予身份，通过{身份ID，证书，公钥，私钥，钱包地址}五元组追踪在区块链中的交易记录（资源交易和信誉度更新）；然后，任务所有者发送资源需求到LBS，资源所有者发送可用资源数量和使用限制到LBS，LBS验证证书后将这些内容写入智能合约，并在达到条件后自动触发，资源的定价和交易被设定了规则；之后，根据可用的通信和计算资源，智能合约执行任务分配算法最小化总延迟，并在准确返回计算结果后对资源提供者的信誉进行评分；这些工作完成后，任务提供者按照资源价格支付资源币；最后，任务所有者签名后广播资源交易记录和更新后的信誉度，运行 基于信誉的共识，由信誉度最高的节点将这些记录收集到区块并附加到区块链。实际过程需要经过多轮验证

![基于区块链的资源交易和任务分配系统组成和工作流](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8753818/8761046/8761664/qiao2-p6-qiao-small.gif "工作流")

这里有3个关注的地方

1. 作者使用公式推导得到了任务分配者效用和资源所有者收益的平衡函数，在**资源交易智能合约**中执行该函数得到合适的定价；

2. 作者使用公式推导最小化了任务执行的总延迟，并在**任务分配智能合约**中执行；

3. 对资源提供者的信誉度评估在得到正确的计算结果后执行，作者也提供了公式推导

整个系统的安全性通过三个方面保证：共识和信誉评估系统保证计算卸载环境可靠；智能合约保证交易记录和信誉度不被篡改；LBS保证通信安全。

## 4. 一些思路

使用一个公链提供基于位置的服务，所有设备在区块链中注册获取其身份，可以更新自己的资源状态或发起资源请求，由于计算卸载是小范围的，设备还需要提供自身的位置，但位置信息应被加密保护，从而保证隐私。

此外区块链中还维持一个信誉评估系统，根据每次资源交易的结果更新设备信誉度。

访问控制系统在其中起到的作用是，接受到资源请求后，根据位置属性和信誉度获取附近设备的地址，设备地址应该作为一种资源提供。得到附近的设备地址后，发起请求建立资源请求者者和资源提供者的 D2D 通信，该D2D网络同时也是一个区块链，任务分配、计算卸载等过程都在这个本地区块链执行。资源请求者得到结果并验证正确后，将这个临时的区块链链接到公链相应的区块，并在主链中更新信誉度和支付数字货币。

该结构类似于哈希表，主链是不断延长的哈希表结构，每个区块是哈希表的一个元素，而每个哈希表元素对应一条或多条临时区块链。

核心思想是将支付、信誉评估、任务分配、计算卸载等功能分离，放在各自适合的区块链中。

### 4.1 功能划分

我们的基本思路是根据需求将各种功能划分到公链和私链上。基本的出发点是，公链的处理速度不够快，计算卸载和内容缓存都对速度有一定要求，而私链覆盖范围不够大，设备可能在任何时间任何地点有内容缓存或计算卸载的需求。因此最终决定将对处理速度有要求的功能放到私链，将设备发现和移动性等功能放到公链，然后通过一定的方法将两条区块链相关联。

#### 公链

首先基于位置的服务（LBS）需要在公链上实现。论文[^qiao2019blockchain]使用了一个可信 LBS，设备只有在其中注册后才能成为合法的 D2D-ECN （D2D边缘计算网络）参与者，然后 LBS 会分配设备ID 和证书（也可以是 IP 或 MAC 地址）用来唯一识别设备，接着发给设备一对公私钥用于通信，最后是钱包地址，用于资源交易（购买或出售存储或计算能力）。资源请求者将资源需求发到 LBS，资源所有者将可用资源数量和使用限制发到 LBS，LBS 验证它们的证书后将这些内容写入智能合约。因此在该论文中，LBS是独立于区块链之外的可信实体，我们知道，第三方的可信是相对的，尤其在关于位置的隐私上，我们更抱有戒心，因此可以直接利用智能合约在公链上提供基于位置的服务。

为了激励用户参与计算卸载和内容缓存，使用资源提供者的资源需要支付数字货币，我们可以直接使用公链的货币，如以太坊，因此如第一部分所述，资源请求者还需要知道资源提供者的钱包地址。

经过分析，我们应在公链上实施的功能包括：设备注册，位置管理，支付。支付可以直接使用公链的账户地址，注册需要维持一个注册表，以设备 ID 为主键，关联设备地址、IP或MAC地址以及设备的公链账户地址。鉴于可能发生的恶意行为，我们可以维持一个简单的恶意行为判决机制或一个信誉系统。这里的关键问题是设备位置的隐私，由于公链中信息是透明的，位置不可以直接进行存储，最好是对其进行加密，所以可能还需要在设备注册时传入一个公钥用于对设备地址加密。资源请求者发起请求后，公链中的 LBS 根据请求者的位置查找其附近的设备，并将这些设备的 IP/MAC 传给请求者，用于建立 D2D 网络。资源匹配、任务划分、计算卸载等其它功能都在D2D网络中进行。

这里不确定的是资源匹配的过程是否在公链中实施，也就是说，是直接将请求者附近所有设备的位置发送过去，还是进行资源匹配后将符合要求的设备位置发送过去。

#### 私链

资源请求者收到附近设备的 IP/MAC 地址后，发起广播请求建立 D2D 网络，然后在 D2D 网络之上建立私有区块链，利用私有区块链提供的智能合约实现任务划分和计算卸载/内容缓存。任务完成后，将整条私链（或私链的哈希）交给公链存储，依附在公链当前的区块上，形成一种类似于如下的结构。

![两条区块链关联](/images/总结-区块链与D2D/两条区块链关联.png)

如同现有的线上支付系统，资源提供者完成任务后在公链中标记任务完成，资源请求者确认后，事先锁定在智能合约中的数字货币被支付。整个交易完成，最后可能会根据交易情况评估信誉或记录恶意行为。

### 4.2 多链关联

就是上面说的将临时区块链的数据链接到公链相关的区块，这一思路的来源是 Dorri 的论文[^dorri2017blockchain]，论文关于 Local BC 包含的交易有一段说明如下

[^dorri2017blockchain]:Dorri A, Kanhers S S, Jurdak R, 等. Blockchain for IoT security and privacy: The case study of a smart home[C/OL]//2017 IEEE International Conference on Pervasive Computing and Communications Workshops (PerCom Workshops). Kona, HI: IEEE, 2017: 618–623. DOI:[10.1109/PERCOMW.2017.7917634](https://doi.org/10.1109/PERCOMW.2017.7917634).

> Besides the headers, each block contains a number of transactions. For each transaction five parameters are stored in the local BC as shown in the top left corner of the Figure 2. The first two parameters are used to chain transactions of the same device to each other and identify each transaction uniquely in the BC. The transaction’s corresponding device ID is inserted on the third field. ”Transaction type” refers to the type of transaction that can be genesis, access, store, or monitor transactions. The transaction is stored on the fifth field if it comes from the overlay network, otherwise, this filed is kept blank. The local BC is kept and managed by a local miner.

其中提到的 Figure 2 如下，Local BC的区块体是一系列交易的集合，每个交易的第一个字段填充相同设备上一个交易的交易号，用来将相同设备的所有交易链接到一起。每个交易的交易号是第二个字段，设备ID是第三个字段。如果交易来自 Overlay，交易内容存在第五个字段，否则留空。

![Figure 2: overlay of smart home](/images/总结-区块链与D2D/54513763-4e0ce980-4993-11e9-9433-6c0ad7714b7b.png)

### 4.3 当前难点

难点也是可行性问题在于

1. 支付合约的设计实现
2. 如何根据加密后的位置查找附近设备
3. 已知 IP/MAC 地址如何组件 D2D 网络
4. 如何利用合约实现内容缓存/计算卸载
5. 完成任务的私链如何存在公链上
   - 如果存哈希，私链主体内容存在哪里
   - 如果存内容，公链的区块是否能容纳