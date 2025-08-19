# Bitwarden自托管搭建


关于密码管理工具也用过一些，从最基本的 Chrome 自带（**警告，有丢失风险，包括存储的通行密钥**），到 LastPass，再到 Keepass，最后选择使用 Bitwarden。

也许 1Password 体验会更好，但是我作为穷鬼，就不去打扰了。<!--more-->

> Bitwarden 是**一款免费且开源的密码管理工具**，它可以帮助用户安全地存储和管理他们的密码，并在不同设备之间同步。它通过加密的密码库来保护用户的敏感信息，即使是 Bitwarden 团队也无法访问用户的密码。

Bitwarden 官方的服务器我也用了很长时间，但是有个问题，现在大多数网站都支持 2FA 也就是两步验证，虽然免费的 Bitwarden 可以存储 2FA 密钥，但是填充要高级用户。

最后我选择使用它的自托管，也就是自己搭建一个 Bitwarden 服务端，这样数据不光可以完全由我们自己掌握，还可以使用 2FA 填充这样的高级功能。

## 准备工作

看了下[官方的配置要求](https://help.ppgg.in/self-hosting/deploy-and-configure/docker/linux-standard-deployment) ，2G RAM 起步，好家伙，这有点离谱；不过我又发现了 Vaultwarden 这个项目：

> Vaultwarden 是一个用于本地搭建 Bitwarden 服务器的第三方 Docker 项目。仅在部署的时候使用 Vaultwarden 镜像，桌面端、移动端、浏览器扩展等客户端均使用官方 Bitwarden 客户端。
>
> **Vaultwarden 很轻量，对于不希望使用官方的占用大量资源的自托管部署而言，它是理想的选择。**
>
> 除不支持 Bitwarden 官方企业版的部分功能外，其他大部分功能均**免费**支持。并跟随官方版本保持及时更新。
>
> 官方版使用 .Net 开发，使用 MSSQL 数据库，要求至少 2GB 内存；Vaultwarden 使用 Rust 编写，改用 SQLite 数据库（现在也支持 MySQL 和 PostgreSQL），运行时只需要 10M 内存，可以说对硬件基本没有要求。

本来是想放到我的服务器上，但是想了想我那服务器半年都可能不会关注一次，就是出问题也大概率发现不了；还是放家里的服务器安全些，虽然持久化都是加密存储，就算被攻破也不会出现明文泄露的情况。

反正部署完直接端口映射出去公网也能用。
## 部署

Vaultwarden 的部署非常简单，如果是和我一样简单使用的话，按照[官方的文档](https://rs.ppgg.in/container-image-usage/starting-a-container)几条命令就完事，毕竟是 Docker 部署。

``` sh
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
```

环境变量：

| 名称            | 含义                       |
| --------------- | -------------------------- |
| ADMIN_TOKEN     | 管理员后台使用，相当于密码 |
| SIGNUPS_ALLOWED | false                      |

其他的环境变量参考文档吧，我目前就用到这几个。

## 启用 SSL

这时候如果尝试访问 Web 服务，大概率进不去，因为要正常运行 Vaultwarden，几乎必须启用 HTTPS，这是因为 Bitwarden 网络密码库使用的  Web Crypto API，大多数浏览器只有在 HTTPS 环境下才能使用。

官方推荐是使用一个 Nginx 代理，HTTPS 由反代负责，还需要申请一个证书，可以使用 ACME 自动申请续订，或者自签一个倒是也能用。

但是我有个黑群晖，并且已经搞了 HTTPS，直接使用它内置的反代就可以了，如果你也有群晖或者其他 NAS 可以在 NAS 里找找，一般都有。

群晖是在：控制面板 - 登录门户 - 高级 - 反向代理服务器，来源一定要选 HTTPS，并且启用 HSTS；自定义标题中点新增选择 WebSocket 启用 WebSocket 支持。

### 群晖自动申请 Let's Encrypt 证书
我这里使用 Cloudflare 来做验证，需要去 Cloudflare 申请好 Token，有修改 DNS 的权限（域名页面的右下角可以看到 API 相关的信息）；然后进入 ssh 执行下面的命令：
``` sh
# 切换到root账户
sudo su
# 进入root home目录
cd ~
# 安装 ACME
curl https://get.acme.sh | sh -s email=xxxx@gmail.com --force

## 设置CF Account信息
export CF_Account_ID=""

## 设置CF Token信息
export CF_Token=""

# 进入/root/.acem.sh目录
cd /root/.acme.sh

# 申请证书，--server letsencrypt 指定申请 letsencrypt 证书
# -d example.com 指定要申请证书的域名是 example.com
# -d *.example.com 说明申请的证书是泛域名证书
./acme.sh --issue --server letsencrypt --dns dns_cf -d example.com -d *.example.com

# 若无意外，证书将会申请成功，并存放在.acme 目录下
# 安装证书
# acme 可以直接调用群晖本地的工具生成临时用户进行证书安装。

# 设置使用临时管理员账户
export SYNO_USE_TEMP_ADMIN=1
# 在/root/.acme.sh目录下执行命令部署证书
# example.com是前面申请证书的域名
./acme.sh --deploy --deploy-hook synology_dsm -d example.com
```
之后进入群晖的任务计划，新建一个任务，使用 root 执行，因为 Let's Encrypt 的证书只有 90 天有效期，需要选择合适的日期进行重复执行，例如每周一次检查。
输入执行命令：`/root/.acme.sh/acme.sh --cron --home /root/.acme.sh`

详细内容参考[这位大佬的文章](https://renyili.org/post/use_acme_update_ssl_certificates/)
## 开始使用

搭建完毕后，首先打开映射的 Web 服务，界面和 Bitwarden 官方基本是一样的，URL 后面加 `/admin` 进入管理页面，输入配置的 ADMIN_TOKEN 就进去了。

进去需要做两个事，因为关闭了自由注册，所以必须管理员邀请注册，但是邀请注册是通过邮件发送的，所以需要先配置一个邮件 SMTP 服务。

> 这里我因为有个阿里企业邮箱，就填了我自己的邮件服务，但是发现免费版本有概率会被 Spam，不是被放垃圾邮件就是发不出去；
>
> 后面找了下免费的类似服务，发现一个 [AhaSend](https://ahasend.com/)，已经搭建好了，后续可能会测试一下，免费的真是且用且珍惜，最终不会还是要自己搭吧，到时候再说。
>
> AhaSend 的免费套餐也够用了，在 Credentials 新建一个 SMTP 凭证就可以使用了。
>
> 不确定使用国内邮箱或者 Outlook 会不会有问题。

邮件服务配完直接填邮箱邀请用户就可以了，按照邮件链接注册，完成。

之后就可以从官方的 Bitwarden 导出密码库，导入我们自己的服务。

之后随便找个客户端或者浏览器扩展，添加新用户，选择自托管，输入我们的域名，登录完成。

强烈建议保存密码的时候手动编辑下名称和分组，一时默认爽，整理的时候头都大了。

## 导入 2FA

因为我平常使用的是 Google Authenticator，现在想转移到 Bitwarden 中一份，当初并没有保存密钥，而 Google Authenticator 中也不支持查看原始密钥；

后来发现了 [decodeGoogleOTP](https://github.com/Kuingsmile/decodeGoogleOTP) 这个项目，可以从 Google Authenticator 的转移二维码中解析原始数据；

将 Google Authenticator 导出的二维码图片保存，执行
``` sh
decodeGoogleOTP -i <input file> -j <output file>
```

执行完毕，得到解密后的 json，把密钥输入到 Bitwarden 就可以了。

记得清理或者保护好自己的 2FA。

