# nmap端口扫描器的简单使用

[官方文档](https://nmap.org/man/zh/)

在我看来nmap所具有一定的不可替代性的，相比fscan或是其他的扫描器，虽然他更加缓慢对于漏洞的扫描相对特化过的扫描器效果没那么好，但是经久不衰的工具总是有他的独到之处

## 使用方法

可以简单也可以相当复杂

`nmap ip`

```shell
[neko@NekoArch ~]$ nmap 114.114.114.114
Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-01 10:26 CST
Nmap scan report for public1.114dns.com (114.114.114.114)
Host is up (0.46s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT   STATE SERVICE
21/tcp open  ftp
53/tcp open  domain
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 44.28 seconds
```

我个人更加常用的参数是这样的

`nmap -T4 -A -v IP`

-T 设置扫描速度

-A 设置全面系统扫描 攻击性扫描

-v 详细信息输出

![image-20231201103512333](/imgs/image-20231201103512333.png)

当我们需要快速发现局域网内的主机时，我们也可以使用-sP来快速ping扫描

```shell
[neko@NekoArch ~]$ nmap -sP 172.17.0.1/24
Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-01 10:46 CST
Nmap scan report for 172.17.0.1
Host is up (0.00028s latency).
Nmap scan report for 172.17.0.2
Host is up (0.00017s latency).
```

有时候-A的过多消息反而可能会让我们漏掉我们需要的消息，所以我们也可以通过指定来确定我们所需要的消息

例如上面我们扫描出了两台机器.1是我的本机.2是测试机器，我们想获取.2设备的系统与系统上运行的软件的版本信息，那么我们就可以使用-O -sV，前者指定了让nmap发起系统版本扫描，后者指定发起服务版本扫描

