第五章 RV32F和RV32D：单精度和双精度浮点数

![](media/image1.emf){width="1.2497692475940507in"
height="2.183333333333333in"}

只有当没有任何东西可以去除，而不是没有东西可以添加时，我们才最终达到了完美。

------Antoine de Saint Exup'ery, L'Avion, 1940

5.1 导言

尽管RV32F和RV32D是分开的，单独的可选指令集扩展，他们通常是包括在一起的。为简洁起见，我们在一章中介绍了几乎所有的单精度和双精度（32位和64位）浮点指令。图5.1是一个
RV32F和RV32D扩展指令集的图形表示。图5.2列出
RV32F的操作码，图5.3列出了RV32D的操作码。和几乎所有其他现代ISA一样，RISC-V服从IEEE
754-2008浮点标准\[IEEE标准委员会2008\]。

5.2浮点寄存器

![](media/image2.png){width="0.7833333333333333in"
height="0.4388888888888889in"}RV32F和RV32D使用32个独立的f寄存器而不是x寄存器。使用两组寄存器的主要原因是：处理器在不增加RISC-V指令格式中寄存器描述符所占空间的情况下使用两组寄存器来将寄存器容量和带宽是乘2，这可以提高处理器性能。使用两组寄存器对RISC-V指令集的主要影响是，必须要添加新的指令来加载和存储数据f寄存器，还需要添加新指令用于在x和f寄存器之间传递数据。图5.4列出了RV32D和RV32F寄存器及对应的由RISC-V
ABI确定的寄存器名称。

如果处理器同时支持RV32F和RV32D扩展，则单精度数据仅使用f寄存器中的低32位。与RV32I中的x0不同，寄存器f0不是硬连线到常量0，而是和所有其他31个
f寄存器一样，是一个可变寄存器。

IEEE
754-2008标准提供了几种浮点运算舍入的方法，这有助于确定误差范围和编写数值库。最准确且最常见的舍入模式是舍入到最近的偶数（RNE）。舍入模式可以通过浮点控制和状态寄存器fcsr进行设置。图5.5显示了fcsr并列出了舍入选项。它还包含标准所需的累积异常标志。

**有什么不同之处？**
ARM-32和MIPS-32都有32个单精度浮点寄存器但都只有16个双精度寄存器。它们都将两个单精度寄存器映射到双精度寄存器的左右两半。x86-32浮点数算术没有任何寄存器，而是使用堆栈代替。堆栈条目是80位宽度提高精度，因此浮点数负载将32位或64位操作数转换为80位，对于存储指令，反之亦然。x86-32的一个后续版本增加了8个传统的64位浮点寄存器以及相关的操作指令。与RV32FD和MIPS-32不同，ARM-32和x86-32忽视了在浮点和整数寄存器之间直接移动数据的指令。唯一的解决方案是先将浮点寄存器的内容存储在内存中，然后将其从内存加载到整数寄存器，反之亦然。

**图5.1**![](media/image3.PNG){width="5.772222222222222in"
height="3.3125in"}**：RV32F和RV32D的指令图示。**

5.3浮点加载，存储和算术指令

对于RV32F和RV32D，RISC-V有两条加载指令（flw，fld）和两条存储指令（fsw，fsd）。他们和lw和sw拥有相同的寻址模式和指令格式。添加到标准算术运算中的指令有：（fadd.s，fadd.d，fsub.s，fsub.d，fmul.s，fmul.d，fdiv.s，fdiv.d），RV32F和RV32D还包括平方根（fsqrt.s，fsqrt.d）指令。它们也有最小值和最大值指令（fmin.s，fmin.d，fmax.s，fmax.d），这些指令在不使用分支指令进行比较的情况下，将一对源操作数中的较小值或较大值写入目的寄存器。

![](media/image2.png){width="0.7833333333333333in"
height="0.4388888888888889in"}许多浮点算法（例如矩阵乘法）在执行完乘法运算后会立即执行一条加法或减法指令。因此，RISC-V提供了指令用于先将两个操作数相乘然后将乘积加上（fmadd.s，fmadd.d）或减去（fmsub.s，fmsub.d）第三个操作数，最后再将结果写入目的寄存器。它还有在加上或减去第三个操作数之前对乘积取反的版本：fnmadd.s，fnmadd.d，fnmsub.s，fnmsub.d。这些融合的乘法
-
加法指令比单独的使用乘法及加法指令更准确，也更快，因为它们只（在加法之后）舍入过一次，而单独的乘法及加法指令则舍入了两次（先是在乘法之后，然后在加法之后）。这些指令需要一条新指令格式指定第4个寄存器，称为R4。图5.2和5.3显示了R4格式，它是R格式的一个变种。

RV32F和RV32D没有提供浮点分支指令，而是提供了浮点比较指令，这些根据两个浮点的比较结果将一个整数寄存器设置为1或0：feq.s，feq.d，flt.s，flt.d，fle.s，fle.d。这些指令允许整数分支指令根据浮点数比较指令设置的条件进行分支跳转。例如，这段代码在f1
\<f2时，则分支跳转到Exit：

> flt x5，f1，f2 ＃如果f1 \< f2，则x5 = 1;否则x5 = 0
>
> bne x5，x0，Exit ＃如果x5！= 0，则跳转到Exit

  imm\[11:0\]   rs1     010   rd    0000111      I flw                   
  ------------- ------- ----- ----- ------------ --------- ------------- --------------------
  imm\[11:5\]   rs2     rs1   010   imm\[4:0\]   0100111   S fsw         
  rs3           00      rs2   rs1   rm           rd        1000011       R4 fmadd.s fmadd.
  rs3           00      rs2   rs1   rm           rd        1000111       R4 fmsub.s fmadd.
  rs3           00      rs2   rs1   rm           rd        1001011       R4 fnmsub.s fmadd.
  rs3           00      rs2   rs1   rm           rd        1001111       R4 fnmadd.s fmadd.
  0000000       rs2     rs1   rm    rd           1010011   R fadd.s      
  0000100       rs2     rs1   rm    rd           1010011   R fsub.s      
  0001000       rs2     rs1   rm    rd           1010011   R fmul.s      
  0001100       rs2     rs1   rm    rd           1010011   R fdiv.s      
  0001100       00000   rs1   rm    rd           1010011   R fsqrt.s     
  0010000       rs2     rs1   000   rd           1010011   R fsgnj.s     
  0010000       rs2     rs1   001   rd           1010011   R fsgnjn.s    
  0010000       rs2     rs1   010   rd           1010011   R fsgnjx.s    
  0010100       rs2     rs1   000   rd           1010011   R fmin.s      
  0010100       rs2     rs1   001   rd           1010011   R fmax.s      
  1100000       00000   rs1   rm    rd           1010011   R fcvt.w.s    
  1100000       00001   rs1   rm    rd           1010011   R fcvt.wu.s   
  1110000       00000   rs1   000   rd           1010011   R fmv.x.w     
  1010000       rs2     rs1   010   rd           1010011   R feq.s       
  1010000       rs2     rs1   001   rd           1010011   R flt.s       
  1010000       rs2     rs1   000   rd           1010011   R fle.s       
  1110000       00000   rs1   001   rd           1010011   R fclass.s    
  1101000       00000   rs1   rm    rd           1010011   R fcvt.s.w    
  1101000       00001   rs1   rm    rd           1010011   R fcvt.s.wu   
  1111000       00000   rs1   000   rd           1010011   R fmv.w.x     

**图5.2：RV32F操作码表包含了指令布局，操作码，格式类型和名称。这张表与下一张表在编码上的主要区别是：对于这张表，前两个指令第12位是0，并且对于其余指令，第25位为0，而在下一张表中，RV32D中的这两个位均为1（基于\[Waterman
and Asanovic 2017\]的表19.2）。**

  imm\[11:0\]   rs1     011   rd    0000111      I fld                   
  ------------- ------- ----- ----- ------------ --------- ------------- --------------------
  imm\[11:5\]   rs2     rs1   011   imm\[4:0\]   0100111   S fsd         
  rs3           01      rs2   rs1   rm           rd        1000011       R4 fmadd.d fmadd.
  rs3           01      rs2   rs1   rm           rd        1000111       R4 fmsub.d fmadd.
  rs3           01      rs2   rs1   rm           rd        1001011       R4 fnmsub.d fmadd.
  rs3           01      rs2   rs1   rm           rd        1001111       R4 fnmadd.d fmadd.
  0000001       rs2     rs1   rm    rd           1010011   R fadd.d      
  0000101       rs2     rs1   rm    rd           1010011   R fsub.d      
  0001001       rs2     rs1   rm    rd           1010011   R fmul.d      
  0001101       rs2     rs1   rm    rd           1010011   R fdiv.d      
  0001101       00000   rs1   rm    rd           1010011   R fsqrt.d     
  0010001       rs2     rs1   000   rd           1010011   R fsgnj.d     
  0010001       rs2     rs1   001   rd           1010011   R fsgnjn.d    
  0010001       rs2     rs1   010   rd           1010011   R fsgnjx.d    
  0010101       rs2     rs1   000   rd           1010011   R fmin.d      
  0010101       rs2     rs1   001   rd           1010011   R fmax.d      
  0100000       00001   rs1   rm    rd           1010011   R fcvt.s.d    
  0100001       00000   rs1   rm    rd           1010011   R fcvt.d.s    
  1010001       Rs2     rs1   010   rd           1010011   R feq.d       
  1010001       rs2     rs1   001   rd           1010011   R flt.d       
  1010001       rs2     rs1   000   rd           1010011   R fle.d       
  1110001       00000   rs1   001   rd           1010011   R fclass.d    
  1100001       00000   rs1   rm    rd           1010011   R fcvt.w.d    
  1100001       00001   rs1   rm    rd           1010011   R fcvt.wu.d   
  1101001       00000   rs1   rm    rd           1010011   R fmv.d.w     
  1101001       00001   rs1   rm    rd           1010011   R fmv.d.wu    

**图5.3：RV32D操作码表包含了指令布局，操作码，格式类型和名称。这两个图中的一些指令并不仅仅是数据宽度不同。只有这张表有**fcvt.s.d**和**fcvt.d.s**指令，而只有另一张表有**fmv.x.w**和**fmv.w.x**.指令（基于\[Waterman
and Asanovic 2017\]的表19.2）。**

       f0 / ft0     FP Temporary
  ---- ------------ ------------------------------------
       f1 / ft1     FP Temporary
       f2 / ft2     FP Temporary
       f3 / ft3     FP Temporary
       f4 / ft4     FP Temporary
       f5 / ft5     FP Temporary
       f6 / ft6     FP Temporary
       f7 / ft7     FP Temporary
       f8 / fs0     FP Saved register
       f9 / fs1     FP Saved register
       f10 / fa0    FP Function argument, return value
       f11 / fa1    FP Function argument, return value
       f12 / fa2    FP Function argument
       f13 / fa3    FP Function argument
       f14 / fa4    FP Function argument
       f15 / fa5    FP Function argument
       f16 / fa6    FP Function argument
       f17 / fa7    FP Function argument
       f18 / fs2    FP Saved register
       f19 / fs3    FP Saved register
       f20 / fs4    FP Saved register
       f21 / fs5    FP Saved register
       f22 / fs6    FP Saved register
       f23 / fs7    FP Saved register
       f24 / fs8    FP Saved register
       f25 / fs9    FP Saved register
       f26 / fs10   FP Saved register
       f27 / fs11   FP Saved register
       f28 / ft8    FP Temporary
       f29 / ft9    FP Temporary
       f30 / ft10   FP Temporary
       f31 / ft11   FP Temporary
  32   32           

**图5.4：RV32F和RV32D的浮点寄存器。单精度寄存器占用了32个双精度寄存器中最右边的一半。第3章解释了RISC-V对于浮点寄存器的调用约定，阐述了FP参数寄存器（fa0-fa7），FP保存寄存器（fs0-fs11）和FP
临时寄存器（ft0-ft11）背后的基本原理（基于\[Waterman and Asanovic
2017\]的表20.1）。**

![](media/image4.PNG){width="5.772222222222222in"
height="0.8229166666666666in"}**图5.5：浮点控制和状态寄存器。它保存舍入模式和异常标志。舍入模式包括向最近的偶数舍入（**frm**中的rte，000）;**
**向零舍入（rtz，001）;
向下**$\mathbf{(}\mathbf{-}\mathbf{\infty)}$**舍入（rdn，010）;
向上**$\mathbf{(}\mathbf{+}\mathbf{\infty)}$**舍入（rup，011）;
以及向最近的最大值舍入（rmm，100）。
五个累积异常标志表示自上次由软件重置字段以来在任何浮点运算指令上出现的异常条件：NV表示非法操作;
DZ表示除以零; OF表示上溢; UF表示下溢; NX表示不精确（基于\[Waterman
andAsanovic 2017\]的图8.2）。**

![](media/image5.PNG){width="5.772222222222222in"
height="1.3965277777777778in"}

**图5.6：RV32F和RV32D转换指令。在列中列出了源数据类型，在行中列出转换的目标数据类型。**

5.4浮点转换和搬运

RV32F和RV32D支持在在32位有符号整数，32位无符号整数，32位浮点和64位之间浮点进行所有组合的转换（只要这个转换是有用，有意义的）。图5.6按源数据类型以及转换后的目的数据类型，罗列了这10条指令。

RV32F还提供了将数据从f寄存器（fmv.x.w）移动到x寄存器的指令，以及反方向移动数据的指令（fmv.w.x）。

5.5其他浮点指令

RV32F和RV32D提供了不寻常的指令，有助于编写数学库以及提供有用的伪指令。（IEEE
754浮点标准需要一种复制并且操作符号并对浮点数据进行分类的方式，这启发我们添加了这些指令。）

第一个是符号注入指令，它从第一个源操作数复制了除符号位之外的所有内容。符号位的取值取决于具体是什么指令：

1.  浮点符号注入（fsgnj.s，fsgnj.d）：结果的符号位是rs2的符号位。

2.  浮点符号取反注入（fsgnjn.s，fsgnjn.d）：结果的符号位与rs2的符号位相反。

3.  浮点符号异或注入（fsgnjx.s，fsgnjx.d）：结果符号位是rs1和rs2的符号位异或的结果。

![](media/image6.png){width="0.6666666666666666in"
height="0.3770833333333333in"}除了有助于数学库中的符号操作，基于符号注入指令我们还提供了三种流行的浮点伪指令（参见第37页的图3.4）：

1.  复制浮点寄存器：

> fmv.s rd，rs事实上是fsgnj.s rd，rs，rs
>
> fmv.d rd，rs事实上是sgnj.d rd，rs，rs。

![](media/image7.PNG){width="5.0680555555555555in"
height="1.332638888888889in"}**图5.7:用C编写的 浮点运算密集型的DAXPY
程序 **

![](media/image8.PNG){width="6.764583333333333in" height="0.75625in"}

**图5.8：DAXPY在四个ISA上生成的指令数和代码大小。它列出了每个循环的指令数量以及指令总数。第7章介绍ARM
Thumb-2，microMIPS和RV32C指令集。**

2.  否定：

> fneg.s rd，rs映射到fsgnjn.s rd，rs，rs
>
> fneg.d rd，rs映射到fsgnjn.d rd，rs，rs。

3.  绝对值（因为0⊕0= 0且1⊕1= 0）：

> fabs.s rd，rs变成了fsgnjx.s rd，rs，rs
>
> fabs.d rd，rs变成了sgnjx.d rd，rs，rs。

第二个不常见的浮点指令是classify分类指令（fclass.s，fclass.d）。分类指令对数学库也很有帮助。他们测试一个源操作数来看源操作数满足下列10个浮点数属性中的哪些属性（参见下表），然后将测试结果的掩码写入目的整数寄存器的低10位。十位中仅有一位被设置为1，其余为都设置为0。

  *x\[rd\]*位   含义
  ------------- -----------------------------------
  0             f \[*rs1*\]为$- \infty$。
  1             f \[*rs1*\]是负规格化数。
  2             f \[*rs1*\]是负的非规格化数。
  3             f \[*rs1*\]是-0。
  4             f \[*rs1*\]是+0。
  5             f \[*rs1*\]是正的非规格化数。
  6             f \[*rs1*\]是正的规格化数。
  7             f \[*rs1*\]为+$\infty$。
  8             f \[*rs1*\]是信号(signaling)NaN。
  9             f \[*rs1*\]是一个安静(quiet)NaN。

5.6使用DAXPY程序比较RV32FD，ARM-32，MIPS-32和x86-32

我们现在将使用DAXPY作为我们的浮点基准对不同ISA进行比较（图5.7）。它以双精度计算$Y\  = \ a \times X + \ Y$，其中X和Y是矢量，a是标量。图5.8总结了DAXPY在四个不同![](media/image9.png){width="0.7in"
height="0.6034722222222222in"}的ISA下对应的指令数和字节数。他们的代码如图5.9至5.12所示。

![](media/image2.png){width="0.7833333333333333in"
height="0.4388888888888889in"}与第2章中的插入排序一样，尽管RISC-V指令集强调本身的简单性，RISC-V版本的不管是指令数量还是代码大小，都接近或者优于其他ISA。在此示例中，RISC-V的比较和执行分支指令和ARM-32和x86-32中更复杂的寻址模式，以及入栈、退栈指令节省了差不多数量的指令。

5.7结束语

少即是多。

------Robert Browning,
1855，极简主义（建筑）建筑学派在20世纪80年代采用这首诗作为公理。

IEEE 754-2008浮点标准\[IEEE Standards Committee
2008\]定义了浮点数据类型，计算精度和所需操作。它的广泛流行大大降低了移植浮点程序的难度，这也意味着不同ISA中的浮点数部分可能比其他章节中描述的其他部分的指令更一致。

5.8 扩展阅读

IEEE Standards Committee. 754-2008 IEEE standard for floating-point
arithmetic. *IEEE*

*Computer Society Std*, 2008.

A. Waterman and K. Asanovi´c, editors. *The RISC-V Instruction Set
Manual, Volume I:*

*User-Level ISA, Version 2.2*. May 2017. URL
[[https://riscv.org/specifications/]{.underline}](https://riscv.org/specifications/).

注记

http://parlab.eecs.berkeley.edu

![](media/image10.PNG){width="5.772222222222222in"
height="2.8722222222222222in"}

**图5.9：图5.7中DAXPY的RV32D代码。十六进制的地址位于机器的左侧，接下来是十六进制的语言代码，然后是汇编语言指令，最后是注释。比较和分支指令避免了ARM-32和X86-32代码中的两条比较指令。**

![](media/image11.PNG){width="5.772222222222222in" height="2.45625in"}

**图5.10：图5.7中DAXPY的ARM-32代码。
与RISC-V相比，ARM-32的自动增量寻址模式可以节省两条指令。与插入排序不同，DAXPY在ARM-32上不需要压栈和出栈寄存器。**

**\
**

![](media/image12.PNG){width="5.772222222222222in"
height="2.7534722222222223in"}

**图5.11：图5.7中DAXPY的MIPS-32代码。三个分支延迟槽中的两个填充了有用的指令。检查两个寄存器之间是否相等的指令避免了ARM-32和x86-32中的两条比较指令。与整数加载不同，浮点加载没有延迟槽。**

![](media/image13.PNG){width="5.772222222222222in"
height="3.6944444444444446in"}

**图5.12：图5.7中DAXPY的x86-32代码。在这个例子中，x86-32缺少寄存器的劣势在这里表现得很明显------有四个变量被分配到了内存，而在其他ISA中，这些变量是被存放在寄存器中的。它展示了x86-32中，如何将寄存器与零比较（**test
ecx，ecx**）以及如何将一个寄存器清零（**xor eax,eax**）。**
