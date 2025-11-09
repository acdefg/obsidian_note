## 中断处理流程
中断处理过程
**中断响应:**
第一步：处理器执行完当前指令，保存下一条指令的 PC 到 mepc 中。 (在 Rob 进行操作)
第二步：设置 mcause 的中断标记为 1，将中断编号写入 mcause，并更新 mtval 为 0。(在 csr_regs 进行操作) 
第三步：将 mstatus 的中断使能位 MIE 保存到 MPIE 中，将 MIE 清零，禁止响应中断。 （在 csr_regs 进行操作）
第四步：将发生中断之前的权限模式保存到 mstatus 的 MPP 中，切换到机器模式。 (在 csr_regs 进行操作)
第五步（mtvec.Mode=0，直通中断）：PC 从 mtvec.Base 处取指令并执行。通常，取回的指令是一条跳转指令，跳转至顶层处理函数。该函数通过分析 mcause [指令的中断类型]获取中断编号，并调用该编号对应的处理函数。(在 Rob 进行操作)
第五步（mtvec.Mode=1，矢量中断）：PC 从 mtvec.Base + 4 * 中断编号处取指令并执行。通常，取回的指令是一条跳转指令，跳转至相应中断的处理函数。 (在 Rob 进行操作)
**中断返回:**
执行 mret 指令可以实现中断返回。此时，处理器执行下列操作：(在 cp0_regs 进行操作) 
• 将 mepc 恢复到 PC。（mepc 保存的是下一条指令的 PC，所以无需调整） 
• 将 mstatus.MPIE 恢复到 mstatus.MIE。 
• 从 mstatus.MPP 恢复发生中断之前的权限模式。

## Core 内中断处理架构设计方案

核外传入的中断信号会经过SYSIO传入对应的核内，并且拉起CSR_REGS中的机器模式中断等待寄存器mip中对应的bit。

CSR_REGS会查看机器模式中断使能寄存器mie中与mip寄存器拉起的位相对应的位是否拉起、机器模式处理器状态寄存器mstatus中的机器模式保留特权状态位MPP和机器模式中断使能位、机器模式中断降级控制寄存器mideleg、中断优先级，以此来判断中断是否有效和输出中断向量。

DECODE模块接收到来自CSR_REGS的中断信息后会先判断之前是否有中断未处理完毕：（1）若处理完毕，则将该中断信息附着到最年轻的指令上，并且stall住前面的模块，不让指令继续进入DECODE模块。（2）若未处理完毕，则DECODE模块不理会传入的中断，不进行任何操作（即使此时传入的中断比流水线中的中断优先级更高，也不会进行处理，不考虑中断嵌套的情况下）。直到ROB返回一个flush信号或中断处理完毕的complete信号，stall操作会关闭，指令继续流入DECODE模块，DECODE模块将此时最新的CSR_REGS传入的中断信息接收，并重复（1）的操作。

指令进入到DISPATCH模块后，ROB会给DISPATCH中的指令分配iid，此时将携带有中断信息的指令的中断信息传入ROB中对应的entry项内。

指令执行完毕提交给 ROB 后，ROB 会根据 iid 检查对应的 entry 项内是否存在中断信息，若存在，则开始响应中断：（1）将下一条指令的 PC 存入机器模式异常保留程序计数器寄存器 mepc 中。（2）设置机器模式异常事件原因寄存器 mcause 中断标记为 1，向量号写入 mcause、并将机器模式异常事件向量寄存器 mtval 清 0。（3）将机器模式处理器状态寄存器 mstatus 的中断使能位 mie 保存到 mpie 中，将 mie 清零，禁止响应中断。（4）将发生中断之前的权限模式保存到 mstatus 的 MPP 中，切换到机器模式。（5）ROB 读取 CSR_REGS 中的机器模式向量基址寄存器 mtvec，mtvec.Mode=0 时为直通中断，PC 从 mtvec.Base 处取指令并执行。通常，取回的指令是一条跳转指令，跳转至顶层处理函数。该函数通过分析 mcause 获取向量号，并调用该编号对应的处理函数；mtvec.Mode=1 时为矢量中断，PC 从 mtvec.Base + 4 * 向量号 处取指令并执行。通常，取回的指令是一条跳转指令，跳转至相应中断的处理函数。

## ref
不错
[RISC-V 特权指令结构 - orangeQWJ - 博客园](https://www.cnblogs.com/orangeQWJ/p/15912780.html)
也还行
[https://zhuanlan.zhihu.com/p/655029162#:\~:text=在APLIC中，hart被划分为一个或多个中断域（interrupt domain）。 一个中断域内的中断投递模式相同（可以是线连接或者MSI），投递目标hart的特权级也相同。,如果hart侧没有实现IMSIC，一个hart的一个特权级只能属于一个中断域。 当APLIC收集到中断时，所有的中断首先由根中断域（root domain）处理。](https://zhuanlan.zhihu.com/p/655029162#:~:text=%E5%9C%A8APLIC%E4%B8%AD%EF%BC%8Chart%E8%A2%AB%E5%88%92%E5%88%86%E4%B8%BA%E4%B8%80%E4%B8%AA%E6%88%96%E5%A4%9A%E4%B8%AA%E4%B8%AD%E6%96%AD%E5%9F%9F%EF%BC%88interrupt%20domain%EF%BC%89%E3%80%82%20%E4%B8%80%E4%B8%AA%E4%B8%AD%E6%96%AD%E5%9F%9F%E5%86%85%E7%9A%84%E4%B8%AD%E6%96%AD%E6%8A%95%E9%80%92%E6%A8%A1%E5%BC%8F%E7%9B%B8%E5%90%8C%EF%BC%88%E5%8F%AF%E4%BB%A5%E6%98%AF%E7%BA%BF%E8%BF%9E%E6%8E%A5%E6%88%96%E8%80%85MSI%EF%BC%89%EF%BC%8C%E6%8A%95%E9%80%92%E7%9B%AE%E6%A0%87hart%E7%9A%84%E7%89%B9%E6%9D%83%E7%BA%A7%E4%B9%9F%E7%9B%B8%E5%90%8C%E3%80%82,%E5%A6%82%E6%9E%9Chart%E4%BE%A7%E6%B2%A1%E6%9C%89%E5%AE%9E%E7%8E%B0IMSIC%EF%BC%8C%E4%B8%80%E4%B8%AAhart%E7%9A%84%E4%B8%80%E4%B8%AA%E7%89%B9%E6%9D%83%E7%BA%A7%E5%8F%AA%E8%83%BD%E5%B1%9E%E4%BA%8E%E4%B8%80%E4%B8%AA%E4%B8%AD%E6%96%AD%E5%9F%9F%E3%80%82%20%E5%BD%93APLIC%E6%94%B6%E9%9B%86%E5%88%B0%E4%B8%AD%E6%96%AD%E6%97%B6%EF%BC%8C%E6%89%80%E6%9C%89%E7%9A%84%E4%B8%AD%E6%96%AD%E9%A6%96%E5%85%88%E7%94%B1%E6%A0%B9%E4%B8%AD%E6%96%AD%E5%9F%9F%EF%BC%88root%20domain%EF%BC%89%E5%A4%84%E7%90%86%E3%80%82)