# 阿里云自建NAT实例配置

<!-- more -->

1. 配置vpc，添加路由表项，将0.0.0.0转发到nat ecs实例
2. 添加iptables规则
    ```shell
    iptables -t nat -A POSTROUTING -s 172.16.0.0/16 -o eth0 -j MASQUERADE
    ```
3. 保存iptables配置
    ```shell
    iptables-save
    ```
4. 开启IP转发
    ```shell
    echo "net.ipv4.ip_forward=1" >>  /etc/sysctl.conf && sysctl -p
    ```