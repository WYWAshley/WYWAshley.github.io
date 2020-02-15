---
layout: post
title: Python3 异常处理
categories: [Python]
description: Decription and usage about Python3 Exception and Assertion
keywords: exception, assertion
---

&emsp;&emsp;本文主要讲了Python3 处理异常的语法，以及如何利用异常处理解决一些实际操作，同时介绍了BaseException类和Assertion。

<img src="/images/posts/Python3-Exception-and-Error/title.png" alt="image-title"  />

## 一、异常处理存在的意义

### 1. 错误

&emsp;&emsp;程序出现错误有两种情况，一种是**语法错误**，Python解释器不能解释该句子，这种错误必须在程序执行前就纠正。另一种**逻辑错误**，可能是由于不完整或者不合法的输入所导致的，也可能是逻辑无法生成、计算、或是输出结果需要的过程无法执行，这些错误分别被称为域错误和范围错误。

### 2. 异常

&emsp;&emsp;当程序出现了错误而在正常控制流以外采取的行为，什么意思呢，就是首先有引起异常发生的错误，然后我们让程序检测并且采取可能的措施。所以说一场就是错误的一种信号，只要解释器检测到了异常条件，它就会通过异常抛出通知当前控制流有错误发生。

&emsp;&emsp;Python用异常对象（exception object）来表示异常，遇到错误之后，会引起异常。如果异常对象并未被处理或者捕捉，程序就会用所谓的回溯（Traceback）终止执行。

&emsp;&emsp;异常处理的意义就是①当异常发生时，能够将异常通知给编程人员或者用户；②使本来已经中断的程序以适当的方式继续运行，或者退出。 并且能够保存用户的当前操作,或者进行数据回稳；③把占用的资源释放掉。

&emsp;&emsp;据说Java的异常处理面试时候经常考，有机会把这个坑填了吧。



## 二、异常处理的基本语法

* 一个 try 语句可能包含多个except子句，分别来处理不同的特定的异常，最多只有一个分支会被执行。

  ```python
  try:
      <执行代码>
  except:
      <发生异常时执行的代码>
  ```

* 处理程序将只针对对应的 try 子句中的异常进行处理，而不是其他的 try 的处理程序中的异常。

  except语句后面如果不指定异常类型，则默认捕获所有异常，你可以通过logging或者sys模块获取当前异常。

  ```python
  try:
      <执行代码>
  except <异常类型，可用元组表示>:
      <发生异常时执行的代码>
  ```

> try 语句按照如下方式工作；
>
> * 首先，执行 try 子句，异常处理并不仅仅处理那些直接发生在 try 子句中的异常，而且还能处理子句中**调用的函数**（甚至间接调用的函数）里抛出的异常。
>
> - 如果没有异常发生，忽略 except 子句，try 子句执行后结束。
> - 如果在执行 try 子句的过程中发生了异常，那么 try 子句余下的部分将被忽略。如果异常的类型和 except 之后的名称相符，那么对应的 except 子句将被执行。
> - 如果一个异常没有与任何的 except 匹配，那么这个异常将会**传递给上层的 try 中**。

* 最后一个except子句可以忽略异常的名称，它将被当作通配符使用。你可以使用这种方法**打印一个错误信息**，然后再次把异常抛出。

  ```python
  try:
      
  except OSError as err:
      
  except ValueError:
      
  except:
      print("Unexpected error:", sys.exc_info()[0])
      raise
  ```

* 使用** else 子句**比把所有的语句都放在 try 子句里面要好，这样可以避免一些**意想不到**，而 except 又无法捕获的异常。

  ```python
  try:
      <执行代码>
  except IOError:
      <发生异常时执行的代码>
  else:
      <没有异常时执行的代码>
  ```

* **finally块**的代码不论程序正常还是异常都会执行到甚至是调用了sys模块的exit函数退出Python环境，finally块都会被执行，因为exit函数实质上是引发了SystemExit异常）

  &emsp;因此finally最适合用来做释放外部资源的操作。等同于通过with关键字指定文件对象的上下文环境并在 离开上下文环境时自动释放文件资源

  ```python
  try:
      <执行代码>
  except IOError:
      <发生异常时执行的代码>
  else:
      <没有异常时执行的代码>
  finally:
      <不管有没有异常都会执行的代码>
  ```

* **抛出异常**

  ```python
  raise [Exception [, args [, traceback]]]
  ```

  &emsp;如果你只想知道这是否抛出了一个异常，并不想去处理它，那么一个简单的 raise 语句就可以再次把它抛出。即捕获异常重复抛出。

  ```python
  >>>try:
          raise NameError('HiThere')
     except NameError:
          print('An exception flew by!')
          raise ##！！不要重复raise e，trace信息会被截断
     
  An exception flew by!
  Traceback (most recent call last):
    File "<stdin>", line 2, in ?
  NameError: HiThere
  ```



## 三、类的继承关系

```python
BaseException
+-- SystemExit // 上面finally模块也提到了
+-- KeyboardInterrupt
+-- GeneratorExit
+-- Exception
+-- StopIteration...
+-- StandardError...
+-- Warning...
```

&emsp;从Exception的层级结构来看，BaseException是最基础的异常类，Exception继承了它。BaseException除了包含所有的Exception外还包含了SystemExit，KeyboardInterrupt和GeneratorExit三个异常。

&emsp;所以，程序在捕获所有异常时**更应该使用Exception**而不是BaseException，因为另外三个异常属于更高级别的异常，合理的做法应该是交给Python的解释器处理。



&emsp;BaseException 速查表：

|         异常名称          |                        描述                        |
| :-----------------------: | :------------------------------------------------: |
|       BaseException       |                   所有异常的基类                   |
|        SystemExit         |                   解释器请求退出                   |
|     KeyboardInterrupt     |             用户中断执行(通常是输入^C)             |
|         Exception         |                   常规错误的基类                   |
|       StopIteration       |                 迭代器没有更多的值                 |
|       GeneratorExit       |        生成器(generator)发生异常来通知退出         |
|       StandardError       |              所有的内建标准异常的基类              |
|      ArithmeticError      |               所有数值计算错误的基类               |
|    FloatingPointError     |                    浮点计算错误                    |
|       OverflowError       |                数值运算超出最大限制                |
|     ZeroDivisionError     |            除(或取模)零 (所有数据类型)             |
|      AssertionError       |                    断言语句失败                    |
|      AttributeError       |                  对象没有这个属性                  |
|         EOFError          |             没有内建输入,到达EOF 标记              |
|     EnvironmentError      |                 操作系统错误的基类                 |
|          IOError          |                 输入/输出操作失败                  |
|          OSError          |                    操作系统错误                    |
|       WindowsError        |                    系统调用失败                    |
|        ImportError        |                 导入模块/对象失败                  |
|        LookupError        |                 无效数据查询的基类                 |
|        IndexError         |              序列中没有此索引(index)               |
|         KeyError          |                  映射中没有这个键                  |
|        MemoryError        |     内存溢出错误(对于Python 解释器不是致命的)      |
|         NameError         |            未声明/初始化对象 (没有属性)            |
|     UnboundLocalError     |               访问未初始化的本地变量               |
|      ReferenceError       | 弱引用(Weak reference)试图访问已经垃圾回收了的对象 |
|       RuntimeError        |                  一般的运行时错误                  |
|    NotImplementedError    |                   尚未实现的方法                   |
|        SyntaxError        |                  Python 语法错误                   |
|     IndentationError      |                      缩进错误                      |
|         TabError          |                   Tab 和空格混用                   |
|        SystemError        |                一般的解释器系统错误                |
|         TypeError         |                  对类型无效的操作                  |
|        ValueError         |                   传入无效的参数                   |
|       UnicodeError        |                 Unicode 相关的错误                 |
|    UnicodeDecodeError     |                Unicode 解码时的错误                |
|    UnicodeEncodeError     |                 Unicode 编码时错误                 |
|   UnicodeTranslateError   |                 Unicode 转换时错误                 |
|          Warning          |                     警告的基类                     |
|    DeprecationWarning     |               关于被弃用的特征的警告               |
|       FutureWarning       |           关于构造将来语义会有改变的警告           |
|      OverflowWarning      |        旧的关于自动提升为长整型(long)的警告        |
| PendingDeprecationWarning |              关于特性将会被废弃的警告              |
|      RuntimeWarning       |      可疑的运行时行为(runtime behavior)的警告      |
|       SyntaxWarning       |                  可疑的语法的警告                  |
|        UserWarning        |                 用户代码生成的警告                 |



### 四、用户自定义异常

&emsp;&emsp;当创建一个模块有可能抛出多种不同的异常时，一种通常的做法是为这个包建立一个基础异常类，然后基于这个基础类为不同的错误情况创建不同的子类

```python
class Error(Exception):
    """Base class for exceptions in this module."""
    pass
 
class InputError(Error):
    """Exception raised for errors in the input.
 
    Attributes:
        expression -- input expression in which the error occurred
        message -- explanation of the error
    """
 
    def __init__(self, expression, message):
        self.expression = expression
        self.message = message
```



### 五、 断言

&emsp;&emsp;语法格式如下：

```python
assert expression
# 等价于
if not expression:
    raise AssertionError
# assertion后跟参数
assert 1==2, '1 不等于 2'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError: 1 不等于 2
```



参考资料：

1. 《[[python中常见的一些错误异常类型](https://www.cnblogs.com/pychina/p/10238978.html)》
2. 《[一文掌握 Python 异常处理的所有知识点](https://juejin.im/entry/5a713192518825732a6dce43)》
3. 《[Python3 错误和异常](https://www.runoob.com/python3/python3-errors-execptions.html)》
4. 《[Python3 assert](https://www.runoob.com/python3/python3-assert.html)》

