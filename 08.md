#### 建议阅读语雀版本
https://www.yuque.com/docs/share/6b1a7c22-b81d-4344-b3ec-b74481bd1cb8?#

> 💡 Tip
> - 记得备份
> - 请仔细阅读所有文字，请记住，他们都不是多余的
> - 本次受害者为Redmi AC2100

### 题外话
#### uboot 的选择
就目前来说，路由器 uboot 主要有两种，breed 和 pb-boot，而就我个人而言，建议先刷 breed，教程多而且好找对应版本，在使用 breed 无法刷入某固件时（比如 Redmi AC1200 的东芝闪存坏块），再选择 pb-boot

---

### 准备
> 💡 Tip
> - Windows Defender 大概率会删除`nc.exe`，建议打开 Windows Defender 窗口以便及时阻止

---

### 降级
最新版的官方固件已经修复了这个 BUG，所以我们需要先行刷入旧固件
```tips
进入路由器后台 -> 常用功能 -> 系统状态 -> 升级检测 -> 系统版本 -> 手动升级
```

---

### 刷 breed
#### telnet

1. 开启 Telnet Client 功能
```tips
控制面板 -> 程序 -> 启用或关闭 Windows 功能 -> 勾选Telnet 客户端
```

2. 关闭防火墙
```tips
控制面板 -> 系统与安全 -> Windows Defender 防火墙 -> 启用或关闭 Windows Defender 防火墙 -> 专用和公用均勾选关闭
```

3. 先用一根网线连接 wan 口和 wan 口隔壁的 lan 口（不能是其他的），再用另一根网线连接路由器和电脑
3. 禁用其他网卡
```tips
控制面板 -> 网络和 Internet -> 网络和共享中心 -> 更改适配器设置 -> 除以太网（连接路由器）的网卡外其他全部禁用
```

5. 禁用完网卡后，更改以太网网卡的信息
```tips
右键打开属性 -> Internet 协议版本 4 -> 填入下面的信息 =>
    IP 地址：192.168.31.177
    子网掩码：255.255.255.0
    默认网关：192.168.31.1
```
#### 开始刷入 breed

1. 安装`npcap-0.9991.exe`
1. 设置 PPPoE 拨号
```tips
账号：123
密码：123
```

3. 开启 telnet
```tips
打开`一键开启telnet.bat` -> 检查弹出的窗口是否有数据包显示，有则回到第一个窗口输入`y`并回车 -> 将`开启telnet命令.txt`中的命令复制粘贴进跳出的`Task_反弹shell`窗口
```

4. 正式刷入 breed

打开`cmd`，输入以下内容
> 💡 Warning
> - telnet 开启成功后请将连接 wan 口和 lan 口的网线拔掉，否则刷入 breed 后易造成网络风暴（摘自恩山论坛）
> - 连接电脑的不用拔

```bash
telnet 192.168.31.1

wget http://192.168.31.177:8081/breed-mt7621-xiaomi-r3g.bin&&nvram set uart_en=1&&nvram set bootdelay=5&&nvram set flag_try_sys1_failed=1&&nvram commit

mtd -r write breed-mt7621-xiaomi-r3g.bin Bootloader
```

1. 还原网卡设置，将以太网改回自动获取 IP 地址
2. 建议等待五分钟后再进行后面的操作

---

### 刷入固件

1. 拔掉路由器电源，按住 reset 键再插电，等待蓝灯闪烁后松开
1. 浏览器输入 192.168.1.1 进入 Breed Web 控制台
1. 正式刷入固件
```tips
固件更新 -> 勾选“固件” -> 选择文件 -> 更新
```

4. 等待进度条跑完

---

### 进入 OpenWrt

1. 浏览器输入 192.168.1.1 进入 OpenWrt 管理界面
```tips
账号：root
密码：password
```

---

### 网络配置

1. 先插好网线~~（怎么会有人不插网线搞了半天）~~
1. 拨号
```tips
网络 -> 接口 -> WAN -> 修改 ->
    基本配置 ->
        协议 -> PPPoE
        用户名 -> 学号
        密码 -> 校园网密码
    保存&应用
网络 -> 无线 ->
    基本设置 ->
        SSID -> WiFi名称
        加密 -> 可看情况选择，否则无脑 WPA2PSK
        密码 -> WiFi密码
    保存&应用
```

3. 解释一下这个无线概况
- 首先，似乎上面是 2.4G 频段的 WiFi，下面是 5G 频段
- 其次，那两个接口叫 apclii0 的我不知道是什么东西，无法禁用
- 最后，高级设置里面有一些其他的设置，不懂勿动

![RM1200 OpenWrt无线界面.png](https://cdn.nlark.com/yuque/0/2022/png/29478561/1663253334741-8ffe517a-cd62-408e-b1ac-12f7e4940479.png#clientId=u14346e32-77e8-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=ub090b066&margin=%5Bobject%20Object%5D&name=RM1200%20OpenWrt%E6%97%A0%E7%BA%BF%E7%95%8C%E9%9D%A2.png&originHeight=902&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=117835&status=done&style=none&taskId=ud98bf103-9472-4e18-9973-a0f20987e2d&title=)

---

### 安装 Dr.com

1. 将插件上传到路由器
```tips
系统 -> 文件传输 -> 选择文件 -> 上传
```

2. 安装
> 💡 Tip
> - 接下来的内容可能需要一点点的 Linux 知识

```tips
打开 PuTTY ->
Host Name -> 192.168.1.1
Port -> 22
Open
```
```
login as: root  // 路由器后台管理账户，默认 root
root@192.168.1.1's password: ...  // 路由器后台管理密码，默认 password
```
当你看到 OpenWrt 的 logo 以及 Linux 系统标志性的 root@OpenWrt`~# 时，就证明你成功进入了路由器系统
接下来是正式安装插件，键入以下命令进行安装
```bash
cd /tmp/upload
opkg install gdut-drcom_6.0-4_mipsel_24kc.ipk
```
等待安装完毕，安装完毕后记得刷新一下浏览器后台界面

1. 配置 Dr.com
```tips
服务 -> Dr.com ->
    勾选“Dr.com认证”和“认证拨号”
    接口名称 -> 选择与 网络 -> 接口 中 WAN6 下方相同的选项
    校园网账号 -> 学号
    密码 -> 校园网密码
    MAC拨号 -> 填写 网络 -> 接口 中 WAN 的 MAC 地址
保存&应用
```

4. 重启并等待一段时间，若 网络 -> 接口 中 WAN 的接收和发送有数据便说明成功连接，当然可以直接上网也可说明
5. 附无法上网原因（摘抄自[1]()）
```warning
1. WAN 中，学号密码输入错误（可能性30%）；
2. Dr.com 插件中，学号密码输入错误（可能性30%）；
3. 路由器的 WAN 口没有与校园网端口连接，即没接网线（可能性20%）；
4. 网线断了，或者路由器坏了（可能性15%）；
5. 压根没开通校园网（可能性4.9%）；
6. 端口被学校网络中心拉黑了（极少出现，可能性0.1%）。
```

---

### 配置防检测
#### 原理及解决方法：（摘抄自[1]()）
```tips
User-Agent字段中包含了操作系统版本信息，而HTTP协议没有对这些信息加密，因此可以直接从这里判断数据包是发自于PC设备还是移动设备。根据这个原理，可以猜测到，可能是由于移动设备发送的数据包被检测到，才导致了被强制断网的情况发生。
所以，解决的办法就是统一所有设备的User-Agent，模拟成只有一台设备在上网的情况
这里推荐使用xmurp-ua插件，作者已经开源，项目地址：https://github.com/CHN-beta/xmurp-ua
```
#### 安装 xmurp-ua

1. 同上，上传插件到路由器
1. 同上，使用 PuTTY登陆路由器系统，并输入以下命令进行安装
```bash
cd /tmp/upload
opkg install mt7621_ua_5.4.69-34_mipsel_24kc.ipk
```

3. 安装压缩内存插件~~（虽然我不知道有什么用）~~，确保已连接网络
```bash
opkg update
opkg install zram-swap
```

4. 重启路由器
```bash
reboot
```
#### 验证防检测效果
使用电脑浏览器打开 http://www.user-agent.cn/，查看`你的ua`是否为`XMURP/1.0`

---

### 其他：
#### 问题整理：
1. https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt#%E7%96%91%E6%83%91%E8%A7%A3%E7%AD%94

#### 参考文献：
[1]：https://post.smzdm.com/p/aoo85457/  
[2]：https://www.right.com.cn/forum/thread-4023907-1-1.html  
[3]：https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt  
