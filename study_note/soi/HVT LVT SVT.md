
[芯片工艺库中的ffg、ssg、ttg和HVT、LVT、SVT（RVT）_svt和lvt有什么区别_Ocean_VV的博客-CSDN博客](https://blog.csdn.net/qq_41634276/article/details/126979760)

HVT LVT SVT 是指工艺库中可提供的 cell 类型，HVT 表示高阈值电压，功耗低（因为 low leakage）、速度慢，LVT 表示低阈值电压，功耗高但速度快，SVT（也有叫 RVT）居中。一般在后端优化过程中，会根据 timing 情况，自动使用上述的几种 cell 类型，timing 紧的地方就选用 LVT，timing 比较松的地方就是用 HVT，即在满足 timing 的前提下，尽量使用 HVT cell，降低功耗