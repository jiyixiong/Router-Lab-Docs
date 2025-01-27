## 网络拓扑

在 TANLabs 提交后，会在实验室的实验网络下进行真机评测。你的软件会运行在树莓派上，和平台提供的机器组成如下的拓扑：

![Topology](img/topology_ripng.png)

这一阶段，PC1、R1、R3、PC2 都由 TANLabs 自动配置和提供，两台路由器上均运行 BIRD 作为标准的路由软件实现。你代码所运行在的树莓派处于 R2 的位置。其中 R2 实际用到的只有两个口，剩余两个口按顺序配置为 `fd00::8:1/112` 和 `fd00::9:1/112` （见 `Router-Lab/Homework/router/main.cpp` 代码中 ROUTER_R2 部分）。初始情况下，R1 和 R3 先不启动 RIP 协议处理程序，这些机器的系统路由表如下：

```text
PC1:
default via fd00::1:1 dev pc1r1
fd00::1:0/112 dev pc1r1 scope link
R1:
fd00::1:0/112 dev r1pc1 scope link
fd00::3:0/112 dev r1r2 scope link
R3:
fd00::4:0/112 dev r3r2 scope link
fd00::5:0/112 dev r3pc2 scope link
PC2:
default via fd00::5:2 dev pc2r3
fd00::5:0/112 dev pc2r3 scope link
```

上面的每个网口名称格式都是两个机器拼接而成，如 `pc1r1` 代表 pc1 上通往 r1 的网口。此时，接上你的树莓派，按照图示连接两侧的 R1 和 R3 后，从 PC1 到 PC2 是不通的。接着，在 R1 和 R3 上都开启 RIPng 协议处理程序 BIRD，它们分别会在自己的 RIPng 报文中宣告 `fd00::1:0/112` 和 `fd00::5:0/112` 的路由。一段时间后它们的路由表（即直连路由 + RIPng 收到的路由）应该变成这样：

```text
R1:
fd00::1:0/112 dev r1pc1 scope link
fd00::3:0/112 dev r1r2 scope link
fd00::4:0/112 via fd00::3:2 dev r1r2
fd00::5:0/112 via fd00::3:2 dev r1r2
fd00::8:0/112 via fd00::3:2 dev r1r2
fd00::9:0/112 via fd00::3:2 dev r1r2
R3:
fd00::1:0/112 via fd00::4:1 dev r3r2
fd00::3:0/112 via fd00::4:1 dev r3r2
fd00::4:0/112 dev r3r2 scope link
fd00::5:0/112 dev r3pc2 scope link
fd00::8:0/112 via fd00::4:1 dev r3r2
fd00::9:0/112 via fd00::4:1 dev r3r2
```

在 Linux 中，上面的 `via fd00:3:2` 和 `via fd00::4:1` 在实际情况下会变成对应的 fe80 开头的 Link Local 地址。

在代码中，已经为大家设置好了一些常量，根据 ROUTER_R{1,2,3} 宏的不同定义取不同的值。在 `Homework/router` 目录下可以看到名为 `r1` `r2` `r3` 的目录，每个目录下都有 Makefile，分别编译出不同宏定义下的路由器。

## 检查内容

评测时 TANLabs 将会自动逐项检查下列内容：

1. 配置网络拓扑，在 R1 和 R3 上运行 BIRD，在 R2 上运行定义了 `ROUTER_R2` 的 RIPng 路由器程序。
2. 等待 RIPng 协议运行一段时间，一分钟后正式开始评测。
3. （15% 分数）测试转发 ICMPv6：在 PC1 上 `ping fd00::5:1` 若干次，在 PC2 上 `ping fd00::1:2` 若干次，测试 ICMP 连通性。
4. （15% 分数）测试转发 TCP：在 PC1 和 PC2 上各监听 80 端口（`sudo nc -6 -l -p 80`），各自通过 nc 访问对方（`nc $remote_ip 80`），测试 TCP 连通性。
5. （20% 分数）判断路由表正误：导出 R1 和 R3 上 系统的路由表（运行 `ip -6 route`），和答案进行比对。
6. （25% 分数）测试 HopLimit 降为 0 时的处理：在 PC1 上 `ping fd00::5:1 -t 2`，应当出现 `Time Exceeded` 的 ICMPv6 响应，从 PC2 同样地运行 `ping fd00::1:2 -t 2`。
7. （15% 分数）测试 ICMP Echo Request 的处理：在 PC1 上 `ping fd00::3:2`，在 PC2 上运行 `ping fd00::4:1`，应当得到响应。
8. （10% 分数）测试转发性能：在 PC2 上运行 `iperf3 -s`，在 PC1 上运行 `iperf3 -c fd00::5:1 -O 5 -P 10`，按照 Bitrate 给出分数，测试转发的效率。

代码量：实现 RIPng 协议约 80 行，实现 HopLimit 降为 0 处理约 40 行，实现 ICMPv6 Echo Reply 响应约 20 行，实现 ICMPv6 Destination Unreachable 响应约 40 行。

在过程中，如果路由器程序崩溃退出，后续的测试项目都会失败。在实验平台上，可以看到功能和性能的原始评测结果，不公布最终折算成绩。需要注意的是，GNU netcat 不支持 IPv6，请使用 netcat-openbsd。

设转发性能为 $s$，所有同学中转发的最高效率为 $s_{max}$，则性能分数 $S$ 为：

$$
S = S_{total} \times e^{c \times (s/s_{max}-1)}
$$

其中 $c$ 为未知常数。由于性能会计入分数，请在通过所有功能测试后，检查一下是否删除了不必要的影响性能的调试代码。

??? question "为何不在 R2 上配置 IP 地址：fd00::3:2 和 fd00::4:1"

    1. Linux 有自己的网络栈，如果配置了这两个地址，Linux 的网络栈也会进行处理，如 ICMP 响应和（可以开启的）转发功能
    2. 实验中你编写的路由器会运行在 R2 上，它会进行 ICMP NDP 响应（HAL 代码内实现）和 ICMP Echo 响应（你的代码实现）和转发（你的代码实现），实际上做的和 Linux 网络栈的部分功能是一致的
    3. 为了保证确实是你编写的路由器在工作而不是 Linux 网络栈在工作，所以不在 R2 上配置这两个 IP 地址
    4. 作为保险，HAL 在 Linux 下会关闭对应 interface 的 IPv6 功能。

??? warning "容易出错的地方"

    1. Metric 计算和更新方式不正确或者不在 [1,16] 的范围内；
    2. 没有正确处理 RIP Response，特别是 metric=16 的处理，参考 [RFC 2080 Section 2.4.2 Response Messages](https://www.rfc-editor.org/rfc/rfc2080.html#section-2.4.2)；
    3. 转发的时候查表 not found，一般情况是路由表出错，或者是查表算法的问题；
    4. 更新路由表的时候，查询应该用精确匹配，但是错误地使用了最长前缀匹配；
    5. 没有对所有发出的 RIP Response 正确地实现水平分割和毒性反转；
    6. 字节序不正确，可以通过 Wireshark 看出。

??? example "可供参考的例子"

    我们提供了 [`r1.pcap`](static/r1.pcap) 和 [`r3.pcap`](static/r3.pcap) （可点击文件名下载）这两个文件，分别是在 R1 和 R3 抓包的结果，模拟了实验的过程：

    1. 开启 R1 R3 上的 BIRD 和 R2 上运行的路由器实现
    2. 使用 ping 进行了若干次连通性测试

    举个例子，从 PC1 到 PC2 进行 ping 连通性测试的网络活动（忽略 RIP）：

    1. PC1 要 ping fd00::5:1，查询路由表得知下一跳是 fd00::1:1。
    2. 假如 PC1 还不知道 fd00::1:1 的 MAC 地址，则发送 NDP 请求（通过 pc1r1）询问 fd00::1:1 的 MAC 地址。
    3. R1 接收到 NDP 请求，回复 MAC 地址（r1pc1）给 PC1（通过 r1pc1）。
    4. PC1 把 ICMPv6 报文发给 R1，目标 MAC 地址为上面 NDP 请求里回复的 MAC 地址，即 R1 的 MAC 地址（r1pc1）。
    5. R1 接收到 IPv6 分组，查询路由表得知下一跳是 fd00::3:2，假如它已经知道 fd00::3:2 的 MAC 地址。
    6. R1 把 IPv6 分组外层的源 MAC 地址改为自己的 MAC 地址（r1r2），目的 MAC 地址改为 fd00::3:2 的 MAC 地址（R2 的 r2r1），发给 R2（通过 r1r2）。
    7. R2 接收到 IPv6 分组，查询路由表得知下一跳是 fd00::4:2，假如它不知道 fd00::4:2 的 MAC 地址，所以丢掉这个 IPv6 分组。
    8. R2 发送 NDP 请求（通过 r2r3）询问 fd00::4:2 的 MAC 地址。
    9. R3 接收到 NDP 请求，回复 MAC 地址（r3r2）给 R2（通过 r3r2）。
    10. PC1 继续 ping fd00::5:1，查询路由表得知下一跳是 fd00::1:1。
    11. PC1 把 ICMPv6 报文发给 R1，目标 MAC 地址为 fd00::1:1 对应的 MAC 地址，源 MAC 地址为 fd00::1:2 对应的 MAC 地址。
    12. R1 查表后把 ICMPv6 报文发给 R2，目标 MAC 地址为 fd00::3:2 对应的 MAC 地址，源 MAC 地址为 fd00::3:2 对应的 MAC 地址。
    13. R2 查表后把 ICMPv6 报文发给 R3，目标 MAC 地址为 fd00::4:2 对应的 MAC 地址，源 MAC 地址为 fd00::4:1 对应的 MAC 地址。
    14. R3 查表后把 ICMPv6 报文发给 PC2，目标 MAC 地址为 fd00::5:1 对应的 MAC 地址，源 MAC 地址为 fd00::5:2 对应的 MAC 地址。
    15. PC2 收到后响应，查表得知下一跳是 fd00::5:2。
    16. PC2 把 ICMPv6 报文发给 R3，目标 MAC 地址为 fd00::5:2 对应的 MAC 地址，源 MAC 地址为 fd00::5:2 对应的 MAC 地址。
    17. R3 把 ICMPv6 报文发给 R2，目标 MAC 地址为 fd00::4:1 对应的 MAC 地址，源 MAC 地址为 fd00::4:2 对应的 MAC 地址。
    18. R2 把 ICMPv6 报文发给 R1，目标 MAC 地址为 fd00::3:1 对应的 MAC 地址，源 MAC 地址为 fd00::3:2 对应的 MAC 地址。
    19. R1 把 ICMPv6 报文发给 PC1，目标 MAC 地址为 fd00::1:2 对应的 MAC 地址，源 MAC 地址为 fd00::1:1 对应的 MAC 地址。
    20. PC1 上 ping 显示成功。


!!! tips "这个实验是怎么设计出来的？"

    第二阶段实验首先是确定了一个网络拓扑，其次才是在网络中测试路由器的功能。如果只有两个路由器，一个运行标准实现，一个运行同学的实现，那就无法判断同学实现的路由器是否正确地把学习到的路由再发送出去，因此最少需要三个路由器。此外，为了模拟终端用户，也是在 R1 R3 的两侧分别设置了两个客户端 PC1 和 PC2，它们对网络中的路由器如何配置是不关心的，它们只知道默认路由，并且可以通过默认路由来访问对方，这也模拟了最常见的网络访问形式。因此，网络拓扑设计成了 PC1-R1-R2-R3-PC2 这样的形式。实际上可以设计更复杂的网络拓扑，但是为了减少同学们理解的负担，就简化成一条线的形式。

    对于如何测试路由器的功能，也是采用了网络中常见的手段：ping，traceroute，iperf 等等。ping 就是考察了路由表的正确性和转发功能；traceroute 在实验中通过设置不同 hlim 来实现，考察了当 hlim 降为 0 时的处理；iperf 则是让同学对网络性能有一个概念，并且知道代码中哪些部分对性能影响大，哪些部分影响不大。

    这一阶段还希望大家理解和掌握网络模拟和调试的一些方法和工具：wireshark 和 netns。首先是希望同学掌握网络中的调试方法，之后在遇到网络不通的时候，不再是毫无方向地尝试各种方法，而是按照一定的思路，查看网络各处的状态，排除出问题所在。其次是希望同学接触一些比较先进的网络技术，比如 netns，可以在 linux 环境中模拟各种各样的网络，也可以让大家了解常见的容器技术是如何实现网络隔离的。如果同学不感兴趣具体原理，也可以直接用提供好的脚本来搭建基于 netns 的网络环境。