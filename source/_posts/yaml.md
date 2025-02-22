---
title: yaml 文件简介
date: 2019-05-12 18:58:02
tags: yaml
categories:
- 技术
- 其他
---

## `YAML` 是什么

简单来说, 一个用于表示格式化数据的文本文件.
同类的还有 `xml`, `json`, `ini` , `plist`等.
通常用于书写配置文件, 例如 `Hexo` 的配置文件就是 `YAML` 格式的. `YAML` 格式文件一般后缀是 `yml`
<!-- more -->

## YAML 能做什么

能用于表达数组, 字典, 简单值.
例如这个 `json` 文件

``` json
{
    "string": "hello, world",
    "number": 12345,
    "array": [
        "a",
        "b",
        "c"
    ],
    "obj": {
        "key1": "value1",
        "key2": "value2"
    }
}
```

如果使用 `xml` 那么可能是这样

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<root>
    <string>hello, world</string>
    <number>12345</number>
    <array>a</array>
    <array>b</array>
    <array>c</array>
    <obj>
        <key1>value1</key1>
        <key2>value2</key2>
    </obj>
</root>
```

如果使用 `YAML` 表达可以是这样

``` yml
string: hello, world
number: 12345
array: 
    - a
    - b
    - c
obj: 
    key1: value1
    key2: value2
```

## `YAML` 的优点

### 文件尺寸小

这个看上面的例子就明白了, 丢掉了一大堆括号, 引号, 文件尺寸还是略微有减小的. 相比 `xml` 就很明显了.

### 不需要引号, 括号之类的边界符号

由于 `YAML` 的定位是编写配置文件, 而配置文件一般是人工编写的, 如果文件结构越简单, 那么编写起来就越容易, 当然也越不容易犯错..
相信编写过 `xml` 配置文件的同学深有体会.

### 层级结构, 易于阅读

`xml`, `json` 中的层级结构并不是必选的, 当被压缩过后, 缩短为一行, 往往需要使用工具将其格式化后才能阅读. 而 `YAML` 中基本上是强行缩进的(不是绝对的).

### 兼容json, 可以无缝迁移

如果你的旧有配置文件使用 `json` 格式, 那么如果要迁移到 `YAML` 时, 配置文件不做修改也可以正常使用, `YAML` 在实现上(1.2 版本), 将 `json` 作为一个子集来支持.

## `YAML` 语法介绍

### 编码

`YAML` 使用UTF-8或是 UTF-16编码.

### 关于缩进

`YAML` 使用缩进表示层级关系, 注意, 不能使用 TAB 来缩进.

``` yml
parent:
  child1: value1
  child2: value2
parent2:
        child1: value1 #空格数量并不重要, 但是同层级需要使用同一个空格数量.
        child2: value2
```

相当于

``` json
{
    "parent": {
        "child1": "value1",
        "child2": "value2"
    },
    "parent2": {
        "child1": "value1",
        "child2": "value2"
    }
}
```

### 注释

不同于 `json`, `YAML` 支持注释, 注释以 `#` 开头

``` yml
# 这是注释
# 多行需要每一行都加一个 #
key: value # #号后的字符会被忽略, 所以可以加在任何地方
```

### 列表(数组, 清单...)

列表使用 -空格 表示 `- `
也可以使用json 的方括号方式 `[obj1, obj2]` 使用 逗号 隔开成员, 注意, 空格前后的字符串会被忽略, 如果需要指定有空格的字符, 用引号包起来.

``` yml
arr1:
- a
- b
- c
arr2: [a, b,  c, "  d"]
arr3:
- name: zhangsan
  age: 20
- name: lisi
  age: 21
```

相当于

``` json
{
    "arr1": [
        "a",
        "b",
        "c"
    ],
    "arr2": [
        "a",
        "b",
        "c",
        "  d"
    ],
    "arr3": [
        {
            "name": "zhangsan",
            "age": 20
        },
        {
            "name": "lisi",
            "age": 21
        }
    ]
}
```

### 字典(哈希表, 散列表)

字典使用 冒号 空格`: ` 隔开key 和value, 空格是必须的, 如果值不是一个基本数据, 则换行后, 使用空格缩进表示, 也可以使用json 方式, 使用`{obj1: obj2}`

``` yml
obj1: value1
obj2:
    value2
obj3:
    subkey1: value
    subkey2:
      subkeyofsubkey2: value
obj4: {key: value}
```

相当于

``` json
{
    "obj1": "value1",
    "obj2": "value2",
    "obj3": {
        "subkey1": "value",
        "subkey2": {
            "subkeyofsubkey2": "value"
        }
    },
    "obj4": {
        "key": "value"
    }
}
```

有一点需要注意的是, `YAML` 中 key 是有顺序的. 例如, 这两个`YAML` 并不相等

``` yml
a: 123
b: 456
```

``` yml
b: 456
a: 123
```

### 基本数据

#### 基本字符串

字符串可以直接书写, 不需要加上引号, 如果必, 例如要在字符串中使用引号, #符号, 也可以使用双引号或是单引号, 引号内部, 可以使用 `\` 转义特殊字符

``` yml
key1: string
key2: "string"
key3: "string\nstring"
key4: "string #这后面的值可以正常使用"
key5: string #这后面的值会忽略
```

相当于

``` json
{
    "key1": "string",
    "key2": "string",
    "key3": "string\nstring",
    "key4": "string #这后面的值可以正常使用",
    "key5": "string"
}
```

#### 多行文本

在需要使用多行文本的时候, 除了使用转义符外, 还有其他选择.

##### 保存新行(Newlines preserved)符号 `|`

保存新行会保留每个换行符, 注意, 新行内容缩进不能小于第一行字符, 多出的缩进会被认为是空格.

``` yml
str: |
  line1
  line2
    two space
  
  last line
```

相当与json 文件

``` json
{
    "str": "line1\nline2\n  two space\n\nlast line\n"
}
```

渲染出来的结果

``` text
line1
line2
  two space

last line
```

##### 折叠新行(Newlines folded) `>`

与保留新行不同的是, 普通的折行不会保留为新行, 需要新行需要空一行才行
注意, 保持缩进一致, 否则可能会有意料之外的结果

``` yml
str: >
  line1,
  still line1
  
  new line
```

相当与json 文件

``` json
{
    "str": "line1, still line1\nnew line\n"
}
```

渲染出来的结果

``` text
line1, still line1
new line
```

### 锚点及引用

当处理大量数据的时候, 往往有些数据是可以直接重复使用的, 或是经过简单修改就可以重复使用, 在其他语言结构中, 我们基本上只能靠手动复制粘贴, 不仅是繁琐, 也增加了错误的可能性.
`YAML` 支持锚点和引用, 这样我们就可以方便的引用其他文件了.
使用 `&` 定义一个锚点, 相当于定义一个变量
使用 `*` 获取一个锚点, 相当取一个变量的值
使用 `<<: *锚点` 展开锚点, 相当于继承, 继承的节点可以覆盖同名的数据或是新增字段.
例如, 我们配置 3 个服务器节点

``` yml
api_server: &main_server
  ip: 192.168.100.10
  port: 80
img_server: *main_server # 直接引用
ftp_server:
  <<: *main_server
  port: 21 # 覆盖数据
web_server:
  <<: *main_server
  index_page: /www/root/index.html # 增加字段
```

相当于 json

``` json
{
    "api_server": {
        "ip": "192.168.100.10",
        "port": 80
    },
    "img_server": {
        "ip": "192.168.100.10",
        "port": 80
    },
    "ftp_server": {
        "ip": "192.168.100.10",
        "port": 21
    },
    "web_server": {
        "ip": "192.168.100.10",
        "port": 80,
        "index_page": "/www/root/index.html"
    }
}
```

### 强制类型制定

通常情况下, `YAML` 会自动推断类型, 某些情况下, 如果你需要强制制定数据的类型, 可以使用 类型制定来操作.
例如:

``` yml
a: 123                     # int
b: "123"                   # string
c: 123.0                   # float
d: !!float 123             # 强制声明为float
e: !!str 123               # 强制声明为 string
f: !!str true              # 强制声明为 string
g: true                    # bool 
h: true and false          # string 
```

### 多个子`YAML` 段

你可以在一个文件中, 包含若干个独立的`YAML`, 使用 `---` 区分就行了, 还可以使用`...` 来显示标识`YAML` 结束(可选), 这个特性可以用于网络通信中在不中断的链接的情况下, 完整的接收多个`YAML` 配置.

``` yml
---
# yaml 1
a: 123
...
---
# yaml 2
a: 456
```
