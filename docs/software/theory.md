# 理论复习

在做实验前，先复习一下网络原理课上讲到的理论，理解一下为什么要实现一个 RIP 协议的路由器，实现的时候需要考虑哪些东西。

## 网络的功能

网络的功能是什么？让两个主机相隔万里也可以通信。如果要用现实生活来比喻，那么就像是全球的快递，可以把货物送到任何一个人的手上。寄过快递或者网购的同学应该都知道，寄件的时候，需要往快递单上写下一个全球唯一的寄件人地址和收件人地址，快递员按照收件人地址，一步一步转运到目的地。

那么，假如你是快递公司，你会如何计划路线，让一个快件从寄件人发送到收件人？如果观察现实生活中快递的路径，经常可以看到这样的流程：

寄件人->寄件人最近的快递站->寄件人所在区->寄件人所在市->寄件人所在省->收件人所在省->收件人所在市->收件人所在区->收件人最近的快递站->收件人

从中间划开，可以看到，快件先从小地方不断转到大地方，再从大地方转到小地方，最后到达终点。每一次转运，都是两种情况：要么是同级转运，比如上面的“寄件人所在省->收件人所在省”，要么是级间转运，比如上面的“寄件人所在区->寄件人所在市”等等。

上面是从全局视角来规划这个事情，但假如，你是某某快递北京市转运中心的员工，对于手上的快递，你要怎么决定，这个快递是给中国转运中心（上一级），还是给天津转运中心（同一级），还是给海淀区转运中心（下一级）呢？

答案是根据收件人的地址进行判断：

1. 首先看收件人是不是北京市的，如果是，那就看是哪个区的，如果是海淀区的，就转到海淀区转运中心（下一级）
2. 如果不是北京的，那就看是不是中国其他省市的，如果是天津市的，那就转到天津市转运中心（同一级）
3. 如果不是中国的，那肯定是到国外的，那就转到中国转运中心，之后再转到国外。

如果每一级快递转运中心都按照这样的逻辑判断，那就不需要一个全局的人来调度了，只需要各转运中心按照自己的逻辑，找到下一家，把快递送到下一家即可。

把上面的这个故事挪到虚拟的网络世界，那么寄件人收件人的地址就是全球可达的 IP 地址，快递转运中心就是各级路由器，转运中心用来判断下一家转运中心的机制就是查路由表。

那么，为了让网络联通，只需要让所有路由器都配置好正确的路由表。但这件事情并不简单，网络中会经常出现变化，总不能让每个路由器旁边都蹲一个人，实时修改配置吧。所以需要一个路由协议，让路由器自己找到正确的路由表。

## RIP 协议的用途

为了让路由器找到正确的路由表，本实验要实现的就是一个简单的路由协议：RIP 协议。简单来说，假如现在要搭建这样一个快递网络：

清华大学 <-> 海淀区 <-> 西城区 <-> 东城区 <-> 故宫

其中海淀区是清华大学的上级，东城区是故宫的上级，海淀区、西城区和东城区平级。海淀区和西城区接壤，西城区和东城区接壤，但是海淀区和东城区不接壤。

那么，我要从清华大学访问故宫的网站，怎么办？第一步很简单，清华大学知道故宫不在海淀区，那么肯定转给自己的上级海淀区。但是海淀区也不知道故宫在哪，海淀区只知道自己和西城区接壤。这时候就需要运行 RIP 协议，让海淀区知道，如果要访问故宫，那就转给西城区。

形象地来说，RIP 协议运行过程如下：

1. 海淀区宣传我这里有清华大学，东城区宣传我这里有故宫
2. 宣传的声音传播距离很短，只有接壤的才能听得到，所以西城区知道海淀区有清华大学，东城区有故宫
3. 然后西城区给东城区宣传，虽然我没有清华大学，但是我可以帮你找到清华大学；西城区给海淀区宣传，虽然我没有故宫，但是我知道故宫怎么走
4. 这时候海淀区知道有故宫这么一个地方，并且去故宫，要先去西城区；东城区知道有清华大学这么一个地方，并且去清华大学，要先去西城区
5. 这时候清华大学和故宫之间的网络就联通了

这就是在接下来实验里要做的。

## 路由表

上面讲到了路由表的重要性，并且可以用路由协议来自动得到一个正确的路由表。那么，路由表具体怎么工作呢？假如要设计一个路由表，要怎么设计？回想上面提到的转运逻辑：

1. 看看收件人是不是我管辖范围下的，如果是，就转给更小范围的转运中心
2. 如果不是我管辖范围下的，但是是属于我邻居的管辖范围，那就转给邻居
3. 否则就转给上一级转运中心

再来回想一下收件人地址是怎么组成的：中国北京市海淀区清华大学，如果对地址进行分段，那就是：

$中国/北京市/海淀区/清华大学$

假如你是北京市转运中心的工作人员，如果要形式化地描述上面的转运逻辑：

1. 看看收件人是不是你管辖范围下的（匹配 $中国/北京市/*$），如果是，就转给更小范围的转运中心（如果匹配 $中国/北京市/海淀区/*$，那么就转给海淀区的转运中心）
2. 如果不是你管辖范围下的（不匹配 $中国/北京市/*$），但是属于你邻居的管辖范围（匹配 $中国/*$），那就转给邻居（如果匹配 $中国/天津市/*$，转给天津市转运中心）
3. 否则就转给上一级转运中心（不匹配 $中国/北京市/*$，不匹配 $中国/*$，匹配 $*$）

根据上面的逻辑，可以很自然地按照地址由大到小进行划分，然后构造上面的“匹配”和“不匹配”条件。但是，一旦条目增加，那么第二条出现不匹配第一条的条件，第三条出现不匹配第一条和第二条的条件，等等，这个算法复杂度就太大了。

换个思路，如果只考虑匹配最精确的那一条，就不需要进行“不匹配”的判断了。比如，$中国/北京市/海淀区/清华大学$ 匹配上面的三条，但是第一条最精确，所以选择第一条；$中国/上海市/浦东新区/浦东机场$，匹配了上面后两条，但是第二条最精确，所以选择第二条；$美国/加利福尼亚州$ 只匹配最后一条，那就是选择第三条。对于算法来说，从多个匹配里找到最精确的，比每次匹配时都要排除掉前面已有的情况，要更加容易。这就是为什么路由表要采用最长前缀匹配。

那么，上面的这个逻辑对应的路由表就像：

```
* -> 中国转运中心
中国/天津市/* -> 天津市转运中心
中国/北京市/海淀区/* -> 海淀区转运中心
```

转运快递的时候，只要在表里寻找最精确的匹配，就可以找到转运的下一个中心。

??? tip "CIDR 表示方法"

    本实验中多次用到了 CIDR 的表示方法来表示一个路由前缀，格式是 ipv6_addr/len，可能表示以下两种意义之一：

    1. 地址是 ipv6_addr，并且最高 len 位和 ipv6_addr 相同的 IPv6 地址都在同一个子网中，常见于对于一个网口的 IP 地址的描述。如 `fd00::a:b/112` 表示 `fd00::a:b` 的地址，同一个子网的地址只有最后 16 位可以不同。
    2. 描述一个地址段，此时 ipv6_addr 除了最高 len 位都为零，表示一个 IP 地址范围，常见于路由表。如 `fd00::a:b/112` 表示从 `fd00::a:0` 到 `fd00::a:ffff` 的地址范围。

## 交换机和路由器的关系

经常有同学在学习的时候，会疑惑交换机和路由器是什么关系？为什么会有三层交换机的说法？

首先，对于网络设备的功能来说，交换机和路由器是很不一样的东西，虽然名字有点像，但是交换机主要工作在数据链路层（二层），针对的主要协议是以太网（Ethernet）；路由器主要工作在网络层（三层），针对的主要协议是 IP。

前面的例子已经讲了，将路由器比喻为快递的转运中心，各个转运中心之间如何按照路由表决定下一跳的路由。但是，这个模型忽略了一个事情，那就是快递总有人要来送，那怎样从海淀区转运中心运到西城区转运中心呢？现实生活中，城市里已经建设好了许多道路，意味着只需要在已有道路上选择一条路径即可。但在网络空间里，道路都是自己建的，需要用一根根网线把网络设备连接起来，那么这时候就要规划这些转运中心之间的道路，要满足有足够的运输能力（带宽），也要考虑建设成本。

还是举北京的例子，海淀区，西城区和石景山区形成一个三角形，两两毗邻。为了让这三个区的转运中心之间有路可走，一种简单的方法是，在两两之间建立一条直路，这样需要建设三条长的路；另一种方法是在三角形中部设置一个集合点，从集合点向三个顶点建设三条比较短的道路。显然，后一种方案建路的长度更短，但是每条路需要建得更宽一些，综合起来还是更优。这还只是三个毗邻的区，如果更多的话，后一种方案的优势就更能体现了。

设想一个电脑教室，几十上百台电脑都需要上网，难道都用网线把它们两两连在一起？更好的方法是，用交换机来做汇聚，用交换机连接许多台电脑，再把交换机连接到其他地方；一个交换机不够，那就再来一个。交换机一个很基本的功能就是简化网络拓扑，并且顺带解决了共享介质的冲突问题（相比一根网线把所有机器串起来），根据其交换表的学习算法，得到一些免费的性能提升（减少了不必要的广播）。

所以，如果从 IP 层来看，交换机是完全透明的，可以当成一条网线，下面连了许多主机。至于三层交换机，其实是市场需求演变出来的产品，很多时候，客户需要的是交换机的多端口汇聚功能和一些简单的路由器功能，因此就把简单的路由器做到了交换机里面了。从抽象的网络功能来看，三层交换机里面是一个交换机 + 路由器的组合，就好像无线路由器是路由器+AP 的组合一样。