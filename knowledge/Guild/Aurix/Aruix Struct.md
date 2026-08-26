

各个核都有各自的 PSPR, DSPR, PCACHE, DCACHE, DLMU

对于PFLASH 使用80000000去CACHE访问，使用A000000去非CACHE访问

本地的DLMU和全局的LMU映射在一块连续的地址上，使用90000000去CACHE访问，使用B0000000去非CACHE访问

访问自己私有的DSPR和PSPR是不需要CACHE的。

[[Core Register]]


[[上下文处理]]

[[中断]]

[[Trap]]