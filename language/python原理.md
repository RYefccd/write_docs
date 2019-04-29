# python笔记

[TOC]

## code objects

Code objects

Code objects represent *byte-compiled* executable Python code， or [bytecode](https://docs。python。org/3。7/glossary。html#term-bytecode)。 The difference between a code object and a function object is that the function object contains an explicit reference to the function’s globals (the module in which it was defined)， while a code object contains no context; also the default argument values are stored in the function object， not in the code object (because they represent values calculated at run-time)。 Unlike function objects， code objects are immutable and contain no references (directly or indirectly) to mutable objects。



Special read-only attributes: 

 `co_consts` is a tuple containing the literals used by the bytecode;

 `co_names` is a tuple containing the names used by the bytecode;

 `co_varnames` is a tuple containing the names of the local variables (starting with the argument names); 

 `co_freevars`is a tuple containing the names of free variables;

`co_name` gives the function name;

 `co_argcount` is the number of positional arguments (including arguments with default values); 

`co_nlocals` is the number of local variables used by the function (including arguments);

`co_cellvars` is a tuple containing the names of local variables that are referenced by nested functions;

 `co_code` is a string representing the sequence of bytecode instructions;

 `co_filename` is the filename from which the code was compiled;

 `co_firstlineno` is the first line number of the function;

 `co_lnotab` is a string encoding the mapping from bytecode offsets to line numbers (for details see the source code of the interpreter);

 `co_stacksize` is the required stack size (including local variables);

 `co_flags` is an integer encoding a number of flags for the interpreter。



## bytecode

LOAD_NAME(*namei*)：Pushes the value associated with `co_names[namei]` onto the stack。

LOAD_CONST(*consti*)：Pushes `co_consts[consti]` onto the stack

LOAD_FAST(*var_num*)：Pushes a reference to the local `co_varnames[var_num] `onto the stack

BINARY_SUBSCR：Implements `TOS = TOS1[TOS]`

STORE_SUBSCR：Implements `TOS1[TOS] = TOS2`

STORE_NAME：

SETUP_LOOP(*delta*)：Pushes a block for a loop onto the block stack。 The block spans from the current instruction with a size of *delta* bytes。

JUMP_ABSOLUTE(*target*)：Set bytecode counter to *target*。

POP_BLOCK：Removes one block from the block stack。 Per frame， there is a stack of blocks， denoting nested loops， try statements， and such。



## cpython栈

CPython 使用一个基于栈的虚拟机。也就是说，它是完全面向栈数据结构的。

CPython 使用三种类型的栈：

1. 调用栈call stack。这是运行 Python 程序的主要结构。它为每个当前活动的函数调用使用了一个东西 —— “帧（frame）”，栈底是程序的入口点。每个函数调用推送一个新的帧到调用栈，每当函数调用返回后，这个帧被销毁。
2. 在每个帧中，有一个计算栈evaluation stack （也称为数据栈data stack）。这个栈就是 Python 函数运行的地方，运行的 Python 代码大多数是由推入到这个栈中的东西组成的，操作它们，然后在返回后销毁它们。
3. 在每个帧中，还有一个块栈block stack。它被 Python 用于去跟踪某些类型的控制结构：循环、`try` / `except` 块、以及 `with` 块，全部推入到块栈中，当你退出这些控制结构时，块栈被销毁。这将帮助 Python 了解任意给定时刻哪个块是活动的，比如，一个 `continue` 或者 `break` 语句可能影响正确的块。



## 垃圾回收机制

 python的垃圾回收机制是以**引用计数**为主，以**标记清除**和**分代收集**为辅。

 python里"万物皆对象"。对象的核心是一个结构体:PyObject

```c
typedef struct_object {
 int ob_refcnt;
 struct_typeobject *ob_type;
} PyObject;
```

 ob_refcnt就是作为引用计数，当一个对象有新的引用时，ob_refcnt就会增加，当对象的引用被删除时，ob_refcnt就会减少，当引用计数为0时，该对象就会从内存中消失，占用的内存空间随即被释放。

 引用计数有2个优点
一是简单；二是实时性。一旦引用消失，内存就直接释放，不须等待到特定时机再处理。这样处理回收内存的时间也被分摊到了平时。

 引用计数的缺点之一是维护引用计数要消耗资源。
另外存在一个循环引用的问题。（引用计数不能处理环形数据结构--也就是含有循环引用的数据结构。引用计数在处理一个大数据结构时效率会很低，比如删除一个包含非常多元素的列表，Python可能必须一次性释放大量对象。减少引用就变成一项复杂的递归工程了。

```python
list1 = []
list2 = []
list1.append(list2)
list2.append(list1)
```

list1和list2相互引用，如果没有其他对象引用他们，那么这两个list的引用计数也仍为1，所占用的内存永远无法被回收。缺点一尚可被接受，但是循环引用会导致内存泄露，这就不得不引入标记清除和分代收集两个机制。

**标记清除（Mark—Sweep）**算法是一种基于追踪回收（tracing GC）技术实现的垃圾回收算法。它分为两个阶段：第一阶段是标记阶段，GC会把所有的『活动对象』打上标记，第二阶段是把那些没有标记的对象『非活动对象』进行回收。那么GC又是如何判断哪些是活动对象哪些是非活动对象的呢？

对象之间通过引用（指针）连在一起，构成一个有向图，对象构成这个有向图的节点，而引用关系构成这个有向图的边。从根对象（root object）出发，沿着有向边遍历对象，可达的（reachable）对象标记为活动对象，不可达的对象就是要被清除的非活动对象。根对象就是全局变量、调用栈、寄存器。

标记清除算法作为Python的辅助垃圾收集技术主要处理的是一些容器对象，比如list、dict、tuple，instance等，因为对于字符串、数值对象是不可能造成循环引用问题。Python使用一个双向链表将这些容器对象组织起来。不过，这种简单粗暴的标记清除算法也有明显的缺点：清除非活动的对象前它必须顺序扫描整个堆内存，哪怕只剩下小部分活动对象也要扫描所有对象。

**分代回收**是一种以空间换时间的操作方式，Python将内存根据对象的存活时间划分为不同的集合，每个集合称为一个代，Python将内存分为了3“代”，分别为年轻代（第0代）、中年代（第1代）、老年代（第2代），他们对应的是3个链表，它们的垃圾收集频率与对象的存活时间的增大而减小。新创建的对象都会分配在年轻代，年轻代链表的总数达到上限时，Python垃圾收集机制就会被触发，把那些可以被回收的对象回收掉，而那些不会回收的对象就会被移到中年代去，依此类推，老年代中的对象是存活时间最久的对象，甚至是存活于整个系统的生命周期内。同时，分代回收是建立在标记清除技术基础之上。分代回收同样作为Python的辅助垃圾收集技术处理那些容器对象。



##描述器

一般来说，描述器（descriptor）是一个有”绑定行为”的对象属性（object attribute），它的**属性访问**被描述器协议方法重写。这些方法是 `__get__()`、 `__set__()`和 `__delete__()`。如果一个对象定义了以上任意一个方法，它就是一个描述器。描述器本质上是一个类对象。（作用：将访问属性转换为访问方法，可以添加一个其他的逻辑（资料描述器））



**描述器的调用机制**

下面我们来说一下，当我们调用`a.m`时的访问顺序

- 程序会先查找 `a.__dict__['m']` 是否存在
- 不存在再到`type(a).__dict__['m']`中查找
- 然后找`type(a)`的父类(不包括元类(metaclass))
- 期间找到的是普通值就输出，如果找到的是一个描述器，则调用`__get__`方法



**调用描述器的原理**：当调用一个属性，而属性指向一个描述器时，为什么就会去调用这个描述器呢，其实这是由`object.__getattribute__()`方法控制的。新定义的一个类继承了object类，也就继承了`__getattribute__`方法。当访问一个属性比如`b.x`时，会自动调用这个方法 `__getattribute__()`的定义如下

```python
def __getattribute__(self, key):
    "Emulate type_getattro() in Objects/typeobject.c"
    v = object.__getattribute__(self, key)
    if hasattr(v, '__get__'):
        return v.__get__(None, self)
    return v
```

上面的定义显示，如果`b.x`是一个描述器对象，即能找到`__get__`方法，则会调用这个get方法，否则就使用普通的属性。 如果在一个类中重写`__getattribute__`，将会改变描述器的行为，甚至将描述器这一功能关闭。



**`__get__`和`__set__`方法中的参数解释**

```python
descr.__get__(self, obj, type=None) -> value
descr.__set__(self, obj, value) -> None
descr.__delete__(self, obj) -> None
```

```python
class M:
    def __init__(self, name):
        self.name = name
        
    def __get__(self, obj, type):
        print('get第一个参数self: ', self.name)
        print('get第二个参数obj: ', obj.age)
        print('get第三个参数type: ', type.name)
        
    def __set__(self, obj, value):
        obj.__dict__[self.name] = value
        
class A:
    name = 'Bob'
    m = M('age')
    def __init__(self, age):
        self.age = age

a = A(20) # age是20
a.m
# get第一个参数self:  age
# get第二个参数obj:  20
# get第三个参数type:  Bob
a.m = 30
a.age # 30
```

总结：

`__get__(self, obj, type=None)`中：

self：描述器类M中的实例m
obj：调用描述器的类A中的实例a
type：调用描述器的类A

`__set__(self, obj, value)`中：

value：对这个属性赋值时传入的值



- 同时定义了`__get__`和`__set__`方法的描述器称为**资料描述器**
- 只定义了`__get__`的描述器称为**非资料描述器**
- 二者的区别是：当属性名和描述器名相同时，在访问这个同名属性时，如果是资料描述器就会先访问描述器，如果是非资料描述器就会先访问属性

## type 和 object

在面向对象体系里面，存在两种关系：

- 父子关系，即继承关系，表现为子类继承于父类，使用它的`__bases__`属性可以查看。
- 类型实例关系，表现为某个类型的实例化。使用它的`__class__`属性可以查看，或者使用`type()`函数查看。

**继承关系**使用**实线**从子到父连接，**类型实例关系**使用**虚线**从实例到类型连接：

![python对象图](/Users/hdc/Documents/其他/markdown_pic/python对象图.png)

总结：
第一列，元类列，type是所有元类的父亲。我们可以通过继承type来创建元类。
第二列，TypeObject列，也称类列，object是所有类的父亲，大部份我们直接使用的数据类型都存在这个列的。
第三列，实例列，实例是对象关系链的末端，不能再被子类化和实例化。



## 作用域

###闭包

闭包是由函数和与其相关的引用环境组合而成的实体。

```python
def outside():
    msg = "Outside!"
    def inside():
        print(msg)
    return inside

another = outside()
another()		# Outside!
```

一般情况下，函数中的局部变量仅在函数的执行期间可用，一旦 `outside()` 执行过后，我们会认为 `msg`变量将不再可用。然而，在这里我们发现 `outside` 执行完之后，在调用 `another` 的时候 `msg `变量的值正常输出了，这就是闭包的作用，闭包使得局部变量在函数外被访问成为可能。

###非局部语句（nonlocal）

非局部语句是Python 3.x中新引入的特性，可以让你给外层但非全局作用域中的变量赋值。官方文档中的说法是，非局部语句可以让所列的标识符（identifier）指向**最近**的嵌套作用域（enclosing scope）中已经绑定过的变量，全局变量除外。

1. 如果没有非局部语句

   一般来说，嵌套函数对于其外层作用域中的变量是有访问权限的。

   ```python
   >>> def outside():
           msg = "Outside!"
   
           def inside():
               print(msg)
   
           inside()
           print(msg)
   
   >>> outside()
   Outside!
   Outside!
   ```

   `inside`成功获得了外层作用域中`msg`的值

   ```python
   >>> def outside():
           msg = "Outside!"
           def inside():
               msg = "Inside!"
               print(msg)
           inside()
           print(msg)
   
   >>> outside()
   Inside! # inside函数打印的msg
   Outside! # outside函数打印的msg
   ```

   在`inside`函数中，Python实际上并没有为之前已经创建的`msg`变量赋值，而是在`inside`函数的局部作用域（local scope）中创建了一个名叫`msg`的新变量。这说明：嵌套函数对外层作用域中的变量其实只有只读访问权限。如果我们在这个示例中的`inside`函数的顶部再加一个`print(msg)`语句，那么就会出现`UnboundLocalError: local variable 'msg' referenced before assignment`这个错误。

2. 使用非局部语句之后

   ```python
   >>> def outside():
           msg = "Outside!"
           def inside():
               nonlocal msg
               msg = "Inside!"
               print(msg)
           inside()
           print(msg)
   
   >>> outside()
   Inside!
   Inside!
   ```

   我们在`inside`函数的顶部添加了`nonlocal msg`语句。这个语句的作用，就是告诉Python解释器在碰到为`msg`赋值的语句时，应该向外层作用域的变量赋值，而不是声明一个重名的新变量。这样，两个函数的打印结果就一致了。

   `nonlocal`的用法和`global`非常类似，只是前者针对的是外层函数作用域的变量，后者针对的则是全局作用域的变量。

3. 什么时候该使用非局部语句

   ```python
   > def outside():
           d = {"outside": 1}
           def inside():
               d["inside"] = 2
               print(d)
           inside()
           print(d)
   
   >>> outside()
   {'inside': 2, 'outside': 1}
   {'inside': 2, 'outside': 1}
   ```

   你可能会想，因为没有使用`nonlocal`，`inside`函数中往字典`d`中插入的`"inside": 2`键值对（key-value pair）不会体现在`outside`函数中。你这么想挺合理，但却是错的。因为字典插入并不是赋值操作，而是方法调用（method call）。所以，这个示例中我们可以不使用`nonlocal`，就能直接操作外层作用域中的变量。（操作不改变d的`id()`即可）



