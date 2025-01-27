# 抄袭与查重

!!! fail "禁止抄袭"

    正如创建仓库时的提示，你提交到 GitLab 上的 **所有代码** 将被作为查重的依据，进行横向和纵向的交叉查重，代码库包括了本实验框架启用以来的所有代码。恶意篡改 Git Commit 时间也视为作弊（比如在 DDL 之后把时间调到 DDL 之前）。一旦确认作弊，你将失去 **实验部分所有的分数** 。请三思而后行！

根据[《清华大学学生纪律处分管理规定实施细则》](https://www.tsinghua.edu.cn/info/1094/82878.htm)：

- 第六章 学术不端、违反学习纪律的行为与处分
    - 第二十一条 有下列违反课程学习纪律情形之一的，给予警告以上、留校察看以下处分：
        - （一）课程作业抄袭严重的；
        - （二）实验报告抄袭严重或者篡改实验数据的；
        - （三）期中、期末课程论文抄袭严重的；
        - （四）在课程学习过程中严重弄虚作假的其他情形。

请同学们三思而后行。

下面是一些关于抄袭与查重的常见问题：

!!! question "到底怎么样才算抄袭？"

    我们首先将使用工具进行自动化查重，并根据结果与相关同学进行面谈，最终由课程教师给出判定。判定的标准是“代码的相似程度是否超出了（不交流具体代码的）正常范围”。由于“是否直接发送代码”是我们无法验证的行为，用这个标准处理是不现实的。对于“大范围交流某次作业的过多实现细节”或者“口述每一行、函数的每一个参数用法和形式”这样的情况，客观后果和“直接发送代码”并无区别。

!!! question "这是希望同学们完全独立完成作业，不能交流吗？"

    我们依然持续鼓励同学间的交流和帮助，但直接发送代码（无论是否完整、是文本或截图）、直接帮助同学编写代码、以过细的粒度指导同学（如口述每一行、函数的每一个参数用法和形式，这样的行为极其容易导致大片重复）都是不可取的。

!!! question "虽然确实和同学交流了代码，但我真的付出了很大努力才完成作业"

    《计算机网络原理》是计算机科学的核心内容，一些基础较为薄弱的同学需要付出更多努力。教学团队尽可能地提供了答疑和支持。如果你确实无法在规定时间内通过自己的努力独立完成作业，请联系教学团队寻求更多帮助和建议。

!!! question "我完全独立地完成了作业，只是因为好心教了其他同学导致出现了代码雷同"

    非常感谢你为助教分担答疑任务，但我们不鼓励直接对代码形式进行教学。尤其考虑到具体实现会限制其他同学的思路，并不对同学培养解决问题的能力具有显著的作用。希望你下次可以把相应的文档、RFC 等发送给他，并且解释的时候尽量屏蔽自己具体实现细节。这对答疑的同学提出了更高的要求，但我们也相信这对双方都是更难但更有收获的方式。

## 典型案例

本课程的案例：

1. 一位同学在做实验的时候，自己编写的代码没有通过所有的测试，在 DDL 后将往年同学公布在 GitHub 的代码原样复制到自己的仓库中提交。
2. DDL 前，A 同学给 B 同学讲解了代码，但 B 同学依然没有搞懂，就从 A 同学处要了一份代码，结果 B 同学直接把 A 同学的代码提交上去。
3. A 同学对着代码，给同学讲解了中厅讲座，并进行了录像，之后 B 同学对着录像把 A 同学的代码摘抄下来提交。

[其他课程的案例](http://jwcbg.cic.tsinghua.edu.cn/jwcbg/detail_cat.jsp?boardid=57&seq=8424)：

```
自动化系某2021级本科生，在2021-2022学年秋季学期某门课程作业中抄袭其他同学的源代码文件。
根据《清华大学学生纪律处分管理规定实施细则》（清校发〔2019〕14号）第二十一条规定，决定给予其警告处分。
```