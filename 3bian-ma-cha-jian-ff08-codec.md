# 编码插件\(Codec\) {#%E7%BC%96%E7%A0%81%E6%8F%92%E4%BB%B6-codec}

Codec 是 logstash 从 1.3.0 版开始新引入的概念\(_Codec_来自_Co_der/_dec_oder 两个单词的首字母缩写\)。

在此之前，logstash 只支持纯文本形式输入，然后以_过滤器_处理它。但现在，我们可以在_输入_期处理不同类型的数据，这全是因为有了**codec**设置。

所以，这里需要纠正之前的一个概念。Logstash 不只是一个`input | filter | output`的数据流，而是一个`input | decode | filter | encode | output`的数据流！_codec_就是用来 decode、encode 事件的。

codec 的引入，使得 logstash 可以更好更方便的与其他有自定义数据格式的运维产品共存，比如 graphite、fluent、netflow、collectd，以及使用 msgpack、json、edn 等通用数据格式的其他产品等。

事实上，我们在第一个 "hello world" 用例中就已经用过_codec_了 ——_rubydebug_就是一种_codec_！虽然它一般只会用在 stdout 插件中，作为配置测试或者调试的工具。

_小贴士：这个五段式的流程说明源自 Perl 版的 Logstash \(后来改名叫 Message::Passing 模块\)的设计。本书最后会对该模块稍作介绍。_

