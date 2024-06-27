# Linux安装配置Clash代理

### 概括描述

Linux环境安装配置Clash工具，以实现代理上网效果。包含下载、安装、配置、运行、测试以及开机自启动、定期自动更新订阅功能的操作文档，希望对你有所帮助。

---

### 一、下载[clash for linux](https://drive.rocklinuxmirror.eu.org/index.php/s/5MndEnkeqzpiiS2/download?path=%2F&files=clash-linux-amd64-v1.18.0.gz)
- 原地址已失效，请使用[镜像地址](https://drive.rocklinuxmirror.eu.org/index.php/s/5MndEnkeqzpiiS2/download?path=%2F&files=clash-linux-amd64-v1.18.0.gz)下载
- ~~[https://github.com/Dreamacro/clash/releases](https://github.com/Dreamacro/clash/releases)~~
- 下载时需注意服务器位数是64还是32位，本文档中使用的64位
![image](https://user-images.githubusercontent.com/43178911/116800809-ee540700-ab36-11eb-940f-25eed6be8a6c.png)

### 二、配置并运行clash

- 命令执行过程（仅供参考,请按本地设置稍作调整）

```bash
[root@localhost ~]# su
[root@localhost ~]# mkdir /opt/clash
[root@localhost ~]# gunzip clash-linux-amd64-v1.18.0.gz
[root@localhost ~]# mv Desktop/clash-linux-amd64-v1.18.0 /opt/clash/clash
[root@localhost ~]# cd /opt/clash/
[root@localhost clash]# wget -O config.yaml [订阅链接]
--2021-05-01 22:39:37--  [订阅链接]
Resolving www.sub-speeder.com (www.sub-speeder.com)... 104.21.18.176, 172.67.182.209, 2606:4700:3035::ac43:b6d1, ...
Connecting to www.sub-speeder.com (www.sub-speeder.com)|104.21.18.176|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 44830 (44K) [application/octet-stream]
Saving to: ‘config.yaml’

100%[======================================>] 44,830       287KB/s   in 0.2s   

2021-05-01 22:39:38 (287 KB/s) - ‘config.yaml’ saved [44830/44830]

[root@localhost clash]# wget -O Country.mmdb https://cdn.jsdelivr.net/gh/Hackl0us/GeoIP2-CN@release/Country.mmdb
--2021-05-01 22:39:55--  https://www.sub-speeder.com/client-download/Country.mmdb
Resolving www.sub-speeder.com (www.sub-speeder.com)... 172.67.182.209, 104.21.18.176, 2606:4700:3035::ac43:b6d1, ...
Connecting to www.sub-speeder.com (www.sub-speeder.com)|172.67.182.209|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3878104 (3.7M) [application/octet-stream]
Saving to: ‘Country.mmdb’

100%[======================================>] 3,878,104   2.86MB/s   in 1.3s   

2021-05-01 22:39:58 (2.86 MB/s) - ‘Country.mmdb’ saved [3878104/3878104]
[root@localhost clash]# chmod +x clash 
[root@localhost clash]# ll
total 19832
-r-x--x--x. 1 root root 9326592 May  1 23:24 clash
-rw-r--r--. 1 root root   44830 May  1 22:39 config.yaml
-rw-r--r--. 1 root root 3878104 Oct  9  2020 Country.mmdb
[root@localhost clash]# ./clash -d .
INFO[0000] Start initial compatible provider Proxy      
INFO[0000] Start initial compatible provider Domestic   
INFO[0000] Start initial compatible provider AsianTV    
INFO[0000] Start initial compatible provider Others     
INFO[0000] Start initial compatible provider GlobalTV
```

### 三、启用系统代理

- 命令行形式开启

```bash
[root@localhost ~]# gsettings set org.gnome.system.proxy mode 'manual'
[root@localhost ~]# gsettings set org.gnome.system.proxy.http port 7890
[root@localhost ~]# gsettings set org.gnome.system.proxy.http host '127.0.0.1'
[root@localhost ~]# gsettings set org.gnome.system.proxy.socks port 7891
[root@localhost ~]# gsettings set org.gnome.system.proxy.socks host '127.0.0.1'
[root@localhost ~]# gsettings set org.gnome.system.proxy ignore-hosts "['localhost', '127.0.0.0/8', '::1']"
```

- 图形化界面形式开启（装有GUI使用）

打开系统设置，点击网络代理右边的 ⚙ 按钮，选择手动，填写 HTTP 和 HTTPS 代理为 127.0.0.1:7890，填写 Socks 主机为 127.0.0.1:7891，即可启用系统代理，如下图所示。

![image](https://user-images.githubusercontent.com/43178911/116800863-599dd900-ab37-11eb-8c86-ceed8bee487b.png)

### 四、测试验证

- 通过curl命令访问www.google.com，响应正常。
![image](https://user-images.githubusercontent.com/43178911/116800843-2fe4b200-ab37-11eb-96fd-5a6ad01ac29b.png)

### 五、配置开机自启动

### **1.创建service文件**

`touch /etc/systemd/system/clash.service`

### **2.编辑service文件**

打开service文件

`vi /etc/systemd/system/clash.service`

填入以下内容**(注意修改clash文件夹路径)**

```bash
[Unit]
Description=clash daemon

[Service]
Type=simple
User=root
ExecStart=/opt/clash/clash -d /opt/clash/clash/
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

保存并退出

### 3**.启动Clash**

`systemctl start clash.service`

### 4**.设置Clash开机自启动**

`systemctl enable clash.service`

### **以下为Clash相关的管理命令**

## 启动Clash ##
`systemctl start clash.service`

## 重启Clash ##
`systemctl restart clash.service`

## 查看Clash运行状态 ##
`systemctl status clash.service`

### 六、配置定时更新订阅

- Clash For Linux 到目前为止没有自动订阅方式，我们做一个计划任务实现更新`config.yaml`
- 用Cron执行计划任务

```bash
[root@localhost ~]# crontab -e
```

- 填入以下内容

```bash
29 6    * * *   root    pgrep clash | xargs kill -s 9 
30 6    * * *   root    mv /opt/clash/config.yaml /opt/clash/configbackup.yaml 
31 6    * * *   root    wget -P /opt/clash/ -O config.yaml [你的订阅链接]
32 6    * * *   root    nohup /opt/clash/clash -d /opt/clash/
```

- 按Esc和:wq保存退出
- 重启crontab，使配置生效

```bash
[root@localhost ~]# systemctl restart crond.service
```

### 七、参考链接

- [https://askubuntu.com/questions/903514/command-to-set-socks-proxy](https://askubuntu.com/questions/903514/command-to-set-socks-proxy)
- [https://github.com/yuanlam/Clash-Linux#enjoy](https://github.com/yuanlam/Clash-Linux#enjoy)
- [https://instruction.lnds.top/software/linux/clash](https://instruction.lnds.top/software/linux/clash)
- [https://www.speederss.best/user/tutorial?os=linux&client=clash##](https://www.speederss.best/user/tutorial?os=linux&client=clash##)

### 八、订阅链接提供商推荐(SPEEDER)

[Speeder](https://www.speeder.one/auth/register?code=ghostxu)

推荐理由

- 稳定：长期有效提供，线路资源丰富，不易掉线
- 低延迟：我自己用的节点里最低可达49ms
- 多设备：最多可支持10台设备同时在线使用
- 适用性强：支持Windows，Linux，Mac，IOS，Android
- 使用便捷：使用教程详细，并且提供一键导入配置功能，操作便捷

### 九、打赏 - JUST FOR FUN
一杯咖啡钱, 打赏金额随意，感谢大家~ :)
|   微信   |   支付宝    |
|------------|-----------|
|<img src="https://github.com/ghostxu97/clash-for-linux/assets/43178911/4914c6f8-ff73-495e-99fc-f4766ccf8959?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Jib3lmZWl5dQ==,size_16,color_FFFFFF,t_70" width="200"/>| <img src="https://github.com/ghostxu97/clash-for-linux/assets/43178911/201ae7e3-6319-420a-88f3-d599dd3fa6f7?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Jib3lmZWl5dQ==,size_16,color_FFFFFF,t_70" width="200"/>  |

