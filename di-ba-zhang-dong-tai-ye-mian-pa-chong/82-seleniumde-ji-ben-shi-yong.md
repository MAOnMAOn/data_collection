## 8.2 Selenium的基本使用

### 1. 准备工作
我们以 Chrome 为例来讲解 Selenium 的用法，在本节开始之前请确保已经正确安装好了 Chrome 浏览器并配置好了 ChromeDriver，另外还需要正确安装好 Python 的 Selenium 库。

### 2. 基本使用
准备工作做好之后，首先我们来大体看一下 Selenium 有一些怎样的功能，先用一段实例代码来感受一下：

```
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium import webdriver

driver = webdriver.Chrome()
try:
    for i in range(1, 9):
        url = 'http://www.vegnet.com.cn/Price/List_p{}_%E5%93%88%E5%AF%86%E7%93%9C.html'.format(str(i))

        driver.implicitly_wait(20)
        driver.get(url)
        locator = (By.CSS_SELECTOR,'body > div.head > div > div.head_list > div.head_logo > a')
        WebDriverWait(driver,20,0.5).until(EC.presence_of_all_elements_located(locator))
        jiage = driver.find_elements_by_css_selector(
            'body > div.box > div.main.gap > div > div.pri_k > p > span:nth-child(6)')
        price = [i.text for i in jiage]
        print(price)
    print(browser.get_cookies())
    print(browser.page_source)
finally:
    driver.quit()
```

运行代码之后可以发现会自动弹出一个 Chrome 浏览器，浏览器首先会跳转到对应网页，然后在针对页面元素进行隐式等待，等待搜索结果加载出来之后，控制台分别会输出当前的 爬取结果，当前的 Cookies 还有网页源代码。

### 3. 访问页面
我们可以用 get() 方法来请求一个网页，参数传入链接 URL 即可，比如在这里我们用 get() 方法访问淘宝，然后打印出源代码，代码如下：

```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
print(browser.page_source)
browser.close()
```

运行之后我们便发现弹出了 Chrome 浏览器，自动访问了淘宝，然后控制台输出了淘宝页面的源代码，随后浏览器关闭。
通过这几行简单的代码我们便可以实现浏览器的驱动并获取网页源码，非常便捷。

### 4. 查找节点
Selenium 可以驱动浏览器完成各种操作，比如填充表单、模拟点击等等，比如我们想要完成向某个输入框输入文字的操作，总得需要知道这个输入框在哪里吧？所以 Selenium 提供了一系列查找节点的方法，我们可以用这些方法来获取想要的节点，以便于下一步执行一些动作或者提取信息。
单个节点

比如我们想要从淘宝页面中提取搜索框这个节点，首先观察它的源代码。可以发现它的 ID 是 q，Name 也是 q，还有许多其他属性，那我们获取它的方式就有多种形式了，比如find_element_by_name() 是根据 Name 值获取，ind_element_by_id() 是根据 ID 获取，另外还有根据XPath、CSS Selector 等获取的方式。
我们用代码实现一下：

```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input_first = browser.find_element_by_id('q')
input_second = browser.find_element_by_css_selector('#q')
input_third = browser.find_element_by_xpath('//*[@id="q"]')
print(input_first, input_second, input_third)
browser.close()
```

在这里我们使用了三种方式获取输入框，根据 ID、CSS Selector 和 XPath 获取，它们返回的结果是完全一致的。

在这里列出所有获取单个节点的方法：

>>find_element_by_id
>>find_element_by_name
>>find_element_by_xpath
>>find_element_by_link_text
>>find_element_by_partial_link_text
>>find_element_by_tag_name
>>find_element_by_class_name
>>find_element_by_css_selector

另外 Selenium 还提供了通用的 find_element() 方法，它需要传入两个参数，一个是查找的方式 By，另一个就是值，实际上它就是 find_element_by_id() 这种方法的通用函数版本，比如 find_element_by_id(id) 就等价于 find_element(By.ID, id)，二者得到的结果完全一致。我们用代码实现一下：

```
from selenium import webdriver
from selenium.webdriver.common.by import By

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input_first = browser.find_element(By.ID, 'q')
print(input_first)
browser.close()
```

这样的查找方式实际上功能和上面列举的查找函数完全一致，不过参数更加灵活。
多个节点.

如果我们查找的目标在网页中只有一个，那么完全可以用 find_element() 方法，但如果有多个节点，再用 find_element() 方法查找就只能得到第一个节点了，如果要查找所有满足条件的节点，那就需要用 find_elements() 这样的方法，方法名称中 element 多了一个 s ，注意区分。

当我们用 find_element() 方法，只能获取匹配的第一个节点，结果是 WebElement 类型，如果用 find_elements() 方法，则结果是列表类型，列表的每个节点是 WebElement 类型。函数的列表如下：

>>find_elements_by_id
>>find_elements_by_name
>>find_elements_by_xpath
>>find_elements_by_link_text
>>find_elements_by_partial_link_text
>>find_elements_by_tag_name
>>find_elements_by_class_name
>>find_elements_by_css_selector

### 5. 动作链
在上面的实例中，一些交互动作都是针对某个节点执行的，比如输入框我们就调用它的输入文字和清空文字方法，按钮就调用它的点击方法，其实还有另外的一些操作它是没有特定的执行对象的，比如鼠标拖拽、键盘按键等操作。所以这些动作我们有另一种方式来执行，那就是动作链。
比如我们现在实现一个节点的拖拽操作，将某个节点从一处拖拽到另外一处，可以用代码这样实现：

```
from selenium import webdriver
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to.frame('iframeResult')
source = browser.find_element_by_css_selector('#draggable')
target = browser.find_element_by_css_selector('#droppable')
actions = ActionChains(browser)
actions.drag_and_drop(source, target)
actions.perform()
```

首先我们打开网页中的一个拖拽实例，然后依次选中要被拖拽的节点和拖拽到的目标节点，然后声明了 ActionChains 对象赋值为 actions 变量，然后通过调用 actions 变量的 drag_and_drop() 方法，然后再调用 perform() 方法执行动作，就完成了拖拽操作。

### 6. 执行 JS 代码
对于某些操作，Selenium API 是没有提供的，如下拉进度条等，可以直接模拟运行 JavaScript，使用 execute_script() 方法即可实现，代码如下：

```
import time
from selenium import webdriver

def scrollTo(url='http://www.idx365.com/?happiness'):
    ''' # Scrollbar drag operation '''
    driver.get(url)
    driver.execute_script('window.scrollTo(0,500);')
    time.sleep(2)
    driver.execute_script('window.scrollTo(0,10000);')
    driver.execute_script('alert("To Bottom")')
    time.sleep(1)
    driver.execute_script('window.scrollTo(0,0);')

scrollTo()
```

在这里我们就利用了 execute_script() 方法拖动滚动条，然后弹出 alert 提示框。
所以说有了这个，基本上 API 没有提供的所有的功能都可以用执行 JavaScript 的方式来实现了。

### 7. 获取节点信息
我们在前面说过通过 page_source 属性可以获取网页的源代码，获取源代码之后就可以使用解析库如正则、BeautifulSoup、PyQuery 等来提取信息了。

不过既然 Selenium 已经提供了选择节点的方法，返回的是WebElement 类型，那么它也有相关的方法和属性来直接提取节点信息，如属性、文本等等。这样的话我们就可以不用通过解析源代码来提取信息了，非常方便。
那接下来我们就看一下可以通过怎样的方式来获取节点信息吧。

***获取属性***

我们可以使用 get_attribute() 方法来获取节点的属性，操作的前提就是先选中这个节点。我们用实例来感受一下：

```
from selenium import webdriver
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
logo = browser.find_element_by_id('zh-top-link-logo')
print(logo)
print(logo.get_attribute('class'))
```

运行之后程序便会驱动浏览器打开知乎的页面，然后获取知乎的 LOGO 节点，然后将它的 class 打印出来。通过 get_attribute() 方法，然后传入想要获取的属性名，就可以得到它的值了。

***获取文本值***

每个 WebEelement 节点都有 text 属性，我们可以通过直接调用这个属性就可以得到节点内部的文本信息了，就相当于 BeautifulSoup 的 get_text() 方法、PyQuery 的 text() 方法。我们用一个实例来感受一下：
```
from selenium import webdriver

browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
input = browser.find_element_by_class_name('zu-top-add-question')
print(input.text)
```
这里们依然是先打开知乎页面，然后获取提问按钮这个节点，再将其文本值打印出来。

另外 WebElement 节点还有一些其他的属性，比如 id 属性可以获取节点 id，location 可以获取该节点在页面中的相对位置，tag_name 可以获取标签名称，size 可以获取节点的大小，也就是宽高，这些属性有时候还是很有用的。
我们用实例来感受一下：

```
from selenium import webdriver

browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
input = browser.find_element_by_class_name('zu-top-add-question')
print(input.id)
print(input.location)
print(input.tag_name)
print(input.size)
```

在这里我们首先获得了提问按钮这个节点，然后调用其 id、location、tag_name、size 属性即可获取对应的属性值。

### 8. 切换Frame
我们知道在网页中有这样一种节点叫做 iframe，也就是子Frame，相当于页面的子页面，它的结构和外部网页的结构是完全一致的。Selenium 打开页面后，它默认是在父级Frame 里面操作，而此时如果页面中还有子 Frame，它是不能获取到子 Frame 里面的节点的。所以这时候我们就需要使用 switch_to.frame() 方法来切换 Frame。
首先用一个实例来感受一下：

```
import time
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to.frame('iframeResult')
try:
    logo = browser.find_element_by_class_name('logo')
except NoSuchElementException:
    print('NO LOGO')
browser.switch_to.parent_frame()
logo = browser.find_element_by_class_name('logo')
print(logo)
print(logo.text)
```

还是以上文演示动作链操作的网页为实例，首先我们通过 switch_to.frame() 方法切换到子 Frame 里面，然后我们尝试获取父级 Frame 里的 LOGO 节点，是不能找到的，找不到的话就会抛出 NoSuchElementException 异常，异常被捕捉之后就会输出 NO LOGO，接下来我们重新切换回父Frame，然后再次重新获取节点，发现即可成功获取。

所以，当页面中包含子 Frame 时，如果我们想获取子Frame 中的节点，需要先调用 switch_to.frame() 方法切换到对应的 Frame，然后再进行操作即可。

### 9. 延时等待
在 Selenium 中，get() 方法会在网页框架加载结束之后就结束执行，此时如果获取 page_source 可能并不是浏览器完全加载完成的页面。所以这里我们需要延时等待一定时间确保节点已经加载出来。等待的方式有两种，一种隐式等待，一种显式等待。

***隐式等待***

当使用了隐式等待执行测试的时候，如果 Selenium 没有在DOM 中找到节点，将继续等待，超出设定时间后则抛出找不到节点的异常, 换句话说，当查找节点而节点并没有立即出现的时候，隐式等待将等待一段时间再查找 DOM，默认的时间是 0。我们用一个实例来感受一下：

```
from selenium import webdriver

browser = webdriver.Chrome()
browser.implicitly_wait(10)
browser.get('https://www.zhihu.com/explore')
input = browser.find_element_by_class_name('zu-top-add-question')
print(input)
```

在这里我们用 implicitly_wait() 方法实现了隐式等待。

***显式等待***

隐式等待的效果其实并没有那么好，因为我们只是规定了一个固定时间，而页面的加载时间是受到网络条件影响的。
所以在这里还有一种更合适的等待方法，它指定好要查找的节点，然后指定一个最长等待时间。如果在规定时间内加载出来了这个节点，那就返回查找的节点，如果到了规定时间依然没有加载出该节点，则会抛出超时异常。运行代码，在网速较佳的情况下是可以成功加载出来的：

```
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

browser = webdriver.Chrome()
browser.get('https://www.taobao.com/')
wait = WebDriverWait(browser, 10)
input = wait.until(EC.presence_of_element_located((By.ID, 'q')))
button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '.btn-search')))
print(input, button)
```

在这里我们首先引入了 WebDriverWait 这个对象，指定好最长等待时间，然后调用它的 until() 方法，传入要等待条件 expected_conditions，比如在这里我们传入了 presence_of_element_located 这个条件，就代表节点出现的意思，其参数是节点的定位元组，也就是 ID 为 q 的节点搜索框。所以这样可以做到的效果就是，在 10 秒内如果 ID 为 q 的节点即搜索框成功加载出来了，那就返回该节点，如果超过10 秒还没有加载出来，那就抛出异常。

对于按钮，可以更改一下等待条件，比如改为 element_to_be_clickable，也就是可点击，所以查找按钮时是查找 CSS 选择器为 .btn-search 的按钮，如果 10 秒内它是可点击的也就是成功加载出来了，那就返回这个按钮节点，如果超过 10 秒还不可点击，也就是没有加载出来，那就抛出异常。

### 10. Cookies 操作
使用 Selenium 还可以方便地对 Cookies 进行操作，例如获取、添加、删除 Cookies 等等。我们再用实例来感受一下：
```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
print(browser.get_cookies())
browser.add_cookie({'name': 'name', 'domain': 'www.zhihu.com', 'value': 'germey'})
print(browser.get_cookies())
browser.delete_all_cookies()
print(browser.get_cookies())
```
首先我们访问了知乎，然后加载完成之后，浏览器实际上已经生成了 Cookies 了，我们调用 get_cookies() 方法就可以获取所有的 Cookies，然后我们添加一个 Cookie，传入一个字典，有 name、domain、value 等内容。接下来我们再次获取所有的 Cookies，可以发现结果就多了这一项 Cookie。最后我们调用 delete_all_cookies() 方法，删除所有的 Cookies，再重新获取，结果就为空了。

### 11. 异常处理
在使用 Selenium 过程中，难免会遇到一些异常，例如超时、节点未找到等错误，一旦出现此类错误，程序便不会继续运行了，所以异常处理在程序中是十分重要的。
在这里我们可以使用 try except 语句来捕获各种异常。
当出现 NoSuchElementException，也就是节点未找到的异常，为了防止程序遇到异常而中断，我们需要捕获一下这些异常。

```
from selenium import webdriver
from selenium.common.exceptions import TimeoutException, NoSuchElementException

browser = webdriver.Chrome()
try:
    browser.get('https://www.baidu.com')
except TimeoutException:
    print('Time Out')
try:
    browser.find_element_by_id('hello')
except NoSuchElementException:
    print('No Element')
finally:
    browser.close()
```

如上例所示，这里我们使用 try except 来捕获各类异常，比如我们对 find_element_by_id() 查找节点的方法捕获 NoSuchElementException 异常，这样一旦出现这样的错误，就进行异常处理，程序也不会中断了。