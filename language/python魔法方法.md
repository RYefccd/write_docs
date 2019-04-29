
# python魔法方法

[TOC]

## 特殊方法和魔法方法

------

Python中有大量类似`__doc__`这种以双下划线开头和结尾的特殊成员及“魔法方法”，它们有着非常重要的地位和作用，也是Python语言独具特色的语法之一！

这些成员里面有些是方法，调用时要加括号，有些是属性，调用时不需要加括号（废话！）。下面将一些常用的介绍一下，

### 1. 构造方法、析构方法

我们最为熟知的基本的魔法方法就是 `__init__` ，我们可以用它来指明一个对象初始化的行为。然而，当我们调用 x = SomeClass() 的时候， `__init__`并不是第一个被调用的方法。事实上，第一个被调用的是 `__new__` ，这个方法才真正地创建了实例。当这个对象的生命周期结束的时候， `__del__` 会被调用。让我们近一步理解这三个方法：

-  __\_\_new\_\___(cls, [...])

  `__new__` 是对象实例化时第一个调用的方法，它只取下 cls 参数，并把其他参数传给 `__init__` 。` __new__`很少使用，但是也有它适合的场景，尤其是当类继承自一个像元组或者字符串这样不可改变的类型的时候。我不打算深入讨论`__new__` ，因为它并不是很有用， [Python文档](http://www.python.org/download/releases/2.2/descrintro/#__new__) 中 有详细的说明。

- __\_\_init\_\___(self, [...])

  类的初始化方法。它获取任何传给构造器的参数（比如我们调用 x = SomeClass(10, 'foo') ， `__init__`就会接到参数 10 和 'foo' 。 `__init__` 在Python的类定义中用的最多。

- __\_\_del\_\___(self)

  `__new__`和` __init__`是对象的构造器， `__del__`是对象的销毁器。它并非实现了语句 `del x` (因此该语句不等同于` x.__del__()`)。而是定义了当对象被垃圾回收时的行为。 当对象需要在销毁时做一些处理的时候这个方法很有用，比如 socket 对象、文件对象。但是需要注意的是，当Python解释器退出但对象仍然存活的时候， `__del__` 并不会执行。 所以养成一个手工清理的好习惯是很重要的，比如及时关闭连接。

  注：此方法一般无须自定义，因为Python自带内存分配和释放机制，除非你需要在释放的时候指定做一些动作。析构函数的调用是由解释器在进行垃圾回收时自动触发执行的。

`__new__`方法创建实例对象供`__init__`方法使用，`__init__`方法定制实例对象。

`__new__ `方法必须返回值，`__init__`方法不需要返回值。(如果返回非None值就报错)

这里有个 `__init__` 和 `__del__`的例子:

```python
from os.path import join

class FileObject:
    '''文件对象的装饰类，用来保证文件被删除时能够正确关闭。'''

    def __init__(self, filepath='~', filename='sample.txt'):
        # 使用读写模式打开filepath中的filename文件
        self.file = open(join(filepath, filename), 'r+')

    def __del__(self):
        self.file.close()
        del self.file

obj = Foo()
del obj
```

### 2. 操作符

使用Python魔法方法的一个巨大优势就是可以构建一个拥有Python内置类型行为的对象。这意味着你可以避免使用非标准的、丑陋的方式来表达简单的操作。

####比较操作符

Python包含了一系列的魔法方法，用于实现对象之间直接比较，而不需要采用方法调用。同样也可以重载Python默认的比较方法，改变它们的行为。下面是这些方法的列表：

- __\_\_cmp\_\___(self, other)

   `__cmp__` 应该在 self < other 时返回一个负整数，在 self == other 时返回0，在 self > other 时返回正整数（在Python3中被废弃了）

- __\_\_eq\_\___(self, other)

  定义等于操作符`==`的行为。

- __\_\_ne\_\___(self, other)

  定义不等于操作符`!=`的行为。

- __\_\_lt\_\___(self, other)

  定义小于操作符`<`的行为。

- __\_\_gt\_\___(self, other)

  定义大于操作符`>`的行为。

- __\_\_le\_\___(self, other)

  定义小于等于操作符`<=`的行为。

- __\_\_ge\_\___(self, other)

  定义大于等于操作符`>=`的行为。

举个例子，假如我们想用一个类来存储单词。我们可能想按照字典序（字母顺序）来比较单词，字符串的默认比较行为就是这样。我们可能也想按照其他规则来比较字符串，像是长度，或者音节的数量。在这个例子中，我们使用长度作为比较标准，下面是一种实现:

```python
class Word(str):
    '''单词类，按照单词长度来定义比较行为'''

    def __new__(cls, word):
        # 注意，我们只能使用 __new__ ，因为str是不可变类型
        # 所以我们必须提前初始化它（在实例创建时）
        if ' ' in word:
            print "Value contains spaces. Truncating to first space."
            word = word[:word.index(' ')]
            # Word现在包含第一个空格前的所有字母
        return str.__new__(cls, word)

    def __gt__(self, other):
        return len(self) > len(other)
    def __lt__(self, other):
        return len(self) < len(other)
    def __ge__(self, other):
        return len(self) >= len(other)
    def __le__(self, other):
        return len(self) <= len(other)
```

现在我们可以创建两个 Word 对象（ Word('foo') 和 Word('bar'))然后根据长度来比较它们。注意我们没有定义 `__eq__` 和 `__ne__` ，这是因为有时候它们会导致奇怪的结果（很明显， Word('foo') == Word('bar') 得到的结果会是true）。根据长度测试是否相等毫无意义，所以我们使用 str 的实现来比较相等。

从上面可以看到，不需要实现所有的比较魔法方法，就可以使用丰富的比较操作。标准库还在 functools 模块中提供了一个类装饰器，只要我们定义 `__eq__` 和另外一个操作符（ `__gt__`, `__lt__` 等），它就可以帮我们实现比较方法。它能帮助我们节省大量的时间和精力。要使用它，只需要它 @total_ordering 放在类的定义之上就可以了

####数值操作符

就像你可以使用比较操作符来比较类的实例，你也可以定义数值操作符的行为。我把它们分成了五类：一元操作符，常见算数操作符，反射算数操作符（后面会涉及更多），原地赋值操作符，和类型转换操作符。

#####一元操作符

一元操作符只有一个操作符。

- __\_\_pos\_\___(self)

  实现取正操作，例如 `+some_object`。

- __\_\_neg\_\___(self)

  实现取负操作，例如 `-some_object`。

- __\_\_abs\_\___(self)

  实现内建绝对值函数 `abs()` 操作。

- __\_\_invert\_\___(self)

  实现取反操作符 `~`。

- __\_\_round\_\___(self， n)

  实现内建函数 `round()` ，n 是近似小数点的位数。

- __\_\_floor\_\___(self)

  实现 `math.floor()` 函数，即向下取整。

- __\_\_ceil\_\___(self)

  实现 `math.ceil()` 函数，即向上取整。

- __\_\_trunc\_\___(self)

  实现 `math.trunc()` 函数，即距离零最近的整数。

#####常见算数操作符

现在，我们来看看常见的二元操作符（和一些函数），像`+，-，*`之类的，它们很容易从字面意思理解。

- __\_\_\_\_add\_\___(self, other)

  实现加法操作。

- __\_\_sub\_\___(self, other)

  实现减法操作。

- __\_\_mul\_\___(self, other)

  实现乘法操作。

- __\_\_floordiv\_\___(self, other)

  实现使用 // 操作符的整数除法。

- __\_\_div\_\___(self, other)

  实现使用 / 操作符的除法。该方法在Python3中废弃. 原因是Python3中，division默认就是true division。

- __\_\_truediv\_\___(self, other)

  实现 _true_ 除法，这个函数只有使用 `from __future__ import division`时才有作用。

- __\_\_mod\_\___(self, other)

  实现 `%` 取余操作。

- __\_\_divmod\_\___(self, other)

  实现 `divmod` 内建函数。

- __\_\_pow\_\___

  实现 `**` 操作符。

- __\_\_lshift\_\___(self, other)

  实现左移位运算符 `<<` 。

- __\_\_rshift\_\___(self, other)

  实现右移位运算符 `>>` 。

- __\_\_and\_\___(self, other)

  实现按位与运算符 `&` 。

- __\_\_or\_\___(self, other)

  实现按位或运算符 `|` 。

- __\_\_xor\_\___(self, other)

  实现按位异或运算符 `^` 。

##### 反射算数运算符

假设针对some_object这个对象:

```python
some_object + other
```

上面的代码非常正常地实现了some_object的`__add__`方法。那么如果遇到相反的情况呢?

```Python
other + some_object
```

这时候，如果other没有定义`__add__`方法，但是some_object定义了`__radd__`, 那么上面的代码照样可以运行（只有当 other 没有定义 `__add__` 时 `some_object.__radd__` 才会被调用）。
这里的`__radd__(self, other)`就是`__add__(self, other)`的反算术运算符。

- __\_\_radd\_\___(self, other)

  实现反射加法操作。

- __\_\_rsub\_\___(self, other)

  实现反射减法操作。

- __\_\_rmul\_\___(self, other)

  实现反射乘法操作。

- __\_\_rfloordiv\_\___(self, other)

  实现使用 `//` 操作符的整数反射除法。

- __\_\_rdiv\_\___(self, other)

  实现使用 `/` 操作符的反射除法。

- __\_\_rtruediv\_\___(self, other)

  实现 _true_ 反射除法，这个函数只有使用 `from __future__ import division` 时才有作用。

- __\_\_rmod\_\___(self, other)

  实现 `%` 反射取余操作符。

- __\_\_rdivmod\_\___(self, other)

  实现调用 `divmod(other, self)` 时 `divmod` 内建函数的操作。

- __\_\_rpow\_\___

  实现 `**` 反射操作符。

- __\_\_rlshift\_\___(self, other)

  实现反射左移位运算符 `<<` 的作用。

- __\_\_rshift\_\___(self, other)

  实现反射右移位运算符` >>` 的作用。

- __\_\_rand\_\___(self, other)

  实现反射按位与运算符 `&` 。

- __\_\_ror\_\___(self, other)

  实现反射按位或运算符 `|` 。

- __\_\_rxor\_\___(self, other)

  实现反射按位异或运算符 `^` 。

##### 原地赋值运算符

Python同样提供了大量的魔法方法，可以用来自定义原地赋值操作的行为。或许你已经了解原地赋值，它融合了“常见”的操作符和赋值操作，如果你还是没听明白，看下面的例子:

```python
x = 5
x += 1 # 也就是 x = x + 1
```

这些方法都应该返回左侧操作数应该被赋予的值（例如， a += b `__iadd__` 也许会返回 a + b ，这个结果会被赋给 a ）,下面是方法列表：

- __\_\_iadd\_\___(self, other)

  实现加法赋值操作。

- __\_\_isub\_\___(self, other)

  实现减法赋值操作。

- __\_\_imul\_\___(self, other)

  实现乘法赋值操作。

- __\_\_ifloordiv\_\___(self, other)

  实现使用 `//=` 操作符的整数除法赋值操作。

- __\_\_idiv\_\___(self, other)

  实现使用 `/=` 操作符的除法赋值操作。

- __\_\_itruediv\_\___(self, other)

  实现 _true_ 除法赋值操作，这个函数只有使用 `from __future__ import division` 时才有作用。

- __\_\_imod\_\___(self, other)

  实现 `%=` 取余赋值操作。

- __\_\_ipow\_\___

  实现 `**=` 操作。

- __\_\_ilshift\_\___(self, other)

  实现左移位赋值运算符 `<<= `。

- __\_\_irshift\_\___(self, other)

  实现右移位赋值运算符 `>>=` 。

- __\_\_iand\_\___(self, other)

  实现按位与运算符 `&=` 。

- __\_\_ior\_\___(self, other)

  实现按位或赋值运算符 `|=` 。

- __\_\_ixor\_\___(self, other)

  实现按位异或赋值运算符 `^=` 。

##### 类型转换操作符

Python也有一系列的魔法方法用于实现类似 `float()` 的内建类型转换函数的操作。它们是这些：

- __\_\_int\_\___(self)

  实现到`int`的类型转换。

- __\_\_long\_\_\_\___(self)

  实现到`long`的类型转换。

- __\_\_float\_\___(self)

  实现到`floa`t的类型转换。

- __\_\_complex\_\___(self)

  实现到`complex`的类型转换。

- __\_\_oct\_\___(self)

  实现到八进制数的类型转换。

- __\_\_hex\_\___(self)

  实现到十六进制数的类型转换。

- __\_\_index\_\___(self)

  实现当对象用于切片表达式时到一个整数的类型转换。如果你定义了一个可能会用于切片操作的数值类型，你应该定义 `__index__`。

  在切片运算中将对象转化为int, 因此该方法的返回值必须是int。用一个例子来解释这个用法。

  ```python
  class Thing(object):
      def __index__(self):
          return 1
  
  thing = Thing()
  list_ = ['a', 'b', 'c']
  print list_[thing]  # 'b'
  print list_[thing:thing]  # []
  ```

  上面例子中, `list_[thing]`的表现跟`list_[1]`一致，正是因为Thing实现了`__index__`方法。

  可能有的人会想，`list_[thing]`为什么不是相当于`list_[int(thing)]`呢? 通过实现Thing的`__int__`方法能否达到这个目的呢?

  显然不能。如果真的是这样的话，那么`list_[1.1:2.2]`这样的写法也应该是通过的。
  而实际上，该写法会抛出TypeError: `slice indices must be integers or None or have an __index__ method`

  下面我们再做个例子,如果对一个dict对象执行`dict_[thing]`会怎么样呢?

  ```python
  dict_ = {1: 'apple', 2: 'banana', 3: 'cat'}
  print dict_[thing]  # raise KeyError
  ```

  这个时候就不是调用`__index__`了。虽然`list`和`dict`都实现了`__getitem__`方法, 但是它们的实现方式是不一样的。
  如果希望上面例子能够正常执行, 需要实现Thing的`__hash__` 和 `__eq__`方法.



  ```python
  class Thing(object):
      def __hash__(self):
          return 1
      def __eq__(self, other):
          return hash(self) == hash(other)
  
  dict_ = {1: 'apple', 2: 'banana', 3: 'cat'}
  print dict_[thing]  # apple
  ```

- __\_\_trunc\_\___(self)

  当调用 `math.trunc(self)` 时调用该方法， __trunc__ 应该返回 self 截取到一个整数类型（通常是long类型）的值。

- __\_\_coerce\_\___(self)

  该方法用于实现混合模式算数运算，如果不能进行类型转换， `__coerce__` 应该返回 None 。反之，它应该返回一个二元组 self 和 other ，这两者均已被转换成相同的类型。

  要了解这个方法,需要先了解`coerce()`内建函数: [官方文档](https://docs.python.org/2/library/functions.html#coerce)上的解释是, coerce(x, y)返回一组数字类型的参数, 它们被转化为同一种类型，以便它们可以使用相同的算术运算符进行操作。如果过程中转化失败，抛出TypeError。

  比如对于`coerce(10, 10.1)`, 因为10和10.1在进行算术运算时，会先将10转为10.0再来运算。因此`coerce(10, 10.1)`返回值是(10.0, 10.1).

  `__coerce__`在Python3中废弃了。

### 3. 类的表示

使用字符串来表示类是一个相当有用的特性。在Python中有一些内建方法可以返回类的表示，相对应的，也有一系列魔法方法可以用来自定义在使用这些内建函数时类的行为。

- __\_\_str\_\___(self)

  定义对类的实例调用 `str()` 时的行为。

- __\_\_repr\_\___(self)

  定义对类的实例调用 `repr()` 时的行为。 `str()` 和 `repr()` 最主要的差别在于“目标用户”。 `repr()` 的作用是产生机器可读的输出（大部分情况下，其输出可以作为有效的Python代码），而 `str()` 则产生人类可读的输出。

- __\_\_unicode\_\___(self)

  对实例使用`unicode()`时调用。`unicode()`与`str()`的区别在于：前者返回值是unicode，后者返回值是str。unicode和str都是`basestring`的子类。

  当你对一个类只定义了`__str__`但没定义`__unicode__`时，`__unicode__`会根据`__str__`的返回值自动实现，即`return unicode(self.__str__())`；但返回来则不成立。

  Python3中，str与unicode的区别被废除了，因而`__unicode__`没有了，取而代之地出现了`__bytes__`。

- __\_\_format\_\___(self, formatstr)

  定义当类的实例用于新式字符串格式化时的行为，例如，"`Hello, {0:abc}".format(a)`等价于`format(a, "abc")`，等价于`a.__format__("abc")`。当定义你自己的数值类型或字符串类型时，你可能想提供某些特殊的格式化选项，这种情况下这个魔法方法会非常有用。

- __\_\_hash\_\___(self)

  定义对类的实例调用 `hash()` 时的行为。它必须返回一个整数，其结果会被用于字典中键的快速比较。同时注意一点，实现这个魔法方法通常也需要实现 `__eq__` ，并且遵守如下的规则： `a == b` 意味着 `hash(a) == hash(b)`。

- __\_\_nonzero\_\___(self)

  定义对类的实例调用 bool() 时的行为，根据你自己对类的设计，针对不同的实例，这个魔法方法应该相应地返回True或False。在Python3中改名为`__bool__`了。

- __\_\_dir\_\___(self)

  定义对类的实例调用 `dir()` 时的行为，这个方法应该向调用者返回一个属性列表。一般来说，没必要自己实现 `__dir__` 。但是如果你重定义了 `__getattr__` 或者 `__getattribute__` （下个部分会介绍），乃至使用动态生成的属性，以实现类的交互式使用，那么这个魔法方法是必不可少的。

###4. 属性访问控制

总有人要吐槽Python缺少对于类的封装,比如希望Python能够定义私有属性，然后提供公共可访问的getter和 setter。Python其实可以通过魔术方法来实现封装。

- __\_\_getattr\_\___(self, name)

该方法定义了你试图访问一个不存在的属性时的行为。因此，重载该方法可以实现捕获错误拼写然后进行重定向, 或者对一些废弃的属性进行警告。

- __\_\_setattr\_\___(self, name, value)

`__setattr__` 是实现封装的解决方案，它定义了你对属性进行赋值和修改操作时的行为。
不管对象的某个属性是否存在,它都允许你为该属性进行赋值,因此你可以为属性的值进行自定义操作。有一点需要注意，实现`__setattr__`时要避免"无限递归"的错误，下面的代码示例中会提到。

- __\_\_delattr\_\___(self, name)

`__delattr__`与`__setattr__`很像，只是它定义的是你删除属性时的行为。实现`__delattr__`是同时要避免"无限递归"的错误。

- __\_\_getattribute\_\___(self, name)

`__getattribute__`定义了你的属性被访问时的行为，相比较，`__getattr__`只有该属性不存在时才会起作用。
因此，在支持`__getattribute__`的Python版本,调用`__getattr__`前必定会调用 `__getattribute__`。`__getattribute__`同样要避免"无限递归"的错误。
需要提醒的是，最好不要尝试去实现`__getattribute__`,因为很少见到这种做法，而且很容易出bug。

例子说明`__setattr__`的无限递归错误:

```python
def __setattr__(self, name, value):
    self.name = value
    # 每一次属性赋值时, __setattr__都会被调用，因此不断调用自身导致无限递归了。
```

因此正确的写法应该是:

```python
def __setattr__(self, name, value):
    self.__dict__[name] = value
```

如果在`__delattr__`实现中出现`del self.name` 这样的代码也会出现"无限递归"错误，这是一样的原因。

下面的例子很好的说明了上面介绍的4个魔术方法的调用情况:

```python
class Access(object):

    def __getattr__(self, name):
        print('__getattr__')
        return super(Access, self).__getattr__(name)

    def __setattr__(self, name, value):
        print('__setattr__')
        return super(Access, self).__setattr__(name, value)

    def __delattr__(self, name):
        print('__delattr__')
        return super(Access, self).__delattr__(name)

    def __getattribute__(self, name):
        print('__getattribute__')
        return super(Access, self).__getattribute__(name)

access = Access()
access.attr1 = True  # __setattr__调用
access.attr1  # 属性存在,只有__getattribute__调用
try:
    access.attr2  # 属性不存在, 先调用__getattribute__, 后调用__getattr__
except AttributeError:
    pass
del access.attr1  # __delattr__调用
```

###5. 构造自定义容器

如果我们要自定义一些数据结构，使之能够跟内置的容器类型表现一样，那就需要去实现某些协议。

这里的协议跟其他语言中所谓的"接口"概念很像，一样的需要你去实现才行，只不过没那么正式而已。

如果要自定义不可变容器类型，只需要定义`__len__` 和 `__getitem__`方法;
如果要自定义可变容器类型，还需要在不可变容器类型的基础上增加定义`__setitem__` 和 `__delitem__`。
如果你希望你的自定义数据结构还支持"可迭代", 那就还需要定义`__iter__`。

- __\_\_len\_\___(self)

需要返回数值类型，以表示容器的长度。该方法在可变容器和不可变容器中必须实现。

- __\_\_getitem\_\___(self, key)

当你执行`self[key]`的时候，调用的就是该方法。该方法在可变容器和不可变容器中也都必须实现。
调用的时候，如果key的类型错误，该方法应该抛出TypeError；
如果没法返回key对应的数值时，该方法应该抛出ValueError。

- __\_\_setitem\_\___(self, key, value)

当你执行`self[key] = value`时，调用的是该方法。

- __\_\_delitem\_\___(self, key)

当你执行`del self[key]`的时候，调用的是该方法。

- __\_\_iter\_\___(self)

该方法需要返回一个迭代器(iterator)。当你执行`for x in container:` 或者使用`iter(container)`时，该方法被调用。

- __\_\_reversed\_\___(self)

如果想要该数据结构被内建函数`reversed()`支持,就还需要实现该方法。

- __\_\_contains\_\___(self, item)

如果定义了该方法，那么在执行`item in container` 或者 `item not in container`时该方法就会被调用。
如果没有定义，那么Python会迭代容器中的元素来一个一个比较，从而决定返回True或者False。

- __\_\_missing\_\___(self, key)

`dict`字典类型会有该方法，它定义了key如果在容器中找不到时触发的行为。
比如`d = {'a': 1}`, 当你执行`d[notexist]`时，`d.__missing__('notexist')`就会被调用。

```python
class FunctionalList:
    '''一个列表的封装类，实现了一些额外的函数式
    方法，例如head, tail, init, last, drop和take。'''

    def __init__(self, values=None):
        if values is None:
            self.values = []
        else:
            self.values = values

    def __len__(self):
        return len(self.values)

    def __getitem__(self, key):
        # 如果键的类型或值不合法，列表会返回异常
        return self.values[key]

    def __setitem__(self, key, value):
        self.values[key] = value

    def __delitem__(self, key):
        del self.values[key]

    def __iter__(self):
        return iter(self.values)

    def __reversed__(self):
        return reversed(self.values)

    def append(self, value):
        self.values.append(value)

    def head(self):
        # 取得第一个元素
        return self.values[0]

    def tail(self):
        # 取得除第一个元素外的所有元素
        return self.valuse[1:]

    def init(self):
        # 取得除最后一个元素外的所有元素
        return self.values[:-1]

    def last(self):
        # 取得最后一个元素
        return self.values[-1]

    def drop(self, n):
        # 取得除前n个元素外的所有元素
        return self.values[n:]

    def take(self, n):
        # 取得前n个元素
        return self.values[:n]
```

### 6. 自省

你可以通过定义魔法方法来控制用于自省（自省（反射）就是面向对象的语言所写的程序在运行时，所能知道对象的类型。简单一句话就是运行时能够获得对象的类型）的内建函数 `isinstance` 和 `issubclass` 的行为。下面是对应的魔法方法：

- __\_\_instancecheck\_\___(self, instance)

  检查一个实例是否是你定义的类的一个实例（例如 `isinstance(instance, class)` ）。

- __\_\_subclasscheck\_\___(self, subclass)

  检查一个类是否是你定义的类的子类（例如 `issubclass(subclass, class)`）。

这几个魔法方法的适用范围看起来有些窄，事实也正是如此。我不会在反射魔法方法上花费太多时间，因为相比其他魔法方法它们显得不是很重要。但是它们展示了在Python中进行面向对象编程（或者总体上使用Python进行编程）时很重要的一点：不管做什么事情，都会有一个简单方法，不管它常用不常用。

### 7. 抽象基类

https://docs.python.org/3.7/library/abc.html

### 8. 可调用对象

你可能已经知道了，在Python中，函数是一等的对象。这意味着它们可以像其他任何对象一样被传递到函数和方法中，这是一个十分强大的特性。

Python中一个特殊的魔法方法允许你自己类的对象表现得像是函数，然后你就可以“调用”它们，把它们传递到使用函数做参数的函数中。

那么，怎么判断一个对象是否可以被执行呢？能被执行的对象就是一个Callable对象，可以用Python内建的callable()函数进行测试。

- __\_\_call\_\___(self, [args...])

  允许类的一个实例像函数那样被调用。本质上这代表了 `x()` 和 `x.__call__()` 是相同的。注意 `__call__` 可以有多个参数，这代表你可以像定义其他任何函数一样，定义 `__call__` ，喜欢用多少参数就用多少。

`__call__` 在某些需要经常改变状态的类的实例中显得特别有用。“调用”这个实例来改变它的状态，是一种更加符合直觉，也更加优雅的方法。一个表示平面上实体的类是一个不错的例子:

```python
class Entity:
        '''表示一个实体的类，调用它的实例
        可以更新实体的位置'''

        def __init__(self, size, x, y):
                self.x, self.y = x, y
                self.size = size

        def __call__(self, x, y):
                '''改变实体的位置'''
                self.x, self.y = x, y
```

### 9. 上下文管理

`with`声明是从Python2.5开始引进的关键词。你应该遇过这样子的代码:

```Python
with open('foo.txt') as bar:
    # do something with bar
```

在with声明的代码段中，我们可以做一些对象的开始操作和清除操作,还能对异常进行处理。
这需要实现两个魔术方法: `__enter__` 和 `__exit__`。

- __\_\_enter\_\___(self)

`__enter__`会返回一个值，并赋值给`as`关键词之后的变量。在这里，你可以定义代码段开始的一些操作。

- __\_\_exit\_\___(self, exception_type, exception_value, traceback)

`__exit__`定义了代码段结束后的一些操作，可以这里执行一些清除操作，或者做一些代码段结束后需要立即执行的命令，比如文件的关闭，socket断开等。如果代码段成功结束，那么`exception_type, exception_value, traceback` 三个参数传进来时都将为None。如果代码段抛出异常，那么传进来的三个参数将分别为: 异常的类型，异常的值，异常的追踪栈。
如果`__exit__`返回True, 那么with声明下的代码段的一切异常将会被屏蔽。
如果`__exit__`返回None, 那么如果有异常，异常将正常抛出，这时候with的作用将不会显现出来。

举例说明：

这该示例中，IndexError始终会被隐藏，而TypeError始终会抛出。

```python
class DemoManager(object):

    def __enter__(self):
        pass

    def __exit__(self, ex_type, ex_value, ex_tb):
        if ex_type is IndexError:
            print ex_value.__class__
            return True
        if ex_type is TypeError:
            print ex_value.__class__
            return  # return None

with DemoManager() as nothing:
    data = [1, 2, 3]
    data[4]  # raise IndexError, 该异常被__exit__处理了

with DemoManager() as nothing:
    data = [1, 2, 3]
    data['a']  # raise TypeError, 该异常没有被__exit__处理

'''
输出:
<type 'exceptions.IndexError'>
<type 'exceptions.TypeError'>
Traceback (most recent call last):
  ...
'''
```

### 10. 描述器对象

描述符是一个类，当使用取值，赋值和删除时它可以改变其他对象。描述符不是用来单独使用的，它们需要被一个拥有者类所包含。描述符可以用来创建面向对象数据库，以及创建某些属性之间互相依赖的类。描述符在表现具有不同单位的属性，或者需要计算的属性时显得特别有用（例如表现一个坐标系中的点的类，其中的距离原点的距离这种属性）。

要想成为一个描述符，一个类必须具有实现 `__get__` , `__set__` 和 `__delete__`三个方法中至少一个。

让我们一起来看一看这些魔法方法：

- __\_\_get\_\___(self, instance, owner)

  定义当试图取出描述符的值时的行为。 instance 是拥有者类的实例， owner 是拥有者类本身。

- __\_\_set\_\___(self, instance, owner)

  定义当描述符的值改变时的行为。 instance 是拥有者类的实例， value 是要赋给描述符的值。

- __\_\_delete\_\___(self, instance, owner)

  定义当描述符的值被删除时的行为。 instance 是拥有者类的实例

现在，来看一个描述符的有效应用：单位转换:

```python
class Meter(object):
    '''米的描述符。'''

    def __init__(self, value=0.0):
        self.value = float(value)
    def __get__(self, instance, owner):
        return self.value
    def __set__(self, instance, owner):
        self.value = float(value)

class Foot(object):
    '''英尺的描述符。'''

    def __get(self, instance, owner):
        return instance.meter * 3.2808
    def __set(self, instance, value):
        instance.meter = float(value) / 3.2808

class Distance(object):
    '''用于描述距离的类，包含英尺和米两个描述符。'''
    meter = Meter()
    foot = Foot()
```

### 11. 拷贝

- __\_\_copy\_\___(self)

  定义对类的实例使用 `copy.copy()` 时的行为。 `copy.copy()` 返回一个对象的浅拷贝，这意味着拷贝出的实例是全新的，然而里面的数据全都是引用的。也就是说，对象本身是拷贝的，但是它的数据还是引用的（所以浅拷贝中的数据更改会影响原对象）。

- __\_\_deepcopy\_\___(self, memodict=)

  定义对类的实例使用 `copy.deepcopy()` 时的行为。 `copy.deepcopy()` 返回一个对象的深拷贝，这个对象和它的数据全都被拷贝了一份。 memodict 是一个先前拷贝对象的缓存，它优化了拷贝过程，而且可以防止拷贝递归数据结构时产生无限递归。当你想深拷贝一个单独的属性时，在那个属性上调用 `copy.deepcopy()` ，使用 memodict 作为第一个参数。

###12. 对象的序列化

Python对象的序列化操作是pickling进行的。pickling非常的重要，以至于Python对此有单独的模块`pickle`，还有一些相关的魔术方法。使用pickling, 你可以将数据存储在文件中，之后又从文件中进行恢复。

下面举例来描述pickle的操作。从该例子中也可以看出,如果通过pickle.load 初始化一个对象, 并不会调用`__init__`方法。

```python
# -*- coding: utf-8 -*-
from datetime import datetime
import pickle

class Distance(object):

    def __init__(self, meter):
        print 'distance __init__'
        self.meter = meter

data = {
    'foo': [1, 2, 3],
    'bar': ('Hello', 'world!'),
    'baz': True,
    'dt': datetime(2016, 10, 01),
    'distance': Distance(1.78),
}
print 'before dump:', data
with open('data.pkl', 'wb') as jar:
    pickle.dump(data, jar)  # 将数据存储在文件中

del data
print 'data is deleted!'

with open('data.pkl', 'rb') as jar:
    data = pickle.load(jar)  # 从文件中恢复数据
print 'after load:', data
```

值得一提，从其他文件进行pickle.load操作时，需要注意有恶意代码的可能性。另外，Python的各个版本之间,pickle文件可能是互不兼容的。

pickling并不是Python的內建类型，它支持所有实现pickle协议(可理解为接口)的类。pickle协议有以下几个可选方法来自定义Python对象的行为。

- __\_\_getinitargs\_\___(self)

如果你希望unpickle时，`__init__`方法能够调用，那么就需要定义`__getinitargs__`, 该方法需要返回一系列参数的元组，这些参数就是传给`__init__`的参数。

该方法只对`old-style class`有效。所谓`old-style class`,指的是不继承自任何对象的类，往往定义时这样表示: `class A:`, 而非`class A(object):`

- __\_\_getnewargs\_\___(self)

跟`__getinitargs__`很类似，只不过返回的参数元组将传值给`__new__`

- __\_\_getstate\_\___(self)

在调用`pickle.dump`时，默认是对象的`__dict__`属性被存储，如果你要修改这种行为，可以在`__getstate__`方法中返回一个state。state将在调用`pickle.load`时传值给`__setstate__`

- __\_\_setstate\_\___(self, state)

一般来说,定义了`__getstate__`,就需要相应地定义`__setstate__`来对`__getstate__`返回的state进行处理。

- __\_\_reduce\_\___(self)

如果pickle的数据包含了自定义的扩展类（比如使用C语言实现的Python扩展类）时，就需要通过实现`__reduce__`方法来控制行为了。由于使用过于生僻，这里就不展开继续讲解了。

令人容易混淆的是，我们知道, `reduce()`是Python的一个內建函数, 需要指出`__reduce__`并非定义了`reduce()`的行为，二者没有关系。

- __\_\_reduce_ex\_\___(self)

`__reduce_ex__` 是为了兼容性而存在的, 如果定义了`__reduce_ex__`, 它将代替`__reduce__` 执行。

下面的代码示例很有意思，我们定义了一个类Slate(中文是板岩的意思)。这个类能够记录历史上每次写入给它的值,但每次`pickle.dump`时当前值就会被清空，仅保留了历史。

```python
# -*- coding: utf-8 -*-
import pickle
import time

class Slate:
    '''Class to store a string and a changelog, and forget its value when pickled.'''
    def __init__(self, value):
        self.value = value
        self.last_change = time.time()
        self.history = []

    def change(self, new_value):
        # 修改value, 将上次的valeu记录在history
        self.history.append((self.last_change, self.value))
        self.value = new_value
        self.last_change = time.time()

    def print_changes(self):
        print 'Changelog for Slate object:'
        for k, v in self.history:
            print '%s    %s' % (k, v)

    def __getstate__(self):
        # 故意不返回self.value和self.last_change,
        # 以便每次unpickle时清空当前的状态，仅仅保留history
        return self.history

    def __setstate__(self, state):
        self.history = state
        self.value, self.last_change = None, None

slate = Slate(0)
time.sleep(0.5)
slate.change(100)
time.sleep(0.5)
slate.change(200)
slate.change(300)
slate.print_changes()  # 与下面的输出历史对比
with open('slate.pkl', 'wb') as jar:
    pickle.dump(slate, jar)
del slate  # delete it
with open('slate.pkl', 'rb') as jar:
    slate = pickle.load(jar)
print 'current value:', slate.value  # None
print slate.print_changes()  # 输出历史记录与上面一致
```

###其他

#### `__name__`

当前模块名。

####`__main__`

模块被直接运行时模块名为`__main__`

#### `__doc__`

说明性文档和信息。Python自建，无需自定义。

```python
class Foo:
    """ 描述类信息，可被自动收集 """

    def func(self):
        pass

# 打印类的说明文档 
print(Foo.__doc__)
```

#### `__module__` 和  `__class__`

`__module__` 表示当前操作的对象在属于哪个模块。

`__class__` 表示当前操作的对象属于哪个类。

这两者也是Python内建，无需自定义。

```python
class Foo:
    pass

obj = Foo()
print(obj.__module__)
print(obj.__class__)

------------
运行结果：
__main__
<class '__main__.Foo'>
```

####`__dict__`

列出类或对象中的所有成员！非常重要和有用的一个属性，Python自建，无需用户自己定义。

```python
class Province:
    country = 'China'
    def __init__(self, name, count):
        self.name = name
        self.count = count

    def func(self, *args, **kwargs):
        print（'func'）

# 获取类的成员
print(Province.__dict__)

# 获取 对象obj1 的成员 
obj1 = Province('HeBei',10000)
print(obj1.__dict__)

# 获取 对象obj2 的成员 
obj2 = Province('HeNan', 3888)
print(obj2.__dict__)
```

#### `__iter__()`

这是迭代器方法！列表、字典、元组之所以可以进行for循环，是因为其内部定义了 `__iter__()`这个方法。如果用户想让自定义的类的对象可以被迭代，那么就需要在类中定义这个方法，并且让该方法的返回值是一个可迭代的对象。当在代码中利用for循环遍历对象时，就会调用类的这个`__iter__()`方法。

普通的类：

```python
class Foo:
    pass

obj = Foo()

for i in obj:
    print(i)

# 报错：TypeError: 'Foo' object is not iterable<br># 原因是Foo对象不可迭代
```

添加一个`__iter__()`，但什么都不返回：

```python
class Foo:

    def __iter__(self):
        pass

obj = Foo()

for i in obj:
    print(i)

# 报错：TypeError: iter() returned non-iterator of type 'NoneType'

#原因是 __iter__方法没有返回一个可迭代的对象
```

返回一个可迭代对象：

```python
class Foo:

    def __init__(self, sq):
        self.sq = sq

    def __iter__(self):
        return iter(self.sq)

obj = Foo([11,22,33,44])

for i in obj:
    print(i)

# 这下没问题了！
```

最好的方法是使用生成器：

```python
class Foo:
    def __init__(self):
        pass

    def __iter__(self):
        yield 1
        yield 2
        yield 3

obj = Foo()
for i in obj:
    print(i)
```

####`__len__()`

在Python中，如果你调用内置的len()函数试图获取一个对象的长度，在后台，其实是去调用该对象的`__len__()`方法，所以，下面的代码是等价的：

```python
>>> len('ABC')
3
>>> 'ABC'.__len__()
3
```

Python的list、dict、str等内置数据类型都实现了该方法，但是你自定义的类要实现len方法需要好好设计。

#### `__slots__`

Python作为一种动态语言，可以在类定义完成和实例化后，给类或者对象继续添加随意个数或者任意类型的变量或方法，这是动态语言的特性。例如：

```python
def print_doc(self):
    print("haha")

class Foo:
    pass

obj1 = Foo()
obj2 = Foo()
# 动态添加实例变量
obj1.name = "jack"
obj2.age = 18
# 动态的给类添加实例方法
Foo.show = print_doc
obj1.show()
obj2.show()
```

但是！如果我想限制实例可以添加的变量怎么办？可以使`__slots__`限制实例的变量，比如，只允许Foo的实例添加name和age属性。

```python
def print_doc(self):
    print("haha")

class Foo:
    __slots__ = ("name", "age")
    pass


obj1 = Foo()
obj2 = Foo()
# 动态添加实例变量
obj1.name = "jack"
obj2.age = 18
obj1.sex = "male"       # 这一句会弹出错误
# 但是无法限制给类添加方法
Foo.show = print_doc
obj1.show()
obj2.show()
```

由于'sex'不在`__slots__`的列表中，所以不能绑定sex属性，试图绑定sex将得到AttributeError的错误。

```python
Traceback (most recent call last):
  File "F:/Python/pycharm/201705/1.py", line 14, in <module>
    obj1.sex = "male"
AttributeError: 'Foo' object has no attribute 'sex'
```

需要提醒的是，`__slots__`定义的属性仅对当前类的实例起作用，对继承了它的子类是不起作用的。想想也是这个道理，如果你继承一个父类，却莫名其妙发现有些变量无法定义，那不是大问题么？如果非要子类也被限制，除非在子类中也定义`__slots__`，这样，子类实例允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。



##python协议

### 一、协议的高度动态本性

#### 1、协议与接口的基本概念

**协议**：是Python中非正式的接口，是令Python这种动态类型语言实现多态的方式。
**接口**：泛指实体把自己提供给外界的一种抽象化物（可以为另一实体），用以由内部操作分离出外部沟通方法，使其能被内部修改而不影响外界其他实体与其交互的方式。
类的接口：类实现或继承的公开属性（方法或数据属性），包括特殊方法，如__\_\_getitem__\_\_或__\_\_add__\_\_。

#### 2、协议是非正式的

`协议是非正式的，只由文档和约定定义，不具备强制性。`
以序列协议为例，假设我们想实现迭代以及in运算，通常需要__\_\_iter__\_\_和__\_\_contains__\_\_方法，但事实上只实现__\_\_getitem__\_\_方法也可以。原因在于当Python发现没有__\_\_iter__\_\_和__\_\_contains__\_\_可用时，会转而调用__\_\_getitem__\_\_方法设法让迭代和in运算符可用。
**小结**：部分实现协议是有用的。

#### 3、使用猴子补丁在运行时实现协议

在运行时修改类或模块而不改动源码，从而实现目标协议接口的操作就是打猴子补丁。

### 二、抽象基类

#### 1、基本概念

`鸭子类型(duck typing)：不关注对象的类型，而是关注对象具有的行为(方法)。`鸭子类型像多态一样工作，但是没有继承。
在鸭子类型中，协议风格的接口与继承完全没有关系，实现同一个协议的各个类是相互独立的。

白鹅类型(goose typing)：只要cls是抽象基类，即cls的元类是abc.ABCMeta,就可以使用isinstance(obj,cls)。

抽象基类（abstract base class,ABC）：抽象基类就是类里定义了纯虚成员函数的类。纯虚函数只提供了接口，并没有具体实现。`抽象基类不能被实例化(不能创建对象)，通常是作为基类供子类继承，子类中重写虚函数，实现具体的接口。`简言之，ABC描述的是至少使用一个纯虚函数的接口，从ABC派生出的类将根据派生类的具体特征，使用常规虚函数来实现这种接口。

#### 2、标准库中的抽象基类

（1）collections.abc中抽象基类
collections.abc模块中各个抽象基类的UML类图如下所示：
![抽象基类的UML类图](/Users/hdc/Documents/其他/markdown_pic/抽象基类的UML类图.png)

- Iterable、 Container 和 Sized
  各个集合应该继承这三个抽象基类， 或者至少实现兼容的协议。
  `Iterable 通过 __iter_ 方法支持迭代；`
  `Container 通过__contains__ 方法支持 in 运算符；`
  `Sized 通过 __len__ 方法支持len() 函数。`
- Sequence、 Mapping 和 Set
  这三个是主要的不可变集合类型， 而且各自都有可变的子类。
- MappingView
  在 Python 3 中， 映射方法 .items()、 .keys() 和 .values() 返回的对象分别是 ItemsView、 KeysView 和 ValuesView 的实例。 前两个类还从 Set 类继承了丰富的接口，涉及集合的全部运算符。
- Callable 和 Hashable
  这两个抽象基类与集合没有太大的关系，只不过因为collections.abc 是标准库中定义抽象基类的第一个模块， 而它们又太重要了，因此才把它们放到 collections.abc 模块中。Callable 或 Hashable 的子类非常少见。这两个抽象基类的主要作用是为内置函数 isinstance 提供支持,以一种安全的方式判断对象能不能调用或散列。
- Iterator
  是 Iterable 的子类。

(2)numbers包中的数字塔
按照自上而下的线性结构，Number是位于最顶端的超类，详细排序如下：

- Number
- Complex
- Real
- Rational
- Integral

例如，检查一个数是否是整数，可以使用isinstance(x,numbers.Integral)。

### 三、抽象基类的使用

#### 1、通过继承声明抽象基类

声明抽象基类最简单的方式是继承abc.ABC或其他抽象基类：`Class Student(abc.ABC)`
注意：在Python3.0~3.3之间，继承抽象基类的语法是：Class Student(metaclass=abc.ABCMeta)。

#### 2、判断子类是否符合接口定义

在定义抽象基类的子类时，子类需要将继承的抽象基类中的抽象方法具体实现。

#### 3、声明虚拟子类实现抽象基类的接口

**虚拟子类**：指的是不通过继承而利用注册把一个类变成抽象基类的子类。
注册虚拟之类的方式是调用register方法，语法是@抽象基类名称.register。
`经注册后的虚拟子类可以被issubclass和isinstance等函数识别，但是注册的类不会从抽象基类中继承任何方法或属性。具体可通过类属性__mro__查看类的真实继承关系。`

### 四、其它

1、抽象基类中声明抽象类方法需要使用`@abc.abstractmethod`标记，并且在`@abc.abstractmethod`和`def`之间不能有其他装饰器。
2、内省类的继承关系的方法：
__\_\_subclasses__\_\_()：返回类的直接子类列表，不含虚拟子类。
__\_\_abcregistry：抽象基类独有的属性，是抽象类注册的虚拟子类的弱引用。
3、__\_\_subclasshook__\_\_：令抽象基类识别没有进行子类化和注册的类，此方法在abc.Sized中有应用。



