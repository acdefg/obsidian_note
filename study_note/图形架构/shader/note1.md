### grid block
- **Grid** 是由多个 **Block** 组成的一个计算任务集合。
- 一个 **Block** 会被发配到某一个 **SM** 上执行，一个 SM 上可以承受的 **Block** 数量由硬件资源决定
- 每个 **Block** 有自己的 **Shared Memory**，可以被该 Block 中的所有线程共享使用, 而不同线程块之间无法直接访问彼此的共享内存, **Shared Memory** 提供了高效的线程间数据交换方式，但仅限于同一个线程块内的线程。
- 不同 grid 中的 block 可以在设备上并行
- 每次 block 可以被调度到任意可用的 SM 上，这里调度顺序任意，可以是并发运行的，也可以是顺序运行的
- block 之间没有确认的执行顺序，block 之间相互独立，（针对同一个架构，SM 数量不同的情况，binary 相互之间可以兼容）

### cuda memory space
- global
- shared
- local
- constant

