| TEX | C   | B   | Memory type            | Description, or Normal region Cacheability          | Shareable?             |
| --- | --- | --- | ---------------------- | --------------------------------------------------- | ---------------------- |
| 000 | 0   | 0   | Strongly-ordered       | Strongly ordered                                    | Shareable              |
| 000 | 0   | 1   | Device                 | Shared device                                       | Shareable              |
| 000 | 1   | 0   | Normal                 | Outer and inner Write-Through, no write allocate    | S bit<sup>a</sup>      |
| 000 | 1   | 1   | Normal                 | Outer and inner write-back, no write allocate       | S bit<sup>a</sup>      |
| 001 | 0   | 0   | Normal                 | Outer and inner Non-cacheable                       | S bit<sup>a</sup>      |
| 001 | 0   | 1   | Reserved               | Reserved                                            | Reserved               |
| 001 | 1   | 0   | IMPLEMENTATION DEFINED | IMPLEMENTATION DEFINED                              | IMPLEMENTATION DEFINED |
| 001 | 1   | 1   | Normal                 | Outer and inner write-back; write and read allocate | S bit<sup>a</sup>      |
| 010 | 0   | 0   | Device                 | Non-shared device                                   | Non-shareable          |
| 010 | 0   | 1   | Reserved               | Reserved                                            | Reserved               |
| 010 | 1   | X   | Reserved               | Reserved                                            | Reserved               |
| 011 | X   | X   | Reserved               | Reserved                                            | Reserved               |

### Device Memory & Strongly Ordered Memory
- 非对齐的数据访问会导致不可预期的行为（可能产生MemMange错误）。
- cache不工作
- 可以保证访问到达的顺序和程序指令的顺序一致
- 两者之间的差异：
- Strongly Ordered Memory的写入必须在达到写入的外设或内存时才能完成
    - Device Memory的写入可以在达到外设或内存前就完成

## 缓存策略

### 解释

- Shareable（S bit）: 用于多核间同步更新cache，此功能FC7300没有做支持，可以忽略此配置。
- Outer & Inner : Inner为L1 Cache，Outer为L2 Cache，FC7300只有Inner（L1 Cache）。
- Cacheable下，永远read allocate，read操作可以将memory加载到cache中


### Write Through，no write allocate

每次写操作会同步到内存和缓存，可以确保数据一致性，但写操作的性能较低，因为需要等待数据写入到内存后才能继续执行
#### Write Back, Write allocate:

Cache命中写入Cache后返回，Cache未命中先在Cache中分配空间后写入Cache
#### Write Back, No write allocate:

Cache命中写入Cache后返回，Cache未命中则写入Memory，不会在Cache中分配空间


1. 执行程序的PFLASH：TEX=000, C=1, B=1（Write Back, no write allocate。

**Cache的使用注意事项

1. Cache开启后，需要先Invalid，否则上电后是随机值，可能会导致ECC错误
2. Cache空间有限(I-Cache16K，D-Cache 8K)，尽量避免所有区域开启Cache。
3. Cache命中率很低的情况下，开启cache比不开启运行速度要低，因为多了一个扫描cache的过程。
4. 如果希望开启cache，即便是纯代码运行区域，也要关闭S bit，因为代码中有大量立即数，这个属于数据部分，开启S bit会导致该区域D-Cache失效，影响执行速度
5. 多核共享的易变更的数据区域不要开启cache，不变更数据区域可以开启cache
6. FC7300的flash和rom区域，不支持clean cache，仅支持invalid cache
7. 对于flash区域，如果开启了cache，数据变更后（flash 擦写之后），注意invalid对应区域的 I-Cache或者D-Cache。
8. Cache一次捕获当前对齐的8个word（cache line）的地址上的数据，因此连续运行代码尽可能放在相近代码中，否则会造成cache空间的大量浪费**