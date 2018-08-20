
# v2ray 部署在 openshift starter
openshift: 内存设置为256M，每 project 可配置 4 Pods。

Docker 镜像：wangyi2005/v2ray:latest，wangyi2005/v2ray:core版本号

环境变量： CONFIG_JSON（配置）、CERT_PEM（证书）、KEY_PEM（私钥）

用notepad++将上述变量中 \r\n 替换为\\n，变成一行，导入容器。

客户端： android Actinium、windows v2ray 可用同一个服务端。

websocket模式： Openshift3 route：edge，v2ray：ws+tls+vmess（不加密）
2.1 服务端配置文件
{
  "log": {
    "loglevel": "warning"
  },
  "inbound": {
    "protocol": "vmess",
    "port": 8080,
    "settings": {
      "clients": [
        {
          "id": "#########################",
          "alterId": 64,
          "security": "none"
        }
      ]
    },
    "streamSettings": {
      "network": "ws"
    }
  },
  "inboundDetour": [],
  "outbound": {
    "protocol": "freedom",
   "settings": {}
  }
}
2.2 pc端配置文件

{
  "log": {
    "loglevel": "warning"
  },
   "inbound": {
    "port": 8080,
    "protocol": "http",
    "listen": "127.0.0.1",
	"settings": {
    }
  },
  "inboundDetour": [],
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "##############.7e14.starter-us-west-2.openshiftapps.com", 
          "port": 443, 
          "users": [
            {
              "id": "#######################",
              "alterId": 64,
              "security": "none"
            }
          ]
        }
      ]
    },
	"streamSettings": {
      "network": "ws",
	  "security": "tls",
	  "tlsSettings": {
        "allowInsecure": true
	  }
    },
	"mux": {
        "enabled": true,
        "concurrency": 8
    }
  },
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    }
  ],
  "dns": {
    "servers": [
      "localhost"
    ]
  },
  "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "AsIs",
      "rules": [
        {
          "type": "chinasites",
          "outboundTag": "direct"
        },
        {
          "type": "field",
          "ip": [
            "0.0.0.0/8",
            "10.0.0.0/8",
            "100.64.0.0/10",
            "127.0.0.0/8",
            "169.254.0.0/16",
            "172.16.0.0/12",
            "192.0.0.0/24",
            "192.0.2.0/24",
            "192.168.0.0/16",
            "198.18.0.0/15",
            "198.51.100.0/24",
            "203.0.113.0/24",
            "::1/128",
            "fc00::/7",
            "fe80::/10"
          ],
          "outboundTag": "direct"
        },
        {
          "type": "chinaip",
          "outboundTag": "direct"
        }
      ]
    }
  }
}
2.3 android Actinium 0.10.2 配置文件

{
  "port": 10808,
  "log": {
    "loglevel": "warning"
  },
  "inbound": {
    "protocol": "socks",
    "listen": "127.0.0.1",
    "settings": {
      "auth": "noauth",
      "udp": true,
     "ip": "127.0.0.1"
    },
	"domainOverride": ["http", "tls"]
  },
  "inboundDetour": [],
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "#################.7e14.starter-us-west-2.openshiftapps.com", 
          "port": 443, 
          "users": [
            {
              "id": "####################",
              "alterId": 64,
              "security": "none"
            }
          ]
        }
      ]
    },
	"streamSettings": {
      "network": "ws",
	  "security": "tls",
	  "tlsSettings": {
        "allowInsecure": true
	  }
    },
	"mux": {
      "enabled": true
    }
  },
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    }
  ],
  "dns": {
    "servers": [
      "localhost"
    ]
  },
  "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "AsIs",
      "rules": [
	  {
        "type": "chinasites",
        "outboundTag": "direct"
      },
	  {
        "type": "chinaip",
        "outboundTag": "direct"
      }
    ]
    }
  },
  "transport": {},
  "#lib2ray": {
    "enabled": true,
    "listener": {
      "onUp": "#none",
      "onDown": "#none"
    },
    "env": [
      "V2RaySocksPort=10808"
    ],
    "render": [],
    "escort": [],
    "vpnservice": {
      "Target": "${datadir}tun2socks",
      "Args": [
        "--netif-ipaddr",
        "26.26.26.2",
        "--netif-netmask",
        "255.255.255.0",
        "--socks-server-addr",
        "127.0.0.1:$V2RaySocksPort",
        "--tunfd",
        "3",
        "--tunmtu",
        "1500",
        "--sock-path",
        "/dev/null",
        "--loglevel",
        "4",
        "--enable-udprelay"
      ],
      "VPNSetupArg": "m,1500 a,26.26.26.1,24 r,0.0.0.0,0"
    }
  }
}
