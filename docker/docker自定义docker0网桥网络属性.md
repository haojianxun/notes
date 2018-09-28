# docker自定义docker0网桥网络属性

vim /etc/docker/daemon.json

```json
{
    "bip": "192.168.1.5/24",
    "fixed-cidr": "10.20.0.0/16",
    "fixed-cidr-v6": "2001:db8::/64",
    "mtu": 1500,
    "default-gateway": "10.20.1.1",
    "dns": ["10.20.1.2","10.20.1.3"]
}
```

核心选项bip  bip即bridge ip之意

文档地址:https://docs.docker.com/network/bridge/#configure-the-default-bridge-network