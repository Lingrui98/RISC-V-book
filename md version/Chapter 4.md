第四章 乘法和除法指令

![](media/image1.emf){width="1.2497692475940507in" height="2.25in"}

若非必要，勿增实体。------奥卡姆的威廉（William of Occam），1320

4.1 导言

RV32M向RV32I中添加了整数乘法和除法指令。图4.1是RV32M扩展指令集的图形表示，图4.2列出了它们的操作码。

除法是直截了当的。可以回想起如下的式子：

$$商 = (被除数 - 余数) \div 除数$$

或者

$$被除数 = 除数 \times 商 + 余数$$

$$余数 = 被除数 - (商 \times 除数)$$

RV32M具有有符号和无符号整数的除法指令：divide(div)和divide
unsigned(divu)，它们将商放入目标寄存器。在少数情况下，程序员需要余数而不是商，因此RV32M提供remainder(rem)和remainder
unsigned(remu)，它们在目标寄存器写入余数，而不是商。

![](media/image2.PNG){width="4.665445100612423in"
height="1.7575765529308836in"}

**图4.1：RV32M指令的图示**

![](media/image3.PNG){width="5.768055555555556in"
height="1.7243055555555555in"}

**图4.2：RV32M操作码映射包含指令布局，操作码，指令格式类型和它们的名称（\[Waterman
and Asanovic 2017\]的表19.2是此图的基础。）**

乘法的式子很简单：

$$积 = 被乘数 \times 乘数$$

它比除法要更为复杂，是因为积的长度是乘数和被乘数长度的和。将两个32位数相乘得到的是64位的乘积。为了正确地得到一个有符号或无符号的64位积，RISC-V中带有四个乘法指令。要得到整数32位乘积（64位中的低32位）就用mul指令。要得到高32位，如果操作数都是有符号数，就用mulh指令；如果操作数都是无符号数，就用mulhu指令；如果一个有符号一个无符号，可以用mulhsu指令。在一条指令中完成把64位积写入两个32位寄存器的操作会使硬件设计变得复杂，所以RV32M需要两条乘法指令才能得到一个完整的64位积。

![](media/image4.png){width="0.7833333333333333in"
height="0.4388888888888889in"}
对许多微处理器来说，整数除法是相对较慢的操作。如前述，除数为2的幂次的无符号除法可以用右移来代替。事实证明，通过乘以近似倒数再修正积的高32位的方法，可以优化除数为其它数的除法。例如，图4.3显示了3为除数的无符号除法的代码。

![](media/image5.PNG){width="5.768055555555556in"
height="1.1722222222222223in"}

**图4.3：RV32M中用乘法来实现除以常数操作的代码。要证明该算法适用于任何除数需要仔细的数值分析，而对于其它除数，其中的修正步骤更为复杂。算法正确性的证明以及产生倒数和修正步骤的算法在\[Granlund
and Montgomery 1994\]中可以找到。**

**有什么不同之处？**
长期以来，ARM-32只有乘法而无除法指令。直到第一台ARM处理器诞生的大约20年后（2005年），除法指令才成为ARM的必要组成部分。MIPS-32使用特殊寄存器（HI和LO）作为乘法和除法指令的唯一目标寄存器。虽然这种设计降低了早期MIPS处理器实现的复杂性，但它需要额外的移动指令以使用乘法或除法的结果，这可能会降低性能。HI和LO寄存器也会增加架构状态，使得在任务之间切换的速度稍慢。

4.2 结束语

最便宜，最快，并且最可靠的组件是那些没有出现的组件。

------C. Gordan Bell，著名小型计算机的架构师

![](media/image6.png){width="0.6263888888888889in"
height="0.6069444444444444in"}

为了为嵌入式应用提供最小的RISC-V处理器，乘法和除法被归入RISC-V的第一个可选标准扩展的一部分RV32M。许多RISC-V处理器将包括RV32M。

4.3 扩展阅读

T. Granlund and P. L. Montgomery. Division by invariant integers using
multiplication. In *ACM SIGPLAN Notices*, volume 29, pages 61--72. ACM,
1994.

A. Waterman and K. Asanovi´c, editors. *The RISC-V Instruction Set
Manual, Volume I: User-Level ISA, Version 2.2*. May 2017. URL
<https://riscv.org/specifications/>.

注记

http://parlab.eecs.berkeley.edu
