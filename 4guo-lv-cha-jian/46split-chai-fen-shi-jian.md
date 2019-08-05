# split 拆分事件 {#split-%E6%8B%86%E5%88%86%E4%BA%8B%E4%BB%B6}

上一章我们通过 multiline 插件将多行数据合并进一个事件里，那么反过来，也可以把一行数据，拆分成多个事件。这就是 split 插件。

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
filter {
    split {
        field => "message"
        terminator => "#"
    }
}

```

## 运行结果 {#%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C}

这个测试中，我们在 intputs/stdin 的终端中输入一行数据："test1\#test2"，结果看到输出两个事件：

```
{
    "@version": "1",
    "@timestamp": "2014-11-18T08:11:33.000Z",
    "host": "web121.mweibo.tc.sinanode.com",
    "message": "test1"
}
{
    "@version": "1",
    "@timestamp": "2014-11-18T08:11:33.000Z",
    "host": "web121.mweibo.tc.sinanode.com",
    "message": "test2"
}

```

## 重要提示 {#%E9%87%8D%E8%A6%81%E6%8F%90%E7%A4%BA}

split 插件中使用的是 yield 功能，其结果是 split 出来的新事件，会直接结束其在 filter 阶段的历程，也就是说写在 split 后面的其他 filter 插件都不起作用，进入到 output 阶段。所以，一定要保证**split 配置写在全部 filter 配置的最后**。

使用了类似功能的还有 clone 插件。

_注：从 logstash-1.5.0beta1 版本以后修复该问题。_

