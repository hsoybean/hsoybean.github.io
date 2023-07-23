## 前言


## 使用小钢炮内置Docker安装Openwrt

使用ssh工具或者window powershell，ssh连接小钢炮系统
```shell
ssh root@192.168.3.100
```
输入密码登入

### 1. 设置网卡混在模式
```shell
vim /etc/rc.local
```
在文件后面添加
```shell
ip link set eth0 promisc on
```
重启小钢炮系统

### 2. Docker添加macvlan网络
```shell
docker network create -d macvlan --subnet=192.168.3.0/24 --gateway=192.168.3.1 -o parent=eth0 macnet
```
subnet的192.168.3.0是网段
gateway的192.168.3.1是网关


### 3. docker安装openwrt
```shell
docker run -d --name="OpenWRT" --restart unless-stopped --network macnet --privileged unifreq/openwrt-aarch64 /sbin/init
```
docker镜像使用的是unifreq/openwrt-aarch64
使用以上命令会先拉取unifreq openwrt镜像，然后再创建和运行容器

### 4. openwrt配置

#### 4.1 设置静态ip
```shell
docker exec -it OpenWRT /bin/bash
vi /etc/config/network
```
在文件中找到ip一行，把默认的192.168.1.1改成192.168.3.88（这个ip根据自家网络自行调整），修改完之后保存并退出后,使用以下命令重启openwrt网络服务，使其生效。
```shell
/etc/init.d/network restart
```

电脑浏览器地址输入上面修改的静态IP地址：192.168.3.88，进入到openwrt登录界面。说明已经设置成功。

#### 4.2 openwrt旁路由配置
- 网络-接口设置
- 添加防火墙自定义规则: 
```shell
iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
```

#### 4.3 测试旁路由网络
- 电脑手动设置ip，网关和DNS服务器修改成N1 openwrt旁路由的ip地址，例如我的就是上面修改成的192.168.3.88
- 手机网络手动设置ip，同样网关和DNS服务器修改成N1 openwrt旁路由的IP地址

对应验证：
- 浏览器打开百度测试是否正常访问，如果正常访问，说明到这里openwrt旁路由就说明配置完成了
- 手机浏览器打开百度测试

### 5. openwrt passwall2配置使用
- 添加订阅地址后，手动订阅更新订阅列表
- 根据需要打开socks代理
- 打开passwall2总开关
Bug：小钢炮中docker安装openwrt，使用passwall2并不能科学上网，需要打开其他设置中的TCP改为所有

验证外网是否可用
- 电脑浏览器打开谷歌
- 手机浏览器打开谷歌




