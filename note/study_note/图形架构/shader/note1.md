### grid block
- **Grid** 是由多个 **Block** 组成的一个计算任务集合。
- 一个 **Block** 会被发配到某一个 **SM** 上执行，一个 SM 上可以承受的 **Block** 数量由硬件资源决定
- 每个 **Block** 有自己的 **Shared Memory**，可以被该 Block 中的所有线程共享使用, 而不同线程块之间无法直接访问彼此的共享内存, **Shared Memory** 提供了高效的线程间数据交换方式，但仅限于同一个线程块内的线程。
- 不同 grid 中的 block 可以在设备上并行
- 每次 block 可以被调度到任意可用的 SM 上，这里调度顺序任意，可以是并发运行的，也可以是顺序运行的
- block 之间没有确认的执行顺序，block 之间相互独立，（针对同一个架构，SM 数量不同的情况，binary 相互之间可以兼容）

### cuda memory space
- global: 缓存空间 L1
- shared: shared memory
- local： thread 私有
- constant

### stream
stream 适用于并发 grid 的模型，grid 的可以并行执行，也可以存在依赖关系，但不同 stream 中的操作可以同时执行，前提是没有数据依赖关系
[一文读懂cuda stream与cuda event](https://zhuanlan.zhihu.com/p/699754357)

### compute subsystem
FEC： firmware context-level scheduling

FE：grid-level scheduling（device level）
FE 硬件任务调度单元，这里的 task 可以是 grid 或者一些 command，我们今天先只关心 grid。
FE 会从显存总加载 task 描述信息，并解析其中的一部分内容用作 grid 调度，处理 grid 依赖关系及优先级并把选出执行的 grid 发给 CDM，还要统计 grid 完成的情况。

CDM：block-level scheduling （core level）
FE 下游的模块被称作 CDM，负责解析 control stream 中关于 grid 的信息，把 grid 按照一定规则生成 block，并选择 SM 进行 launch。
怎么选择 SM：
CDM 通过和 PDS 握手来观察 SM 上可供分配的 workgroup slots，PDS 需要计数并上报给 CDM，CDM 会对 SM 做 load balance 处理。

PDS：MP scheduling
到一个 block 之后开始组 warp，一个 warp 就是 32 thread，PDS 要进行 SM partition 的选择（load balance），进行资源管理并上报 block 完成情况。

stream processor：thread-level scheduling
SM 是 warp 执行的硬件单元，每个 warp 都会按照取值译码发射执行写回来处理每条指令。

任务完成的 flow 刚好和 launch 相反，SM report warp end 给 PDS，PDS report block end 给 CDM，CDM 进而 report end 给 FE，FE 在 grid 结束后可以更新 sem 并开始调度有依赖的 grid

### warp 调度
同一个 warp 内的线程是 SIMD，执行同样的指令
不同的 warp 是 SIMT，针对分支需要通过 active mask 选择对应的线程去执行

Warp 调度的目的是从“满足执行条件”的 warp 中选出适合执行的 warp 来发射指令，这是一个复杂的过程，我们先略过细节，看看单个 warp 的处理，**需要兼顾局部性和负载均衡性**

一个 warp 在等待时（延迟长或者等到数据依赖），scheduler 会切换其他的 warp


