现在机场大多数使用的是Dnsmasq将网站解析劫持到SNI proxy反向代理的页面上。

这里是使用WARP Client进行代理解锁Netflix等。
可以去cloudfalre官方页面有详细的安装流程和原理，不赘述。
个人认为官方socks这种代理方式更灵活且优雅。
https://developers.cloudflare.com/warp-client/setting-up/linux

这里写下我的配置过程

1. 注册客户端

   ```shell
   warp-cli register
   ```

2. 设置WARP代理模式

   ```shell
   warp-cli set-mode proxy
   ```

3. 连接WARP

   ```shell
   warp-cli connect
   ```

​		此时WARP会使用socks5本机代理127.0.0.1：40000

4. 打开warp always-on

   ```shell
   warp-cli enable-always-on
   ```

5. 测试socks代，理检查ip是否改变

   ```shell
   export ALL_PROXY=socks5://127.0.0.1:40000
   curl ifconfig.me
   ```

6. 打开x-ui的配置文件

   ```shell
   vim /usr/local/x-ui/bin/config.json	
   ```

7. 修改sniffing段落，inbounds要启动sniffing

   ```json
   "sniffing": {
       "enabled": true,
       "destOverride": ["http", "tls"]
   }
   ```

8. 修改outbounds和分流规则，我的配置是代理转发了netflix和openai

   ```json
    "outbounds": [
           {
               "tag": "default",
               "protocol": "freedom"
           },
           {
               "tag":"socks_out",
               "protocol": "socks",
               "settings": {
                   "servers": [
                        {
                           "address": "127.0.0.1",
                           "port": 40000
                       }
                   ]
               }
           }
       ],
       "routing": {
           "rules": [
               {
                   "type": "field",
                   "outboundTag": "socks_out",
                   "domain": [
                   	"geosite:netflix",
                   	"geosite:openai",
                   	"domain:statsigapi.net"
            		]
               },
               {
                   "type": "field",
                   "outboundTag": "default",
                   "network": "udp,tcp"
               }
           ]
       }
   ```

9. 更新geosite和geoip

   ```shell
   curl -s -L -o /usr/local/x-ui/bin/geosite.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat
   curl -s -L -o /usr/local/x-ui/bin/geoip.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat
   ```

10. 重新启动x-ui的xray

    ```shell
    x-ui
    ```



