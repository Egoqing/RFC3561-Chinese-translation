# RFC6351-Chinese-translation
RFC6351:Ad hoc On-Demand Distance Vector (AODV) Routing中文翻译

- [RFC3561:Ad hoc On-Demand Distance Vector (AODV) Routing](https://www.rfc-editor.org/rfc/rfc3561.html#section-6.12)
- [论文:Ad-hoc on-demand distance vector routing](https://ieeexplore.ieee.org/document/749281)
- Translator：[ego-Dong](https://github.com/Egoqing)
<br/>

## 目录
- [AODV RFC3561 阅读翻译](#aodv-rfc3561-阅读翻译)
  - [目录](#目录)
  - [摘要](#摘要)
  - [4. 适用性声明](#4-适用性声明)
  - [3. AODV术语](#3-aodv术语)
  - [5 消息帧格式](#5-消息帧格式)
    - [5.1 Route Request (RREQ)消息格式](#51-route-request-rreq消息格式)
    - [5.2 Route Reply (RREP) 消息格式](#52-route-reply-rrep-消息格式)
    - [5.3 Route Error (RERR)消息格式](#53-route-error-rerr消息格式)
    - [5.4 Route Reply Acknowledgment (RREP-ACK)消息格式](#54-route-reply-acknowledgment-rrep-ack消息格式)
  - [6 AODV 操作（AODV Operation）](#6-aodv-操作aodv-operation)
    - [6.1 序列号维护](#61-序列号维护)
    - [6.2 路由表表项和前驱节点列表](#62-路由表表项和前驱节点列表)
    - [6.3 生成路由请求](#63-生成路由请求)
    - [6.4 控制路由请求消息的传播](#64-控制路由请求消息的传播)
    - [6.5 处理和转发路由请求](#65-处理和转发路由请求)
    - [6.6 生成路由回复](#66-生成路由回复)
      - [6.6.1 被请求目的节点生成路由回复报文](#661-被请求目的节点生成路由回复报文)
      - [6.6.2 中间节点生成路由回复](#662-中间节点生成路由回复)
      - [6.6.3 生成无偿RREP](#663-生成无偿rrep)
    - [6.7 路由回复的接收处理与转发](#67-路由回复的接收处理与转发)
    - [6.8 单向链路上的操作](#68-单向链路上的操作)
    - [6.9 Hello 报文](#69-hello-报文)
    - [6.10 本地连接维护](#610-本地连接维护)
    - [6.11 路由错误RERR报文,路由过期和路由删除](#611-路由错误rerr报文路由过期和路由删除)
    - [6.12 本地修复](#612-本地修复)
    - [6.13 重启后的操作](#613-重启后的操作)
    - [6.14 接口](#614-接口)
  - [7 AODV和聚合网络](#7-aodv和聚合网络)
  - [8 使用AODV连接其他网络](#8-使用aodv连接其他网络)
  - [9 扩展](#9-扩展)
    - [9.1 Hello间隔扩展格式](#91-hello间隔扩展格式)
  - [10 配置参数](#10-配置参数)
  - [11 安全注意事项](#11-安全注意事项)
  - [12 IANA 考虑事项](#12-iana-考虑事项)
  - [13 IPv6 注意事项](#13-ipv6-注意事项)
  - [14 致谢](#14-致谢)
  - [参考文献](#参考文献)
  - [版权声明](#版权声明)

## 摘要
- Ad hoc On-Demand Distance Vector（AODV）路由协议旨在供ad hoc网络中的移动节点使用。
  它提供快速适应动态链路条件、低处理和内存开销、低网络利用率，并确定通往ad hoc网络内目的地的单播路由。
  它使用目的地序列号，以确保在任何时候都不存在环路问题（即使面对路由控制信息的异常传递），
  避免了与经典距离矢量协议相关的问题（如 "计数到无穷大"）。
<br/>

## 4. 适用性声明
- AODV路由协议是为拥有几十到几千个移动节点的移动ad hoc网络设计的。
- AODV可以处理低、中和相对高的移动率，以及各种数据流量水平。
- AODV被设计用于节点都能相互信任的网络中，或者通过使用预先配置的密钥，或者因为已知没有恶意入侵的节点。
- AODV的设计是为了减少控制流量的传播，消除数据流量的开销，以提高可扩展性和性能。
<br/>

## 3. AODV术语
- 活跃路由(active route)
  - 一条通往目的地的路由，其路由表条目被标记为有效。只有活跃的路由可以用来转发数据包。

- 广播（broadcast）
  - 广播是指向IP有限广播地址（255.255.255.255）传输。广播数据包可能不会被盲目转发，
    但广播对于实现AODV消息在整个ad hoc网络中的传播很有用。

- 目的（destination）
  - 数据包将被传送到的一个IP地址，即“目的节点”。
  - 当自身的地址于目的地址相吻合时，节点就知道自己是一个典型数据包的目的节点
  - 目的节点的路由是由AODV协议提供的，它在路由发现消息中携带所需目的节点的IP地址。

- 转发节点（forwarding node）
  - 一个同意转发以另一节点为目的地的数据包的节点，其方法是将数据包沿着
    使用路由控制信息建立的路径转发到更接近单播目的地的下一跳。

- 转发路由（forward route）
  - 一条为从发起路由发现操作的节点向其期望的目的地发送数据包而建立的路由。

- 无效路由（invalid route）
  - 一个已经过期的路由，在路由表条目中以无效的状态表示。
  - 无效路由是用来存储以前有效的路由信息，并延长时间。
  - 无效路由不能用于转发数据包，但它可以提供对路由修复有用的信息，也可以为未来的RREQ消息提供信息。

- 源节点（originating node）
  - 发起AODV路由发现消息的节点
  - 发现消息将由ad hoc网络中的其他节点处理并可能重传
  - 例如，发起路由发现过程并广播RREQ消息的节点被称为RREQ消息的发起节点

- 反向路径（reverse route）
  - 一条为了将回复（RREP）数据包从目的地或从拥有通往目的地的路由的中间节点转发回源节点而建立的路由

- 序列号（sequence number）
  - 一个由每个发端节点保持的单调增长的数字。在AODV路由协议消息中，它被其他节点用来确定来自发端节点的
    信息的新鲜程度。
<br/>

## 5 消息帧格式
### 5.1 Route Request (RREQ)消息格式
![](https://github.com/Egoqing/RFC3561-Chinese-translation/blob/main/Img/RREQ.jpg)
- Type：1
- J：加入标志；保留给组播
- R：修复标志；保留给组播
- G：Gratuitous RREP标志，表示Gratuitous RREP是否应该单播到目的地IP地址字段中指定的节点（见[6.3](#63-生成路由请求)、[6.6.3节](#663-生成无偿rrep)）
- D：仅目的地标志；表示只有目的地可以响应这个RREQ（见[6.5节](#65-处理和转发路由请求)）
- U：未知序列号；表示目的地序列号未知（见[6.3节](#63-生成路由请求)）
- Reserved：填充0；接收时被忽略
- Hop Count：从发起者IP地址到处理请求的节点的跳数。
- RREQ ID：一个序号，与发端节点的IP地址一起唯一识别特定RREQ
- Destination IP Address：需要路由的目的IP地址
- Destination Sequence Number：发送RREQ的节点 在过去收到的 对目的地的任何路由的最新序列号。
- Originator IP Address：发起路由请求的节点IP地址
- Originator Sequence Number：指向路由请求发起节点的路由条目中所使用的当前序列号。
<br/>

### 5.2 Route Reply (RREP) 消息格式
![](https://github.com/Egoqing/RFC3561-Chinese-translation/blob/main/Img/RREP.jpg)
- Type：2
- R：修复标志；用于多播
- A：需要确认标志位；见[5.4节](#54-route-reply-acknowledgment-rrep-ack消息格式)和[6.7节](#67-路由回复的接收处理与转发)
- Reserved：填充0；接收时被忽略
- Prefix Size：如果非零，则 5 位前缀大小指定所指示的*下一跳可用于与请求的目的地具有相同路由前缀（由前缀大小定义）的任何节点*
  <font color=blue> 
  - 子网前缀;前缀相同代表在同一子网？路由前缀是否就代表子网号？
  - 下一跳节点可以为子网内的所有节点转发数据
  - 是否意味着，节点收到RREP，设置前向指针时，要将前往某个子网的下一条设置为转发该RREP到自身的邻居节点的IP地址
  - 前缀大小与IP掩码作用相同，用于说明子网中有多少个节点
  </font>
- Hop Count：从发起者IP地址到目的地IP地址的跳数。对于多播路由请求，这表示到发送RREP的多播树成员的跳数。
- Destination IP Address：为其提供路由的目的地 IP 地址，即该路由的目的节点
- Destination Sequence Number：与路由关联的目标序列号
- Originator IP Address：发起路由请求的源节点的IP地址
- Lifetime：接收 RREP 的节点认为路由有效的时间（以毫秒为单位）
- 请注意，前缀大小允许子网路由器为路由前缀定义的子网中的每个主机提供路由，路由前缀由子网路由器的 IP 地址和前缀大小确定。 
  为了利用这个特性，子网路由器必须保证所有共享指定子网前缀的主机的可达性。 详见[第7节](#7-aodv和聚合网络)。 
  **当前缀大小不为零时，任何路由信息（和前体数据）都必须与子网路由相关，而不是该子网上的单个目标 IP 地址**
- 当发送 RREP 消息的链路可能不可靠或单向时，使用“A”位。 当 RREP 消息包含设置的“A”位时，RREP 的接收者应该返回一个 RREP-ACK 消息。 
  请参见[第 6.8 节](#68-单向链路上的操作)
<br/>

### 5.3 Route Error (RERR)消息格式
![](https://github.com/Egoqing/RFC3561-Chinese-translation/blob/main/Img/RERR.jpg)
- Type：3
- N：勿删除标志，当节点对链路进行本地修复时设置，上游节点不应删除该路由
- Reserved：填充0；接收时被忽略
- DestCount：消息中包含的不可达目的地的数量； 必须至少为 1
- Unreachable Destination IP Address：由于链接中断而无法访问的目标的 IP 地址
- Unreachable Destination Sequence Number：路由表条目中，前一个不可达的目的IP地址字段中列出的目标的序列号
- 每当链路中断导致一个或多个目的地变得无法从某些节点的邻居到达时，都会发送 RERR 消息。 
  请参阅[第 6.2 节](#62-路由表表项和前驱节点列表)了解有关如何维护此确定的适当记录的信息，以及有关如何创建目的地列表的规范的[第6.11节](#611-路由错误rerr报文路由过期和路由删除)
<br/>

### 5.4 Route Reply Acknowledgment (RREP-ACK)消息格式
- 必须发送路由回复确认（RREP-ACK）消息以响应设置了“A”位的 RREP 消息（参见[第5.2节](#52-route-reply-rrep-消息格式)）
- 当可能存在阻止路由发现周期的完成的单向链路的危险时，需要回复RREP-ACK
  - 可能指的是RREP丢包的可能
![](https://github.com/Egoqing/RFC3561-Chinese-translation/blob/main/Img/RREP-ACK.jpg)
- Type：4
- Reserved：填充0；接收时被忽略
<br/>

## 6 AODV 操作（AODV Operation）
- 本节描述节点生成路由请求 (RREQ)、路由回复 (RREP) 和路由错误 (RERR) 消息以用于向目的地进行单播通信的场景，
  以及如何处理消息数据。 为了正确处理消息，必须在路由表条目中为感兴趣的目的地维护某些状态信息。
- 所有 AODV 消息都使用 **UDP** 发送到端口 654。  
### 6.1 序列号维护
- 每个节点上的每个路由表条目**必须**包含目的节点（IP地址）序列号的最新可用信息。这个序号称为**目的序列号**
- 每当节点 从可能收到的与该目的地相关的 RREQ、RREP 或 RERR 消息中 获取到有关序列号的新（即，不陈旧的）信息时，它就会更新
- AODV 依赖于网络中的每个节点来拥有并维护其**目的序列号**，以保证通往该节点的所有路由的无环路
<br/>

- 一个目标节点在两种情况下增加自己的序列号
  - 在节点发起路由发现之前，它必须增加**自己的序列号**。 这可以防止与先前建立的，指向 RREQ 发起者的，反向路由发生冲突。
  - <font color=red>在目的节点为响应 RREQ 发起 RREP 之前，它必须将自己的序列号更新为当前序列号和 RREQ 包中的目标序列号
    中的</font>**<font color=red>最大值</font>**
<br/>

- 当目的地**增加**其序列号时，它必须将序列号值视为**无符号数**
  - 可表示更大范围的非负整数
  - 取到最大值，再递增将循环到0，而不是(绝对值)最大负整数（有符号数用二进制补码表示）
  - 这与处理比较两个 AODV 序列号的结果的方式相反
<br/>

- 为了确定关于目的地的信息不是陈旧的，节点将其（已知的）序列号的当前数值 与从 传入的 AODV 消息中 获得的数值进行比较。
  - **比较**序列号时，需要将序列号值视为**有符号数**
  - 如果从传入序列号的值中减去当前存储的序列号的结果小于零，则必须丢弃 AODV 消息中与该目的地相关的信息，
    因为与节点当前存储的信息相比，该信息是陈旧的
  <font color=blue>  
  - 值得注意的是，当两个无符号数的第一位不同时，大数减小数可能会减出负值。可能作者默认不会出现符号位不同相比较的情况
  </font> 
<br/>

- **唯一的意外情况**:当与路由表中前往目的节点的下一跳间的链路断开或超时时，由节点递增其路由表中目的节点的序列号
  - 节点通过查询其路由表来确定哪些目的地使用特定的下一跳
  - 在这种情况下，对于使用下一跳的每个目的地，节点都会增加序列号并将路由标记为无效（另请参见[第 6.11](#611-路由错误rerr报文路由过期和路由删除)、[6.12 节](#612-本地修复)）
  - 这样才能确保后续路由信息更新时，保存的是一条新发现的路由
  <font color=blue> 
  - 只是标记为无效，而不是直接删除，是为了让节点知道，如此“新鲜”的路由已经失效，避免将不新鲜的路由当作新的可用路由
  </font>
  - 当节点收到关于目的节点更新鲜的路由信息时，需要更新路由表中的相应条目
  - 递增目的节点序列号后，后续下游中间节点没有足够新鲜的路由，直到RREQ传到目的节点那里，目的节点更新自己的序列号，
  然后回应RREP
<br/>

- 只有在以下情况下，节点才可以更改目的地的路由表条目中的序列号：
  - 它本身就是目标节点，并为自己提供一条新路由，或者
  - 它接收到一个 AODV 消息，其中包含有关目标节点序列号的新信息
  - 通往目标节点的路径到期或中断
<br/>

### 6.2 路由表表项和前驱节点列表
- 当一个节点收到来自邻居的AODV控制包，或为一个特定的目的地或子网创建或更新路由时，它将检查其路由表是否有该目的地的条目
  - 如果该目的地没有相应的条目，就会创建一个条目
  - 序列号要么根据控制包中包含的信息确定，要么将有效序号字段设置为假
  - 只有当控制包中新的序列号是以下三种情况之一时，路由才会被更新：
    - 大于路由表中的目的地序列号
    - 序列号相等，但（新信息的）跳数加1，小于路由表中现有的跳数
    - 序列号是未知的<font color=blue>（路由表中的序列号未知）</font>
<br/>

- 路由表项的寿命字段要么从控制包中确定，要么被初始化为*ACTIVE_ROUTE_TIMEOUT*
  这个路由现在可以用来发送任何排队的数据包，并满足任何未完成的路由请求。
<br/>

- 传输数据包时，不仅要更新前往目的节点的路由的生存时间，还有更新前往源节点的路由的生存时间
  - 每次使用路由转发数据包时，节点前往 源、目的地 和 通往目的地路径上的下一跳 的活跃路由寿命字段被更新为
    不低于*当前时间*加上*ACTIVE_ROUTE_TIMEOUT*  
  - 由于每个源和目的对之间的路由是预期对称的，所以 沿着返回IP源的反向路径上的前一跳 的活跃路由寿命，也被更新为不低于
  当前时间加*ACTIVE_ROUTE_TIMEOUT*
  - 无论目的地是一个节点还是一个子网，每次使用该路由时都会更新活跃路由的寿命
<br/>

- 节点不仅要将经过其的每条有效路由记录在路由表中，还需要维护一个可能在此路由上转发数据包的前驱列表
  - **<font color=blue>可能有多个源节点需要通过该节点前往同一个目的节点**；
    导致在前往同一个目的节点的路由上，该节点有多个前驱节点</font>
  - 在检测到下一跳链路丢失的情况下，这些前驱节点将收到来自该节点的通知
  - 路由表条目中的前驱节点列表包含那些向其生成或转发路由回复的*邻居节点*
<br/>

### 6.3 生成路由请求
- <font color=blue>根据上下文，本节中的“节点”都是指“RREQ发起者”即源节点</font>
- 当一个节点确定它需要一个到目的地的路由而没有一个可用的路由时，它就会转播RREQ
  - 产生这种情况的原因是——该节点以前不知道目的地，或者以前到目的地的有效路由**过期**或被标记为**无效**
  - <font color=red>RREQ消息中的目的地序列号字段是最后一个已知的该目的地的目的节点序列号，并从路由表的
    目的地序列号字段中复制出来</font>
    <font color=blue>
    - 路径上的中间节点可能有多种大于请求中的序列号但各不相同的目的节点序列号
    </font>
    - 如果不知道序列号，必须设置未知序列号标志
  - RREQ消息中的发起者序列号是节点自己的序列号，它在插入RREQ之前被递增
    <font color=blue>
    - 有点模糊：中间节点也会去更新自己的序列号，并放到RREQ中吗？
    - 按照5.1节的描述，这个字段看上去是为了保证反向路由的新鲜程度，因此应该只有源节点（RREQ发起者）才应该这样做
    </font> 
  - RREQ ID字段从当前节点使用的最后一个RREQ ID开始递增1
  - 每个节点只保留一个RREQ ID
  - Hop Count字段被设置为零
<br/>

- 在广播RREQ之前，发端节点将缓存RREQ ID和RREQ的Originator IP地址（自己的地址）持续*PATH_DISCOVERY_TIME*
  - 这样，当该节点再次收到来自其邻居的数据包时，它将不会重新处理和重新转发该数据包
<br/>

- 发端节点经常期望与目的地节点进行双向通信。在这种情况下，仅发端节点拥有通往目的地节点的路由是不够的
  - 目的节点也必须拥有回到发端节点的路由
  - 为了尽可能有效地实现这一点，中间节点生成的任何RREP（如[第6.6节](#66-生成路由回复)）传递给发端节点时，
    应该伴随着一些通知目的节点返回发端节点的路由的行动
  - 发端节点通过设置'G'标志在中间节点中选择这种操作模式。关于中间节点响应设置了'G'标志的RREQ所采取的行动，详见[6.6.3节](#663-生成无偿rrep)
<br/>

- 一个节点不应该每秒发出超过*RREQ_RATELIMIT*个RREQ消息
  - 在广播RREQ后，一个节点等待RREP（或其他控制消息，其中有关于到适当目的地的路由的当前信息）
  - 如果在*NET_TRAVERSAL_TIME*毫秒内没有收到路由，节点可以再次尝试通过广播另一个RREQ来发现一个路由
    直到在最大的TTL值下，达到最大值*RREQ_RETRIES*次
  - 每次新的尝试都必须增加和更新RREQ ID
  - 对于每次尝试，IP头的TTL字段根据[第6.4节](#64-控制路由请求消息的传播)中规定的机制进行设置，以便能够
    控制RREQ在每次重试中的传播范围。
<br/>

- 等待路由的数据包（即，在RREQ发送后等待RREP）应该被缓冲。缓冲应该是 "先入先出"（FIFO）
  - 如果在最大TTL下尝试了*RREQ_RETRIES*次数而没有收到任何RREP
  - 那么所有前往相应目的地的数据包都应该从缓冲区中丢弃，并且应该向应用程序发送目的地不可达消息
<br/>

- 为了减少网络中的拥堵，源节点对单一目的地的路由发现的重复尝试必须利用二进制指数退避
  - 在源节点第一次广播RREQ时，它等待*NET_TRAVERSAL_TIME*毫秒以接收RREP
  - 如果在这段时间内没有收到RREP，源节点就会发送一个新的RREQ
  - 当计算发送第二个RREQ后等待RREP的时间时，源节点必须使用二进制指数退避
  - 因此，对应于第二个RREQ的RREP的等待时间是*2 * NET_TRAVERSAL_TIME*毫秒
  - 如果在这个时间段内没有收到RREP，可以发送另一个RREQ，在第一个RREQ之后最多再尝试*RREQ_RETRIES*次
  - **对于每一个额外的尝试，RREP的等待时间乘以2，这样时间就符合二进制指数退避**
<br/>

### 6.4 控制路由请求消息的传播
- 为了防止不必要的全网传播RREQ，发端节点应该使用**扩展环搜索技术**
  - 在扩展环搜索中，发端节点最初在RREQ包的IP头中使用TTL = *TTL_START*，并将接收RREP的超时时间设置为*RING_TRAVERSAL_TIME*毫秒
  - RING_TRAVERSAL_TIME的计算方法如[第10节](#10-配置参数)中所述
  - 用于计算RING_TRAVERSAL_TIME的TTL_VALUE被设定为等于IP头中TTL字段的值
  - 如果RREQ超时而没有相应的RREP，发端者会再次广播RREQ，其TTL按*TTL_INCREMENT*递增
  - 这种情况一直持续到RREQ中设置的TTL达到*TTL_THRESHOLD*为止。
  - 在达到阈值后，每次使用的 TTL 都将被设置为 *NET_DIAMETER*。
  - 达到阈值之前，每次尝试的接收RREP的超时都是RING_TRAVERSAL_TIME
  - 当希望所有的重试都能穿越整个ad hoc网络时，可以通过配置TTL_START和TTL_INCREMENT都是与NET_DIAMETER相同的值来实现
<br/>

- 存储在无效路由表项中的跳数Hop Count表示路由表中到该目的地的最后已知跳数
  - **无效路由中存储了很多历史信息——软状态**
  - 当以后需要到同一目的地的新路由时（例如，在路由丢失时），RREQ IP头中的TTL最初被设置为Hop Count加       TTL_INCREMENT
  - 此后，在每次超时后，TTL都以TTL_INCREMENT递增，直到达到TTL=TTL_THRESHOLD
  - 在此之外，将使用TTL=NET_DIAMETER。一旦TTL = NET_DIAMETER，等待RREP的超时将被设置为    NET_TRAVERSAL_TIME，如[6.3节](#63-生成路由请求)所规定
<br/>

- 在（current_time + DELETE_PERIOD）之前，过期的路由表项不应该被删除（见[第6.11节](#611-路由错误rerr报文路由过期和路由删除)）
  - 否则，与路由相对应的软状态（例如，最后已知的跳数）将被丢失
  - 此外，还可以配置一个较长的路由表项清除时间。
  - 任何等待RREP的路由表项都不应该在（current_time + 2 * NET_TRAVERSAL_TIME）之前被清除 
<br/>

### 6.5 处理和转发路由请求
- 当一个节点收到一个RREQ时，它首先创建或更新一个没有有效序列号的去往前一跳的路由（见[第6.2节](#62-路由表表项和前驱节点列表)）
- 然后检查它是否在至少最后的*PATH_DISCOVERY_TIME*内收到一个具有相同发起人的IP地址和RREQ ID的RREQ
  - 如果已经收到了这样的RREQ，该节点将默默地丢弃新收到的RREQ
- 本小节的其余部分描述了对没有被丢弃的RREQ采取的行动，即**首次收到RREQ**应采取的行动
  - <font color=blue> 因此，即使后续收到了跳数更小的RREQ(源IP与RREQ ID相同)，会**直接丢弃**，而不是更新前往源节点的路由</font>
<br/>

- 首先，它首先将RREQ中的跳数值增加1，加上通过中间节点的新的一跳（上一跳到本节点）
  - 然后，该节点使用最长前缀匹配，搜索到发端人IP地址的反向路由（见[第6.2节](#62-路由表表项和前驱节点列表)）
  - 如果有必要，将创建该路由，或使用路由表中RREQ的发起人序列号进行更新
  - 如果该节点收到一个RREP返回到发起RREQ的节点（由发起者IP地址识别），就需要这个反向路由
- 当反向路由被创建或更新时，也会对该路由进行以下操作
  - 比较RREQ的发起者序列号与路由表条目中的相应目的地序列号，如果前者大于现有值，则复制该序列号
  - 有效序列号字段被设置为真
  - 设置路由表中的下一跳为从其处收到RREQ的节点（从IP头中的源IP地址获得，通常不等于RREQ消息中的发起者IP地址字段）的IP地址
    - **<font color=blue>设置反向指针，指向发来第一个收到的RREQ 的邻居节点，作为前往源节点路由(反向路径)的下一跳节点</font>**
  - 跳数从RREQ消息中的 "Hop Count"中复制
<br/>

- 每当收到RREQ消息时，面向起源者IP地址的反向路由条目的寿命被设置为（*ExistingLifetime*，*MinimalLifetime*）中
  的最大值，其中
  MinimalLifetime = (current time + 2* NET_TRAVERSAL_TIME - 2 * HopCount * NODE_TRAVERSAL_TIME)
  <font color=blue>
  - 来回穿越网络的时间减去在源与中间节点往返的时间
  </font>
<br/>  

- 当前节点可以使用反向路由来转发数据包，其方式与路由表中的任何其他路由相同
<br/>

- 如果一个节点没有生成RREP（遵循[第6.6节](#66-生成路由回复)的处理规则），并且如果传入的IP头的TTL大于1，则该节点在
  其配置的每个接口上更新并广播RREQ到地址255.255.255.255
  - 更新RREQ，出站IP头中的TTL或跳数限制字段减少1，而RREQ消息中的跳数字段增加1，加上通过中间节点的新跳
    - <font color=blue>疑问：不是在收到RREQ的时候就加1了吗？解答：这个加一还是相比于刚收到的RREQ来说的</font>
  - 最后设置RREQ请求的目的节点序列号为*收到的RREQ报文中的目的节点序列号*和*节点当前持有的目的节点序列号*中的最大值
  - 然而，中间节点**不能**修改其维护的目的地序列号的值，<font color=red>即使在传入的RREQ中收到的值大于转发节点当前维护的值</font>
    - **<font color=blue>因为此时中间节点不能确定，现在前往目的节点的路由是否还会经过它</font>**
    - <font color=blue>源节点请求路由时，目的序列号还比较旧，发现的时候要更新RREQ中的目的序列号，以便找到新鲜的路由</font>
<br/>

- 否则，如果一个节点确实生成了RREP，那么该节点就会丢弃RREQ
  - 请注意，如果中间节点对某一特定目的地的RREQs的每一次传输都进行回复，那么结果可能是该目的地没有收到任何发现消息
    - <font color=blue>中间节点代为回复，路由发现报文不会传到目的节点</font>
  - 在这种情况下，目的地没有从RREQ消息中了解到到发端节点的路由。这可能导致目的地启动路由发现
  - 为了让目的地了解到到发端节点的路由，如果出于任何原因，目的地可能需要到发端节点的路由，发端节点应该在RREQ中设置
    "无偿RREP"（'G'）标志
  - 如果作为对设置了 "G "标志的RREQ的回应，一个中间节点返回一个RREP，它也必须向目的地节点单播一个无偿的RREP（见[第6.6.3节](#663-生成无偿rrep)）
<br/>

### 6.6 生成路由回复
- 一个节点产生RREP，如果有以下情况之一
  - 它本身是目的地
  - 它有一条通往目的地的活跃路由，且路由表条目中的目的地序列号大于或等于RREQ的目的地序列号，并且 "仅目的地"（'D'）
    标志未被设置
<br/>

- 在生成RREP消息时，节点将RREQ消息中的目的地IP地址和发起人序列号复制到RREP消息的相应字段中
  - 根据是否为被请求的目的节点，节点对RREP的处理方式不同
    - [是目的节点](#661-被请求目的节点生成路由回复报文)
    - [是有到目的地足够新鲜路由的中间节点](#662-中间节点生成路由回复)
  <font color=blue>
  - 复制发起人的序列号可以保证RREP沿着一条足够新鲜的路径到达发起路由请求的源节点
  - 困扰的是，[5.3节](#53-route-error-rerr消息格式)中并没有存放发起者序列号的字段
  </font> 
<br/>

- 一旦创建，RREP将被**单播**到面向RREQ发起者的下一跳，根据路由表中针对RREQ发起者的表项确定下一跳
  - 当RREP被转发回发起RREQ消息的节点时，在每一跳上Hop Count字段都会递增一
  - 因此，当RREP到达发端者时，Hop Count代表目的地与发端者的距离，以跳数计
<br/>

#### 6.6.1 被请求目的节点生成路由回复报文
- 如果生成节点是目的地本身
  - 如果RREQ数据包中的序列号等于其序列号加一后的值，则目的节点必须将自身的序列号增加一
    <font color=blue>
    - 取自身和RREQ中序列号的最大值（见[6.1节](#61-序列号维护)目的节点增加其序列号的两种情况）
    - 结合前后两种说法可以得出结论：RREQ中的目的节点序列号至多只可能比目的节点的序列号多1
    - 即网络中前往该节点的最新的路由断开，中间节点修复路由时将目的节点序列号递增1
    - 但也有例外，即节点刚开机，它不知到自己的序号是多少（unknow）
    </font>
  - 否则在生成RREP之前，目的节点不改变其序列号。
  - 目的地节点将**其（也许是新增加的）序列号**放入**RREP的目的地序列号字段**中
  - 并在RREP的跳数字段中输入数值0
  - 目的地节点将数值MY_ROUTE_TIMEOUT（见[第10节](#10-配置参数)）复制到RREP的Lifetime字段中
  - 在温和的限制条件下，每个节点可以重新配置其MY_ROUTE_TIMEOUT的值（见[第10节](#10-配置参数)）
<br/>

#### 6.6.2 中间节点生成路由回复
- 如果生成RREP的节点不是目的地节点，而是从发起者到目的地路径上的一个中间跳，它将**其已知的目的地序列号**复制
  到**RREP消息的目的地序列号字段**中。
- 中间节点将最后一跳节点（它从该节点收到RREQ，如IP头中的源IP地址字段所示）放入前向路由条目，即目的地IP地址的条目，
  的前驱节点列表
- 中间节点还会将面向目的地的下一跳放入反向路由条目的前驱节点列表，完成RREQ消息数据中的发起者IP地址字段对应条目的更新
- 中间节点将其距离目的地的跳数（由路由表中的跳数表示）Count字段放在RREP中
- RREP的Lifetime字段是通过从其路由表条目中的过期时间减去当前时间来计算的。
<br/>

#### 6.6.3 生成无偿RREP
- 在一个节点收到RREQ并以RREP回应后，它将丢弃RREQ
- 如果RREQ设置了 "G "标志，并且中间节点向发端节点返回了RREP，它还必须向目的地节点单播一个无偿的RREP
  （告知目的节点前往源节点的路由）
- 将被发送至所需目的地的无偿RREP在RREP消息字段中包含以下值
  - Hop Count：发端节点的路由表条目中显示的跳数
  - Destination IP Address：发起RREQ的节点的IP地址
  - Destination Sequence Number：来自RREQ的发端者序列号
  - Originator IP Address：RREQ中的目的地节点的IP地址
  - Lifetime：中间节点所知道的通往RREQ发起者的路由的剩余寿命
<br/>

- 然后，无偿的RREP被发送到目的地节点路由上的下一跳，就像目的地节点已经为发端节点发出了RREQ
  而这个RREP是为了响应那个（虚构的）RREQ而产生的
- 无论 "G "位是否被设置，发送给RREQ发起者的RREP都是一样的。
<br/>

### 6.7 路由回复的接收处理与转发
- 当一个节点收到RREP消息时，它会搜索（使用最长前缀匹配）到前一跳的路由。
  - 此处前一跳指去往目的地的下一跳，节点从其处获得RREP
  - 如果需要，将为前一跳创建一条路由，但没有有效的序列号（见[6.2节](#62-路由表表项和前驱节点列表)）
  <font color=blue>
  - 因为无法从RREP中获知前一跳(前往目的地路由上的下一跳)节点的序列号
  - 接收到RREQ时，同样也会创建一个前往前一跳，序列号无效的路由
  </font>
- 该节点将RREP中的跳数值增加1，算上通过中间节点的新跳。把这个增加的值称为 "新跳数"
  - <font color=blue>从前一跳到本节点，跳数增加一</font>
- 然后，如果该目的地的**正向路由还不存在**，则**创建该路由**
- 否则，节点将消息中的目的地序列号与它自己存储的RREP消息中的目的地IP地址的目的地序列号相比较
  - 经过比较，只有在以下情况下才会更新现有的条目
    - 路由表中的序列号在路由表条目中被标记为无效 <font color=blue>（序列号无效和路由无效是两个概念）</font>
    - RREP中的目的地序列号大于节点的目的地序列号副本，并且路由表中的已知值是有效的
    - 序列号是相同的，但路由被标记为不活跃
    - 序列号相同，而新跳数小于路由表条目中的跳数
<br/>

- 如果到目的地的路由表条目被创建或更新，则会发生以下操作
  - 路由被标记为活跃的 <font color=blue>(过期的路由被标记为不活跃)</font>
  - 目的地序列号被标记为有效 <font color=blue>(中断的路由被标记为无效)</font>
  - 路由条目中的下一跳被指定为*从其处收到RREP的节点*，该节点由IP头的源IP地址字段表示
  - 跳数被设置为"新跳数"的值
  - 过期时间被设置为当前时间加上RREP消息中的寿命值
  - **设置路由表项中的目标序列号为RREP消息中的目标序列号**
- 当前节点随后可以使用该路由将数据包转发到目的地
<br/>

- 如果当前节点不是RREP消息中发起者IP地址所指示的节点，并且如上所述创建或更新了一个转发路由（正向路由）
  - 则该节点会查阅其起源节点的路由表条目，以确定RREP数据包的下一跳
  - 然后使用该路由表条目中的信息向发起者转发RREP
  - 如果一个节点通过可能存在错误或单向的链接转发RREP，该节点应设置 "A "标志
  - 要求RREP的接收者通过发送RREP-ACK消息来确认收到RREP（参见[第6.8节](#68-单向链路上的操作)）
<br/>

- 当任何节点发送RREP时，相应目的地节点条目的前驱节点列表将被更新
  - 在其中加入 *RREP被向其转发的下一跳节点*，即反向路径上的下一跳
  - 此外，在每个节点，用于转发RREP的（反向）路由的寿命被更新为 现有值和
    当前时间 + *ACTIVE_ROUTE_TIMEOUT* 中的最大值
  - 最后，前往 面向目的节点的下一跳的 对应条目 的前驱节点列表 被更新为包含面向源的下一跳
<br/>

### 6.8 单向链路上的操作
- RREP传输有可能失败，特别是如果触发RREP的RREQ传输发生在单向链路上
  - 如果同一路由发现尝试产生的其他RREP没有到达发出RREQ消息的节点，那么发起者将在超时后重新尝试路由发现（见[6.3节](#63-生成路由请求)）
  - 然而，同样的情况很可能在没有任何改进的情况下重复发生，**即使反复重试也不会发现路由**
  - 除非采取纠正措施，否则即使发端人和目的地之间的双向路由确实存在，这种情况也会发生
  - 使用广播传输RREQ的链路层将无法检测到这种单向链路的存在
  - 在AODV中，任何节点只对具有相同RREQ ID的第一个RREQ采取行动，而忽略任何后续的RREQ
  - 例如，假设第一个RREQ沿着一条有一个或多个单向链路的路径到达。随后的RREQ可能通过双向路径到达
    （假设存在这种路径）但它将被忽略
<br/>

- 为了防止这个问题
  - 当一个节点检测到它传输的RREP消息失败时，它会在一个 "黑名单 "集中记住失败的RREP的下一跳
  - 这种失败可以通过没有链路层或网络层的确认，例如，RREP- ACK）来检测
  - 一个节点会忽略从其 "黑名单" 集中的任何节点收到的所有RREQ
  - 在BLACKLIST_TIMEOUT期间（见[第10节](#10-配置参数)）之后，节点将从黑名单集中删除
  - 该周期应设置为6.3节所述的执行允许的路由请求重试次数所需时间的上限
<br/>

- 请注意，RREP-ACK数据包不包含任何关于它所确认的RREP的信息
  - 收到RREP-ACK的时间很可能就在 RREP以'A'位发送的时间 之后
  - 这一信息预计足以向RREP发送者提供保证，即该链路目前是双向的，而不需要真正依赖被确认的特定RREP消息
  - 然而，这种保证通常不能被期望永久地保持有效
<br/>

### 6.9 Hello 报文
- 节点可以通过广播本地Hello消息提供连接信息
  - **节点应该只在它是活跃路由的一部分时使用Hello消息**
  - 每个HELLO_INTERVAL毫秒，节点检查它是否在最后的HELLO_INTERVAL内发送了一个广播
    例如，RREQ或适当的第2层消息
  - 如果没有，它可能会广播一个TTL=1的RREP，称为Hello消息，其RREP消息字段设置如下
    - Destination IP Address：节点的IP地址
    - Destination Sequence Number：节点的最新序列号
    - Hop Count：0
    - Lifetime：ALLOWED_HELLO_LOSS * HELLO_INTERVAL
<br/>

- 节点可以通过监听其邻居集合的数据包来确定连接性
  - 如果在过去的DELETE_PERIOD内，它收到了来自邻居的Hello消息，而在超过ALLOWED_HELLO_LOSS * HELLO_INTERVAL
    毫秒的时间内，没有收到任何来自该邻居的数据包（Hello消息或其他）；则节点应该认为与该邻居的链接目前已经丢失
    - <font color=blue>邻居位于活跃路由上，但却未在规定时间里收到其Hello报文</font>
  - 当这种情况发生时，节点应该按照[第6.11节](#611-路由错误rerr报文路由过期和路由删除)的规定进行
<br/>

- 每当一个节点收到来自邻居的Hello消息时，该节点应确保它有一个到邻居的活跃路由，如果有必要，可以创建一个
  - 如果一个路由已经存在，那么该路由的寿命应该被增加，如果有必要，至少是ALLOWED_HELLO_LOSS * HELLO_INTERVAL
  - 到邻居的路由，如果存在的话，随后必须包含来自Hello消息的最新目的地序列号
  - 当前节点现在可以开始使用这个路由来转发数据包
  - 由Hello消息创建的、不被任何其他活跃路由使用的路由将有空的前驱节点列表，**如果邻居搬走并发生邻居超时，**
    **将不会触发RERR消息**
    <font color=blue>
    - 由Hello消息创建，意味着该路由是前往邻居的路由
    - 而前往邻居的路由即为两者间的链路
    - 也就是说不再活跃路由上的链路断开不会触发RERR
    </font>
<br/>

### 6.10 本地连接维护
- 每个转发节点应该跟踪其与活动下一跳的持续连接性
  - 即，在最后的ACTIVE_ROUTE_TIMEOUT期间，向哪些下一跳转发过数据包或从哪些前驱（前一跳）接收过数据包
    以及在最后的（ALLOWED_HELLO_LOSS * HELLO_INTERVAL）期间传输了Hello消息的邻居
  - 节点可以使用一个或多个可用的链路或网络层机制，保持其与这些活动下一跳的持续连接的准确信息，如下所述
    - 任何合适的链路层通知，如IEEE 802.11提供的通知，可用于每次数据包被传输到活动下一跳时确定连接性
      例如，没有链路层ACK或在发送RTS后未能得到CTS，甚至在最大数量的重传尝试后，仍没有，表明与该活动下一跳的链接丢失
    - 如果第二层通知不可用，当下一跳预计转发数据包时，应使用被动确认，方法是监听信道中下一跳进行的传输尝试
      如果在NEXT_HOP_WAIT毫秒内没有检测到传输，或者下一跳是目的地（因此不应该转发数据包），应该使用以下方法之一来
      确定连接性
      - 接收来自下一跳的任何数据包（包括Hello消息）
      - 向下一跳发出RREQ单播，要求提供到下一跳的路由
      - 向下一跳单播ICMP回声请求消息
- 如果通过上述任何一种方法都无法检测到与下一跳的链接，则转发节点应假定该链接已丢失，并按照[第6.11节](#611-路由错误rerr报文路由过期和路由删除)中规定的方法
  采取纠正措施。
<br/>

### 6.11 路由错误RERR报文,路由过期和路由删除
- 一般来说，路由错误和连接中断处理需要以下步骤
  1. 将现有路由无效化
  2. 列出受影响的目的节点，即受影响的路由
  3. 如果有的话，判断哪些邻居节点可能会受到影响（每条路由条目有一个活跃邻居表）
  4. 对这些邻居节点发出适当的 RERR 消息
<br/>

- 路由错误（RERR）消息可以是广播（如果有许多前体）、单播（如果只有1个前体），或迭代单播给所有前体（如果广播不合适）
  - 即使RERR消息被迭代地单播给几个前体，它也被认为是一个单一的控制消息，以便在下面的文本中进行描述
  - 基于这种理解，一个节点不应该每秒产生超过RERR_RATELIMIT RERR消息
<br/>
 
- 节点在三种情况下启动对RERR消息的处理 
  1. 如果它在传输数据时检测到其路由表中活跃路由的下一跳出现链路中断（并且可能尝试过路由修复但失败）
     <font color=blue>
     - **<font color=blue>这意味着，如果可以的话，中间节点会先修复，在发送RERR</font>**
     - 而且先报错再修复显然是不合理的，会造成不必要的开销
     </font> 
  2. 如果它得到一个数据包，目的地是它没有活跃路由的节点，并且没有成功修复（如果使用本地修复）
  3. 如果它从一个邻居那里收到一个或多个活跃路由的RERR
<br/>

- 对于情况（1）节点首先制定一个不可达目的地列表，包括不可达邻居和本地路由表中使用不可达邻居作为下一跳的
  任何额外目的地或子网（见[第7节](#7-aodv和聚合网络)） 
  - 在这种情况下，如果发现一个子网路由是新的不可达的，那么不可达目的子网IP地址由路由条目中，子网前缀追加0构成。
    而在已知前驱节点的路由表中，存在带有子网前缀长度的路由表信息，可以兼容该子网
    <font color=blue>
    - 可以匹配更长前缀的表项也一定能匹配更短前缀
    - 子网前缀追加0到底是什么意思呢？为什么要这么做？
    - 是指由节点的子网前缀追加0构成子网地址吗？
    </font>
- 对于情况（2），不可达目的列表中只有一个无法到达的目的地，也就是无法传送的数据包的目的地
- 对于情况（3），不可达目的列表中应包含：在收到的RERR中，且本地路由表中有对应条目的目的节点，且这些路由条目的下一跳
  为 收到的RERR 的发送者
<br/>

- 列表中的一些无法到达的目的地可能被邻居节点使用，因此可能有必要发送一个（新的）RERR
  - RERR中应该包含那些属于已创建的不可达目的地列表，且有一个非空的前驱节点列表的目的节点
  - 应当接收RERR的邻居全都至少属于一个 在新创造的RERR中的不可达目的节点 的前驱节点列表
  - 如果只有一个唯一的邻居需要接收RERR，RERR应该向该邻居单播
  - 否则，RERR通常被发送到本地广播地址（目的地IP=255.255.255.255，TTL=1）
    并在数据包中包括不可到达的目的地和它们相应的目的地序列号
  - RERR数据包的DestCount字段表示数据包中包含的不可达目的地的数量
<br/>

- 就在传输 RERR 之前，对路由表进行某些更新，这可能会影响不可达目的节点的目的地序列号
  对于每一个不可达的目的节点，相应的**路由表条目更新**如下
  1. 此路由条目的目的节点序列号，如果存在且有效，则在上述情况（1）、（2）中，**目的节点序列号递增1**
    在上述情况（3），从收到的RERR中**复制目的节点序列号到路由表**
    - <font color=blue>可见就算不启动本地修复也需要将目的节点序列号递增1</font>
  2. 通过**将路由条目标记为无效**来使得该条目无效
  - Lifetime 字段更新为当前时间加上 DELETE_PERIOD，在此之前，不应删除该条目
<br/>

- 请注意，路由表中的 Lifetime 字段具有双重作用
  - 对于活跃路由，它是到期时间，对于无效路由，它是删除时间
  - 如果接收到前往无效路由的数据包，则 Lifetime 字段将更新为当前时间加上 DELETE_PERIOD
    [第10节](#10-配置参数)讨论了 DELETE_PERIOD 的确定
<br/>

### 6.12 本地修复
- 当活跃路由中发生链路中断时
  - 如果目标距离不超过 MAX_REPAIR_TTL 跳，中断链路的上游节点可以选择本地修复链路
  - 为了修复链路中断，节点**增加目的地的序列号**，然后为该目的地广播一个 RREQ
  - RREQ 的 TTL 最初应设置为 max(MIN_REPAIR_TTL, 0.5 * #hops) + LOCAL_ADD_TTL
    - 其中#hops 是当前无法传递的数据包距离发送者(源节点)的跳数
    - 因此，本地修复尝试通常对原始节点不可见 <font color=blue>（RREQ广播是会同时朝着源和目的扩散的）</font>
    - 并且始终具有 TTL >= MIN_REPAIR_TTL + LOCAL_ADD_TTL
  - 启动修复的节点然后等待发现周期以接收响应于 RREQ 的 RREP
  - 在本地修复期间，数据包必须被缓冲
  - 如果在发现周期结束时，修复节点还没有收到该目的地的 RREP（或其他创建或更新路由的控制消息），
    它会按照[第 6.11 节](#611-路由错误rerr报文路由过期和路由删除)中的描述继续发送该目的地的 RERR 消息。
    - <font color=blue> **先修复再报错** </font>
<br/>

- 另一方面，如果节点在发现期间收到一个或多个 RREP（或其他创建或更新的到所需目的地的路由的控制消息）
  - 它首先比较新路由的跳数和路由表中前往该目的地无效路由条目中的跳数
  - 如果新确定的到目的地的路由的跳数大于先前已知路由的跳数，则节点应该为目的地发出 RERR 消息，
    并设置“N”位标志位
    <font color=blue>
    - 修复成功，但是前往目的节点的跳数发生了变化
    - 根据下文，发送RERR是为了源节点能找到到更好的目的地的新路由
    </font>
  - 然后它按照[第 6.7 节](#67-路由回复的接收处理与转发)中的描述，更新其前往该目的地的路由条目
<br/>

- 接收到设置了“N”标志的 RERR 消息的节点**不得**删除到该目的地的路由
  - 如果前往目的地的路由上有多个前驱节点，且RERR 从该路由上的下一跳抵达节点，则采取的唯一行动应该是重传消息
  - 当源节点收到从该路由上的它的下一跳发来的设置了“N”标志的 RERR 消息时，则源节点可以选择重新启动路由发现，
    如[第 6.3 节](#63-生成路由请求)所述
<br/>

- 对路由中的中断链路进行本地修复有时会导致到这些目的地的路由长度增加
  - 在本地修复链路可能会增加能够传送到目的地的数据包的数量，因为当 RERR 传输到源节点时，数据包不会被丢弃
  - 在本地修复链路中断之后向始发节点发送 RERR 可以允许源节点基于当前节点位置找到到更好的前往目的地的新路由
  - 然而，它不需要源节点重建路由，因为源节点可能已经完成或几乎完成了数据会话
<br/>

- 当活跃路由上的一条链路中断后，通常会有多个目的地无法到达
  - 只有有数据包发往目的节点时，位于中断链路上游的节点才会立即进行本地修复路由
  - 其他使用相同链路的路由必须被标记为**无效**，但进行本地修复的节点可以将每个新丢失的路由标记为本地可修复
  - 路由表中的这个本地修复标志必须在路由**超时**时重置（例如，路由在ACTIVE_ROUTE_TIMEOUT毫秒内一直是非活跃状态）
  - 在超时发生之前，当有数据包前往其他目的地时,这些其他路由将会被**按需**修复
  - 因此，这些路由会根据需要进行修复；<font color=red>如果数据包没有到达该路由，则该路由将不会被修复</font>
  - 或者，根据本地拥塞，节点可以开始为其他路由建立本地修复的过程，而无需等待新的数据包到达
  - 通过主动修复因链路丢失而中断的路由，这些路由的将传入数据包不会受到修复路由的延迟，可以立即转发
  - 但是，在接收到数据包之前修复路由存在修复不再使用的路由的风险
  - 因此，根据网络中的本地流量以及是否正在经历拥塞，节点可以选择在接收到数据包之前主动修复路由；
    否则，它可以等到收到数据，然后开始修复路由
<br/>

### 6.13 重启后的操作
- 参与ad hoc网络的节点在重启后必须采取某些行动
  - 因为它可能会失去所有目的地的所有序列号记录，包括它自己的序列号
  - 然而，可能会有邻近的节点把这个节点作为一个活跃的下一跳。这有可能造成路由环路
  - 为了防止这种可能性，重启的每个节点在**传送任何路由发现消息**之前都要等待DELETE_PERIOD
  - 如果节点收到RREQ、RREP或RERR控制包，它应该根据控制包中的序列号信息酌情创建路由条目，但**必须不转发**任何控制包
  - 如果节点收到其他目的地的数据包，它应该按照第6.11小节所述广播一个RERR，并且必须重置等待计时器，
    使其在当前时间加上DELETE_PERIOD后失效
    <font color=blue>
    - 新启动的节点，路由表都是空的。但此时关机前就相邻的节点可能仍维持着邻接关系，即邻居们认为其一直保持活跃
    - 上游节点将数据包转发给新开机节点。而新开机节点因收到目的地是没有路由的数据包，初始化了一个RERR并发送
    - 因为路由表是空的，也不知道前驱节点有谁，只能广播RERR。
      - 由于IP报文传播过程中，IP头中源IP地址和目的IP字段保持不变。因此节点是不知道数据包是从哪个邻居发来的
    </font>
  - 可以证明 [[4](#参考文献)]，当重启的节点从等待阶段出来并再次成为一个活跃的路由器时，它的邻居都不会再把它作为
    一个活跃的下一跳
  - 一旦它收到任何其他节点的RREQ，它自己的序列号就会被更新，因为RREQ总是携带在路上看到的最大目的地序列号
  - 如果没有这样的RREQ到达，该节点必须将自己的序列号初始化为零
<br/>

### 6.14 接口
- 每当收到一个数据包时，AODV必须知道数据包到达的特定接口
  - 因为AODV应该在有线和无线网络上顺利运行，而且因为AODV很可能也会被用于多个无线设备
  - 这包括RREQ、RREP和RERR消息的接收
  - 每当收到来自新邻居的数据包时，接收该数据包的接口就会和所有其他适当的路由信息一起被记录到该邻居的路由表条目中
  - 同样，每当学习到一个新目的地的路由时，可以到达目的地的接口也被记录到目的地的路由表条目中
- 当有多个接口时，一个节点重发RREQ消息时，会在所有被配置为在ad-hoc网络中运行的接口上重播该消息
  - 除了那些已知所有节点邻居已经收到RREQ的接口
- 当一个节点需要传输RERR时，它应该只在那些有该路由的邻居前体节点的接口上传输。 
<br/>

## 7 AODV和聚合网络
- AODV被设计用于具有IP地址的移动节点，这些节点之间不一定有联系，以创建一个ad hoc网络
  - 在某些情况下，移动节点的集合可能以一种固定的关系运作，并共享一个共同的子网前缀
    在一个已经形成的ad hoc网络区域内一起移动。称这样的节点集合为 **子网**
  - 在这种情况下，子网中的单个节点有可能为子网中的所有其他节点发布可达性通告，即用RREP消息响应
    任何 请求到具有（相同）子网路由前缀的任意节点的路由 的RREQ消息
  - 称这个单一节点为 **子网路由器** 
  <font color=red>
  - 为了使子网路由器能够为整个子网运行AODV协议，它必须为整个子网保持一个目的序列号
  </font>
  - 在子网路由器发送的任何此类RREP消息中，RREP消息的前缀大小字段必须被设置为子网前缀的长度
  <font color=red>
  - 共享子网前缀的其他节点不应发出RREP消息，而应将RREQ消息转发给子网路由器
  </font>

<br/>

- 对给出子网路由（即前缀长度不为零）的RREP的处理与对特定主机RREP消息的处理相同
  - 每个收到带有前缀大小信息的RREP的节点应创建或更新子网的路由表条目
    包括由子网路由器提供的序列号，以及适当的前驱节点信息
  - 然后，在未来，该节点可以使用该信息来避免为同一子网的其他节点发送未来的RREQ
<br/>

- 当一个节点使用子网路由时，可能是一个数据包被路由到子网的一个IP地址，而这个地址没有分配给
  ad hoc网络中的任何现有节点
  - 当这种情况发生时，子网路由器必须向发送节点返回ICMP Host Unreachable消息
  - 收到这种ICMP消息的上游节点应该记录特定的IP地址是不可达的信息，但不应该使任何匹配的子网前缀的路由条目无效
  - <font color=blue>以免影响后续前往子网的路由;单个节点不可达不代表整个子网不可达</font>
<br/>

- 如果子网中的多个节点发布由子网前缀定义的子网的可达性
  - 那么具有最低IP地址的节点被选为子网路由器
  - 而所有其他节点**必须停止**宣传可达性
<br/>

- 本规范中没有定义默认路由（即路由前缀长度为0的路由）的行为 
- 选择共享前缀位的路由应遵循最长的匹配优先的原则 <font color=blue>（找匹配度最长的条目）</font>
[回到顶部](#目录)
<br/>

## 8 使用AODV连接其他网络
- 在某些配置中，一个临时网络可能能够在不使用AODV的外部路由域之间提供连接
  - 如果与其他网络的联系点可以作为外部路由域内任何相关网络的子网路由器（见第7节）那么ad hoc网络可以
    保持与外部路由域的连接
  - 事实上，外部路由网络可以将AODV定义的ad hoc网络作为*过境网络*
  - 为了提供这一功能，一个与外部网络的联系点（称之为基础设施路由器）必须作为 外部网络中每个感兴趣的子网
    的子网路由器，而基础设施路由器可以为外部网络提供可达性
  - 这包括需要为该外部子网维护一个目标序列号
  - 如果多个基础设施路由器提供对同一外部子网的可达性，这些基础设施路由器必须合作（通过本规范范围以外的方式）
    为临时访问这些子网提供一致的AODV语义
<br/>

## 9 扩展
- 在本节中，规定了RREQ和RREP消息的扩展格式。所有这些扩展出现在消息数据之后，并具有以下格式
![](https://github.com/Egoqing/RFC3561-Chinese-translation/blob/main/Img/Extension.jpg)
  - Type：1-255
  - Length：特定类型数据的长度，不包括扩展部分的类型和长度字段，单位为字节
  - 类型在128和255之间的扩展可能不会被跳过。关于扩展的规则将被更全面地阐述，并符合处理IPv6选项的规则。
<br/>

### 9.1 Hello间隔扩展格式
![](https://github.com/Egoqing/RFC3561-Chinese-translation/blob/main/Img/Hello-Interval.jpg)
- Type：1
- Length：4
- Hello Interval：Hello消息的连续传输之间的毫秒数
  - Hello Interval扩展可以附加到TTL==1的RREP报文中，由相邻的接收者用来确定对后续的这类RREP报文
    （即Hello报文；见[第6.9节](#69-hello-报文)）的等待时间
<br/>

## 10 配置参数
- 本节给出了与AODV协议操作相关的一些重要参数的默认值
  - 一个特定的移动节点可能希望改变某些参数，特别是
    - NET_DIAMETER
    - MY_ROUTE_TIMEOUT
    - ALLOWED_HELLO_LOSS
    - RREQ_RETRIES
    - HELLO_INTERVAL
  - 在后一种情况下，节点应在其Hello消息中公布HELLO_INTERVAL，方法是在RREP消息中附加
    Hello Interval Extension
  - 这些参数的选择可能会影响协议的性能
  - 改变NODE_TRAVERSAL_TIME也会改变节点对NET_TRAVERSAL_TIME的估计,因此只能在适当了解ad hoc网络中
    其他节点的行为后才能进行
  - MY_ROUTE_TIMEOUT的配置值必须至少是2*PATH_DISCOVERY_TIME

  |参数名称|值|
  |:-----:|:---:|
  |ACTIVE_ROUTE_TIMEOUT|3,000毫秒|
  |ALLOWED_HELLO_LOSS|2|
  |BLACKLIST_TIMEOUT|RREQ_RETRIES * NET_TRAVERSAL_TIME|
  |DELETE_PERIOD|见下文|
  |HELLO_INTERVAL|1,000毫秒|
  |LOCAL_ADD_TTL|2|
  |MAX_REPAIR_TTL|0.3 * NET_DIAMETER|
  |MIN_REPAIR_TTL|见下文|
  |MY_ROUTE_TIMEOUT|2 * ACTIVE_ROUTE_TIMEOUT|
  |NET_DIAMETER|35|
  |NET_TRAVERSAL_TIME|2 * NODE_TRAVERSAL_TIME * NET_DIAMETER|
  |NEXT_HOP_WAIT|NODE_TRAVERSAL_TIME + 10|
  |NODE_TRAVERSAL_TIME|40毫秒|
  |PATH_DISCOVERY_TIME|2 * NET_TRAVERSAL_TIME|
  |RERR_RATELIMIT|10|
  |RING_TRAVERSAL_TIME|2 * NODE_TRAVERSAL_TIME * (TTL_VALUE + TIMEOUT_BUFFER)|
  |RREQ_RETRIES|2|
  |RREQ_RATELIMIT|10|
  |TIMEOUT_BUFFER|2|
  |TTL_START|1|
  |TTL_INCREMENT|2|
  |TTL_THRESHOLD|7|
  |TTL_VALUE|见下文|

- MIN_REPAIR_TTL应该是到目的地的最后已知跳数
- 如果使用Hello消息，那么ACTIVE_ROUTE_TIMEOUT参数值必须大于（ALLOWED_HELLO_LOSS * HELLO_INTERVAL）
  - <font color=blue>路由寿命应该大于链路寿命，不活跃的路由上的链路断开后不会触发RERR(6.9节)</font>
- 对于给定的 ACTIVE_ROUTE_TIMEOUT 值，这可能需要对 HELLO_INTERVAL 的值进行一些调整，因此需要
  在 Hello 消息中使用 Hello Interval Extension
<br/>

- TTL_VALUE 是执行扩展环搜索时 IP 头中的 TTL 字段的值
  - 这已经在[6.4节](#64-控制路由请求消息的传播)中进一步描述
- TIMEOUT_BUFFER 是可配置的。 其目的是为超时提供缓冲区，以便如果 RREP 由于拥塞而延迟，
  当 RREP 仍在返回源的途中，降低RREP接收超时发生的概率
  - 要省略此缓冲区，请设置 TIMEOUT_BUFFER = 0
<br/>

- DELETE_PERIOD 旨在提供一个时间上界，在这段时间内上游节点 A 可以将邻居 B 作为前往目的地 D 的活跃下一跳，
  而B到D的路由已经失效
  - 超出这段时间后B可以删除前往D的无效路由
  - 上界的确定在一定程度上取决于底层链路层的特性
  - 如果 Hello 消息用于确定到下一跳节点的持续连接的可用性，则 DELETE_PERIOD 必须至少为 
    ALLOWED_HELLO_LOSS * HELLO_INTERVAL
  - 如果链路层反馈被用于检测链路丢失，则 DELETE_PERIOD 必须至少为 ACTIVE_ROUTE_TIMEOUT
  - 如果收到来自邻居的 hello 消息但到该邻居的数据包丢失（例如，由于临时链路不对称）
    - 我们必须对底层链路层做出更具体的假设
    - 我们假设这种不对称性在超过一定时间后就不会持续，例如 HELLO_INTERVAL 的倍数 K
    - 换句话说，如果链路正在工作并且邻居没有发送其他流量，则节点总是可以收到后续K个Hello中的至少一个
    - 综上所述   
      - *DELETE_PERIOD = K * max (ACTIVE_ROUTE_TIMEOUT, HELLO_INTERVAL)*
        (K = 5 建议值)
<br/>

- NET_DIAMETER衡量网络中两个节点间的最大可能跳数
- NODE_TRAVERSAL_TIME是对数据包平均一跳穿越时间的，应该包括
  - 排队时延、中断处理时延、传输时延
  - <font color=blue>从节点收到数据包开始，到下一跳收到数据包结束
- 根据6.3节可知，NET_TRAVERSAL_TIME是数据包在网络中往返一次所需的时间
- 根据6.5节可知，在 PATH_DISCOVERY_TIME 内节点会丢弃重复的路由请求
  </font>
- 如果链路层指示用于检测链路中断，例如在 IEEE 802.11 [[5](#参考文献)] 标准中，则 ACTIVE_ROUTE_TIMEOUT
  应该被设置为更长的时间（至少 10,000 毫秒）
- 如果 Hello 消息用于本地连接信息，则 TTL_START 应至少设置为 2
- AODV 协议的性能对这些常数的选择值**很敏感**，这通常取决于底层链路层协议、无线电技术等的特性
- 如果使用扩展环搜索，则应适当增加 BLACKLIST_TIMEOUT
  - 在这种情况下，它应该是
  - {(TTL_THRESHOLD - TTL_START)/TTL_INCREMENT + 1 + RREQ_RETRIES} * NET_TRAVERSAL_TIME
  - <font color=blue> 即网络中广播RREQ的持续时间, 黑名单上的节点不应该响应RREQ</font>
  - 这是为了考虑可能的额外路由发现尝试
[回到顶部](#目录)
<br/>

## 11 安全注意事项
- 目前，AODV 没有指定任何特殊的安全措施
  - 然而，路由协议是冒充攻击的主要目标
  - 在节点成员身份未知的网络中，很难确定是否发生了冒名攻击，安全防范技术也很困难
  - 然而，当网络成员已知并且存在此类攻击的危险时，必须使用身份验证技术来保护 AODV 控制消息
    例如那些涉及生成不可伪造和加密的强消息摘要或数字签名的技术
  - 虽然 AODV 对用于此目的的身份验证机制没有限制，但对于节点共享允许使用 AH 的适当安全关联的情况，
    IPsec AH 是一个合适的选择
<br/>

- 特别是，应该对 RREP 消息进行身份验证以避免创建到所需目的地的虚假路由
  - 否则，攻击者可能伪装成所需的目的地，并恶意拒绝向目的地提供服务和/或恶意检查和消耗旨在传递到目的地的流量
  - RERR 消息虽然危险性较小，但应该进行身份验证，以防止恶意节点破坏作为通信伙伴的节点之间的有效路由
<br/>

- AODV 不对将地址分配给移动节点的方法做出任何假设，只是假定它们具有唯一的 IP 地址
  - 因此，除了由于通用协议规范而自然产生的问题外，无需特别考虑 IPsec 身份验证标头或密钥交换机制的适用性
  - 然而，如果自组织网络中的移动节点具有预先建立的安全关联，则假定创建安全关联的目的包括授权处理AODV控制消息的目的
  - 鉴于这种理解，移动节点应该能够使用基于其 IP 地址的相同身份验证机制，就像它们在其他情况下使用的一样。
<br/>

## 12 IANA 考虑事项
- AODV 为发送到端口 654 的消息定义了一个“类型”字段
  - 已为此类型字段的值创建了一个新注册表，并分配了以下值

    |消息类别|值|
    |:---:|:---:|
    |Route Request (RREQ)|1|
    |Route Reply (RREP)|2|
    |Route Error (RERR)|3|
    |Route-Reply Ack (RREP-ACK)|4|
<br/>

- AODV 控制消息可以有扩展名，目前，只定义了一个扩展。 
  - 已为扩展的 Type 字段创建了一个新注册表：

    |扩展类别|值|
    |:---:|:---:|
    |Hello Interval|1|
  
  - 可以使用标准操作 见[[2](参考文献)] 分配消息类型或扩展类型的未来值 
<br/>

## 13 IPv6 注意事项
- 有关 IPv6 的详细操作，请参见[[6](参考文献)]。 协议的唯一变化是扩大了地址字段。
<br/>

## 14 致谢
- 特别感谢 UCSB 的 Ian Chakeres 为最近的修订提供了广泛的建议和贡献。 
- 我们感谢宾夕法尼亚大学 Carl Gunter 小组以及斯坦福大学和 CMU 所做的工作，
  以确定以前版本的 AODV 可能遭受路由循环的一些条件（特别是涉及重新启动和丢失的 RERR）。 
  这些努力的贡献者包括 Karthikeyan Bhargavan、Joshua Broch、Dave Maltz、Madanlal Musuvathi 
  和 Davor Obradovic。 
  他们还提出了 DELETE_PERIOD 的想法，为此必须维护到特定目的地的过期路由（尤其是序列号）。 
- 我们也感谢 Sung-Ju Lee（特别是关于本地维修）、Mahesh Marina、Erik Nordstrom（为第 6.11 节提供文本）、
  Yves Prelot、Marc Mosko、Manel Guerrero Zapata、Philippe Jacquet 和 Fred Baker 提出的意见和改进建议
<br/>

## 参考文献
- [1] Bradner, S. "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, 
  RFC 2119, March 1997.
- [2] Narten, T. and H. Alvestrand, "Guidelines for Writing an IANA Considerations Section
  in RFCs", BCP 26, RFC 2434, October 1998.
- [3] Manner, J., et al., "Mobility Related Terminology", Work in Progress, July 2001.
- [4] Karthikeyan Bhargavan, Carl A. Gunter, and Davor Obradovic. Fault Origin Adjudication. 
  In Proceedings of the Workshop on Formal Methods in Software Practice, Portland, OR, 
  August 2000.  
- [5] IEEE 802.11 Committee, AlphaGraphics #35, 10201 N.35th Avenue, Phoenix AZ 85051. 
  Wireless LAN Medium Access Control MAC and Physical Layer PHY Specifications, June 1997. 
  IEEE Standard 802.11-97.
- [6] Perkins, C., Royer, E. and S. Das, "Ad hoc on demand distance vector (AODV) routing for
  ip version 6", Work in Progress.

## 版权声明
- 版权所有 (C) 互联网协会 (2003)。版权所有。本文档及其翻译可以复制和提供给其他人，对其进行评论或以其他方式解释或
  协助其实施的衍生作品可以全部或部分准备、复制、出版和分发，不受任何形式的限制, 前提是上述版权声明和本段包含在所有此类副本和衍生作品中
[回到顶部](#目录)
