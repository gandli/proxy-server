<h1 align="center">Welcome to proxy-server 👋</h1>
<p>
    <img alt="Version" src="https://img.shields.io/badge/version-0.0.1-blue.svg">
</p>

使用 docker 部署 v2ray（vmess ws tls）+ caddy（申请免费证书）代理服务器

```bash
proxy-server
├── caddy
│   └── Caddyfile
├── v2ray
│   ├── config.json
│   └── vpoint_vmess_freedom.json
├── docker-compose.yml
└── README.md
```

## 依赖

- [docker](https://www.docker.com/)
- [v2ray](https://github.com/v2ray/v2ray-core)
- [caddy](https://github.com/caddyserver/caddy)

## 配置

1. docker-compose.yml

```yaml
version: '3.3'

services:
  v2ray:
    container_name: proxy-server_v2ray_1
    image: v2ray/official
    restart: always
    volumes:
      - ./v2ray:/etc/v2ray
    ports:
      - "127.0.0.1:10086:10086"

  caddy:
    container_name: proxy-server_caddy_1
    image: caddy
    restart: always
    volumes:
      - ./caddy:/etc/caddy
      - ./caddy/data:/data
    ports:
      - "80:80"
      - "443:443"
```

2. caddy/Caddyfile

```
example.com {
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }

  reverse_proxy /ray localhost:10086 {
    transport http {
      tls
    }
  }
}
```

3. v2ray/config.json

```json
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 10086,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
            "alterId": 64
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/ray"
        },
        "security": "tls"
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

## 使用

在 proxy-server 文件夹下运行以下命令启动代理服务器：

```docker-compose up -d```
此命令将在后台启动 v2ray 和 caddy 服务。您可以使用以下命令查看服务状态：

```docker-compose ps```
如果服务启动成功，您应该会看到类似于以下输出：

```bash
    Name                  Command               State           Ports
------------------------------------------------------------------------------

proxy-server_caddy_1   /usr/bin/caddy run -conf ...   Up      0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
proxy-server_v2ray_1   /usr/bin/v2ray/v2ray -co ...   Up      127.0.0.1:10086->10086/tcp
```

Give a ⭐️ if this project helped you!

***
_This README was generated with ❤️ by [readme-md-generator](https://github.com/kefranabg/readme-md-generator)_
