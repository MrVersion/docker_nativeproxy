# docker_nativeproxy

# 两种启动方式
## 一、脚本安装
### bbr加速脚本
`cd /usr/src && wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh`


### nativeproxy 安装

`wget https://raw.githubusercontent.com/imajeason/nas_tools/main/NaiveProxy/install.sh && bash install.sh`

## 二、Docker 安装启动
### 服务端安装
1. 拉镜像
`docker pull pocat/naiveproxy`

2. 创目录
`mkdir -p /etc/naiveproxy /var/www/html /var/log/caddy`

3. 创建 caddyfile 配置文件
```
$ cat > /etc/naiveproxy/Caddyfile <<EOF
{
  admin off
  log {
      output file /var/log/caddy/access.log
      level INFO
  }
  servers :443 {
      protocols h1 h2 h3
  }
}

:80 {
  redir https://{host}{uri} permanent
}

:443, your_domain.com  # 修改成你自己的域名
tls your@email.com  # 邮箱地址随便填，符合格式就行
route {
  forward_proxy {
      basic_auth user_name your_password  # 将 user_name 和 your_password 改成自己的用户名和密码
      hide_ip
      hide_via
      probe_resistance rP7uSWkJpZzfg5g2Qr.com  # 改成像密码一样的域名
  }
  file_server {
      root /var/www/html
  }
}
EOF
```
4. docker 启动
`docker run --network host --name naiveproxy -v /etc/naiveproxy:/etc/naiveproxy -v /var/www/html:/var/www/html -v /var/log/caddy:/var/log/caddy -e PATH=/etc/naiveproxy/Caddyfile --restart=always -d pocat/naiveproxy`



### 客户端安装
1. 拉镜像
`docker pull pocat/naiveproxy:client`

2. 编写配置文件
```
{
"listen": "socks://127.0.0.1:1080",
"proxy": "https://user:pass@example.com"
}
```
3. 启动docker 
`docker run --network host --name naiveproxy -v /etc/naiveproxy:/etc/naiveproxy --restart=always -d pocat/naiveproxy:client`
