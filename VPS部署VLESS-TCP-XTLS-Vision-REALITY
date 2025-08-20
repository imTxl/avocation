## 1.准备工作

### 基础环境

- 系统：Debian 12
- CPU：1核
- 内存：1GB
- 磁盘：10GB



## 2.涉及网址

**Xray-GitHub官网：**https://github.com/XTLS

**Xray-Install 脚 本：**https://github.com/XTLS/Xray-install/blob/main/README_zh-Hans.md

**Xray-Config 配 置：**https://github.com/XTLS/Xray-examples/tree/main/VLESS-TCP-XTLS-Vision-REALITY



## 3.优化系统环境并部署Xray获取信息

### 3.1 升级系统包

```shell
apt update && apt upgrade -y
```

### 3.2 优化系统配置

#### 3.2.1 调整 xray 的进程限制（systemd）

```shell
# 执行命令（新创建即可）
tee /etc/systemd/system/xray.service.d/override.conf >/dev/null << 'EOF'
[Service]
# 文件描述符上限，1C1G 主机用 65535 足够
LimitNOFILE=65535
# 进程数上限
LimitNPROC=65535
EOF

# 刷新配置
systemctl daemon-reload 
# 重启xray服务
systemctl restart xray

# 检查是否生效
pid=$(pidof xray)
cat /proc/$pid/limits | grep "open files" # 如果输出是65535表示正常
```

#### 3.2.2 系统级 TCP 优化（适配小内存）

```shell
# 执行命令
tee /etc/sysctl.d/99-xray-light.conf >/dev/null << 'EOF'
# 接收队列 & SYN backlog（缩小到适中）
net.core.somaxconn = 4096
net.ipv4.tcp_max_syn_backlog = 4096
net.core.netdev_max_backlog = 8192

# 短连接优化
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_tw_reuse = 1

# 本机可用临时端口范围
net.ipv4.ip_local_port_range = 10240 65535

# 缓冲区（适配 1G 内存）
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
EOF

# 刷新配置
sysctl --system
```

#### 3.2.3 登录会话的 fd 上限（可选）

```shell
tee -a /etc/security/limits.d/99-nofile.conf >/dev/null << 'EOF'
* soft nofile 65535
* hard nofile 65535
EOF
```

### 3.3 安装Xray

```shell
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

### 3.4 移除Xray

```shell
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ remove
```

### 3.5 获取基础信息

```shell
# 获取UUID
xray uuid
25bfe0be-c804-4dbf-9068-xxxxxxxxxxxxxx

# 获取密钥对
xray x25519
Private key: CNqbT9xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Public key: MRjScT2xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 获取shortIds（可自行生成）
```



## 4.修改Xary配置文件

```json
// 编辑配置文件vi /usr/local/etc/xray/config.json
{
    "log": {
        "loglevel": "warning",
        "access": "/var/log/xray/access.log",
        "error": "/var/log/xray/error.log"
    },
    "inbounds": [
        {
            "port": 443, 
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "25bfe0be-c804-4dbf-9068-xxxxxxxxxxxxxx", // 获取到的UUID
                        "flow": "xtls-rprx-vision"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "dest": "www.apple.com:443", // 找个延时最低的443的域名
                    "serverNames": [
                        "www.apple.com"    // 与dest的域名地址保持一致
                    ],
                    "privateKey": "CNqbT9xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", // 获取密钥对的 Private key
                    "shortIds": [ // 可以不用动，如果想要修改，可以改成一个16位以内且双数的值，尽量在6位以上
                        "",
                        "012345678xxxxxxx"
                    ]
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls",
                    "quic"
                ],
                "routeOnly": true
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ],
    "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "domain": ["geosite:category-ads-all"],
        "outboundTag": "block"
      }
    ]
  }
}
```



## 5.检查配置并启动服务

### 检查配置

```shell
# 执行命令
xray -test -config /usr/local/etc/xray/config.json

>>> Xray 25.8.3 (Xray, Penetrates Everything.) bd86732 (go1.24.5 linux/amd64)
A unified platform for anti-censorship.
2025/08/15 05:22:28.135477 [Info] infra/conf/serial: Reading config: &{Name:/usr/local/etc/xray/config.json Format:json}
Configuration OK.
```

### 启动服务并查看状态

```shell
systemctl restart xray
systemctl status xray

>>> ● xray.service - Xray Service
     Loaded: loaded (/etc/systemd/system/xray.service; enabled; prese>
    Drop-In: /etc/systemd/system/xray.service.d
             └─10-donot_touch_single_conf.conf, override.conf
     Active: active (running) since Fri 2025-08-15 05:49:41 UTC; 1h 5>
       Docs: https://github.com/xtls
   Main PID: 42250 (xray)
      Tasks: 7 (limit: 1100)
     Memory: 59.0M
        CPU: 17.689s
     CGroup: /system.slice/xray.service
             └─42250 /usr/local/bin/xray run -config /usr/local/etc/x>

Aug 15 05:49:41 g0usfm5wen.bytevirt.com systemd[1]: Started xray.serv>
Aug 15 05:49:41 g0usfm5wen.bytevirt.com xray[42250]: Xray 25.8.3 (Xra>
Aug 15 05:49:41 g0usfm5wen.bytevirt.com xray[42250]: A unified platfo>
Aug 15 05:49:41 g0usfm5wen.bytevirt.com xray[42250]: 2025/08/15 05:49>
```

