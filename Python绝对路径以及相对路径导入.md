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

#### `import`语句风格

作为python的[官方](https://realpython.com/python-code-quality/)样式说明，[PEP8](http://pep8.org/#imports)提出了几点你在写import语句时应该注意的。大致概括如下：

1. 导入语句应该位于文件的最上部，在模块说明或者文档之下。

2. 导入应该根据导入的内容进行区分。通常有三种分类：
    * 标准库导入(python内置库)
    * 安装的第三方库(已经安装的并且不属于当前应用的模块)
    * 本地应用导入(属于当前应用的模块)

3. 不同分类的模块应该用空行分开。

当然，你也可以按照导入的包或者模块的字母排序。这样你可以快速定位到特定的资源,特别是当你有很多资源被导入的时候。

下面是一个示例，展示了如何优雅的导入。

```python
'''
Illustration of good import statement styling.

Note that the imports come after the docstring.
'''

# Standard library imports
import datetime
import os

# Third party imports
from flask import Flask
from flask_restful import Api
from flask_sqlalchemy import SQLAlchemy

# Local application imports
from local_module import local_class
from local_package import local_function

```

上面的这些import语句被分成了三个不同的分组，中间用一行空行分隔。在每个分组内又根据字母顺序排列。

### 绝对路径导入

你已经基本掌握了如何写import语句并且优雅的表达出来。现在让我们更深入的来学习一下绝对路径导入。

绝对路径导入需要表达清楚资源被导入时相对于项目根路径的完整路径。

#### 语法以及实际案例

我们先假设你有如下的一个目录结构：

```
└── project
    ├── package1
    │   ├── module1.py
    │   └── module2.py
    └── package2
        ├── __init__.py
        ├── module3.py
        ├── module4.py
        └── subpackage1
            └── module5.py
```

这里有一个目录，`project`，它又包含了两个子目录，`package1`以及`package2`。

目录`package1`里面有两个文件，`module1.py`和`module2.py`。

目录`package2`内有三个文件：两个模块，`module3.py`和`module4.py`，已经一个初始化文件，`__init__.py`。其中还有一个目录，`subpackage1`，这个子目录中又包含一个文件，`module5.py`。

让我们假设如下：

1. `package1/module2.py`包含一个函数，`function1`。
2. `package2/__init__.py`包含一个类，`class1`。
3. `package2/subpackage1/module5.py`包含一个函数，`function2`。

下面就是绝对路径导入时的实际案例。

```python
from package1 import module1
from package1.module2 import function1
from package2 import class1
from package2.subpackage1.module5 import function2
```

注意到我们必须在导入时指定它的详细路径，相对于包的顶层目录。这就相当于这个文件的路径，只不过我们将斜杠(`/`)换成了点(`.`)。

#### 绝对路径导入的利弊

绝对路径导入由于其直观往往是大家的首选。只要看一下导入语句你就能知道资源是从什么位置导入的。再者，就算当前import语句的位置发生了变化，此绝对路径导入的资源依然有效。实际上，PEP8明确表明了推荐使用绝对路径导入。

然后，有的时候绝对路径导入却会使代码变的复杂，这取决与程序目录结构的复杂程度。假如现在有一个导入语句如下：

```python
from package1.subpackage2.subpackage3.subpackage4.module5 import function6
```

这看上去很荒谬，是吧？幸运的是，相对路径导入在这个例子中可以作为很好的替代选项!

### 相对路径导入

相对路径导入指的是导入资源相对于当前位置(当前import语句所在的位置)的相对路径。这里有两种相对导入的类型：隐式的和显式的。隐式相对导入已经在python3中被弃用，所以我在这里不会谈论它。

#### 语法以及实际案例

相对路径导入的语法取决于当前的位置以及将要被导入的模块、包或对象的位置。以下是几个相对路径导入的例子：

```python
from .some_module import some_class
from ..some_package import some_function
from . import some_class
```

可以看到每个导入语句中都至少有一个点号。相对路径导入使用点号来标识相对位置。

一个点号代表了相关的模块或者包和当前的主位置在一个目录下。两个点号意味着在当前位置的父级目录下，也就是上一级目录。三个点号则意味着在上上层目录下，以此类推。如果你使用过类Unix系统的话，你会感觉很熟悉。

我们先假设你有一个和先前一样的目录结构：

```
└── project
    ├── package1
    │   ├── module1.py
    │   └── module2.py
    └── package2
        ├── __init__.py
        ├── module3.py
        ├── module4.py
        └── subpackage1
            └── module5.py
```

回顾一下文件的内容：

1. `package1/module2.py`包含一个函数，`function1`。
2. `package2/__init__.py`包含一个类，`class1`。
3. `package2/subpackage1/module5.py`包含一个函数，`function2`。

你可以在`package1/module1.py`中这样导入`function1`:

```python
# package1/module1.py

form .module2 import function1
```

你在这里使用了一个点号，因为`module2`和当前模块(`module1.py`)在一个目录下面。

你在`package2/module3.py`中导入`class1`,`function2`时如下：
```python
# package2/module3.py

from . import class1
from .subpackage1.module5 import function2
```

在第一行中,单点号说明你从当前目录中导入了class1. 回顾一下：当我们导入一个文件夹的时候，实际上我们是导入了该文件夹下面的`__init__.py`文件。

在第二行中，你使用的单点号因为subpackage1和当前文件(module3.py)在相同的目录。

#### 相对路径导入的利弊

相对路径导入一个显而易见的优点就是十分的简洁明了。根据当前所在的位置，我们可以将前面用到的冗长的表达转换为如下简单的表述：
```python
from ..subpackage4.module5 import function6
```

不过，相对路径导入却也容易使导入语句变的混乱，特别是在一些目录结构会发生变化的共享项目。相对了路径导入还有一个缺点就是可读性较差，让人很难清楚地了解到资源所在的位置。

### 结论

以上我们已经快速的了解了绝对以及相对路径导入！你已经掌握了导入语句的实际使用方法，还知道了一些他们之间的区别。

有了这个新技能后，你就可以正确的导入标准库，第三方库，以及自己的本地包或模块。当然，你通常还是需要首先考虑使用绝对路径导入，除非你的资源路径特别长使得导入语句特别长的时候。

本篇import教程到此结束，谢谢

[Reference：absolute-vs-relative-python-imports](https://realpython.com/absolute-vs-relative-python-imports/)

