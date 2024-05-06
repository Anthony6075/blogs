---
title: Linux sed命令使用方法
categories:
- computer_science
tags:
- linux
- shell
---


# 1. 缩写

`sed`是`stream editor`的缩写。

# 2. 常用选项

1. `-i`，原地修改输入文件

# 3. 常见使用场景

# 3.1 替换某个文本的所有出现

使用方法为`sed 's/<word1>/<word2>/g' input.txt`

1. `s`指substitution(替换)

1. `/`是分隔符，也可以是其他字符作分隔符，如`+`

1. `<word1>`是被替换的字符串

1. `<word2>`是要替换为的字符串

1. `g`表示global(全局替换)，不指定`g`的话只会替换每一行的第一次出现的`<word1>`。

# 3.2 删除指定行

行编号都是从1开始

- `sed '<n>d' input.txt`，删除第`<n>`行

- `sed '$d' input.txt`，删除最后一行

- `sed '<start>,<end>d' input.txt`，删除第`<start>`到`<end>`行(闭区间)

# 4. `sed`的正则表达式

`sed`默认使用Basic Regular Expression(BRE)语法，如果指定了`-E`或者`-r`或者`--regexp-extended`选项则使用Extended Regular Expression(ERE)语法。

BRE和ERE的唯一区别在于这些特殊字符的行为：`?`, `+`, `(`, `)`,`{`, `}`,`|`。在BRE中它们直接使用就是普通字符，加上`\`前缀之后才是特殊的元字符；而在ERE中正好相反，直接使用是元字符，加上`\`前缀变成普通字符。

Basic Regular Expression(BRE)语法常用规则如下：

1. `<char>`，普通字符`<char>`匹配自身

1. `*`，匹配0次或多次前面的项(单个普通字符，单个位于`\`后的特殊字符，单个`.`，单个grouped regexp，或者单个方括号表达式)

1. `\+`，类似`*`，但匹配1次或多次

1. `\?`，类似`*`，但匹配0次或1次

1. `.`，匹配任意单个字符，包括newline

1. `^`，表示行的开头

1. `$`，表示行的结束

1. `[<list>]`，匹配`<list>`中的任意单个字符，`<list>`也可以包括`char1-char2`用于指定其中的所有字符(闭区间)，`<list>`中也可以包括字符类(如`[:blank:]`表示空字符，即空格和tab)

1. `[^<list>]`，类似`[<list>]`，但匹配不在`<list>`中的任意单个字符

1. `\(<regexp>\)`，将`<regexp>`当做一个整体，构成一个grouped regexp(使用举例：整体应用`*`操作符等)

1. `<regexp1>\|<regexp2>`，匹配`<regexp1>`或者`<regexp2>`

1. `<regexp1><regexp2>`，匹配`<regexp1>`和`<regexp2>`的连接

1. `\n`，匹配newline

1. `\<char>`，匹配特殊字符`<char>`，`<char>`可以是`$`, `*`, `.`, `[`, `\`, `^`

# 5. 参考

1. [GNU sed文档](https://www.gnu.org/software/sed/manual/html_node/index.html#Top)

1. [GNU sed正则表达式](https://www.gnu.org/software/sed/manual/html_node/sed-regular-expressions.html#sed-regular-expressions)

1. [GNU sed BRE语法](https://www.gnu.org/software/sed/manual/html_node/BRE-syntax.html#BRE-syntax)