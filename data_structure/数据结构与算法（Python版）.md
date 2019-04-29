# 数据结构与算法（Python版）

[TOC]

## 数据结构

数据结构可以分为逻辑结构和物理结构两类。

逻辑结构：数据对象中数据元素之间的互相关系。
1. 集合结构：数据元素除了同属于一个集合外，之间没有其他关系。
2. 线性结构：数据元素间是一对一的关系。
3. 树形结构：数据元素间存在一对多的层次关系。
4. 图形结构：数据元素是多对多的关系。

物理结构：数据的逻辑结构在计算机中的存储形式。
1. 顺序存储结构：把数据结构存放在地址连续的存储单元里，数据间的逻辑关系和物理关系是一致的。
2. 链式存储结构：把数据元素存放在任意的存储单元里，可以连续，也可以不连续，用一个指针存放数据元素的地址，存储关系不能反映其逻辑关系。

抽象数据类型（ADT）：一个数学模型及定义在该模型上的一组操作。抽象数据模型的定义仅取决于它的一组逻辑特性，而与其在计算机内部如何表示和实现无关。

### 物理结构
#### 数组（array-based sequence）

python内置的序列类有`list、tuple、str`。

#### 链表（linked list）

存储数据元素的域称为数据域；存储直接后继结点存储位置的域称为指针域。这两部分组成的存储映像称为`结点（node）`。n个结点链结成一个链表。
列表的第一个结点称为链表的`头（head）`，最后一个结点称为链表的`尾（tail）`。

*  单链表（singly linked list）：每个结点存储一个元素的引用，最后一个元素指向`None`。
*  循环链表（circularly linked list）：最后一个元素的`next`指向`head`，就形成了一个环。
*  双向链表（doubly linked list）：
*  位置列表（the positional list）

#### 链表与数组实现的序列比较

### 栈（stack）

栈：限定仅在一端进行插入或删除操作的线性表，这一段称为`栈顶（top）`，另一端称为`栈底（bottom）`。栈的修改按照`后进先出（LIFO：last in  first out）`原则。

#### 栈的抽象数据类型

    stack()                     创建一个空的栈

    push(item)              O(1)添加元素到栈顶   
    pop()                   O(1)删除栈顶元素，并返回它
    
    top()                   O(1)返回栈顶元素，但不是删除它
    is_empty()              O(1)测试是否为空栈
    size()                  O(1)返回栈里元素个数

| operation    | bottom s top | return value |
| ------------ | ------------ | ------------ |
| s.push(1)    | [1]          | -            |
| s.push(2)    | [1, 2]       | -            |
| s.pop        | [1]          | 2            |
| s.top()      | [1]          | 1            |
| s.is_empty() | [1]          | False        |
| s.size()     | [1]          | 1            |

#### 实现栈

    用list来实现：
    stack method                realization with python list
    s.push(item)                l.append(item)
    s.pop()                     l.pop()
    s.top()                     l[-1]
    s.is_empty()                len(l) == q
    s.size()                    len(l)

定义stack类：

```
class stack:

	def __init__(self):
		self._data = []

	def is_empty(self):
		return self._data == []

	def push(self, item):
		self._data.append(item)

	def pop(self):
		return self._data.pop()

	def top(self):
		if self.is_empty():
			raise Empty('stack is empty')
		return self._data[-1]

	def size(self):
		if self.is_empty():
			raise Empty('stack is empty')
		return len(self._data)
```

#### 栈的应用

符号匹配：

```
def is_matched(expr):

	lefty = '({['
	righty = ')}]'
	S = stack()

	for c in expr:
		if c in lefty:
			S.push(c) 
		elif c in righty:
			if S.is_empty():
				return False 
			if righty.index(c) != lefty.index(s.pop()):
				return False 
	
	return s.is_empty()
```

十进制转换为二进制：

后缀表达式求值：

### 队列（queue）

队列只允许在一端进行插入，另一端进行删除元素的线性表。允许插入的一端叫做`队尾（rear）`，删除的一端称为`队头（front）`。队列的修改原则为`先进先出（FIFO：first in  first out）`。

#### 队列的抽象数据类型

    queue()                     创建一个空的队列

    enqueue(item)           O(1)添加元素到队尾   
    dequeue()               O(1)删除队头元素，并返回它
    
    front()                 O(1)返回队头元素，但不是删除它
    is_empty()              O(1)测试是否为空队列
    size()                  O(1)返回队列元素个数

| operation    | front q rear | return value |
| ------------ | ------------ | ------------ |
| q.enqueue(1) | [1]          | -            |
| q.enqueue(2) | [1, 2]       | -            |
| q.dequeue    | [2]          | 1            |
| q.front()    | [2]          | 2            |
| q.is_empty() | [2]          | False        |
| q.size()     | [2]          | 1            |

#### 实现队列

```
class queue:

	DEFAULT_CAPACITY = 10

	def init (self):
		self._data = [None] * queue.DEFAULT_CAPACITY 
		self._size = 0 
		self._front = 0		#队头的索引

	def is_empty(self):
		return self._size == 0

	def dequeue(self):
		if self.is_empty():
			raise Empty('Queue is empty') 
		answer = self._data[self._front] 
		self._data[self._front] = None 
		self._front = (self._front + 1) % len(self._data) 
		self._size −= 1 
		return answer

	def enqueue(self, item):
		if self._size == len(self._data):
			self._resize(2 * len(self.data))
		avail = (self._front + self._size) % len(self._data) 
		self._data[avail] = item
		self._size += 1

	def _resize(self, cap): 	#扩容
		old = self._data
		self._data = [None] * cap
		walk = self._front
		for k in range(self._size):
			self._data[k] = old[walk] 
			walk = (1 + walk) % len(old)
		self._front = 0

	def front(self):
	 	if self.is_empty():
			raise Empty('Queue is empty') 
		return self._data[self._front]

	def size(self):
		return self._size
```

#### 队列的应用

模拟：约瑟夫问题（一群人围成一个圈，依次报数，数字到特定num时，移除这个人，游戏继续，直到剩最后一个人）：

```
def Josephus(namelist, num):

	simqueue = queue()
	for name in namelist:
		simqueue.enqueue(name)

	while simqueue.size() > 1:
		for i in range(num):
			simqueue.enqueue(simqueue.dequeue())
		simqueue.dequeue()
		
	return simqueue.dequeue()
```

模拟：打印机


### 双端队列（deques）

双端队列（double-ended queue）是允许在两端端进行插入，删除元素的线性表。

#### 双端队列的抽象数据类型

    deque()                     创建一个空的队列

    add_front(item)         O(1)添加元素到deque的front   
    add_rear(item)          O(1)添加元素到deque的rear
    remove_front()          O(1)删除front元素，并返回它
    remove_rear()           O(1)删除rear元素，并返回它
    
    front()                 O(1)返回front元素，但不是删除它
    rear()                  O(1)返回rear元素，但不是删除它
    is_empty()              O(1)测试是否为空的双端队列
    size()                  O(1)返回双端队列元素个数

| operation        | front d rear | return value |
| ---------------- | ------------ | ------------ |
| d.add_front(1)   | [1]          | -            |
| d.add_rear(2)    | [1, 2]       | -            |
| d.add_front(3)   | [3, 1, 2]    | -            |
| d.add_rear(4)    | [3, 1, 2, 4] | -            |
| d.remove_front() | [1, 2, 4]    | 3            |
| d.remove_rear()  | [1, 2]       | 4            |
| q.is_empty()     | [1, 2]       | False        |
| q.size()         | [1, 2]       | 2            |

#### Python中的双端队列

Python中的collections模块中包含duque

| deque             | collections.deque | description                        |
| ----------------- | ----------------- | ---------------------------------- |
| d.add_front(item) | d.appendleft()    | add to beginning                   |
| d.add_rear(item)  | d.append()        | add to end                         |
| d.remove_front()  | d.popleft()       | remove from beginning              |
| d.remove_rear()   | d.pop()           | remove from end                    |
| d.front()         | d[0]              | access front element               |
| d.rear()          | d[-1]             | access rear element                |
| d.size()          | len(d)            | number of elements                 |
|                   | d[j]              | access arbitrary entry by index    |
|                   | d[j] = val        | modify arbitrary entry by index    |
|                   | d.clear()         | clear all contents                 |
|                   | d.rotate(k)       | circularly shift rightward k steps |
|                   | d.remove(item)    | remove first matching element      |
|                   | d.count(item)     | count number of matches for item   |

#### 双端队列的应用

回文检查：

```
def palchecker(astring):

	chardeque = deque()

	for ch in astring:
		chardeque.add_rear(ch)

	stillequal = True

	while len(chardeque) > 1 and stillequal:
		if chardeque.remove_front() != chardeque.remove_rear():
			stillequal = False

	return stillequal
```

### 树（tree）

树的定义

定义一：
树由一组结点和一组连接结点的边组成。树具有以下属性：
1. 树的一个结点被指定为`根结点`
2. 除了根结点以外，每个结点n唯一的与另一个结点p通过边连接，其中p是n的父结点
3. 从根结点到每个结点的的路径唯一
4. 如果树的每个结点最多只有两个子结点，我们就说该树为二叉树

定义二：
树为n个结点的有限集，n为0时为空树，在任意非空树中：
1. 有且仅有一个特定的称为`根（root）`的结点
2. 其余结点由0个或者多个`子树（subtree）`组成，子树也是一棵树。

结点分类：
结点的`度（degree）`：结点拥有的子树数
树的度：树内个结点的度的最大值
`叶结点（leaf）`：度为0的结点

结点间关系：
`孩子（child）`、`双亲（parent）`：结点的子树的根称为该结点的孩子，相应的该结点称为孩子的双亲
`兄弟（sibling）`：同一双亲的孩子之间互称为兄弟
结点的`祖先（ancestor）`：根到该结点所经分支上的所有结点。
结点的`子孙（descendant）`：以该结点为根的子树中的所有结点。

树的其他概念：
结点的`层次（level）`：根到该结点所经过的分支数目
树的深度或`高度（depth）`：树中结点的最大层次。
`森林（forest）`：m棵互不相交的树的集合

#### 树的抽象数据类型

    所有树的结点抽象成position，p.element()返回存储在位置p的元素时：
    t.root()                    返回根结点的位置，如果为空树返回None
    
    t.is_root(p)                p结点为根结点时，返回True  
    t.parent(p)                 返回p结点的父结点的位置，p为根结点时返回None 
    t.num_children(p)           返回p结点的子结点数
    t.children(p)               生成一个p结点的子结点的迭代器
    t.is_leaf(p)                p结点为叶子结点时返回True
    
    t.depth(p)                  返回p结点的层次
    t.height(p)                 返回以p结点为根结点的子树的高度
    
    len(t)                      返回树中结点个数 
    t.is_empty()                结点数为0时，返回True
    t.positions()               生成一个树t位置的迭代器
    iter(t)                     生成一个树t结点的迭代器

#### 实现树类

```
class Tree:

	class Position:

		def element(self):
			raise NotImplementedError('must be implemented by subclass')

		def __eq__(self, other):
			raise NotImplementedError('must be implemented by subclass')

		def __ne__(self, other):
		 	return not (self == other) 

		def root(self):
			raise NotImplementedError('must be implemented by subclass')

		def parent(self, p):
			raise NotImplementedError('must be implemented by subclass')

		def num_children(self, p):
			raise NotImplementedError('must be implemented by subclass')

		def children(self, p):
			raise NotImplementedError('must be implemented by subclass')

		def __len__(self):
			raise NotImplementedError('must be implemented by subclass')

		def is_root(self, p):
			return self.root( ) == p

		def is_leaf(self, p):
			return self.num_children(p) == 0

		def is_empty(self):
			return len(self) == 0

		def depth(self, p):
			if self.is_root(p):
				return 0 
			else:
				return 1 + self.depth(self.parent(p))

		def _height2(self, p): 
			if self.is_leaf(p):
				return 0 
			else:
				return 1 + max(self._height2(c) for c in self.children(p))

		def height(self, p=None):
		 	if p is None:
				p = self.root() 
			return self._height2(p)
```

#### 二叉树

二叉树的作为树的子类，另外支持三个ADT

    t.left(p)                   返回p结点左孩子的位置，如果没有返回None 
    t.right(p)                  返回p结点右孩子的位置，如果没有返回None
    t.sibling(p)                返回p结点的兄弟的位置，如果没有返回None

```
class BinaryTree(Tree):

	def left(self, p):
		raise NotImplementedError('must be implemented by subclass') 

	def right(self, p):
	 	raise NotImplementedError('must be implemented by subclass') 
	 
	def sibling(self, p):
		parent = self.parent(p) 
		if parent is None: 
			return None
		else:
			if p == self.left(parent):
				return self.right(parent)
			else:
				return self.left(parent)
	
	def children(self, p):
		if self.left(p) is not None:
			yield self.left(p) 
		if self.right(p) is not None:
			yield self.right(p)
```

树的实现一般使用链式结构。更新链式二叉树的操作函数：

    t.add_root(p)               为一个空树创建根结点，将e作为根结点的值，返回根结点的位置，如果树非空，则出现错误
    t.add_left(p, e)            将e作为p的左孩子结点，返回e结点的位置，如果p已经有左孩子结点，则发生错误
    t.add_right(p, e)           将e作为p的右孩子结点，返回e结点的位置，如果p已经有右孩子结点，则发生错误
    t.replace(p, e)             将p结点位置的值替换为e，返回替换前的值
    t.delete(p)                 删除p结点，用它的子结点替换它，返回结点的值，如果有两个子结点，则发生错误
    t.attach(p, t1, t2)         将t1，t2子树作为p结点的左子树，右子树，如果p不是叶子结点，则发生错误

用链式结构实现二叉树：

```
class LinkedBinaryTree(BinaryTree):

	class Node:

		__slots__= '_element', '_parent', '_left', '_right'
		def __init__(self, element, parent=None, left=None, right=None):
			self._element = element 
			self._parent = parent 
			self._left = left 
			self._right = right

		class Position(BinaryTree.Position):

			def __init__(self, container, node):
				self._container = container 
				self._node = node

			def element(self):
				return self._node._element

			def __eq__(self, other):
				return type(other) is type(self) and other._node is self._node

		def validate(self, p):

			if not isinstance(p, self.Position):
				raise TypeError('p must be proper Position type') 
			if p._container is not self:
				raise ValueError('p does not belong to this container')
			if p._node._parent is p._node:
				raise ValueError('p is no longer valid') 
			return p._node

		def _make_position(self, node):
			return self.Position(self, node) if node is not None else None

		
		def __init__(self):

			self._root = None 
			self._size = 0

		def __len__(self):
			return self._size

		def root(self):
			return self._make_position(self._root)

		def parent(self, p):

			node = self._validate(p) 
			return self._make_position(node._parent)

		def left(self, p):

			node = self._validate(p) 
			return self._make_position(node._left)

		def right(self, p):

			node = self._validate(p) 
			return self._make_position(node._right)

		def num_children(self, p):

			node = self._validate(p) 
			count = 0 
			if node._left is not None:
				count += 1 
			if node._right is not None:
				count += 1 
			return count

		
		def _add_root(self, e):

			if self._root is not None: 
					raise ValueError('Root exists') 
			self._size = 1 
			self._root = self._Node(e) 
			return self._make_position(self._root)

		def _add_left(self, p, e):
		
			node = self._validate(p) 
			if node._left is not None: 
				raise ValueError('Left child exists') 
			self._size += 1 
			node._left = self._Node(e, node)
			return self._make_position(self._left)

		def _add_right(self, p, e):

			node = self._validate(p) 
			if node._right is not None: 
				raise ValueError('Right child exists') 
			self._size += 1 
			node._right = self._Node(e, node)
			return self._make position(node._right)

		def _replace(self, p, e):

			node = self._validate(p) 
			old = node._element 
			node. _element = e 
			return old


		def _delete(self, p):

			node = self._validate(p) 
			if self.num children(p) == 2: 
				raise ValueError('p has two children') 
			child = node._left if node._left else node._right 
			if child is not None:
				child._parent = node._parent 
			if node is self._root:
				self._root = child 
			else:
				parent = node._parent
				if node is parent._left:
					parent._left = child
				else:
					parent._right = child 
			self._size −= 1 
			node._parent = node 
			return node._element

		def _attach(self, p, t1, t2):

			node = self._validate(p) 
			if not self.is_leaf(p): 
				raise ValueError('position must be leaf') 
			if not type(self) is type(t1) is type(t2): 
				raise TypeError( Tree types must match ) 
			self._size += len(t1) + len(t2) 
			if not t1.is_empty(): 
				t1._root._parent = node 
				node._left = t1._root 
				t1._root = None 
				t1._size = 0 
			if not t2.is_empty():
				t2._root._parent = node 
				node._right = t2._root 
				t2._root = None
				t2._size = 0
```

#### 树的遍历

前序遍历：先访问根结点，然后前序遍历左子树，再前序遍历右子树。
中序遍历：先中序遍历左子树，然后访问根结点，再中序遍历右子树。
后序遍历：先后续遍历左子树，然后后续遍历右子树，再访问根结点。
层序遍历：从上到下逐层遍历，在同一层中，从左到右对结点逐个访问。

实现二叉树的前序（preorder）、中序（inorder）、后序（postorder）遍历有两个常用的方法：
一是递归(recursive)，
二是使用栈实现的迭代版本(stack+iterative)。
这两种方法都是O(n)的空间复杂度。

```python
#recursive

def preorder(tree):
    if tree:
        print(tree.getRootVal())
        preorder(tree.getLeftChild())
        preorder(tree.getRightChild())
   
def inorder(tree):
    if tree:
        inorder(tree.getLeftChild())
        print(tree.getRootVal())
        inorder(tree.getRightChild())
        
def postorder(tree):
    if tree:
        postorder(tree.getLeftChild())
        postorder(tree.getRightChild())
        print(tree.getRootVal())
```

```python
#stack+iterative

def preorderTraversal(self, root):
   ans, stack = [], [root]
   while stack:
       node = stack.pop()
       if node:
           ans.append(node.val)
           stack.append(node.right)
           stack.append(node.left)
   return ans

def inorderTraversal(self, root):
    ans, stack = [], []
    while True:
        while root:
            stack.append(root)
            root = root.left
        if not stack:
            return ans
        node = stack.pop()
        ans.append(node.val)
        root = node.right
        
def postorderTraversal(self, root):
    ans, stack = [], [root]
    while stack:
        node = stack.pop()
        if node:
            ans.append(node.val)
            stack.append(node.left)
            stack.append(node.right)
    return ans[::-1]
```

```       python
def levelOrder(self, root):
    ans, level = [], [root]
    while root and level:
        ans.append([node.val for node in level])
        level = [kid for n in level for kid in (n.left, n.right) if kid]
    return ans
```

### 堆（heap）

堆通常是一个可以被看做一棵树的数组对象（索引从1开始）。堆总是满足下列性质：
1. 堆中某个结点的值总是不大于或不小于其父结点的值；
2. 堆总是一棵完全二叉树。
  将根结点最大的堆叫做最大堆，根结点最小的堆叫做最小堆。
  完全二叉树，结点`p`的左子节点为`2p`，右子节点为`2p+1`。

#### 堆的抽象数据类型

    BinaryHeap()        创建新的二叉堆
    insert(k)           添加新项
    findMin()           返回键值最小的项
    delMin()            返回键值最小的项，并删除
    isEmpty()           堆是否为空
    size()              堆中的项数
    buildHeap(list)     用所给列表创建一新的堆

#### 堆的实现
实现最小堆

```
class BinHeap:
	def __init__(self):
		self.heapList = [0]
		self.currentSize = 0

	def perUp(self, i):
		while i // 2 > 0:
			if self.heapList[i] < self.heapList[i // 2]:#小于父节点
				self.heapList[i], self.heapList[i // 2] = self.heapList[i // 2], self.heapList[i]
			i //= 2

	def insert(self, k):#将k放在heapList最后，与父节点比较，小于父节点则交换，直到根结点
		self.heapList.append(k)
		self.currentSize += 1
		self.percUp(self.currentSize)

	def percDown(self, i):
		while i * 2 < self.currentSize:
			mc = self.minChild(i)
			if self.heapList[i] > self.heapList[mc]:#大于最小子节点
				self.heapList[i], self.heapList[mc] = self.heapList[mc], self.heapList[i]
			i = mc

	def minChild(self, i):#寻找i结点的最小子结点
		if i * 2 + 1 > self.currentSize:
			return i * 2
		else:
			return i * 2 if self.heapList[i * 2] < self.heapList[i * 2 + 1] else i * 2 + 1
	
	def delMin(self):
		ans = self.heapList[1]#根结点最小，用以返回结果
		self.heapList[1] = self.heapList[-1]#将列表最后一项移至根结点，依次与最小子结点比较，直到叶子结点
		self.currentSize -= 1
		self.heapList.pop()
		self.percDown(1)
		return ans

	def builtHeap(self, list):#从堆的叶子节点的上一次开始操作
		i = len(list) // 2
		self.currentSize = len(list)
		self.heapList = [0] + list[:]
		while i > 0:
			self.percDown(i)
			i -= 1
```

### 优先队列（priority queue）

在优先队列中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出 （first in, largest out）的行为特征。

#### 优先队列的抽象数据类型

    P.add(k, v)
    P.min()             return (k,v)
    P.remove_min()      return (k,v)
    P.is_empty()
    len(P)

#### 优先队列的实现
使用堆来实现

### 搜索树（search tree）

二叉搜索树中的关键字总是以满足二叉搜索树性质的方式来存储：
x为二叉树的一个结点，如果`y是左子树`中的一个结点，那么`y.key ≤ x.key`；如果`y是右子树`中的一个结点，那么`y.key ≥ x.key`。
中序遍历搜索树后，是一个已排序的列表。

#### map的抽象数据类型

使用二叉搜索树来实现map

    map()               创建空的map
    put(key, val)       添加新的新的键值对
    get(key)            
    del map[key]
    len()
    in                  key在map中时，返回True

#### 搜索树的实现

```
class TreeNode:
	
	def __init__(self, key, val, left = None, right = None, parent = None):
		self.key = key
		self.val = val
		self.leftChild = left
		self.rightChild = right
		self.parent = parent

	def hasLeftChild(self):
		return self.leftChild

	def hasRightChild(self):
		return self.rightChild

	def isLeftChild(self):
		return self.parent and self.parent.leftChild == self

	def isRightChild(self):
		return self.parent and self.parent.rightChild == self
	
	def isRoot(self):
		return not self.parent

	def isLeaf(self):
		return not self.leftChild and not self.rightChild

	def hasAnyChildren(self):
		return self.leftChild or self.rightChild

	def hasBothChildren(self):
		return self.leftChild and self.rightChild

	def replaceNodeData(self, key, val, lc, rc):
		self.key = key
		self.val = val
		self.leftChild = lc
		self.rightChild = rc
		if self.hasLeftChild():
			self.leftChild.parent = self
		if self.hasRightChild():
			self.rightChild.parent = self


class BinarySearchTree:
	def __init__(self):
		self.root = None
		self.size = 0

	def __len__(self):
		return self.size


	def put(self, key, val):
		parentNode = None
		currentNode = self.root
		while currentNode:
			x = currentNode
			currentNode = currentNode.leftChild if key < currentNode.key else currentNode.rightChild
		self.parent = parentNode
		if parentNode == None:
			self.root = TreeNode(key, val)
		elif val < x.val:
			parentNode.leftChild = TreeNode(key, val, parent = parentNode)
		else:
			parentNode.rightChild = TreeNode(key, val, parent = parentNode)

	def __setitem__(self, k, v):
		self.put(k, v)


	def get(self, key):
		currentNode = self.root
		while currentNode and currentNode.key != key:
			currentNode = currentNode.leftChild if key < currentNode.key else currentNode.rightChild
		return currentNode.val if currentNode and currentNode.key == key else None

	def __getitem__(self, key):
		return self.get(key)

	def __contauins__(self, key):
		if self.get(key) != None:
			return True
		else:
			return False


	def delete(self, key):
		Node2Move = self.get(key)
		if Node2Move = None:
			raise KeyError('Error, key not in tree')
		elif Node2Move.leftChild == None:
			self.transplant(Node2Move, Node2Move.rightChild)
		elif Node2Move.rightChild == None:
			self.transplant(Node2Move, Node2Move.leftChild)
		else:
			successorNode = Node2Move.findSuccessor()
			if successorNode.parent != 	Node2Move:
				self.transplant(successorNode, successorNode.rightChild)#后继结点没有左孩子
				successorNode.rightChild = Node2Move.rightChild
				successorNode.rightChild.parent = successorNode
			self.transplant(Node2Move, successorNode)
			successorNode.leftChild = Node2Move.leftChild
			successorNode.leftChild.parent = successorNode
		
	def transplant(self, u, v):#用以v为根的子树替换以u为根的子树
		if u.parent == None:
			self.root = v
		elif u == u.parent.leftChild:
			u.parent.leftChild = v
		elif:
			u.parent.rightChild = v
		if v:
			v.parent = u.parent

	def findSuccessor(self):#key in the tree
		currentNode = self.rightChild
		while currentNode.leftChild:
			currentNode = currentNode.leftChild
		return currentNode

	def __delitem__(self, key):
		self.delete(key)
```

### 图（graph）

## 算法

### 递归（recursion）

### 查找（search）

#### 顺序查找

顺序查找（sequential search）：从表中的第一个（或最后一个）记录开始，逐个进行记录的关键字和给定值比较，若相等，则查找成功找到查找记录，都不匹配时，查找失败。

```
def sequentialSearch(alist, item):  #无序列表
    pos = 0
    
    while pos < len(alist):
        if alist[pos] == item:
            return True
        else:
            pos += 1      
    return False
```

#### 二分查找

二分查找（binary search）：先确定待查记录所在的范围，然后逐步缩小范围直到找到或找不到该记录为止。（有序列表）
`mid = (left + right)/2`，可以转换为`mid = left + 1/2*(right - left)`，`1/2`系数可以变换，如插值查找（`(key - a[left])/(a[right] - a[left])`），斐波那契查找（`mid = left + F(k - 1) - 1`）。

```python
def binarySearch(alist, item):
    left, right = 0, len(alist) - 1
    
    while left <= right:
        mid = (left + right) // 2
        if alist[mid] == item:
            return True
        elif target < nums[mid]:
            right = mid - 1
        else:
            left = mid + 1
    return False
```

#### Hash查找



```

```

### 排序（sort）

稳定性：两个排序关键字相等时，排序后，位置不变，则排序方法是稳定的。

#### 冒泡排序

冒泡排序（bubble sort）：两两比较相邻的关键字，如果反序则交换，直到没有反序的记录为止。
短冒泡排序：发现列表已经排序，提前停止。

```
def bubbleSort(alist):
	for i in range(len(alist) - 1, 0, -1):
		for j in range(i):
			if alist[j] > alist[j + 1]:
				alist[j], alist[j + 1] = alist[j + 1], alist[j]
				

def shortBubbleSort(alist):
	exchanges = True
	i = len(alist)

	while i > 0 and exchanges:
		exchanges = False
		for j in range(i):
			if alist[j] > alist[j + 1]:
				exchanges = True
				alist[j], alist[j + 1] = alist[j + 1], alist[j]
		i -= 1
```

#### 选择排序

选择排序（selection sort）：通过`n - i`次关键字间的比较，从`n - i + 1`个记录中选出关键字最小的记录，并和第`i`个记录交换。

```
def selectionSort(alist):
	for i in range(len(alist) - 1, 1, -1):
		positionOfMax = 0
		for j in range(1, i + 1):
			if alist[i] > alist[positionOfMax]:
				positionOfMax = i
		alist[i], alist[positionOfMax] = alist[positionOfMax], alist[i]
```

#### 插入排序

插入排序（insertion sort）：将一个记录插入到已经排好序的有序表中，从而得到一个新的、记录数增1的有序表。

```
def insertionSort(alist):
	for i in range(1, len(alist)):
		currentvalue = alist[i]
		position = i

		while position > 0 and alist[position - 1] > currentvalue:
			alist[position] = alist[position - 1]
			position -= 1

		alist[position] = currentvalue
```

#### 希尔排序

希尔排序（shell's sort）：把记录按下标的一定增量分组，对每组使用直接插入排序算法排序；当增量减至1时，算法便终止。

```
def shellSort(alist):	#增量第一次为n/2,下一次为n/4，直到为1
	sublistCount = len(alist) // 2
	while sublistCount > 0:
		for i in range(sublistCount):
			gapInsertionSort(alist, i, sublistCount)
		sublistCount //= 2

	def gapInsertionSort(alist, start, gap):
		for i in range(start + gap, len(alist), gap):
			currentValue = alist[i]
			position = i

			while position > start and alist[position - gap] > currentValue:
				alist[position] = alist[position - gap]
				position -= gap
			alist[position] = currentValue
```

#### 归并排序

归并排序（merging sort）：假设初始序列含有n个记录，则可以看成是n个有序的子序列，每个子序列的长度为1，然后两两归并，得到floor（n）个长度为2或为1的有序子序列；再两两归并，直到得到长度为n的有序序列。

```
def mergeSort(alist):
	n = len(alist)

	if n < 2:
		return
	                                   #divide
	mid = n // 2                       
	leftHalf = alist[:mid]
	rightHalf = alist[mid:]
	                                   #conquer
	mergeSort(leftHalf)                
	mergeSort(rightHalf)
	                                   #merge results
	merge(leftHalf, rightHalf, alist)  

def merge(s1, s2, s)
	i, j = 0, 0
	while i + j < len(s):
		if j == len(s2) or (i < len(s1) and s1[i] < s2[j]):
			s[i + j] = s1[i]
			i += 1
		else:
			s[i + j] = s2[j]
			j += 1
```

#### 快速排序

快速排序（quick sort）：通过一趟排序将待排序记录分割成独立的两部分，其中一部分记录的记录的关键字均比另一部分记录的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序的目的。帮助拆分列表的关键字称为`枢轴（pivot）`

```
def quickSort(alist):
	n = len(alist)
	if n < 2:
		return
	
	pivot = partition(alist, 0, len(alist) - 1)

	quickSort(alist[:pivot])
	quickSort(alist[pivot + 1:])
	
def partition(alist, low, hight):
	pivotValue = alist[low]
	i, j = low + 1, hight
	while i < j:
		while i < j and alist[i] <= pivotValue:
			i += 1
		while i < j and pivotValue >= alist[j]:
			j -= 1
		alist[i], alist[j] = alist[i], alist[j]
	alist[i], alist[low] = alist[low], alist[i]
	return i
```

#### 排序算法分析

    算法名称              时间复杂度               空间复杂度     稳定性
               最佳         平均        最差         最差
    冒泡排序    O(n)        O(n^2)      O(n^2)      O(1)        稳定
    选择排序    O(n^2)      O(n^2)      O(n^2)      O(1)        稳定    
    插入排序    O(n)        O(n^2)      O(n^2)      O(1)        稳定
    希尔排序    O(nlogn)    O(n^2)      O(n^2)      O(1)        不稳定
    归并排序    O(nlogn)    O(nlogn)    O(nlogn)    O(n)        稳定
    堆排序      O(nlogn)    O(nlogn)    O(nlogn)    O(1)        不稳定
    快速排序    O(nlogn)    O(nlogn)    O(n^2)      O(nlogn)    不稳定

