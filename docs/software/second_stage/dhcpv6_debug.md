## 本地测试

在编写 DHCPv6 协议的路由器的时候，可以在本机搭建一个虚拟的和评测一样的网络拓扑，这样就可以在虚拟网络中按照评测流程一步一步进行，同时观察程序的行为，可以大大地加快代码的调试。

本地测试主要分为两个部分：首先，搭建网络拓扑，将网络中各个设备连接好；其次，在各个设备上，运行相应的软件，并配置好网络。

网络拓扑的具体过程见[附录中的拓扑搭建文档](../../appendix/topology.md)。

### 快捷脚本

实验仓库中，在 `Setup/dhcpv6` 目录下提供了 netns 配置脚本以及评测脚本，你可以参考里面的内容进行配置。

具体来说，可以按照下列的顺序在本地进行评测：

1. 打开一个命令行窗口，进入 `Setup/dhcpv6/setup` 目录，执行 `sudo ./setup-netns.sh` 配置 netns 环境，然后执行 `sudo ./start-r1.sh` 在 R1 上启动你编写的 DHCPv6 路由器，不要退出；
2. 再打开一个命令行窗口，进入 `Setup/dhcpv6/test` 目录，依次执行 `sudo ./test2.sh`，`sudo ./test3.sh` 到 `sudo ./test5.sh`，对应第二项到第五项测试；
5. 所有项目评测完毕后，可以退出 R1 上的 DHCPv6 路由器。