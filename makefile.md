# 参考

[使用函数 — 跟我一起写Makefile 1.0 文档 (seisman.github.io)](https://seisman.github.io/how-to-write-makefile/functions.html)

## make参数

### -r,--no-builtin-rules

取消所有隐含规则（除了后缀规则）

### -f

指定makefile文件

### -B`, `--always-make

认为所有的目标都需要更新（重编译）。

### -C

自动打开-w

### -w,--print-directory

指定-C会自动打开。在make过程中输出一些有用的信息，让你看到目前的工作目录

### -e

使用环境变量中定义的变量覆盖makefile和make的命令行参数指定的变量	

### -n`, `--just-print`, `--dry-run`, `--recon

不执行参数，这些参数只是打印命令，不管目标是否更新，把规则和连带规则下的命令打印出来，但不执行，这些参数对于我们调试makefile很有用处。

### -t`, `--touch

这个参数的意思就是把目标文件的时间更新，但不更改目标文件。也就是说，make假装编译目标，但不是真正的编译目标，只是把目标变成已编译过的状态。

## 隐含规则

### 自动生成.o文件

```makefile
foo : foo.o bar.o
    cc –o foo foo.o bar.o $(CFLAGS) $(LDFLAGS)
```

自动生成以下内容

```makefile
foo.o : foo.c
    cc –c foo.c $(CFLAGS)
bar.o : bar.c
    cc –c bar.c $(CFLAGS)
```

```makefile
foo.o : foo.p
#如果像这样只写依赖不写命令，还是会自动生成类似上面的内容
#但是换成模式的可以防止自动生成
%.o:%.s
```

### 双后缀规则

```makefile
.c.o:
    $(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<
```

把后缀 `.hack` 和 `.win` 加入后缀列表中的末尾。

```makefile
.SUFFIXES:              # 删除默认的后缀
.SUFFIXES: .c .o .h   # 定义自己的后缀
```

## 模式规则

### 定义

模式规则并不会匹配目录下所有文件，而只是会匹配生成目标文件所需的却没有自定义生成规则的文件

目标中的模式的 `%` 决定了依赖目标中 `%` 的样子。例如有一个模式规则如下：

```makefile
%.o : %.c ; <command ......>;
```

```makefile
%.o : %.c
    $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

可以使用**$@**和**$<**

## 静态模式



## 变量

### 可能存在 的问题

#### 变量定义时和随后的注释之间的空格

这些空格会被保留下来，如果这样使用这个变量会出错
`dir=..    #sdaf`
`dirT:`
      `cd $(dir)/subdir`

### 定义

#### =

特点：前面的变量可以使用后面的变量
缺点：可能花很长时间调用函数，可能递归套用(变量可以使用另一个变量定义)

#### :=

特点：前面的变量不能使用后面的变量

#### ?=

特点：如果这个变量先前已经被定义，则不在定义。如果没有定义过，则定义

#### +=

为变量追加值

### 自动变量

#### $<

表示依赖目标的第一个值

模式规则中表示所有依赖目标的**一个**值

```makefile
%.o : %.c
    $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

#### $@

模式规则中表示所有的目标的**一个**值

```makefile
%.o : %.c
    $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

#### $%

如果目标是归档成员，则该变量表示目标的归档成员名称。

#### $?

所有的依赖文件，以空格分开，这些依赖文件的修改日期比目标的创建日期晚。   **一般用这个，只有当源文件有修改的时候才重新编译**

#### $^

所有的依赖文件，以空格分开，不包含重复的依赖文件。

#### $+

所有的依赖文件，以空格分开，并以出现的先后为序，可能包含重复的依赖文件。

#### $*

不包含扩展名的目标文件名称

#### 加D或F

### 特殊用法

#### 变量替换

替换匹配到的结尾字符串

**样例**：
$(var:.o=.c)
$(var:%.o=%.c)

#### 变量嵌套

**样例**：
a=b
b=c
$($(a))

#### 变量连接

样例：
$a_$b

#### 通过变量的值使用函数

`func:=sort`
`numbers:=a b c d e`
`$($(func) $(numbers))`

### override

有的变量是make的命令行参数，使用普通的方法无法修改，可以使用override关键字进行修改

### 环境变量

默认情况下环境变量（**MAKEFILE**）会被makefile文件和make的命令行参数指定的变量所覆盖
使用make的使用使用**-e**可以让环境变量的值覆盖这些值

**样例：**
`override CFLAGS += -g`
`override CFLAGS := -g`

### 目标变量

对指定的target设置“局部变量”

**样例：**
`var=5`
`prog:var=3`
`prog:s1`
    `@echo $(var)`
`s1:`
    `@echo $(var)`
**结果：**
make：3 3
make s1：5

### 模式变量

```makefile
%.o : CFLAGS = -O
```

## 条件判断

### 样例

```makefile
ifeq ($(CC),gcc)
    $(CC) -o foo $(objects) $(libs_for_gcc)
else
    $(CC) -o foo $(objects) $(normal_libs)
endif
```

### 条件关键字

ifeq
ifneq
ifdef
ifndef

## 转义

### $$

表示$

### \%

## 执行指令

可以直接使用命令行的命令

### @

隐藏指令被调用时候的提示输出(同bat)



## 子系统

### 调用子系统：

subsystem:
	$(MAKE) -C subdir

subsystem:
	cd subdir&$(MAKE)

### 传递变量至子系统:

#### SHELL和MAKEFLAGES变量始终被传递下去

export var=1

var=1
export var

unexport var

## 函数

### 定义

**不允许出现\<Tab\>**

也可以称作**多行变量**

define
...
endef

### 使用

```makefile
$(<function> <arguments>)
```

参数之间用逗号隔开

### 自带函数

#### foreach

语法

```makefile
$(foreach <var>,<list>,<text>)
```

例子：

```makefile
names := a b c d
files := $(foreach n,$(names),$(n).o)
结果：a.o b.o c.o d.o
```

#### if

语法

```makefile
$(if <condition>,<then-part>,<else-part>)
$(if <condition>,<then-part>)
```

#### call

唯一的可以用来创建/使用带参数自定义函数的方法

语法

```makefile
$(call <expression>,<parm1>,<parm2>,...,<parmn>)
#<expression>是自己定义的
```

样例：

```makefile
reverse =  $(1) $(2)
foo = $(call reverse,a,b)
```

```makefile
reverse =  $(2) $(1)
foo = $(call reverse,a,b)
```

#### origin

用来查询变量的来源

```makefile
$(origin <variable>)
#注意， `<variable>` 是变量的名字，不应该是引用。所以你最好不要在 `<variable>` 中使用
```

**返回值：**

undefined：如果 `<variable>` 从来没有定义过，origin函数返回这个值 `undefined`

default：如果 `<variable>` 是一个默认的定义，比如“CC”这个变量，这种变量我们将在后面讲述。

environment：如果 `<variable>` 是一个环境变量，并且当Makefile被执行时， `-e` 参数没有被打开。

file：如果 `<variable>` 这个变量被定义在Makefile中。

command line：如果 `<variable>` 这个变量是被命令行定义的。

override：如果 `<variable>` 是被override指示符重新定义的。

automatic：如果 `<variable>` 是一个命令运行中的自动化变量。关于自动化变量将在后面讲述。

#### shell

调用shell

```makefile
contents := $(shell cat foo)
files := $(shell echo *.c)
```

#### 字符串操作函数

#### subst

```makefile
$(subst <form>,<to>,<text>)
```

- 功能：把字串 `<text>` 中的 `<from>` 字符串替换成 `<to>` 。
- 返回：函数返回被替换过后的字符串。

#### patsubst

```makefile
$(patsubst <pattern>,<replacement>,<text>)
```



- 功能：查找 `<text>` 中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式 `<pattern>` ，如果匹配的话，则以 `<replacement>` 替换。这里， `<pattern>` 可以包括通配符 `%` ，表示任意长度的字串。如果 `<replacement>` 中也包含 `%` ，那么， `<replacement>` 中的这个 `%` 将是 `<pattern>` 中的那个 `%` 所代表的字串。（可以用 `\` 来转义，以 `\%` 来表示真实含义的 `%` 字符）
- 返回：函数返回被替换过后的字符串。

- 示例：

  > ```makefile
  > $(patsubst %.c,%.o,x.c.c bar.c)
  > ```

#### strip

```makefile
$(strip <string>)
```

- 功能：去掉 `<string>` 字串中开头和结尾的空字符。
- 返回：返回被去掉空格的字符串值。

#### findstring

```makefile
$(findstring <find>,<in>)
```

- 功能：在字串 `<in>` 中查找 `<find>` 字串。
- 返回：如果找到，那么返回 `<find>` ，否则返回空字符串。

#### filter

```makefile
$(filter <pattern...>,<text>)
```

- 功能：以 `<pattern>` 模式过滤 `<text>` 字符串中的单词，保留符合模式 `<pattern>` 的单词。可以有多个模式。
- 返回：返回符合模式 `<pattern>` 的字串。

#### filter-out

```makefile
$(filter-out <pattern...>,<text>)
```

- 功能：以 `<pattern>` 模式过滤 `<text>` 字符串中的单词，去除符合模式 `<pattern>` 的单词。可以有多个模式。
- 返回：返回不符合模式 `<pattern>` 的字串。

#### sort

```makefile
$(sort <list>)
```

- 功能：给字符串 `<list>` 中的单词排序（升序）。
- 返回：返回排序后的字符串。

#### word

```makefile
$(word <n>,<text>)
```

- 功能：取字符串 `<text>` 中第 `<n>` 个单词。（从一开始）
- 返回：返回字符串 `<text>` 中第 `<n>` 个单词。如果 `<n>` 比 `<text>` 中的单词数要大，那么返回空字符串。

#### wordlist

```makefile
$(wordlist <ss>,<e>,<text>)
```

- 功能：从字符串 `<text>` 中取从 `<ss>` 开始到 `<e>` 的单词串。 `<ss>` 和 `<e>` 是一个数字。
- 返回：返回字符串 `<text>` 中从 `<ss>` 到 `<e>` 的单词字串。如果 `<ss>` 比 `<text>` 中的单词数要大，那么返回空字符串。如果 `<e>` 大于 `<text>` 的单词数，那么返回从 `<ss>` 开始，到 `<text>` 结束的单词串。

#### words

```makefile
$(words <text>)
```

- 功能：统计 `<text>` 中字符串中的单词个数。
- 返回：返回 `<text>` 中的单词数。

#### firstword

```makefile
$(firstword <text>)
```

- 功能：取字符串 `<text>` 中的第一个单词。
- 返回：返回字符串 `<text>` 的第一个单词。

#### 目录操作函数

#### dir

```makefile
$(dir <names...>)
```

- 功能：从文件名序列 `<names>` 中取出目录部分。目录部分是指最后一个反斜杠（ `/` ）之前的部分。如果没有反斜杠，那么返回 `./` 。
- 返回：返回文件名序列 `<names>` 的目录部分。

- 示例： `$(dir src/foo.c hacks)` 返回值是 `src/ ./` 。

#### notdir

```makefile
$(notdir <names...>)
```

- 功能：从文件名序列 `<names>` 中取出非目录部分。非目录部分是指最後一个反斜杠（ `/` ）之后的部分。
- 返回：返回文件名序列 `<names>` 的非目录部分。
- 示例: `$(notdir src/foo.c hacks)` 返回值是 `foo.c hacks` 。

#### suffix

```makefile
$(suffix <names...>)
```

- 功能：从文件名序列 `<names>` 中取出各个文件名的后缀。
- 返回：返回文件名序列 `<names>` 的后缀序列，如果文件没有后缀，则返回空字串。
- 示例： `$(suffix src/foo.c src-1.0/bar.c hacks)` 返回值是 `.c .c`。

#### basename

```makefile
$(basename <names...>)
```

- 功能：从文件名序列 `<names>` 中取出各个文件名的前缀部分。
- 返回：返回文件名序列 `<names>` 的前缀序列，如果文件没有前缀，则返回空字串。
- 示例： `$(basename src/foo.c src-1.0/bar.c hacks)` 返回值是 `src/foo src-1.0/bar hacks` 。

#### addsuffix

```makefile
$(addsuffix <suffix>,<names...>)
```

- 功能：把后缀 `<suffix>` 加到 `<names>` 中的每个单词后面。
- 返回：返回加过后缀的文件名序列。
- 示例： `$(addsuffix .c,foo bar)` 返回值是 `foo.c bar.c` 。

#### addprefix

```makefile
$(addprefix <prefix>,<names...>)
```

- 功能：把前缀 `<prefix>` 加到 `<names>` 中的每个单词后面。
- 返回：返回加过前缀的文件名序列。
- 示例： `$(addprefix src/,foo bar)` 返回值是 `src/foo src/bar` 。

#### join

```makefile
$(join <list1>,<list2>)
```

- 功能：把 `<list2>` 中的单词对应地加到 `<list1>` 的单词后面。如果 `<list1>` 的单词个数要比 `<list2>` 的多，那么， `<list1>` 中的多出来的单词将保持原样。如果 `<list2>` 的单词个数要比 `<list1>` 多，那么， `<list2>` 多出来的单词将被复制到 `<list1>` 中。
- 返回：返回连接过后的字符串。
- 示例： `$(join aaa bbb , 111 222 333)` 返回值是 `aaa111 bbb222 333` 。

### 控制make的函数

#### error

会报错退出

```makefile
$(error <text ...>)
```

```makefile
ifdef ERROR_001
    $(error error is $(ERROR_001))
endif
```

#### warning

```makefile
$(warning <text ...>)
```

