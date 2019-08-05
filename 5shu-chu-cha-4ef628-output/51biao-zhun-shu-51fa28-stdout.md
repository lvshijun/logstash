# 标准输出\(Stdout\) {#%E6%A0%87%E5%87%86%E8%BE%93%E5%87%BA-stdout}

和之前_inputs/stdin_插件一样，_outputs/stdout_插件也是最基础和简单的输出插件。同样在这里简单介绍一下，作为输出插件的一个共性了解。

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
output {
    stdout {
        codec => rubydebug
        workers => 2
    }
}
```

## 解释 {#%E8%A7%A3%E9%87%8A}

输出插件统一具有一个参数是`workers`。Logstash 为输出做了多线程的准备。

其次是 codec 设置。codec 的作用在之前已经讲过。可能除了`codecs/multiline`，其他 codec 插件本身并没有太多的设置项。所以一般省略掉后面的配置区段。换句话说。上面配置示例的完全写法应该是：

```
output {
    stdout {
        codec => rubydebug {
        }
        workers => 2
    }
}
```

单就_outputs/stdout_插件来说，其最重要和常见的用途就是调试。所以在不太有效的时候，加上命令行参数`-vv`运行，查看更多详细调试信息。

