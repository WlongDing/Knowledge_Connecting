# 第一课：TriCore 架构与 TC375 启动总览

## 本课目标

完成本课后，应能不看资料画出：

```text
PORST / Reset
    ↓
Boot Firmware（芯片 ROM 中）
    ↓ 读取并校验 BMHD，确定 Boot Mode 与用户起始地址
CPU0 Application Startup Software
    ↓
早期 CPU 状态 → CSA/栈 → 时钟/Flash/Cache → BIV/BTV → C 运行环境
    ↓                              ↓
释放 CPU1/CPU2                    DATA copy / BSS clear
    ↓                              ↓
各核启动流程 ─────────────────→ Cpu0/1/2_Main
```

## 1. 第一个认知转换：启动入口不等于 Cortex-M 向量表首项

Cortex-M 常从地址 0 读取初始 MSP 和 Reset_Handler。TC375 上电后先执行芯片 Boot Firmware。Boot Firmware 检查启动配置和 BMHD，随后才把执行权交给用户 Application Startup Software。

因此分析 TC375 启动时，应把它拆成三层：

1. **Boot Firmware**：芯片提供，决定从哪里、以什么模式启动；
2. **Application Startup Software（SSW）**：工程提供，建立 CPU 和 C 运行环境；
3. **Application**：各核 `main()` 及业务代码。

本周先回答“每层负责什么”，暂时不要陷入所有安全自检细节。

## 2. 第二个认知转换：CSA 与普通栈并存

TriCore 有地址寄存器 A0～A15 和数据寄存器 D0～D15。A10 通常作为栈指针，但函数调用、中断和 Trap 的上下文管理还依赖 Context Save Area（CSA）。

启动代码必须先根据链接器给出的 CSA 区域，建立空闲上下文链，并设置 FCX/LCX；否则后续函数调用或异常处理没有可靠的上下文资源。

先记住三个寄存器：

- `PCXI`：指向前一个上下文信息；
- `FCX`：空闲 CSA 链的头；
- `LCX`：接近耗尽时的边界提示。

这解释了为何 TC375 链接文件不仅分配栈，还必须为每个核分配 CSA。

## 3. 第三个认知转换：中断路由和异常入口是两层问题

外设事件不会像 Cortex-M 那样仅凭一个固定 IRQn 进入 NVIC。要先配置该事件对应的 Service Request Node：

- `TOS`：交给哪个服务提供者，例如 CPU0、CPU1、CPU2 或 DMA；
- `SRPN`：服务请求优先级，也用于 CPU 中断向量索引；
- `SRE`：是否使能该服务请求；
- `SRR`：是否存在 pending 请求。

CPU 收到请求后，依据本核 BIV 和 SRPN 进入中断向量表。Trap 则使用另一张由 BTV 指向的表：先按 8 个 Trap Class 进入，再从 D15 中读取 TIN 判断具体原因。

可先用下面两条链来记忆：

```text
外设事件 → SRC(TOS/SRPN/SRE) → Interrupt Router → CPUx → BIV + SRPN → ISR
CPU 异常 → Trap Class → BTV + Class offset → TSR → D15.TIN 识别原因
```

## 4. 第一轮工程寻路

在 ADS 中导入与你的板卡相匹配的官方 Blinky 或 STM Interrupt 示例。不同 ADS/iLLD 版本目录可能不同，请按文件名和符号搜索，而不是死记路径。

依次找到并记录：

1. `Cpu0_Main.c`、`Cpu1_Main.c`、`Cpu2_Main.c`；
2. `Ifx_Cfg_Ssw.h`、`Ifx_Cfg_Ssw.c`、`Ifx_Cfg_SswBmhd.c`；
3. 含 `_START`、`__Core0_start` 或同类启动入口的 SSW 文件；
4. 含 `Ifx_Ssw_initCSA` 或同类逻辑的文件；
5. `Lcf_Tasking_Tricore_Tc.lsl` 或 `Lcf_Gnuc_Tricore_Tc.lsl`；
6. 构建生成的 `.map` 文件；
7. 搜索以下链接符号或同义符号：`USTACK`、`ISTACK`、`CSA`、`INTTAB`、`TRAPTAB`。

对每项只写一句“它负责什么”，先不抄代码。

## 5. 调试实验：从用户启动入口走到 main

### 实验准备

- 使用原始可运行示例，先完整 Build、Flash、Run；
- 保存一份构建日志和 `.map`；
- 暂时不要修改 BMHD，也不要关闭看门狗或安全检查来“绕过”问题。

### 观察点

根据工具链实际符号名，在以下位置设置断点：

1. 用户 SSW 最早可见入口（常见为 `_START` 一类符号）；
2. CPU0 启动函数（常见为 `__Core0_start` 一类符号）；
3. CSA 初始化附近；
4. BIV/BTV 写入附近；
5. C 初始化或 copy/clear table 处理附近；
6. `core0_main()` / `Cpu0_Main.c` 入口；
7. CPU1/CPU2 启动入口及其 `main()`。

每到一处记录：

| 观察项 | 记录内容 |
|---|---|
| PC | 当前符号与地址 |
| A10 | 当前栈地址，属于哪个核的哪块 RAM |
| FCX / LCX | CSA 初始化前后是否改变 |
| BIV / BTV | 指向的地址是否与 `.map` 中 INTTAB/TRAPTAB 一致 |
| CPU 状态 | 其他核处于 HALT、RUN 还是已进入 main |
| 全局变量 | DATA 是否已复制、BSS 是否已清零 |

调试器若默认“运行到 main”，需要关闭该选项或增加早期断点，否则会误以为复位后直接到 main。

## 6. `.map` 文件练习

从 `.map` 中找到以下对象并填表：

| 对象 | 地址 | 大小 | 所属物理内存 | 哪个核使用 |
|---|---:|---:|---|---|
| CPU0 USTACK |  |  |  |  |
| CPU0 ISTACK |  |  |  |  |
| CPU0 CSA |  |  |  |  |
| CPU0 INTTAB |  |  |  |  |
| CPU0 TRAPTAB |  |  |  |  |
| `.data` |  |  |  |  |
| `.bss` |  |  |  |  |
| `Cpu0_Main` |  |  |  |  |

然后用调试器寄存器值验证至少三项：A10 对 USTACK、BIV 对 INTTAB、BTV 对 TRAPTAB。

## 7. 本课验收题

不看资料回答：

1. 为什么 TC375 不能把“复位入口”简单理解为地址 0 的 Reset_Handler？
2. 已经有栈，为什么链接文件还必须分配 CSA？
3. 一个 STM 事件要让 CPU1 以优先级 20 响应，至少要配置哪条硬件链？
4. BIV 和 BTV 分别解决什么问题？
5. 链接文件中的 `INTTAB`、`TRAPTAB`、`CSA` 符号，分别会被启动代码写入或用于初始化什么？

达标标准：能画出启动主线，并用 `.map` 地址和运行时寄存器互相印证，而不是只会复述 API 名称。

## 8. 提交给导师的信息

完成后提供：

- 芯片完整 OPN / step；
- 开发板、ADS、编译器与 iLLD 版本；
- 上述 `.map` 表格；
- 你的启动流程图；
- 5 道验收题的答案；
- 断点未命中、寄存器无法读取等具体问题。

下一课将基于这些实际地址，深入解释 TC375 的本地/全局地址、DSPR/PSPR/DLMU/LMU 和 cached/uncached 映射。
