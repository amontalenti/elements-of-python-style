# Python写作指南 #
> 本文基于[The Elements of Python Style](https://github.com/amontalenti/elements-of-python-style/blob/master/README.md#paradigms-and-patterns)翻译润色，才疏学浅难免有不足之处，如有错漏之处恳请指教。

> 译者 [@williamfzc](https://github.com/williamfzc)  
> 错漏及建议欢迎提交issue联系我

这篇文章基于PEP8，覆盖语法、模块布局、范式和架构等多个方面，介绍了一些个人认为比较好的python编写风格。

原文介绍：  
This document goes beyond PEP8 to cover the core of what I think of as great Python style. It is opinionated, but not too opinionated. It goes beyond mere issues of syntax and module layout, and into areas of paradigm, organization, and architecture. I hope it can be a kind of condensed ["Strunk & White"][strunk-white] for Python code.

[strunk-white]: https://en.wikipedia.org/wiki/The_Elements_of_Style

# 目录 

  * [Python写作指南](#Python写作指南)
    * [遵从大部分PEP8规范](#遵从大部分PEP8规范)
    * [关于每行代码的最大长度](#关于每行代码的最大长度)
    * [命名的一致性](#命名的一致性)
    * [不值得钻牛角尖的一些点](#不值得钻牛角尖的一些点)
    * [写好docstring](#写好docstring)
    * [范式 & 设计模式](#范式-设计模式)
    * [“python之禅”在你代码中的具体应用](#“python之禅”在你代码中的具体应用)
    * [一些无关好坏的主观比较](#一些无关好坏的主观比较)
    * [有用的库与项目结构](#有用的库与项目结构)
    * [灵感来源](#灵感来源)
    * [编著者](#编著者)

## 遵从大部分PEP8规范

PEP8涵盖了大部分用户最常用的使用内容，例如空格、函数/类/方法之间的换行符、模块的导入、针对不再被推荐的功能的警告等等，且将他们管理地非常好。

在PEP8准则的实践上，有一套非常好的辅助工具flake8，能够帮助你检测出python代码中的语法错误与不足之处。

PEP8是指一套设计准则，理论上来说他不是一套需要百分百严格执行的规定。

而遵守唯一的准则，很容易在没有明确说明的地方引起人们的争议。本文将针对这些争议点，提出个人认为比较好的解决方案，供各位参考。

## 关于每行代码的最大长度

关于flake8中严格的“每行代码不能超过79个字符”的限制，当他偶尔影响了你的正常使用，无需为此感到苦恼，请无视他。但大多数情况下，如果你的代码经常超过这个限制，那你的代码中应该有设计不到位的地方。

理论上来说，你90%以上的代码的长度应该是少于79个字符的。

## 命名的一致性

团队如果在命名规则上能够共同遵守一些简单的规则，将会很有效地减少很多不必要的协作问题。

### 建议的命名规则

请注意第一条，`HTTPWriter`而不是`HttpWriter`，专有名词保持大写，如API。

- Class names: `CamelCase`, and capitalize acronyms: `HTTPWriter`, not `HttpWriter`.
- 常规变量名: `lower_with_underscores`.
- 方法/函数名: `lower_with_underscores`.
- 模块文件: `lower_with_underscores.py`. (但最好是使用那些连下划线都不需要的词)
- 静态变量: `UPPER_WITH_UNDERSCORES`.
- Precompiled regular expressions: `name_re`.

其他详情参见 [the Pocoo team][pocoo].

[pocoo]: https://flask.palletsprojects.com/en/1.1.x/styleguide/

### 下划线的使用

包括了前下划线与后下划线（`_prefix or suffix_`）。  

函数经常在名称前加单下划线来表示他是私有的，几乎是一个约定俗成的事实。但这个规则应该被谨慎使用，甚至只应该在以下两种情况下使用：

- 在会被广泛使用的API内部；
- 涉及[information hiding（信息隐藏）](http://wiki.c2.com/?InformationHiding)；

PEP8建议应该使用单下划线来避免命名与built-in命名冲突：

	sum_ = sum(some_long_list)
	print(sum_)

但事实上比起加下划线，或许你应该优先考虑换另一个词语。  

对于前置双下划线，理论上它**只应该**在遇到需要[name mangling behavior](https://docs.python.org/2/tutorial/classes.html#private-variables-and-class-local-references)的情况下使用，但很多情况下其实我们并不需要；  
而对于前后置双下划线（例如`__len__`）它只应该在实现python标准协议的过程中使用，它是为python内部协议特意保留的命名空间，不应该被随意在团队工作中使用。

**译者观点**   
介绍一下name mangling behavior。python在运行时会将由双下划线标识的变量在前方加上类名，例如在A类中的变量`__b`，在运行时会变成`_A__b`。这是因为python没有像其他语言一样的`private`关键词，只能通过这种方法起到标识变量的效果。这种方法有利于子类重写父类的方法而不会破坏类内部的方法调用。    
但这个机制是“防君子不防小人”（怪怪的）的，即便你使用了双下划线，其他人依旧可以强行使用类似`_A__b`的方法访问这个变量。在日常开发中有一个比较约定俗成的通用方法是在变量前加入单下划线而不是使用name mangling behavior，只起到一个警告的效果而不是完全阻止。

### 避免过于简单的命名

- 例如`i`，`x`，`_`等，在简单的函数中使用无可厚非，像lambda/列表解析等
- 但除了这些情况之外千万不要随意这么用，这将会给其他人的阅读造成很大的困难

### 业界观念上比较认可的命名方式

下列规则理论上应该被严格执行：

- 实例方法中第一个参数名称固定为`self`
- 类方法`@classmethod`中第一个参数名称固定为`cls`
- 可变参数的命名应固定采用`*args`, `**kwargs`

## 不值得钻牛角尖的一些点

这个部分列举了一些规则，不遵守这些规则并不会给你的开发带来任何便利（或者很少）。所以我们不应该在这上面花费太多时间钻牛角尖，仅仅遵守他们就行了。

### 在声明基类的时候统一继承于`object`

	# bad
	class JSONWriter:
	    pass
	
	# good
	class JSONWriter(object):
	    pass

在python2中，遵守这条规则非常非常重要，因为旧式类与新式类的表现有一定的差别。在python3，所有的类都默认继承自object，即所有的类都是新式类，所以这条规则在python3变得不那么重要了。

### 变量不要重名

	# bad
	class JSONWriter(object):
	    handler = None
	    def __init__(self, handler):
	        self.handler = handler
	
	# good
	class JSONWriter(object):
	    def __init__(self, handler):
	        self.handler = handler

### 使用列表解析替代map/filter

	 # bad
	map(truncate, filter(lambda x: len(x) > 30, items))
	
	 # good
	[truncate(x) for x in items if len(x) > 30]

在团队协作中，你的代码应该更注重可读性。在一些情况下，可能map与filter比起列表解析的方式更具有可读性，在这些情况下请自行选用。

### 使用小括号给过长的代码换行

换行的原则是**尽量不要出现反斜杠**，直接给出两个例子：

	# bad
	from itertools import groupby, chain, \
	                      izip, islice
	
	# good
	from itertools import (groupby, chain,
	                       izip, islice)

	# bad
	response = Search(using=client) \
	           .filter("term", cat="search") \
	           .query("match", title="python")
	
	# good
	response = (Search(using=client)
	            .filter("term", cat="search")
	            .query("match", title="python"))

### 使用`isinstance`而不是`type`进行类型比较

`isinstance`比起`type`，能够覆盖更多的情况，例如子类与抽象类。  
在大部分情况下应该尽量使用`isinstance`。  
当然，如果需要进行精确的类型判断或其他特殊情况时，请自行根据需要选用。

### 使用`with`语句处理文件与锁

`with`语句能够很方便地帮助开发者解决文件关闭与锁释放的问题，即使在代码执行过程中遇到异常。比起`try/finally`的组合要方便许多。

	# bad
	somefile = open("somefile.txt", "w")
	somefile.write("sometext")
	return
	
	# good
	with open("somefile.txt", "w") as somefile:
	    somefile.write("sometext")
	return

### 与`None`的比较用`is`而不是`==` 

`None`是`Nonetype`的唯一实例，即所有对它的引用事实上都链接到同一个对象。   
所以比较的时候我们并不需要特地调用`__eq__`将这个过程复杂化（使用==会调用对象的`__eq__`方法并以此判断他们是否相等），直接使用is比较他们是不是同一个对象即可。

	# bad
	if item == None:
	    continue
	
	# good
	if item is None:
	   continue

在与`None`的比较上，使用`is`代替`==`是一种又快又稳定的方法。

### 避免使用`sys.path`黑科技

**译者观点**   
原文的这个部分描述的不是很清楚，这里加了一部分我个人的理解。如有需要请参阅原文。

可能很多人使用类似`sys.path.insert(0, "../")`的方式来实现python的动态导入，但这种做法是非常影响可读性的，其他人很难轻易通过这个知道你到底想干嘛。   

python已经具备了非常完善的模块处理机制。通过修改`PYTHONPATH`可以调整加载的模块，或是运行时使用`-m`可以以模块方式运行python代码。另外，通过import与`__import`之类的方式也能够有效地解决动态调用模块的问题。   

模块不应该需要依赖工作目录的文件结构才能正常运行，这不免有些跨越层级了。模块应该在模块层面上解决互相依赖的问题（例如import），而不是通过文件层面。

### 尽量少/不自定义异常类型

> 如果你非要这么做的话，也尽量不要定义太多个。  

事实上，python已经提供了非常丰富的异常类型供用户选择。就目前而言，在日常开发中这些异常类型基本已经能够满足绝大多数的开发需要。

一个判定你是否需要自定义异常类型的方法是，考虑一下是否每次用户调用这个函数时都需要捕获这类异常。如果是，那你可能应该自定义，但这种情况相对来说非常少见。   

一个例子： [tornado.web.HTTPError
](http://www.tornadoweb.org/en/stable/web.html#tornado.web.HTTPError)

这类异常针对的是，所有的发生在框架范围内或用户代码内的http错误。

## 写好docstring

### 短docstring尽量只用一行

	# bad
	def reverse_sort(items):
	    """
	    sort items in reverse order
	    """
	
	# good
	def reverse_sort(items):
	    """Sort items in reverse order."""

### 用reST风格写docstring

	def get(url, qsargs=None, timeout=5.0):
	    """Send an HTTP GET request.
	
	    :param url: URL for the new request.
	    :type url: str
	    :param qsargs: Converted to query string arguments.
	    :type qsargs: dict
	    :param timeout: In seconds.
	    :rtype: mymodule.Response
	    """
	    return request('get', url, qsargs=qsargs, timeout=timeout)

在上面的例子中，关于timeout的类型是可以略去的，因为timeout的默认值已经很明显地表现了它是一个`float`类型；而qsargs没有指定它是什么类型，所以docstring为其做了标识；rtype主要用于标识返回类型。

代码被阅读的频率要远高于它被修改的频率，docstring的存在能够有效地降低阅读成本，是非常必要的。

但是也要注意的是，恰当的注释会对开发有所帮助，过多的注释反而会给人带来困扰。理论上，开发者应优先对一些会被广泛复用的代码（函数）加上注释，并在每次函数进行更新时对docstring进行同步更新。

## 范式 & 设计模式

### 函数 vs 类

- 多用函数而不是类。函数与模块是代码复用的基本单位，而且他们是最灵活的。
- 类比起函数更加“高级”，它一般被广泛应用在一些更为庞大的功能上例如容器的实现/代理/类型系统等。但是高级意味着，它的维护成本需要相应的提升。在大多数情况下，它很可能是一把“牛刀”，往往我们只需要使用函数就足以应对这些情况。
- 一些人喜欢用类将相关的函数进行分类，但这是错的。要达到同样的目的，应该使用module而不是class。尽管有些时候类可以表现地像是一个mini版本的命名空间（使用`@staticmethod`），但类中的方法通常应该围绕着对象的内部操作展开，而不仅仅只是一系列行为的分组。
- 通过一个模块去调用函数比通过类来调用要清晰得多。例如你需要管理一系列时间相关的函数：
	- 使用一个名为`lib.time`的模块来管理，可以灵活的根据需要进行导入与重命名；
	- 使用一个名为`TimeHelper`的类来管理，在使用时你甚至还需要构建一个它的子类来使用他的方法；

**译者观点**   
换言之，在python中，module的存在替在其他语言中举足轻重的class分担了许多工作。因为class比起module而言过于笨重了。对于一些轻量级的场景而言，同样的情况下使用module会让你的代码更加pythonic。  
不过不可否认的是，在很多情况下class比起module能够更好地处理复杂的逻辑，所以还是需要根据实际情况灵活选用。

### 命令式编程 vs 声明式编程

#### 概念 ####

- 命令式编程（Imperative）：喜欢大量使用可变对象和指令，我们总是习惯于创建对象或者变量，并且修改它们的状态或者值，或者喜欢提供一系列指令，要求程序执行。

- 声明式编程（Declarative）：对于声明式的编程范式，你不在需要提供明确的指令操作，所有的细节指令将会更好的被程序库所封装，你要做的只是提出你要的要求，声明你的用意即可。

#### 结论 ####

- 在python中应该尽可能使用声明式编程。
- 代码应该表达的是，它想完成什么样的功能，而不是描述怎么样完成这个功能。

一个关于list的例子，大大降低了复杂度，提高了效率并提高了可读性。

	# bad
	filtered = []
	for x in items:
	    if x.endswith(".py"):
	        filtered.append(x)
	return filtered
	
	# good
	return [x
	        for x in items
	        if x.endswith(".py")]

**译者观点**   
个人觉得这里作者想通过这个例子表达的意思应该是，多利用python自有的一些功能（或者自行编写的一些子函数）简化功能的实现，减少代码中“描述怎么样完成这个功能”的部分（实际上是分摊到不同地方中去了）。   
例如例子中列表解析是python提供的功能，对于列表解析的实现是由python完成的，在阅读代码时我们需要关注的并不是他是怎么实现的，而是这个函数到底会得到一个什么样的结果。这在团队工作中是极为重要的。

### 同样功能下，越干净，越短，越好

	# bad
	def dedupe(items):
	    """Remove dupes in-place, return items and # of dupes."""
	    seen = set()
	    dupe_positions = []
	    for i, item in enumerate(items):
	        if item in seen:
	            dupe_positions.append(i)
	        else:
	            seen.add(item)
	    num_dupes = len(dupe_positions)
	    for idx in reversed(dupe_positions):
	        items.pop(idx)
	    return items, num_dupes
	
	This same function can be written as follows:
	
	# good
	def dedupe(items):
	    """Return deduped items and # of dupes."""
	    deduped = set(items)
	    num_dupes = len(items) - len(deduped)
	    return deduped, num_dupes

- 在这个函数下，后者相比前者的优势不只在于行数
- 逻辑更加清晰
- 调试效率非常高
- 造成更少的bug

### 在参数与返回值上尽量使用简单类型

- 函数操作应该基于数据，尽量使用简单类型而不是自定对象
- 简单类型：`set, tuple, list, int, float, bool` 
- 在上述类型无法满足的情况下再考虑使用更为复杂的类型

**译者观点**   
使用基本类型作为函数IO一定程度上也能够降低整体的耦合性。

### 避免传统的OOP编程思想

- 在java与C++中，代码的复用基本上通过类继承与多态实现；
- 在python中，尽管我们也可以这么做，但事实上这并不是很pythonic。
- 在python中，代码的复用最普遍的做法是通过模块与函数来实现。python的核心开发者曾经做过一个演讲批评了类被滥用的行为["Stop Writing Classes"][stop-classes]，函数在python中是“一等公民”，在很多时候我们并不需要特地去构建一个类。如果你在实际使用中使用类实现代码的重用，请重新考虑一下，尤其是当你的类名与其内部函数很相似的时候。 `(e.g. Runnable.run())`
- 对于多态，应该使用鸭子类型来解决。在python中，我们关注的不再是对象本身是什么东西，而是它能够被怎么使用。

[stop-classes]: https://www.youtube.com/watch?v=o9pEzgHorH0

**译者观点**   
作者这个部分并不是说OOP编程不好，而是在OOP的实现上python的方式可能跟其他语言有所不同。

### mixin好，但不要滥用 

关于mixin的概念详见[这里](http://blog.csdn.net/gzlaiyonghao/article/details/1656969)。

- 使用mixin可以简单有效达成基类重用的目的，而不必深入其内部的类型层次结构。
- 但过多的嵌套会大幅度加大代码的阅读难度，在使用前要简单评估一下是否过度设计。

关于使用上的规范：

		class APIHandler(AuthMixin, RequestHandler):
		    """Handle HTTP/JSON requests with security."""

这样设计可以直接在函数签名透露出：“这个类混合了auth的行为，他是一个RequestHandler。”

**译者观点**   
在命名中直接体现函数的功能是非常优雅的，函数的设计应该遵守这个原则，让其他用户直接通过名字与docstring就能够准确地使用这个函数。

### 小心地使用框架

- python有非常非常多的第三方库，能让你很方便地构建自己的框架，实现自己需要的功能。
- 在使用中需要非常注意，不要跟其他的库有冲突（包括命名/重复功能等）

一个糟糕的例子：

	from something import *
	# 千万别这样，除非你真的很清楚这里面有什么东西且不会跟你造成冲突

### 尊重元编程

- 元编程是python中非常重要的核心组成部分，许多功能都是基于此实现的，包括装饰器/上下文管理器/描述符/导入等等；
- 熟悉元编程能够让你应对非常复杂/多样的场景，自定义对象的表现，是你编写属于自己的框架的第一步。

### 不要害怕带有双下划线的方法（内部方法）

- 他们无非就是一些一开始选定的用于实现内部协议的命名空间，并没有什么太过神奇的地方；
- 不过需要承认的是，他们比起常规函数确实可能会造成一些令人困扰的结果；
- 没有充分考虑就贸然重写他们并不是一个好主意，除非你真的有一个足够充分的理由。

## “python之禅”在你代码中的具体应用

### 美比丑好
以编写优美的代码为目标。

**译者观点**   
在团队协作中，编程的目的绝对绝对不只是完好的实现功能，尤其对于python这种与自然语言已经非常接近的高级语言。

### 准确比模糊好

在代码中应该尽量将代码翻译为标准的英文（人类语言）且让人能够基本明白执行这段代码会发生什么。

### 扁平比嵌套好

直接扔两个例子：

	# bad
	if response:
	    if response.get("data"):
	        return len(response["data"])
	
	# good
	if response and response.get("data"):
	    return len(response["data"])

### 可读性非常重要

- 最好不要用缩写
- 不要怕参数名太长
- 多加注释

### 错误永远不应该被轻易忽略

- 最大的根源来自于`except: pass`，这种做法很可能会漏掉奇怪的错误而导致后期难以调试，永远都不要这么用它。
- 如果错误不处理，至少应该有log记录它曾经发生过。

### 所有代码都应该容易解释

- 大多数python函数与对象都有一种易于解释的实现方法
- 如果你的实现方法难以让其他人看懂，或许应该考虑重构一下
- 通常可以通过拆分来解决

### 测试代码是必要的

比起代码的美丑，正确性无疑是更为重要的。  
难以保证正确性的优美代码 < 能保证正确的丑代码

## 一些无关好坏的主观比较

下面会介绍的内容是工程师因自身编程习惯的不同可能会出现的分歧。不过这些内容不涉及好坏，使用方式的差异也不会对团队工作造成很大的影响。

**译者观点**   
在合理且团队允许的情况下，每个工程师的编程习惯都应该被尊重，个人习惯的不同不应该成为你修改其他人代码的理由。

### `str.format` 与 `%`

前者稳定性更强，后者更为简洁

### `if item` 与 `if item is not None`

- 首先明确这两个是不一样的。
- if item还会比较空字符串与空列表的问题，而后者仅仅只会比较它是否是None本身。
- 如果这个判断不影响正确性，那么两者的使用应该是自由的。

### 多行字符串

直接上两个例子：

	msg = ("Hello, wayward traveler!\n"
	       "What shall we do today?\n"
	       )
	print(msg)
	

	msg = """Hello, wayward traveler!
	What shall we do today?
	"""
	print(msg)

看个人选择。

如果多行字符串被放到函数里，需要变成：

	def abc():
		msg = """Hello, wayward traveler!
	What shall we do today?
		"""
	print(msg)

比较丑。

### `raise`应该抛出类还是实例

	raise ValueError
	raise ValueError()

- 在上述情况的实现上，python会自动把类转换为实例；
- 建议还是用第二种，因为可以加入一些帮助信息以方便后期调试；
- 但不是硬性要求，因为两者最终表现确实是一样的

## 一些很有用的内部库与三方库

### The Standard Library

- `import datetime as dt`: 用这种方法引用时间
- `dt.datetime.utcnow()`: 比起`.now()`，这个是真正的本地时间（utc）
- `import json`: 用json进行数据交换，json是一种普遍认为较好的数据交换方式
- `from collections import namedtuple`: 快速定义一个轻量级的数据类型，类似一个轻量级的不带方法的class
- `from collections import defaultdict`: 默认字典
- `from collections import deque`: 快速的双端队列
- `from itertools import groupby, chain`: chain用于迭代器组合，groupby用于数据分类
- `from functools import wraps`: 构造一个好的装饰器必不可少
- `argparse`: 用于处理命令行参数
- `fileinput`: 用于文件读取，支持原位修改，个人感觉并没有很方便
- `log = logging.getLogger(__name__)`: 日志管理
- `from __future__ import absolute_import`: 解决自定义模块与内部模块引用可能冲突的问题（python3已强制使用）

### Common Third-Party Libraries

- `python-dateutil` for datetime parsing and calendars
- `pytz` for timezone handling
- `tldextract` for better URL handling
- `msgpack-python` for a more compact encoding than JSON
- `futures` for Future/pool concurrency primitives
- `docopt` for quick throwaway CLI tools
- `py.test` for unit tests, along with `mock` and `hypothesis`

### Local Development Project Skeleton

对于所有python项目与库而言，不要在根目录里加入`__init__.py`文件！

一个项目结构最好是这样摆放：

- `mypackage/__init__.py` 不如 `src/mypackage/__init__.py`
- `mypackage/lib/__init__.py` 不如 `lib/__init__.py`
- `mypackage/settings.py` 不如 `settings.py`

注：此处的根目录为项目根目录。

其他文件：

- `README.rst`/`README.md` 使用reST/Markdown向新用户描述这个项目的功能，介绍整个项目。
- `setup.py` 用于在不同机器上安装这个项目包。
- `requirements.txt` 声明了这个项目所需的依赖库，需要`pip`。
- `dev-requirements.txt` 与上面不同的是，它用于管理一些调试过程中需要的依赖库。
- `Makefile` 用于简单的 build/lint/test/run 操作步骤

Also, always [pin your requirements](http://nvie.com/posts/better-package-management/).


## 灵感与启发

下面介绍的链接或许可以给你一些启发，让你在开发过程中可以写出风格更好、更加pythonic的代码。

- Python's stdlib [`Counter` class][Counter], implemented by Raymond Hettinger
- The [`rq.queue` module][rq], originally by Vincent Driessen
- This document's author also wrote [this blog post on "Pythonic" code][idiomatic]

Go forth and be Pythonic!

```
$ python
>>> import antigravity
```

[Counter]: https://github.com/python/cpython/blob/57b569d8af2b3263c5d9e6d75fb308f89ea17ac6/Lib/collections/__init__.py#L446-L841
[rq]: https://github.com/nvie/rq/blob/master/rq/queue.py
[idiomatic]: http://www.pixelmonkey.org/2010/11/03/pythonic-means-idiomatic-and-tasteful

## 编著者

- Andrew Montalenti ([@amontalenti][amontalenti]): original author
- Vincent Driessen ([@nvie][nvie]): edits and suggestions

[amontalenti]: http://twitter.com/amontalenti
[nvie]: http://twitter.com/nvie

---

Like good Python style? Then perhaps you'd like to [work on our team of Pythonistas][tweet] at Parse.ly!

[tweet]: https://twitter.com/amontalenti/status/682968375702716416
