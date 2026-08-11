# FC7300F8MDQ FLEXCAN Chapter 66 集中学习设计

## 1. 目标与学习者画像

本课程以《FC7300F8MDQ Reference Manual V0.5》Chapter 66（手册第 2288-2424 页）为硬件事实来源，以 `D:\7300_MCAL\MCAL` 中的 Can 驱动为实现对照，帮助学习者从“理解 CAN 协议”进阶到“能独立编写寄存器级 FLEXCAN 收发驱动并完成真实总线调试”。

已确认的学习条件：

- 学习者理解 CAN 帧格式、仲裁和位时序，但没有 CAN/FLEXCAN 驱动编写经验。
- 采用集中式训练，不按周安排。
- 具备 FC7300F8MDQ 开发板、CAN 收发器和 USB-CAN 或其他 CAN 节点。
- 目标不是只会修改 MCAL 配置，而是能够脱离参考驱动重建核心流程。

## 2. 课程组织原则

课程采用任务驱动闭环：

`原理 -> 寄存器 -> 状态/时序 -> 独立伪代码 -> 最小代码 -> 上板验证 -> 故障实验 -> MCAL 对照 -> 复盘`

Chapter 66 是判断寄存器行为和操作顺序的第一依据。MCAL 只在独立推导或最小实验之后使用，用来验证顺序、发现量产级保护措施，并区分 FLEXCAN 硬件复杂度与 AUTOSAR 框架复杂度。

课程共 8 个训练单元，每单元预计 2-4 小时，可集中在 4-6 天完成。实际推进以单元验收结果为准，不以时间到点为准。

## 3. 八个训练单元

### 单元 1：FLEXCAN 全景与实例差异

- 建立 Protocol Engine、Controller Host Interface、Message Buffer RAM、外部 CAN 收发器和 CAN 总线之间的关系。
- 理解发送仲裁、接收匹配、move-in、move-out 的职责边界。
- 区分 FLEXCAN0-7 与 FLEXCAN8-13 的 MB 数量和 CAN FD/Enhanced FIFO 能力，理解 PNET 只适用于 FLEXCAN0-2。
- 产出模块框图、实例能力表和第一版寄存器分组图。

### 单元 2：Freeze、复位、时钟与 ECC RAM

- 掌握 Disable、Freeze、Normal 模式以及 `MDIS/HALT/FRZ/FRZACK/NOTRDY/LPMACK` 的请求-应答关系。
- 理解软复位与硬复位的差异，以及哪些 RAM/寄存器不受复位影响。
- 在写其他配置前显式初始化 MB、RXIMR 和所启用特性的 RAM 区域，避免 ECC 错误。
- 产出带超时的最小初始化骨架和关键寄存器快照。

### 单元 3：Classical CAN 位时序与 Loop-Back

- 从 CAN 时钟、Tq、Sync/Segment1/Segment2、SJW 和采样点推导 CTRL1/CBT 参数。
- 独立计算并验证至少一种 500 kbit/s Classical CAN 配置。
- 先用内部 Loop-Back 验证协议引擎和 MB，再接入外部收发器。
- 产出位时序计算表、Loop-Back 报文记录和失败诊断表。

### 单元 4：Message Buffer 状态机与轮询收发

- 掌握 MB 的 C/S、ID、数据区、时间戳和 CODE 状态。
- 实现一个 Tx MB 和一个 Rx MB 的标准帧与扩展帧收发。
- 发送时保证 ID/数据先写、C/S[CODE] 最后提交；接收时使用 IFLAG 判断报文到达。
- 严格执行 Rx 的 BUSY 检查、内容读取、IFLAG W1C 和 TIMER 解锁顺序。
- 使用 USB-CAN 完成真实双向收发。

### 单元 5：中断收发与数据一致性

- 建立 `IMASK -> IFLAG -> ISR -> MB service -> W1C` 的完整路径。
- 理解 W1C 标志不能用有竞争风险的读改写方式清除。
- 理解 move-in/move-out 非原子、发送 abort、MB inactivation、MB lock 以及软件并发保护。
- 将单元 4 的轮询收发改造成中断收发，并验证连续报文和中断重入边界。

### 单元 6：ID Mask、Matching 与 Rx FIFO

- 从单个 Rx MB 的 ID/IDE/RTR 掩码开始，验证精确匹配与范围匹配。
- 理解 IRMQ、RXIMRn、全局掩码、MRP 和匹配优先级。
- 对比 Legacy Rx FIFO 与 Enhanced Rx FIFO 的能力、RAM 占用、过滤元素、中断/DMA行为和 CAN FD 兼容性。
- 完成命中、未命中、warning、overflow 和 FIFO 清除实验。

### 单元 7：CAN FD、BRS 与 TDC

- 理解 Classical/FD 帧混合、EDL、BRS、ESI 及 DLC 9-15 到 12/16/20/24/32/48/64 字节的映射。
- 配置名义相位与数据相位位时序，解释 FDCTRL、FDCBT 或增强位时序寄存器的选择关系。
- 使用 USB-CAN 完成 CAN FD+BRS 收发。
- 理解 TDC 的使用条件、二次采样点、offset 和超范围诊断。

### 单元 8：错误恢复与 MCAL 对照验收

- 主动制造 ACK 缺失、位速率错误和 Bus Off，观察 ECR/ESR1 以及中断标志变化。
- 建立错误状态、错误计数、Bus Off 和恢复动作的因果链。
- 对照 MCAL 的初始化、发送、接收和错误处理调用链，识别超时、SchM 临界区、多核、配置变体和 AUTOSAR API 带来的封装。
- 完成最终盲测任务，不依赖 MCAL 代码提示重建核心流程。

## 4. 每单元固定学习闭环

1. 用 3-5 个问题检查前置知识。
2. 只阅读完成当前任务必需的手册页面。
3. 整理寄存器职责、允许写入模式、硬件应答位、W1C 位和复位影响。
4. 不查看 MCAL，先画状态流程并写伪代码、超时和失败返回路径。
5. 实现单一最小能力，避免同时引入初始化、中断、FIFO 和 CAN FD。
6. 从寄存器层、总线层、行为层验证结果。
7. 主动注入至少一种错误，并记录预期与实测差异。
8. 再阅读 MCAL 对应实现，回答“顺序是否一致、增加了哪些保护、哪些复杂度来自 AUTOSAR”。
9. 形成 `软件动作 -> 寄存器变化 -> 硬件状态 -> IFLAG/ESR -> 总线现象` 的因果链。

## 5. 每单元提交物与通过标准

每单元提交：

- 一张状态流程或寄存器因果链。
- 一段最小可运行代码。
- 一份关键寄存器快照。
- 一份 USB-CAN 报文或错误记录。
- 三条踩坑记录。

通过当前单元必须同时满足：

- 能解释每次关键寄存器写入的目的和顺序。
- 所有硬件握手轮询都有超时和失败出口。
- 正确处理 W1C、BUSY、MB lock 和 TIMER 解锁。
- 能区分系统配置、FLEXCAN 模块、协议/MB 和物理总线四类故障。
- 不查看 MCAL 时仍能重写当前单元的核心流程。

## 6. 教学互动方式

- 教师每次只讲一个核心模型，然后给出一道检查题或寄存器推演题。
- 学习者回答后，教师依据手册纠正理解，再进入伪代码和实验。
- 学习者提供代码、寄存器快照或 USB-CAN 记录，教师进行证据驱动的检查。
- 未通过当前单元验收时，暂停新增功能，先完成定位和修正。
- 每次对话明确当前单元、已通过项目、待提交证据和下一步动作。

## 7. 系统边界与故障分层

Chapter 66 定义 FLEXCAN 模块本身。真实上板还依赖 PCC 时钟门控/时钟源、PinMux/IOMUX、SCM FLEXCAN routing、中断控制器以及收发器 GPIO/电源控制。这些依赖只在对应实验需要时补充，不扩展成另一套课程。

故障统一按四层定位：

1. 系统层：时钟门控、PinMux、复位、routing、收发器使能。
2. 模块层：Disable/Freeze/Normal 状态、请求-应答和 RAM/ECC。
3. 数据层：MB CODE、ID、DLC、Mask、IMASK/IFLAG 和解锁顺序。
4. 总线层：终端电阻、ACK、位时序、物理波形和错误帧。

## 8. MCAL 对照范围

参考驱动目录保持只读。核心对照文件为：

- `Src/Can/src/Can_Hal.c`
- `Src/Can/include/Can_Hw.h`
- `Src/Can/include/Can_Reg.h`
- `Src/Can/src/Can.c`

优先追踪三条调用链：

- `Can_Init -> Can_Hal_Init -> Can_HL_InitController`
- `Can_Write -> Can_Hal_Write -> Can_LL_UpdateTransmitMB`
- `Can_Hal_ProcessRx -> Can_LL_ProcessRxNormal`

已识别的版本核对项是：Chapter 66 芯片能力表给出的 Enhanced FIFO depth 为 20，而参考驱动 `Can_Reg.h` 的 `FLEXCAN_ENHANCED_FIFO_DEPTH` 为 12。课程中将通过寄存器配置、代码用途和驱动版本核对其含义，不在缺少证据时直接判定任一方错误。

## 9. 最终成果与验收

最终成果包括：

- Chapter 66 知识地图。
- “功能-寄存器-MCAL 函数”对照表。
- 最小寄存器级 FLEXCAN 驱动，覆盖初始化、轮询收发、中断收发和诊断。
- Classical CAN 与 CAN FD 位时序计算记录。
- USB-CAN 收发、过滤、FIFO 和错误实验记录。
- 常见故障检查表。

最终盲测要求学习者不依赖 MCAL 代码提示完成：

1. 从复位状态初始化一个支持 CAN FD 的 FLEXCAN 实例。
2. 以指定波特率收发标准帧和扩展帧。
3. 使用 Rx MB 掩码只接收目标 ID。
4. 将轮询接收切换为中断接收。
5. 正确处理 IFLAG、BUSY 和 TIMER 解锁。
6. 制造 ACK 缺失或位速率错误并解释 ECR/ESR1。
7. 完成 Bus Off 恢复。
8. 切换到 CAN FD+BRS 并解释 DLC、数据相位和 TDC。

当以上任务均有代码、寄存器和总线证据，并且学习者能解释关键因果链时，课程目标达成。
