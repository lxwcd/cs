深入理解计算机系统——第二章 Representing and Manipulating Information  
      
> [深入理解计算机系统 课程视频](https://www.bilibili.com/video/BV1iW411d7hd?p=2)  
> [进制转换（二进制、八进制、十进制、十六进制）超详细](https://zhuanlan.zhihu.com/p/459817484)  
      
# 2.1 Information Storage  
**bytes:**  Rather than accessing individual bits in memory, most computers use blocks of eight bits, or ***bytes***, as the smallest addressable unit of ***memory***.  
      
**memory:** A machine-level program views memory as a very large array of bytes, referred to as ***virtual memory***.  
      
**virtual address space:** Every byte of memory is identified by a unique number, known as its ***address***, and set of memory is identified by a unique number, known as the ***virtual address space***.  
[Understanding virtual address and virtual address space](https://stackoverflow.com/questions/9414565/understanding-virtual-address-and-virtual-address-space)  
      
**program objects:** program data, instructuctions and control information.  
      
## 2.1.1 Hexadecimal Notation  
**hexadecimal numbers:** write bit patterns as base-16. Hexadecimal uses digits `0` through `9` along with characters `A` through `F` to represent 16 possible values. (characters `A` through `F` may be written in either upper or lower case)  
      
In C, numeric constants starting with `0x` or `0X` are interpreted as being in hexadecimal.  
      
**Converting between decimal and hexadecimal:**  
To convet a decimal number `x` to hexadecimal, we can repeatedly devide `x` by `16`, giving a quotient `q` and demainder `r`, such that `x = q * 16 + r`. We then use the hexadecimal digit representing r as the ***least significant digit*** and generate the remaining digits by repeating the process on `q`.  
      
Conversely, to convert a hexadecimal number to decimal, we can multiply each of the hexadecimal digits by the approprite power of 16.  
      
转换方法适用于其他进制之间转化（如十进制和二进制）。  
      
## 2.1.2 Data Size  
**words:** every computer has a ***word size***, indicating the nominal size of pointer data. Since a ***virtual address*** is encoded by such a word,, the most important system parameter determined by the word size is the maximum size of the virtual address space. That is, for a machine with a `w-bit` word size, the virtual addresses can range from 0 to 2<sup>w </sup>-1, giving the program access to at most 2<sup>w </sup>bytes.  
      
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112118081270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
      
The C language allows a variety of ways to order the keywords and to include or omit optional keywords. As examples, all of the following declarations have identical meaning:  
>unsigned long  
unsigned long int  
long unsigned  
long unsigned int  
      
The above figure shows that a pointer uses the full word size of the machine.  
      
## 2.1.3 Addressing and Byte Ordering  
For program objects that span multiple bytes, we must establish two conventions: **what the address of the object will be, and how we will order the bytes in memory.**  
      
In virtually all machines, a multi-byte object is stored as a contiguous sequence of bytes, with the address of the object given by the smallest address of the bytes used.  
      
For ordering the bytes representing an object, there are two common conventions.  
      
Some machines choose to store the object in memory ordered from least significant byte to most, while other machines  
store them from most to least.  
      
**little endian:**  the least significant byte comes first.  
      
**big endian:** the most significant byte comes first.  
      
## 2.1.7 Bit-Level Operations in C  
One useful feature of C is that it supports bitwise Boolean operations. In fact, the symbols we have used for the Boolean operations are exactly those used by C:  
>| for or, & for and, ~ for not, and ^ for exclusive-or.  
      
## 2.1.8 Logical Operations in C  
C also provides a set of logical operators `||`, `&&`, and `!`, which correspond to the `or`, `and`, and `not` operations of logic.  
      
**distinction between logical operators and bit-level operators:**  
      
1. The logical operations treat any `nonzero` argument as representing `true` and argument `0` as representing `false`.  
They return either `1` or `0`, indicating a result of either `true` or `false`, respectively.  
      
2. logical operators **do not evaluate their second argument** if the result of the expression can be determined by evaluating the **first argument**.  
      
## 2.1.9 Shift Operations in C  
Shift operations associate from **left to right**, so $x << j << k$ is equivalent to $(x << j) << k$.  
      
**left shift:** For an operand x having bit representation  $[x_{w−1}, x_{w−2}, . . . , x_0]$, the C expression `x << k` yields a value with bit representation $[x_{w−k−1}, x_{w−k−2}, . . . , x_0,0, . . . , 0]$. That is, `x` is shifted `k` bits to the `left`, dropping off the `k most significant bits` and filling the right end with `k zeros`. The `shift amount` should be a value between 0 and `w − 1`.  
      
**right shift:**  
1. **Logical**. A logical right shift fills the left end with `k zeros`, giving a result  
$[0, . . . , 0, x_{w−1}, x_{w−2}, . . . x_k]$.  
      
2. **Arithmetic**. An arithmetic right shift fills the left end with `k repetitions of the most significant bit`, giving a result $[x_{w−1}, . . . , x_{w−1}, x_{w−1}, x_{w−2}, . . . x_k]$.  
This convention might seem peculiar, but as we will see, it is useful for operating on ***signed integer data***.  
      
算数右移在左侧高位填充**符号位**，在补码除法中需用到算数右移。  
      
# 2.2 Integer Representations  
## 2.2.1 Integer Data Types  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121192722875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121192745640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121192921836.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
## 2.2.2 Unsigned Encodings  
We write a bit vector as either $\vec{x}$, to denote the entire vector, or as $[x_{w−1}, x_{w−2}, . . . , x_0]$ to denote the individual bits within the vector.  
      
We can express this interpretation as a function **B2U<sub>w</sub>** (for “binary to unsigned,” length w):  
      
For vector $\vec{x}$ = $[x_{w−1}, x_{w−2}, . . . , x_0]$:  
$$B2U_w(\vec{x}) = \sum_{i=0}^{w-1} x_i2^i	$$  
      
## 2.2.3 Two’s-Complement Encodings  
The most common computer representation of signed numbers is known as ***two’s-complement*** form. This is defined by interpreting the `most significant bit` of the word to have `negative weight`. We express this interpretation as a function **B2T<sub>w</sub>** (for “binary to two’s complement” length w):  
      
For vector $\vec{x}$ = $[x_{w−1}, x_{w−2}, . . . , x_0]$:  
$$B2T_w(\vec{x}) = -x_{w-1}2^{w-1} + \sum_{i=0}^{w-2} x_i2^i	$$  
      
The most significant bit $x_{w−1}$ is also called the ***sign bit***.  
      
$TMin_w = -2^{w-1}$  
$TMax_w = 2^{w-1} - 1$  
      
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191121195048700.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)**A few points are worth highlighting about these numbers.**  
      
1. The two’s-complement range is asymmetric:  
**|TMin| = |TMax| + 1**;  
that is, there is no positive counterpart to `TMin`.  
      
This asymmetry arises because **half the bit patterns** (those with the sign bit set to 1) represent **negative** numbers, while **half** (those with the sign bit set to 0) represent **nonnegative** numbers.  
      
Since 0 is nonnegative, this means that it can represent one less positive number than negative.  
      
2.  **UMax = 2TMax + 1**  
      
All of the bit patterns that denote negative numbers in two’s-complement notation become positive values in an unsigned representation.  
      
## Alternative representations of signed numbers  
There are two other standard representations for signed numbers:  
      
### Sign magnitude  
**The most significant bit** is a **sign bit** that determines whether the remaining bits should be given negative or positive weight:  
      
$$B2S_w(\vec{x}) = (-1)^{x_{w-1}} \sum_{i=0}^{w-2}x_i2^i$$  
      
**disadvantage:** that there are two different encodings of the number 0.  
      
### Ones’ complement  
This is the same as two’s complement, except that the most significant bit has weight $−(2^{w−1} − 1)$ rather than $−2^{w−1}$  
      
      
$$B2T_w(\vec{x}) = -x_{w-1}(2^{w-1}-1) + \sum_{i=0}^{w-2} x_i2^i	$$  
      
从公式可以看出： 对于负数：反码 = 1 - 补码  
即：负数的补码 = 反码 + 1。  
对正数，$x_{w-1}$ 是0， 因此 补玛与原码，反码相同。  
      
The term “**two’s-complement**” arises from the fact that for nonnegative x we compute a w-bit representation of −x as $2^w − x$ (a single two.)  
例如： 对于  `w` 是 4 bit， -6 的补码是 1010， 6的补码是 0110，`模` $2^w$ 是16，16 - 6 = 10 （1010）。  
对于补码， 可以想时钟模型，时钟圈一个周期 `12` 小时， 即模为 `12`， 正数表示 `顺时针`， 负数表示 `逆时针`， `-3` 则为`逆时针`转 `3` 小时，对应`顺时针`转 `9` 小时到同样位置。  
      
The term “**ones’ complement**” comes from the property that we can compute −x in this notation as [111 . . . 1]− x (multiple ones).  
      
反码， [111 . . . 1] 和 [000 . . . 0] 都表示0。前者是 -0， 后者 +0。  
      
因此， 反码和符号量都有缺陷。  
      
## 2.2.4 Conversions between Signed and Unsigned  
> the effect of casting is to keep the bit values identical but change how these bits are interpreted  
      
function **T2U** describes the **conversion** of a **two’scomplement** number to its **unsigned**  counterpart, while **U2T** converts in the opposite direction.  
      
Now define the function $T2U_w$ as $T2U_w(x) =. B2U_w(T2B_w(x))$.  
      
For x such that $TMin_w ≤ x ≤ TMax_w$:  
$$T2U_w(x) =  
\begin{cases}  
x + 2^w,& \text{$x \lt 0$} \\[2ex]  
\ \ \ x,& \text{$x\geq0$}  
\end{cases}$$  
      
**Unsigned to two’s-complement conversion:**  
      
For u such that $0 ≤ u ≤ UMax_w$:  
$$U2T_w(u) =  
\begin{cases}  
\ \ \ u,& \text{$u \leq TMax_w$} \\[2ex]  
u - 2^w,& \text{$u \gt TMax_w$}  
\end{cases}$$  
      
将补码转换为unsigned，比特位数值并未改变，如当 w = 4时，`-5` 补码为 `1011` ，转换为 `unsigned`则会解析为 `11`。  
      
## 2.2.5 Signed versus Unsigned in C  
      
Generally, most numbers are signed by default.  
      
C allows conversion between unsigned and signed. Although the C standard does not specify precisely how this conversion should be made, most systems follow the rule that **the underlying bit representation does not change**.  
      
**Conversions can happen as follows:**  
      
**1. explicit casting**  
```  
int tx, ty;  
unsigned ux, uy;  
tx = (int) ux; // us casts to int  
uy = (unsigned) ty; // ty casts to unsigned  
```  
**2. assignment**  
      
from right to left  
```  
int tx, ty;  
unsigned ux, uy;  
tx = ux; // cast to signed  
uy = ty; // casr to unsigned  
```  
**3. printf**  
When printing numeric values with printf, the directives %d, %u, and %x are used to print a number as a signed decimal, an unsigned decimal, and in hexadecimal format, respectively.  
      
**Note that printf does not make use of any type information**, and so it is possible to print a value of type int with directive %u and a value of type unsigned with directive %d.  
```  
int x = -1;  
unsigned u = 2147483648; /* 2 to the 31st */  
printf("x = %u = %d\n", x, x);  
printf("u = %u = %d\n", u, u);  
```  
```  
x = 4294967295 = -1  
u = 2147483648 = -2147483648  
```  
**4. expressions containing combinations of signed and unsigned quantities**  
      
**低级（表示范围小）**转化为**高级（表示范围大）**  
      
如：-1 < 0U  
左边 signed， 有边 unsigned， 因此左边转化为 unsigned，然后进行操作运算，因此该表示为假。  
      
## 2.2.6 Expanding the Bit Representation of a Number  
> One common operation is to convert between integers having different word sizes while retaining the same numeric value.  
      
扩展数值的比特位后保持结果不变。  
      
**1. Expansion of an unsigned number by zero extension**  
      
Define bit vectors $\vec u = [u_{w−1}, u_{w−2}, . . . , u_0]$ of width w and $\vec {u'}$ = [0, . . . , 0, $u_{w−1}, u_{w−2}, . . . , u_0]$ of width w', where w' > w.  
      
Then $B2U_w$$(\vec u)$ = $B2U_{w'}(\vec {u'})$.  
      
对于无符号整数，扩大其尺寸只需在高位填 `0`。  
      
**2. Expansion of a two’s-complement number by sign extension**  
      
Let `w' = w + k`. What we want to prove is that:  
      
$$B2T_{w+k}([\underbrace{{\color{blue}x_{w-1}}, . . . , {\color{blue}x_{w-1}}}_\text{k times}, {\color{blue}x_{w-1}}, x_{w-2}, . . . , x_0]) = B2T_w([{\color{blue}x_{w-1}}, x_{w-2}, . . . , x_0])$$  
      
Thus, the task can be reduced to prove that:  
      
$$B2T_{w+1}([{\color{blue}x_{w-1}}, {\color{blue}x_{w-1}}x_{w-2}, . . . , x_0]) = B2T_w([{\color{blue}x_{w-1}}, x_{w-2}, . . . , x_0])$$  
      
$$\begin{aligned}  
B2T_{w+1}([{\color{blue}x_{w-1}}, {\color{blue}x_{w-1}}x_{w-2}, . . . , x_0]) &= -{\color{blue}x_{w-1}}2^w + {\color{blue}x_{w-1}}2^{w-1} + \sum_{i=0}^{w-2}x_i2^i \\[6ex]  
&= -{\color{blue}x_{w-1}}(2^w - 2^{w-1}) + \sum_{i=0}^{w-2}x_i2^i \\[6ex]  
&= -{\color{blue}x_{w-1}}2^{w-1} + \sum_{i=0}^{w-2}x_i2^i \\[6ex]  
&= B2T_w([{\color{blue}x_{w-1}}, x_{w-2}, . . . , x_0])  
\end{aligned}$$  
      
对于有符号位的整数，在高位填充其符号位后数值保存不变。  
      
## 2.2.7 Truncating Numbers  
**1. Truncation of an unsigned number**  
      
$$\begin{aligned}  
B2U_w([x_{w-1}, x_{w-2}, . . . , x_0]) \ mod \ 2^k &= \Biggl[  
\sum_{i=0}^{w-1}x_i2^i\Biggr] \ mod \ 2^k \\[4ex]  
&=  \Biggl[  
\sum_{i=0}^{k-1}x_i2^i\Biggr] \ mod \ 2^k \\[4ex]  
&= \sum_{i=0}^{k-1}x_i2^i \\[4ex]  
&= B2U_k([x_{k-1}, x_{k-2}, . . . , x_0])  
\end{aligned}$$  
      
In this derivation, we make use of the following property:  
      
$$\begin{cases}  
2^i \ mod \ 2^k = 0 &\text{(i$\geq k$)} \\[2ex]  
2^i \ mod \ 2^k = 2^i &\text{(i < k)} \\[1ex]  
\end{cases}$$  
      
对于无符号整数，直接将多出的高位去掉。  
      
**2. Truncation of a two’s-complement number**  
      
> A similar property holds for truncating a two’s-complement number, except that it then converts the most significant bit into a sign bit.  
      
$$B2T_k([x_{k−1}, x_{k−2}, . . . , x_0]) = U2T_k(B2U_w([x_{w−1}, x_{w−2}, . . . , x_0]) \ mod 2^k)$$  
      
将补码的比特位减小，同样是直接去掉高位多余的位，然后解析的时候将剩下的最高位当作符号位。  
# 2.3 Integer Arithmetic  
## 2.3.1 Unsigned Addition  
Let us define the operation $+^u_w$  for arguments `x` and `y`, where 0 ≤ x, y < $2^w$,  
      
as the result of truncating the integer sum `x + y` to be `w` bits long and then viewing the result as an unsigned number.  
      
This can be characterized as a form of modular arithmetic, computing the sum modulo $2^w$ by simply discarding any bits with weight greater than $2^{w−1}$ in the bit-level representation of `x + y`.  
      
For x and y such that 0 $\leq x$, y < $2^w$:  
      
$$x + ^u_wy = \begin{cases}  
x + y, &\text{x + y < $2^w$ \ \ Normal} \\[2ex]  
x + y - 2^w, &\text{$2^w$ $\leq {x + y}$ < $2^{w+1}$ \  \ Overflow}  
\end{cases}$$  
      
两个无符号整数相加，如果结果超过最大范围，则其值为 x + y - $2^w$  
      
**Unsigned negation**  
For every value x, there must be some value $-^u_w x$ such that -$\pmb{^u_w x }$ + $\pmb{^u_w x = 0}$.  
      
For any number x such that 0 ≤ x < $2^w$, its `w-bit unsigned negation` $-^u_w x$  is given by the following:  
      
$$-^u_w x = \begin{cases}  
x, &\text{x = 0} \\[2ex]  
2^w - x, &\text{x > 0} \\[1ex]  
\end{cases}$$  
      
无符号整数取反  
例如： w 为 4 bit， unsigned x = 6， 表示为 0110，则 $-^u_4 6$ 为 10， 表示为 1010，两者相加为 0。这里 negation 相当于时钟逆时针走，如果模为 16，顺时针走 6 小时， 相当于逆时针走 10 小时。  
      
## 2.3.2 Two’s-Complement Addition  
For integer values x and y in the range $−2^{w−1} ≤ x,  y ≤ 2^{w−1} − 1$:  
      
$$x + ^t_w y = \begin{cases}  
x + y - 2^w, & \text{$2^{w-1}\leq {x + y}$\ \ \ Positive overflow} \\[2ex]  
x + y, &\text{$-2^{w-1}\leq {x + y} < 2^{w-1}$ \ \ \ Normal}\\[2ex]  
x + y + 2^w, &\text{$x + y < -2^{w-1}$ \ \ \ Negative overflow}\\[2ex]  
\end{cases}$$  
      
## 2.3.3 Two’s-Complement Negation  
      
For x in the range $TMin_w ≤ x ≤ TMax_w$, its two’s-complement negation $-^t_w x$ is given by the formula:  
      
$$-^t_w x = \begin{cases}  
TMin_w, &\text{x = $TMin_w$} \\[2ex]  
-x, &\text{x > $TMin_w$} \\[1ex]  
\end{cases}$$  
      
## negate a number  
将一个数取反再加1。适用于 unsigned 和 signed。  
      
如：对 unsigned 6， 表示为 0110， 取反加1 后的 1010，与前面结果相同。  
      
对 signed 6， 1010 即为补码 -6，正确。  
      
对 signed -6， 表示为 1010， 取反加1 后为 0110， 即 6。正确。  
      
negate a number 其实就是取其补码，当前二进制位与其补码相加后为0。  
而原码与反码相加后得到全1，再加1得到 0。  
      
## 2.3.4 Unsigned Multiplication  
      
For x and y such that $0 ≤ x, y ≤ UMax_w$:  
      
$$x *\ ^u_w y = (x . y) \ mod \ 2^w$$  
      
##  2.3.5 Two’s-Complement Multiplication  
Truncating a two’s-complement number to w bits is equivalent to first computing its value modulo $2_w$ and then converting from unsigned to two’s complement, giving the following:  
      
For x and y such that $TMin_w ≤ x, y ≤ TMax_w$:  
      
$$x *\ ^t_w y = U2T_w((x . y) \ mod \ 2^w)$$  
      
补码的乘法需要扩展符号位计算，因此乘积的全部比特位和无符号计算结果不相同，见下表：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191122214846981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
      
相关计算说明见：  
[关于补码（有符号）乘法遇到的疑惑](https://www.jianshu.com/p/47b46439f695)  
[how to do two complement multiplication and division of integers?](https://stackoverflow.com/questions/20793701/how-to-do-two-complement-multiplication-and-division-of-integers)  
      
前面扩展比特位部分介绍过，对于补码在高位填充符号位后其数值不变，因此乘法先扩展符号位再计算。  
      
## 2.3.6 Multiplying by Constants  
$$\begin{aligned}  
B2U_{w+k}([x_{w-1}, x_{w-2}, . . . , x_0, {\color{blue}0}, . . . , {\color{blue}0}]) &= \sum_{i=0}^{w-1}x_i2^{i+k} \\[4ex]  
&= \Biggl[\sum_{i=0}^{w-1}x_i2^{i}\Biggr] \cdot 2^k \\[4ex]  
&= x2^k  
\end{aligned}$$  
      
When shifting left by k for a fixed word size, the high-order k bits are discarded (truncating the high-order k bits), yielding:  
$$[x_{w-k-1}, x_{w-k-2}, . . . , x_0, {\color{blue}0}, . . . , {\color{blue}0}]$$  
      
So, for 0 $\leq k$ < w, the C expression `x << k` yields the value $x2^k\ mod \ 2^w = x *^u_w 2^k$.  
      
**Two’s-complement multiplication by a power of 2**  
      
Since the bit-level operation of fixed-size two’s-complement arithmetic is equivalent to that for unsigned arithmetic, we can make a similar statement about the relationship between left shifts and multiplication by a power of 2 for two’s-complement arithmetic：  
      
such that 0 ≤ k < w, the C expression x << k yields the value $x * \ ^t_w 2^k$  
      
**Note that multiplying by a power of 2 can cause overflow with either unsigned or two’s-complement arithmetic.**  
      
整数乘法实际为向左位移的过程，如 `w` 为 `4`，`x` 为 `2`，即 `0010`，计算 `2 * 5`：  
根据 5 =  $2^2$ + $2^0$，即将 `0010` 先向左移 `2` 位，得到 `1000` ，再加上 `0010` 左移 `0` 位（即保持不变）的结果，最后得到 `1010` ，即 `10`。  
      
注意：左移时右侧补 `0`，左侧高位去掉，即使用`逻辑左移`。  
      
补码乘法计算和无符号整数相同？  
      
## 2.3.7 Dividing by Powers of 2  
Dividing by a power of 2 can also be performed using shift operations, but we use a right shift rather than a left shift. The two different right shifts—**logical and arithmetic**—serve this purpose for unsigned and two’s-complement numbers, respectively.  
      
除法计算采用右移，但和乘法不同，`无符号整数`采用`逻辑右移`，而`补码`则采用`算数右移`。  
      
**Integer division always rounds toward zero.**  
      
**Notation:**  
1. $\lfloor$a$\rfloor$  
      
For any real number a, define $\lfloor$a$\rfloor$ to be the unique integer `a'` such that `a' ≤ a < a' + 1`.  
      
As examples, $\lfloor3.14\rfloor$ = 3, $\lfloor−3.14\rfloor$ = −4, and $\lfloor3\rfloor$  = 3.  
      
2. $\lceil a\rceil$  
      
Similarly, define a to be the unique integer a' such that a' − 1 < a ≤ a'.  
      
As examples, $\lceil3.14\rceil$ = 4, $\lceil-3.14\rceil$= −3, and $\lceil3\rceil$ = 3.  
      
For $x \geq 0$ and $y \geq  0$, integer division should yield $\lfloor x/y\rfloor$.  
      
while for $x \lt 0$ and $y \gt 0$, it should yield $\lceil x/y\rceil$.  
      
That is, it should **round down a positive** result but **round up a negative one**.  
      
对于正数相除，结果取下限；对于负数，结果取上限。  
      
### Unsigned division by a power of 2  
Performing `logical right shift` for unsigned division by a power of 2.  
      
### Two’s-complement division by a power of 2  
The case for dividing by a power of 2 with `two’s-complement arithmetic` is slightly more complex.  
      
1. the shifting should be performed using an **arithmetic right shift**, to ensure that negative values **remain negative**.  
      
   **However**, this causes the result to be rounded downward rather than toward zero.  
      
2. correcting this improper rounding by **"biasing"** the value before shifting:  
          
    This technique exploits the following property :  
          
    $$\lceil x/y\rceil = \lfloor (x + y - 1)/y\rfloor  \qquad \text{(y > 0)}$$  
      
   To prove the above formula, suppose that $x = ky + r$, where $0 \leq r \lt y$, giving $(x + y - 1)/y = k + (r + y - 1)/y$, and so $\lfloor (x + y - 1)/y\rfloor = k + \lfloor (r + y - 1)/y\rfloor$.  
      
   If $r = 0$, $0 \lt (y - 1)/y \lt 1$, so the latter term will equal 0.  
      
   If $r > 0$, because $r$ is an integer, so $r \geq 1$.  
   $(r + y - 1)/y = 1 + (r - 1)/y$  
   $0 \leq (r - 1)/y \lt 1$  
   Thus, the latter term will equal to 1.  
         
   The C expression:  
   $$(x \lt 0 \ ?\  x + (1 \lt \lt k)-1 : x) \gt \gt k$$  
          
    Note: $1 \lt \lt k$ equals to $2^k$  
      
# 2.4 Floating Point  
## 2.4.2 IEEE Floating-Point Representation  
The IEEE floating-point standard represents a number in a form $V$ = $(−1)^s \times M \times 2^E$:  
      
 - The sign $s$ determines whether the number is negative ($s$ = $1$) or positive ($s$ = $0$), where the interpretation of the sign bit for numeric value $0$ is handled as a special case.  
       
 - The *significand* $M$ is a fractional binary number that ranges either between $1$ and $2 - \epsilon$ or between $0$ and $1 − \epsilon$ .($\epsilon$ is usually $2^{-k} \ (k \gt 0$))  
       
 - The *exponent* $E$ weights the value by a (possibly negative) power of 2.  
       
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123130838278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
      
The bit representation of a floating-point number is divided into three fields to encode these value:  
      
 - The single sign bit $s$ directly encodes the sign $s$.  
 - The $k$-bit exponent field $exp = e_{k−1} \cdots e_1e_0$ encodes the exponent $E$.  
 - The $n$-bit fraction field $frac = f_{n−1} \cdots  f_1f_0$ encodes the *significand* $M$, but the value encoded also depends on whether or not the *exponent* field equals $0$.  
      
In the **single-precision** floating-point format (a float in C), fields $s$, $exp$, and $frac$ are $1$, $k$ = $8$, and $n$ = $23$ bits each, yielding a $32$-bit representation.  
      
In the **double-precision** floating-point format (a double in C), fields $s$, $exp$, and $frac$ are $1$, $k$ = $11$, and $n$ = $52$ bits each, yielding a $64$-bit representation.  
      
### Case 1: Normalized Values  
This is the most common case. It occurs when the bit pattern of *exp* is neither all zeros (numeric value $0$) nor all ones (numeric value $255$ for single precision, $2047$ for double).  
      
In this case, the exponent field is interpreted as representing a **signed integer in biased form**.  
      
That is, the exponent value is $E$ = $e − Bias$, where $e$ is the unsigned number having bit representation $e_{k−1} \cdots e_1e_0$ and *Bias* is a bias value equal to $2^{k−1} − 1$ ($127$ for single precision and $1023$ for double).  
      
This yields exponent ranges from $−126$ to $+127$ for single precision and $−1022$ to $+1023$ for double precision.  
      
#### 偏移值 Bias  
**阶码 $E$ 用偏移的目的**  
因为指数可能是负数，为了不在阶码中引入符号位， 采用阶码形式将数值分成负数和非负数， 而无需用补码形式。  
      
**阶码 $E$ 偏移值的选取**  
偏移值选范围的中间值，而 **Normalized** 形式无全 $0$ 和全 $1$，因此范围是 $1 ～ 254$ （单精度），中间值即为 $127$，表示范围实际是 $-126 ～ 127$。  
      
The fraction field ***frac*** is interpreted as representing the fractional value $f$, where $0 \leq f \lt 1$, having binary representation $0.f_{n−1} \cdots f_1f_0$, that is, with the binary point to the left of the most significant bit.  
      
The ***significand*** is defined to be $M = 1 + f$.  
      
This is sometimes called an ***implied leading 1 representation***, because we can view $M$ to be the number with binary representation $1.f_{n−1}f_{n−2}\cdots f_0$.  
      
This representation is a trick for getting **an additional bit of precision for free**, since we can always adjust the exponent $E$ so that ***significand*** $M$ is in the range $1 \leq M \lt 2$.  
      
### Case 2: Denormalized Values  
When the **exponent field** is **all zeros**, the represented number is in ***denormalized*** form.  
      
In this case, the exponent value is $E = 1 − Bias$, and the ***significand*** value is $M = f$ , that is, the value of the ***fraction*** field **without an implied leading** $1$.  
      
#### Purpose of denormalized numbers  
1. They provide a way to represent numeric value $0$, since with a **normalized** number we must always have $M \geq 1$, and hence we cannot represent $0$.  
In fact, the floating-point zero has two representations: $+0.0$ and $-0.0$.  
$+0.0$: a bit pattern of all zeros.  
$-0.0$: sign bit is $1$, the other fields are all zeros.  
      
2. Representing numbers that are very close to $0.0$. They provide a property known as ***gradual underflow*** in which possible numeric values are spaced evenly near $0.0$.  
      
#### Why set the bias this way for denormalized values?  
Having the exponent value be $1 − Bias$ rather than simply $−Bias$ might seem counterintuitive. We will see shortly that it provides for smooth transition from denormalized to normalized values.  
      
最小的规格化值的 $E = 1 - Bias$，为了让非规格化和规格化值平滑过度。  
      
### Case 3: Special Values  
      
A final category of values occurs when the ***exponent*** field is **all ones**.  
      
When the ***fraction*** field is **all zeros**, the resulting values represent infinity, either $+\infty$ when $s = 0$ or $−\infty$ when $s = 1$.  
      
**Infinity** can represent results that ***overflow***, as when we multiply two very large numbers, or when we divide by zero.  
      
When the ***fraction*** field is nonzero, the resulting value is called a **NaN**, short for “***not a number***.” Such values are returned as the result of an operation where the result cannot be given as a real number or as infinity, as when computing $\sqrt {−1}$ or $\infty − \infty$.  
      
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123181335197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123180858300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
      
### 示例  
![1](https://img-blog.csdnimg.cn/53fc49804df24e5ab8ce97df5db08ee4.png)  
      
<br/>  
      
根据公式 $V$ = $(−1)^s \times M \times 2^E$：  
1. 一般情况计算  
	- `s` 表示正负符号，占最高一位，0 表示正数，1 表示负数。  
	- `M` 为图 2.32 中 `frac` 部分，范围为 [0,1) 或 [1,2)，其位数为 `n`，`frac` 部分值为 `f`，`M = 1+f`（一般情况）。  
	- 对于格式 A，`n` 为 2，如果该部分为 `11`，则 $f = 1 * 2^{-1} + 1 * 2^{-2}$ = $\frac{3}{4}$，$M$ = $1+f$ = $\frac{7}{4}$。  
	- `E` 为图 2.32 中 `exp` 部分，位数为 `k`，偏移量 `Bias` 为  $2^{k−1} − 1$，值为 `e`，$E$ = $e − Bias$（一般情况）。  
	- 对于格式 A，`k` 为 3，偏移量为 $2^{3−1} − 1$ = $3$。如果 `exp` 部分为 `011`，则 `e` 的值为 3，因此 `E` 为 0。  
      
2. 特殊情况 `exp` 全 0  
这时 $E = 1 − Bias$，$M = f$。  
3. 特殊情况 `exp` 全 1  
	- `frac` 全 0  
	结果为无穷，正数为正无穷，负数为负无穷。  
	- `frac` 不是全 0  
	结果为 `NAN`。  
      
*************  
      
4. 格式 A 数字 `1` 表示  
	- 正数则 `s` 为 0。  
	- `M` 为 1，则 `f` 为 0，即 `00`。  
	- $2^{E}$ 为 1，则 `E` 为 0，因为 `Bias` 为 3，`e` 为 3，即 `011`。  
	- 最终值为 `0 011 00`。  
	      
5. 格式 B 数字 $\frac{1}{2}$ 表示  
	- 正数则 `s` 为 0。  
	- $\frac{1}{2}$ 可以表示为 $1 \times 2^{-1}$，`M` 为 1，`E` 为 -1。  
	- 因为 `Bias` 为 1，则 `e` 为 0，对应上面特殊情况，不能用常规公式计算。  
	- 对于特殊情况， `e` 为 0，则 `E` 为 0，修改表示方式，此时 `M` 为 $2^{-1}$，且 $M = f$，因此 `frac` 部分为 `100`。  
	- 最终结果为 `0 00 100`。  
      
6. 格式 B 数字 $\frac{11}{8}$ 表示  
	- 正数则 `s` 为 0。  
	- $\frac{11}{8}$ 可以表示为 $\frac{11}{8} \times 1$，即 `M` 为 $\frac{11}{8}$ ，`E` 为 0。  
	- `f` 为 $\frac{3}{8}$，`e` 为 1，因此 `frac` 部分为 `011`，`exp` 部分为 `01`。  
	- 最终结果为 `0 01 011`。  
      
7. 格式 A 数字 $\frac{11}{8}$ 表示  
	- 正数则 `s` 为 0。  
	- 这个格式 `exp` 部分只有 2 位，因此需要 round to 1.5，即 $\frac{3}{2}$。  
	- $\frac{3}{2} \times 1$，即 `M` 为 $\frac{3}{2}$ ，`E` 为 0，`f` 为 $\frac{1}{2}$，`e` 为 3。  
	- 最终结果为 `0 011 10`。  
      
## 2.4.4 Rounding  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123182458272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZTU2Nw==,size_16,color_FFFFFF,t_70)  
      
**1. Round-to-even**  
      
Round-to-even (also called round-to-nearest) is the default mode. It attempts to find a **closest** match.  
      
The only design decision is to determine the effect of rounding values that are **halfway** between two possible results. Round-to-even mode adopts the convention that it rounds the number either upward or downward such that the *least significant digit* of the result is **even**.  
      
**It will round upward about 50% of the time and round downward about 50% of the time.**  
      
这种情况是当处理大量数据时，保证基本一半是数据向上转换，一半数据向下转换，因此全部数据的平均值误差更小。  
      
**2. Round-toward-zero**  
Round-toward-zero mode rounds positive numbers downward and negative numbers upward.  
      
**3. Round-down**  
Round-down mode rounds both positive and negative numbers  
downward.  
      
**4. Round-up**  
Round-up mode rounds both positive and negative numbers upward.  
