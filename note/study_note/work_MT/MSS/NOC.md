![[image-16.png]]

~~~
流水线阶段（典型 3-4 周期）：

Cycle 0:  Head Flit 到达输入端口
          │
Cycle 1:  [RC Stage] 路由计算 → 确定输出端口 (East)
          │
Cycle 2:  [VA Stage] VC 分配 → 在 East 方向分配 VC1
          │                    同时查询该 VC 的 credit
          │
Cycle 3:  [SA Stage] Switch 分配 → 请求 Crossbar
          │
Cycle 4:  [ST Stage] 通过 Crossbar 传输到 East 输出端口
          │
          Body Flit 跟随同一 VC，无需重新 VA
~~~

VC allocator：
![[image-17.png|227x377]]

input->output：对 P 个 port V 个 virtual channel，一共有 PV 个 V:1 的 arbiter，V 个下游的 virtual channel 里面选择一个来接收
output：对 P 个 port V 个 virtual channel，一共有 PV 个 PV:1 的 arbiter，前一级每个 VC 都会往这里发信息，需要仲裁 PV：1

SA：
![[image-18.png|273x363]]
区别是按照 P 来看的