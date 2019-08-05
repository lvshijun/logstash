# GeoIP 地址查询归类 {#geoip-%E5%9C%B0%E5%9D%80%E6%9F%A5%E8%AF%A2%E5%BD%92%E7%B1%BB}

GeoIP 是最常见的免费 IP 地址归类查询库，同时也有收费版可以采购。GeoIP 库可以根据 IP 地址提供对应的地域信息，包括国别，省市，经纬度等，对于可视化地图和区域统计非常有用。

## 配置示例 {#%E9%85%8D%E7%BD%AE%E7%A4%BA%E4%BE%8B}

```
filter {
    geoip {
        source => "message"
    }
}

```

## 运行结果 {#%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C}

```
{
       "message" => "183.60.92.253",
      "@version" => "1",
    "@timestamp" => "2014-08-07T10:32:55.610Z",
          "host" => "raochenlindeMacBook-Air.local",
         "geoip" => {
                      "ip" => "183.60.92.253",
           "country_code2" => "CN",
           "country_code3" => "CHN",
            "country_name" => "China",
          "continent_code" => "AS",
             "region_name" => "30",
               "city_name" => "Guangzhou",
                "latitude" => 23.11670000000001,
               "longitude" => 113.25,
                "timezone" => "Asia/Chongqing",
        "real_region_name" => "Guangdong",
                "location" => [
            [0] 113.25,
            [1] 23.11670000000001
        ]
    }
}

```

## 配置说明 {#%E9%85%8D%E7%BD%AE%E8%AF%B4%E6%98%8E}

GeoIP 库数据较多，如果你不需要这么多内容，可以通过`fields`选项指定自己所需要的。下例为全部可选内容：

```
filter {
    geoip {
        fields => ["city_name", "continent_code", "country_code2", "country_code3", "country_name", "dma_code", "ip", "latitude", "longitude", "postal_code", "region_name", "timezone"]
    }
}

```

需要注意的是：`geoip.location`是 logstash 通过`latitude`和`longitude`额外生成的数据。所以，如果你是想要经纬度又不想重复数据的话，应该像下面这样做：

filter { geoip { fields =&gt; \["city\_name", "country\_code2", "country\_name", "latitude", "longitude", "region\_name"\] remove\_field =&gt; \["\[geoip\]\[latitude\]", "\[geoip\]\[longitude\]"\] } } \`\`\`

## 小贴士 {#%E5%B0%8F%E8%B4%B4%E5%A3%AB}

geoip 插件的 "source" 字段可以是任一处理后的字段，比如 "client\_ip"，但是字段内容却需要小心！geoip 库内只存有公共网络上的 IP 信息，查询不到结果的，会直接返回 null，而 logstash 的 geoip 插件对 null 结果的处理是：**不生成对应的 geoip.字段。**

所以读者在测试时，如果使用了诸如 127.0.0.1, 172.16.0.1, 182.168.0.1, 10.0.0.1 等内网地址，会发现没有对应输出！

