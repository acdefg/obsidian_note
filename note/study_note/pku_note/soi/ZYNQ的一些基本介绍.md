## ZYNQ 芯片开发流程
ZYNQ 的开发也是先硬件后软件的方法。具体流程如下：

在 Vivado 上新建 工程，增加一个嵌入式的源文件。

在 Vivado 里添加和配置 PS 和 PL 部分基本的外设，或需要添加自定义的外设。

在 Vivado 里生成顶层 HDL 文件，并添加约束文件。再编译生成比特流文件（ （*.bit ）。

导出 硬件信息 到 SDK 软件开发环境，在 SDK 环境里可以编写一些调试软件验证硬件和软件，结合比特流文件单独调试 ZYNQ 系统。

在 SDK 里生成 FSBL 文件。

在 VMware 虚拟机里生成 u boot.elf 、 bootloader 镜像。

在 SDK 里通过 FSBL 文件 , 比特流文件 system.bit 和 u boot.elf 文件生成一个 BOOT .bin 文件。

在 VMware 里生成 Ubuntu 的内核镜像文件 Zimage 和 Ubuntu 的 根 文件系统。另外还需要要对 FPGA 自定义的 IP 编写驱动。

把 BOOT 、内核、设备树、根文件系统 文件放入到 SD 卡 中，启动开发板电源， Linux 操作系统会从 SD 卡里启动。
                        
原文链接：https://blog.csdn.net/szm1234/article/details/121902787
