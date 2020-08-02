## 存储器、CPU

> From 看雪学院

|            | 速度           | 成本   |
| ---------- | -------------- | ------ |
| 外部存储器 | 读写速度比较慢 | 300/T  |
| 内部存储器 | 比较快         | 300/4G |

内部存储器分为RAM（随机存储器），ROM（只读存储器），区别在于断电数据是否消失，ROM不会消失，而RAM会消失。

存储单元，每个单元由存储单元的地以及存储单元的内容构成，均为16进制。

CPU对存储器的读写是通过三种总线完成。

![image.png](https://i.loli.net/2020/07/18/EyLhi5bVZmsAY8t.png)

* 地址总线的宽度决定了CPU的寻址能力；
* 数据总线的宽度决定了CPU与其他器件进行数据传送的一次数据传送量；
* 控制总线宽度决定了CPU对系统中其他器件的控制能力。

一个CPU有N根地址总线，则可以说这个CPU的地址总线的宽度为N，最多可以寻找$2^N$个内存单元。8086有20根地址线。

一根数据总线可以传输一个0或1,8086的数据总线宽度分别为16根。一次可以传送的数据为2B

## 寄存器

> 程序需要加载到内存中，CPU通过寄存器(CS: IP)寻址来确定即将执行指令位置

### 介绍

并列于运算器和控制器，容量有限速度快，可以用来暂时存储指令、数据和地址，由于存在于CPU中，所以数量有限。

8086有14个寄存器，所有寄存器都是16位的，可以存放两个字节，即存放一个字。

### 组成

16位CPU所含有的寄存器有14个：

* **4个数据寄存器 AX、BX、CX、DX**
* **2个变址寄存器SI、DI和两个指针寄存器SP、BP**
* **4个段寄存器ES、CS、SS、DS**
* **一个指令指针寄存器IP和1个标志寄存器Flags**

32位的CPU比14位的多了两个寄存器，是段寄存器FS、GS，另外对于两种CPU中寄存器的区别就是32位的开头加一个E，除了段寄存器。

![image.png](https://i.loli.net/2020/07/18/EfTI7HLMOU4PGmV.png)

64位的前面加的是R

![image.png](https://i.loli.net/2020/07/18/j6ZEe4iLuzG78tl.png)

* 通用寄存器包括8个寄存器，数据寄存器的全部，变址寄存器和指针寄存器的全部

* 只有数据寄存器可以向下兼容，32位的分出来16位的，16位的分出来8位的

### 功能（分开介绍）

#### 通用寄存器

可以用来传送和暂存数据，可参与算术逻辑运算并保存运算结果

#### 数据寄存器

* 保存操作数和运算结果
* 每一个数据寄存器都可以当做两个单独的寄存器使用

|    寄存器    |                             说明                             |
| :----------: | :----------------------------------------------------------: |
| AX（AL、AH） |               累加寄存器，用于乘除输入输出操作               |
| BX（BL、BH） |               基地址寄存器，作为存储器指针使用               |
| CX（CL、CH） | 计数寄存器，在<font color="red">循环</font>和字符串操作中，要用它来控制循环次数，在位操作中，当移多位时，要用CL指明移位的位数 |
| DX（DL、DH） | 数据寄存器，进行乘除运算时，可作为默认的操作数参与运算，也可用于I/O的端口地址 |

![image.png](https://i.loli.net/2020/07/18/dUD4T1m3AeYhCSr.png)

<font color="red">在16位CPU中，AX、BX、CX、DX不能存放存储单元的地址，32位寄存器中不仅可以传送数据、暂存数据保存算术逻辑运算结果，而且可以作为指针寄存器</font>

**For Example：**

MOV AX,11F6H  ->  11存入AH、F6存入AL

MOV  AL,0AH    ->   F6 + 0A = 100H, 9位，超过AL，所以去掉最高位，变成00H

### CS: IP

任意时刻，CPU将CS: IP指向的内容作为即将执行的指令

#### 物理地址

通常文件包含两个段，代码段和地址段

* 代码段可读、不可写、可执行，存储程序的指令
* 数据段可读、可写、可执行，存储程序中要用到的数据

运行一段代码时，首先将代码段加入到内存中，CPU找到代码段需要物理地址。

> 物理地址 = 基础地址 + 偏移地址
>
> 基础地址 = 段地址 x 10H（$16_{10}$）

![image.png](https://i.loli.net/2020/07/18/fvZW6ozJOSnhYa5.png)

<font color="red">物理地址可以由不同的段地址和偏移地址形成</font>

**8086中有4个段寄存器：**

* CS代码段 【主要关注】
* DS数据段：存放操作的数据
* SS堆栈段：存放暂时不用但需要保存的数据
  * 举个例子，写代码时的return，return回来的位置就是记录在堆栈段的
  * 常用于响应中断或子程序调用
* ES附加段：存放操作的数据【一部分指令需要】

无论段的个数有多少个，类型只有四种，四个逻辑段，8086每类逻辑段的数量最多为64k个

在任何一个程序模块中，每一种类型的逻辑段最多只有1个，如果说程序太大，一个段里放不下，那就只能再编写一个模块。要分时运行，不能同时做，

#### 过程

1. CS、IP加入地址加法器，进行相加转换为20位的地址
2. 通过地址总线输入到内存当中，找到该地址的数据，并加载到数据总线当中，传入CPU
3. 到了 指令缓存器中
4. 由于执行了内存中的三条指令，所以IP + 3
5. 将结果存放在AX寄存器中

#### 修改CS：IP

CPU是由CS：IP中的内容决定执行命令

修改：

* 同时修改CS：IP内容
  * jmp 段地址：偏移地址
  * For Example：jmp 2AE3:3 -> 从2AE33H处读取指令
  * jmp 3:0B16  - > 从00B46H处读取指令
* 修改IP的内容
  * jmp 某一合法寄存器
  * For Example：jmp bx;
    * 执行前：bx = 0B16H, CS = 2000H, IP = 0003H
    * 执行后：bx = 0B16H, CS = 2000H, IP = 0B16H

## 分段管理及标志寄存器

![image.png](https://i.loli.net/2020/07/18/blmH4hNZ1LV6Yie.png)

### 分段管理

在8086中，一个存储单元对应一个物理地址，还有多个逻辑地址。

* 物理地址

  * 一个存储单元的编号，

  * 每个物理存储单元都有一个20位的编号（地址总线20条），
  * 物理地址范围00000H~FFFFFH（$2^{20}$ = 1M）<font color="red">4个二进制位可以构成16进制位</font>

* 逻辑地址

  * 编程时采用逻辑地址，段基地址：段内偏移地址

* 物理地址就是将逻辑地址左移4位，加上偏移地址就得到20位物理地址（物理 = 段基 + 偏移）【移4位，十进制数16的16进制为10H】

![image.png](https://i.loli.net/2020/07/18/xYpcNAq8tr3Qj14.png)

#### 段寄存器与逻辑段

8086CPU有4个段寄存器，**每个段寄存器用来确定一个逻辑段的起始位置**：

> 段寄存器中的值表明相应逻辑段在内存中的位置

* CS 指明代码的起始地址
  * 利用CS：IP取得下一条要执行的指令
* SS 指明堆栈段的起始地址
  * 利用SS：SP操作堆栈顶的数据
* DS 指明数据的起始地址
  * 利用DS：EA存取数据段中的数据
* ES 指明附加段的起始地址
  * 利用ES：EA存取附加段中的数据

EA是偏移地址，称为有效地址EA

**若操作数在主存中，存取方法有直接寻址、寄存器间接寻址、寄存器相对寻址、基址变址寻址方式、相对基址变址寻址方式**

没有指明段前缀时，一般的数据访问在DS（数据段）

ForExample：

```汇编
	MOV AX,[1000H]; = MOV AX,DS:[1000H];

​	从默认的DS段中取出数据DS代表段基地址；[ ？] 代表偏移地址

​	MOV AX,CS:[1000H]

​	从指定CS段取出数据
```

如果没有指明段基地址，就要使用 DS 默认段基地址

### 标志寄存器

![image.png](https://i.loli.net/2020/07/19/lD4YFLRyo9dSaPi.png)

* 8086CPU有9个有效标志

| 标志位 | 标志位名称 |   =1   |    =0    |
| :----: | :--------: | :----: | :------: |
|   CF   |  进位标志  |  进位  |  无进位  |
|   PF   |  奇偶标志  |   偶   |    奇    |
|   AF   |  辅助进位  |  进位  |  无进位  |
|   ZF   |   零标志   | 等于零 | 不等于零 |
|   SF   |  符号标志  |   负   |   非负   |
|   TF   |  跟踪标志  |        |          |
|   IF   |  中断标志  |  允许  |   禁止   |
|   DF   |  方向标志  |  减少  |   增加   |
|   OF   |  溢出标志  |  溢出  |  未溢出  |

标志位可以分为

* 状态标志**【用于记录程序运行结果的状态信息】**
  * CF ZF SF PF OF AF
* 控制标志【**用于控制处理器执行指令**】
  * DF  IF TF

#### 进位标志CF

> 当运算结果的最高有效位有进位（加法）或借位（减法）时，进位标志置1，CF= 1；否则 =0

如：2C +7C = A8, 没有进位，CF= 0；没有超出8个字节，还在一个字当中

C2 + C7 = (1)89 有进位，CF = 1

#### 零标志ZF

> 若运算结果为 0 ，则ZF= 1

与运算结果相反，

**注意：**结果看的是单个字节当中，所以像之前的计算，有进位变成100，但是在8位中都为 0 ，所以按照结果为 0

#### 符号标志SF

> 若运算结果最高位为1，则SF=1

有符号运算，最高位代表符号

注：有符号范围：-128~127

---

计算机在存储数据时，是以补码形式存在的

---

同样还是在一个字节中看结果，即使进位了，也只看八位中的最高位，因为超出了有符号数的范围，由正值变成负值

#### 奇偶标志

> 若运算结果最低字节中“1”的个数为零或偶数时，PF= 1

#### 溢出标志

> 若运算结果有溢出，则OF = 1

如果运算结果超出了范围，就产生了溢出，有溢出，说明有符号数的运算结果不正确

**说明：**我们通常认为溢出就是因为进位时当前存储格式的位数不够引起，比如8位寄存器：11111111B + 1B = 100000000B。超过了8位的1被认为是溢出寄存器（放不下），当然也是进位的那个1

#### 溢出与进位

* 溢出表示超出了范围，已经不正确
* 进位超出范围，仍然正确

---

有符号无符号指的是最高位是否是符号位，即是以补码的形式看待还是以源码的形式看待

* CF：0~255 / 0X00~0XFF（8位）
  * 0~65536 / 0X0000~0XFFFF（16位）
* OF：-128~127 / 0X80~0XEF（8位）
  * -32768~32767 / 0X8000~0XEFFF（16位）

---

![image.png](https://i.loli.net/2020/07/19/U6dk17qWcSZnzPI.png)

#### 辅助进位标志

> 若运算时$D_{3}$（低半字节）有进位或借位时，AF = 1

#### 方向标志

> 用于串操作指令中，控制地址的变化方向
>
> * 设置DF为0，存储器地址自动增加
> * 设置DF为1，存储器地址自动减少
>
> For Example：
>
> * CLD 用于复位方向标志，执行后DF = 0
> * STD 用于置位方向标志，执行后DF = 1

#### 中断允许标志IF

> 用于控制外部可屏蔽中断是否可以被处理器响应
>
> * 设置IF 为0，禁止中断
> * 设置为1，允许中断
>
> For Example：
>
> * CLI 用于复位中断标志
> * STI 用于置位

#### 陷阱标志TF

> 用于控制处理器进入单步操作方式
>
> * 设置TF 为0，处理器正常工作
> * 设置TF为 1，处理器单步执行指令
>
> 单步执行指令：处理器在每条指令执行结束时，便产生一个编号为1的内部中断

## 指令简介

> 指令由操作码和操作数构成
>
> * 操作码：给出该指令应完成何种操作
> * 操作数：描述该指令的操作对象
>
> 操作码不可缺少，但是操作数可以没有，也可以有一个操作数或两个操作数

* 零操作数指令
  * 指令格式中没有操作数或操作数是隐含约定的
* 一操作数
  * 有一个或者还有一个隐含的双操作数
* 二操作数
  * 两个操作数【常见】
    * **目的操作数**
    * **源操作数**

由此，操作数分为目的操作数和源操作数

* 源操作数智能读取的操作数
* 目的操作数：可读取可写入（存放操作结果）的操作数

|  MOV   |     AX     |    10    |
| :----: | :--------: | :------: |
| 操作码 | 目的操作数 | 源操作数 |

## 寻址方式

> 指令中指明操作数存放位置的表达式

操作数的数据存放位置有三种情况：

1. 存放指令中（立即数）
   * 操作数包含在指令中，即被操作数据直接表示在指令的操作数字段中，也就是紧跟在操作码之后
     * MOV AL，10H
2. 存于寄存器中（寄存器操作数）
   * 数据存放在CPU的一个寄存器中
     * INC  CX
3. 存放于存储器中（存储器操作数）
   * 数据在内存中或在I/O端口中，存放数据的偏移地址以某种方式表示在指令中
     * MOV AX，[2500H] ，其中[2500] 为存储器操作数
     * 存储器操作数中操作数字段指示此操作数的偏移地址，而段地址由某个段寄存器来提供，此例中默认为数据段DS

### 立即数寻址方式

> 只允许源操作数为立即数，目标操作数必须是寄存器或存储器，其作用是给寄存器或存储单元赋值
>
> 汇编中，立即数不能作为指令中的第一操作数，也就是说不能MOV 80H，AL
>
> * MOV AL，80H
> * MOV AX，1234H

![image.png](https://i.loli.net/2020/07/19/JnNB8EQzRjxgG7Z.png)

### 寄存器寻址

> 表示格式：直接在指令中写出寄存器的名称
>
> * INC BX  BX中的数据自增1
> * MOV AX，CX   CX中的数据赋给AX

### 存储器寻址

> 直接在指令中写出寄存器名称
>
> * 直接寻址方式
> * 寄存器间接寻址方式
> * 寄存器相对寻址方式
> * 基址加变址寻址方式
> * 相对基址加变址寻址方式

**存储器有两种：内存和I/O端口**

#### 直接寻址

> 操作数存在内存中，操作数的偏移地址直接表示在指令中
>
> 表示格式：[偏移地址]
>
> 默认操作数存放在内存的数据段中
>
> * MOV AL，[1064H]

需要获取实际内存的数据，通过偏移地址获取内存的实际地址，借助段地址，默认存放在数据段中，所以数据段 x 10H + 偏移地址得到实际地址。

<img src="https://i.loli.net/2020/07/19/H1dNFypzXU43MQL.png" alt="image.png" style="zoom:50%;" />

#### 段超越

* 操作数也允许存放在其他段中（ES，SS），此时应该在指令中指明段超越
* 若操作数不在指令默认的DS段中，而在其它某个段中，则需要在指令中加以表示，称为段超越
* 直接寻址方式中操作数在附加段中，应该表示为 MOV AL，ES：[1064H]

#### 寄存器间接寻址方式

> 操作数存在存储器中，操作数的偏移地址在BX、SI、DI、BP的某个寄存器中
>
> 若以BX、SI、DI作为间接寻址寄存器，则默认操作数存放在数据段中，用DS寄存器中的内容作为地址
>
> 若以BP作为间接寻址寄存器，则默认操作数存放在堆栈段中，用SS寄存器中的内容作为地址

<img src="https://i.loli.net/2020/07/19/k1ED79RqawQv6Vx.png" alt="image.png" style="zoom:50%;" />

也存在段超越，若以BP作为间接寻址寄存器，则默认操作数存放在堆栈段中，用SS寄存器中的内容作为地址。

* MOV  AX，DS：[BP]
* MOV CH, SS: [SI]

#### 寄存器相对寻址方式

> 操作数在存储器中，操作数的有效地址是一个基址寄存器（BX、BP）或变址寄存器（SI、DI）的内容加上指令中给定的8位或16位位移量之和。
>
> 如果SI、DI、BX中的内容作为有效地址的一部分，那么引用的段寄存器是DS，如果BP中的内容作为有效地址的一部分，那么引用的段寄存器是 SS
>
> * 下面指令中，源操作数采用寄存器相对寻址，引用的段寄存器是SS：MOV BX，[BP - 4]
> * 下面的指令中，目的操作数采用寄存器相对寻址，引用段寄存器是ES：MOV ES：[BX+5]，AL
>
> 如： MOV CL，[BX+1064H]

<img src="https://i.loli.net/2020/07/19/lV4aCcT8A3zeIpg.png" alt="image.png" style="zoom:50%;" />

#### 基址加变址寻址方式

通常把BX、BP看作是基址寄存器，把SI、DI看作变址寄存器，可把两种方式结合起来形成一种新的寻址方式

基址加变址的寻址方式是把一个基址寄存器BX或BP的内容，加上变址寄存器SI或DI的内容，并以一个段寄存器作为地址基准，作为操作数的地址。

如：MOV AH，[BP] [SI]

<img src="https://i.loli.net/2020/07/19/bC9JaODEMiRq12m.png" alt="image.png" style="zoom:50%;" />

#### 相对基址加变址寻址方式

> 通常把BX、BP看作基址寄存器，SI、DI看作变址寄存器，把一个基址寄存器BX、BP的内容，加上变址寄存器SI、DI的内容，再加上指令中给定的8位或16位位移量，并以一个段寄存器作为地址基准，作为操作数的地址
>
> * 当基址寄存器为BX时，段寄存器使用DS
> * 当基址寄存器为BP时，段寄存器则用SS
>
> 如：MOV [BX+DI+1234H], [AH]

<img src="https://i.loli.net/2020/07/19/tcFM38ls2zwnTrQ.png" alt="image.png" style="zoom:50%;" />

#### 总结

指令要有操作码，目的操作数，源操作数

## 数据传送指令

包括

* 通用传送指令
* 累加器专用传送指令
* 地址传送指令
* 标志传送指令

### 通用传送指令

#### 基本传送指令（MOV）

* 指令格式：MOV、DST、SRC
* 源操作数和目的操作数可用**前6种寻址方式的任何一种**
* 操作：将SRC内容赋给DST
* <font color="red">所有通用传送指令都不影响标志位</font>(前面提到的那9个标志位)

<font color="gree">注意：不能用MOV指令实现以下：</font>

* 存储器操作数之间不能直接传送

  * MOV [1000H], [DI]   (X)

  改为：

  * MOV  AX，[DI]      MOV  [1000H],  AX

* 立即数不能直接送段寄存器

  * MOV  DS,  2000H   (X)

  改为：

  * MOV  AX，2000H       MOV  DS，AX

* 段寄存器之间不能直接传送

  * MOV ES，DS  （X）

  改为：

  * MOV  AX，DS        MOV  ES，AX

* CS 只可以作为源操作数【<font color="red">CS：IP指向当前我们要执行的指令，如果更改了CS就会导致相关指令无法被执行</font>】

  * MOV  CS，AX   （X）

  改为：

  * MOV  AX，CS

* 源操作数和目的操作数的宽度必须相同

![image.png](https://i.loli.net/2020/07/19/2IEsgacjmBJOnA5.png)

#### 堆栈指令（PUSH、POP）

> 与SS、SP有关，SP永远指向栈顶

##### 入栈指令：PUSH  src

* (SP) <- (SP) - 2
* ((sp)+1,(sp))  <-  (src)

把一个字压入由SP指向的堆栈区

如：PUSH  AX

> 若（AX）=50A0H； SP = 2002H；SS = 6000H
>
> 执行PUSH  AX
>
> 再执行 PUSH  BX
>
> 设（BX） =  ABCDH
>
> ![image.png](https://i.loli.net/2020/07/19/YW19MKxvomjpOA2.png)

##### 出栈指令：POP  dst

* (dst)  <-  ((sp)+1, (sp))
* (SP)  <-  (SP) + 2

把 SP 所指向的堆栈顶部的一个字送入目的地址，同时进行修改堆栈指针

如：POP   BX    POP AX

堆栈的用途是存放中断

![](https://img-blog.csdnimg.cn/20200725090410541.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

* 存放寄存器或存储器中暂时不使用的数据，在使用这些数据时可以方便地将其弹出
* 调用子程序或发生中断时要保护断点信息（入栈），子程序或中断返回时恢复断点信息（出栈）

**注意的问题：**

* 堆栈操作都按字操作
* PUSH、POP指令的操作数可以是CPU内部寄存器或存储单元
* PUSH CS 合法，，POP CS 非法【CS不能随便修改，不能做操作数】
* 执行PUSH指令， (SP) -2  -> (SP)，低字节放在低地址，高字节放在高地址
* SP总指向栈顶；堆栈的最大容量即为SP初值与SS之差

##### 交换指令

XCHG  dst，src； （dst） < - >  (src)

可以实现寄存器之间、寄存器和存储器之间

**注意：**

* **存储器之间不能直接交换**
* 段寄存器不能作为操作数
* 允许字或字节操作

### 累加器专用传送指令

##### 输入指令（IN）

> 用于CPU从外设端口接收数据

* IN  AL, DATA8   从8位端口地址输入一个字节
* IN  AX, DATA8   从8位端口地址输入一个字
* IN  AL, DX         从16位端口地址输入一个字节
* IN  AX, DX         从16位端口地址输入一个字

##### 输出指令

> 用于CPU向外设端口发送数据

![image.png](https://i.loli.net/2020/07/19/vs1plDXPLT26rV5.png)

与输入指令一致

### 目的地址传送指令

> 8086提供三条
>
> * LEA
> * LDS
> * LES

<img src="https://i.loli.net/2020/07/19/bTnWrzR6jQUSNCP.png" alt="image.png" style="zoom:50%;" />

<img src="https://i.loli.net/2020/07/19/i3CPsvcV6EWIMYU.png" alt="image.png" style="zoom: 50%;" />

<img src="https://i.loli.net/2020/07/19/rjy2cetNdYKZTUi.png" alt="image.png" style="zoom:50%;" />

<img src="https://i.loli.net/2020/07/19/sgjNH69yl71Phrp.png" alt="image.png" style="zoom:50%;" />

LES 与 LDS 工作原理相同

### 标志传送指令

> 8086 有四种标志传送操作指令

<img src="https://i.loli.net/2020/07/19/FTkB3DJdI1wiZWs.png" alt="image.png" style="zoom:50%;" />

<img src="https://i.loli.net/2020/07/19/bzgtorvNEneyHp9.png" alt="image.png" style="zoom:50%;" />

<img src="https://i.loli.net/2020/07/19/U4eyJ8fl35HskhX.png" alt="image.png" style="zoom:50%;" />

## 算术运算指令（+ - x /）

### 加法指令

8086有5条加法指令：

* ADD 加法指令
* ADC 带进位加法指令
* INC 加 1 指令，自增 1 
* AAA 加法ASCII调整指令
* DAA 加法十进制调整指令

#### ADD 无进位加法指令

> ADD dest，src
>
> dest  <-  dest + src
>
> * src：立即数，通用寄存器，存储器
> * dest：通用寄存器，存储器
>
> <img src="https://i.loli.net/2020/07/19/VgP6rLhjfiXGBRo.png" alt="image.png" style="zoom:50%;" />

<img src="https://i.loli.net/2020/07/19/fCxnYvZAMDa3JUR.png" alt="image.png" style="zoom: 50%;" />

#### 带进位的ADC

> ADC dest，src
>
> dest <- dest + src + c
>
> C：进位标志C的现行值（上条指令C值）
>
> 特点与ADD相同，主要用于多字节运算中

![image.png](https://i.loli.net/2020/07/19/2JsPWolMTr1xL86.png)

#### 加1指令INC

> INC dest
>
> dest：通用寄存器、存储器
>
> 用于在循环程序中修改地址指针和循环次数
>
> 标志位影响情况，不影响进位 C

![image.png](https://i.loli.net/2020/07/19/27Nsj5ZyCmUEabB.png)

### 减法指令

* SUB 减法指令
* SBB 进位减法指令
* DEC 减 1 指令
* NEG 求补指令
* CMP 比较指令
* AAS 减法ASCII调整指令
* DAS 减法十进制调整指令

#### SUB无进位减法

> SUB dest，src
>
> src：立即数，通用寄存器、存储器
>
> dest：通用寄存器，存储器
>
> 指令影响标志位：A、C、O、P、S、Z

#### SBB 带进位减法

> SBB dest，src
>
> dest <- dest - src - C
>
> src：立即数，通用寄存器、存储器
>
> dest：通用寄存器、存储器

#### DEC

> DEC dest
>
> dest：通用寄存器、存储器，不能是段寄存器
>
> 用于循环修改地址指针和循环次数
>
> 不影响C

#### 求补指令 NEG

> NEG dest
>
> dest  <- 0-dest
>
> dest：通用寄存器、存储器
>
> 把操作数按位求反后末位 +1
>
> <img src="https://i.loli.net/2020/07/19/Wl4jsN632YbSxF1.png" alt="image.png" style="zoom:50%;" />

#### 比较指令CMP

> CMP dest，src
>
> dest - src   结果不保留，只是用来影响标志位A、C、O、P、S、Z
>
> src：立即数，通用寄存器，存储器
>
> dest：通用寄存器，存储器
>
> <img src="https://i.loli.net/2020/07/19/rjDuv26FoYTeMsd.png" alt="image.png" style="zoom:50%;" />

<img src="https://i.loli.net/2020/07/19/5kNgP3Z7LYfS4Jd.png" alt="image.png" style="zoom:50%;" />

通过标志位来判断。

### 乘法指令

#### 无符号乘法MUL

> NUL src
>
> 字节操作数：AX  <- （AL）*（SRC）
>
> 字操作数：DX：AX  <-  (AX)*(SRC)

#### 带符号乘法IMUL

> IMUL  SRC
>
> 同 MUL 的操作，但是操作数和乘积均带符号
>
> 按有符号数的规则相乘

### 除法指令

#### 无符号除法DIV

> DIV  SRC
>
> 字节除数：AL <- (AX)/(SRC)  之商
>
> ​						AH <- (AX)/(SRC) 之余数
>
> 字除数：AL  <- (DX:AX)/(SRC)  之商
>
> ​				  DX  <-  (DX:AX)/(SRC)  之余数

#### 带符号除法

> IDIV  SRC

商和余数都是带符号的，商的符号符合一般代数符号规则，余数的符号与被除数相同

![image.png](https://i.loli.net/2020/07/19/j4mNIEKLBq9CtAM.png)

![image.png](https://i.loli.net/2020/07/19/6FrHEsL2ukTaeZy.png)

## 逻辑运算和移位指令

### 逻辑运算

#### 测试指令

> TEST  dest，src
>
> SRC：立即数、通用寄存器、存储器
>
> dest：通用寄存器，存储器
>
> 规则同AND，操作数想与，结果不保存，用来改变标志位

#### 或指令OR

> dest、src不能同时为存储器操作数

#### 异或指令XOR

> 不能同时为存储器操作数
>
> dest：寄存器、存储器
>
> src：立即数、寄存器、存储器

#### 非NOT

> NOT  dest
>
> 操作数：寄存器、存储器，不能是立即数

### 移位指令

<img src="https://i.loli.net/2020/07/19/eh6v5O9P7qIdCTk.png" alt="image.png" style="zoom:50%;" />

![image.png](https://i.loli.net/2020/07/19/4irguqRUB9Xh7Wz.png)

> 移位等于 1 或者 CL，如果不是 1 ，尽量赋给 CL

#### 逻辑左移/算术左移SHL/SAL

> 实现相同的操作：
>
> > 相当于无符号数 * 2
>
> SHL 是逻辑左移，右边的位补零

#### 逻辑右移指令SHR

> 相当于无符号数 / 2
>
> > SHR 右移时，它的最高位用 0 填补，最低位移入 CF
> >
> > * SHR  BYTE  PTR[DI + BP], 1
> > * SHR  BL, 1
> > * SHR  AX, CL 

#### 算术右移指令SAR

> 和刚才的类似，**但是<font color="red">最高位不补零</font>**，最高位和移位前的最高位一致。
>
> SHR 右移时，最高位不变，最低位移入CF
>
> > MOV  AL, 88H
> >
> > MOV  AL, 2
> >
> > SAR  AL, CL
> >
> > RESULT :  AL = E2H

#### 不含进位标志循环左移指令ROL

> ROL 是循环左移，左边移出的位补到右边，并把这个值传给 CF

#### 不含C的循环右移指令ROR

> ROR类似于ROL，是循环右移，移出的位补到最左边，并且传给CF

#### 含C的循环左移指令RCL

> 循环左移，进位值（原CF）到低位，高位进CF
>
> > MOV  AL, 11110000B
> >
> > RCL  AL, 1
> >
> > 执行后AL = 11100000B  CF = 1

#### 含C的循环右移指令RCR

> 同RCL

![ULvr3d.png](https://s1.ax1x.com/2020/07/23/ULvr3d.png)

## 串操作指令

> 串：内存中一段地址相连的字节或字；串操作也叫数据块操作；可实现存储器数据间的直接传送
>
> > 8086 有5种基本串操作：
> >
> > MOVS  串传送指令
> >
> > CMPS  串比较指令
> >
> > SCAS  串扫描指令
> >
> > LODS  取串指令
> >
> > STOS  存串指令

讲 MOV 的时候说过存储器数据间不能直接传送，如果想要传递需要借助通用寄存器。

### MOVS B/W

> 串传送有2中格式：MOVSB / MOVSW

![UOCyvQ.png](https://s1.ax1x.com/2020/07/23/UOCyvQ.png)

![UOCHKJ.png](https://s1.ax1x.com/2020/07/23/UOCHKJ.png)

![UOPeG8.png](https://s1.ax1x.com/2020/07/23/UOPeG8.png)

 ![UOiFlF.png](https://s1.ax1x.com/2020/07/23/UOiFlF.png)

![UOiVm9.png](https://s1.ax1x.com/2020/07/23/UOiVm9.png)

**简化代码：**

![UOFl40.png](https://s1.ax1x.com/2020/07/23/UOFl40.png)

![UOFsgO.png](https://s1.ax1x.com/2020/07/23/UOFsgO.png)

### LODS

> 有两种格式：LODSB / LODSW

[![UOFo28.png](https://s1.ax1x.com/2020/07/23/UOFo28.png)](https://imgchr.com/i/UOFo28)

### STOS

> 两种格式：STOSB / STOSW

![UOZGnS.png](https://s1.ax1x.com/2020/07/23/UOZGnS.png)

### CMPS

> 结果不保存，只更改标志

![UOZahn.png](https://s1.ax1x.com/2020/07/23/UOZahn.png)

### SCAS

[![UOZh1x.png](https://s1.ax1x.com/2020/07/23/UOZh1x.png)](https://imgchr.com/i/UOZh1x)

### CMPS 和 SCAS 可与前缀 REPE / REPZ 和 REPNE / REPNZ 联合工作

> ①REPE / REPZ
>
> * 当相等/为零时重复串操作
>
> ②REPNE / REPNZ
>
> * 当不相等/不为零时重复串操作

## 程序控制指令

> 转移指令、循环控制指令、过程调用指令、中断指令

### 转移指令

> 控制程序（记得CS：IP吧）从一处转换到另一处执行，在 CPU 内部，转移是通过将**目标地址**传送给 **IP**实现
>
> 转移指令包括：
>
> * 无条件转移指令
> * 条件转移指令

#### 无条件转移指令 JMP

> JMP 语句标号
>
> JMP LP；

#### 条件转移指令

##### 根据单个条件标志转移

① Z标志

* JZ / JNZ
* z  --  zero

② C标志

* JC / JNC
* C -- 进位

③ P标志

* JP（JPE）/ JNP（JPO）
* 奇偶验证

④ S标志

* JS / JNS
* 负号跳转

⑤ O标志

* JO / JNO
* 溢出跳转

##### 根据两个无符号数大小关系转移

> JB （低于跳转）、JNAE；JNB、JAE
>
> JBE（不高于跳转）、JNA；JNBE、JA（高于跳转）

##### 根据两个带符号数比较结果转移

> JL（JNGE）/ JNL（JGE）: 小于跳转 / 不小于
>
> JLE（JNG）/ JNLE（JG）
>
> L - Less; G - Great; E - Equal
>
> 注意：所有条件转移指令都是段内（-128 ~ +127）范围内转移

### 过程（子程序）

> 子程序：程序中具有独立功能的部分编写成独立程序模块，通过PLC和ADDP实现
>
> - 子程序调用   CALL  子过程名
> - 返回指令   RET  在子程序结尾，用来返回主程序

### 循环控制指令

#### 无条件循环

> LOOP  语句标号
>
> 执行操作：
>
> * （CX） <-   (CX)  - 1
> * 若CX ≠ 0，转向目标地址去执行，否则执行LOOP指令之后的指令
>
> 解释：遇到这条指令，首先执行CX = CX - 1；然后判断CX 的值，若 CX ！= 0，则转移到 Lable 处执行程序，否则向下继续执行

#### 条件循环

![UO7nyT.png](https://s1.ax1x.com/2020/07/23/UO7nyT.png)

[![UO73k9.png](https://s1.ax1x.com/2020/07/23/UO73k9.png)](https://imgchr.com/i/UO73k9)

### 中断指令

> * 中断调用：INT  n
>   * n   -    中断号：0 ~ 255
> * 中断返回 ：IRET

## 处理器控制类指令

### 标志处理指令

![UOH8Hg.png](https://s1.ax1x.com/2020/07/23/UOH8Hg.png)

![UOHIbD.png](https://s1.ax1x.com/2020/07/23/UOHIbD.png)

# 8086/8088微处理器

组成：

* 运算器
* 控制器
* 寄存器

提出几个问题：

* 8086 / 8088 CPU能够实现指令并行流水工作的原因
* 实地址模式下的存储器地址变换原理
* 如何知道CPU当前工作状态及指令运算结果的特征

> 能够并行处理16位的二进制码，对内对外都是如此

<img src="https://img-blog.csdnimg.cn/20200729085935810.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

二者内部都是16位的体系结构，在外部：

* 8086 对外的通道就是数据总线是16位的
* 8088 是8位的

有助于与其他 8 位的芯片与接口连接，因为通道的宽度不一样，所以在一些引线的功能上有一些细微的区别。

## 工作模式

> * 最小模式 - 单处理器模式
> * 最大模式
>
> 总线连接：最大模式下会通过总线控制器与外部的控制总线连接，而最小模式是直接连接。

选择工作在哪个工作模式，8088 由MN/MX引线的状态决定。

## 主要引线

### 8086/8088在最小工作模式下访问一次内存或者接口的引脚

> ALE 地址锁存信号，决定了地址锁存器
>
> RESET 复位信号
>
> ALU  运算器

#### 地址信号

> 微处理器读取一条指令的控制过程（涉及到三大信号：地址信号、控制信号、数据信号）
>
> * 发出读取数据所在的目标地址
>   * 内存储器单元地址
>   * I/O接口地址
> * 发出读控制信号
> * 送出传输的数据

其地址总线的宽度是20位的，也就是20为地址信号（可以管理1M的地址单元，可以产生1M个编码）：

> 可以管理地址单元数取决于地址总线的宽度

* AD0-AD7：低8位地址和低8位数据信号分时复用，传送地址信号时为单向，传送数据信号时为双向（先有地址，后有数据）
* A16-A19：高4位地址信号，与状态信号分时复用
* A8-A15：8位地址信号

#### 主要控制信号

#代表上横线，代表低电平有效

#WR #RD分别是读写信号，上横线，低电平有效

读和写的对象到底是内存还是接口，取决于另外一个引脚IO/#M，为0表示访问内存，为1表示访问接口

> 虽然我们确定了读写的操作，由最小工作模式示意图可以得知，数据信号的输入和输出并不是数据总线直接与8086/8088的数据引线D0和D7相连，而是通过数据收发器的环节

针对这个环节的控制信号：

* #DEN：低电平有效，允许进行读/写操作，数据收发器的片选信号，其有效，数据才能进入CPU，就像ALE地址锁存器一样
* DT/#R：数据收发器的传送方向控制

是数据收发器的选通信号以及数据传输方向的控制信号

#### For Example

* 当#WR = 1，#RD = 0，IO/#M = 0时，表示在读存储器操作

### READY 信号

外部同步控制信号：CPU访问一次内存或者访问一次接口所需要的一个操作的周期，正常情况下一次需要的是 4 个时钟周期（一个周期基本上是0.2us）【也称为一个总线周期】，一个时钟周期就是时钟频率（基本上是5MHz）倒过来。

内存比CPU慢，接口更慢，所以不一定在4 个时钟周期内就能够完成，解决方案：

增加一个时钟周期，当CPU访问时发出一个访问信号，也就是在第3个时钟周期开始检测它的各个引脚

* 如果READY是高电平，那就是READY，送上第4 个时钟周期，一次访问就结束了
* 如果是低电平，那么就需要等待，等待就是在第三个时钟周期后插入一个时钟周期，称之为TW，等待周期，如果插入一个再去检测READY还是低电平，那就接着插入第二个TW，直到高电平

### 中断请求和相应信号

* INTR： 可屏蔽中断请求输入端（CPU可以不理睬）
* NMI：非屏蔽中断请求输入端
* #INTA：中断响应输出端

### 总线保持信号

> 工作在CPU直接存储器存取的时候，CPU将总线控制权交出
>
> * 一些高速的外部设备直接将数据写入内存或者直接读取

* HOLD：当CPU以外的其他设备要求占用总线时，通过该引脚向CPU发出请求
* HLDA：总线保持响应信号输出端，CPU对HOLD信号的响应信号

## 内部结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729144335468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729111753118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

BIU 总线接口单元，EU 执行单元

BIU与EU合称为8086、dao8088两大独立工作单元。其中BIU负责dao从内版存指定区域取出指令传权送到指令队列中排队；执行指令时所需要的操作数也由BIU从相应的内存区域取出，传送给执行部件EU。指令执行的结果如果需要存入内存的话，也由BIU写入相应的内存区域。总之，BIU同外部总线连接为EU完成所有的总线操作，并形成20位的内存物理地址。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729112402761.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

> 指令预取队列，使得EU 和BIU 两个部分可以同时进行工作，实现指令的并行执行，提高了效率，降低了存储器存取速度的要求

## 实模式存储器寻址与总线

### 实模式存储器寻址

* 内存分段管理思想
* 实模式下的内存地址变换
* 段寄存器的应用
* 堆栈的概念

> 8088需要管理1M的内存，也即利用16位的体系结构来产生一个20位地址，这样的能力要通过内存分段管理方式实现。
>
> 欲实现对1M内存空间的正确访问，每个内存单元在整个内存空间中必须具备唯一地址，内存地址变换就是：如何将直接产生的16位编码变换为20位的物理地址。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729151636412.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729151629361.png)

<font color="red">实际上是要把32位的逻辑地址变换为20位的物理地址。</font>

段基地址决定了存储单元在内存中的位置，逻辑段的起始地址（每个逻辑段内第一个单元）称为段首，任何一个段的段首地址都能被16整除，

> 内存的分段是逻辑分段，不是物理段，各个逻辑段在地址上可以不相连，可以部分重合可以完全重合，当一个单元在这个时间作为这个段使用时，使用的是这个段的段基地址：偏移地址，当另一个时刻做另一个段使用时，物理地址是不变的，由于偏移地址变为零，段基地址就是之前的物理地址除以10H

推出：也就是每个内存单元具有唯一的物理地址，但是可以有多个逻辑地址

* 一个内存单元可以在不同的时刻属于相同（或不同类型）的段。

* 一个内存单元可以在同一时刻属于不同类型的段，我们知道一个程序模块中，段的类型只有4种

编写程序时无法考虑段基地址，只能考虑偏移地址

> 内存中，一般的逻辑段我们关注的是段首以及某一个单元相对于段首的距离，对于堆栈段，我们关注的首先是栈底，然后是栈顶
>
> * 栈底 = 栈顶，空栈
> * 栈顶 = 栈首，满栈

## 总线

> * CPU 工作时序：CPU各引脚信号在时间上的关系
> * 总线周期：一个总线周期至少包括4个时钟周期

时序图：横轴是时间轴，纵坐标是各个引脚的幅值。所以看时序图通常是竖着看的，某一个时刻各个引脚之间的关系。

观察时序图，CLK是时钟信号，有一些信号有上有下，是总线信号，总线信号我们需要关注的是一组信号，而不是单个是否有效【鼓起来有效，平下去无效】，IO/#M比较特殊，虽然只有一根但是还是有上有下，因为这个信号有两种可能，可能高电平也可能低电平。

> 总线：是一组导线和相关的控制、驱动电路的集合，是计算机系统各部件之间传输地址、数据和控制信息的通道。
>
> * CPU总线
> * 系统总线
> * 外部总线【USB总线】

单总线结构会导致效率低而且总线争用导致拥塞，双总线结构有面向CPU的和面向主存的

* 面向CPU的双总线结构，存储器与I/O接口之间无直接通道，当需要高速的外部设备需要访问内存时，必须要经过CPU而不能直接访问内存
* 面向主存的双总线结构，在单总线的基础上增加了一条CPU到存储器的高速总线

> 功能：
>
> * 数据传送【同步、READY】
> * 仲裁控制【总线争用的情况下判定归谁使用】
> * 出错处理
> * 总线驱动

> 总线性能指标
>
> * 总线带宽，单位时间内总线上可传送的数据量
>   * 总线带宽 = 位宽 x 工作频率
> * 总线位宽，能同时传送的数据位数
> * 工作频率，总线带宽 = (位宽 / 8) x (工作频率 / 每个存取周期的时钟数)

# 汇编语言程序设计

> 汇编语言源程序：用助记符编写
>
> 汇编程序：源程序的编译程序
>
> 汇编语言程序  ->  汇编程序  ->  机器语言目标程序（计算机能够识别）

* 输入汇编语言源程序 -> 源文件.ASM
* 汇编  ->  目标文件.OBJ
* 链接  ->  可执行文件.exe

> 汇编语言语句类型：
>
> * 指令性语句：CPU执行的语句，能够生成目标代码
> * 指示性语句：CPU不执行，由汇编程序执行的语句

## 操作数 

> 操作数：
>
> * 寄存器
> * 存储器单元
> * 常量
> * 变量或标号
> * 表达式

<font color="red">字符串常量通常是加单引号代表的，如果不加单引号就会按照数字常量处理，例如ABCD，不加单引号就需要加H，为ABCDH代表16进制数，进一步，如果操作数的第一位是字母，为了区分数字或者字符串，数字时需要加上0，为0ABCDH</font>

> 变量
>
> 某一个内存单元的符号地址，为存储器操作数
>
> 属性：
>
> ​	段值  —- 变量所在段的段地址
>
> ​	偏移量 — 变量所指单元的偏移地址
>
> ​	类型 — 字节型、字形、双字型

## 运算符

> **取值运算符：**
>
> 用于分析存储器操作数的属性
>
> 获取变量的属性值
>
> * OFFSET  –  取得其后变量或标号的偏移地址【LEA指令也可以】
> * SEG  – 取得其后变量或标号的段地址
>
> ```commonlisp
> MOV  AX, SEG  DATA  ;  取变量DATA段地址送给AX
> 
> MOV DS,  AX  ;  把段地址送给DS
> 
> MOV BX,  OFFSET  DATA   ;   取变量偏移地址送给BX【等价于LEA  BX，DATA】
> ```

> **属性运算符**  PTR
>
> 用于指定其后存储器操作数的类型，单操作数格式指令中，如果操作数是存储器操作数必须要声明字长，声明办法就是利用属性运算符
>
> MOV  BYTR  PTR[BX], 12H   ; 把12H送到BX所指目标存储器单元中
>
> 这里用PTR 声明了BX的字长，就是一个bytr，指定存储器操作数[BX] 为字节型，所以源操作数就是12H，当然如果 PTR 前面的不是 BYTR，而是 WORD， 那源操作数就不再是12H，而是0012H

## 伪指令

> 伪指令：帮助计算机理解助记符指令编写的汇编语言程序

### 数据定义伪指令

> 数据定义伪指令决定所定义变量的类型
>
> 定义字符串必须使用DB伪指令

<img src="https://img-blog.csdnimg.cn/20200729223337919.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200729215325361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom: 67%;" />

<img src="https://img-blog.csdnimg.cn/20200729222835106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200729222746674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

#### 重复操作符

> 常用于声明一个数据区
>
> 当同样的操作数重复多次时，可以使用重复操作符，为一个数据区的各单元设置相同的初值。
>
> For Example：M1  DB  10  DUP  (0)

#### ? 号

> 表示随机值，用于预留存储空间，占 1 个字节单元
>
> For Example：
>
> * MEM1  DB  34H,  ‘A’,  ?
> * DW  20  DUP (?)  预留了40个字节单元，每单元为随机值

<img src="https://img-blog.csdnimg.cn/20200729224930969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200729225025374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200729224930969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200729225025374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

### 符号定义伪指令

<img src="https://img-blog.csdnimg.cn/20200730064136181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

> EQU 说明的表达式不占用内存空间

### 段定义伪指令

<img src="https://img-blog.csdnimg.cn/20200730064842915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200730065327966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200730064842915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200730065327966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

### 设定段寄存器伪指令

> 说明所定义逻辑段的性质
>
> 源程序中必须声明一个代码段

```commonlisp
ASSUME 段寄存器名字：段名[，段寄存器名：段名，...]
```

### 结束伪指令

> 表示源程序结束
>
> END  [标号]

### 过程定义伪指令

> 定义一个过程体
>
> 执行一条过程调用CALL指令时，系统有两项工作，找入口地址和保护断点
>
> * 系统自动把CALL指令的下一条指令的偏移地址压入到堆栈中

```commonlisp
过程名  PROC[NEAR / FAR]
	.
	.
	.
	RET
过程名  ENDP

; 过程名：实际上就是过程的入口地址，即第一条指令在内存中的符号地址
; NEAR/ FAR：若为近过程，NEAR可以省略，远过程FAR是不可以省略的
; RET：返回断点
```

<img src="https://img-blog.csdnimg.cn/20200730072325731.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" style="zoom:50%;" />

### 调整偏移量伪指令

<img src="https://img-blog.csdnimg.cn/20200730082531360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom: 67%;" />

## 完整源程序示例

<img src="https://img-blog.csdnimg.cn/20200730070150237.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200730070920583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

只有经过段寄存器初始化，前面声明的那些段才会真正占据了内存，之前只是定义了，只有代码段不需·要初始化

> MOV 不允许为段寄存器直接赋值，所以不可以直接用立即数当作源操作数，要使用通用寄存器

## 系统功能调用

> 调用BIOS / DOS功能

<img src="https://img-blog.csdnimg.cn/20200730083113793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

> 软中断指令调用，中断类型码：21H

<img src="https://img-blog.csdnimg.cn/20200730083435658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom: 67%;" />

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730084427564.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

> 没有入口参数，输入就是键盘，运行完这两行代码屏幕上会出现光标，然后需要输入字符，键盘扫描吗转换为ASCII码，存放在出口参数AL中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730084733726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

> 不能总是输入单个字符，当我们输入字符串时，不能全部存在寄存器中，而需要存放在内存中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730085421649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

> N1是字节，所以缓冲区大小必须小于255

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730090230612.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

首先把10号功能送进AL中，缓冲区要求必须定义在数据段，缓冲区首地址必须给DX

<font color="red">注意：这里的DX不是间指寄存器之一，指令系统中，能够放进方括号作为间接寄存器表示偏移地址的只有DX、DP、SI、DI</font>

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020073009053153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730090825785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

> 字符串的输出显示
>
> AH   功能号 09H
>
> DS：DX    待输出字符串的偏移地址
>
> INT   21H

> 要输出必须要先定义，且必须定义在数据段，而且首地址必须由DX标识

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730091237251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730091547428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730091652689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730091822691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

# 汇编程序设计示例

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730103113271.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

> 串操作：每一条指令都是一个循环体，那么要推出这个循环就需要条件，对于所有串操作指令，我们都要知道串长度值和操作方向
>
> * 串长度值必须送给CX 

# 存储器

> 内存条一般都由8片芯片组成，原因在于：
>
> 每一个芯片上不管有多少个单元，每个单元里不是8位二进制码，如果是8片的话，每个单元里只有1位，也就是说可以描述为 4G  x  1b

容量的描述：

* 内存，8G、4G字节，我们直接把字节忽略掉了，因为我们认定每个字节里一定是8位二进制码，1字节数
* 芯片，？ x  ？b，多少乘以多少位，例如 8k  x  8b

<img src="https://img-blog.csdnimg.cn/20200731070714560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />



## 实现存储器芯片和系统的连接

<img src="https://img-blog.csdnimg.cn/20200731074955931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom: 67%;" />

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020073107522027.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731081025644.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731081425154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

> EEPROM写操作：
>
> * 根据参数定时写入
> * 通过判断READY/#BUSY端的状态进行写入
>   * 仅当该端为高电平时才可写入下一个字节
> * 中断控制方式
>   * 当READY/#BUSY端为高电平时，该高电平可作为中断请求信号

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020073109415832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

## 存储器扩展技术

> 用已有的多片存储器芯片构造一个需要的存储空间

> 存储器芯片的存储容量等于  单元素  x  每单元的位数
>
> * 单元素   —   字节数
> * 每单元位数   —  字长
>
> 哪个不满足就扩展哪个

### 第一题

<img src="https://img-blog.csdnimg.cn/20200801113649113.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

### 第二题

<img src="https://img-blog.csdnimg.cn/20200801113917182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200801114327314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/20200801114353477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801114649808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)

<img src="https://img-blog.csdnimg.cn/20200801114353477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801114649808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801114812139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Jva29CYXNpbGlzaw==,size_16,color_FFFFFF,t_70)