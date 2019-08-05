# 读取网络数据\(TCP\) {#%E8%AF%BB%E5%8F%96%E7%BD%91%E7%BB%9C%E6%95%B0%E6%8D%AE-tcp}

未来你可能会用 Redis 服务器或者其他的消息队列系统来作为 logstash broker 的角色。不过 Logstash 其实也有自己的 TCP/UDP 插件，在临时任务的时候，也算能用，尤其是测试环境。

_小贴士：虽然`LogStash::Inputs::TCP`用 Ruby 的`Socket`和`OpenSSL`库实现了高级的 SSL 功能，但 Logstash 本身只能在`SizedQueue`中缓存 20 个事件。这就是我们建议在生产环境中换用其他消息队列的原因。_

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
input {
    tcp {
        port => 8888
        mode => "server"
        ssl_enable => false
    }
}

```

## 常见场景 {#%E5%B8%B8%E8%A7%81%E5%9C%BA%E6%99%AF}

目前来看，`LogStash::Inputs::TCP`最常见的用法就是配合`nc`命令导入旧数据。在启动 logstash 进程后，在另一个终端运行如下命令即可导入数据：

```
# nc 127.0.0.1 8888 < olddata
```

这种做法比用`LogStash::Inputs::File`好，因为当 nc 命令结束，我们就知道数据导入完毕了。而用 input/file 方式，logstash 进程还会一直等待新数据输入被监听的文件，不能直接看出是否任务完成了。

