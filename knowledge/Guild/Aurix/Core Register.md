
![[Pasted image 20260826223944.png]]

PCXI用于存储CPU上下问的，FCX与LCX也是。

BIV：中断向量表的起始地址
BTV：Trap向量表的起始地址

ISP：中断堆栈指针，取决于PSW中的某一配置，user stack point和interrupt stack point是否是分开的。如果没有分开则ISP这个是没有用的。

堆栈指针一般存于A10寄存器。

访问内核寄存器必须通过特殊的指令去访问
	MTCR 去写内核寄存器
	MFCR 去读内核寄存器

![[Pasted image 20260826224622.png]]



A11类似LR

