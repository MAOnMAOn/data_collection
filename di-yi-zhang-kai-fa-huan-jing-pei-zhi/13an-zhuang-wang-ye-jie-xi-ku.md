## 1.3 解析库的安装

获取网页代码之后，下一步就是从网页中提取信息，提取信息的方式有多种多样，可以使用正则来提取，但是写起来会相对比较繁琐。在这里还有许多强大的解析库，如 LXML、BeautifulSoup、PyQuery 等等，提供了非常强大的解析方法，如 XPath 解析、CSS 选择器解析等等，利用它们我们可以高效便捷地从从网页中提取出有效信息。

本节我们就来介绍一下这些库的安装过程。

### 1、LXML的安装
在 Linux 平台下安装问题不大，可以先尝试使用 Pip 安装，命令如下

`pip3 install lxml`

如果报错，大多是因为缺少必要的库。执行如下命令安装所需的库即可：

```
sudo yum groupinstall -y development tools
sudo yum install -y epel-release libxslt-devel libxml2-devel openssl-devel
```

主要是 libxslt-devel libxml2-devel 这两个库，lxml依赖于它们。安装好后重新尝试 Pip 安装即可。

### 2、BeautifulSoup、PyQuery 的安装
对于 BeautifulSoup 与 PyQuery ，一般选择pip安装的方式进行安装。

### 3、Tesserocr的安装
#### （1）OCR
OCR，即 Optical Character Recognition，光学字符识别。是指通过扫描字符，然后通过其形状将其翻译成电子文本的过程。那么对于图形验证码来说，它都是一些不规则的字符，但是这些字符确实是由字符稍加扭曲变换得到的内容。

此时，可使用 OCR 技术来将其转化为电子文本，然后爬虫将识别结果提交给服务器，便可以达到自动识别验证码的过程。

Tesserocr 是 Python 的一个 OCR 识别库，但其实是对 Tesseract 做的一层 Python API 封装，所以它的核心是 Tesseract，所以在安装 Tesserocr 之前我们需要先安装 Tesseract，这里我们来了解下它们的安装方式。

#### （2）centos7.3下进行安装
安装命令如下：

`yum install -y tesseract`

不同发行版本运行如上命令即可完成 Tesseract 的安装。安装完成之后便可以调用 tesseract 命令。现在我们查看一下其支持的语言：

`tesseract --list-langs`

运行结果示例：

```
List of available languages (3):
eng
osd
equ
```

结果显示其只支持几种语言，如果我们想要安装多国语言还需要安装语言包，官方叫做 tessdata。其下载链接为：https://github.com/tesseract-ocr/tessdata。
利用 Git 命令将其下载下来并迁移到相关目录即可，迁移命令如下：

```
git clone https://github.com/tesseract-ocr/tessdata.git
sudo mv tessdata/* /usr/share/tesseract/tessdata
```

现在再次调用 tesseract 命令，即可发现其列出的语言就多了非常多，比如 chi_sim 就代表简体中文，这就证明语言包安装成功了。接下来再安装 Tesserocr 即可，直接使用 Pip 安装：

`pip3 install tesserocr pillow`

#### （3）对tesseract验证安装

我们可以使用 Tesseract 和 Tesserocr 来分别进行测试。
下面我们以如下的图片为样例进行测试，如图所示：
![验证码识别述][1]
  [1]: https://pic2.zhimg.com/v2-cc6a4ae2d91aefe336570341748d5f99_b.jpg

我们首先用命令行进行测试，将图片下载保存为 image.png，然后用 Tesseract 命令行测试，命令如下：

`tesseract testpic.jpg result -l eng && cat result.txt`

我们调用 tesseract 命令，第一个参数为图片名称，第二个参数 result 为结果保存的目标文件名称，-l 指定使用的语言包，在此使用 eng 英文，然后再用 cat 命令将结果输出。

我们还可以利用 Python 代码来测试，这里就需要借助于 Tesserocr 库了，测试代码如下：

```
import tesserocr
from PIL import Image
image = Image.open('testpic.jpg')
print(tesserocr.image_to_text(image))
```

在这里我们首先利用 Image 读取了图片文件，然后调用了 tesserocr 的 image_to_text() 方法，再将将其识别结果输出。

另外我们还可以直接调用 file_to_text() 方法，也可以达到同样的效果：

```
import tesserocr
print(tesserocr.file_to_text('testpic.jpg'))
```

运行结果：

`79"”`

成功输出结果，则证明 Tesseract 和 Tesserocr 都已经安装成功。

### 4、相关链接
#### （1）BeautifulSoup
官方文档：https://www.crummy.com/software/BeautifulSoup/bs4/doc
中文文档：https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh
PyPi：https://pypi.python.org/pypi/beautifulsoup4

#### （2）PyQuery 
GitHub：https://github.com/gawel/pyquery
PyPi：https://pypi.python.org/pypi/pyquery
官方文档：http://pyquery.readthedocs.io

#### （3）Tesseroc
Tesserocr GitHub：https://github.com/sirfz/tesserocr
Tesserocr PyPi：https://pypi.python.org/pypi/tesserocr
Tesseract下载地址：http://digi.bib.uni-mannheim.de/tesseract
Tesseract GitHub：https://github.com/tesseract-ocr/tesseract
Tesseract 语言包：https://github.com/tesseract-ocr/tessdata
Tesseract 文档：https://github.com/tesseract-ocr/tesseract/wiki/Documentation