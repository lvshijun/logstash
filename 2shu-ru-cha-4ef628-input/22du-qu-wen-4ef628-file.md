# 读取文件\(File\) {#%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6-file}

分析网站访问日志应该是一个运维工程师最常见的工作了。所以我们先学习一下怎么用 logstash 来处理日志文件。

Logstash 使用一个名叫_FileWatch_的 Ruby Gem 库来监听文件变化。这个库支持 glob 展开文件路径，而且会记录一个叫_.sincedb_的数据库文件来跟踪被监听的日志文件的当前读取位置。所以，不要担心 logstash 会漏过你的数据。

_sincedb 文件中记录了每个被监听的文件的 inode, major number, minor number 和 pos。_

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
input
    file {
        path => ["/var/log/*.log", "/var/log/message"]
        type => "system"
        start_position => "beginning"
    }
}
```

## 解释 {#%E8%A7%A3%E9%87%8A}

有一些比较有用的配置项，可以用来指定_FileWatch_库的行为：

* discover\_interval

logstash 每隔多久去检查一次被监听的`path`下是否有新文件。默认值是 15 秒。

* exclude

不想被监听的文件可以排除出去，这里跟`path`一样支持 glob 展开。

* sincedb\_path

如果你不想用默认的`$HOME/.sincedb`\(Windows 平台上在`C:\Windows\System32\config\systemprofile\.sincedb`\)，可以通过这个配置定义 sincedb 文件到其他位置。

* sincedb\_write\_interval

logstash 每隔多久写一次 sincedb 文件，默认是 15 秒。

* stat\_interval

logstash 每隔多久检查一次被监听文件状态（是否有更新），默认是 1 秒。

* start\_position

logstash 从什么位置开始读取文件数据，默认是结束位置，也就是说 logstash 进程会以类似`tail -F`的形式运行。如果你是要导入原有数据，把这个设定改成 "beginning"，logstash 进程就从头开始读取，有点类似`cat`，但是读到最后一行不会终止，而是继续变成`tail -F`。

## 注意 {#%E6%B3%A8%E6%84%8F}

1. 通常你要导入原有数据进 Elasticsearch 的话，你还需要[filter/date](http://doc.yonyoucloud.com/doc/logstash-best-practice-cn/filter/date.html)插件来修改默认的"@timestamp" 字段值。稍后会学习这方面的知识。
2. _FileWatch_只支持文件的**绝对路径**，而且会不自动递归目录。所以有需要的话，请用数组方式都写明具体哪些文件。
3. _LogStash::Inputs::File_只是在进程运行的注册阶段初始化一个_FileWatch_对象。所以它不能支持类似 fluentd 那样的`path => "/path/to/%{+yyyy/MM/dd/hh}.log"`写法。达到相同目的，你只能写成 `path => "/path/to/*/*/*/*.log"`。
4. `start_position`
   仅在该文件从未被监听过的时候起作用。如果 sincedb 文件中已经有这个文件的 inode 记录了，那么 logstash 依然会从记录过的 pos 开始读取数据。所以重复测试的时候每回需要删除 sincedb 文件。
5. 因为 windows 平台上没有 inode 的概念，Logstash 某些版本在 windows 平台上监听文件不是很靠谱。windows 平台上，推荐考虑使用 nxlog 作为收集端，参阅本书[稍后](http://doc.yonyoucloud.com/doc/logstash-best-practice-cn/ecosystem/nxlog.html)章节。



