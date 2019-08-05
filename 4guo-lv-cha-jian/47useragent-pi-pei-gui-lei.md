# UserAgent 匹配归类 {#useragent-%E5%8C%B9%E9%85%8D%E5%BD%92%E7%B1%BB}

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
filter {
    useragent {
        target => "ua"
        source => "useragent"
    }
}
```



