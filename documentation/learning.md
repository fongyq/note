---
layout: page
title: 学而时习之
disable_anchors: true
description: ~
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

## `2021-06-19`
> 时钟周期、CPU周期、指令周期

## `2021-06-18`
> 32位和64位

在计算机概念中，提到32位和64位一般有两种情况：一种是指CPU字长，另一种是指程序指令长度（操作系统、应用软件）。

- CPU字长

  这里的32位和64位，通常称为CPU中“算术逻辑单元（ALU）”的宽度，代表的是CPU一次能并行处理的数据位数，也叫字长。数据总线通常与ALU具有相同的宽度。

  CPU字长决定能控制的总线根数。
    - 32位最大控制的总线为32根，一次能处理最大数据位数是32 bit = 4 byte，32位的CPU只有32位的寄存器。
    - 64位最大控制的总线为64根，一次能处理最大数据位数是64 bit = 8 byte。

  总线的根数决定寻址能力，`寻址范围 = 寻址单元大小 * 2^寻址字长`。CPU对内存的寻址单元大小是 1B，因此32位CPU对内存的最大寻址能力是 $2^{3}2 = 4GB$（给电脑加到8GB内存是没有用的）；64位CPU对内存最大寻址能力是$2^{64} = 256TB$。

  <img src="pictures/computer_arch.png" width="400" />

- 指令长度

  64位和32位软件，实际上是指程序 `指令` 是64位还是32位的。
  一般而言，32位指令在64位机器上可以兼容执行，但是64位指令在32位机器（CPU）上执行就比较困难，因为32位的寄存器存不下64位的指令。

  操作系统其实也是一种程序，我们也会看到操作系统会分成32位操作系统、64位操作系统，其代表意义就是操作系统中程序的指令是多少位。比如，64位操作系统的指令就是64位，不能装在32位机器上。

## `2021-05-04`
> HTTP 和 HTTPS

HTTP（HyperText Transfer Protocol，超文本传输协议）是一种应用层协议，提供发布和接收 HTML 页面的方法，被用于 Web 浏览器和网站服务器之间传递信息。

HTTP 一般存在如下问题：
- 请求信息明文传输，容易被窃听截取。
- 数据的完整性未校验，容易被篡改。
- 没有验证对方身份，存在冒充危险。

HTTPS（Hypertext Transfer Protocol Secure，超文本传输安全协议）利用 SSL（Secure Socket Layer，安全套接字层）/TLS（Transport Layer Security，传输层安全） 来加密数据包，提供对网站服务器的身份认证，保护交换数据的隐私与完整性。HTTPS 并不是一项新的应用层协议，本质就是构建在 SSL/TLS 之上的 HTTP 协议。

比较：
- HTTP 页面响应速度比 HTTPS 快，主要是因为 HTTP 仅使用 TCP 三次握手建立连接，而 HTTPS 除了 TCP 连接之外，还要建立会话秘钥。
- HTTP 和 HTTPS 使用的端口不一样，前者是 80，后者是 443。
- HTTPS 要比 HTTP 更耗费服务器资源。

<img src="pictures/https.png" width="400" />

## `2021-04-27`
> RPC（Remote Procedure Call，远程过程调用）

在分布式计算，远程过程调用是一个计算机通信协议。该协议允许运行于一台计算机的程序调用另一个地址空间的子程序，而程序员就像调用本地程序一样，无需额外地为这个交互作用编程。
RPC 是一种 `服务器-客户端` 模式，经典实现是一个通过 `发送请求-接受回应` 进行信息交互的系统。

[RPC 需要解决三个问题](https://www.zhihu.com/question/25536695/answer/221638079)：

- **Call ID 映射**。在本地调用中，函数体是直接通过函数指针来指定的。但是在远程调用中，函数指针行不通，因为两个进程的地址空间是完全不一样的。
在 RPC 中，所有的函数都必须有自己的 ID，这个ID 在所有进程中都是唯一确定的。客户端在做远程过程调用时，必须附上这个 ID，然后还需要在客户端和服务端分别维护一个 `函数 <--> Call ID` 的映射表（一般是哈希表）。两者的表不一定需要完全相同，但相同的函数对应的 Call ID 必须相同。当客户端需要进行远程调用时，它就查一下这个表，找出相应的 Call ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。

- **序列化和反序列化**。客户端怎么把参数值传给远程的函数呢？在本地调用中，我们只需要把参数压到栈里，然后让函数去栈里读就行。
但是在远程过程调用时，客户端跟服务端是不同的进程，不能通过内存来传递参数。甚至有时候客户端和服务端使用的都不是同一种语言（比如服务端用 C++，客户端用 Java 或者 Python）。
这时候就需要客户端把参数先转成一个字节流，传给服务端后，再把字节流转成能读取的格式。这个过程叫序列化和反序列化。同理，从服务端返回的值也需要序列化反序列化的过程。可以使用 Protobuf 或者 FlatBuffers 之类的。

- **网络传输**。客户端和服务端往往是通过网络连接的。所有的数据都需要通过网络传输，因此就需要有一个网络传输层。网络传输层需要把 Call ID 和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。只要能完成这两者的，都可以作为传输层使用。因此，它所使用的协议其实是不限的。尽管大部 RPC 框架都使用 TCP 协议，但其实 UDP 也可以，而 gRPC 干脆就用了 HTTP2 。


## `2021-04-20`
> python: [`numpy.savez`](https://numpy.org/doc/stable/reference/generated/numpy.savez.html)

```python
numpy.savez(file, *args, **kwds)
```

`numpy.save` 保存一个数组到一个二进制的文件中，保存格式是 `.npy`；`numpy.savez` 同样是保存数组到一个二进制的文件中，但是厉害的是：它可以保存多个数组到同一个文件中，保存格式是 `.npz` ，它其实就是多个`numpy.save` 保存 `.npy` 之后再通过打包（未压缩）的方式把这些文件归到一个文件上。

还可以给保存的数组自定义 key，方便导入和使用。

```python
>>> import numpy as np
>>> x = np.arange(10)
>>> y = np.sin(x)
>>> np.savez('xy.npz', x=x, y=y)
>>> xy = np.load('xy.npz')
>>> _x = xy['x']
>>> _y = xy['y']
```

相比之下，[h5py](https://stackoverflow.com/questions/27710245/is-there-an-analysis-speed-or-memory-usage-advantage-to-using-hdf5-for-large-arr) 有着更清晰的组织逻辑和更高的读写效率。

此外，[`numpy.savetxt`](https://numpy.org/doc/stable/reference/generated/numpy.savetxt.html) 可以直接将数据保存为 txt 文本。

> python: [`numpy.vectorize`](https://numpy.org/doc/stable/reference/generated/numpy.vectorize.html)

```python
class numpy.vectorize(pyfunc, otypes=None, doc=None, excluded=None, cache=False, signature=None)
```

`numpy.vectorize` 定义了一个矢量类，输入是嵌套化的对象序列或者是 numpy 数组，
输出是单个或元组的 numpy 数组；功能跟 map 很类似，将函数 `pyfunc` 作用在序列化的对象上；功能无特殊优化，本质上就是循环调用。

```python
>>> import numpy as np
>>> def foo(x):
...     return x**2
...
>>> vfoo = np.vectorize(foo)
>>> va = [1,2,3,4]
>>> vfoo(va)
array([ 1,  4,  9, 16])
```

## `2021-04-20`
> 定点数和浮点数

数字计算机只能处理离散数据，二进制的位数直接决定了它能表示的离散数据个数，也决定了它所能表示的信息量，对于 $n$ 位二进制数，它可以表示的信息量为 $2^n$。

精度问题：
```python
>>> 6.6 + 1.3
7.8999999999999995
>>> 6.6 + 1.3 == 7.9
False
```

实数有两种表示格式，分别是定点数和浮点数，定点数不一定是整数，浮点数不一定是小数。

#### 定点数（fixed point numbers）

定点数约定机器数中的小数点总是固定在某个特定的位置，整数部分和小数部分位长固定，当需要表示绝对值特大或者特小的数需要很大的存储空间。虽然理论上定点数的小数点的位置可以任意规定，但通常只会用定点数表示 **纯小数** 或 **整数**，计算处理比较方便。

金融/货币计算中，为了保证较高的精度，一般使用定点数据类型，如 NUMERIC、DECIMAL。有时候也直接用整型表示金额，比如将 0.01元实际表示为 1，1.00元表示为 100，整数计算效率高、存储小。

#### 浮点数（floating point numbers）

浮点数使用科学计数法存储数字（格式：符号位 + 指数 + 尾数），小数点的位置根据指数的大小而浮动。

<img src="pictures/float_point.image" width="400" />

- 符号位 S，即0和1，是整个数字正负的符号。

- 阶码 E，也有正负，并且不用真值来表示，通常会用阶码的真值加上一个偏移量，作为实际存储的偏移值。在短浮点数 float 中，这个偏移量为127，假设实际指数为3，则阶码表示为：127 + 3 = 130（二进制：1000 0010）。float 阶码将 1~254 映射到 -126~127 这 254 个有正有负的数上，因为浮点数不仅仅要表示很大的数，还希望表示很小的数，所以指数位也会有负数。0和255被预留出来表示特殊数：0、无穷大、无穷小、NaN。

- 尾数 M（mantissa），或称有效数字（significand），规定将原数尾数转化为 `1.xxxx` 的形式，以 1 为默认最高位，然后储存的时候并不储存最高位1（视其为隐藏的），只存储小数点后面的部分，这样可以使尾数表示的精度更高，即存储位数更多，比实际位数多一位。

IEEE 754 浮点数的一般格式如下： $N = (-1)^S \times 1.M \times 2^E$。

<img src="pictures/float_double.image" width="800" />

float 和 double 的精度是由尾数的位数来决定的，$2^{-23} \approx 1.2 \times 10^{-7} $，因此 float 的精度为6~7位，同理 double 的精度为15~16位。

不考虑符号的话，float 能够表示的最小的数为 $1 \times 2^{-126} \approx 1.175 \times 10^{-38}$ ，最大的数为 $(1.1111...)_2 \times 2^{127} \approx 3.4 \times 10^{38}$。

#### 定点数和浮点数的区别

- 表示范围：浮点数一部分位为指数，相同位长，浮点数格式所能表示的数值范围远远大于定点数格式。

- 精度大小：浮点数格式只有一部分位是有效数值位，相同位长（比如都用32 bit表示），浮点格式的精度比定点格式低。

- 运算复杂度：浮点数主要包括指数和尾数两部分，运算时需要对阶、尾数计算、规格化等步骤，浮点运算比定点运算复杂。

- 溢出：定点运算在数超过可表示数值范围即发生溢出；在浮点运算中，只有规格化后数值超过指数所能表示的范围才溢出。

<details>
  <summary>参考资料</summary>

  <https://juejin.cn/post/6860445359936798734>

  <https://segmentfault.com/a/1190000024485146>
    
  <https://blog.csdn.net/sinat_27143551/article/details/103033589>

  <https://justjavac.com/codepuzzle/2012/11/02/codepuzzle-float-from-surprised-to-ponder.html>

  <https://dev.mysql.com/doc/refman/5.6/en/fixed-point-types.html>

</details>

## `2021-04-03`
> c++: 表达式中的类型转换

```cpp
int a = 0x7fffffff; // INT_MAX
int b = 2;
double c = a * b; // signed int overflow
```

今天犯了一个低级错误，上例在计算 `a * b` 时会先将结果类型转成 `signed int`，从而产生溢出，正确做法是先对表达式中的操作数预先做强制类型转换：
```cpp
double c = double(a) * b; // or: (double)a * b
```

## `2021-02-23`
> python3: f-string

f-string，即格式化字符串常量（formatted string literals），是Python3.6新引入的一种字符串格式化方法。形式上是以`f`或`F`修饰的字符串（`f'xxx'`或`F'xxx'`），以大括号`{}`标明被替换的字段（大括号内外所用的引号不能冲突，可以分别使用单引号和双引号）。f-string在本质上并不是字符串常量，而是一个在运行时运算求值的表达式。

f-string可以解析str、int、字典、元组、列表、集合等类型，还可以解析函数。

```python
>>> var = 'hello'
>>> f'{var} world' 
'hello world'
>>> num = 24.578 
>>> f'{num:.2f}' 
'24.58'
>>> lt = [1,2,3,4]
>>> f'{sum(lt)}' 
'10'
>>> info = {'name': 'tom', 'age': '20'} 
>>> f'my name is {info["name"]}'
'my name is tom'
```

## `2021-02-22`
> python: 进制转换

`bin`、`oct`、`hex`分别将整数转换成二进制、八进制、十六进制的字符串（`str`）。

```python
>>> bin(0x0f) 
'0b1111'
>>> bin(100)
'0b1100100'
>>> oct(100)
'0o144'
>>> hex(100)
'0x64'
```

`int`可以将其他进制的整数/字符串转换成十进制整数。

```python
>>> 0xf
15
>>> int('0xf')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: invalid literal for int() with base 10: '0xf'
>>> int('0xf', base=16) 
15
```


## `2021-01-21`
> python2: 编解码错误（UnicodeEncodeError，UnicodeDecodeError）

python2的字符串有`str`和`unicode`两种类型：

```python
s = '关关雎鸠' # type(s): str
u = u'关关雎鸠' # type(u): unicode
```

类型转换：

```python
print s.decode('utf-8')
print u.encode('utf-8')
```

python2认为unicode才是字符的唯一内码，而字符集如gbk、utf8、ascii都是字符的二进制（字节）编码形式。encode：unicode -> gbk等，decode：gbk等 -> unicode。unicode需要**编码**成相应的字符串才能存储（文件）、输出（打印），字符串需要**解码**成unicode对象才能完成拼接（`s+u`）、格式化（`%s`）、求长（`len`）等操作；在进行同时包含`str`和`unicode`的运算时，python2将`str`解码成`unicode`再执行运算。

python2本身不知道`str`类型的编码方式，只能使用`sys.getdefaultencoding()`方式来解码，一般默认为`ascii`，因此对中文不支持。修改方式（全局）：

```python
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
```

print默认将字符打印到标准输出流`sys.stdout`，python2会按照`sys.stdout.encoding`给`unicode`编码之后输出（如果`sys.stdout.encoding='ascii'`，则输出中文会有问题），至于`str`对象，则直接输出（由操作系统解决）。如果python2代码是通过管道/子进程方式运行，`sys.stdout.encoding`会失效为`None`。

使用`codecs`可以改变`sys.stdout.encoding`：

```python
import codecs
UTF8Writer = codecs.getwriter('utf-8')
sys.stdout = UTF8Writer(sys.stdout)
```

但是`codecs`与默认的`sys.stdout`行为相反，它会把`str`对象使用`sys.getdefaultencoding()`方式来解码成`unicode`再输出。

建议：永远使用`u`的形式定义中文字符串。也可通过导入`unicode_literals`包使得默认编码为`unicode`。
```python
>>> from __future__ import unicode_literals
>>> s = '中国'
>>> type(s)
<type 'unicode'>
>>> u = u'中国'
>>> type(u)
<type 'unicode'>
>>> s + u
u'\u4e2d\u56fd\u4e2d\u56fd'
```

另外，python的编译器是默认使用`ascii`解释代码文件，如果源程序中包含中文（字符串、注释等）就会报错。在代码文件的第一行加上`# -*- coding: utf-8 -*-`可指定编码类型以支持中文。

## `2021-01-20`
> 计算机字符集

- ASCII
  - 8个比特（1字节），128个标准 ASCII 码 + 128个扩充码
  - 为数字、标点、字母、一些符号编码
- GB2312
  - 对ASCII的中文扩展
  - 两个ASCII码大于127的字符连在一起表示一个汉字
  - 为ASCII中的字符都重新用两个字节编码，这就是“全角”字符
  - 原本在127以下的叫“半角”字符
- GBK
  - 对GB2312的扩展
  - 只要求第一个字符大于127，第二个不要求
- Unicode
  - 国际标准组织（ISO）整合了各个国家地区的编码，用16位（两个字节）表示所有字符
    - 半角字符高位补0
  - 一个字符就是两个字节
  - Unicode是一个符号集（character set），没有规定二进制如何存储（如何节省空间，如何让编解码高效）
- UTF
  - 满足计算机网络传输的编码格式
  - utf-8：每次传输8比特数据，还有utf-16、utf-32
  - utf是互联网最广泛使用的Unicode实现方式，是变长编码

例如，汉字“严”的Unicode编码是4e25（十六进制），UTF-8编码是e4b8a5。
```python
>>> u = u'\u4e25'
>>> print u
严
>>> s = u.encode('utf-8')
>>> s
'\xe4\xb8\xa5'
```

## `2020-11-20`
> shell: 批量结束进程

假如要结束`run_demo`指令产生的所有进程，方法如下：

- `ps -ef | grep run_demo | grep -v grep | awk '{print $2}' | xargs kill -9`

  - `ps -ef` 打印当前所有进程的信息

  - `grep run_demo` 筛选目标指令相关进程的信息，以行为单位

  - `grep -v grep` 反向选择，筛选字符串中不包括`grep`的行（`ps -ef`会打印出执行的命令）

  - `awk '{print $2}'` 按默认的空格符分割行，打印分割后的第2个域，即进程号PID

  - `xargs` 将上一个命令的标准输出转换成命令行参数

  - `kill -9 PID` 强制结束进程

- `ps -ef | grep run_demo | grep -v grep | awk '{print "kill -9 " $2}' | bash`

- `killall run_demo`

  - 按进程名结束进程

  - 使用参数`-i`结束前先询问

  - `killall run_*`结束所有以`run_`开头的进程

## `2020-10-26`
> shell: `jobs`

`ctrl+z`将前台正在执行的任务放到后台挂起（`Stopped`）。

`jobs`查看linux中后台任务列表和任务状态：

  - `jobs`命令执行的结果中，`[num]`是后台任务的编号，`+`表示最新的任务，`-`表示第二新的任务，其余没有符号。

  - `jobs -l`可显示任务的进程号pid。

  - 任务状态可以是`Running`、`Stopped`、`Terminated`。

`fg %num`将挂起的任务转为前台执行。

`bg %num`将挂起的任务从状态`Stopped`转为`Running`，仍在后台运行。

`kill %num`或`kill pid`终止后台进程，状态变成`Terminated`。

## `2020-10-09`
> shell: 重定向

shell中常用的文件描述符（file descriptor）有：

  - 0：标准输入（stdin）

  - 1：标准输出（stdout）

  - 2：标准错误（stderr）

标准输出和标准错误默认打印到终端，使用`>`可以进行重定向（覆盖写）：

  - `./run.sh 1> log.txt` 将标准输出重定向到文件log.txt，`1>`可缩写为`>`。

  - `./run.sh 2> err.txt` 将标准错误重定向到文件err.txt。

  - `./run.sh >& log.txt` 将标准输出和标准错误都重定向到文件log.txt。

    - 等价于`./run.sh &> log.txt`

    - 等价于`./run.sh > log.txt 2>&1`

`2>&1`表示将标准错误重定向到标准输出，这里的`&`类似于转义符。需要注意顺序，`./run.sh 2>&1 > log.txt`并不会把标准错误重定向到文件log.txt，因为将标准错误重定向到标准输出时是默认打印到终端的。

`>> log.txt`表示重定向追加到文件log.txt，会保留文件之前的内容；`1>>`和`2>>`同理。

**注** ：python 程序的 print 输出并不能及时打印到屏幕/文件，这是因为有输出缓存机制，加上参数 `-u` 可以不缓存直接输出避免延迟（force the stdout and stderr streams to be unbuffered）。
```python
python -u run.py
```

## `2020-09-07`
> shell: `nohup` `&` `&&` `|` `||`

- `nohup ./run.sh`

  - 提示：nohup: ignoring input and appending output to 'nohup.out'
  - run.sh的输出不会出现在前台
  - 响应SIGINT信号，Ctrl+C可以中止程序
  - 免疫SIGHUP信号，关闭session（终端窗口）不会杀死程序
  - 可以通过kill杀死程序：`killall run.sh`
  - 查看进程：`ps -ef | grep 'run.sh'`

- `./run.sh &`

  - 后台运行，会显示进程号；默认输出到屏幕
  - 通过`jobs`和`fg`切换到前台运行
  - 免疫SIGINT信号，Ctrl+C不可以中止程序
  - 响应SIGHUP信号，关闭session（终端窗口）可以杀死程序

- `nohup ./run.sh &`
  - 后台运行，会显示进程号，前台无输出
  - 同时免疫SIGINT信号和SIGHUP信号
  - 只能通过kill中止程序：`killall run.sh`
  - 重定向输出：`nohup ./run.sh > log.txt 2>&1 &`

- `&&`

  - 串联指令，前一条命令执行成功，才执行后一条

- `|`

  - 管道，上一条命令的输出作为下一条的输入

- `||`

  - 上一条命令执行失败，才执行下一条

## `2020-09-05`
> vi/vim快捷键

- 查找

  - `/term` 向（光标）后查找，`?term` 向前查找
    - `n` 下一个
    - `N` 上一个

- 替换

  - `:%s/old_term/new_term/g`

  - `:%s/term//gn` 匹配个数（`n`表示预览指令效果，不会实际执行）

- 复制粘贴

  - `Y` 或 `yy` 复制整行
  - `P` 粘贴至上行
  - `p` 粘贴至下行

- 删除

  - `D` 从光标删除至行尾
  - `dd` 剪切（删除）整行
  - `3dd` 向下删除3行

- 翻阅

  - `Ctrl+F` 后翻
  - `Ctrl+B` 前翻

- 行号

  - `:set number` 显示行号
  - `:set nonumber` 取消行号

**注** ：`Ctrl+S` 阻断向终端输出，`Ctrl+Q` 恢复向终端输出。

<img src="pictures/vi-vim-cheat-sheet.gif" width="500" />

## `2020-08-19`
> python偏函数：`funtools.partial`

```python
def partial(func, /, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = {**keywords, **fkeywords}
        return func(*args, *fargs, **newkeywords)
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```
`partial`的作用是部分使用某个函数，即冻结住某个函数的某些参数（\*args, **keywords），让它们保证为某个值，并生成一个可调用的新函数对象（newfunc），这样就能够直接调用该函数对象，并且仅使用很少的参数（\*fargs, **fkeywords）。
```python
>>> def foo(a, b, key=True):                    
...     if key: return a*b                        
...     else: return -1*a*b                          
...                                             
>>> pa = functools.partial(foo, b=-1, key=True) 
>>> list(map(pa, [1,2,3]))                      
[-1, -2, -3]                                       
```

被`partial`装饰的函数配合`map`使用（比如tensorflow的`map_fn`），只需要给定单一输入（例中为参数a）。

## `2020-08-11`
> 分卷压缩

- **rar**

  - 压缩，每个分卷最大为50MB。
    ```bash
    rar a -v50m out.rar oxf.mat
    ```

  - 解压
    ```bash
    unrar x out.part1.rar
    ```

- **zip**

  - 压缩，每个分卷最大为50MB。
    ```bash
    zip out.zip oxf.mat
    zip -s 50m out.zip --out partout
    ```

  - 解压
    ```bash
    cat partout.z* > partout_re.zip
    unzip partout_re.zip
    ```

- **tar** 

  - 压缩，每个分卷最大为50MB。
    ```bash
    tar czvf out.tar.gz oxf.mat
    split -d -b 50m out.tar.gz
    ```

  - 解压
    ```bash
    cat x* > out_x.tar.gz
    tar zxvf out_x.tar.gz
    ```

## `2020-08-08`
> python [`pickle`](https://docs.python.org/3/library/pickle.html)

pickle模块实现了用于序列化和反序列化Python对象结构的二进制协议。pickle模块只能在Python中使用，几乎所有的数据类型（列表、字典、集合、类等）都可以用pickle来序列化，以二进制的形式序列化后保存到文件中（.pkl），不能直接打开进行预览。

以下函数基于Python 3.8。

- `pickle.dump(obj, file, protocol=None, *, fix_imports=True, buffer_callback=None)`

  将数据序列化后存入文件（'wb'）。

- `pickle.load(file, *, fix_imports=True, encoding="ASCII", errors="strict", buffers=None)`

  将文件的内容反序列化读出（'rb'），此时要让Python能够找到数据对应的类的定义。

- `pickle.dumps(obj, protocol=None, *, fix_imports=True, buffer_callback=None)`

  将对象序列化为bytes形式，而不是存入文件。

- `pickle.loads(data, *, fix_imports=True, encoding="ASCII", errors="strict", buffers=None)`

  从bytes中读取序列化前的对象。

> python3 `bytes`

bytes以字节序列的形式（二进制形式）存储数据，不关心数据的具体形式（字符串、图像、视频）和内容。

通过encode和decode，bytes和字符串（`str`，unicode）可以相互转换。

在将字符串存入磁盘和从磁盘读取字符串的过程中，Python自动完成了编码和解码；使用bytes类型，则是自己指定编解码方式。

```python
>>> bytes("hello world", encoding="utf8")
b'hello world'
>>> "hello world".encode("utf8")
b'hello world'
>>> "hello world".encode("utf8").decode("utf8")
'hello world'

>>> a = bytes("hello world", encoding="utf8")
>>> a[0]         
104              
>>> a[1]         
101              
>>> len(a)       
11               
>>> a[:5]        
b'hello'             
>>> type(a)      
<class 'bytes'>  
```

> 协同过滤

协同过滤（Collaborative Filtering）是一种在推荐系统中广泛使用的技术。该技术通过分析**用户**或者**事物**之间的相似性（“协同”），来预测用户可能感兴趣的内容并将其推荐给用户。这里的相似性可以是人口特征（性别、年龄、居住地等）的相似性，也可以是历史浏览内容的相似性（比如都关注过和中餐相关的内容）等。比如，用户A和B都是居住在北京的年龄在20-30岁的女性，并且都关注过化妆品和衣物相关的内容，这种情况下，协同过滤可能会认为A和B相似程度很高，于是可能会把A关注但B没有关注的内容推荐给B，反之亦然。

- 基于存量（Memory-based）的协同过滤
  - 基于用户（User-based）的协同过滤

    - 收集用户信息。
    
    - 最近邻搜索(Nearest neighbor search, NNS)，计算用户之间的相似度。

    - 产生推荐结果。例如：透过对A用户的最近邻用户进行统计，选择出现频率高且在A用户的评分项目中不存在的推荐给A。

  - 基于项目（Item-based）的协同过滤
    
    基本假设：能够引起用户兴趣的项目，必定与其之前评分高的项目相似。

    - 收集用户信息。

    - 针对项目的最近邻搜索，计算项目之间的相似度。

    - 产生推荐结果。由于未考虑用户间的差别，所以精度比较差。但是却不需要用户的历史数据，或是进行用户识别。对于项目来讲，它们之间的相似性要稳定很多，因此可以离线完成工作量最大的相似性计算步骤，从而降低了在线计算量，提高推荐效率。

- 基于模型（Model-based）的协同过滤
  
  以存量为基础（Memory-based）的协同过滤的缺点是数据稀疏，难以处理大数据量，这会影响即时结果。以模型为基础的协同过滤是先用历史数据得到一个模型，再用此模型进行预测。

- 优点

  推荐个性化、自动化程度高，能够有效的利用其他相似用户的反馈信息，加快个性化学习的速度。

- 缺点

  新用户问题（New User Problem），系统开始时推荐质量较差；新项目问题（New Item Problem），质量取决于历史数据集；稀疏性问题（Sparsity）；系统扩展性问题（Scalability）。

## `2020-06-01`
> F12

在网页按F12，点击箭头可选择网页内容进行编辑，但是每次编辑都要用箭头重新选择。在console输入：`document.body.contentEditable="true"`就可以对任意内容进行编辑了。

F12可以用来查找网页中图片、视频的下载地址。

## `2020-03-19`
> python `isinstance`

`isinstance(object, classinfo)` 函数来判断一个对象是否是一个已知的类型，`classinfo`可以是元组。`isinstance()` 会认为子类是一种父类类型，考虑继承关系，而 `type()`不考虑继承。

```python
>>> a = 2
>>> isinstance(a, (int, str))
True
>>> class A(object):
...     pass
...
>>> class B(A):
...     pass
...
>>> b = B()
>>> isinstance(b, A)
True
>>> type(b) == A
False
```

> python `zip`

`zip([iterable, ...])` 将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表。

利用`*`号操作符，可以将元组解压为列表。

```python
>>> a = [1,2,3]
>>> b = [5,6,7,8]
>>> zipped = zip(a, b)
>>> zipped
<zip object at 0x7fde6fa9f780>
>>> list(zipped)
[(1, 5), (2, 6), (3, 7)]
>>> list(zipped) ## 只能取一次值
[]
>>> unzipped = zip(*list(zipped))
>>> list(unzipped)
[(1, 2, 3), (5, 6, 7)]
```

## `2020-03-09`
> 进程和线程

`程序（program）`是指令、数据及其组织形式的描述，`进程（process）`是程序的真正运行实例。若干进程有可能与同一个程序相关系，且每个进程皆可以同步（循序）或异步（平行）的方式独立运行。

`线程（thread）`是操作系统能够进行运算调度的最小单位。大部分情况下，它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。

- 不同进程间数据很难共享。
- 同一进程下不同线程间数据很容易共享，多线程技术：每一个线程都代表一个进程内的一个独立执行上下文。
- 进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束才能使用这一块内存。
- 进程间不会相互影响、更加安全稳定，而一个线程挂掉将导致整个进程挂掉。
- 多进程创建销毁、切换复杂速度慢，多线程可以进行快速创建和数据共享。

**并行（parallel）**：在同一时刻，有多条指令在多个处理器上同时执行。无论从微观还是从宏观来看，二者都是一起执行的。

**并发（concurrency）**：在同一时刻，只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替地执行（时间片轮转进程调度算法）。



## `2020-02-29`
> python `vars`

`vars`函数返回对象的属性和属性值对应的字典对象。

```python
class Info(object):
    def __init__(self, age, sex):
        self.age = age
        self.sex = sex

class User(object):
    def __init__(self, name, info):
        self.name = name
        self.info = info

info = Info(25, 'man')
print(vars(info))
print(vars(User('fong', info)))
print(vars(User('fong', vars(info))))
```

```
{'age': 25, 'sex': 'man'}
{'name': 'fong', 'info': <__main__.Info object at 0x000001987824A490>}
{'name': 'fong', 'info': {'age': 25, 'sex': 'man'}}
```

注意最后两行的差异。

## `2020-02-27`
> python [`h5py`](https://docs.h5py.org/en/stable/index.html)

h5py文件是存放两类对象的容器：dataset 和 group。dataset 是数据项；group 是像文件夹一样的容器，类似于字典，有键和值；group 中可以存放 dataset 或者其他 group 。

dataset 和 group 还可以设置属性 attrs。

```python
>>> import h5py
>>> import numpy as np
>>> fw = h5py.File("test.hdf5", 'w')
>>> g = fw.create_group("grp")
>>> d = fw.create_dataset("dst", data=np.random.random([3,5]))
>>> g_sub = g.create_group("grp-sub")
>>> g["grp-dst"] = 5
>>> fw.keys()
<KeysViewHDF5 ['dst', 'grp']>
>>> fw['dst'][()]
array([[0.71277767, 0.66396684, 0.01376168, 0.47915281, 0.5329076 ],
       [0.38888691, 0.78488278, 0.22909742, 0.0611211 , 0.37825911],
       [0.87039116, 0.15093241, 0.8224189 , 0.41678523, 0.5964381 ]])
>>> fw['dst'][0,2:5] ## slice
array([0.01376168, 0.47915281, 0.5329076 ])
>>> g['grp-sub'].name, g['grp-dst'].shape, g['grp-dst'].dtype, g["grp-dst"][()]
('/grp/grp-sub', (), dtype('int64'), 5)
>>> fw.attrs['task'] = 'image retrieval'
>>> list(fw.attrs.items())
[('task', 'image retrieval')]
>>> fw.close()
```

```
fw
├── dst
└── grp
    ├── grp-dst
    └── grp-sub
```

## `2020-02-27`
> python缓存装饰器：`functools.lru_cache`

LRU，即Least Recently Used，最近最少使用，是一种常用的页面置换算法，选择最近最久未使用的页面予以淘汰。

`lru_cache`根据参数缓存每次函数调用结果，对于相同参数的，无需重新函数计算，直接返回之前缓存的返回值。`maxsize`指定了缓存的长度，`typed=True`则不同类型的函数参数将单独缓存，例如，f(3)和f(3.0)将视为不同的调用。

```python
import functools

@functools.lru_cache(maxsize=128, typed=False)
def fibonacci(n:int) -> int:
    if n == 0: return 0
    elif n == 1: return 1
    elif n > 1: 
        return fibonacci(n-2) + fibonacci(n-1)
```

## `2020-01-14`
> `HSL`和`HSV`色彩空间

`HSL`和`HSV`都是一种将`RGB`色彩模型中的点描述在圆柱坐标系中的表示法。这两种表示法试图做到比基于笛卡尔坐标系的几何结构`RGB`更加**直观**。

这个圆柱的中心轴取值为自底部的黑色到顶部的白色而在它们中间的是灰色，绕这个轴的角度对应于“色相”，到这个轴的距离对应于“饱和度”，而沿着这个轴的高度对应于“亮度”或“明度”。

`HSL`即色相、饱和度、亮度（Hue, Saturation, Lightness），又称`HSI`（Hue, Saturation, Intensity）。这三个维度是相互独立的，而`RGB`三个维度不是独立的。

`HSV`即色相、饱和度、明度（Hue, Saturation, Value），又称`HSB`（Hue, Saturation, Brightness）。

- 色相（H）是色彩的基本属性，就是平常所说的颜色名称，如红色、黄色等。
- 饱和度（S）是指色彩的纯度，越高色彩越纯，低则逐渐变灰，取 0 - 100% 的数值。
- 明度（V）、亮度（L），取 0 - 100% 。

<img src="pictures/hsl-hsv.png" width="360" align="center" />

- `HSV` 的 V（明度）控制纯色中混入黑色的量，越往上，值越大，黑色越少，颜色明度越高。
- `HSV` 的 S（饱和度）控制纯色中混入白色的量，越往右，值越大，白色越少，颜色纯度越高。
- `HSL` 的 L（亮度），从下至上，混入的黑色逐渐减少，直到 50% 位置处完全没有黑色，也没有白色，纯度达到最高；继续往上，纯色混入的白色逐渐增加，到达最高点变为纯白色，明度最高。
- 两个表示法在数学上都是圆柱，但`HSV`在概念上可以被认为是颜色的倒圆锥体（黑点在下顶点，白色在上底面圆心），`HSL`在概念上可以被认为是一个双圆锥体或圆球体（白色在上顶点，黑色在下顶点，最大横切面的圆心是半程灰色）。注意：尽管在`HSL`和`HSV`中“色相”指称相同的性质，但是它们的“饱和度”的定义是明显不同的。
- Photoshop颜色选择器采用`HSV`；Windows操作系统颜色选择器采用`HSL`。

## `2020-01-09`
> HTTP协议常见代码

- `1xx` 临时响应，需要请求者继续执行操作
  - `100 Continue` 请求已被服务器接收，浏览器应继续发送请求的其余部分。
  
- `2xx` 请求被成功处理。
  - `200 Ok` 一切正常。

- `3xx` 重定向
  - `301 Permanently Moved` 请求的网页已永久移动到新位置，自动将请求者转到新位置。
  
- `4xx` 请求错误
  - `400 Bad Request` 请求出现语法错误。
  - `403 Forbidden` 资源不可用，服务器理解客户的需求，但是拒绝处理，通常由于服务器上文件或目录的权限设置问题。
  - `404 Not Found` 无法找到指定位置的资源。
  - `410 Gone` 请求的资源已永久删除。
  
- `5xx` 服务器错误
  - `500 Internal Server Error` 服务器遇到了意料不到的情况，不能完成客户的请求。
  - `502 Bad Gateway` 服务器作为网关或代理，从上游服务器收到无效响应。
  - `503 Service Unavilable` 服务器由于维护或者负载过重未能应答。
  - `504 Gateway Timeout` 作为代理或网关服务器使用，表示不能及时地从远程服务器获得应答。

## `2020-01-05`
> python `os.walk`

```python
os.walk(top, topdown=True, onerror=None, followlinks=False)
```

`os.walk`是目录树生成（directory tree generator）函数，通过在目录树中游走输出 `top` 目录下的所有目录及文件；返回一个生成器，每一个元素是一个三元组：(root, dirs, files)，类型是 (string, list, list)。

```
root
├── a.txt
├── b.txt
└── fa
    ├── aa.txt
    ├── ab.txt
    └── faa
        └── aaa.txt
```

以上目录树返回的结果转换成 list 之后是：
```
[('root', ['fa'], ['a.txt', 'b.txt']), 
('root/fa', ['faa'], ['aa.txt', 'ab.txt']), 
('root/fa/faa', [], ['aaa.txt'])]
```

## `2020-01-01`
> shell `^M`

运行 shell 脚本文件报错，出现 `^M` 字样，是因为编辑脚本文件是在 Windows 系统下进行的，导致shell 脚本文件是 dos 格式，即每一行结尾以 `\r\n` 来标识，而 unix 格式的文件行尾则以 `\n` 来标识。

解决方案
- 使用 `dos2unix sh_file` 指令。

- 用 vim 打开该脚本文件；输入 `:set ff?`，若输出 `fileformat＝dos` 可以判定是格式问题；输入 `:set fileformat=unix` 即可。

## `2019-12-27`
> 欧拉数$e$

$$e = \sum_{n=0}^{+\infty} \frac{1}{n!} = \lim_{n \to +\infty} (1+\frac{1}{n})^n \approx 2.7183 $$

- 假设银行一年复利为$r$，一年按$n$次计算复利，每次利率为$\frac{r}{n}$，当$n \to +\infty$，一年的收益率为
  
  $$(1+\frac{r}{n})^n \approx e^r. $$

- 对含$n$个样本的数据进行$n$次有放回均匀采样，当$n \to +\infty$，有

  $$ (1-\frac{1}{n})^n \approx \frac{1}{e} \approx 36.8\% $$

  的样本从未被选中。