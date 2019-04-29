# SICP

[TOC]

## scheme

### scheme语法

`前缀表达式`

`正则序求值`：完全展开而后归约。

这种求值模型是先不求出运算对象的值，直到实际需要它们的值时再去做。首先用运算对象表达式去替换形式参数，直到得到一个只包含基本运算符的表达式，然后在去执行求值。

`应用序求值`：先求值参数而后应用。

解释器将先对组合式的各个元素求值，即将过程体中的每一个形参用实参代替，然后在对这一过程体求值。

**定义变量**：

(define var value)

例：

```scheme
(define size 2)
```

**过程（函数）定义**：

(define (⟨name⟩ ⟨formal parameters⟩) ⟨body⟩)

例：

```scheme
(define (square x) (* x x))
```

**条件表达式**：

(cond (⟨p_1⟩ ⟨e_1⟩)
​     	   (⟨p_2⟩ ⟨e_2⟩)
​     		 …
​      	   (⟨p_n⟩ ⟨e_n⟩))

例：

```scheme
(define (abs x)
  (cond ((> x 0) x)
        ((= x 0) 0)
        ((< x 0) (- x))))
```

**if表达式**：

(if ⟨predicate⟩ ⟨consequent⟩ ⟨alternative⟩)

例：

```scheme
(define (abs x)
  (if (< x 0)
      (- x)
      x))
```

**逻辑运算符**：

(and ⟨e_1⟩ … ⟨e_n⟩)

(or ⟨e_1⟩ … ⟨e_n⟩)

(not ⟨e⟩)

**let表达式**：

(let ((⟨var_1⟩ ⟨exp_1⟩)
​      (⟨var_2⟩ ⟨exp_2⟩)
​      		…
​      (⟨var_n⟩ ⟨exp_n⟩))
  ⟨body⟩)

**lambda表达式**：

(lambda  ⟨formal parameters⟩ ⟨body⟩)


### 其他

运行scheme程序

`scheme < test`

```scheme
define (sqrt-iter guess x)
  (if (good-enough? guess x)
      guess
      (sqrt-iter (improve guess x) x)))
```

##数学
