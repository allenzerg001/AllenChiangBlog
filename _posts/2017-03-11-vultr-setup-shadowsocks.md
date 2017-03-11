# 搭建shadowsocks

## 注意

考虑到后期使用锐速加速，其支持的系统内核版本有限。
这里系统版本最好选择cnetos 6的版本

## 安装shadowsocks

```
yum install m2crypto python-setuptools
easy_install pip
pip install shadowsocks
```

```
vi  /etc/shadowsocks.json
```


```
{
    "server":"0.0.0.0",
    "server_port":443,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"123456",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

如果你的服务端运行在CentOS 6及以上版本的话，这时候你配置好客户端上的shadowsocks后，发现还是连不上Google，原因是由于CentOS默认开启了防火墙，需要手动加入相应规则。

```
iptables -F
iptables -A INPUT -p tcp --dport 8388 -j ACCEPT
```

## 启动命令


```
ssserver -c /etc/shadowsocks.json
```

后台运行命令

```
nohup ssserver -c /etc/shadowsocks.json
```

## 锐速加速


```
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
```

[serverspeeder破解版](https://github.com/91yun/serverspeeder)

