## 6.1 网页字符编码的处理

不论你是有着多年经验的 Python 老司机还是刚入门 Python 不久的新贵，你一定遇到过UnicodeEncodeError、UnicodeDecodeError 错误，有时我们拿着拿着 encode、decode 函数去转换可以解决问题，有时候怎么试都没辙，只有借用 Google 。

为了减少编码问题给我们实际工作带来的困扰，这里我们选择了 Linux 运行环境，并选用了对于中文字符更为友好的 Python3 进行代码的编写。不过，我们还是有必要了解一些基本概念，在下次遇到类似问题时，尽量避免重蹈覆辙。

### 1. 基本概念
完全理解 Python 字符编码前，我们有必要把一些基础概念弄清楚，虽然有些概念我们每天都在接触甚至在使用它，但并不一定真正理解它。比如：字节、字符、字符集、字符码、字符编码。

***字节***

字节（Byte）是计算机中存储数据的单元，一个字节等于一个8位的比特，计算机中的所有数据，不论是磁盘文件上的还是网络上传输的数据（文字、图片、视频、音频文件）都是由字节组成的。

***字符***

你正在阅读的这篇文章就是由很多个字符（Character）构成的，字符一个信息单位，它是各种文字和符号的统称，比如一个英文字母是一个字符，一个汉字是一个字符，一个标点符号也是一个字符。

***字符集***

字符集（Character Set）就是某个范围内字符的集合，不同的字符集规定了字符的个数，比如 ASCII 字符集总共有128个字符，包含了英文字母、阿拉伯数字、标点符号和控制符。而 GB2312 字符集定义了7445个字符，包含了绝大部分汉字字符。

***字符码***

字符码（Code Point）指的是字符集中每个字符的数字编号，例如 ASCII 字符集用 0-127 这连续的128个数字分别表示128个字符，"A" 的编号就是65。

***字符编码***

字符编码（Character Encoding）是将字符集中的字符码映射为字节流的一种具体实现方案，常见的字符编码有 ASCII 编码、UTF-8 编码、GBK 编码等。某种意义上来说，字符集与字符编码有种对应关系，例如 ASCII 字符集对应 有 ASCII 编码。ASCII 字符编码规定使用单字节中低位的7个比特去编码所有的字符。例如"A" 的编号是65，用单字节表示就是0×41，因此写入存储设备的时候就是b'01000001'。

***编码、解码***

编码的过程是将字符转换成字节流，解码的过程是将字节流解析为字符。


### 2. 字符编码的演进
理解了这些基本的术语概念后，我们就可以开始讨论计算机的字符编码的演进过程了。

***从 ASCII 码说起***

计算机发明于美国，在英语世界里，常用字符的个数非常有限，26个字母（大小写）、10个数字、标点符号、控制符，这在计算机中用一个字节的存储空间来表示一个字符绰绰有余，因为一个字节相当于8个比特位，8个比特位可以表示256个符号。于是美国国家标准协会ANSI制定了一套字符编码的标准叫 ASCII(American Standard Code for Information Interchange)，每个字符都对应唯一的一个数字，比如字符 "A" 对应数字是65，"B" 对应 66，以此类推。最早 ASCII 只定义了128个字符编码，包括96个文字和32个控制符号，一共128个字符只需要一个字节的7位就能表示所有的字符，因此 ASCII 只使用了一个字节的后7位，剩下最高位1比特被用作一些通讯系统的奇偶校验。

***扩展的 ASCII：EASCII(ISO/8859-1)***

然而计算机普及到西欧时，发现还有很多字符是 ASCII 字符集中没有的，显然 ASCII 已没法满足需求了，好在 ASCII 字符只用了字节的前7位 0×00~0x7F 共128个字符，于是在 ASCII 的基础上把原来的7位扩充到8位，把0×80-0xFF这后面的128个数字利用起来，叫 EASCII ，它完全兼容ASCII，扩展出来的符号包括表格符号、计算符号、希腊字母和特殊的拉丁符号。然而 EASCII 时代是一个混乱的时代，各个厂家都有自己的想法，大家没有统一标准，他们各自把最高位按照自己的标准实现了自己的一套字符编码标准。

不过众多的 ASCII 扩充字符集之间互不兼容，导致人们无法正常交流，例如200在CP437字符集表示的字符是 È ，在 ISO/8859-1 字符集里显示的就是 ╚，于是国际标准化组织制定的一系列8位字符集标准 ISO/8859-1(Latin-1)，它继承了 CP437 字符编码的128-159之间的字符，所以它是从160开始定义的，ISO-8859-1在 CP437 的基础上重新定义了 160~255之间的字符。

***多字节字符编码 GBK***

ASCII 是单字节编码，对于拉丁语系国家来说通过扩展最高位，单字节表示所有的字符已经绰绰有余，但是对于亚洲国家来说一个字节就显得捉襟见肘了。于是中国人自己弄了一套叫 GB2312 的双字节字符编码，又称GB0，1981 由中国国家标准总局发布。GB2312 编码共收录了6763个汉字，同时他还兼容 ASCII，GB 2312的出现，基本满足了汉字的计算机处理需要，它所收录的汉字已经覆盖中国大陆99.75%的使用频率。

不过 GB2312 还是不能100%满足中国汉字的需求，对一些罕见的字和繁体字 GB2312 没法处理，后来就在GB2312的基础上创建了一种叫 GBK 的编码，GBK 不仅收录了27484个汉字，同时还收录了藏文、蒙文、维吾尔文等主要的少数民族文字。同样 GBK 也是兼容 ASCII 编码的，对于英文字符用1个字节来表示，汉字用两个字节来标识。

***Unicode 的问世***

全世界各地的文字的表示，这已经大大超出了ASCII 码甚至GBK 的范围了，虽然各国可以制定自己的编码方案，但是数据在不同国家传输就会出现各种各样的乱码问题。如果只用一种字符编码就能表示地球甚至火星上任何一个字符时，问题就迎刃而解了。

于是统一联盟国际组织提出了Unicode 编码，Unicode 的学名是”Universal Multiple-Octet Coded Character Set”，简称为UCS。它为世界上每一种语言的每一个字符定义了一个唯一的字符码，Unicode 标准使用十六进制数字表示，数字前面加上前缀 U+，比如字母『A』的Unicode编码是 U+0041，汉字『中』的Unicode 编码是U+4E2D

Unicode有两种格式：UCS-2和UCS-4。UCS-2就是用两个字节编码，一共16个比特位，这样理论上最多可以表示65536个字符，不过要表示全世界所有的字符显示65536个数字还远远不过，因为光汉字就有近10万个，因此Unicode4.0规范定义了一组附加的字符编码，UCS-4就是用4个字节（实际上只用了31位，最高位必须为0）。理论上完全可以涵盖一切语言所用的符号。

***Unicode 的局限***

但是 Unicode 有一定的局限性，一个 Unicode 字符在网络上传输或者最终存储起来的时候，并不见得每个字符都需要两个字节，比如字符“A“，用一个字节即可，还要用两个字节，显然太浪费空间了。

第二问题是，一个 Unicode 字符保存到计算机里面时就是一串01数字，那么计算机怎么知道一个2字节的Unicode字符是表示一个2字节的字符呢，例如“汉”字的 Unicode 编码是 U+6C49，我可以用4个ascii数字来传输、保存这个字符；也可以用utf-8编码的3个连续的字节E6 B1 89来表示它。关键在于通信双方都要认可。因此Unicode编码有不同的实现方式，比如：UTF-8、UTF-16等等。Unicode就像英语一样，做为国与国之间交流世界通用的标准，每个国家有自己的语言，他们把标准的英文文档翻译成自己国家的文字，这是实现方式，就像utf-8。

***UTF-8***

UTF-8（Unicode Transformation Format）作为 Unicode 的一种实现方式，广泛应用于互联网，它是一种变长的字符编码，可以根据具体情况用1-4个字节来表示一个字符。比如英文字符这些原本就可以用 ASCII 码表示的字符用UTF-8表示时就只需要一个字节的空间，和 ASCII 是一样的。对于多字节（n个字节）的字符，第一个字节的前n为都设为1，第n+1位设为0，后面字节的前两位都设为10。剩下的二进制位全部用该字符的unicode码填充。

以『好』为例，『好』对应的 Unicode 是597D，对应的区间是 0000 0800—0000 FFFF，因此它用 UTF-8 表示时需要用3个字节来存储，597D用二进制表示是： 0101100101111101，填充到 1110xxxx 10xxxxxx 10xxxxxx 得到 11100101 10100101 10111101，转换成16进制是 e5a5bd，因此『好』的 Unicode 码 U+597D 对应的 UTF-8 编码是 “E5A5BD”。可以用 Python3 代码来验证：

```
>>> a = u"好"
>>> a
'好'
>>> b = a.encode('utf-8')
>>> len(b)
3
>>> b
'\xe5\xa5\xbd'
```

### 3. Python 编码问题
说完理论，再来说说 Python 中的编码问题。Python 的诞生时间比 Unicode 要早很多，Python2 的默认编码是ASCII，Python3 的默认编码是 UTF-8

```
>>> # 这是python2代码
>>> import sys
>>> sys.getdefaultencoding()
'ascii'
```

所以在 Python2 中，源代码文件必须显示地指定编码类型，否则但凡代码中出现有中文就会报语法错误

`# coding=utf-8`
或者是：
`# -*- coding: utf-8 -*-`

***Python2 字符类型***

在 python2 中和字符串相关的数据类型有 str 和 unicode 两种类型，它们继承自 basestring，而 str 类型的字符串的编码格式可以是 ascii、utf-8、gbk等任何一种类型。

对于汉字『好』，用 str 表示时，它对应的 utf-8 编码 是’\xe5\xa5\xbd’，对应的 gbk 编码是 ‘\xba\xc3’，而用 unicode 表示时，他对应的符号就是u’\u597d’，与u”好” 是等同的。

***str 与 unicode 的转换***

在 Python 中 str 和 unicode 之间是如何转换的呢？这两种类型的字符串之间的转换就是靠decode 和 encode 这两个函数。encode 负责将unicode 编码成指定的字符编码，用于存储到磁盘或传输到网络中。而 decode 方法是根据指定的编码方式解码后在应用程序中使用。
```
>>> # 以下是python2代码
>>> #从unicode转换到str用 encode
>>> b  = u'好'
>>> c = b.encode('utf-8')
>>> type(c)
<type 'str'>
>>> c
'\xe5\xa5\xbd'

#从str类型转换到unicode用decode
>>> d = c.decode('utf-8')
>>> type(d)
<type 'unicode'>
>>> d
u'\u597d'
```

***UnicodeXXXError 错误的原因***

在字符编码转换操作时，遇到最多的问题就是 UnicodeEncodeError 和 UnicodeDecodeError 错误了，这些错误的根本原因在于 Python2 默认是使用 ascii 编码进行 decode 或者 encode 操作的，例如：

```
>>> s = '你好'
>>> s.decode()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```

当把 s 转换成 unicode 类型的字符串时，decode 方法默认使用 ascii 编码进行解码，而 ascii 字符集中根本就没有中文字符『你好』，所以就出现了 UnicodeDecodeError，正确的方式是显示地指定 UTF-8 字符编码。

```
>>> s.decode('utf-8')
u'\u4f60\u597d'
```

同样，对于 encode 操作，把 unicode字符串转换成 str类型的字符串时，默认也是使用 ascii 编码进行编码转换的，而 ascii 字符集找不到中文字符『你好』，于是就出现了UnicodeEncodeError 错误。
```
>>> a = u'你好'
>>> a.encode()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
```

当然， str 类型与 unicode 类型的字符串混合使用时，str 类型的字符串会隐式地将 str 转换成 unicode字符串，如果 str字符串是中文字符，那么就会出现UnicodeDecodeError 错误，因为 python2 默认会使用 ascii 编码来进行 decode 操作。我们这里感受一下：
```
>>> s = '你好'  # str类型
>>> y = u'python'  # unicode类型
>>> s + y    # 隐式转换，即 s.decode('ascii') + u
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```

所以说，正确地方式是显示地指定 UTF-8 字符编码进行解码：

```
>>> s.decode('utf-8') +y
u'\u4f60\u597dpython'
```

***乱码***

所有出现乱码的原因都可以归结为字符经过不同编码解码在编码的过程中使用的编码格式不一致，比如在 Python2 中，有：

```
# encoding: utf-8

>>> a='好'
>>> a
'\xe5\xa5\xbd'
>>> b=a.decode("utf-8")
>>> b
u'\u597d'
>>> c=b.encode("gbk")
>>> c
'\xba\xc3'
>>> print c
��
```

utf-8编码的字符‘好’占用3个字节，解码成Unicode后，如果再用gbk来解码后，只有2个字节的长度了，最后出现了乱码的问题，因此防止乱码的最好方式就是始终坚持使用同一种编码格式对字符进行编码和解码操作。

### 4. Python3 出现 UnicodeEncodeError
有时候，我们在 Python3 下，打印字符串，也会出现 UnicodeEncodeError，这很有可能是我们的字符串并非为 utf8 编码，比如类似于'ANSI_X3.4-1968'这样的编码。所以为了能够正常输出 utf8，需要通过以下代码把原来的编码变成 utf8：

```
import sys
import io
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
```

然后再次检验stdout是否为utf-8：

```
>>> sys.stdout
<_io.TextIOWrapper name='' encoding='utf-8'>
```

以上即可解决问题。