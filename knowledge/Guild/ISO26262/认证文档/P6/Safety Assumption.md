# FC4150 MCAL Safety Assumption

> 文档编号: FC-QP-FSM-P4-021 | 版本: A5 | 状态: Approved

---

## 1. Purpose

定义MCAL作为SEooC开发的流程，由于MCAL开发早于客户系统且用于多个系统，因此对技术安全需求和概念做出假设作为MCU安全软件架构定义的起点。

## 2. Scope

- **项目**: FC4150 MCAL (FC-FCSW002-FSMB)
- **安全等级**: ISO26262 ASIL B
- **目标IC**: FC4150系列 (FC4150F512, FC4150F1M, FC4150F2M，含100/144/176引脚封装)
- **模块**: FEE, CAN, LIN, SPI, ETH, MCU, WDG, GPT, FLS, ADC, DIO, PORT, PWM, ICU, Crypto, Common, I2C, TrgSel, DMA

## 3. SEooC软件组件范围定义

### 3.1 目的假设

假想我们要做的有哪些东西，general speaking

- MCAL按AUTOSAR 4.3.1定义划分为: Memory、Communication、Microcontroller、Memory、I/O、Crypto Drivers等子系统
- 每个MCAL模块为独立软件组件，进行集成测试和单元测试
- 使用EB Tresos工具进行配置和代码生成

### 3.2 外部接口假设

假想有哪些外部的接口，与外部的软件接口的联系

| 假设ID | 假设内容 |
|--------|----------|
| SWR_FSA_001 | MCAL不定义安全状态，异常时返回错误由客户进入系统级安全状态 |
| SWR_FSA_002 | 通信驱动需通过AUTOSAR标准BSW通信接口模块调用 |
| SWR_FSA_003 | FEE/FLS需通过NVM→Memif→FEE→FLS链路调用 |
| SWR_FSA_004 | WDG需通过WdgM和WdgIf模块调用 |
| SWR_FSA_005 | MCU在启动阶段由客户直接调用API |
| SWR_FSA_006 | DMA/IIC/TRGSEL由RTE层直接调用 |
| SWR_FSA_007 | I/O驱动需通过HwIOAbs模块调用 |
| SWR_FSA_054 | 客户使用DEM报告MCAL运行错误并存储至NVM |
| SWR_FSA_055 | DET用于开发调试阶段，生产阶段应移除 |

### 3.3 外部环境假设

假设给谁用的，谁来用

| 假设ID | 假设内容 |
|--------|----------|
| SWR_FSA_008 | 适用于FC4150系列IC |
| SWR_FSA_009 | 使用EB Tresos Studio进行配置和代码生成 |
| SWR_FSA_010 | 默认配置适用于Flagchip FC4150开发板 |
| SWR_FSA_011 | 适用于AUTOSAR 4.3.1，其他版本需禁用版本检查 |

## 4. SEooC安全要求假设

安全机制安全状态的假想

| 假设ID | 假设内容 |
|--------|----------|
| SWR_FSA_012 | MCU安全状态: 正常运行、RESET、断电；持续复位不属于安全状态 |
| SWR_FSA_013 | 系统需能自恢复到安全状态 |
| SWR_FSA_014 | MCAL不提供切换安全状态API，仅返回错误和报告诊断事件 |
| SWR_FSA_015 | 客户需在MCAL返回错误时切换系统到安全状态 |

### 4.1 内部安全要求

假想要做ASILx的东西

所有模块(FEE/CAN/LIN/SPI/ETH/MCU/WDG/GPT/FLS/ADC/DIO/PORT/PWM/ICU/Crypto/Common/I2C/TrgSel/DMA)均按ASIL B开发，遵循各自AUTOSAR SRS规范。

### 4.2 外部安全要求

假想用户应该怎样来安全的使用我们提供的东西

| 假设ID | 假设内容 |
|--------|----------|
| SWR_FSA_034 | 通信驱动需E2E保护(滚动计数器、校验和、CRC) |
| SWR_FSA_035 | NVM数据需CRC校验 |
| SWR_FSA_036 | 模块初始化后需进行配置寄存器检查 |
| SWR_FSA_037 | 需按预配置周期喂狗 |
| SWR_FSA_038 | DMA传输需比较数据CRC并检查总数据量 |
| SWR_FSA_039 | ADC启动时需与内部已知电压比较校验 |
| SWR_FSA_040 | DMA访问范围需受限于源/目标内存范围 |
| SWR_FSA_041 | 需检测和处理中断溢出 |
| SWR_FSA_042 | 需提供适当的异常响应 |
| SWR_FSA_043 | 需提供相同ASIL级别的通知或回调处理 |
| SWR_FSA_044 | 需在系统级执行相关性失效分析(DFA) |

## 5. 法律和标准要求

- ISO 26262 V2018 (ASIL B)
- AUTOSAR V4.3.1

## 6. 安装、调试、维护要求

假想用户拿到我们的东西之后应该怎么样才能用起来

| 假设ID | 假设内容 |
|--------|----------|
| SWR_FSA_047 | 客户需加载MCAL配置文件到EB Tresos并配置数据项 |
| SWR_FSA_048 | 配置通过工具安全检查后生成动态代码文件 |
| SWR_FSA_049 | 客户需执行SW系统集成 |
| SWR_FSA_050 | 使用支持功能安全的IAR编译器 |
| SWR_FSA_051 | 或使用支持功能安全的Green Hills编译器 |
| SWR_FSA_052 | 使用支持功能安全的调试器工具 |
| SWR_FSA_053 | 各驱动模块初始化函数需在启动阶段按序执行，周期函数需按周期运行 |

## 7. 运行要求

NA
