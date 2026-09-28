### NNEXV 是一套面向多节点、多线路、多规则场景设计的中转与转发管理系统。系统由控制面板、节点运行时、隧道管理、规则管理、流量统计、用户与授权、支付注册、备份恢复、节点间 FastPath 高速传输层等模块组成。

NEXV 的核心目标不是单纯“把端口转发出去”，而是把节点管理、规则编排、批量运维、流量控制、可靠传输、商业授权和数据备份整合到同一套系统中。

当前正式版节点间高速传输层采用：

NEXV FastPath NXF/2

NXF/2 是面向节点间转发场景设计的持久化可靠 UDP 传输协议，重点优化复杂公网、高 RTT、高并发、多短连接、混合业务和跨地区中转场景。

## 功能介绍

* 节点与隧道管理
* 实时状态与流量监控
* 用户注册与登录
* 套餐商城与订单管理
* 钱包余额直接购买套餐
* 套餐自动续费
* GB / TB 流量限制
* 套餐数量购买
* 用户和订单管理
* 品牌名称、Logo、浏览器图标自定义
* 普通用户监控数据权限隔离

### 支付系统

支持：

* 微信
* 支付宝 EasyPay
* USDT-TRC20
* Polygon
* BSC（BEP20）
* Aptos

订单到账后自动进行支付核验，并自动发放对应权益。

## 商业授权系统

NEXV 同时集成商业授权系统。

首次部署后会进入授权页面，可以：

1. 直接输入授权码激活
2. 从授权页面购买授权

支持购买：

* 1 个月
* 3 个月
* 6 个月
* 12 个月

支付成功后，系统会从对应周期的授权码库存中，按照生成顺序自动分配授权码。

一键安装 Docker、Docker Compose V2，并设置开机自启：

```bash
apt update && apt install -y curl ca-certificates && \
curl -fsSL https://get.docker.com | sh && \
systemctl enable --now docker && \
docker --version && \
docker compose version && \
systemctl is-active docker && \
docker run --rm hello-world
```

# 安装 NEXV + Nginx + HTTPS

## 第一步：安装 NEXV

先将 NEXV 1.0.tar.gz 上传到 /root/

然后执行：

```bash
tar -xzf NEXV\ 1.0.tar.gz

cd NEXV\ 1.0

bash install.sh install
```
已有NEVX 升级：

```bash
tar -xzf NEXV\ 1.0.tar.gz

cd NEXV\ 1.0

./install.sh backup
./install.sh upgrade
```

安装完成后，按照安装程序提示继续操作。

---

## 第二步：安装 Nginx

执行：

```bash
apt update
apt install -y nginx certbot python3-certbot-nginx

rm -f /etc/nginx/sites-enabled/default
```

假设你的域名是：

`nexv.net`

创建 Nginx 配置：

```bash
cat > /etc/nginx/sites-available/nexv.net <<'EOF'
server {
    listen 80;
    server_name nexv.net;

    location / {
        proxy_pass http://127.0.0.1:6366;

        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_read_timeout 300;
        proxy_send_timeout 300;
    }
}
EOF
```

启用 Nginx 配置：

```bash
ln -sf /etc/nginx/sites-available/nexv.net /etc/nginx/sites-enabled/nexv.net

nginx -t && systemctl enable nginx && systemctl restart nginx
```

---

# 第三步：申请 HTTPS 证书

执行：

```bash
certbot --nginx -d nexv.net
```

如果询问是否将 HTTP 自动跳转到 HTTPS，选择：

```text
2
```

---

# 第四步：检查安装状态

执行：

```bash
nginx -t

systemctl status nginx --no-pager

ss -lntp | grep -E ':80|:443|:6366'
```

如果正常，可以使用浏览器访问：

**https://nexv.net/**

---

# 使用自己的域名

如果你的实际域名不是 `nexv.net`，例如：

`panel.example.com`

那么需要将上面所有：

`nexv.net`

替换为：

`panel.example.com`

尤其需要修改：

```nginx
server_name panel.example.com;
```

Nginx 配置文件：

```bash
/etc/nginx/sites-available/panel.example.com
```

HTTPS 申请：

```bash
certbot --nginx -d panel.example.com
```

---

## DNS 要求

申请 HTTPS 之前，请先将域名的 **A 记录** 指向这台服务器的公网 IPv4 地址。

例如：

| 类型 | 主机记录  | 记录值        |
| -- | ----- | ---------- |
| A  | `@`   | 服务器公网 IPv4 |
| A  | `www` | 服务器公网 IPv4 |

DNS 生效后，再执行 Certbot 申请 HTTPS，否则证书可能无法正常签发。

---

## 注意事项

* 请确保服务器已经开放 `80` 和 `443` 端口。
* NEXV 默认通过 `127.0.0.1:6366` 提供 Web 服务。
* Nginx 负责对外提供 HTTP / HTTPS 访问。
* HTTPS 配置完成后，建议使用 HTTPS 域名访问 NEXV。
* 授权码需要通过有效授权渠道取得。
