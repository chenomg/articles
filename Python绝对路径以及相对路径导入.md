​	如果你的一个项目中有超过一个文件，那你就很有可能需要在你的代码开始时使用import语句。

​	及时对于参与过几个项目的Python工程师， import的用法也可能是模糊的。你阅读这篇文章应该就是为了能够加深对python的import模块的理解吧，特别是绝对路径以及相对路径导入。

​	在这篇教程中，你将会学习这两者之间的区别，以及他们的利弊。让我们开始吧。

### 快速概览Import

​	首先，你想知道import是如何工作的你就要先理解什么是python的模块和包。模块就是后缀为`.py`的文件，包是一个包含模块的文件夹（或者是包含`__init__.py`文件的文件夹）。

​	当你一个模块中的代码需要访问另一个模块或包的代码时，你就需要`import it`.

#### `import`是如何工作的

​	但是到底`import`到底是怎么工作的呢？你一般会这样导入

```python
import abc
```

​	这时python首先就会在`sys.modules`中查找。这个模块缓存了当前已被导入过的模块名。

​	如果没有在模块缓存中找到，python就会继续在内建的模块列表中查找。这些是python预装的模块，可以在python标准库中找到。如果名字还是不能在标准库中找到，python然后会在`sys.path`中定义的文件夹列表中查找。这个列表通常会包括当前的目录，会首先在这个目录中查找。

​	当python找到这个模块后，就会把它绑定到这个本地作用域中的变量名。这就意味着`abc`现在已经被定义而且可以在当前文件中被正常的使用而不抛出`NameError`。

​	如果名字最终没找到，你就会得到一个`ModuleNotFoundError`。点[这里](https://docs.python.org/3/reference/import.html)（python文档）你可以找到更多关于import的资料。

> **注意：安全担忧**
> 注意到Python的导入系统有一些重大的安全隐患。这主要原因就在于本身的灵活性。比如模块的缓存是可写入的，并且存在通过import模块去复写Python核心函数库。导入第三方库也有可能使你的应用暴露安全威胁。
> 这里有一些资源可以帮助你学到更多关于安全风险以及如何去避免的知识：
> [10 common security gotchas in Python and how to avoid them](https://hackernoon.com/10-common-security-gotchas-in-python-and-how-to-avoid-them-e19fbe265e03)
> * 作者：Anthony Shaw，（关于python的import模块五讲）*
> [Episode #168: 10 Python security holes and how to plug them](https://talkpython.fm/episodes/show/168/10-python-security-holes-and-how-to-plug-them)
> * 来自：TalkPython podcast（小组成员在27:15左右开始谈论import）*

#### import语法

现在你应该已经知道了`import`语句是如何工作的，让我们来了解一下具体的语法。你可以导入包(package)以及模块(module)。(注意当你导入一个包的时候自己上是导入了包里面的`__init__.py`文件,作为模块).你同样可以从包或模块中导入特定的对象。

这里通常有两种导入的语法。当你使用第一种时，你直接导入资源，比如：

```python
import abc
```

此时`abc`可以是包或者模块。

第二种语法，你可以从一个包或者模块中导入另外的资源。比如：

```python
from abc import xyz
```

此时`xyz`可以是一个模块，子包，或者对象，比如一个class或者function。

你还可以重命名你导入的资源，就像：

```python
import abc as other_name
```

这会在当前的脚本内将导入的资源`abc`重命名为`other_name`。这时你必须在代码中将对应的名字改为`other_name`,否则原先的名字将不能被识别。

[Reference：absolute-vs-relative-python-imports](https://realpython.com/absolute-vs-relative-python-imports/)

