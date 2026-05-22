# HCDN — 基于P2P的直播组网集群解决方案（完整面试准备）

> 中国移动事业群平台组项目，本人负责P2P预研、核心代码实现、优化及项目上线。

**技术选型说明**：本项目并非直接使用 WebRTC 库，而是**从 WebRTC 中提取部分协议和思路**，在移动直播场景下做了针对性的裁剪和适配：

| | 说明 |
|---|---|
| **采用** | ICE 打洞框架（host/srflx candidate、连通性检查）、STUN 协议（RFC 5389）、NACK 重传机制、生产者消费者数据流模型 |
| **裁剪** | 去掉 DTLS/SRTP 加密层（内网可信环境，加密层带来不必要的 CPU 开销）、去掉 SDP 协商（信令格式自定义，更轻量）、去掉 SCTP 数据通道 |
| **适配** | 信令传输改为调度服务器 WebSocket 中转；candidate 优先级策略针对直播低延迟调整；新增 CDN 回退路径作为 P2P 降级兜底 |

---

## 一、项目简介

### 30秒版

> "这是我在中国移动事业群平台组参与的项目，是一个基于P2P的直播边缘加速系统。我负责P2P核心模块的开发，包括ICE打洞、STUN探测、缓存管理、内存池设计等。项目的核心价值是通过P2P技术将CDN带宽成本降低60%-80%，整流方案放大率3倍（现网数据），多子流方案放大率8倍（mock测试数据）。"

### 3分钟版（详细介绍）

> "这个项目的背景是：中国移动的直播业务，CDN带宽成本非常高。我们用P2P技术让用户之间互相分享视频流，减少CDN的压力。
>
> 系统是三层树形拓扑：一层节点从CDN拉流，二层节点通过P2P从一层节点拉流，SDK终端通过P2P从二层节点拉流。
>
> 我负责的核心模块：
> 1. **ICE打洞**：我自研了一套裁剪版ICE协议栈，从WebRTC中提取协议思路，针对直播场景裁剪了DTLS加密、SDP协商、SCTP数据通道，只保留了candidate收集、连通性检查、打洞的核心逻辑。实现了5种NAT类型检测和4种candidate类型。
>
> 2. **P2P订阅发布**：统筹打洞流程、数据传输、NACK丢包重传、QOS统计、链路保活。
>
> 3. **缓存模块**：链表存储TS chunk + 位图结构快速查找 + 生产者消费者模式 + 多线程安全。
>
> 4. **内存池**：模仿nginx的arena分配思想，增加了预注册固定大小桶（O(1)分配释放）和空闲块复用。
>
> 5. **优化成果**：NACK命中率从50%提升到90%；多子流方案放大率从3倍提升到8倍。"

---

## 二、项目背景

**问题**：直播场景下，用户播放 HLS 流时完全依赖 CDN 拉流，CDN 带宽成本随并发用户数线性增长。

**目标**：让用户节点之间通过 P2P 互相分担流量，从而减少 CDN 出口带宽，同时不影响用户播放体验。

**核心指标**：P2P 分担率（放大率），现网数据整流放大率约 **3倍**，多子流方案约 **8倍**（mock测试）。

---

## 三、整体架构

```
┌─────────────────────────────────────────────────────────┐
│                    调度服务器 (Schedule Server)           │
│   负责：分组、分配父子节点关系、下发信令、心跳管理          │
└──────────────────────┬──────────────────────────────────┘
                       │ WebSocket 信令
        ┌──────────────┴──────────────┐
        │                             │
   ┌────▼────┐                  ┌────▼────┐
   │ 1层节点  │◄────P2P UDP─────►│ 1层节点  │  ← 直接从CDN拉整流
   │(Level1) │                  │(Level1) │
   └────┬────┘                  └────┬────┘
        │ P2P UDP                    │ P2P UDP
   ┌────▼────┐                  ┌────▼────┐
   │ 2层节点  │                  │ 2层节点  │  ← 从Level1 P2P获取
   │(Level2) │                  │(Level2) │
   └────┬────┘                  └─────────┘
        │ P2P UDP
   ┌────▼────┐
   │  SDK节点 │                                ← 播放器SDK
   │ (Level3)│
   └─────────┘
```

**三层节点体系**（见 `node_info.h`）：
- **Level 1**：路由器/机顶盒插件节点，直连 CDN，为下层节点提供数据源
- **Level 2**：转发节点，从 Level1 P2P 获取数据后再向下转发
- **Level 3（SDK）**：播放器端，消费数据

---

## 四、模块划分

```
starry_sky/src/
├── p2p/
│   ├── p2p_manager         # P2P总管理，单例，协调所有子模块
│   ├── peer_connection     # 单条P2P链路封装（打洞+数据收发）
│   ├── hole_puncher       # ICE打洞实现
│   ├── socket_manager      # UDP Socket管理
│   ├── subscriber         # 订阅端（下层节点）：拉流、NACK重传
│   ├── publisher          # 发布端（上层节点）：推流、转发
│   ├── keep_alive         # 心跳模块
│   ├── candidate           # ICE候选者（host/srflx）
│   └── nat_detect/
│       ├── nat_detect      # NAT类型探测入口
│       ├── stun_msg        # STUN协议报文解析与构造
│       ├── stun_transaction# STUN事务管理
│       ├── ice_natdetect   # ICE格式NAT探测
│       └── ice_stack       # ICE协商栈
├── cache/
│   ├── ts_cache_manager   # TS缓存管理（多TS滑动窗口）
│   ├── data_chunk          # Chunk数据结构 + 内存池
│   ├── memory_manager      # 内存池（arena + 空闲桶）
│   ├── data_session        # 单路流会话，协调拉流/P2P/消费
│   └── data_consumer       # 数据消费者（生产者-消费者模式）
├── report/
│   ├── report_manager      # QOS上报总入口
│   └── statistics          # 统计数据收集
└── http/
    ├── http_client         # CDN HTTP拉流客户端
    └── http_server         # 本地HTTP服务（对接播放器）
```

---

## 五、核心模块详解

### 5.1 ICE打洞（最核心的模块）

#### 为什么不用现成的WebRTC库

> "我考虑过直接用WebRTC，但有几个问题：
> 1. **太重**：WebRTC包含DTLS/SRTP加密、SDP协商、SCTP数据通道等，我们的场景是内网可信环境，加密层带来不必要的CPU开销
> 2. **延迟**：WebRTC的ICE完整流程需要SDP交换，我们的信令格式自定义更轻量
> 3. **适配性**：我们需要在路由器/机顶盒等嵌入式设备上运行，WebRTC的依赖太多
>
> 所以我从WebRTC中提取了ICE协议思路，自己实现了一套裁剪版：
> - 保留了：candidate收集、连通性检查、打洞、Nominate确认
> - 裁剪了：DTLS/SRTP加密、SDP协商、SCTP数据通道
> - 改造了：信令传输改为WebSocket中转（而非SDP交换）、candidate优先级针对直播低延迟调整
> - 新增了：CDN回退路径作为P2P降级兜底"

#### 打洞的完整流程

> "打洞分6步：
>
> 1. **STUN探测**：节点启动时向STUN服务器发送Binding Request，获取自己的公网IP:端口（SRFLX candidate），同时判断NAT类型
>
> 2. **候选者收集**：收集4种candidate——HOST（本地地址）、SRFLX（STUN映射地址）、PRFLX（打洞时发现的地址）、HOST_SYMMETRIC（对称型NAT专用）
>
> 3. **角色分配**：调度服务器指定Master和Slave角色
>
> 4. **候选者交换**：通过调度服务器的WebSocket中转，交换双方的candidate列表
>
> 5. **打洞尝试**：
>    - 按优先级组合candidate pair
>    - Master先发Binding Request（间隔500ms）
>    - Slave后发Binding Request（间隔1000ms）
>    - 双方同时向对方发UDP包，NAT留下'洞'
>
> 6. **连通性检查**：收到Binding Response → 成功；超时10秒 → 失败，回退到relay或CDN"

#### NAT类型检测

**为什么需要先探测NAT？**
不同 NAT 类型（Full Cone / Restrict Cone / Port Restrict Cone / Symmetric）打洞策略不同，Symmetric NAT 几乎无法直接打洞，需要走 TURN 中继或特殊处理。

**实现细节**（`stun_defines.h` + `stun_msg.h`）：
- 实现完整 STUN RFC 3489/5389 协议：Binding Request / Binding Response / Binding Error Response
- 通过向两个不同 STUN Server 发包、比较映射地址，判断 NAT 类型
- 探测结果写入 `NatInfo` 结构，包含：natType、映射地址（ipv4/ipv6）、本地地址

**五种NAT类型**（`ice_definitions.h`）：
```
NAT_NONE → NAT_FULLCONE → NAT_RESTRICTCONE → NAT_PORTRESTRICTCONE → NAT_SYMMETRIC → NAT_UDP_BLOCKED
```

#### 对称型NAT处理

> "对称型NAT是最难处理的，因为它对不同目标使用不同的外部端口，STUN映射地址无法用于打洞。
>
> 我的方案：
> 1. 探测时识别对称型NAT
> 2. 生成HOST_SYMMETRIC_CANDIDATE
> 3. 增加打洞重试次数和尝试端口预测
> 4. 如果仍然失败，回退到relay中继或CDN回源
>
> 实际效果：Full Cone/Restrict Cone打洞成功率>95%，Symmetric NAT约60%。"

#### STUN Binding Request原理

> "节点向 STUN 服务器发一个 UDP 包，STUN 服务器收到后把这个包的源 IP:Port（即 NAT 给这个 UDP 流量映射的公网地址）写进响应包返回。节点收到响应就拿到了自己的 srflx 候选者（公网地址）。连通性检查时，双方互发 STUN Binding Request 到对方的候选者地址，能收到响应就说明这对地址可以打通。"

#### UPnP（NAT穿透的另一种方式）

> "UPnP是NAT穿越的另一种方式，直接在路由器上映射端口，不需要打洞。
>
> 流程：
> 1. 用miniupnpnc库的SSDP协议发现网关设备（IGD）
> 2. 调用UPNP_AddPortMapping映射内部端口到外部端口
> 3. 映射成功后，外部设备可以直接通过公网IP:端口访问
> 4. 定时续租（renew_upnp_port_map），防止映射过期
>
> 注意：UPnP需要路由器支持，且用户需要开启UPnP功能，不是所有场景都可用。"

---

### 5.2 缓存模块

#### 缓存模块设计

> "缓存模块需要解决4个问题：chunk存储、快速查找、NACK重传、多线程安全。
>
> 数据结构：
> - **链表**存储DataChunk对象（按chunkIndex排序）
> - **unordered_map**建立chunkIndex→DataChunk的索引（O(1)查找）
> - **位图（bitmap）**标记已收到的chunk序号（快速判断缺失的chunk）
>
> 线程模型：
> - **生产者**：asio收包回调线程，收到数据后调用addChunk()
> - **消费者**：业务线程，调用getChunk()获取数据
> - **NACK线程**：定期检查位图，发现缺失的chunk发起重传请求"

#### NACK命中率从50%提升到90%

> "原实现的问题：用估算的chunk总数作为范围找缺口，误报了大量'还没到达但正在路上'的chunk为丢失。
>
> 举个例子：当前已收到chunk_0~chunk_5，但chunk_3还没到。原实现会立即认为chunk_3丢失，发起NACK重传。但实际上chunk_3可能只是延迟了，还在路上。
>
> 优化方案：
> - 不再估算总数，改为只在已收到后续chunk时才认定中间序号丢失
> - 增加等待窗口（约1500ms），超过窗口才认定为真正丢失
> - 避免误报导致的不必要重传，减少带宽浪费
>
> 效果：NACK命中率从50%提升到90%，同时减少了不必要的重传。"

#### 位图结构

> "每个 `ts_cache` 在创建时就知道这个 TS 文件会被切成多少个 chunk（`m_exp_fragment_total`），可以预分配固定大小的 bitmap。具体用 `std::vector<std::atomic<uint64_t>>`，大小是 `ceil(chunk_count / 64)`，每个 bit 对应一个 chunk 是否已收到。不用 `std::bitset` 是因为它的大小必须在编译期确定，而 chunk 数量在运行时才知道。"
>
> "用位图怎么快速找到哪些 chunk 没收到？把 bitmap 每个 word 按位取反，取反后值为 1 的位就是未收到的 chunk。利用 `__builtin_ctzll`（Count Trailing Zeros）一次找到 word 里最低的 1 位，处理完后用 `word &= word - 1` 清掉这一位，继续找下一个。每个 64-bit word 最多循环它实际缺失 chunk 数次，远比遍历整个已收 chunk 的 `unordered_map` 再做差集高效。"

---

### 5.3 内存池设计

#### 为什么需要内存池

> "P2P场景中需要频繁分配和释放内存：
> - DataChunkExt对象（每个视频分片）
> - 网络包buffer（收发数据）
> - candidate对象（ICE候选地址）
>
> 直接用malloc/free的问题：
> 1. 频繁系统调用，性能差
> 2. 内存碎片严重（大小不一的对象混合分配）
> 3. 多线程环境下锁竞争
>
> 内存池的好处：
> 1. 预分配大块内存，内部分配无需系统调用
> 2. 固定大小对象池化，无碎片
> 3. 释放时归还池子，复用率高"

#### 内存池设计细节

> "我模仿了nginx的arena分配思想，但做了几处改进：
>
> **三层分配结构**：
>
> **第一层——预注册固定大小桶（O(1)）**：
> - 启动时通过register_block_size()注册常用大小
> - 比如DataChunkExt的大小是256字节，就注册一个256字节的桶
> - 分配时直接从桶的free_blocks取，释放时直接归还，O(1)
> - 桶用完了自动扩容（expand_pool），mmap分配1MB大块，切割成固定大小
>
> **第二层——空闲块复用（O(1)）**：
> - 对于未注册的大小，先查leisure_node_manager
> - 之前释放的内存块按size分桶存储在unordered_map中
> - 查找O(1)，复用已释放的内存
>
> **第三层——arena bump pointer分配**：
> - 如果空闲块也没有，从arena中分配
> - arena用mmap分配1MB大块，内部用bump pointer快速分配
> - 每个分配前面放fragment_node记录大小和状态（支持释放）
> - arena满了就新建一个arena
>
> **释放的关键设计**：
> - free_memory(ptr, size)由调用方传入size
> - 预注册桶：直接通过size定位桶，O(1)归还
> - 未注册大小：通过指针偏移找到fragment_node，加入空闲链表
> - 调用方传size的设计省去了反查表的内存和锁开销"

#### 与nginx的核心差异

| 特性 | Nginx | 我的实现 |
|------|-------|---------|
| 个人释放 | ❌ 不支持 | ✅ 支持 |
| 线程安全 | ❌ 单线程 | ✅ mutex保护 |
| 内存来源 | malloc | mmap |
| 固定大小优化 | ❌ 无 | ✅ 预注册桶O(1) |
| 空闲块复用 | ❌ 无 | ✅ leisure_node_manager |

#### thread_local到全局单例的演进

> "最初用thread_local无锁设计：
> - `thread_local` 相当于给每个线程一个独立的 ptmalloc-like arena（零竞争）
> - 但问题是分配和释放必须在同一线程
> - `DataChunkExt` 的生命周期跨线程：在 asio 收包回调里分配，在上层业务回调（可能在另一个线程）里 delete
> - 这等价于 ptmalloc 的 "foreign free"（跨 arena 释放），`thread_local` 方案没有这个机制，轻则内存泄漏，重则数据结构损坏
>
> 改成全局单例+mutex后跨线程安全，mutex 持锁时间（list push/pop）比 ptmalloc arena 锁持有时间（bin扫描+合并+topchunk检查）短很多。"

#### 与ptmalloc对比

| ptmalloc 问题 | 本项目解决方式 |
|---|---|
| arena 锁竞争（多线程分配时串行） | 预注册 `MemoryPool` 桶，热路径只操作 `list<void*>`，持锁时间极短 |
| unsorted bin 整理开销 | `free_memory(ptr, size)` 调用方传 size 直接定位桶，无需读 chunk header，O(1) 直接归还 |
| 碎片（反复切割/合并导致碎片） | mmap 1MB 大块用 bump pointer 线性分配，固定大小块复用时直接从 free_list 取，**不切割不合并** |
| chunk header 内存开销（每块前8字节存元数据） | 预注册桶的块没有 header，调用方已知 size，节省每块 8B 的元数据开销 |

---

### 5.4 P2P订阅发布模块

#### 订阅发布流程

> "节点有两种角色：
> - **publisher（发布者）**：拥有数据，向下层节点分发
> - **subscriber（订阅者）**：需要数据，从上层节点拉取
>
> 流程：
> 1. 节点启动后，向调度服务器注册（WebSocket连接）
> 2. 调度服务器分配父节点，下发调度指令
> 3. subscriber与publisher进行ICE打洞
> 4. 打洞成功后，建立P2P数据通道（UDP）
> 5. publisher开始推送数据给subscriber
> 6. subscriber收到数据后：
>    - 放入缓存模块
>    - 检查是否有缺失chunk，发起NACK重传
>    - 更新QOS统计（丢包率、RTT、带宽）
> 7. 定期发送心跳包维持链路（5秒超时）
>
> 信令消息类型：
> - HEARTBEAT：心跳保活
> - GENERAL_CMD：订阅/取消/转发/调度/状态通知
> - RAW_DATA：视频数据
> - NACK：丢包重传请求
> - DISCONNECT：断开连接"

#### NACK重传

> "接收端检测到 chunk 序号有缺口时，向发送端请求补发。发送端在 `ts_cache_manager` 里缓存最近的 TS chunk，收到 NACK 后从缓存里查找对应 chunk 重传。如果发送端缓存里已经没有（超过缓存窗口），接收端会从 CDN 补偿拉取。"

#### P2P链路断了怎么办

> "有3层兜底机制：
> 1. **NACK重传**：偶尔丢包，通过NACK请求重传，不影响播放
> 2. **重新打洞**：链路完全断开，向调度服务器请求重新分配父节点，重新打洞
> 3. **CDN回退**：P2P完全不可用时，回退到CDN拉流，保证播放不中断"

---

### 5.5 整流方案

#### 需求背景

> 直播场景下，用户播放 HLS 流时完全依赖 CDN 拉流，CDN 带宽成本随并发用户数线性增长。目标是让用户节点之间通过 P2P 互相分担流量。

#### 整流架构

```
CDN ──完整流──→ 一层节点 ──完整流──→ 二层节点 ──完整流──→ SDK

上行瓶颈分析：
- 一层节点上行带宽 = B Mbps
- 一路流码率 = R Mbps
- 一层节点最多服务 ⌊B/R⌋ 个二层节点
- 假设B=20Mbps, R=2Mbps → 最多10个二层节点

放大率 = 总SDK数 / CDN流量
- 考虑打洞成功率(80%)和掉线 → 实际约3x
```

---

### 5.6 多子流方案

#### 需求背景

> 整流方案中每个节点都必须拉取并缓存**完整的一路流**，当码率较高（如 4Mbps+）时，Level1 节点的上行带宽成为瓶颈，无法同时服务多个下层节点。

#### 解决思路

> 将一路完整的 TS 流按 chunk 序号取模切割为 N 路子流（subBase=N），每个节点只负责其中一路（subId），下层节点同时订阅 N 个上层节点，合并还原完整流。

#### 与整流的差异

| 维度 | 整流 | 多子流 |
|---|---|---|
| 每节点缓存 | 完整流 100% | 1/N 子流 |
| 上行带宽需求 | 全码率 | 码率/N |
| 下层订阅数 | 1个父节点 | N个父节点（各持一路子流） |
| 放大率 | ~3x（现网） | ~8x（mock测试） |
| 容错性 | 单点故障即中断 | 缺失子流走NACK/CDN补偿 |
| 实现复杂度 | 低 | 高 |

#### 子流切分机制

**chunk ID 编码**（`ts_cache_manager.h`）：
```
chunk_id (64bit):
┌──────────────────┬──────────────────┬──────────────────┐
│   tsId (32bit)   │ chunkCount(16bit)│  chunkIndex(16bit)│
└──────────────────┴──────────────────┴──────────────────┘
```

**子流归属判定**：
```cpp
// 节点只缓存 chunkIndex % subBase == subId 的 chunk
uint16_t get_chunk_count(uint64_t chunk_id) {
    uint16_t sum = (chunk_id >> 16) & 0xFFFF;
    return sum / sub_base + (sub_id < sum % sub_base ? 1 : 0);
}
```

#### 多子流下的NACK挑战

> 整流只需向一个父节点发 NACK；多子流需要知道缺失的 chunk 属于哪一路子流，再向对应的父节点发重传请求。
>
> `ts_cache_manager::get_ts_lost_seqs_list()` 返回缺失的 chunk_id 列表，订阅端根据 `chunk_id % subBase` 确定该向哪个父节点发 NACK。

---

### 5.7 放大率优化

#### 整流3倍和多子流8倍是怎么来的

> **放大率定义**：1份CDN流量能服务多少个终端用户
>
> **整流方案**（3倍）：
> - 每个节点缓存完整流，一个节点只能服务有限个下层节点
> - 一层节点上行带宽有限（通常10-20Mbps），一路流2Mbps
> - 一个一层节点最多服务5-10个二层节点
> - 考虑打洞成功率（80%）和节点掉线率，实际放大率约3倍
>
> **多子流方案**（8倍）：
> - 1路完整流切分为8个子流
> - 每个节点只负责1/8的子流（上行带宽降为1/8）
> - 同样的上行带宽，一个节点可以服务8倍的用户
> - mock测试数据：8倍放大率
>
> **关键区别**：
> - 整流：每个节点传完整流 → 上行瓶颈 → 放大率低
> - 多子流：每个节点只传1/8 → 上行降8倍 → 放大率高

#### 多子流的容错

> "缺失某个子流时，先NACK请求重传；重传失败，回源CDN获取缺失的子流。不会因为一个节点故障导致整条链路中断。"

---

## 六、实际运转流程（整流）

```
1. 初始化
   android_init(json) → p2p_manager::reinit()
   → STUN 探测（nat_detect）获取 NatInfo
   → socket_manager 创建 UDP socket

2. 入网
   start() → ReportManager::notifyByAuth()
   → 调度服务器鉴权 → 下发 NodeInfo（groupId、peers 列表、hbInterval 等）

3. 建立 P2P 连接
   on_node_action(JOIN) → subscriber::add_node_info()
   → hole_puncher（MASTER 角色）发送 STUN Binding Request
   → 对端（SLAVE）回 Binding Response → 打洞成功
   → 发送 SUBSCRIBE 命令

4. 数据流转
   上层节点 publisher::on_pc_raw_data()
   → ts_cache_manager::add_chunk()    # 缓存入库
   → 下层 subscriber::on_subscribe_data()  # 推给订阅者
   → DataSession::rxData()            # 交给消费者
   → DataConsumer（生产者-消费者队列）  # 播放器取流

5. 异常处理
   chunk 缺失 → subscriber::add_lost_seq() → NACK 重传
   心跳超时 → peer_connection 断开 → subscriber 重新选父节点
   父节点全断 → 回退 CDN 拉流（http_client）

6. 退网
   stop() → publisher/subscriber close() → keep_alive stop()
   → ReportManager 上报最终 QOS 数据
```

---

## 七、心跳模块

#### 为什么用UDP心跳而不是TCP

> "P2P 通信走 UDP，心跳也在同一 UDP socket 上复用，减少资源占用。同时 UDP 心跳无重连开销，能快速感知链路断开。"

#### 实现细节

> "间隔由服务器配置（`hbInterval`），默认 1000ms，获取到IP前每 500ms 发一次（`get_ip_interval`）。PONG 响应中若 `code == KA_PONG_CODE_GROUP_NOT_FOUND(1)`，说明集群已解散，触发退网逻辑。心跳报文携带 MD5 签名（`sign()` 函数），防止伪造。"

---

## 八、内存池（两个项目统一设计）

`edgelive` 和 `hls-live-plugin` 均采用相同的内存池实现，通过 `#define OPEN_MEMORY_MANAGER 1` 编译期开关控制是否启用。

**两项目内存池调用路径**（`data_chunk.cpp`）：
```cpp
// 构造：buffer 走内存池
#if OPEN_MEMORY_MANAGER
    buffer = (uint8_t*)memory_manager::get_instance()->allot_memory(CHUNK_PRE_ALLOC_SIZE);
#else
    buffer = (uint8_t*)malloc(CHUNK_PRE_ALLOC_SIZE);
#endif

// operator new：DataChunkExt 对象本身走内存池
#if OPEN_MEMORY_MANAGER
    return memory_manager::get_instance()->allot_memory(size);
#else
    void* p = malloc(size);
    if (!p) throw std::bad_alloc();
    return p;
#endif

// operator delete：传入 size，直接定位桶归还，无需反查表
#if OPEN_MEMORY_MANAGER
    memory_manager::get_instance()->free_memory(p, sizeof(DataChunkExt));
#else
    free(p);
#endif
```

---

## 九、QoS统计体系

### 统计维度全景

**连接质量层（peer_connection / subscriber）**

| 指标 | 含义 | 采集方式 |
|---|---|---|
| `lost_rate` | 下行丢包率（%） | 每5秒采样一次，`StatisticsVector` 滑动平均，窗口保留最近1000条 |
| `rtt` | 往返延迟（ms） | 心跳包携带 `timestamp / LSR / DLSR` 三元组，类 RTCP RR 机制计算 |
| `nack_cnt` | NACK重传请求次数 | 首次超时50ms触发，后续100/200ms指数退避 |

**卡顿层（lag）**

| 指标 | 含义 |
|---|---|
| `lag.cnt` | 卡顿次数，阈值200ms无数据算一次 |
| `lag.duration` | 累计卡顿时长（ms） |
| `queueEmptyCount` | 数据队列空次数 |
| `queueEmptyRatio` | 队列空占播放时长的比例 |

**打洞层（hole_punch）**

| 指标 | 含义 |
|---|---|
| `upper_hole_punch_success_rate` | 向上层节点打洞成功率 |
| `down_hole_punch_success_rate` | 向下层节点打洞成功率 |

**订阅切换层（subscription_switch）**

| 原因 | 触发场景 |
|---|---|
| `by_father_back_off` | 父节点主动退避（资源不足） |
| `by_stream_suspend` | 父节点流中断 |
| `by_father_obsolete` | 父节点被云端淘汰 |
| `by_schedule` | 云端主动调度优化 |
| `by_active` | 本端主动切换（丢包率高/RTT高） |
| `by_father_leave` | 父节点正常离开 |

**流量/数据层（SDKQos / BWDist）**

| 分类 | 指标 |
|---|---|
| 字节数 | `byteTotalCDN` / `byteTotalP2P`（CDN/P2P各自来源） |
| Miss细分 | `byteCDNMiss` / `byteP2PMiss`，其中各自再分 `Expired`（过期miss）和 `Repeat`（重复miss） |
| TS命中率 | `ts_hit_rate`：NACK命中缓存的比例，滑动平均 |

**CDN补偿层**

| 指标 | 含义 |
|---|---|
| `compensateCDNCount` | 补片请求次数 |
| `compensateCDNByte` | 补片总字节数 |
| `compensateOver300Count` | 补片耗时超过300ms的次数 |

---

## 十、面试问答准备

### 第一轮：项目全貌

**Q：先介绍一下这个项目解决了什么问题？**

> CDN 直播是树形拓扑，所有用户都从 CDN 源站拉流，并发量大时带宽成本是瓶颈。HCDN 的思路是让终端节点互相分享带宽（P2P），上层节点推流给下层节点，CDN 只需服务少量根节点，从而降低 CDN 带宽成本、提升放大率（1份CDN流量→N份用户侧流量）。

**Q：你负责哪些部分，核心贡献是什么？**

> 负责从零预研P2P打洞方案（ICE/STUN），主导核心模块（缓存/P2P订阅发布/内存池）的设计和实现，以及项目上线。代码层面的关键产出是：NACK查找命中率从50%提升到90%、实现内存池将高频小对象分配从堆分配改为池化、多子流放大率达到8x。

---

### 第二轮：P2P 打洞

**Q：为什么不直接用 WebRTC 库，而是自己实现一套？**

> WebRTC 库是为浏览器音视频通话设计的，直接用有几个问题：① DTLS+SRTP 强制加密，内网/可信节点之间不需要，但会带来额外 CPU 开销；② SDP 协商格式冗余，对直播信令来说过重；③ 库体积大，在路由器/嵌入式设备上不友好。因此策略是选取 WebRTC 的协议思路（ICE 框架、STUN 探测、NACK 重传），用 C++ 自己实现一个裁剪版，并针对直播场景做适配。

**Q：P2P 的核心难点是什么？为什么不直接让两个节点互发 UDP？**

> 绝大多数设备在 NAT 后面，没有公网 IP。直接发 UDP 对方收不到，因为 NAT 设备会丢弃未经映射的入站包。必须先用打洞技术让 NAT 在双方各自打开一个"通道"，这就是 ICE 解决的问题。

**Q：你们为什么选 ICE 而不是自己写 UDP 打洞？**

> 自己写只能处理 Full Cone 和 Address Restricted NAT，碰到 Symmetric NAT 就无能为力。ICE 框架统一了三种候选者类型——host（直连）、srflx（STUN映射）、relay（TURN中继），能根据 NAT 类型自动选最优路径，有 RFC 5245 规范可参考。

**Q：Symmetric NAT 碰到了怎么处理？**

> 代码里通过 `HOST_SYMMETRIC_CANDIDATE` 单独标记这类节点，打洞失败后上报 `DOWN_P2P_CONNECTION_FAILURE`，由调度服务器换一个 NAT 类型更友好的上层节点，或者这个节点退避到直接从 CDN 拉流。

**Q：为什么用 UDP 而不是 TCP 传 P2P 数据？**

> 直播对延迟敏感，TCP 的可靠传输通过重传+拥塞控制保证，遇到丢包会引入队头阻塞。UDP 允许我们自己控制重传策略：只对关键 chunk 发 NACK 重传，超时后直接从 CDN 补偿而不是死等，把延迟控制在可接受范围内。

---

### 第三轮：缓存模块

**Q：P2P 网络丢包是常态，你们怎么解决？**

> 用 NACK（Negative Acknowledgement）机制。接收端检测到 chunk 序号有缺口时，向发送端请求补发。发送端在 `ts_cache_manager` 里缓存最近的 TS chunk，收到 NACK 后从缓存里查找对应 chunk 重传。如果发送端缓存里已经没有（超过缓存窗口），接收端会从 CDN 补偿拉取。

**Q：NACK 要快速查找 chunk，缓存结构是怎么设计的？**

> 用 `unordered_map<chunk_id, DataChunkExt*>` 存已缓存的 chunk，O(1) 查找。没用数组是因为 chunk_id 不连续（多子流按 subId 分流后有跳跃），数组会浪费大量空间。缓存窗口只保留最近 3 个 TS 分片（几秒内），内存占用可控，超出窗口的 chunk 淘汰。

**Q：简历写命中率从50%提升到90%，原来哪里有问题？**

> 原实现判断"chunk 丢失"的逻辑有缺陷：用估算的 chunk 总数作为范围来找缺口，会把"还没到达但正在路上"的 chunk 也当成丢失，触发大量误报 NACK。优化后改为：只在已经收到后续 chunk 的情况下才认定中间序号丢失（等待窗口约 1500ms），避免乱序包被误判。

**Q：生产者消费者在这里解决什么问题？**

> P2P 收包在 asio 的 io_context 线程（网络线程），播放器取数据在解码线程，两个线程速率不匹配。用 `DataConsumer` 的 mutex + condvar 实现线程安全队列：网络线程 push 数据后 notify，解码线程 wait 被唤醒后 pop，完全解耦两端的速率差异。

---

### 第四轮：内存池

**Q：你为什么要做内存池，直接 malloc 不行吗？**

> 需要先理解 ptmalloc（glibc 默认 malloc）的运转逻辑才能回答这个问题。ptmalloc 在多线程下有三个关键结构：
> - **main arena**：进程唯一，多线程竞争时用 mutex 串行化
> - **thread arena**：每个线程最多创建一个，用 mmap 分配，通过 arena 锁减少竞争
> - **bins**：按大小分类的空闲链表（fastbin / smallbin / largebin / unsorted bin）
>
> 在 P2P 收包热路径上，`DataChunkExt` 和 `buffer`（1536B）每个 chunk 各创建一次、各释放一次，频率极高。ptmalloc 的问题有两个：
> 1. **锁竞争**：多核高并发下 arena 数量不足时仍然串行，分配延迟不稳定
> 2. **碎片**：1536B 属于 smallbin 范围，反复切割/合并，长时间运行后堆碎片严重

**Q：你们内存池和 ptmalloc 相比，具体解决了哪几个问题？**

> | ptmalloc 问题 | 本项目解决方式 |
> |---|---|
> | arena 锁竞争 | 预注册 `MemoryPool` 桶，热路径只操作 `list<void*>`，持锁时间极短 |
> | unsorted bin 整理开销 | `free_memory(ptr, size)` 调用方传 size 直接定位桶，无需读 chunk header，O(1) 直接归还 |
> | 碎片 | mmap 1MB 大块用 bump pointer 线性分配，固定大小块复用时直接从 free_list 取，不切割不合并 |
> | chunk header 内存开销 | 预注册桶的块没有 header，节省每块 16B 的元数据开销 |

**Q：ptmalloc 的 fastbin 不也是按大小分桶、不合并的吗？和你们的区别是什么？**

> fastbin 确实是最接近本项目设计的结构——按大小分桶的单链表，释放后不立即合并。但有几点区别：
> 1. fastbin 只覆盖小于 160B 的块，1536B 的 buffer 不在 fastbin 范围
> 2. fastbin 用 LIFO 单链表，每个节点前面有 8B chunk header，而本项目的 `MemoryPool` 桶用 `list<void*>` 直接存用户指针，零 header 开销
> 3. fastbin 的分配仍在 ptmalloc 的 arena 锁保护下，本项目的 mutex 粒度更细

**Q：为什么 free_memory 要调用方传 size，而不是像 ptmalloc 一样在块头存元数据？**

> ptmalloc 在每块用户内存前面存 `malloc_chunk` 结构（`prev_size` + `size` + 标志位，共 16B），`free()` 时从 `ptr - 16` 读出大小。本项目的使用场景是编译期已知 size 的固定对象，调用方持有 size 是零成本的：
> 1. 节省每块 16B 内存（1000 个 buffer 就省 16KB）
> 2. 释放路径消除一次 cache miss（不需要读 header 所在的前一个 cache line）
> 3. 强迫调用方用 `free_memory` 接口有防御性设计的意味

---

### 第五轮：放大率优化

**Q：放大率具体是怎么定义和衡量的？**

> 放大率 = 所有节点上行总带宽 / CDN 下行总带宽。理想情况是 N 个节点形成一棵树，每份 CDN 流量被逐级放大 N 倍。通过 QOS 上报收集每个节点的上行/下行字节数，服务端聚合计算。

**Q：整流只有 3x，多子流能到 8x，原因是什么？**

> 整流瓶颈在单节点上行带宽：每个节点要转发完整码率的流（如 2Mbps），实际家庭宽带上行普遍只有 6-8Mbps。多子流把完整流拆成 N 份子流，每个节点只转发 1/N 的码率，上行压力降为 1/N，理论上放大率是整流的 N 倍。

**Q：多子流场景下某个子流节点挂了怎么办？**

> `subscriber` 检测到 `on_pc_disconnect()` 或心跳超时后，立即触发 `try_subscribe_if_need()` 重新选该子流的父节点。期间丢失的 chunk 通过 NACK 触发向其他子流节点重传，或超时后从 CDN 补偿拉取。

---

### 第六轮：系统健壮性

**Q：心跳用 UDP 发，UDP 不可靠，丢了怎么办？**

> 单次心跳丢包不会立即断链，设计了超时阈值（连续多次未收到才认定断链）。心跳包本身很小，丢包概率远低于数据包。丢失后触发重连逻辑，上报给调度服务器更新节点状态。

**Q：调度服务器不可用时，已有 P2P 链路还能工作吗？**

> 能，短期内可以。`peer_connection` 有独立心跳维持链路，数据收发不依赖调度服务器。调度服务器只负责信令（打洞+节点分配），断开后无法新建 P2P 连接或重新调度，已有连接继续工作。

---

### 第七轮：缓存模块深入（链表+位图+生产者消费者+多线程写入）

**Q：为什么用链表存 TS，用哈希表不行吗？**

> 单纯哈希表能 O(1) 查找，但没有顺序信息——不知道哪个 TS 最旧、应该淘汰谁。淘汰策略是 FIFO：只保留最新 3 个 TS 分片，超出后删最旧的。链表天然维护插入顺序，从链表头 pop 是 O(1) 淘汰最旧的，尾部 push 是 O(1) 插入最新的。同时保留一个 `unordered_map<tsId, ts_cache*>` 做哈希索引：链表管顺序，哈希管查找，全部 O(1)。

**Q：多个 consumer 同时挂载时怎么保证每个 consumer 都能收到全量数据？**

> 每个 `DataConsumer` 实例有自己独立的 `_flvQueue`，生产者调 `DataSession::addConsumer()` 注册后，每次 `onReceivedFLVTag()` 遍历所有注册的 consumer 逐个调用，相当于广播。Queue 里存的是 `SharedPtr<FLVTag>`，数据本身引用计数共享，不会复制 FLV 数据体。

**Q：多线程写入的场景具体是什么？**

> 多子流场景。把一路 2Mbps 的直播流按 chunk 序号取模分成 N 份子流（比如 N=3），三个 subscriber 各自跑在网络线程里，同时向同一个 `ts_cache` 写入不同序号的 chunk。三个线程并发写同一个 `ts_cache`，是真实存在的竞态。

**Q：bitmap 的 `fetch_or` 为什么能保证多线程安全？**

> `std::atomic<uint64_t>::fetch_or` 是不可分割的原子读改写操作，硬件层面对应 LOCK OR 指令（x86）或 LDSET（ARM），由 CPU 保证原子性。三个线程分别写 bit 0、bit 1、bit 2，`fetch_or` 保证不会出现"读-改-写"的中间状态。另外 `fetch_or` 返回旧值，通过判断旧值该 bit 是否已经是 1，可以检测重复写，直接丢弃重复数据。

---

## 十一、高频面试问题

### Q1: "你在项目中遇到的最大困难是什么？"

> "NACK命中率只有50%的问题。一开始以为是网络丢包严重，后来分析发现是我们的丢包判定逻辑有bug——用估算的chunk总数作为范围找缺口，误报了大量'还在路上'的chunk为丢失。修正了判定逻辑，增加等待窗口后，命中率从50%提升到90%。"

### Q2: "你做过的最有技术含量的优化是什么？"

> "多子流方案的设计。从整流到多子流，核心思路是将1路完整流切分为N个子流，每个节点只负责1/N的数据。这样同样的上行带宽可以服务N倍的用户。但实现复杂度很高，需要协调多路订阅的合并、子流缺失时的NACK/CDN补偿、子流同步等问题。"

### Q3: "如果让你重新设计，你会怎么改？"

> "1. 内存池改用sharded锁或per-thread cache，减少mutex竞争
> 2. 增加P2P链路的带宽自适应，根据网络状况动态调整子流数量
> 3. 增加节点动态调度，根据QOS数据实时调整树形拓扑
> 4. 增加端到端加密（当前内网场景不需要，但未来可能需要）"

### Q4: "TCP和UDP在P2P中怎么选？"

> "P2P数据传输用UDP，原因：
> 1. UDP可以打洞，TCP打洞难度大
> 2. 实时视频对延迟敏感，UDP没有重传和拥塞控制
> 3. 我们自己实现了NACK重传，按需重传比TCP的全量重传更高效
>
> 信令通道用TCP（WebSocket），原因：信令不能丢失，TCP保证可靠传输；信令量小，TCP的开销可以接受。"

### Q5: "你的内存池和glibc的ptmalloc有什么区别？"

> "ptmalloc是通用内存分配器，我的内存池是场景专用：
>
> | 特性 | ptmalloc | 我的内存池 |
> |------|----------|-----------|
> | 分配策略 | 按大小分bin | 预注册固定大小桶 |
> | 释放策略 | 合并相邻chunk，放入unsorted bin | 直接归还桶（O(1)） |
> | 碎片处理 | 合并+分箱 | 固定大小无碎片 |
> | 线程安全 | per-thread arena | 全局mutex |
> | 内存来源 | brk/mmap | mmap |
>
> 核心区别：我的内存池用'调用方传size'的设计，省去了ptmalloc在chunk header里存储大小的开销，释放路径纯O(1)。"

---

## 十二、项目亮点总结

| 序号 | 亮点 | 一句话描述 |
|------|------|-----------|
| 1 | 自研裁剪版ICE协议栈 | 从WebRTC提取协议思路，针对直播场景裁剪加密/SDP/SCTP |
| 2 | 多子流方案 | 放大率从3倍提升到8倍，上行带宽降为1/N |
| 3 | 内存池优化 | 高频小对象池化，释放O(1)，支持个人释放 |
| 4 | NACK命中率50%→90% | 修正丢块判定逻辑，减少误报 |
| 5 | CDN回退兜底 | P2P失败时无缝降级到CDN拉流 |
| 6 | 跨平台边缘节点 | Android/Linux/路由器，PluginBus插件化 |

---

## 十三、技术桥梁：从HCDN到视频行为分析

| 我的项目经验 | JD要求 | 如何桥接 |
|-------------|--------|----------|
| P2P视频传输 | 视频数据处理链路 | "我做过视频流P2P传输，理解视频从CDN拉流到终端播放的全链路" |
| 自研ICE/STUN | 网络通信基础 | "我从WebRTC提取协议思路自研了ICE协议栈" |
| HLS/FLV处理 | 视频数据处理 | "我处理过HLS和FLV格式，理解封装格式和流媒体协议" |
| 边缘节点部署 | 端侧AI部署 | "我做过边缘计算节点部署，理解资源受限环境下的优化" |
| 内存池优化 | 性能调优 | "我设计实现过内存池，高频小对象池化，释放O(1)" |
| NACK优化50%→90% | 系统优化能力 | "我定位并修复了NACK误判问题，命中率提升到90%" |
