

Failure Modes Effects Analysis ：失效模式影响分析

- 返回值超时范围
- 生成的驱动状态无法控制(超出范围)
- 如果读取的值用于数组索引那么可能损坏全局内存



| Possible effects                                                                                                                                                              | Failure Mode Description                                                                                          | Coding Guideline                                                                                                                                                                                                                                    | Examples |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| '- return value is out of range<br>- generate the driver's state is uncontrollable (out of range)<br>- corruption of global memory if read values are used for array indexing | Read/write a bit-field from the hardware peripheral is out of range as described in the hardware Reference Manual | Use DEM module to informed the hard ware issue.<br>The following measures can be used on the situation:<br>- the processing flow might continue with saturated value<br>- the processing flow might continue with masked value<br>- exit processing | 'NA      |



非预期的事件：外设未初始化的情况下，触发中断事件

Memory corruption, segmentation fault  -  Interrupts are triggered before ADC driver is initialized

Driver has  unexpected behavior, CPU stuck in interrupt routine -- Spurious interrupts



处理流程（通常处于繁忙等待状态）会因外围设备刷新硬件状态而被阻塞。

- HW peripheral status used for driver synchronization is frozen.

- Protect the loops waiting for a hardware event against endless iterations using maximum iteration counts
意思就是防止无止境的基于硬件状态的死循环