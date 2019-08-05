# Key-Value 切分 {#key-value-%E5%88%87%E5%88%86}

在很多情况下，日志内容本身都是一个类似于 key-value 的格式，但是格式具体的样式却是多种多样的。logstash 提供`filters/kv`插件，帮助处理不同样式的 key-value 日志，变成实际的 LogStash::Event 数据。

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
filter {
    ruby {
        init => "@kname = ['method','uri','verb']"
        code => "event.append(Hash[@kname.zip(event['request'].split(' '))])"
    }
    if [uri] {
        ruby {
            init => "@kname = ['url_path','url_args']"
            code => "event.append(Hash[@kname.zip(event['uri'].split('?'))])"
        }
        kv {
            prefix => "url_"
            source => "url_args"
            field_split => "&"
            remove_field => [ "url_args", "uri", "request" ]
        }
    }
}

```

## 解释 {#%E8%A7%A3%E9%87%8A}

Nginx 访问日志中的`$request`，通过这段配置，可以详细切分成`method`,`url_path`,`verb`,`url_a`,`url_b`...

