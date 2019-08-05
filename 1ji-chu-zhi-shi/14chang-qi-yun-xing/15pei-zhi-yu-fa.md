# 配置语法 {#%E9%85%8D%E7%BD%AE%E8%AF%AD%E6%B3%95}

Logstash 社区通常习惯用_shipper_，_broker_和_indexer_来描述数据流中不同进程各自的角色。如下图：

![](http://doc.yonyoucloud.com/doc/logstash-best-practice-cn/images/logstash-arch.jpg "logstash arch")

不过我见过很多运用场景里都没有用 logstash 作为_shipper_，或者说没有用 elasticsearch 作为数据存储也就是说也没有_indexer_。所以，我们其实不需要这些概念。只需要学好怎么使用和配置 logstash 进程，然后把它运用到你的日志管理架构中最合适它的位置就够了。

## 语法 {#%E8%AF%AD%E6%B3%95}

Logstash 设计了自己的 DSL —— 有点像 Puppet 的 DSL，或许因为都是用 Ruby 语言写的吧 —— 包括有区域，注释，数据类型\(布尔值，字符串，数值，数组，哈希\)，条件判断，字段引用等。

### 区段\(section\) {#%E5%8C%BA%E6%AE%B5-section}

Logstash 用`{}`来定义区域。区域内可以包括插件区域定义，你可以在一个区域内定义多个插件。插件区域内则可以定义键值对设置。示例如下：

```
input {
    stdin {}
    syslog {}
}

```

### 数据类型 {#%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B}

Logstash 支持少量的数据值类型：

* bool

```
debug => true
```

* string

```
host => "hostname"
```

* number

```
port => 514
```

* array

```
match => ["datetime", "UNIX", "ISO8601"]
```

* hash

```
options => {
    key1 => "value1",
    key2 => "value2"
}
```

**注意**：如果你用的版本低于 1.2.0，_哈希_的语法跟_数组_是一样的，像下面这样写：

```
match => [ "field1", "pattern1", "field2", "pattern2" ]
```

### 字段引用\(field reference\) {#%E5%AD%97%E6%AE%B5%E5%BC%95%E7%94%A8-field-reference}

字段是`Logstash::Event`对象的属性。我们之前提过事件就像一个哈希一样，所以你可以想象字段就像一个键值对。

_小贴士：我们叫它字段，因为 Elasticsearch 里是这么叫的。_

如果你想在 Logstash 配置中使用字段的值，只需要把字段的名字写在中括号`[]`里就行了，这就叫**字段引用**。

对于**嵌套字段**\(也就是多维哈希表，或者叫哈希的哈希\)，每层的字段名都写在`[]`里就可以了。比如，你可以从 geoip 里这样获取_longitude_值\(是的，这是个笨办法，实际上有单独的字段专门存这个数据的\)：

```
[geoip][location][0]
```

_小贴士：logstash 的数组也支持倒序下标，即`[geoip][location][-1]`可以获取数组最后一个元素的值。_

Logstash 还支持变量内插，在字符串里使用字段引用的方法是这样：

```
"the longitude is %{[geoip][location][0]}"
```

### 条件判断\(condition\) {#%E6%9D%A1%E4%BB%B6%E5%88%A4%E6%96%AD-condition}

Logstash从 1.3.0 版开始支持条件判断和表达式。

表达式支持下面这些操作符：

* equality, etc: ==, !=, &lt;, &gt;, &lt;=, &gt;=
* regexp: =~, !~
* inclusion: in, not in
* boolean: and, or, nand, xor
* unary: !\(\)

通常来说，你都会在表达式里用到字段引用。比如：

```
if "_grokparsefailure" not in [tags] {
} else if [status] !~ /^2\d\d/ and [url] == "/noc.gif" {
} else {
}

```

## 命令行参数 {#%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0}

Logstash 提供了一个 shell 脚本叫`logstash`方便快速运行。它支持一下参数：

* -e

意即_执行_。我们在 "Hello World" 的时候已经用过这个参数了。事实上你可以不写任何具体配置，直接运行`bin/logstash -e ''`达到相同效果。这个参数的默认值是下面这样：

```
input {
    stdin { }
}
output {
    stdout { }
}

```

* --config 或 -f

意即_文件_。真实运用中，我们会写很长的配置，甚至可能超过 shell 所能支持的 1024 个字符长度。所以我们必把配置固化到文件里，然后通过`bin/logstash -f agent.conf`这样的形式来运行。

此外，logstash 还提供一个方便我们规划和书写配置的小功能。你可以直接用`bin/logstash -f /etc/logstash.d/`来运行。logstash 会自动读取`/etc/logstash.d/`目录下所有的文本文件，然后在自己内存里拼接成一个完整的大配置文件，再去执行。

* --configtest 或 -t

意即_测试_。用来测试 Logstash 读取到的配置文件语法是否能正常解析。Logstash 配置语法是用 grammar.treetop 定义的。尤其是使用了上一条提到的读取目录方式的读者，尤其要提前测试。

* --log 或 -l

意即_日志_。Logstash 默认输出日志到标准错误。生产环境下你可以通过`bin/logstash -l logs/logstash.log`命令来统一存储日志。

* --filterworkers 或 -w

意即_工作线程_。Logstash 会运行多个线程。你可以用`bin/logstash -w 5`这样的方式强制 Logstash 为**过滤**插件运行 5 个线程。

_注意：Logstash目前还不支持输入插件的多线程。而输出插件的多线程需要在配置内部设置，这个命令行参数只是用来设置过滤插件的！_

**提示：Logstash 目前不支持对过滤器线程的监测管理。如果 filterworker 挂掉，Logstash 会处于一个无 filter 的僵死状态。这种情况在使用 filter/ruby 自己写代码时非常需要注意，很容易碰上`NoMethodError: undefined method '*' for nil:NilClass`错误。需要妥善处理，提前判断。**

* --pluginpath 或 -P

可以写自己的插件，然后用`bin/logstash --pluginpath /path/to/own/plugins`加载它们。

* --verbose

输出一定的调试日志。

_小贴士：如果你使用的 Logstash 版本低于 1.3.0，你只能用`bin/logstash -v`来代替。_

* --debug

输出更多的调试日志。

_小贴士：如果你使用的 Logstash 版本低于 1.3.0，你只能用`bin/logstash -vv`来代替。_

