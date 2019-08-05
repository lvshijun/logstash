# JSON 编解码 {#json-%E7%BC%96%E8%A7%A3%E7%A0%81}

在上一章，已经讲过在 codec 中使用 JSON 编码。但是，有些日志可能是一种复合的数据结构，其中只是一部分记录是 JSON 格式的。这时候，我们依然需要在 filter 阶段，单独启用 JSON 解码插件。

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
filter {
    json {
        source => "message"
        target => "jsoncontent"
    }
}

```

## 运行结果 {#%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C}

```
{
    "@version": "1",
    "@timestamp": "2014-11-18T08:11:33.000Z",
    "host": "web121.mweibo.tc.sinanode.com",
    "message": "{\"uid\":3081609001,\"type\":\"signal\"}",
    "jsoncontent": {
        "uid": 3081609001,
        "type": "signal"
    }
}

```

## 小贴士 {#%E5%B0%8F%E8%B4%B4%E5%A3%AB}

如果不打算使用多层结构的话，删掉`target`配置即可。新的结果如下：

```
{
    "@version": "1",
    "@timestamp": "2014-11-18T08:11:33.000Z",
    "host": "web121.mweibo.tc.sinanode.com",
    "message": "{\"uid\":3081609001,\"type\":\"signal\"}",
    "uid": 3081609001,
    "type": "signal"
}
```



