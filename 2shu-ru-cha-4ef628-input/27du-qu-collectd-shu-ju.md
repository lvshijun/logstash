# collectd简述 {#collectd%E7%AE%80%E8%BF%B0}

collectd 是一个守护\(daemon\)进程，用来收集系统性能和提供各种存储方式来存储不同值的机制。它会在系统运行和存储信息时周期性的统计系统的相关统计信息。利用这些信息有助于查找当前系统性能瓶颈（如作为性能分析`performance analysis`）和预测系统未来的 load（如能力部署`capacity planning`）等

下面简单介绍一下: collectd的部署以及与logstash对接的相关配置实例

## collectd的安装 {#collectd%E7%9A%84%E5%AE%89%E8%A3%85}

### 解决依赖 {#%E8%A7%A3%E5%86%B3%E4%BE%9D%E8%B5%96}

```
rpm -ivh "http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm"
yum -y install libcurl libcurl-devel rrdtool rrdtool-devel perl-rrdtool rrdtool-prel libgcrypt-devel gcc make gcc-c++ liboping liboping-devel perl-CPAN net-snmp net-snmp-devel

```

### 源码安装collectd {#%E6%BA%90%E7%A0%81%E5%AE%89%E8%A3%85collectd}

```
wget http://collectd.org/files/collectd-5.4.1.tar.gz
tar zxvf collectd-5.4.1.tar.gz
cd collectd-5.4.1
./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var --libdir=/usr/lib --mandir=/usr/share/man --enable-all-plugins
make && make install

```

### 安装启动脚本 {#%E5%AE%89%E8%A3%85%E5%90%AF%E5%8A%A8%E8%84%9A%E6%9C%AC}

```
cp contrib/redhat/init.d-collectd /etc/init.d/collectd
chmod +x /etc/init.d/collectd

```

### 启动collectd {#%E5%90%AF%E5%8A%A8collectd}

```
service collectd start

```

## collectd的配置 {#collectd%E7%9A%84%E9%85%8D%E7%BD%AE}

以下配置可以实现对服务器基本的**CPU、内存、网卡流量、磁盘 IO 以及磁盘空间占用**情况的监控:

```
Hostname "host.example.com"
LoadPlugin interface
LoadPlugin cpu
LoadPlugin memory
LoadPlugin network
LoadPlugin df
LoadPlugin disk

<Plugin interface>
    Interface "eth0"
    IgnoreSelected false

</Plugin>
<Plugin network>
    <Server "10.0.0.1" "25826"> ## logstash 的 IP 地址和 collectd 的数据接收端口号
    </Server>
</Plugin>
```

## logstash的配置 {#logstash%E7%9A%84%E9%85%8D%E7%BD%AE}

以下配置实现通过 logstash 监听`25826`端口,接收从 collectd 发送过来的各项检测数据:

### 示例一： {#%E7%A4%BA%E4%BE%8B%E4%B8%80%EF%BC%9A}

```
input {
 collectd {
    port => 25826 ## 端口号与发送端对应
    type => collectd
}

```

### 示例二：（推荐） {#%E7%A4%BA%E4%BE%8B%E4%BA%8C%EF%BC%9A%EF%BC%88%E6%8E%A8%E8%8D%90%EF%BC%89}

```
udp {
    port => 25826
    buffer_size => 1452
    workers => 3          # Default is 2
    queue_size => 30000   # Default is 2000
    codec => collectd { }
    type => "collectd"
}

```

## 运行结果 {#%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C}

下面是简单的一个输出结果：

```
{
  "_index": "logstash-2014.12.11",
  "_type": "collectd",
  "_id": "dS6vVz4aRtK5xS86kwjZnw",
  "_score": null,
  "_source": {
    "host": "host.example.com",
    "@timestamp": "2014-12-11T06:28:52.118Z",
    "plugin": "interface",
    "plugin_instance": "eth0",
    "collectd_type": "if_packets",
    "rx": 19147144,
    "tx": 3608629,
    "@version": "1",
    "type": "collectd",
    "tags": [
      "_grokparsefailure"
    ]
  },
  "sort": [
    1418279332118
  ]
}

```

## 参考资料 {#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99}

* collectd支持收集的数据类型：[http://git.verplant.org/?p=collectd.git;a=blob;hb=master;f=README](http://git.verplant.org/?p=collectd.git;a=blob;hb=master;f=README)

* collectd收集各数据类型的配置参考资料：[http://collectd.org/documentation/manpages/collectd.conf.5.shtml](http://collectd.org/documentation/manpages/collectd.conf.5.shtml)

* collectd简单配置文件示例：[https://gist.github.com/untergeek/ab85cb86a9bf39f1fc6d](https://gist.github.com/untergeek/ab85cb86a9bf39f1fc6d)



