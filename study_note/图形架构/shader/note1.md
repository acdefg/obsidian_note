### grid block
- **Grid** 是由多个 **Block** 组成的一个计算任务集合。
- 一个 **Block** 会被发配到某一个 **SM** 上执行，一个 SM 上可以承受的 **Block** 数量由硬件资源决定
- 每个 **Block** 有自己的 **Shared Memory**，可以被该 Block 中的所有线程共享使用。
- **Shared Memory** 提供了高效的线程间数据交换方式，但仅限于同一个线程块内的线程。
- **Grid** 中的每个线程块都拥有自己的 **Shared Memory**，而不同线程块之间无法直接访问彼此的共享内存。