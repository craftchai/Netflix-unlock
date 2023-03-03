# Netflix-unlock

奈飞解锁

操作系统：Ubuntu 22.04


安装x-ui：

bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)

关闭防火墙：ufw disable

检测是否解锁奈飞：

#项目地址：https://github.com/sjlleo/netflix-verify

#下载检测解锁程序
wget -O nf https://github.com/sjlleo/netflix-verify/releases/download/v3.1.0/nf_linux_amd64 && chmod +x nf

#执行
./nf

#通过代理执行
./nf -proxy socks5://127.0.0.1:30000





一、原生IP解锁





二、二级代理解锁



xray 配置模版：

//本地监听配置
{
    "listen": "127.0.0.1",
    "port": 30000, 
    "protocol": "socks", 
    "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
    }
}

//路由规则
{
    "type": "field",
    "outboundTag": "netflix_proxy",
    "domain": [
        "geosite:netflix",
        "geosite:disney"
    ]
}

//二级代理
//填入你自己的解锁节点配置







三、WARP代理解锁
官方客户端不支持arm架构VPS

安装WARP：

#官方教程：https://pkg.cloudflareclient.com/install

#安装WARP仓库GPG 密钥：
curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

#添加WARP源：
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

#更新APT缓存：
apt update

#安装WARP：
apt install cloudflare-warp

#注册WARP：
warp-cli register

#设置为代理模式（一定要先设置）：
warp-cli set-mode proxy

#连接WARP：
warp-cli connect

#查询代理后的IP地址：
curl ifconfig.me --proxy socks5://127.0.0.1:40000
xray 完整配置模版：

{
  "api": {
    "services": [
      "HandlerService",
      "LoggerService",
      "StatsService"
    ],
    "tag": "api"
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 62789,
      "protocol": "dokodemo-door",
      "settings": {
        "address": "127.0.0.1"
      },
      "tag": "api"
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "netflix_proxy",
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "127.0.0.1",
            "port": 40000
          }
        ]
      }
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "policy": {
    "system": {
      "statsInboundDownlink": true,
      "statsInboundUplink": true
    }
  },
  "routing": {
    "rules": [
      {
        "type": "field",
        "outboundTag": "netflix_proxy",
        "domain": [
          "geosite:netflix",
          "geosite:disney"
        ]
      },
      {
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "type": "field"
      },
      {
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "blocked",
        "type": "field"
      },
      {
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ],
        "type": "field"
      }
    ]
  },
  "stats": {}
}





四、DNS劫持解锁



确保端口未被占用：53/80/443

解锁机设置：

#项目地址：https://github.com/myxuchangbin/dnsmasq_sniproxy_install

#下载脚本
wget --no-check-certificate -O dnsmasq_sniproxy.sh https://raw.githubusercontent.com/myxuchangbin/dnsmasq_sniproxy_install/master/dnsmasq_sniproxy.sh

#执行安装脚本
bash dnsmasq_sniproxy.sh

#关闭本地DNS服务
systemctl stop systemd-resolved && systemctl disable systemd-resolved && rm -rf /etc/resolv.conf && echo 'nameserver 8.8.8.8'>/etc/resolv.conf
节点服务器设置：

#关闭本地DNS服务
systemctl stop systemd-resolved && systemctl disable systemd-resolved 

#设置DNS服务器
rm -rf /etc/resolv.conf && echo 'nameserver 146.56.99.155'>/etc/resolv.conf

#默认优先ipv6，直接禁用ipv6
echo "1" > /proc/sys/net/ipv6/conf/all/disable_ipv6 
