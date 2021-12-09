# fhs-install-v2ray

> 欲查阅以简体中文撰写的介绍，请访问：[README.zh-Hans-CN.md](README.zh-Hans-CN.md)

> Bash script for installing V2Ray in operating systems such as Debian / CentOS / Fedora / openSUSE that support systemd

該腳本安裝的文件符合 [Filesystem Hierarchy Standard (FHS)](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)：

```
installed: /usr/local/bin/v2ray
installed: /usr/local/bin/v2ctl
installed: /usr/local/share/v2ray/geoip.dat
installed: /usr/local/share/v2ray/geosite.dat
installed: /usr/local/etc/v2ray/config.json
installed: /var/log/v2ray/
installed: /var/log/v2ray/access.log
installed: /var/log/v2ray/error.log
installed: /etc/systemd/system/v2ray.service
installed: /etc/systemd/system/v2ray@.service
```

## 重要提示

**不推薦在 docker 中使用本專案安裝 v2ray，請直接使用 [官方映象](https://github.com/v2fly/docker)。**  
如果官方映象不能滿足您自定義安裝的需要，請以**復刻並修改上游 dockerfile 的方式來實現**。  

本專案**不會為您自動生成配置檔案**；**只解決使用者安裝階段遇到的問題**。其他問題在這裡是無法得到幫助的。  
請在安裝完成後參閱 [文件](https://www.v2fly.org/) 瞭解配置檔案語法，並自己完成適合自己的配置檔案。過程中可參閱社群貢獻的 [配置檔案模板](https://github.com/v2fly/v2ray-examples)  
（**提請您注意這些模板複製下來以後是需要您自己修改調整的，不能直接使用**）

## 使用

* 該腳本在執行時會提供 `info` 和 `error` 等信息，請仔細閱讀。

### 安裝和更新 V2Ray

```
// 安裝執行檔和 .dat 資料檔
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

### 安裝最新發行的 geoip.dat 和 geosite.dat

```
// 只更新 .dat 資料檔
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)
```

### 移除 V2Ray

```
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --remove
```

### 解決問題

* 「[不安裝或更新 geoip.dat 和 geosite.dat](https://github.com/v2fly/fhs-install-v2ray/wiki/Do-not-install-or-update-geoip.dat-and-geosite.dat)」。
* 「[使用證書時權限不足](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates)」。
* 「[從舊腳本遷移至此](https://github.com/v2fly/fhs-install-v2ray/wiki/Migrate-from-the-old-script-to-this)」。
* 「[將 .dat 文檔由 lib 目錄移動到 share 目錄](https://github.com/v2fly/fhs-install-v2ray/wiki/Move-.dat-files-from-lib-directory-to-share-directory)」。
* 「[使用 VLESS 協議](https://github.com/v2fly/fhs-install-v2ray/wiki/To-use-the-VLESS-protocol)」。

> 若您的問題沒有在上方列出，歡迎在 Issue 區提出。

**提問前請先閱讀 [Issue #63](https://github.com/v2fly/fhs-install-v2ray/issues/63)，否則可能無法得到解答並被鎖定。**

## 貢獻

請於 [develop](https://github.com/v2fly/fhs-install-v2ray/tree/develop) 分支進行，以避免對主分支造成破壞。

待確定無誤後，兩分支將進行合併。


宝塔方式：

准备一个域名和一台vps，并将域名解析到vps。Freenom 可以注册免费域名

搭建好宝塔并安装nginx

宝塔和nginx完成以后，回到vps SSH窗口

执行命令
```
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```
执行完后，回到宝塔面板，

打开宝塔进入：/usr/local/etc/v2ray  

编辑config.json这个文件，打开文件后先清空里面的内容，再粘贴下面代码进去并保存
```
{
  "log": {
    "loglevel": "info",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
    {
      "port": 10000,
      "listen":"127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "000fe881-b655-4212-b804-b00f9970d5aa",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/happy"
        }
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

代码中的000fe881-b655-4212-b804-b00f9970d5aa可以变更一下。比如换几个数字。相当于是个密码。但是格式必须相同(小火箭里的UUID指的就是这串代码)

然后宝塔新建一个网站(域名是文章开头你解析的)，

首先申请SSL证书(这步不用说了吧)

然后点击配置文件，在配置文件最顶部添加以下代码

```
# 定义变量
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}
```

然后添加配置到宝塔配置文件中，以下代码
```
    #v2配置文件
location /happy {
    proxy_pass       http://127.0.0.1:10000;
    proxy_redirect             off;
    proxy_http_version         1.1;
    proxy_set_header Upgrade   $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host      $http_host;
    }
```
保存

回到vps SSH窗口

启动v2ray
```
systemctl start v2ray
```
设置开机自启
```
systemctl enable v2ray
```
OK，V2ray服务端已全部完成

v2ray其他常用命令
## 启动
```
systemctl start v2ray
```
## 停止
```
systemctl stop v2ray
```
## 重启
```
systemctl restart v2ray
```
## 开机自启
```
systemctl enable v2ray
```
##卸载v2ray
先停止v2ray
```
systemctl stop v2ray
systemctl disable v2ray
```
再执行一键移除
```
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --remove
```

安卓、PC、V2rayNG配置：

选择Vmess方式

地址（alterId）：域名  
端口：443  
用户（id）：000fe881-b655-4212-b804-b00f9970d5aa  
额外ID：64  
加密方式：auto  
传输方式：ws  
伪装域名：域名（或者为空）  
path：/happy  
底层传输安全（tls）：tls  
跳过证书验证：false  

参考案例地址：https://iooqp.cn/10725.html  
