![python](https://upload.wikimedia.org/wikipedia/commons/f/f8/Python_logo_and_wordmark.svg)

对于初学者来说，closure 可能是个比较难懂的概念。来看看 [**wiki**](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29) 上对于 closure 的说法:

>   a **closure** (also **lexical closure** or **function closure**) is a technique for implementing lexically scoped name binding in a language with first-class functions

嗯…应该没有看懂吧？

其实文中所提的 *binding* 其实白话的来说，就是变量在一个函数中是如何被决定它的值会是多少 (lookup)。

而 lexical binding ，根据 [**EmacsWiki**](https://www.emacswiki.org/emacs/DynamicBindingVsLexicalBinding#toc2) 上的说明，就是每个 scope (function, class, …etc) 会有各自的一张 variable lookup table 。

#### LEGB

根据 Python.org 的 tutorial 中的[说明](https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces)，当一个变量被使用时，会遵循 LEGB 的规则，也就是 **L**ocal、**E**nclosing、**G**lobal 与 **B**uiltins

让我以下面这段代码为例吧:

```python
glob = 3
def func(x):
    y = x + glob
    def inner():
        return y + 1
    return inner, abs
```

Local，顾名思义，就是在 local variables 里查找。以上面的例子来说， `y` 就是 `func`的 local variable ，因为 `y` 是在 `func` 里才被定义的。

Enclosing，也就是 enclosing scope (别急，等会解释)

Global，也就是 global variable ，在上面的例子里， `func` 里用到的 `glob` 就会是定义在 `func` 外面的 `glob` 。

Builtins，也就是从 `builtins` 模组里去找，上面的例子里就是最后用到的 `abs` 。

当以上都找不到这个你要的变数时，就会引发 `NameError` 。

#### Function Scope

上面的 LGEB 有提到 enclosing scope。在 Python 里创造一个 scope 的最简单的方式是 function 。顺道一提，在 Python 里 `for` 是不会创造一个 scope 的喔！譬如你可以试试下面这串程式码:

```python
for i in range(10):
    pass
print(i)
# 应该会看到 9
```

p.s 有机会再来聊聊 Python 的 lazy binding 好了，有机会的话….？

言归正传。也就是说，在 Python 中当你定义一个 function 时，你就创造了一个 scope ，这个 scope 会影响到你这个 function 里所有 local 与 non-local 变数会如何被参照。

(可以参考: [Python命名空间和作用域窥探](http://python.jobbole.com/81367/))

local 变数我想大家应该不陌生，但这 non-local 又是啥鬼？

这主要是因应 nested scope 而衍生的定义。由于在 Python 里，所有东西都是 `object` ，而 `object` 是 first-class citizen (定义看[这边](https://en.wikipedia.org/wiki/First-class_citizen))，所以 function 也是 first-class citizen 。具体来说，由于这样的设计，你可以在 function 里定义 function 并回传 function 。又因为每定义一个 function 就会产生一个 scope ，所以当你在 function 里又定义 function 时，一个 nested scope 就会被产生 (不负责任白话翻译: 包在 scope 里的 scope)

上述 Python.org 的 [tutorial](https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces) 里就有提到，当走到 LEGB 的 E 时，Python 会从最近的 enclosing scope 向外找起，那这些 enclosing scopes 里的所有变数，就是所谓的 non-local variable。

举例说明好了

```python
def outer(a):
    b = a
    def inner():
        c = 3
        def inner_inner(b):
            r = b+c
            return b+c
        return inner_inner
    return inner
foo = outer(10)
bar = foo()
bar(1) # 4
```

对 `outer` 来说， `b` 是它的 local variable；对 `inner` 来说， `c` 是它的 local variable，另一方面虽然没用到，但是 `b` 是它的 non-local variable (因为离它最近的 scope 是 `outer` 所创造出来的 scope，而 `b` 是在这个 scope 里的)；对 `inner_inner` 来说， `r` 是它的 local variable ，值被指定为 `b+c` ，而这时的 `b` 并不是在 `outer` 的 scope 里被创造的 `b` ，而是经由参数传递进来的，所以也是 local variable 。反观 `c` ，则是 `inner` 的 scope 里的 `c` ，对 `inner_inner` 来说，是属于 non-local variable。

#### Under the Hood: __closure__

在科技里，我总是相信不存在魔法。所以到底 Python 的开发者是如何实现 function closure 的？

老实说我 C 很烂，所以我找不出来 CPython 是怎么实作 closure 的，但至少，我可以从 Python 官方的 language reference 找[**答案**](https://docs.python.org/3/reference/datamodel.html#index-34)。

其实也不是什么稀奇的事，就是 `__closure__` 这个属性 (attribute) 。根据 Python 的 Data Model 的定义， `__closure__` 会是一个唯读属性；资料型态是 `tuple` ，所以是 immutable 的。

那知道这个能干嘛？又是 read-only 又是 immutable ，好像也不知道能对它做什么。

说实在的，我也不知道能干嘛，但 language reference 里的一句话引起了我的注意:

>   `None` or a tuple of cells that contain bindings for the function’s free variables. See below for information on the `cell_contents` attribute.

那至少，我们可以用 `__closure__` 检视我们对于 Python closure 的理解吧？动手试试吧！

_1_.  有 non-local variable 就会有 `__closure__` ?

错！要有用到才会形成 `__closure__` ，否则就是 `None` 。

```python
def foo():
    a = 3
    def bar():
        print("bar")
    return bar
bar = foo()
bar.__closure__ is None
# >>> True
```

这个例子有趣的地方是，你可以发现当一个 function 的内容并没有用到任何 free variable 时，这时 `__closure__` 会是 `None` 。以上面的例子来看，虽然对 `bar` 来说，有个 `a` 这个 non-local variable ，但由于 `bar` 没有用到它，也因此没有任何 free variable，所以这时 `bar.__closure__` 也就还是 `None` 。

_2_.  没有 free variable 就不会有 `__closure__` ?

错！如果 inner scope 有用到 free variable ，就会被包含到 outer scope 里。

```python
def foo():
    a = 3
    def bar():
        def hell():
            return a
        return hell
    return bar
bar = foo()
bar.__closure__ is None
# False
bar.__closure__
# (<cell at 0x109d54408: int object at 0x10787eaf0>,)
bar().__closure__ == bar.__closure__
# True 
bar.__closure__[0].cell_contents
# 3
```

在这个例子里，虽然 `bar` 没有用到任何 free variable ，但是 `hell` 透过 nested scope 取得了 `foo` 里的 `a` ，间接影响了 `bar.__closure__` 。

_3_.  Bonus Question

以下代码有错误吗?

```python
def foo():
    def bar():
        return bar # Error?
    return bar
bar = foo() # Error?
```

会有 error 吗？

如果没有的话，想想为什么吧 : ) (**Hint: LEGB**)

#### 学 closure 要干嘛?

如果你的背景是 JavaScript ，我想这个问题应该是再明显不过，当然非常实用！举例来说，在 JavaScript 里不乏看到这样的代码:

```javascript
(function(msg){
  var x = 3;
  function inner(){
      // do things with x and msg
  }
  return inner;
})('my awesome string');
```

也就是利用匿名函数的 scope 进行 variable 的隔离 (例如上面的 `x`)。

很不幸的，Python 的 `lambda` 只能有一句 statement ，很难做到等价的 JavaScript code ，只能用 `def` (但这样就不匿名了 Q___Q)。

但除此之外，常见的就会是可带参数的 decorator 。假设你现在必须写个 function 的 decorator ，他会让函数在回传 `None` 时，改回传另一个你指定的值，那简单的实作可能会长这样:

```python
def return_default(value):
    def deco(func):
        def wrapped(*args, **kwargs):
            ret = func(*args, **kwargs)
            if ret is None:
                ret = value
            return ret
        return wrapped
    return deco
```

接着你可以这样使用 `return_default` :

```python
@return_default(10)
def at_least_10(x):
    if x >= 10:
        return x
@return_default("python")
def greeting(msg):
    return msg
x = at_least_10(3) # 10
msg = greeting(None) # "python"
```

简单的说， `return_default` 这个 decorator 利用了 closure ，在自己的 scope 里定义了一个 inner scope 把 `value` 放在里面的 `deco` 的 closure 里，才能做到这种夹带参数的 decorator 。

如果我们用刚刚学过的 `__closure__` 检视，就能发现:

```python
return_default(10).__closure__
# (<cell at 0x109d54528: int object at 0x10787ebd0>,)
return_default(10).__closure__[0].cell_contents
# 10
return_default("hello").__closure__[0].cell_contents
# 'hello'
```

如果这样还说服不了你，那让我用几个有名的 open source project 的 code 当例子吧:

-   [attrs 的 ](https://github.com/python-attrs/attrs/blob/e114561a061422f6ef0e720faeda4e3bd2baa9ac/src/attr/_make.py#L697)`@attr.s`
-   PySpark 的 `@since`


好啦，Python 的 function closure 就简单介绍到这里啦。

Happy Python Programming!

#### References

-   [https://en.wikipedia.org/wiki/Closure_(computer_programming)](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29)
-   <https://www.emacswiki.org/emacs/DynamicBindingVsLexicalBinding#toc2>
-   <https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces>
-   <https://gist.github.com/DmitrySoshnikov/700292>
-   <http://stupidpythonideas.blogspot.com/2015/12/how-lookup-works.html>
-   [https://docs.python.org/3/reference/datamodel.html](https://docs.python.org/3/reference/datamodel.html#index-34)
-   <http://python.jobbole.com/81367/>

>   改编自: [dboyliao@Medium](https://medium.com/@dboyliao/%E8%81%8A%E8%81%8A-python-closure-ebd63ff0146f)  