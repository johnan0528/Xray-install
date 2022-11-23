## [Xray](https://xtls.github.io/Xray-docs-next/)手动安装教程

准备软件

- [Xshell 7 免费版](https://www.xshell.com/zh/free-for-home-school)
- [WinSCP](https://winscp.net/eng/docs/lang:chs)

重装系统

- Debian 10
- Debian 11
- Ubuntu 18.04
- Ubuntu 20.04

开始安装

- 使用Xshell 7登录你的VPS
- 使用root用户登陆

0. 已有SSL证书

- 如果你之前用acme申请了SSL证书，将证书文件改名为`fullchain.cer`，将私钥文件改名为`private.key`，使用WinSCP登录你的VPS，将它们上传到`/etc/ssl/private/`目录，执行下面的命令，跳过步骤1。

```
chown -R nobody:nogroup /etc/ssl/private/
```

- [使用证书时权限不足](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates-zh-Hans-CN)

#### 1.用[acme](https://github.com/acmesh-official/acme.sh)申请 SSL 证书

- 你先要购买一个域名，然后添加一个子域名，将子域名指向你VPS的IP。等待5-10分钟，让DNS解析生效。你可以通过ping你的子域名，查看返回的IP是否正确。确认DNS解析生效后，再执行下面的命令（每行命令依次执行）。
- 注意：将`chika.example.com`替换成你的子域名。

<details><summary>点击查看详细步骤</summary> 

```
apt update -y          #Debian/Ubuntu 命令
apt install -y curl    #Debian/Ubuntu 命令
apt install -y socat    #Debian/Ubuntu 命令
```

```
yum update -y          #CentOS 命令
yum install -y curl    #CentOS 命令
yum install -y socat    #CentOS 命令
```

```
alias acme.sh=~/.acme.sh/acme.sh
```
-安装 Acme 脚本
```
curl https://get.acme.sh | sh
```
-1、安装 Acme 脚本之后，请先执行下面的命令（下面的邮箱为你的邮箱）
```
~/.acme.sh/acme.sh --register-account -m xxxx@xxxx.com
```
-80 端口空闲的验证申请
-如果你还没有运行任何 web 服务, 80 端口是空闲的, 那么 Acme.sh 还能假装自己是一个 WebServer, 临时监听在 80 端口, 完成验证
```
~/.acme.sh/acme.sh  --issue -d mydomain.com   --standalone
```
-Nginx 的方式验证申请
-这种方式需要你的服务器上面已经部署了 Nginx 环境，并且保证你申请的域名已经在 Nginx 进行了 conf 部署。（被申请的域名可以正常被打开）
```
~/.acme.sh/acme.sh --issue  -d mydomain.com   --nginx
```
-安装证书到指定文件夹
-注意, 默认生成的证书都放在安装目录下: ~/.acme.sh/, 请不要直接使用此目录下的证书文件。
-正确的使用方法是使用 --install-cert 命令,并指定目标位置, 然后证书文件会被copy到相应的位置，比如下面的代码
```
~/.acme.sh/acme.sh --installcert -d mydomain.com --key-file /root/private.key --fullchain-file /root/cert.crt
```
-上面的 /root/private.key 以及 /root/cert.crt 是把密钥和证书安装到 /root 目录，并改名为 private.key 和 cert.crt

-升级 Acme.sh 到最新版本
```
~/.acme.sh/acme.sh --upgrade
```
-如果你不想手动升级, 可以开启自动升级:
```
~/.acme.sh/acme.sh  --upgrade  --auto-upgrade
```
-之后, acme.sh 就会自动保持更新了.
</details>

- 备份已申请的SSL证书：使用WinSCP登录你的VPS，进入`/etc/ssl/private/`目录，下载证书文件`fullchain.cer`和私钥文件`private.key`。
- SSL证书有效期是90天，每隔60几天会自动更新。[速率限制](https://letsencrypt.org/zh-cn/docs/rate-limits/)，超过次数会报错。
- 需要下载VPS上申请的证书到本地,改名之后再通过Winscp登录vps上传到所需要的目录里面.
2. 安装[Nginx](http://nginx.org/en/linux_packages.html)

- Debian 10/11

```
apt install -y gnupg2 ca-certificates lsb-release debian-archive-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring.gpg && printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" > /etc/apt/sources.list.d/nginx.list && printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900" > /etc/apt/preferences.d/99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && printf "[Service]\nExecStartPost=/bin/sleep 0.1" > /etc/systemd/system/nginx.service.d/override.conf

```

- Ubuntu 18.04/20.04

```
apt install -y gnupg2 ca-certificates lsb-release ubuntu-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring.gpg && printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" > /etc/apt/sources.list.d/nginx.list && printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900" > /etc/apt/preferences.d/99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && printf "[Service]\nExecStartPost=/bin/sleep 0.1" > /etc/systemd/system/nginx.service.d/override.conf
```

3. 安装[Xray](https://github.com/XTLS/Xray-core/releases)

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

4. 下载[配置](https://github.com/chika0801/Xray-examples)

- [VLESS-TCP-XTLS-Vision](https://github.com/chika0801/Xray-examples/tree/main/VLESS-TCP-XTLS-Vision) with fallbacks function

```
curl -Lo /etc/nginx/nginx.conf https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS/nginx.conf && curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS-Vision/config_server_fallbacks.json
```

5. 启动程序

```
systemctl restart nginx && systemctl restart xray
```

```
systemctl status nginx && systemctl status xray
```

| 项目 | |
| :--- | :--- |
| 配置 | /usr/local/etc/xray/config.json |
| 证书 | /etc/ssl/private/fullchain.cer |
| 私钥 | /etc/ssl/private/private.key |
| 路由规则文件 | /usr/local/share/xray/ |
| 查看日志 | journalctl -u xray --output cat -e |
| 实时日志 | journalctl -u xray --output cat -f |

## v2rayN配置指南

1. 点击[v2rayN](https://github.com/2dust/v2rayN/releases)进入下载页面。找到最新版本，点击**▸ Assets**展开列表，找到名为**v2rayN-Core.zip**的链接并下载。把压缩包解压，启动v2rayN.exe。

2. 点击 **设置 — 路由设置** 取消勾选“启用路由高级功能”，点击“基础功能”，点击“一键导入基础规则”，确定，确定。

3. 点击 **服务器 — 添加[VLESS]服务器**

| 选项 | 值 |
| :--- | :--- |
| 地址(address) | VPS的IP |
| 端口(prot) | 443 |
| 用户ID(id) | chika |
| 流控(flow) | xtls-rprx-vision |
| 传输协议(network) | tcp |
| 传输层安全(tls) | tls |
| SNI | 证书中包含的域名 |
| uTLS | chrome |

4. 点击 **检查更新 — Update Geo files** 在信息栏确认有提示“下载 GeoFile: geoip 成功”，“下载 GeoFile: geoip 成功”。

5. 点击服务器列表中刚才新增的服务器，**按回车键**设为活动服务器。

6. 右键点击屏幕右下角的v2rayN图标，点击 **系统代理 — 自动配置系统代理**。
