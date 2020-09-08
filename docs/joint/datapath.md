# 第三部分：数据通路设计

## ARP 表自动学习

在 Loopback 基础上，对收到的以太网帧进行解析，判断它的 EtherType （跳过 VLAN），如果是一个 ARP 请求，则提取出字段，然后插入到 ARP 表中。在仿真中观察学习到的 ARP 表项，然后用 ILA 在板子上观察实际学习到的内容。

你可以用 arping 命令来不断发送 ARP 请求，这样有助于调试。如果不想用 ILA，也可以把信号接到 LED 上，但注意如果变换的频率过快，人眼是不能捕捉到 LED 的亮度变化的。

!!! question

    1. ARP 的 EtherType 在哪个偏移？
    2. OS 一般会在什么情况下发送 ARP 到你的路由器？
    3. 如何在 OS 添加一条指向你的路由器的路由？

## ARP 自动回复

在本实验中，你可以把 ARP 回复实现在硬件中，也可以实现在软件中。实现在硬件中的好处是方便调试，不需要等到软件写好再调，坏处时如果软件需要访问 ARP 表则需要额外的工作。如果要在硬件中写，则需要对 ARP 请求进行判断，如果目标 IP 地址是你的路由器在对应 VLAN 的对应的 IP 地址（自己定，VLAN 通过 VLAN Tag 读出），则构造一个回应然后发出去，而不是原来的 Loopback。回应中，需要注意 MAC 地址和 VLAN Tag 的填写。

如果实现正确，通过 arping 应该可以得到稳定的低延迟的回复。路由器的 IP 地址和 MAC 地址可以自己定，但建议在标准中用于自由分配的段中进行分配。

!!!! question

    1. 哪些 ARP 类型不需要回应？
    2. 如何在构造的 ARP 回应和交换 MAC 地址的原数据二者之间进行选择？

## 路由查询和转发

在接受到 IP 包的时候，需要判断是否应该转发，如果应该转发，需要通过查表得知下一跳的信息，再根据下一跳查询到下一跳的 MAC 地址。由于查询过程和接收过程是同时发生的，你需要考虑如何做到高效，尽量保证在需要转发的时候，所需要的信息都已经准备好，可以立即发出去。发出去的时候要进行字段的修改，包括 TTL Checksum，MAC 地址和 VLAN Tag。

具体过程：解析 EtherType 和 Dst IP，判断是否应该转发，如果转发则查询转发表，如果查到了，则得到了下一跳和下一条所在的网口。接着，用下一跳地址（直连路由特殊处理）查询 ARP 表，得到目的 MAC 地址，然后修改 TTL 和 Checksum，再发出。

在仿真中仔细阅读每一步查询的结果和更新后的内容，比对一下各个字段，主要是 MAC 地址、VLAN Tag、TTL 和 Checksum。如果看起来没有问题，可以尝试按照写死的转发表，搭建一个测试网络，用两台电脑互相 ping，抓包查看具体情况。这个时候，ILA 会比较有用，可以看到中间过程每一步查表的结果。另外建议用 Linux 系统进行测试，因为它可以关掉许多不相关的流量，让调试变得比较方便。

!!! question

    1. 如果查询转发表，没有找到，应该怎么处理？
    2. 如果查询 ARP 表，没有找到，应该怎么处理？
    3. 如果 TTL 减到了 0，应该怎么处理？
    4. 增量更新 Checksum 的话，会出现哪些特殊情况（提示：RFC 也犯过的错）？

## 转发正确性测试和压力测试

在上一步能 ping 通的基础上，还需要进一步的正确性测试和压力测试。

正确性测试主要包括：Checksum 各种情况都能正确计算，可以通过减少 ping 的间隔来快速测得；大包也可以正常转发，可以通过 ping 指定 payload 大小或者用 iperf3 测速得到

压力测试：通过 iperf3 进行测速，如果速度出现突然降到零然后不再能 ping 通的情况，一般说明部分逻辑不够鲁棒，或者转发速度跟不上接收速度时，丢包的逻辑不能正确地完整丢包。这种时候，可以通过 ILA 查看内部状态，找到出问题的地方，也可以在 testbench 中增加发包频率，模拟压力测试的情况，然后找到出问题的地方。

!!! question

    1. AXI-Stream RX 是没有 tready 信号的，意味着一直会接收数据，怎么实现一个缓冲区？
    2. 在实现一个缓冲区的基础上，如何保证丢包的正确性？