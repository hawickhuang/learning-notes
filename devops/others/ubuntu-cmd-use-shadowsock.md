# Ubuntu命令行使用shadowsocks代理置

<!-- more -->

1. 安装shadowsocks

    ```bash
    apt-get install shadowsocks
    ```

2. 配置shadowsocks, 在/etc/shadowsocks/config.json文件中输入以下内容

    ```json
    {
        "server":"",
        "server_port":8388,
        "local_address": "127.0.0.1",
        "local_port":1080,
        "password":"",
        "timeout":300,
        "method":"aes-256-cfb",
        "fast_open": true,
        "workers": 1
    }
    ```

3. 重启shadowsocks

    ```bash
    nohup sslocal -c /etc/shadowsocks/config.json &
    ```

4. 安装privoxy

    ```bash
    apt-get install privoxy
    ```

5. 配置privoxy，在/etc/privoxy/config文件中输入以下内容

    ```bash
    forward-socks5  /       127.0.0.1:1080  .
    listen-address  localhost:8118
    forward         172.16.*.*/    .
    forward         10.*.*.*/       .
    forward         127.*.*.*/      .
    forward         mirrors.cloud.aliyuncs.com/     .
    ```

6. 重启privoxy

    ```bash
    /etc/init.d/privoxy restart
    ```