## Xray手动安装教程

准备软件

- [Xshell 7 免费版](https://www.netsarang.com/en/free-for-home-school/)
- [WinSCP](https://winscp.net/eng/download.php)

重装系统（新手不建议开始就[网络重装系统](https://github.com/bohanyang/debi)，请去你VPS网站上操作，建议重装系统为Deian10或11）

- Debian 10
- Debian 11
- Ubuntu 18.04
- Ubuntu 20.04

开始安装

- 使用Xshell 7连接你的VPS
- 使用root用户登陆
- 请从步骤0开始按顺序操作
- 如果你已有SSL证书，将公钥文件改名为fullchain.pem，将私钥文件改名为privkey.pem，使用WinSCP连接你的VPS，将它们上传到/etc/ssl/private/目录，执行`chown -R nobody:nogroup /etc/ssl/private/`命令，跳过步骤1

0.安装curl

```
apt update -y && apt install -y curl
```

1.[申请免费的SSL证书](https://github.com/acmesh-official/acme.sh)

- 你先要购买一个域名，然后添加一个子域名，将子域名指向你VPS的IP。因为DNS解析需要一点时间，建议设置好了等5分钟，再执行下面的命令（每行命令依次执行）。你可以通过ping你的子域名，查看返回的IP是否正确。注意：将chika.example.com替换成你的子域名。

<pre>apt install -y socat

curl https://get.acme.sh | sh

alias acme.sh=~/.acme.sh/acme.sh

acme.sh --upgrade --auto-upgrade

acme.sh --set-default-ca --server letsencrypt

acme.sh --issue -d chika.example.com --standalone --keylength ec-256

acme.sh --install-cert -d chika.example.com --ecc \

--fullchain-file /etc/ssl/private/fullchain.pem \

--key-file /etc/ssl/private/privkey.pem

chown -R nobody:nogroup /etc/ssl/private/</pre>

- 备份已申请的SSL证书：使用WinSCP连接你的VPS，进入/etc/ssl/private/目录，下载公钥文件fullchain.pem和私钥文件privkey.pem

2.安装Nginx

- Debian 10/11
```
apt install -y gnupg2 ca-certificates lsb-release debian-archive-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring.gpg && printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/debian `lsb_release -cs` nginx\n" > /etc/apt/sources.list.d/nginx.list && printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" > /etc/apt/preferences.d/99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/systemd/system/nginx.service.d/override.conf
```

- Ubuntu 18.04/20.04
```
apt install -y gnupg2 ca-certificates lsb-release ubuntu-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring.gpg && printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx\n" > /etc/apt/sources.list.d/nginx.list && printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" > /etc/apt/preferences.d/99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/systemd/system/nginx.service.d/override.conf
```

3.安装Xray

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --version 1.5.0
```

4.1下载Nginx和Xray的配置文件（二选一）

- [VLESS-TCP-XTLS](https://github.com/chika0801/Xray-examples/tree/main/VLESS-TCP-XTLS)（推荐使用）

```
curl -Lo /etc/nginx/nginx.conf https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS/nginx.conf && curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS/config_server_dns_routing_enhance.json
```

- [VLESS-gRPC-TLS](https://github.com/chika0801/Xray-examples/tree/main/VLESS-gRPC-TLS)

```
curl -Lo /etc/nginx/nginx.conf https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-gRPC-TLS/nginx.conf && curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-gRPC-TLS/config_server_dns_routing_enhance.json
```

- 若更换了配置文件，需要重启Nginx和Xray，使其生效。

4.2下载[路由规则文件加强版](https://github.com/Loyalsoldier/v2ray-rules-dat)

```
curl -Lo /usr/local/share/xray/geosite.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat && curl -Lo /usr/local/share/xray/geoip.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat
```

5.重启Nginx和Xray

```
systemctl stop nginx && systemctl stop xray && systemctl start nginx && systemctl start xray
```

6.查看Nginx和Xray状态

```
systemctl status nginx && systemctl status xray
```

7.其它

- Xray配置文件路径`/usr/local/etc/xray/config.json` Nginx配置文件路径`/etc/nginx/nginx.conf` 路由规则文件目录`/usr/local/share/xray`

- 修改服务器配置文件的方法：使用WinSCP连接你的VPS，进入/usr/local/etc/xray/目录，双击config.json文件编辑，找到"id": "chika"，修改后并保存，然后重启Nginx和Xray，使其生效。

- SSL证书是每90天自动更新，更新时需要使用80端口，因此在Nginx的配置文件中，没有监听80端口。申请免费证书，每周限制5次，超过次数会报错，[具体限制规则](https://letsencrypt.org/zh-cn/docs/rate-limits/)

<details><summary>自动更新路由规则文件加强版命令</summary>

```
printf "* 7 * * * /root/update_geodata.sh\n" > /root/update_geodata && crontab /root/update_geodata && printf "curl -sSLo /usr/local/share/xray/geosite.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat\ncurl -sSLo /usr/local/share/xray/geoip.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat\nsleep 2\nsystemctl restart xray\n" > /root/update_geodata.sh && chmod +x /root/update_geodata.sh
```
</details>

<details><summary>手动更新SSL证书命令</summary>

```
acme.sh --renew -d chika.example.com --force --ecc
```
</details>

## Windows系统电脑科学上网的方法

1.下载和设置v2rayN

[打开链接](https://github.com/2dust/v2rayN/releases) 点击最新版本栏里的“▸ Assets `4`”，找到名为`v2rayN-Core.zip`的链接并下载。把压缩包解压，找到`v2rayN-Core`文件夹里的`v2rayN.exe`并双击运行。

- 点击 设置 — 参数设置 — v2rayN设置，勾选“更新Core时忽略Geo文件”，将“Core类型”改为“Xray_core”，确定。
- 点击 设置 — 路由设置，将“域名解析策略”改为“IPIfNonMatch”，取消勾选“启用路由高级功能”，将“域名匹配算法”改为“mph”，点击“基础功能”，点击“一键导入基础规则”，确定，确定。
- 右键点击屏幕右下角的v2rayN图标，点击“系统代理 — 自动配置系统代理”。

2.在v2rayN中添加服务器

<details><summary>VLESS-TCP-XTLS</summary>

点击“服务器 — 添加[VLESS]服务器”，按下图所示填写，地址填写你的子域名(例如chika.example.com)

![VLESS-TCP-XTLS](https://user-images.githubusercontent.com/88967758/132801053-cc8b3aee-5da8-45d5-9e23-115f3b766e52.jpg)</details>

<details><summary>VLESS-gRPC-TLS</summary>

点击“服务器 — 添加[VLESS]服务器”，按下图所示填写，地址填写你的子域名(例如chika.example.com)

![VLESS-gRPC](https://user-images.githubusercontent.com/88967758/132800221-1e67083c-6d38-4f00-8f24-38ae688f3d09.jpg)</details>

点击“检查更新
- 点击“Xray-Core — 是否下载? — 是”。
- 点击“Update GeoSite — 是否下载? — 是”。
- 点击“Update GeoIP — 是否下载? — 是”。

## 安卓系统手机科学上网的方法

1.在电脑上下载v2rayNG

[打开链接](https://github.com/2dust/v2rayNg/releases) 点击最新版本栏里的“▸ Assets”，找到名为v2rayNG_1.x.x_arm64-v8a.apk的文件并下载，通过数据线连接手机和电脑，将下载的文件复制到你的手机内。

2.在你的手机中打开文件管理APP，找到你刚才复制进来的apk文件，并安装它。

3.在手机上进入v2rayNG，点击左上角的“≡”打开菜单，点击“设置”，“域名策略”改为“IPIfNonMatch”，“预定义规则”改为“绕过局域网及大陆地址”。

4.返回v2rayNG主界面，点击右上角的“+”，点击“扫描二维码”。在电脑上双击屏幕右下角的v2rayN图标，打开v2rayN的主界面，点击选中要使用的服务器整行，点击“分享”，用手机扫描屏幕上的二维码。

5.在手机上点击右下角的圆形图标，第一次会弹出一个网络连接请求的对话框，点确定即可，这时圆形图标变绿，提示“服务启动成功”。

## 参考

[Nginx](http://nginx.org/en/linux_packages.html)

[Xray-install](https://github.com/XTLS/Xray-install)

[v2ray-fhs-install-wiki](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates)

[acme.sh](https://github.com/acmesh-official/acme.sh)

[配置文件](https://github.com/lxhao61/integrated-examples)
