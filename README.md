# IOS-UnblockNeteaseMusic

原项目地址: https://github.com/nondanee/UnblockNeteaseMusic 
Goproxy控制面板：https://github.com/snail007/proxy_admin_free/blob/master/README_ZH.md

详细参考链接

https://www.moerats.com/archives/938/

https://www.lajiblog.com/index.php/archives/4/

https://github.com/nondanee/UnblockNeteaseMusic/issues/65


为了简洁以下以 CentOS 7为参照系统

如需详细配置，查阅以上链接

# 准备

1.打开vps所用端口，自行谷歌

如果不理解，请搜索 如何打开xx云所有端口
    
2.准备域名并把它解析到自己的VPS上

3.申请ssl证书

腾讯云免费证书：https://console.cloud.tencent.com/ssl


## 1.安装 nginx 
```
yum install nginx

mkdir /etc/nginx/cert

```
申请的证书复制到cert文件里，可以用Xftp上传

配置nginx
```
vim /etc/nginx/nginx.conf
```
将server 配置改成如下：
```
    server {
        listen       443;
        #listen       [::]:80 default_server;
        server_name  您的域名;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        ssl on;
        ssl_certificate cert/xxx.crt;# 改为自己申请得到的 crt 文件的名称
        ssl_certificate_key cert/xxx.key;# 改为自己申请得到的 key 文件的名称
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        location / {
                proxy_pass http://localhost:6666;#转发到unblock代理
        }
    }

```
切换英文输入法 按` Esc `然后输入`:wq!` 回车保存

`6666`自行修改，需要和下面docker端口一致

启动 `nginx` 并设置为开机启动
```
sudo systemctl start nginx.service

sudo systemctl enable nginx.service

```
在浏览器输入`https://你的域名`能打开nginx 页面为成功，因为设置了转发可能会显示 `error`

## 2.安装docker
```
#CentOS系统
curl -sSL https://get.docker.com/ | sh
systemctl start docker
systemctl enable docker
```

## 3. 运行UnblockNeteaseMusic
```
docker run --restart=always --name unmusic -d -p 6666:8080 nondanee/unblockneteasemusic -e https://你的域名
```
`6666 `自行修改,需要和nginx 端口保持一致


## 运行完后可通过HTTP PROXY进行解锁

http-proxy: `你的域名:6666`

可直接使用在Window 客户端(建议http测试成功再进行ss转发)

如果不需要ss转发，只是用http 代理，无需继续下面的教程

#### 以下为使用shadowsocks 转发

#### 需要完成以上http的搭建，不可跳过直接进行ss转发

## 4. 使用Goproxy 控制面板实现转发
下载并安装
```
curl -L https://raw.githubusercontent.com/snail007/proxy_admin_free/master/install_auto.sh | bash
```
 访问页面进行代理转发设置
 >打开浏览器访问：http://IP:32080 , 首次默认账号是root，密码是123，登录后记得第一时间修改。
### 完成后 http-proxy & shadowsocks 皆可以使用

以 Quantumult X 配置为例

```
server_local
shadowsocks=你的域名:8888,method=chach20-ietf,password=password,fast-open=false,udp-relay=false,tag=Unblock

policy
static=NeteaseMusic,Unblock,direct

Rule
host-suffix, music.163.com, NeteaseMusic
host-suffix, music.126.net, NeteaseMusic
user-agent, NeteaseMusic*, NeteaseMusic
user-agent, NeteaseMusic**, NeteaseMusic
user-agent, 网易云音乐*, NeteaseMusic
user-agent, 网易云音乐**, NeteaseMusic
user-agent, %E7%BD%91%E6%98%93%E4%BA%91%E9%9F%B3%E4%B9%90*, NeteaseMusic
user-agent, %E7%BD%91%E6%98%93%E4%BA%91%E9%9F%B3%E4%B9%90**, NeteaseMusic
ip-cidr, 59.111.181.0/24, NeteaseMusic
ip-cidr, 115.236.118.0/24, NeteaseMusic
ip-cidr, 223.252.199.0/24, NeteaseMusic
host-keyword, netease, NeteaseMusic
```
