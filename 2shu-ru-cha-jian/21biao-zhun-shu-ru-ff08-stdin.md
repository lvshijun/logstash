# 标准输入\(Stdin\) {#%E6%A0%87%E5%87%86%E8%BE%93%E5%85%A5-stdin}

我们已经见过好几个示例使用`stdin`了。这也应该是 logstash 里最简单和基础的插件了。

所以，在这段中，我们可以学到一些未来每个插件都会有的一些方法。

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
input {
    stdin {
        add_field =
>
 {"key" =
>
 "value"}
        codec =
>
 "plain"
        tags =
>
 ["add"]
        type =
>
 "std"
    }
}

```

## 运行结果 {#%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C}

用上面的新`stdin`设置重新运行一次最开始的 hello world 示例。我建议大家把整段配置都写入一个文本文件，然后运行命令：`bin/logstash -f stdin.conf`。输入 "hello world" 并回车后，你会在终端看到如下输出：

```
{
       
"message"
 =
>
"hello world"
,
      
"@version"
 =
>
"1"
,
    
"@timestamp"
 =
>
"2014-08-08T06:48:47.789Z"
,
          
"type"
 =
>
"std"
,
          
"tags"
 =
>
 [
        [
0
] 
"add"

    ],
           
"key"
 =
>
"value"
,
          
"host"
 =
>
"raochenlindeMacBook-Air.local"

}

```

## 解释 {#%E8%A7%A3%E9%87%8A}

_type_和_tags_是 logstash 事件中两个特殊的字段。通常来说我们会在_输入区段_中通过_type_来标记事件类型 —— 我们肯定是提前能知道这个事件属于什么类型的。而_tags_则是在数据处理过程中，由具体的插件来添加或者删除的。

最常见的用法是像下面这样：

```
input {
    stdin {
        type =
>
 "web"
    }
}
filter {
    if [type] == "web" {
        grok {
            match =
>
 ["message", %{COMBINEDAPACHELOG}]
        }
    }
}
output {
    if "_grokparsefailure" in [tags] {
        nagios_nsca {
            nagios_status =
>
 "1"
        }
    } else {
        elasticsearch {
        }
    }
}

```

看起来蛮复杂的，对吧？

继续学习，你也可以写出来的。

