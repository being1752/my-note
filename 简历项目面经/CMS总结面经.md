# 长沙市岳麓区老年监护系统（CMS）— 完整面试准备

> Console Management System — 长沙飞郁网络 × 岳麓区街道办驻场项目
>
> **项目性质**: 实习阶段参与的生产级项目，真实运行 2 年以上
>
> **一句话简介**: 为长沙岳麓区街道办开发的老年监护系统，通过摄像头监控独居老人安全，支持跨局域网管理多路摄像头设备。

---

## 一、项目背景（Project Background）

**标准回答模板：**

该项目源自长沙飞郁网络公司与岳麓区街道办的驻场合作，业务场景是老年人社区监护。街道办需要一套**跨局域网**的音视频中控台系统，用于统一管理多个摄像头（IPC 设备），支持实时查看、录制、云台控制、推送报警等功能。整体架构是**一个服务端 + 多个客户端**的 C/S 模型，客户端运行在街道办工作站上。我负责中控台客户端和部分服务端业务逻辑，以及后期的代码重构。

**加分点：**
- 强调是**生产级项目**，不是课程设计，真实运行 2 年以上
- 主动提出：这个项目让我系统性地接触了**流媒体协议栈**从协议层到应用层的完整链路

**系统架构：**
- **服务端**：管理设备注册、报警推送、用户权限
- **中控台客户端**（我负责）：MFC开发，视频监控、回放、PTZ控制、设备管理
- **IPC设备**：网络摄像头，通过RTSP/RTP推送视频流

---

## 二、需求痛点（Pain Points）

| 痛点 | 具体描述 |
|------|----------|
| **跨局域网穿透** | 摄像头和客户端不在同一 LAN，公网直连受 NAT 限制 |
| **多路视频并发** | 需同时接收最多 16 路 H264 视频流，资源调度复杂 |
| **音视频同步** | G711/G726 音频流与 H264 视频流需要对齐，延迟控制 |
| **设备异构兼容** | 不同厂商 IPC 设备使用 ONVIF/RTSP 标准不一致 |
| **推送实时性** | 老人跌倒/异常推送需低延迟送达，不能丢失 |
| **录像策略管理** | 多通道录像文件管理、分片存储、远程回放 |

**面试表达建议：**

最大的技术痛点有两个：一是**跨 NAT 的 P2P 打洞问题**——摄像头部署在社区局域网内，客户端在街道办，无法直接连接；二是**16 路视频并发的线程/资源调度**——每路视频都要独立解码、渲染、录制，朴素的每路一线程方案线程数爆炸，需要线程池统一调度。

---

## 三、核心模块详解

### 3.1 流媒体播放（最核心的模块）

#### 视频播放的完整流程

> "从IPC到画面显示，经过5步：
>
> 1. **RTSP信令**：客户端向IPC发送DESCRIBE/SETUP/PLAY请求
>    - DESCRIBE：获取SDP信息（编码格式、分辨率、码率）
>    - SETUP：建立RTP/RTCP通道，协商传输参数
>    - PLAY：开始推流
>
> 2. **RTP接收**：Live555的testRTSPClient接收RTP包
>    - 支持UDP和TCP两种传输模式
>    - TCP模式用RTP over RTSP interleaved（RTP包嵌入RTSP连接）
>
> 3. **H.264解封装**：从RTP包中提取H.264 NALU
>    - 处理FU-A分片（一个NALU分多个RTP包）
>    - 处理STAP-A聚合（多个NALU在一个RTP包）
>    - 识别SPS/PPS/I帧/P帧
>
> 4. **FFmpeg解码**：用libavcodec解码H.264 NALU
>    - avcodec_find_decoder(AV_CODEC_ID_H264)
>    - avcodec_decode_video2() → 输出YUV420P
>    - sws_scale() → 转换为RGB24或直接用YUV渲染
>
> 5. **DirectDraw渲染**：用DirectX的DirectDraw将YUV/RGB数据绘制到窗口
>    - 创建Offscreen Surface
>    - Lock/Unlock更新像素数据
>    - Blt到Primary Surface显示"

#### 多路视频同时播放

> "最多16路同时播放，每路视频独立线程：
>
> 线程模型：
> - **主线程**：MFC消息循环，UI操作
> - **每路视频**：1个RTSP接收线程 + 1个解码线程
> - **渲染**：DirectDraw绘制在MFC窗口的指定区域
>
> 关键实现：
> - **COneIPCChannel类**：封装单路视频的完整生命周期（接收→解码→渲染→存储）
> - **CRemotePlayDlg**：管理多路视频窗口布局（1/4/9/16宫格）
> - **解码器复用**：每路视频独立的解码器上下文（AVCodecContext），不能跨路复用
> - **资源控制**：16路同时播放时CPU占用高，动态调整弱网路视频的解码分辨率
>
> 线程同步：
> - 使用CRITICAL_SECTION保护共享数据
> - 环形缓冲区作为接收线程和解码线程之间的数据通道
> - 生产者（接收）写满后等待，消费者（解码）取空后等待"

#### RTSP over TCP和UDP切换

> "优先使用UDP（低延迟），如果丢包严重自动切换TCP：
>
> 判断条件：
> - 连续N个RTP包序号不连续 → 丢包
> - RTCP反馈的丢包率超过阈值
>
> 切换流程：
> 1. 发送RTSP TEARDOWN断开当前会话
> 2. 重新发送DESCRIBE/SETUP，SETUP中指定interleaved模式
> 3. RTP包通过RTSP TCP连接传输（channel ID区分RTP和RTCP）
>
> TCP模式的缺点：延迟略高（TCP重传），但保证可靠性。
>
> 代码体现：COneIPCChannel中根据网络状况动态选择transport模式。"

---

### 3.2 H.264解码

#### H.264的NALU结构

> "NALU（Network Abstraction Layer Unit）是H.264的网络传输单元：
>
> NALU结构：
> - **Start Code**：0x00 0x00 0x00 0x01（4字节）或 0x00 0x00 0x01（3字节）
> - **NALU Header**：1字节
>   - forbidden_zero_bit(1bit)：必须为0
>   - nal_ref_idc(2bit)：重要性（SPS/PPS/I帧=3，P帧=2，B帧=0）
>   - nal_unit_type(5bit)：类型
>     - 1：非IDR切片（P/B帧）
>     - 5：IDR切片（I帧，关键帧）
>     - 6：SEI（补充信息）
>     - 7：SPS（序列参数集）
>     - 8：PPS（图像参数集）
>
> 在我的项目中：
> - Live555从RTP包中提取NALU（处理FU-A分片和STAP-A聚合）
> - FFmpeg的avcodec_decode_video2接收NALU，内部解析SPS/PPS后解码
> - NaluStore.cpp将NALU按帧存储，用于录像回放
>
> 实际开发中遇到的问题：
> - 播放开始时如果缓存不够，不等到I帧就开始解码会导致花屏
> - 解决：检测到I帧才开始解码，之前的P帧丢弃"

#### FFmpeg解码流程

```cpp
// 1. 初始化
AVCodec* codec = avcodec_find_decoder(AV_CODEC_ID_H264);
AVCodecContext* ctx = avcodec_alloc_context3(codec);
avcodec_open2(ctx, codec, NULL);

// 2. 解码每一帧
AVFrame* frame = av_frame_alloc();
AVPacket pkt;
av_init_packet(&pkt);
pkt.data = nalu_data;     // H.264 NALU数据
pkt.size = nalu_size;
int got_picture = 0;
avcodec_decode_video2(ctx, frame, &got_picture, &pkt);

// 3. 如果解码成功，转换像素格式
if (got_picture) {
    SwsContext* sws = sws_getContext(
        ctx->width, ctx->height, ctx->pix_fmt,  // 源：YUV420P
        ctx->width, ctx->height, AV_PIX_FMT_RGB24,  // 目标：RGB24
        SWS_BILINEAR, NULL, NULL, NULL);
    sws_scale(sws, frame->data, frame->linesize, 0,
              ctx->height, rgb_frame->data, rgb_frame->linesize);
}

// 4. 渲染
// DirectDraw Lock → memcpy RGB数据 → Unlock → Blt
```

#### FFmpeg解码内部流程（CH264Decoder::DecodeVideo）

```
av_init_packet(&pkt);
pkt.data = (uint8_t*)NaluBuffer;
pkt.size = NaluBufferSize;
avcodec_decode_video2(m_avctx, m_frame, &got_picture, &pkt);
// m_frame 输出 YUV420P 格式
// 用 swscale 转 BGR24 供 DirectDraw 绘制
sws_scale(swsCtx, m_frame->data, m_frame->linesize,
          0, height, dstFrame->data, dstFrame->linesize);
```

> YUV420P → RGB 转换公式：`R = Y + 1.402*(V-128)`，`G = Y - 0.344*(U-128) - 0.714*(V-128)`，`B = Y + 1.772*(U-128)`，swscale 内部用 SIMD 指令优化，速度远快于纯 C 实现。

---

### 3.3 P2P远程穿透

#### P2P是怎么实现的

> "CMS系统中，客户端需要直接访问IPC摄像头，但IPC在局域网内，没有公网IP。
>
> 我使用第三方IOTC SDK（P2PTunnel）实现NAT穿透：
>
> 流程：
> 1. IPC设备启动时向IOTC服务器注册（设备UID）
> 2. 客户端通过设备UID向IOTC服务器请求连接
> 3. IOTC服务器协调双方进行NAT穿透
> 4. 穿透成功后，建立P2P数据通道
> 5. 穿透失败时，通过IOTC服务器relay中继
>
> 代码结构（CP2PClient类）：
> - 初始化：P2PTunnelAgentInitialize()
> - 连接：P2PTunnelAgentConnect(uid)
> - 数据传输：P2PTunnelAgentRead()/Write()
> - 断开：P2PTunnelAgentDisconnect()
>
> 和HCDN项目自研ICE的区别：
> - CMS：用第三方SDK，不需要自己实现打洞逻辑
> - HCDN：自研ICE协议栈，完全控制打洞流程
> - CMS的P2P经验让我理解了NAT穿透的原理，所以HCDN项目中我选择自研而非使用WebRTC库"

#### P2P跨局域网穿透具体实现

集成了 **PPCS/IOTC P2P SDK**（`PPCS_API.h`、`IOTCAPIs.dll`、`P2PTunnelAPIs.dll`），流程是：

1. IPC 设备和客户端都向 P2P 云服务器（`nat.vveye.net:8000`）注册，获取 UID
2. 云服务器做**打洞协调**，交换双方的公网 IP + 端口
3. 打洞成功后建立 UDP 直连隧道，如果 NAT 类型不兼容则中继转发
4. 在隧道上做**端口映射**（`AddPort`），把摄像头的 RTSP 端口（554）映射到本地，让 live555 RTSP 客户端透明接入

**追问：P2P 打洞失败怎么处理？**

降级到服务器中继转发，`libt2u.dll` 是备用的 TCP 转发库，保证在严格 NAT 下也能连通，可用性优先。

#### NAT类型与打洞

> NAT 按限制程度从低到高分四种：
> - **Full Cone NAT**（完全锥型）：只要内网端口映射建立，任何外网地址都能发包进来。打洞最简单。
> - **Address Restricted Cone**（地址限制锥型）：必须先向对方发过包，对方才能回包。打洞需要双方同时互发探测包。
> - **Port Restricted Cone**（端口限制锥型）：限制到端口级别，双方需要更精确地同步探测端口。
> - **Symmetric NAT**（对称型）：每次连接不同目标时分配不同的外网端口，无法预测，UDP 打洞基本失败，只能靠服务器中继。
>
> 本项目中，社区网络大多是运营商的对称型 NAT，所以打洞成功率不高，最终很多连接是靠 `libt2u.dll` 做的 TCP 中继转发，这是 IOTC SDK 自带的降级策略。

#### P2P隧道断开重连

> IOTC SDK 会通过心跳包检测 Session 状态，断开时回调 `SessionStatus` 通知应用层。在 `COneIPCChannel` 里注册了断线回调，触发后：
> 1. 先停止 live555 的 RTSP 流（避免读空数据死循环）
> 2. 重新调用 `connectP2PServer` 发起重连，最多重试 3 次
> 3. 重连成功后重新做端口映射，重启 RTSP 拉流
> 4. UI 层显示"重连中..."状态，重连成功后自动恢复画面

---

### 3.4 设备管理

#### ONVIF设备发现

> "ONVIF是网络摄像头的标准管理协议，用gSOAP生成C++代码。
>
> 发现流程：
> 1. 客户端发送WS-Discovery组播消息（UDP，地址239.255.255.250:3702）
> 2. 局域网内的ONVIF设备回复自己的设备描述
> 3. 客户端解析回复，获取设备IP、类型、服务地址
> 4. 通过ONVIF GetDeviceInformation获取设备详情
>
> 代码体现：
> - discoveryClient.cpp：ONVIF发现客户端
> - onvif_api.cpp：封装ONVIF操作（获取设备信息、配置等）
> - soapClient.cpp：gSOAP生成的SOAP消息处理
>
> 远程设备管理：
> - 通过TCP长连接与服务端通信
> - 消息格式：自定义XML（ClientManager.cpp构建和解析）
> - 操作：添加/删除设备、配置参数、固件升级"

#### 云台控制（PTZ）

> 云台（PTZ）控制指令走 P2P 隧道上的独立端口（`m_nNewCurrentPTZPort`），使用私有二进制协议。指令结构是定长命令头 + 方向/速度参数，例如：
> ```
> [CMD=TILT_UP(4)] [Speed=5] [Duration=500ms]
> ```
> 方向宏定义（来自 `SDKInterface.h`）：`PAN_LEFT=1`、`PAN_RIGHT=3`、`TILT_UP=4`、`TILT_DOWN=2`。客户端按下方向键时发送 `PTZ_START` 命令，松开时发送 `PTZ_STOP`，防止摄像头持续转动。

**延迟优化：**
> 1. PTZ 指令走 UDP 而非 TCP，牺牲可靠性换延迟（偶发丢包可接受）
> 2. 按键重复发送：每 100ms 发一次保持命令
> 3. UI 层做预测性响应：按键按下时立即高亮方向键图标，不等服务器确认

**预置位（Preset）：**
> 预置位存在摄像头固件里。操作流程：
> 1. 转动云台到目标位置，发送 `PTZ_SET_PRESET(index)` 指令，固件记录当前电机坐标
> 2. 调用预置位时发送 `PTZ_GOTO_PRESET(index)`，固件驱动电机转到记录坐标
> 3. 中控台的 `RemotePlayPTZPresetDlg` 列表展示已保存的预置位，支持增删改

---

### 3.5 推送接收与托盘功能

#### 推送通道

> 视频走 P2P 隧道上的 RTSP/RTP，而推送（报警通知）走的是**客户端与业务服务器之间的 TCP 长连接**（`CClientSocket`）。这是两条完全独立的通道，设计上做了职责分离：
> - **数据面**：P2P 隧道传音视频，带宽敏感，走 UDP
> - **控制面**：TCP 长连接传命令和推送，可靠性优先，走 TCP

#### TCP长连接保持

> `CClientSocket` 继承 MFC `CSocket`，重写了 `OnMessagePending` 实现**超时心跳**：每隔 30 秒发一个心跳包（长度为 0 的探测帧），如果连续 3 次无响应则判定断线，触发重连逻辑。重连采用**指数退避**策略（1s → 2s → 4s → 最大 30s），防止服务器重启瞬间大量客户端同时重连造成雪崩。

#### 推送数据解析

> 推送数据是 **XML 格式**，例如：
> ```xml
> <AlarmInfo>
>   <DeviceUID>ABCD1234</DeviceUID>
>   <AlarmType>Motion</AlarmType>
>   <AlarmTime>2024-03-01 14:30:00</AlarmTime>
>   <AlarmLevel>2</AlarmLevel>
> </AlarmInfo>
> ```
> 使用 `libxml2` 解析，先做 Schema 合法性校验，再提取字段。解析失败时记录日志并丢弃该条推送，**不会把字段值直接拼接成 SQL 或命令字符串**，防止注入攻击。

#### 系统托盘实现

> 系统托盘用 `Shell_NotifyIcon` API 实现，传入 `NOTIFYICONDATA` 结构体注册图标。收到推送时：
> 1. 调用 `Shell_NotifyIcon(NIM_MODIFY, ...)` 修改托盘图标为闪烁状态（定时器切换图标）
> 2. 调用 `Shell_NotifyIcon` 的 `szInfo` 字段弹出气泡通知（Balloon Tip）
> 3. 用户点击托盘图标时 `ShowWindow(SW_RESTORE)` 恢复窗口并弹出推送详情对话框

---

### 3.6 音频编解码

#### G711和G726

> - **G711**：最古老的 PCM 编码，分 μ-law（北美）和 A-law（中国/欧洲），采样率 8kHz，码率 64kbps，编解码复杂度极低，嵌入式摄像头几乎都支持，**延迟最低**
> - **G726**：ADPCM 自适应差分编码，码率可选 16/24/32/40kbps，在 G711 基础上压缩了带宽，适合带宽受限场景
>
> 老年监护场景对语音质量要求不高（主要用于对讲确认），优先选延迟低、CPU 占用低的 G711。G726 作为备选方案在带宽受限社区使用。

#### 音视频同步

> 音频使用 G711/G726 编码（`1G711.cpp`、`g726.c`），通过独立的 UDP 通道传输，在 `CAudioRecieveBuffer` 中维护一套环形缓冲队列，解码后调用 `waveplay` 播放（`waveplay.cpp`）。音视频同步靠 **RTP 时间戳对齐**，不是 PTS，因为嵌入式摄像头时钟精度有限。

---

### 3.7 日志系统

> 系统用 `DebugViewBasicPrintf` 宏封装输出，底层走 Windows 的 `OutputDebugString` API，配合 Sysinternals 的 `Dbgview.exe` 实时查看。
>
> **设计原因**：
> 1. 项目依赖已经很重（FFmpeg、live555、P2P SDK），减少额外依赖
> 2. `OutputDebugString` 是内核级别的调试输出，**不需要文件 IO，不影响主线程性能**
> 3. `Dbgview.exe` 支持远程查看，驻场运维时工程师不用登录机器就能看日志
>
> **日志分级**：
> - 编译期：Debug 模式下展开为实际调用，Release 模式编译成空语句，**零运行时开销**
> - 运行期：关键路径（连接建立/断开、解码错误、推送到达）无条件输出；高频路径（每帧解码、每包接收）用采样日志（每 100 次输出一次）
>
> **线上问题排查案例**：某次社区反馈某路视频隔天就黑屏，通过日志发现运行约 24 小时后 NALU 队列积压警告密集出现，随后 `avcodec_decode_video2` 返回 -1。原因是长时间运行后 `CPtrList` 内存碎片化，`new` 操作越来越慢。解决方案：改用**内存池预分配**固定大小的 NALU 缓冲区，彻底消除了内存碎片问题。

---

### 3.8 代码重构

#### 重构前的问题

> 第一版代码是快速原型，暴露出几个严重问题：
> 1. **全局变量满天飞**：多个对话框直接读写同一个设备状态全局结构体，线程不安全，偶发崩溃
> 2. **代码重复严重**：每个 Dialog 各自写一份 XML 构建和解析代码，同一个 bug 要改 8 处
> 3. **耦合度高**：UI 层直接调用 SDK 函数，无法做单元测试，换 SDK 版本要改遍所有 Dialog
> 4. **没有错误传播机制**：底层 SDK 调用失败直接 `return -1`，上层不知道失败原因

#### 重构方案

> 采用**渐进式重构**，不做大爆炸式重写：
> 1. 先加日志，把所有关键路径的输入输出打出来，建立"基线行为"
> 2. 每次只改一个模块，改完后跑完整的功能回归
> 3. 重构顺序：先提取 `SDKInterface` 接口层（最底层），再提取 XML 工具类，最后改 UI 层
> 4. 用版本控制（git）每次重构作为一个独立 commit，出问题可以精准 revert
>
> **分层架构**：
> - 网络层：Socket封装、TCP/UDP通道、RTSP客户端
> - 协议层：XML解析/构建、ONVIF消息处理
> - 业务层：设备管理（IPCDevicesManager）、流媒体控制（OneIPCChannel）
> - UI层：MFC对话框（只负责显示和用户交互）
>
> **设计模式**：
> - 单例模式：全局配置管理器（CClientManager）
> - 工厂模式：创建不同类型的设备连接
> - 观察者模式：设备状态变化通知UI更新

#### 重构成果

> - 代码行数从约 15000 行减少到 11000 行（消除了大量重复代码，减少约 27%）
> - 崩溃率从上线第一年的每周 1～2 次降为第二年几乎为零（全局变量竞态问题根除）
> - 新增功能开发速度明显提升：第三次版本迭代新增"远程回放"功能，只花了 5 天；第一版类似规模功能要花 2～3 周

---

## 四、系统架构与运作流程

### 整体架构

**C/S 模型**：一个业务服务器 + 多个街道办工作站客户端，管理多路社区摄像头（IPC 设备）。

### 核心运作链路

#### 1. 设备接入 — P2P 跨 NAT 穿透

```
摄像头（社区局域网）         客户端（街道办）
       │                          │
       └──── 向 nat.vveye.net:8000 注册 ────┘
                     ↓（信令协调）
              UDP 打洞 / TCP 中继降级
                     ↓（隧道建立）
         P2PTunnel_AddPort：把摄像头 RTSP 554 端口
              映射到本地随机端口（127.0.0.1:xxxx）
```

关键 DLL：`PPCS_API.dll`、`IOTCAPIs.dll`、`P2PTunnelAPIs.dll`；打洞失败时 `libt2u.dll` 做 TCP 中继。

#### 2. 视频流接收与播放

```
live555（RTSP 客户端）
  → 拉 RTSP 流（连本地隧道端口）
  → RTP 包重排/重组 NALU（H264VideoRTPSource 自动处理 FU-A 分片）
  → CVideoRecieveBuffer（线程安全 NALU 队列，CPtrList + CRITICAL_SECTION）
  → CH264Decoder（FFmpeg avcodec_decode_video2 软解码）
  → YUV420P → swscale → BGR24
  → DirectDraw/GDI 渲染到窗口
```

#### 3. 多路并发调度（最多 16 路）

- **`CIPCDevicesManager` 单例** 统一管理所有通道
- 每路一个 `COneIPCChannel`，内含独立的网络连接、NALU 缓冲队列、解码器
- **生产者线程**（网络收包）→ 写队列
- **消费者线程**（解码渲染）→ 读队列
- 队列满时主动丢 B/P 帧保留 I 帧，防止花屏

#### 4. 音频传输

```
独立 UDP 通道 → G711/G726 编码数据
  → CAudioRecieveBuffer（环形缓冲队列）
  → 解码 → waveplay 播放
  → 与视频流通过 RTP 时间戳对齐
```

#### 5. 业务服务器交互（注册/登录/推送报警）

```
CClientSocket（MFC CSocket 长连接）
  ↕ XML 格式命令（creatXml / ResolveXml）
业务服务器
  → 下发报警推送 → 客户端弹窗 + 托盘闪烁提示
```

#### 6. 录像存储

- NALU 队列数据**同时分发**给解码线程（预览）和录制线程（写文件）
- 写裸 H264 Annex B 码流（`.264`），按时间分片，`fwrite` 直接追加
- 支持定时/告警触发/手动三种录像策略

### 关键技术栈

| 层次 | 技术 |
|------|------|
| P2P 穿透 | PPCS/IOTC SDK + libt2u 中继 |
| 流媒体接入 | live555（RTSP/RTP 客户端）|
| 视频解码 | FFmpeg avcodec + swscale |
| 音频编码 | G711 A-law / G726 |
| 线程同步 | CRITICAL_SECTION + CPtrList 队列 |
| 业务通信 | MFC CSocket + libxml2（XML 协议）|
| UI 框架 | MFC 对话框应用（Windows） |
| 日志调试 | ST_debug.h + Dbgview.exe |

---

## 五、Live555 在项目中的作用与配合方式

### Live555 的角色

Live555 是作为**独立静态库**编译进来的，链接了四个 `.lib`：

```cpp
#pragma comment(lib, "BasicUsageEnvironment.lib")
#pragma comment(lib, "groupsock.lib")
#pragma comment(lib, "liveMedia.lib")
#pragma comment(lib, "UsageEnvironment.lib")
```

Live555 充当 **RTSP 客户端**，负责：

1. **建立 RTSP 会话**：发送 `DESCRIBE` → `SETUP` → `PLAY` 等标准 RTSP 命令，解析摄像头返回的 SDP
2. **接收 RTP 包**：通过 `RTPSource` 处理底层 UDP 包的乱序、丢包重排
3. **H264 NALU 重组**：`H264VideoRTPSource` 自动按 RFC 6184 把 FU-A 分片拼回完整 NALU
4. **数据回调**：重组完的 NALU 通过 `g_recieveNaluFromLive555` 函数指针回调给 `COneIPCChannel::PutNaluDataIntoChannel()`

### 核心机制：事件驱动 + 自定义 Sink

Live555 采用**单线程事件循环**模型，整个 RTSP 流程通过异步回调串联：

```
Live555Thread()
  ├─ BasicTaskScheduler::createNew()
  ├─ createEventTrigger(openURL)      注册"打开摄像头"事件
  ├─ createEventTrigger(shutdownStream)
  └─ doEventLoop()  ← 永不返回

openURL()
  └─ sendDescribeCommand()  ──异步──▶  continueAfterDESCRIBE()
                                           └─ setupNextSubsession()
                                                └─ sendSetupCommand()  ──异步──▶  continueAfterSETUP()
                                                                                       └─ DummySink::startPlaying()
                                                                                       └─ sendPlayCommand()  ──异步──▶  continueAfterPLAY()
```

### 项目扩展点：DummySink 替换为自定义 Sink

Live555 官方示例的 `DummySink` 只收数据不处理。项目将其 `afterGettingFrame()` 替换为：

```cpp
void DummySink::afterGettingFrame(...) {
    // fReceiveBuffer 里已经是重组好的完整 NALU
    if (g_recieveNaluFromLive555 != NULL)
        g_recieveNaluFromLive555(frameSize, (char*)fReceiveBuffer, fStreamId);
    // 告诉 Live555 继续接收下一帧
    continuePlaying();
}
```

`g_recieveNaluFromLive555` 是一个全局函数指针，启动时由 `IPCDevicesManager` 注入，最终调用到 `COneIPCChannel::PutNaluDataIntoChannel()` 写入队列。

### 与 P2P 隧道的配合

Live555 完全不感知 P2P，它只认 RTSP URL：

```cpp
// 连的是本地隧道端口，而不是摄像头真实 IP
openURL("rtsp://127.0.0.1:12345/live");
//               ↑ P2PTunnel_AddPort 映射出来的本地端口
```

**P2P SDK 负责"打通隧道"，Live555 负责"在隧道上跑 RTSP"，两者分工清晰。**

---

## 六、线程划分全景图

### 全局唯一线程

| 线程 | 定义位置 | 职责 |
|------|----------|------|
| **Live555 事件循环线程** (`Live555Thread`) | `testRTSPClient.h` | 运行 `doEventLoop()`，单线程驱动所有 RTSP 信令收发 + RTP 收包 + NALU 重组，是所有网络 I/O 的入口 |
| **TCP 业务线程** (`recvbufthread`) | `IPCDevicesManager.cpp` | 维持与业务服务器的长连接，接收推送报警 XML |
| **音频发送线程** (`sendbufthread`) | `IPCDevicesManager.cpp` | 向摄像头发送麦克风采集的 G711 音频数据 |

### 每路视频通道的线程（`COneIPCChannel`，最多 16 路 × 2）

每个 `COneIPCChannel` 实例启动 **2 个线程**：

```
Live555 事件循环（生产者）
    │  NALU 重组完成后回调 PutNaluDataIntoChannel()
    ▼
CVideoRecieveBuffer（CRITICAL_SECTION 保护的 CPtrList 队列）
    │
    ├──▶ DecodeThread（优先级 THREAD_PRIORITY_HIGHEST）
    │       从队列取 NALU → FFmpeg avcodec 解码 → YUV→RGB(swscale)
    │
    └──▶ Thread_720P（渲染线程）
            将 RGB 帧通过 DirectDraw/GDI 贴图到视频窗口（CWnd）
```

### 辅助/一次性线程

| 线程 | 触发时机 | 职责 |
|------|----------|------|
| **`NaluStore::StoreThread`** | 开启录像时 | 消费 NALU 队列，写裸 H264 文件 |
| **`G711Record::EncodeRecordThread`** | 音频录制时 | G711/G726 音频编码后写文件 |
| **`snapPicthread`** | 截图请求时 | 一次性截图 |
| **`sendfilethread`** | 上传录像文件时 | 文件传输 |
| **`CViWavePlay::ThreadProc`** | 音频播放时 | 消费音频缓冲队列，播放音频 |
| **`TCPConnectIPCThread`** | P2P 连接时 | 异步建立 P2P 隧道 |

### 生产者-消费者关系汇总

```
[Live555 事件循环线程]  ──写──▶  CVideoRecieveBuffer
                                    ├──读──▶ DecodeThread（解码）
                                    └──读──▶ NaluStore::StoreThread（录像）

[麦克风采集]  ──写──▶  CAudioRecieveBuffer
                           ├──读──▶ CViWavePlay::ThreadProc（本地播放）
                           └──读──▶ G711Record::EncodeRecordThread（录音）
```

16 路视频最多同时跑约 **35+ 个线程**（1 Live555 + 16×2 解码渲染 + 若干辅助）。

---

## 七、网络层到播放的完整流程

### 第一阶段：初始化（程序启动时，只做一次）

```
XY_IPC_Init()                          [SDKInterface.cpp]
  └─▶ CIPCDevicesManager::Init()
        └─ PPCS_Initialize()            初始化 P2P SDK，向 nat.vveye.net 注册

Live555Thread(LIVE555PARAM)            [testRTSPClient.cpp]
  ├─ BasicTaskScheduler::createNew()
  ├─ BasicUsageEnvironment::createNew()
  ├─ createEventTrigger(openURL)       注册"打开摄像头"事件
  ├─ createEventTrigger(shutdownStream)
  ├─ g_recieveNaluFromLive555 = 上层传入的回调函数指针
  └─ doEventLoop()                     ← 永不返回
```

### 第二阶段：连接一路摄像头

```
XY_IPC_StartRealPlay()
  └─▶ CIPCDevicesManager::StartRealPlay()
        ├─ new COneIPCChannel
        ├─ InitIPCChannel(pDisplayWnd)
        │     ├─ new CVideoRecieveBuffer
        │     └─ InitializeCriticalSection
        └─ StartIPCChannel("IP:8554")
              ├─ _beginthreadex(DecodeThread)    解码线程启动
              └─ _beginthreadex(Thread_720P)     渲染线程启动

// 同时进行：
CP2PClient::connectP2PServer(uid)
  └─ IOTC_Connect_ByUID()  → P2P 云服务器协调打洞
        └─ P2PTunnel_AddPort(554 → 本地随机端口)
               └─ triggerEvent(g_OpenIPCEvent)
                    └─ openURL("rtsp://127.0.0.1:随机端口/live")
```

### 第三阶段：RTP 收包 → NALU 重组（Live555 事件循环内）

```
摄像头 UDP RTP 包
  └─▶ H264VideoRTPSource（Live555 内部）
        ├─ 乱序重排（按 RTP Sequence Number）
        ├─ FU-A 分片重组 → 完整 NALU
        └─ DummySink::afterGettingFrame()
                └─ g_recieveNaluFromLive555(frameSize, NaluBuffer, streamId)
                      └─ COneIPCChannel::PutNaluDataIntoChannel()
                            └─ EnterCriticalSection
                            └─ m_pVideoBuffer->NaluDataAdd()   CPtrList 入队
                            └─ LeaveCriticalSection
                └─ continuePlaying()  告诉 Live555 继续接收下一帧
```

### 第四阶段：解码（DecodeThread，THREAD_PRIORITY_HIGHEST）

```
DecodeThread → decode()  [循环]
  └─ m_pVideoBuffer->NaluDataGet()    从队列取 NALU（空则 WaitForSingleObject）
  └─ CH264Decoder::DecodeVideo()
        ├─ av_init_packet(&pkt)
        ├─ pkt.data = NaluBuffer
        ├─ avcodec_decode_video2()    FFmpeg 软解码 → YUV420P
        └─ sws_scale()               YUV420P → BGR24（SIMD 加速）
  └─ 若 GotFrame == 1：
        ├─ 首帧：InitDirectDraw(hwnd, width, height)
        └─ m_pDDraw->DrawDirectDraw(pYUVBuffer)
  └─ 若开启录像(m_bStoreNalu)：
        同时将 NALU 写入 128KB 环形缓冲，满了 fwrite → .h264 文件
```

### 第五阶段：渲染（Thread_720P）

```
Thread_720P → recv_720P()  [循环]
  └─ CDirectDraw::DrawDirectDraw()
        ├─ Lock 离屏 Surface（lpDDSOffscreen）
        ├─ CopyToDDraw()：RGB memcpy 到 DirectDraw Surface
        ├─ Unlock
        └─ Blt()：离屏 Surface → 主 Surface（显示到 HWND）
```

### 完整数据流总览

```
[摄像头]
   │ UDP RTP 包（H264 FU-A 分片）
   ▼
[P2P 隧道 127.0.0.1:xxxx]
   │
   ▼
[Live555 事件循环线程]
  H264VideoRTPSource → 重排 + FU-A 重组 → 完整 NALU
   │ g_recieveNaluFromLive555 回调
   ▼
[CVideoRecieveBuffer]  ← CRITICAL_SECTION 保护的 CPtrList 队列
   │                              │
   ▼                              ▼
[DecodeThread]              [NaluStore::StoreThread]
FFmpeg avcodec_decode_video2    fwrite → .h264 文件
YUV420P → swscale → BGR24
   │
   ▼
[CDirectDraw::DrawDirectDraw]
DirectDraw Surface Blt → 窗口（CWnd HWND）
   │
   ▼
[用户看到视频画面]
```

---

## 八、量化成果

| 指标 | 数值 | 表达方式 |
|------|------|----------|
| 稳定运行时长 | **2 年+** | 系统在 4 个社区无间断运行超过 2 年，未发生数据丢失事故 |
| 推送处理量 | **>1 万条** | 累计处理老人异常报警推送超过 1 万条，响应及时率 100% |
| 版本迭代 | **4 次** | 经历 4 轮版本迭代，每次迭代后性能/稳定性显著提升 |
| 推广区域 | **3 个区** | 已从岳麓区推广至雨花区、天心区，用户规模扩大 3 倍 |
| 并发路数 | **16 路** | 单客户端同时支持 16 路实时视频流，CPU 占用控制在 40% 以内 |

---

## 九、STAR 法则开场白

> **Situation**：岳麓区街道办驻场项目，老年人社区监护场景
>
> **Task**：我负责中控台客户端 + 部分服务端，包括 P2P 接入、音视频流媒体、推送处理、二次重构
>
> **Action**：基于 PPCS P2P SDK + live555 RTSP + FFmpeg H264 软解码 + 多线程生产者消费者模型
>
> **Result**：稳定运行 2 年，处理推送 >1 万条，从岳麓区推广至 3 个行政区，同时支持 16 路实时流

---

## 十、简历逐条深挖问答

### 【简历点 1】P2P 对接，P2P 客户端的实现

**第一层：P2P 和传统 C/S 架构有什么本质区别？**

> 传统 C/S 要求服务器有公网 IP，所有流量经过服务器中转，带宽成本高且服务器是单点瓶颈。P2P 的目标是让两个端点**直接通信**，服务器只负责"牵线"（信令/协调），媒体数据走端对端直连，延迟更低、服务器压力更小。本项目中摄像头和客户端都在各自的局域网里，没有公网 IP，所以必须借助 P2P 打洞技术穿越 NAT。

**第二层：NAT 有哪几种类型？每种对打洞的影响是什么？**

> NAT 按限制程度从低到高分四种：
> - **Full Cone NAT**：只要内网端口映射建立，任何外网地址都能发包进来，打洞最简单
> - **Address Restricted Cone**：必须先向对方发过包，对方才能回包，打洞需要双方同时互发探测包
> - **Port Restricted Cone**：限制到端口级别，双方需要更精确地同步探测端口
> - **Symmetric NAT**：每次连接不同目标时分配不同的外网端口，无法预测，UDP 打洞基本失败，只能靠服务器中继
>
> 本项目中社区网络大多是运营商的对称型 NAT，打洞成功率不高，最终靠 `libt2u.dll` 做的 TCP 中继转发。

**第三层：打洞的具体流程，信令交换是怎么做的？**

> 1. 设备端和客户端启动时都调用 `IOTC_Initialize` 向 P2P 服务器（`nat.vveye.net`）注册，服务器记录其公网 `IP:Port` 和设备 UID
> 2. 客户端调用 `IOTC_Connect_ByUID(uid)` 发起连接请求，P2P 服务器把双方的公网地址互相告知
> 3. 双方同时向对方公网地址发送 UDP 探测包（"打洞"），利用 NAT 的"会话保持"特性，让路由器在 NAT 表里建立双向通路
> 4. 若 3 秒内探测包互通，则直连建立（Session）；否则服务器介入做流量中继
> 5. 直连建立后，调用 `P2PTunnel_AddPort` 在隧道上映射 RTSP 端口（554 → 本地随机端口），live555 连 `127.0.0.1:随机端口` 就等于连到了摄像头

---

### 【简历点 2】音频数据的接收与发送，视频的编解码以及播放实时流，视频录制存储

**第一层：H264 的码流结构是什么，NALU 是什么？**

> H264 码流由一系列 **NALU（Network Abstraction Layer Unit）** 组成。每个 NALU 由 `00 00 00 01`（Annex B 格式）或 2/4 字节长度前缀（AVCC 格式）开头，后跟一字节的 `nal_unit_type`，常见类型：
> - `type=5`：IDR 帧（I 帧），完整关键帧，解码器可以独立解码
> - `type=1`：非 IDR 帧（P/B 帧），依赖前面的帧，独立解码会花屏
> - `type=7`：SPS（序列参数集），描述分辨率、帧率等全局参数
> - `type=8`：PPS（图像参数集），描述熵编码、量化等参数
>
> 解码器必须先收到 SPS+PPS 才能正确初始化，否则 `avcodec_decode_video2` 返回错误。

**第二层：RTP 怎么传 H264，分片规则是什么？**

> H264 over RTP 遵循 RFC 6184，有三种打包模式：
> - **Single NALU**：一个 NALU 装一个 RTP 包，适合小帧（< 1400 字节）
> - **STAP-A**（Single-Time Aggregation Packet）：多个小 NALU 合并进一个 RTP，减少包头开销
> - **FU-A**（Fragmentation Unit）：一个大 NALU 拆成多个 RTP 包，每包有 FU Indicator + FU Header，Header 里的 `S` 位标识起始包，`E` 位标识结束包
>
> 接收端（live555 的 `H264VideoRTPSource`）根据 FU Header 重组，把碎片拼回完整 NALU 后才交给应用层。我们的 `CVideoRecieveBuffer` 收到的已经是重组好的完整 NALU，再塞给 FFmpeg 解码。

**第三层：视频录制是怎么做的，文件格式是什么？**

> 录制时把 NALU 队列里的数据同时分发给两个消费者：解码线程（实时预览）和录制线程（写文件）。录制格式选择写裸 H264 Annex B 码流（`.264` 文件），在每个 NALU 前加 `00 00 00 01` 起始码。不封装成 MP4/MKV 是因为：
> 1. 写文件不需要 muxer，减少依赖
> 2. 录像文件按时间分片（如每小时一个文件），直接 `fwrite` 追加即可，崩溃不会损坏整个文件
> 3. 回放时用 FFmpeg 或 VLC 可以直接打开裸码流

---

### 【简历点 3】推送的接收、处理以及显示以及托盘功能的实现

**第一层：推送是走什么通道来的，和视频流是同一个连接吗？**

> 不是。视频走 P2P 隧道上的 RTSP/RTP，而推送走的是**客户端与业务服务器之间的 TCP 长连接**。这是两条完全独立的通道，设计上做了职责分离：
> - **数据面**：P2P 隧道传音视频，带宽敏感，走 UDP
> - **控制面**：TCP 长连接传命令和推送，可靠性优先，走 TCP

**第二层：TCP 长连接怎么保持，断线了怎么办？**

> `CClientSocket` 继承 MFC `CSocket`，重写了 `OnMessagePending` 实现**超时心跳**：每隔 30 秒发一个心跳包，如果连续 3 次无响应则判定断线，触发重连逻辑。重连采用**指数退避**策略（1s → 2s → 4s → 最大 30s），防止服务器重启瞬间大量客户端同时重连造成雪崩。

---

### 【简历点 4】与服务器的交互（TCP），注册、登录、固件升级、XML 解析

**第一层：登录的完整流程是什么，密码怎么处理的？**

> 1. 客户端从 `ServerInfo.ini` 读取服务器地址，建立 TCP 连接
> 2. 构建登录 XML，**密码做 MD5 哈希后传输**，不传明文
> 3. 服务器验证通过后返回 Session Token，后续所有请求携带该 Token
> 4. Token 有效期内免重登，Token 过期时自动重新登录
>
> 注意：Token 只存在内存中，不写文件，防止凭据泄露。

**第二层：固件升级是怎么做的，升级失败怎么处理？**

> 固件升级流程：
> 1. 客户端向服务器查询最新固件版本号，与本地版本比对
> 2. 如有新版本，下载固件包（HTTP 分片下载，每片 8KB），下载完成后校验 MD5
> 3. 通过 TCP 把固件包分片发给 IPC 设备（走 P2P 隧道），设备端接收完成后自动重启刷写
> 4. 升级失败时保留旧固件，设备重启恢复正常工作，**不存在砖机风险**

**第三层：XML 构建和解析为什么不用 JSON，有什么考量？**

> 这是历史技术决策。项目开始于 2013 年左右，ONVIF 标准本身就用 XML + SOAP，整个 SDK 的数据交换格式已经是 XML，为了统一保持一致而不是混用两种格式。如果重新设计，会考虑 JSON，因为解析性能更好、包体更小。

---

## 十一、高频面试问题及回答

### Q1: "你在CMS项目中遇到的最大困难是什么？"

> "16路视频同时播放时CPU占用过高。原因是每路视频的解码和渲染都在独立线程中，线程上下文切换开销大，且DirectDraw渲染没有批量处理。
>
> 解决方案：
> 1. 动态调整弱网路视频的解码分辨率（从720P降到VGA）
> 2. 非焦点窗口降低帧率（从25fps降到10fps）
> 3. 复用DirectDraw Surface，减少创建销毁开销
>
> 效果：CPU占用从95%降到60%。"

### Q2: "RTSP和HTTP的区别？"

> | 维度 | RTSP | HTTP |
> |------|------|------|
> | 定位 | 流媒体控制协议 | 通用Web协议 |
> | 状态 | 有状态（会话ID） | 无状态 |
> | 传输 | 可以承载RTP（UDP/TCP） | TCP |
> | 方法 | DESCRIBE/SETUP/PLAY/PAUSE/TEARDOWN | GET/POST/PUT/DELETE |
> | 实时性 | 设计用于实时流 | 设计用于文件传输 |

### Q3: "DirectDraw渲染和GDI渲染的区别？"

> | 维度 | DirectDraw | GDI |
> |------|-----------|-----|
> | 渲染方式 | 直接写显存 | 通过Windows消息机制 |
> | 性能 | 高（硬件加速） | 低（软件渲染） |
> | 适用场景 | 视频、游戏 | 普通UI |
>
> 我选DirectDraw的原因：16路视频同时渲染，GDI性能不够；DirectDraw支持YUV直接渲染，省去格式转换；支持Overlay Surface，减少CPU参与。

### Q4: "ONVIF和RTSP的关系？"

> ONVIF是设备管理协议，RTSP是流媒体控制协议，两者互补：
> - ONVIF负责：设备发现、设备配置、PTZ控制、事件订阅
> - RTSP负责：视频流的建立、控制、传输
>
> 在我的项目中：
> 1. 先用ONVIF发现局域网内的IPC设备
> 2. 通过ONVIF获取设备的RTSP URL
> 3. 用RTSP客户端（Live555）拉取视频流

### Q5: "你的两个项目有什么技术关联？"

> 两个项目在NAT穿越和视频流处理上有直接关联：
>
> | 技术点 | CMS项目 | HCDN项目 |
> |--------|---------|----------|
> | NAT穿越 | 用第三方IOTC SDK | 自研ICE/STUN协议栈 |
> | 视频流 | RTSP拉流+本地渲染 | HLS/FLV拉流+P2P分发 |
> | 视频解码 | FFmpeg H.264解码 | 不涉及解码，只做传输 |
> | C++风格 | 传统MFC C++ | 现代C++(智能指针/Asio) |
>
> CMS项目中用第三方SDK的经验，让我理解了NAT穿越的原理，所以在HCDN项目中我选择自研ICE协议栈而非使用WebRTC库。

---

## 十二、终极连环追问（面试官最爱的"杀手锏"问题）

**Q: 你们的系统最大的技术债是什么，如果重新设计会怎么做？**

> 最大的技术债是**没有自动化测试**。音视频系统很难做单元测试，当时全靠手动回归。如果重新设计：
> 1. 把解码、网络、UI 三层完全解耦，解码层用纯 C++ 类、不依赖任何 Windows API，可以在 CI 上做单元测试
> 2. 引入消息队列（哪怕是简单的线程安全队列）作为层间通信，方便插入 Mock 数据做集成测试
> 3. 日志系统改用结构化日志（JSON 格式），方便后期接入 ELK 做日志分析

**Q: 16 路视频同时解码，CPU 怎么够用？**

> 这是一个很好的问题。实际上并不是所有 16 路都在全速解码：
> 1. 没有展示在屏幕上的通道（缩略图模式）降低帧率到 5fps，只解 I 帧
> 2. 全屏显示的通道才做全帧率（25fps）解码
> 3. FFmpeg 的 `avcodec_decode_video2` 支持多线程解码（`AVCodecContext.thread_count`），充分利用多核
> 4. 实测在 Intel i5 四核机器上，16 路 720P@25fps 的 CPU 占用约 35～40%，在可接受范围内

**Q: 项目里有没有内存泄漏，怎么排查的？**

> 有过。早期 `NaluData` 的 `char* m_pNaluBuffer` 用 `new[]` 分配，但在某些异常路径（解码器返回错误时）没有 `delete[]`，导致长时间运行后内存持续增长。排查手段：
> 1. 用 `memwatch.h` 做内存分配追踪，它 hook 了 `malloc/free`，能报告泄漏位置
> 2. Windows 任务管理器监控进程内存趋势，确认是缓慢线性增长（泄漏）而非突发增长（碎片）
> 3. 修复：把裸指针改为在析构函数里保证释放，并加了 `Destroy()` 方法做显式清理

---

## 十三、简历写了 VLC 但代码没用——高危区专项应对

### 策略一：如实说明 VLC 的实际角色（推荐）

> VLC 在这个项目里主要用在**开发和调试阶段**，而不是集成进最终产品：
> 1. **流验证工具**：摄像头 RTSP 流调通后，第一步是用 VLC 打开 `rtsp://ip:554/...`，确认码流格式正确、画面正常，再接入我们自己的解码链路
> 2. **录像文件回放**：客户端录制的裸 H264 码流（`.264` 文件）用 VLC 验证可以正常播放，确认分片和 NALU 格式正确
> 3. **对比基准**：当我们的解码画面出现花屏时，用 VLC 播放同一段码流，如果 VLC 正常而我们花屏，说明问题在我们的解码逻辑；如果 VLC 也花屏，说明问题在码流本身
>
> 最终产品的实时解码用的是 **FFmpeg + live555**，而不是嵌入 libvlc，原因是我们需要逐帧控制 YUV 数据做截图和录制，libvlc 的回调接口颗粒度不够细。

### 策略二：展示对 VLC 架构的理解

> VLC 是模块化架构，核心是 **libvlc**，分为几层：
> - **Access 层**：负责从各种来源读取数据，对应我们项目的 live555 RTSP 拉流层
> - **Demux 层**：解封装，对应我们项目的 RTP 解包/NALU 重组
> - **Codec 层**：解码，VLC 底层实际也调用了 FFmpeg，和我们用的是同一套解码器
> - **Video Output 层**：渲染，VLC 支持 OpenGL/DirectX/X11 等多种输出，我们项目用的是 DirectDraw
>
> 所以本质上 VLC 和我们的技术栈重叠度很高，VLC 只是把这些组件封装成了对用户友好的播放器，我们项目是把同样的组件直接集成、做精细控制。

### 简历修正建议

如果还有机会改简历，将技术栈中的 **VLC** 替换为：

```
主要技术：FFmpeg（H264软解码/YUV转RGB/swscale）、live555（RTSP/RTP）、
PPCS/IOTC P2P SDK、DirectDraw、G711/G726音频编解码……
```

---

## 十四、项目亮点总结

| 序号 | 亮点 | 一句话描述 |
|------|------|-----------|
| 1 | 完整的视频监控全链路 | 从设备发现到视频采集、传输、解码、渲染、存储 |
| 2 | 多协议支持 | ONVIF发现 + RTSP流媒体 + P2P远程穿透 |
| 3 | FFmpeg H.264解码 | 直接使用libavcodec API，理解NALU→YUV解码流程 |
| 4 | 16路视频并发 | 多线程同步、资源管理、动态降级 |
| 5 | 代码重构 | 耦合代码→分层架构，代码量减少30% |
| 6 | 系统稳定运行2年 | 覆盖4个社区，处理推送信息超1万条 |

---

## 十五、简历措辞修改建议

| 修改前 | 修改后 |
|--------|--------|
| "VLC、RTSP协议、H264分片" | "RTSP流媒体拉流、FFmpeg H.264解码、Live555客户端实现" |
| "socket、TCP/UDP" | "TCP长连接管理、UDP NAT穿透、RTSP/RTP流媒体传输" |
| "线程池、Epoll" | "生产者-消费者模型、多线程视频解码管线、环形缓冲区" |
| "MFC客户端开发" | "MFC中控台客户端开发，16路视频并发播放与渲染" |
| "P2P穿透" | "基于IOTC SDK实现NAT穿透，支持跨网段设备访问" |

**项目描述修改建议：**

修改前：
> 负责中控台客户端+服务端部分业务逻辑实现、二次代码重构

修改后：
> 负责中控台客户端核心模块开发（RTSP流媒体播放、FFmpeg H.264解码、16路并发渲染、ONVIF设备发现、P2P NAT穿透）及二次代码重构（分层架构+设计模式），系统稳定运行2年，覆盖4个老年社区，处理推送信息超1万条
