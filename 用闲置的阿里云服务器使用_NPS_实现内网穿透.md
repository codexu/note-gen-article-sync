# 用闲置的阿里云服务器使用 NPS 实现内网穿透

最近有个项目需要给外地的同事预览一下，但是公司没有可以公网访问的测试服务器，所以想到用内网穿透的方式让外地同事可以访问到我的本机。刚好我有一台阿里云的服务器，双十一打折买了3年，1000左右，2核8G，买完就一直闲置，这次刚好可以用上。

## 服务器

首先介绍一下我的服务器：

CPU&内存：2核(vCPU) 8 GiB
操作系统：Alibaba Cloud Linux 3.2104 LTS 64位

### 使用 docker 安装 NPS

下载yum源采用阿里云的镜像源

```sh
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

查看仓库中的所有版本，默认安装最新版本

```sh
yum list docker-ce --showduplicates | sort -r
```

安装docker-ce

```sh
yum install docker-ce -y
```

配置docker镜像源

```sh
vim /etc/docker/daemon.json
```

启动docker服务

```sh
systemctl start docker
```

拉取 NPS 镜像

```sh
docker pull ffdfgdfg/nps
```

启动 NPS

```sh
docker run -d --name=nps --restart=always --net=host -v /opt/nps/conf:/conf ffdfgdfg/nps
```

### 配置安全组

默认的服务器不会开启这几个端口，所以你需要手动去添加：

*   8080: NPS web 管理端口。
*   8024: 服务端客户端通信端口。
*   5173: 这个是我本地服务的端口，所以服务器也用了同样的，这个自定义即可。

> 如果端口和你现任的端口有冲突，可以查看[配置文档](https://ehang-io.github.io/nps/#/server_config)去修改。

![km2h3y.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/96176ced5d6e43c0a516fdc4b8689ce1~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY29kZXh1:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjU1OTMxODc5ODY0MDgwNyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1734679700&x-orig-sign=%2Fcv2BulgJBPt6HUBPuy8zqyIX6M%3D)

### Web 管理

NPS 提供了 web 界面，方便配置，做好上面的步骤后，可通过，公网ip:web界面端口（默认8080），用户名 admin，密码 123 登录访问。

![ioeeil.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/e5f1d77261e74cf3bb4ccc08007ce3eb~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY29kZXh1:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjU1OTMxODc5ODY0MDgwNyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1734679700&x-orig-sign=AvnIGr%2FdeB7nwjIPYxFth7I03FQ%3D)

首先在菜单栏中进入客户端，点击新增

![5zbara.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/b30b1630a99748f1aff54a29a6dbc743~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY29kZXh1:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjU1OTMxODc5ODY0MDgwNyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1734679700&x-orig-sign=KLdSUxshsXAQUvwl0wYiaCcxlIk%3D)

*   备注：随便填
*   Basic 认证用户名：不用管
*   Basic 认证密码：不用管
*   唯一验证密钥：不用管
*   压缩和加密：是

创建后，可以看到新增的客户端，链接状态是离线，没有问题。点击左侧的加号，可以看到客户端命令，这个很重要，在客户端需要执行，用来与服务器链接。

![k3jjil.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/ab3bcced1280427191778cb41421a065~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY29kZXh1:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjU1OTMxODc5ODY0MDgwNyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1734679700&x-orig-sign=Dz%2FdYR4FB598E7%2BhEGfW8IlmR4s%3D)

还有就是看一下客户端 ID，上图中的第一列。

随后菜单选择 TCP 隧道，点击新增。

![a2v8e7.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/7306afeb781645cea88a42bf0b684d8e~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY29kZXh1:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjU1OTMxODc5ODY0MDgwNyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1734679700&x-orig-sign=NWeoSQDeDpbBC9ZvNcKY84T3LzY%3D)

*   模式：TCP 隧道
*   客户端ID：填客户端页面中你创建的那个客户端 ID
*   服务端端口：这里我选择了和我本机项目一样的端口，5173，主要是供外网访问时的端口，你可以填任何。
*   目标 (IP:端口)：这里指的是你的本机，IP 就是本机 127.0.0.1 即可，端口是你的项目端口，我这里是 5173。

![1bg9qv.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/a47684709d864ac39f7b1c4dcedf2534~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY29kZXh1:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjU1OTMxODc5ODY0MDgwNyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1734679700&x-orig-sign=GLxtufRajX38x%2BHXorewBHPQ6VQ%3D)

> 状态是离线是正常的，因为我们还没有在客户端进行配置。

## 本机

我本机是 mac，访问 GitHub 去下载对应的客户端，[https://github.com/ehang-io/nps/releases](https://github.com/ehang-io/nps/releases/tag/v0.26.10)。

![a1dob7.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/dab4cdef14584239a3e7cb69b2929ac4~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY29kZXh1:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjU1OTMxODc5ODY0MDgwNyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1734679700&x-orig-sign=mZVK52fbqWC6CydlgQIl68aZtk0%3D)

这里记得选 client 后缀的文件。

我在 `~/` 路径下创建了 npc 文件夹，并解压到这里。

进入 `~/npc` 运行：

```sh
./npc -server=*.*.*.*:8024 -vkey=av3*****yiepb1 -type=tcp
```

这段代码就是上文提到的创建的客户端后展示的那段代码。

如果你看到 Successful connection with server 证明链接成功了。

这时看到 web 界面中，状态也变成了在线。

![oh4uul.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/32fb574bf349465c9201c91b97d20f68~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY29kZXh1:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjU1OTMxODc5ODY0MDgwNyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1734679700&x-orig-sign=kjoJ1RB8YT5wcZ1UgrT6UIl5ico%3D)

之后通过公网 IP+端口 访问一下，发现项目已经可以在公网正常访问了。

## 参考

[NPS 中文文档](https://ehang-io.github.io/nps/)
