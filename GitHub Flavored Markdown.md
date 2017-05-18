# GitHub Flavored Markdown 规范
版本 0.27-gfm (2017-2-20)

这份规范基于 [John MacFarlane](http://github.com/jgm) 的 [CommonMark Spec](http://spec.commonmark.org/)

1. 引言
	1.1. GitHub Flavored Markdown 是什么
	1.2. Markdown 是什么
	1.3. 为什么需要一个标准
	1.4. 关于这份文档
	
2. 基础
	2.1. 字符和行
	2.2. 制表符
	2.3. 不安全字符
	
3. 块级元素和行内元素
	3.1. 优先级
	3.2. 容器块和叶块
	
4. 叶块
	4.1. Thematic Breaks
	4.2. ATX 

## 1 引言
### 1.1 GFM 是什么？
GitHub Flavored Markdown通常缩写为GFM，是目前对GitHub.com和GitHub Enterprise的用户内容支持的Markdown方言。

这个基于CommonMark Spec的正式规范定义了这种方言的语法和语义。

GFM是CommonMark的严格超集。 所有在GitHub用户内容中支持，但未在原始CommonMark Spec上指定的功能，被称为扩展，并且被突出显示。

### 1.2 Markdown 是什么？
Markdown是一种用于编写结构化文档的纯文本格式，基于用于在电子邮件和usenet帖子中指示格式化的约定。 2004年由John Gruber开发，他在Perl中编写了第一个Markdown-to-HTML转换器，很快就变得无所不在。 在接下来的十年中，许多语言开发了数十种实现方式。 有些扩展了原始Markdown语法，其中包含脚注，表格和其他文档元素的约定。 一些Markdown文档也可以转换为HTML以外的格式。 像Reddit，StackOverflow和GitHub这样的网站有数百万人使用Markdown。 而Markdown开始在互联网之外使用，撰写书籍，文章，幻灯片，信件和讲义。

Markdown与许多其他轻量级标记语法（通常更容易编写）的区别，是其可读性。 正如Gruber写道：

> Markdown格式化语法的首要设计目标是使其尽可能可读。 这个想法是，Markdown格式的文档应该是原样的，作为纯文本，而不是像标签或格式化指令那样被标记。
([http://daringfireball.net/projects/markdown/](http://daringfireball.net/projects/markdown/))

可以通过将[AsciiDoc](AsciiDoc)的示例与Markdown的等效示例进行比较来说明这一点。 以下是AsciiDoc手册的AsciiDoc示例：

[AsciiDoc]: http://www.methods.co.nz/asciidoc/

```
1. List item one.
+
List item one continued with a second paragraph followed by an
Indented block.
+
.................
$ ls *.sh
$ mv *.sh ~/tmp
.................
+
List item continued with a third paragraph.

2. List item two continued with an open block.
+
--
This paragraph is part of the preceding list item.

a. This list is nested and does not require explicit item
continuation.
+
This paragraph is part of the preceding list item.

b. List item b.

This paragraph belongs to item two of the outer list.
--
```

下面是Markdown实现的等效示例：

```
1.  List item one.

    List item one continued with a second paragraph followed by an
    Indented block.

        $ ls *.sh
        $ mv *.sh ~/tmp

    List item continued with a third paragraph.

2.  List item two continued with an open block.

    This paragraph is part of the preceding list item.

    1. This list is nested and does not require explicit item continuation.

       This paragraph is part of the preceding list item.

    2. List item b.

    This paragraph belongs to item two of the outer list.
```

AsciiDoc版本可以说比较容易编写，你不需要担心缩进，但是Markdown版本更容易阅读，列表项目的嵌套在源中显而易见，而不仅仅是处理过的文档。

### 1.3 为什么需要一个规范?
John Gruber对Markdown语法规范没有一个明确的描述。 以下是一些不能解答的问题的例子：

1. 子列表需要多少缩进？ 规范说，延续段落需要缩进四个空格，但对子列表来说并不明确。 也许很自然地想到，他们也必须缩进四个空格，但是`Markdown.pl`没有具体说明。 这不是一个“小问题”，在这个问题上的矛盾往往会导致用户在编写真实文档时，出现意料之外的结果。
([John Gruber的观点](http://article.gmane.org/gmane.text.markdown.general/1997))

2. 在块引用或标题之前是否需要空行？ 大多数实现不需要空行。 然而，这可能会导致硬包装文本(Hard-Wrapped Text)的意想不到的结果，以及解析中的歧义（注意，一些实现将标题放在blockquote内，而其他实现不会）。
([John Gruber 对空行的观点](http://article.gmane.org/gmane.text.markdown.general/2146))

3. 在缩进代码块之前是否需要空行？ （`Markdown.pl`需要它，但是在文档中没有提及，有些实现不需要它。）

	```
	paragraph
	    code?
	```
	
4. 确定列表项目被包裹在`<p>`标签中的确切规则是什么？ 列表可能部分“松散”，部分“紧密”？ 我们应该怎么做这样的列表？

	```
	1. one
	
	2. two
	3. three
	```
	
	还有这样的？

	```
	1.  one
	    - a
	
	    - b
	2.  two
	```
	
	([John Gruber 的观点](http://article.gmane.org/gmane.text.markdown.general/2554))

5. 列表标记可以缩进吗？ 有序列表标记可以向右对齐吗？

	```
	 8. item 1
	 9. item 2
	10. item 2a
	```

6. 这个列表在第二个项目中有一个主题中断(Thematic Break)，还是两个列表由主题中断分隔？

	```
	* a
	* * * * *
	* b
	```

7. 当列表标记从数字更改为分隔符时，我们有两个列表还是一个？ （Markdown语法描述指出是两个，但perl脚本和许多其他实现都只产生一个。）

	```
	1. fee
	2. fie
	-  foe
	-  fum
	```

8. 内联结构的标记的优先规则是什么？ 例如，以下是否是有效的链接，还是代码块优先级更高？

	```
	[a backtick (`)](/url) and [another backtick (`)](/url).
	```

9. 什么是斜体强调（emphasis）和粗体强调（strong emphasis）标记的优先规则？ 例如，应该如何解析以下内容？

	```
	* foo * bar * baz *
	```
	
10. 块级和内联结构之间的优先级规则是什么？ 例如，应该如何解析以下内容？

	```
	- `a long code span can contain a hyphen like this
	  - and it can screw things up`
	```
	
11. 列表项可以包含标题标记吗？（`Markdown.pl` 不允许这样做，但允许引用块中包含标题标记） 

	```
	- # Heading
	```

12. 列表项可以为空吗？

	```
	* a
	*
	* b
	```

13. 链接可以定义在引用和列表中吗？

	```
	> Blockquote [foo].
	>
	> [foo]: /url
	```
	
14. 如果一个参数有多个定义，那个优先级更高？
	
	```
	[foo]: /url1
	[foo]: /url2
	
	[foo][]
	```
	
在没有规范的情况下，早期的实现会查看`Markdown.pl`来解决这些歧义。 但是`Markdown.pl`充满了漏洞，在很多情况下都不能正确的显示结果，所以它不是一个令人满意的替代规范。

因为没有明确的规范，实现方式已经相当分散。 因此，用户经常惊讶地发现，在一个系统上呈现的效果（例如，github wiki），在另一个系统上呈现不同的效果（例如，使用pandoc转换为docbook）。 更糟糕的是，Markdown中的任何内容都不是“语法错误”，所以经常不能立刻发现区别。

### 1.4 关于本文
本文试图明确地指定Markdown语法。 它包含许多横向对比Markdown和HTML的例子。 这些旨在进行一致性测试。 随附的脚本`spec_tests.py`可用于针对任何Markdown程序运行测试：

```
python test/spec_tests.py --spec spec.txt --program PROGRAM
```

本文档描述了如何将Markdown解析为抽象语法树，使用语法树而不是HTML的抽象表示有特定的优势。 但是，HTML能够表示我们需要的结构区别，并且测试选择HTML,使得可以在不编写抽象语法树渲染器的情况下针对实现运行测试。

本文档是从Markdown写的一个文本文件`spec.txt`生成的，并附有一个小的扩展用于并行测试。 脚本`tools/makespec.py`可用于将`spec.txt`转换为HTML或CommonMark（可以将其转换为其他格式）。

在下面的示例中，`→` 代表制表符。

---

## 2 基础
### 2.1 字符和行
任何字符序列都是有效的CommonMark文档。

一个字符是一个Unicode代码点。虽然一些代码点（例如，组合重音符）在直观上不符合字符，但是为了本规范的目的，所有代码点都被视为字符。

此规范未指定编码;它认为行由字符组成，而不是字节。符合一致的解析器可能被限制为某种编码。

一行是除换行（`U+000A`）或回车（`U+000D`）之外的零个或多个字符的序列，后面是一行结尾或文件结尾。

行尾是换行符（`U+000A`），回车符（`U+000D`）不加换行符，或回车符加换行符。

不包含字符的行或仅包含空格（`U+0020`）或制表符（`U+0009`）的行视为空行。

在本规范中将使用以下定义的字符类：

空格字符是空格（`U+0020`），制表符（`U+0009`），换行符（`U+000A`），行列表（`U+000B`），Form Feed（`U+000C`）或回车（`U+000D`)。

空格是一个或多个空格字符的序列。

Unicode空格字符是Unicode `Zs`类中的任何代码点，或者制表符（`U+0009`），回车符（`U+000D`），换行符（`U+000A`）或Form Feed（`U+000C`）。

Unicode空格是一个或多个Unicode空格字符的序列。

一个空格是`U+0020`。

非空格字符为，不是空格字符的任何字符。

一个ASCII标点符号是`!`, `"`, `#`, `$`, `%`, `&`, `'`, `(`, `)`, `*`, `+`, `,`, `-`, `.`, `/`, `:`, `;`, `<`, `=`, `>`, `?`, `@`, `[`, `\`, `]`, `^`, `_`, `` ` ``, `{`, `|`, `}`, 或 `~`.

标点符号是ASCII标点符号或Unicode类`Pc`，`Pd`，`Pe`，`Pf`，`Pi`，`Po`或`Ps`中的任何内容。

### 2.2 制表符

行内的制表符不会解释为空格。 然而，在定义块时，空白字符有助于理解块的区域，制表符可以解释为4个连续的空格字符。

因此，可以使用制表符来代替缩进代码块中的四个空格。（但请注意，内部制表符以制表符的形式传递，而不会解释为空格。）

再次说明，在下面的示例中，`→` 代表制表符，`·`代表空格。

Example 1
```
→foo→baz→→bim

<pre><code>foo→baz→→bim
</code></pre>
```

Example 2
```
··→foo→baz→→bim

<pre><code>foo→baz→→bim
</code></pre>
```

Example 3
```
····a→a
····ὐ→a
    
<pre><code>a→a
ὐ→a
</code></pre>
```

下面的例子中，列表项后面紧跟的段落以制表符缩进，这和 4 个空格的效果一样：
Example 4
```
··-·foo

→bar

<ul>
<li>
<p>foo</p>
<p>bar</p>
</li>
</ul>
```

Example 5
```
-·foo

→→bar

<ul>
<li>
<p>foo</p>
<pre><code>··bar
</code></pre>
</li>
</ul>
```

通常，引用块`>`可以选择性的跟一个空格，空格不被视为内容的一部分。 在以下情况下，`>`后跟一个制表符，将其视为空格。 因为这些空格之一被认为是分隔符的一部分，所以`foo`被认为是在引用块中缩进了六个空格，所以我们得到一个从两个空格开始的缩进代码块。

Example 6
```
>→→foo

<blockquote>
<pre><code>··foo
</code></pre>
</blockquote>
```

Example 7
```
-→→foo

<ul>
<li>
<pre><code>··foo
</code></pre>
</li>
</ul>
```

Example 8
```
····foo
→bar

<pre><code>foo
bar
</code></pre>
```

Example 9
```
·-·foo
···-·bar
→·-·baz

<ul>
<li>foo
<ul>
<li>bar
<ul>
<li>baz</li>
</ul>
</li>
</ul>
</li>
</ul>
```

Example 10
```
#→Foo

<h1>Foo</h1>
```

Example 11
```
*→*→*→

<hr·/>
```

### 2.3 不安全字符

出于安全考虑，Unicode字符`U+0000`必须替换为REPLACEMENT CHARACTER（`U+FFFD`）。

## 3 块和内联

我们可以将文档看作一个块结构的集合，如段落，引用块，列表，标题，规则和代码块。 一些块（如引用块和列表项）包含其他块; 其他（如标题和段落）包含内联内容 - 文本，链接，强调文本，图像，代码等。

### 3.1 优先级

块结构始终优先于内联结构。 例如，以下是一个包含两个项目的列表，而不是包含了一个项目的列表：

Example 12
```
-·`one
-·two`

<ul>
<li>`one</li>
<li>two`</li>
</ul>
```

这意味着解析可以分两步进行：首先，辨别文档的块结构; 第二步，段落，标题和其他块结构中的文本行被解析为内联结构。 第二步需要链接引用定义的信息，只有第一步结束时才可用。 请注意，第一步需要按顺序处理，但第二步可以并行处理，因为一个块元素的内联解析不会影响任何其他元素的内联解析。

### 3.2 容器块和叶块

块级元素分两种：
容器块，可以包含其他块；
叶块，不能包含其他块。

## 4 叶块 Leaf Blocks

这个章节介绍不同的叶块。

### 4.1 主题中断（水平分割线） THematic Breaks

由0-3空格的缩进的行，后跟三个或更多连续的`-`,`_`或`*`，后面可以选择任意数量的空格，形成一个主题中断。

Example 13

```
***
---
___

<hr·/>
<hr·/>
<hr·/>
```

不支持的字符：

Example 14
```
+++

<p>+++</p>
```

Example 15
```
===

<p>===</p>
```

字符数量不够：

Example 16
```
--
**
__

<p>--
**
__</p>
```

允许 1-3 个空格缩进：
Example 17
```
·***
··***
···***

<hr·/>
<hr·/>
<hr·/>
```

4 个空格就太多了：
Example 18
```
····***

<pre><code>***
</code></pre>
```

Example 19
```
Foo
····***

<p>Foo
***</p>
```

多于 3 个字符同样有效：
Example 20
```
_____________

<hr·/>
```

这些字符之间可以有空格：
Example 21
```
·-·-·-

<hr·/>
```

Example 22
```
·*··*·**·**·**·**·*

<hr·/>
```

Example 23
```
-·-·-·····-·-··-·-

<hr·/>
```

行末可以有空格：
Example 24
```
-·-·-·-·····

<hr·/>
```

但是，行内不能出现其他字符：
Example 25
```
_·_·_·_·a

a------

---a---

<p>_·_·_·_·a</p>
<p>a------</p>
<p>---a---</p>
```

必须是同一种非空白字符，以下仍不是一个水平线：
Example 26
```
·*-*

<p><em>-</em></p>
```

水平线前后不需要空行：
Example 27
```
-·foo
***
-·bar

<ul>
<li>foo</li>
</ul>
<hr />
<ul>
<li>bar</li>
</ul>
```

水平线可以打断一个段落：
Example 28
```
Foo
***
bar

<p>Foo</p>
<hr·/>
<p>bar</p>
```

如果符合上述条件的破折号也可以被解释为 __Setext 标题__ ，那么优先解释为 __Setext标题__ 。 例如，这是一个 __Setext 标题__ ，而不是一个后跟水平线的段落：
Example 29
```
Foo
---
bar

<h2>Foo</h2>
<p>bar</p>
```

如果水平线和列表项同时打断一行，水平线优先级更高：
Example 30
```
*·Foo
*·*·*
*·Bar

<ul>
<li>Foo</li>
</ul>
<hr />
<ul>
<li>Bar</li>
</ul>
```

如果想用水平线分割列表，使用其他表示列表的字符：
```
-·Foo
-·*·*·*

<ul>
<li>Foo</li>
<li>
<hr />
</li>
</ul>
```

### 4.2 ATX 标题 ATX headings

__ATX 标题__ 由一串可以解析为内联内容的字符，放在1-6个未转义的`#`打开序列和任意数量的未转义`#`的可选关闭序列之间。 `#`的打开序列后面必须有空格或行尾。 `#`s 的可选关闭序列前面必须有一个空格，后面可以是空格。 开头`#`字符可以缩进0-3个空格。 在被解析为内联内容之前，标题原始内容中前面和后面的空白会被忽略。 标题级别等于打开序列中`#`的个数。

标题示例：
Example 32
```
# foo
## foo
### foo
#### foo
##### foo
###### foo

<h1>foo</h1>
<h2>foo</h2>
<h3>foo</h3>
<h4>foo</h4>
<h5>foo</h5>
<h6>foo</h6>
```

超过 6 个 `#` 不是标题：
Example 33
```
#######·foo

<p>#######·foo</p>
```

`#` 字符和标题内容之间至少需要一个空格，除非标题为空。 请注意，许多实现目前不要求加空格。 然而，这个空格是原始的 ATX 实现所必需的，它有助于防止像下面这样的东西被解析为标题：
Example 34
```
#5·bolt

#hashtag

<p>#5 bolt</p>
<p>#hashtag</p>
```

这不是一个标题，因为`#`被转义了
Example 35
```
\##·foo

<p>## foo</p>
```

文本内容解析为内联元素
Example 36
```
#·foo·*bar*·\*baz\*

<h1>foo·<em>bar</em>·*baz*</h1>
```

前面和后面的空白字符，解析时会忽略：
Example 37
```
#···········foo···········

<h1>foo</h1>
```

可以缩进 1-3 个空格：
*Example 38*
```
 ###·foo
 ·##·foo
 ··#·foo
   
<h3>foo</h3>
<h2>foo</h2>
<h1>foo</h1>
```

4 个空格则太多了：
*Example 39*
```
····#·foo

<pre><code>#·foo
</code></pre>
```

*Example 40*
```
foo
····#·bar
    
<p>foo
#·bar</p>
```

`#`关闭序列可选：
*Example 41*
```
##·foo·##
··###···bar····###
  
<h2>foo</h2>
<h3>bar</h3>
```

关闭序列的长度不必和打开序列的一样：
*Example 42*
```
#·foo·
##################################
#####·foo·##

<h1>foo</h1>
<h5>foo</h5>
```

关闭序列之后可以有空格：
*Example 43*
```
###·foo·###····

<h3>foo</h3>
```

除了空格之外的`#`序列不会视为关闭序列，将算作标题内容的一部分：
*Example 44*
```
###·foo·###·b

<h3>foo·###·b</h3>
```

关闭序列之前必须有一个空格：
*Example 45*
```
#·foo#

<h1>foo#</h1>
```

转义的`#`不是关闭序列的一部分：
*Example 46*
```
###·foo·\###
##·foo·#\##
#·foo·\#

<h3>foo·###</h3>
<h2>foo·###</h2>
<h1>foo·#</h1>
```

__ATX 标题__ 前后不需要空行，并且它可以打断段落：
*Example 47*
```
****
##·foo
****

<hr·/>
<h2>foo</h2>
<hr·/>
```

*Example 48*
```
Foo·bar
#·baz
Bar·foo

<p>Foo·bar</p>
<h1>baz</h1>
<p>Bar·foo</p>
```

_ATX 标题_ 可以为空：
*Example 49*

```
##·
#
###·###

<h2></h2>
<h1></h1>
<h3></h3>
```

### 4.3 Setext 标题 Setext Headings

_Setext 标题_ 由一行或多行文本组成，每行包含至少一个非空字符，不超过3个空格缩进，后跟一个 _Setext 标题下划线_ 。文本行必须满足上边的条件，如果后边没有跟 _Setext 标题下划线_ ，它们将被解释为一个段落：它们不能被解释为代码块， _ATX 标题_ ， _块引用_ ， _水平线_ ， _列表项_ 或 _HTML块_ 。

一个 _setext标题下划线_ 是一个`=`字符序列或`-`字符序列，不超过3个空格缩进和任意数量的尾随空格。包含一个`-`的行,如果可以解释为空列表项，则应以此方式解释，而不能解释为 _setext标题下划线_ 。

如果 _setext标题下划线_ 使用`=`字符，代表1级标题，`-` 字符代表2级标题。标题的内容是将前面的文本行解析为`CommonMark`内联内容的结果。

一般来说， _setext标题_ 前后不需要空行。但是，它不能打断一个段落，所以当一个 _setext标题_ 来自一个段落之后，它们之间需要一个空行。


简单的例子：
*Example 50*
```
Foo·*bar*
=========

Foo·*bar*
---------

<h1>Foo·<em>bar</em></h1>
<h2>Foo·<em>bar</em></h2>
```

标题的内容可以超过一行：
*Example 51*
```
Foo·*bar
baz*
====

<h1>Foo·<em>bar
baz</em></h1>
```

标题下划线的长度任意：
*Example 52*
```
Foo
-------------------------

Foo
=

<h2>Foo</h2>
<h1>Foo</h1>
```

标题的内容最多可以缩进三个空格，缩进的空格可以不加标题下划线
*Example 53*
```
···Foo
---

··Foo
-----

··Foo
··===
  
<h2>Foo</h2>
<h2>Foo</h2>
<h1>Foo</h1>
```

缩进四个空格就太多了：
*Example 54*
```
····Foo
····---

····Foo
---

<pre><code>Foo
---

Foo
</code></pre>
<hr·/>
```

标题下划线可以缩进最多三个空格，也可以跟尾随空格：
*Example 55*
```
Foo
···----·····      

<pre><code>Foo
---

<h2>Foo</h2>
```

缩进四个空格就太多了：
*Example 56*
```
Foo
····----

<p>Foo
---</p>
```     

标题下划线中间不能有空格：
*Example 57*
```
Foo
=·=

Foo
---·-

<p>Foo
=·=</p>
<p>Foo</p>
<hr·/>
```

标题内容可以加尾随空格：
*Example 58*
```
Foo··
-----

<h2>Foo</h2>
```

标题内容可以加反斜线：
*Example 59*
```
Foo\
----

<h2>Foo\</h2>
```

块级元素的优先级高于行内元素，以下会解析为标题：
*Example 60*
```
`Foo
----
`

<a title="a lot
---
of dashes"/>

<h2>`Foo</h2>
<p>`</p>
<h2>&lt;a title=&quot;a lot</h2>
<p>of dashes&quot;/&gt;</p>
```

标题下划线不能是列表项或引用块的延续行
*Example 61*
```
> Foo
---

<blockquote>
<p>Foo</p>
</blockquote>
<hr />
Example 62
```

*Example 62*
```
> foo
bar
===

<blockquote>
<p>foo
bar
===</p>
</blockquote>
```

*Example 63*
```
- Foo
---

<ul>
<li>Foo</li>
</ul>
<hr />
```

段落后跟 _Setext 标题_ ，要以空行隔开，否则段落也会解析为标题的一部分：
*Example 64*
```
Foo
Bar
---

<h2>Foo
Bar</h2>
```

但是，在 _Setext 标题_ 前后一般不需要空行：
*Example 65*
```
---
Foo
---
Bar
---
Baz

<hr />
<h2>Foo</h2>
<h2>Bar</h2>
<p>Baz</p>
```

_Setext 标题_ 可以为空：
*Example 66*
```
====

<p>====</p>
```


_Setext 标题文本_ 不能解释为段落以外的块结构，所以以下的情况会被解析为水平线：
*Example 67*
```
---
---

<hr/>
<hr/>
```

*Example 68*
```
-·foo
-----

<ul>
<li>foo</li>
</ul>
<hr />
```

*Example 69*
```
····foo
---

<pre><code>foo
</code></pre>
<hr />
```

*Example 70*
```
>·foo
-----

<blockquote>
<p>foo</p>
</blockquote>
<hr />
```

如果想在标题中显示 `> foo`，可以使用反斜线`\`转义：
*Exampel 71*
```
\>·foo
------

<h2>&gt; foo</h2>
```

__兼容性说明：__ 大多数现有的Markdown实现不允许 _settext标题_ 的文本跨越多行， 但是对于如何解释没有共识。

```
Foo
bar 
---
baz
```

人们可以找到四种不同的解释:
1. 段落“Foo”，标题“bar”，段落“baz”
2. 段落“Foo bar”，水平线，段落“baz”
3. 段落“Foo bar --- baz”
4. 标题“Foo bar”，段落“baz”

解释4最自然，解释4通过允许多行标题来增加CommonMark的表现力。 想要解释1，可以在第一段之后放一个空行：
*Example 72*
```
Foo

bar
---
baz

<p>Foo</p>
<h2>bar</h2>
<p>baz</p>
```

想要解释2，可以在水平线前后加空行：
*Example 73*
```
Foo
bar

---

baz

<p>Foo
bar</p>
<hr />
<p>baz</p>
```

或者水平线使用不会和标题冲突的语法
*Example 74*
```
Foo
bar
*·*·*
baz

<p>Foo
bar</p>
<hr />
<p>baz</p>
```

想要解释3，可以使用反斜线转义：
*Example 75*
```
Foo
bar
\---
baz

<p>Foo
bar
---
baz</p>
```

### 4.4 缩进代码块

缩进代码块由一个或多个由空白行分隔的缩进块组成。 缩进的块是非空白行的序列，每个缩进四个或更多个空格。 代码块的内容是行的文字内容，包括尾行尾数(_trailing line endings_)，减去四个缩进空格。 缩进的代码块没有信息字符串。

缩进代码块不能中断段落，因此段落和后续缩进代码块之间必须有空白行。 （但是，代码块和下一段之间不需要空行）。
*Example 76*
```
····a·simple
······indented code block
      
<pre><code>a·simple
··indented·code·block
</code></pre>
```

如果既可以作为代码块进行解释，又可以解释为列表项目，则列表项目解释优先：
*Example 77*
```
··-·foo

····bar
   
<ul>
<li>
<p>foo</p>
<p>bar</p>
</li>
</ul> 
```

*Example 78*
```
1.··foo

····-·bar

<ol>
<li>
<p>foo</p>
<ul>
<li>bar</li>
</ul>
</li>
</ol>
```

以下代码是字面文本，不会解析为Markdown：
*Example 79*
```
····<a/>
····*hi*

····-·one

<pre><code>&lt;a/&gt;
*hi*

-·one
</code></pre>
```

三个被空行隔开的块：
*Example 80*
```
····chunk1

····chunk2
··
·
·
····chunk3

<pre><code>chunk1

chunk2



chunk3
</code></pre>
```

超过四个空格的空白也会解析出来：
*Example 81*
```
····chunk1
······  
······chunk2
      
<pre><code>chunk1
··
··chunk2
</code></pre>
```

缩进的代码不能中断一行：
*Example 82*
```
Foo
····bar
    
<p>Foo
bar</p>
```

但是，少于四个前导空格的非空行将立刻结束代码块，缩进代码后面会是段落：
*Example 83*
```
····foo
bar

<pre><code>foo
</code></pre>
<p>bar</p>
```

缩进代码块可以出现在其他块级元素前后：
*Example 84*
```
# Heading
····foo
Heading
------
····foo
----

<h1>Heading</h1>
<pre><code>foo
</code></pre>
<h2>Heading</h2>
<pre><code>foo
</code></pre>
<hr />
```

第一行缩进可以多于四个空格：
*Example 85*
```
········foo
····bar
    
<pre><code>····foo
bar
</code></pre>
```

缩进代码块不包含前后空行：
*Example 86*
```

····
····foo
····

<pre><code>foo
</code></pre>
```

尾随空格包含在代码块内：
*Example 87*
```
····foo··

<pre><code>foo··
</code></pre>
```

### 4.5 栅栏代码块

代码栅栏是至少三个连续的反引号字符（`` ` ``）或波浪号（` ~ `）的序列。 （波浪号和反引号不能混合。）栅栏代码块以代码栅栏开头，缩进不超过三个空格。

具有 _打开代码栅栏_ 的行可以包含一些文本在代码栅栏之后（可选），这是前导和尾随空格的修剪，并称为 _信息字符串_ 。信息字符串不能包含任何反引号字符（否则一些内联代码将被错误地解释为一个栅栏代码块的开始。）

代码块的内容由所有后续行组成，直到与代码块相同类型的 _关闭代码栅栏_ 以（反引号或波浪号）开始，并且至少与 _打开代码栅栏_ 一样多的反引号或波浪号）。如果打开代码栅栏缩进了N个空格，则从内容的每一行中删除最多N个缩进的空格（如果存在）（如果内容行不缩进，则保持不变，如果缩进少于N个空格，则删除所有缩进。）

_关闭代码栅栏_ 可以缩进最多三个空格，但后面只能跟空格（解析时忽略）。如果到达包含块（或文档）的末尾，并且没有找到关闭代码栅栏，则代码块包括 _打开代码栅栏_ 之后的所有行，直到包含块（或文档）的结尾。 （一个替代规范将要求在没有找到关闭代码栅栏的情况下进行回溯，但是这使得解析效率更低，而且这里描述的行为似乎没有什么不利。）

栅栏代码块会中断一个段落，前后不需要空行。

栅栏代码块的内容被视为文字文本，不会被视为内联。 _信息字符串_ 的第一个字通常用于指定代码示例的语言，并在`code`标签的`class`属性中呈现。但是，该规范并不要求对信息字符串的任何特定处理。

几个带反引号的例子：
*Example 88*
~~~
```
<
 >
```

<pre><code>&lt;
 &gt;
</code></pre>
~~~

带波浪号：
*Example 89*
```
~~~
<
 >
~~~

<pre><code>&lt;
 &gt;
</code></pre>
```

打开和关闭时所用字符必须一致：
*Example 90*
```
@bug
```

*Example 91*
```
@bug
```

关闭代码栅栏至少要和打开代码栅栏一样长：
*Example 92*
~~~
````
aaa
```
``````

<pre><code>aaa
```
</code></pre>
~~~

*Example 93*
```
~~~~
aaa
~~~
~~~~

<pre><code>aaa
~~~
</code></pre>
```

未关闭的代码块会一直延续到文本结束：
*Example 94*
~~~
```

<pre><code></code></pre>
~~~

*Example 95*
~~~
`````

```
aaa

<pre><code>
```
aaa
</code></pre>
~~~

*Example 96*
~~~
> ```
> aaa

bbb

<blockquote>
<pre><code>aaa
</code></pre>
</blockquote>
<p>bbb</p>
~~~

代码块的内容包含所有的空行：
*Example 97*
~~~
```

··
```

<pre><code>
··
</code></pre>
~~~

代码块可以为空：
*Example 98*
~~~
```
```

<pre><code></code></pre>
~~~

栅栏可以缩进，要是这样，如果内容存在缩进，将会删除等量的缩进长度：
*Example 99*
~~~
·```
·aaa
aaa
```

<pre><code>aaa
aaa
</code></pre>
~~~

*Example 100*
~~~
··```
aaa
··aaa
aaa
  ```
<pre><code>aaa
aaa
aaa
</code></pre>
~~~

*Example 101*
~~~
···```
···aaa
····aaa
··aaa
   ```

<pre><code>aaa
·aaa
aaa
</code></pre>
~~~

4 个空格将会产生缩进代码块：
*Example 102*
~~~
    ```
    aaa
    ```

<pre><code>```
aaa
```
</code></pre>
~~~

关闭栅栏可以缩进 0-3 个空格，而且不用和打开栅栏缩进长度一致：
*Example 103*
~~~
```
aaa
··```
 
<pre><code>aaa
</code></pre>
~~~


*Example 104*
~~~
···```
aaa
··```
 
<pre><code>aaa
</code></pre>
~~~

代码块没有关闭，因为缩进了 4 个空格：
*Example 105*
~~~
```
aaa
····```
 
<pre><code>aaa
····```
</code></pre>
~~~

代码栅栏中间不能包含空格：
*Example 106*
~~~
```·```
aaa
 
<p><code></code>
aaa</p>
~~~

*Example 107*
```
~~~~~~
aaa
~~~ ~~
 
<pre><code>aaa
~~~ ~~
</code></pre>
```

栅栏代码块可以中断段落，后边可以直接跟其他段落，不需要加空行：
*Example 108*
~~~
foo
```
bar
```
baz
 
<p>foo</p>
<pre><code>bar
</code></pre>
<p>baz</p>
~~~

其他块级元素也可以放在栅栏代码块的前后：
*Example 109*
```
foo
---
~~~
bar
~~~
# baz
 
<h2>foo</h2>
<pre><code>bar
</code></pre>
<h1>baz</h1>
```

_信息字符串_ 可以跟在 _打开代码栅栏_ 的后边，第一个单词通常用于指定代码示例的语言，并在`code`标签的`class`属性中呈现，并加上 `language-` 前缀。
*Example 110*
~~~
```ruby
def foo(x)
  return 3
end
```
 
<pre><code class="language-ruby">def foo(x)
  return 3
end
</code></pre>
~~~

*Example 111*
```
~~~~    ruby startline=3 $%@#$
def foo(x)
  return 3
end
~~~~~~~
 
<pre><code class="language-ruby">def foo(x)
  return 3
end
</code></pre>
```

*Example 112*
~~~
````;
````
 
<pre><code class="language-;"></code></pre>
~~~

代码栅栏字符为反引号时， _信息字符串_ 不能包含反引号： 
*Example 113*
~~~
``` aa ```
foo
 
<p><code>aa</code>
foo</p>
~~~

关闭代码字符串不能包含信息字符串：
*Example 114*
~~~
```
``` aaa
```
 
<pre><code>``` aaa
</code></pre>
~~~

### 4.6 HTML 块

HTML块是一组被视为原始HTML的行（并且不会在HTML输出中转义）。

有七种HTML块，可以通过它们的起始和结束条件来定义。 该块以满足起始条件的行（最多三个空格可选缩进）开头。 如果没有遇到满足结束条件的行，则以满足匹配结束条件的第一个后续行或文档或其他容器块的最后一行结束。 如果第一行满足起始条件和结束条件，则该块将仅包含该行。

1. __起始条件：__ 以`<script`，`<pre`，`<style`开头（大小写不敏感），后面跟`>`或直到行末
	__结束条件：__ 包含`</script>`，`</pre>`，`</style>`
	
2. __起始条件：__ 包含`<!--`
	__结束条件__ 包含`-->`

3. __起始条件：__ 包含`<?`
	__结束条件__ 包含`?>`
	
4. __起始条件：__ `<!`，后面跟大写的 ASCII 字母
	__结束条件__ 包含`>`
	
5. __起始条件：__ 包含`<![CDATA[	`
	__结束条件__ 包含`]]>`
	
6. __起始条件：__ `<`或`</`后面跟着 `address`，`article`，`aside`，`base`，`basefont`，`blockquote`，`body`，`caption`，`center`，`col`，`colgroup`，`dd`，`details`，`dialog`，`dir`，`div`，`dl`，`dt`，`fieldset`，`figcaption`，`figure`，`footer`，`form`，`frame`，`legend`，`li`，`link`，`main`，`menu`，`menuitem`，`meta`，`nav`，`noframes`，`ol`，`optgrouup`，`option`，`p`，`param`，`section`，`source`，`summary`，`table`，`tbody`，`td`，`tfoot`，`th`，`thead`，`title`，`tr`，`track` 其中之一，`ul`，行末为`>`或`/>`。
	__结束条件__ 后面跟着空行
	
7. __起始条件：__ 包含完整的起始标签和闭合标签
	__结束条件__ 后面跟着空行

除类型7之外的所有类型的HTML块可能会中断段落，类型7的块可能不会中断段落。（此限制旨在防止不必要的解释封装段落中的长标签作为启动HTML块。）

典型的类型6：






























