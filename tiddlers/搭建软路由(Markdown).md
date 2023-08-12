# 软路由

---

## 我现在的网络拓扑

![](https://s1.ax1x.com/2023/08/08/pPZZSUI.png)

---

## r2s(大部分的功能全都部署在这里)

### openwrt系统刷机

准备一张tf卡和[balenaEtcher](https://www.balena.io/)(刷机工具)

然后准备openwrt系统,我直接从直接从[OpenWrt固件下载与在线定制编译 (supes.top)](https://supes.top/?version=22.03&target=rockchip%2Farmv8&id=friendlyarm_nanopi-r2s)构建了自己的openwrt

选择如下
![](https://s1.ax1x.com/2023/04/29/p91rpND.jpg)

互换默认网口的原因是默认的Wan口是USB3.0改装来的,网速不能千兆,不能让瓶颈出现在进入的位置

### KMS服务器

这东西开箱即用,什么都不用设置

### SmartDNS

参考[如何开启SmartDNS及相关配置 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv12437146/)

#### 关闭魔法

PassWall,Bypass,SSR-Plus,OpenClash,PassWall2,VSSR (Hello World)之类的==全部关闭==

#### 调整Lan口

网络-接口界面设置Lan口,使用自定义的DNS服务器,地址为访问 openwrt 的ip(后台地址)

![](https://s1.ax1x.com/2023/04/29/p916u4O.png)
![](https://s1.ax1x.com/2023/04/29/p916Q8e.png)

#### 关闭MWAN3分流服务

我构建自己的 openwrt ,安装了MWAN3,但是我没找到他在那里,只找到一个MWAN,所以先跳过这一部分
![](https://s1.ax1x.com/2023/04/29/p91cERg.png)

下面是教程中这一部分的操作
> 检查服务内的【MWAN3分流服务】是否为关闭，该固件下默认为关闭；如果没有此插件即可无视得了
> ![](https://i0.hdslb.com/bfs/article/ba95fb47ee1d6f73f92e8817db4392effb8153e5.png@942w_509h_progressive.webp)

#### 设置Turbo ACC Center

![](https://s1.ax1x.com/2023/04/29/p91c6QH.png)


#### 配置SmartDNS

> 以下内容分为两部分设置，分别是【国内组】&【国外组】；
> 第一组设置，建议名称为：【China】，默认端口6053，重定向选择【替换dnsmap上游服务器】，开启【双栈IP优选】、开启【TCP服务器】、开启【域名预加载】、开启【过期缓存服务】、缓存大小可设置【2048】、域名TTL为空、最小值默认300、最大值为3600或为空，然后保存；
> ![](https://i0.hdslb.com/bfs/article/faab7ebdf7286701e72cb916d9de773c93baf28c.png@942w_935h_progressive.webp)

> 第二组设置，建议名称【Overseas】默认端口5335，勾选-TCP服务器、跳过测速、跳过双栈优选、跳过cache；
> ![](https://i0.hdslb.com/bfs/article/8d7056fffa43034c5eb7ffb3fa9bf2d4d57d08f2.png@942w_722h_progressive.webp)

> 添加上游服务器如下，下面两个表都填写保存后，挨个点击每条后面的【编辑】修改组名，国内对应【China】国外对应【Overseas】，之后保存并应用；

我设置国内对应`smartdns-China`,国外(第二DNS服务器)对应`smartdns-Overseas`

![](https://s1.ax1x.com/2023/04/29/p91hrfP.png)


#### 修改DHCP/DNS

![](https://s1.ax1x.com/2023/04/29/p91cLmn.png)

(如果只是配置SmartDNS,就配置这个)

### Adguard Home

参考[如何正确使用smartdns搭配adguardhome， 优选dns并去除广告(smzdm.com)](https://post.smzdm.com/p/ag82pod6/)

#### 设置1745重定向

![](https://s1.ax1x.com/2023/08/09/pPZd7gU.png)

#### 设置Adguard Home的上游DNS

打开3000端口的Adguard Home网页管理界面,进入DNS设置
![](https://s1.ax1x.com/2023/08/09/pPZwlrQ.png)

设置上游DNS为SmartDNS
![](https://s1.ax1x.com/2023/08/09/pPZwwMF.png)

#### 设置规则

![](https://s1.ax1x.com/2023/08/09/pPZ0GOe.png)

添加黑名单,我是用了这份[jhsvip/ADRuls: AdGuard Home 规则，整合了80多万条规则(github.com)](https://github.com/jhsvip/ADRuls)

### ~~ShadowSocksR Plus+~~

**我不是很推荐这个软件**,因为他的黑白名单功能在我这里一直没有用;而且在使用一些节点的时候,一段时间后CPU占用会飙升到100%导致整个软路由瘫痪,只能强制断电然后接电重启

不过如果如果只是为了追求设置简单,并想要网速尽可能快,可以使用这个

#### 设置运行模式与DNS

设置运行模式为"绕过中国大陆ip",如果前面像我一样配置了SmartDNS,可以同时配置DNS
![](https://s1.ax1x.com/2023/08/09/pPZ0Sds.png)

#### 配置节点

![](https://s1.ax1x.com/2023/08/09/pPZ0FzT.png)



### OpenClash

#### 一些知识-运行模式

- Redir-Host
    - 兼容模式
    
      - > 信息来源:[常规设置 · vernesong/OpenClash Wiki (github.com)](https://github.com/vernesong/OpenClash/wiki/常规设置)
        >
        > 客户端进行通讯时DNS由Clash先进行并发查询，等待返回结果后再尝试进行规则判定和连接。
        >
        > 当判定需要代理时，使用fallback组DNS的查询结果进行通讯
        >
        > 实际效果：客户端响应速度一般，可能出现网页加载时间过长的情况。
        
      - > 信息来源:ChatGPT
        >
        > 在此模式下，OpenClash会通过修改系统的DNS配置，将所有的DNS请求重定向到本地的Clash代理客户端。这样，所有的网络流量都会通过Clash进行代理。
    - TUN模式
    
      - > 信息来源:[常规设置 · vernesong/OpenClash Wiki (github.com)](https://github.com/vernesong/OpenClash/wiki/常规设置)
        >
        > 此模式与Redir-Host（兼容）模式类似，不同在于能够代理所有UDP链接，提升nat等级，改善游戏联机体验。
      
      - > 信息来源:ChatGPT
        >
        > 在这种模式下，OpenClash会创建一个虚拟网络接口（TUN接口），并将所有的网络流量通过该接口发送到Clash代理客户端。这种模式适用于需要对所有网络流量进行代理的情况，包括UDP和TCP流量。
    - TUN-混合模式(UDP-TUN,TCP-转发)
    
      - > 信息来源:ChatGPT
        >
        > 这种模式结合了TUN模式和转发模式。UDP流量将通过TUN接口发送到Clash代理客户端，而TCP流量将通过转发方式发送到Clash代理客户端。这种模式适用于需要同时代理UDP和TCP流量的场景。
- Fake-IP
    - 增强模式
    
      - > 信息来源:[常规设置 · vernesong/OpenClash Wiki (github.com)](https://github.com/vernesong/OpenClash/wiki/常规设置)
        >
        > 客户端进行通讯时会先进行DNS查询目标IP地址，拿到查询结果后再尝试进行连接。
        >
        > Fake-IP 模式在客户端发起DNS请求时会立即返回一个保留地址（198.18.0.1/16），同时向上游DNS服务器查询结果，如果判定返回结果为污染或者命中代理规则，则直接发送域名至代理服务器进行远端解析。
        >
        > 此时客户端立即向Fake-IP发起的请求会被快速响应，节约了一次本地向DNS服务器查询的时间。
        >
        > 实际效果：客户端响应速度加快，浏览体验更加顺畅，减轻网页加载时间过长的情况。
        
      - > 信息来源:ChatGPT
        >
        > 在此模式下，OpenClash会使用IP包的伪装技术，将所有的网络流量发送到Clash代理客户端。Clash会修改源IP和目标IP，以实现代理功能。这种模式适用于需要伪装IP地址的场景，以绕过一些网络限制。
    - TUN模式
    
      - > 信息来源:[常规设置 · vernesong/OpenClash Wiki (github.com)](https://github.com/vernesong/OpenClash/wiki/常规设置)
        >
        > 此模式与Fake-IP（增强）模式类似，不同在于能够代理使用域名的UDP链接。
      
      - > 信息来源:ChatGPT
        >
        > 与Redir-Host模式中的TUN模式类似，OpenClash会创建一个虚拟网络接口（TUN接口），并将所有的网络流量通过该接口发送到Clash代理客户端。这种模式适用于需要对所有网络流量进行代理的情况，包括UDP和TCP流量。
    - TUN-混合模式(UDP-TUN,TCP-转发)
    
      - > 信息来源:ChatGPT
        >
        > 这种模式结合了TUN模式和转发模式。UDP流量将通过TUN接口发送到Clash代理客户端，而TCP流量将通过转发方式发送到Clash代理客户端。这种模式适用于需要同时代理UDP和TCP流量的场景。
    
    总结:
    
    > 信息来源:ChatGPT
    >
    > 总结来说，Redir-Host模式适用于通过修改系统DNS配置或虚拟网络接口来实现代理的场景，而Fake-IP模式适用于使用IP伪装技术实现代理的场景。TUN模式适用于需要对所有网络流量进行代理的情况，而TUN-混合模式适用于需要同时代理UDP和TCP流量的场景。具体选择哪种模式取决于你的需求和网络环境。

#### 选择运行模式

因为考虑到需要配合SmartDNS和Adguard Home,选择Redir-Host(TUN模式)

![](https://s1.ax1x.com/2023/08/10/pPej4r6.png)

#### 选择Meta内核

上图中把`使用Meta内核`勾选上

#### 选择代理模式

上图中的`代理模式`选择`Rule【策略代理】`

#### DNS设置

![](https://s1.ax1x.com/2023/08/10/pPev9Ig.png)

#### ipv6

我这里因为机场不支持ipv6,所以设置允许解析ipv6,但是不代理ipv6

![](https://s1.ax1x.com/2023/08/10/pPexpOx.png)

#### 与SmartDNS和Adguard Home的配合

如果想配合前面配置的SmartDNS和Adguard Home,按照下面的方式配置

##### Adguard Home

`1745重定向`设置成`无`

![](https://s1.ax1x.com/2023/08/10/pPevhWj.png)

##### OpenClash

勾选`自定义上游DNS服务器`

![](https://s1.ax1x.com/2023/08/10/pPevoyq.png)

设置DNS服务器

同界面往下滑,填写DNS服务器为本机,端口为Adguard Home的1745端口

![](https://s1.ax1x.com/2023/08/10/pPevqTU.png)

![](https://s1.ax1x.com/2023/08/10/pPevXY4.png)

---

配置到这里之后,r2s的dns应该是这个样子的了:
![](https://s1.ax1x.com/2023/08/10/pPmZjQf.png)

#### 下载内核

![](https://s1.ax1x.com/2023/08/10/pPmPODs.png)

滚动到下面,依次点击三个`检查并更新`,下载内核

![](https://s1.ax1x.com/2023/08/10/pPmiC2F.png)

待左侧三个`当前内核版本`均变为绿色后再进行下一步

#### 添加订阅配置

![](https://s1.ax1x.com/2023/08/10/pPmFb0s.png)

### Samba

先对硬盘分区,然后根据固件的支持格式化文件系统(我选择了btrfs),挂载到/mnt下面,然后参考我的进行配置

![](https://s1.ax1x.com/2023/08/10/pPmn6oQ.png)

### Aria2

因为这个固件编译时安装的aria2模块有启动问题(后来再安装的不会有问题),所以需要自己配置一下

把[Aria2 完美配置 | aria2.conf (p3terx.github.io)](https://p3terx.github.io/aria2.conf/)下载下来,放置到你喜欢的位置,我这里选择放置到下载目录下面的子文件夹下面

下载目录我设置为`/mnt/mmcblk0p3/Aria2`,我把配置文件存放在`/mnt/mmcblk0p3/Aria2/config/aria2.conf.main`

设置aria2开机自启动:`aria2c --conf-path=/mnt/mmcblk0p3/Aria2/config/aria2.conf.main &`

![](https://s1.ax1x.com/2023/08/10/pPmuPFH.png)

然后查看aria2的web页面

![](https://s1.ax1x.com/2023/08/10/pPmulfs.png)

![](https://s1.ax1x.com/2023/08/10/pPmuDpR.png)

如果看到红框处的内容,说明aria2启动成功,运行正常

### Alist

如下图开启,然后打开web控制界面

![](https://s1.ax1x.com/2023/08/10/pPm1se0.png)

输入默认的账号密码后,如下进入管理界面

![](https://s1.ax1x.com/2023/08/10/pPm3p0P.png)

然后添加你想加的网盘或者本地驱动器,参考文档:[通用项 | AList文档 (nn.ci)](https://alist.nn.ci/zh/guide/drivers/common.html#排序相关)

![](https://s1.ax1x.com/2023/08/10/pPm3Gc9.png)

### Rclone

因为这个固件编译时安装的Rclone模块,自带的两个管理面板全部连接不上rclone后端,所以需要自己配置一下

rclone的配置文件需要自己生成,所以需要先在本机上面[下载rclone](https://rclone.org/downloads/),然后配置,最后上传到openwrt上面,可以参考这个视频:[openwrt使用rclone挂载阿里云盘到本地_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Ca411d7tU/?vd_source=19b149e88035c5170227f5c152bfd0b1)

完成后,在开机启动脚本里面加上这样一句:`/usr/bin/rclone rcd -vv --rc-web-gui --rc-web-gui-no-open-browser --rc-addr=10.0.0.1:5572 --rc-user=admin --rc-pass=admin --config=/root/.config/rclone/rclone.conf --rc-allow-origin=* --log-file=/var/log/rclone.log &`

![](https://s1.ax1x.com/2023/08/10/pPm8JPS.png)

然后浏览器访问`10.0.0.1:5572`,账号密码使用上面命令中的参数,全为`admin`

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810173409043.png)

然后应该可以看到状态是:已经和rclone后端连接

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810173517697.png)

### qBittorrent

设置启用,写好下载目录后`保存并应用`,打开web界面

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810173805292.png)

输入上面提示的默认用户名和密码后就可以看到界面了

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810173848347.png)

### ipv6

ipv6pd,中继,桥接和nat之间的关系

> 信息来源:[SLAAC 环境下的 IPv6 桥接与中继 - Menci's Blog](https://blog.men.ci/ipv6-slaac-relay-and-bridge/)
>
> * ipv6pd
>
>     * PD（Prefix Delegation）是 DHCPv6 的一项扩展，用于 DHCP 服务器将一整段地址分配给 DHCP 客户端。这种情况一般常见于 ISP 为用户分配 IPv6 地址。在客户端获取地址时，DHCPv6 服务器（作为上级路由器）会添加一条路由，将整个被下发的网段路由到客户端。这样一来，整个地址块（一般为 `/64` 或者 `/60`）均可被客户端网络使用。
>
>         DHCPv6 客户端收到由 PD 下发的前缀后，即可通过 SLAAC 等方式为整个网络内的所有主机配置 IPv6 地址，这个过程不再需要上级路由的参与。这也是一般家用网络中最常见的 IPv6 地址分配方式
>
> * ipv6中继
>
>     * 在这种情况下，除 NAT 之外，使 LAN 中的终端设备接入 IPv6 的主要思路是，假装这些设备被直接接入到 WAN 中，从 WAN 上的上级路由获得 IPv6 地址，并在 LAN 和 WAN 之间对 SLAAC 和 NDP 协议进行代理 —— 双向转发 SLAAC 与 NDP 包，并将源 MAC 地址改为我们的路由器的 MAC 地址。在 WAN 和 LAN 中的设备看来，对方网络中的 IP 地址由我们的路由器所持有，所有流量均由我们的路由器。
>
> * ipv6桥接
>
>     * 我们可以将 IPv4 和 IPv6 视作独立的网络接口。假设我们有 WAN4、WAN6、LAN4、LAN6，将 WAN6 与 LAN6 桥接，保持 WAN4 与 LAN4 上原有的 NAT 配置。这样一来，桥接会使得 WAN 与 LAN 中主机的 IPv6 流量互通，无需关心地址分配与邻居发现上的任何问题。
>
> * ipv6nat
>
>     * 参考ipv4nat

#### ipv6中继

这是我在家庭宽带上面使用的方案,ISP提供的光猫完成了 PPPoE 拨号、DHCPv6 PD 获取前缀、SLAAC 下发地址的整个过程，接入光猫（上级路由）的路由器无法获得前缀

> 路由器本身处于一个没有 DHCPv6 PD 分配的环境中，只能通过 SLAAC 从上级路由（WAN）获得单个 IPv6 地址，应该如何为网络中（LAN）的主机分配 IPv6 地址？

这时候就可以使用ipv6中继

这是我的在家里的网络拓扑

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810191750201.png)

ipv6中继参考:[二级路由(openwrt)开启ipv6中继(ipv4和ipv6共存) – late哥哥笔记 (lategege.com)](https://www.lategege.com/?p=676)

##### 配置wan6

首先确认openwrt的接口,我这里wan口有两个,实际上物理接口只有一个;我这里分出两个逻辑接口是为了可以分开配置ipv4和ipv6:ipv4设置成nat,ipv6设置成中继

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810194118380.png)

配置wan6协议为dhcpv6,这是为了wan口可以获取到上级路由分配的ipv6地址

##### 配置wan

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810200354926.png)

下图中勾选`指定的主接口`,`RA服务`、`dhcpv6服务`、`NDP代理`全部设置成`中继服务`

>  wan口的中继服务配置的意思是让wan口继承上级路由的ipv6地址分配能力,从而进一步往lan口分发这种能力

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810200708387.png)

##### 配置lan

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810201254104.png)

协议配置成静态地址

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810201338914.png)

ipv6设置全部设置成中继模式

> 因为我配置了nat6,不方便修改演示,所以这里偷了别人的一张图,图来源:[二级路由(openwrt)开启ipv6中继(ipv4和ipv6共存) – late哥哥笔记 (lategege.com)](https://www.lategege.com/?p=676)
>
> ![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810201601305.png)

#### ipv6 nat

这是我在学校,宿舍有线网的配置,我只拿到了一个/64的ipv6地址

> 信息来源:[Openwrt ipv6 配置 | 默 (jasper1024.com)](https://jasper1024.com/jasper/codsa242cp3/)
>
> 校园网 ipv6 环境千差万别,每个学校只能对号入座
>
> 总结有 3 种配置: 中继(Relay) 桥接(Brige) 和 NAT6,以下纯个人建议,请结合实际尝试
>
> - 校园网有 PD/SLAAC -> 基本同家宽.难度 easy 很多路由器默认配置就能用
> - 需要认证才能有地址 -> 先找能完成认证的插件吧
> - 插线就有 ipv6 地址
>     - 不能访问公网,还得认证 -> 先找能完成认证的插件吧
>     - 能访问公网:
>         - 只有 /128 的 -> 大概率还有别的限制,NAT6 更直接
>         - 有 /64 -> 优先尝试中继(Relay),但是学校对 mac 地址有限制那就没辙了
>         - 所有情况都能用桥接(Brige)

这是我的在宿舍的网络拓扑:

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810202521984.png)

我们学校的情况:

* 限制了一个账号只能有两个设备,似乎是根据mac来限制的;我和室友一起使用一个账号,他的设备是电脑,我的设备的路由器

* 我把我的一级路由mac地址设置为无感验证设备,就没有认证问题的存在了

* 我的一级路由分配到了/64的ipv6地址,但这是dhcp分配的,所以我没法使用ipv6中继

    > 信息来源:QQ的大佬![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810203018547.png)
    > dhcp是有状态的,不太可能共享给下游

    如何分别出我的ipv6是DHCP分配的还是SLAAC分配的?

    > 信息来源:ChatGPT
    >
    > 在Linux上使用 `ip a` 命令查看网络接口的IPv6地址配置信息时，DHCP分配和SLAAC分配的地址会有不同的标识。下面是可能的输出示例：
    >
    > 1. DHCP分配的IPv6地址：
    >    
    >    如果你的IPv6地址是通过DHCP分配的，你可能会在IPv6地址信息中看到以下类似的标识：
    >
    >    ```
    >    inet6 2001:db8::1234:abcd:efgh/64 scope global dynamic
    >    ```
    >
    >    在这个示例中，关键词 "dynamic" 表示这个IPv6地址是通过动态主机配置协议（DHCPv6）分配的。
    >
    > 2. SLAAC分配的IPv6地址：
    >    
    >    如果你的IPv6地址是通过SLAAC分配的，你可能会在IPv6地址信息中看到以下类似的标识：
    >
    >    ```
    >    inet6 2001:db8::5678:ijkl:mnop/64 scope global dynamic mngtmpaddr
    >    ```
    >
    >    在这个示例中，虽然也有 "dynamic"，但额外的关键词 "mngtmpaddr" 表示这个IPv6地址是通过SLAAC分配的。这个关键词表示 "managed temporary address"，它是一个表明SLAAC分配的标识。
    >
    > 需要注意的是，标识可能会因不同的操作系统版本和配置而有所不同。因此，最好仔细查看输出中的信息，寻找与IPv6地址分配方式相关的关键词。

* ipv6桥接会把下级设备的mac地址暴露给上级设备,这会导致我的一级路由在校园网里面相当于使用了两个设备,这不行

* 综上,我只能选择ipv6nat

参考:

* [校园网环境下Openwrt配置ipv6教程——以nat6为例 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/492774540)(提供了配置步骤)
* [再说 OpenWRT 校园网 IPv6 NAT6-OPENWRT专版-恩山无线论坛 - Powered by Discuz! (right.com.cn)](https://www.right.com.cn/forum/thread-2661027-1-1.html)(提供了nat6脚本与相关的答疑解惑)

##### 设置wan

设置成下面的样子,取消勾选`指定的主接口`,`RA服务`、`dhcpv6服务`、`NDP代理`全部设置成`已禁用`

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810200708387.png)

##### 设置lan

设置成下面这个样子,`RA服务`、`dhcpv6服务`全部设置成`服务器模式`,`NDP代理`设置成`已禁用`

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810212209049.png)

##### 测试一级路由的ipv6工作时候正常

我一级路由接上校园网就被分配到了一个ipv6,/64的地址,我现在要测试他是否正常

ssh连接上一级路由,然后运行下面的命令进行测试

```bash
curl 6.ipw.cn
```

查看这个命令是否返回了本机的ipv6地址

```bash
curl mirrors6.bfsu.edu.cn
```

查看是否获取到了网页内容

都满足的情况下,说明一级路由的ipv6工作正常

##### 设置lan的内网前缀

> 信息来源:[校园网环境下Openwrt配置ipv6教程——以nat6为例 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/492774540)
>
> 在路由器自身能够访问ipv6后,我们要解决因为地址不足造成内网设置无法访问ipv6的问题

设置一个dd开头的前缀

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810210631859.png)

##### ~~安装ipv6nat必要的软件包~~

```bash
opkg install ip6tables
opkg install kmod-ipt-nat6
```

如果你和我输出的结果一样,说明已经固件已经包含了这两个软件,不需要再安装了

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810211016056.png)

##### 设置开机ipv6nat脚本

知乎上的修改版脚本如下

```bash
#!/bin/sh /etc/rc.common
# NAT6 init script for OpenWrt // Depends on package: kmod-ipt-nat6

# edited by Sad Pencil at 2020-02-09
# replace route command with ip command to solve issues on new OpenWRT


# edited by Sad Pencil at 2021-11-29
# update line WAN6_INTERFACE=$(uci get "network.$WAN6_NAME.device" || uci get "network.$WAN6_NAME.ifname")


START=55

# Options
# -------

# Use temporary addresses (IPv6 privacy extensions) for outgoing connections? Yes: 1 / No: 0
PRIVACY=1

# Maximum number of attempts before this script will stop in case no IPv6 route is available
# This limits the execution time of the IPv6 route lookup to (MAX_TRIES+1)*(MAX_TRIES/2) seconds. The default (15) equals 120 seconds.
MAX_TRIES=15

# An initial delay (in seconds) helps to avoid looking for the IPv6 network too early. Ideally, the first probe is successful.
# This would be the case if the time passed between the system log messages "Probing IPv6 route" and "Setting up NAT6" is 1 second.
DELAY=5

# Logical interface name of outbound IPv6 connection
# There should be no need to modify this, unless you changed the default network interface names
# Edit by Vincent: I never changed my default network interface names, but still I have to change the WAN6_NAME to "wan" instead of "wan6"
WAN6_NAME="wan6"

# ---------------------------------------------------
# Options end here - no need to change anything below

boot() {
        [ $DELAY -gt 0 ] && sleep $DELAY
        WAN6_INTERFACE=$(uci get "network.$WAN6_NAME.ifname")
        logger -t NAT6 "Probing IPv6 route"
        PROBE=0
        COUNT=1
        while [ $PROBE -eq 0 ]
        do
                if [ $COUNT -gt $MAX_TRIES ]
                then
                        logger -t NAT6 "Fatal error: No IPv6 route found (reached retry limit)" && exit 1
                fi
                sleep $COUNT
                COUNT=$((COUNT+1))
                PROBE=$(ip -6 route | grep -i '^default.*via' | grep -i -F "dev $WAN6_INTERFACE" | grep -i -o 'via.*' | wc -l)
        done

        logger -t NAT6 "Setting up NAT6"

        if [ -z "$WAN6_INTERFACE" ] || [ ! -e "/sys/class/net/$WAN6_INTERFACE/" ] ; then
                logger -t NAT6 "Fatal error: Lookup of $WAN6_NAME interface failed. Were the default interface names changed?" && exit 1
        fi
        WAN6_GATEWAY=$(ip -6 route | grep -o '2001.*1102' | sed s'/1102/1101/g')
        if [ -z "$WAN6_GATEWAY" ] ; then
                logger -t NAT6 "Fatal error: No IPv6 gateway for $WAN6_INTERFACE found" && exit 1
        fi
        LAN_ULA_PREFIX=$(uci get network.globals.ula_prefix)
        if [ $(echo "$LAN_ULA_PREFIX" | grep -c -E "^([0-9a-fA-F]{4}):([0-9a-fA-F]{0,4}):") -ne 1 ] ; then
                logger -t NAT6 "Fatal error: IPv6 ULA prefix $LAN_ULA_PREFIX seems invalid. Please verify that a prefix is set and valid." && exit 1
        fi

        ip6tables -t nat -I POSTROUTING -s "$LAN_ULA_PREFIX" -o "$WAN6_INTERFACE" -j MASQUERADE
        if [ $? -eq 0 ] ; then
                logger -t NAT6 "Added IPv6 masquerading rule to the firewall (Src: $LAN_ULA_PREFIX - Dst: $WAN6_INTERFACE)"
        else
                logger -t NAT6 "Fatal error: Failed to add IPv6 masquerading rule to the firewall (Src: $LAN_ULA_PREFIX - Dst: $WAN6_INTERFACE)" && exit 1
        fi

        ip -6 route add 2000::/3 via "$WAN6_GATEWAY" dev "$WAN6_INTERFACE"
        if [ $? -eq 0 ] ; then
                logger -t NAT6 "Added $WAN6_GATEWAY to routing table as gateway on $WAN6_INTERFACE for outgoing connections"
        else
                logger -t NAT6 "Error: Failed to add $WAN6_GATEWAY to routing table as gateway on $WAN6_INTERFACE for outgoing connections"
        fi

        if [ $PRIVACY -eq 1 ] ; then
                echo 2 > "/proc/sys/net/ipv6/conf/$WAN6_INTERFACE/accept_ra"
                if [ $? -eq 0 ] ; then
                        logger -t NAT6 "Accepting router advertisements on $WAN6_INTERFACE even if forwarding is enabled (required for temporary addresses)"
                else
                        logger -t NAT6 "Error: Failed to change router advertisements accept policy on $WAN6_INTERFACE (required for temporary addresses)"
                fi
                echo 2 > "/proc/sys/net/ipv6/conf/$WAN6_INTERFACE/use_tempaddr"
                if [ $? -eq 0 ] ; then
                        logger -t NAT6 "Using temporary addresses for outgoing connections on interface $WAN6_INTERFACE"
                else
                        logger -t NAT6 "Error: Failed to enable temporary addresses for outgoing connections on interface $WAN6_INTERFACE"
                fi
        fi

        exit 0
}
```

自己需要再次修改的地方

* 脚本第38行
    ```bash
    WAN6_INTERFACE=$(uci get "network.$WAN6_NAME.ifname")
    ```
    这是用来设置wan6对应的物理网口的,我发现我运行这个脚本会出错,所以我直接根据openwrt,luci界面,网络-接口的显示,写死了这行命令,更改为:
    
    ```bash
    WAN6_INTERFACE=eth1
    ```
    
* 脚本第58行

    ```bash
    WAN6_GATEWAY=$(ip -6 route | grep -o '2001.*1102' | sed s'/1102/1101/g')
    ```

    这是用来设置ipv6网关的,我发现openwrt,luci界面,状态-概览显示我ipv6网关是`fw80`开头的地址,虽然根据脚本原帖[再说 OpenWRT 校园网 IPv6 NAT6-OPENWRT专版-恩山无线论坛 - Powered by Discuz! (right.com.cn)](https://www.right.com.cn/forum/thread-2661027-1-1.html)的评论,这样有时候可能也可以,但是对我来说不够保险

    使用`ip a`查看全部网络连接的详细信息,只看wan下面inet6后面的部分,举例:`inet6 2001:250:4402:2001:2e15:e1ff:fe0e:2a82/128`

    那么ipv6网关应该是`2001:250:4402:2001::1`,写死这行命令,更改为:

    ```
    WAN6_GATEWAY="2001:250:4402:2001::1"
    ```

然后把这个修改后的脚本写入`/etc/init.d/nat6`

设置权限并让其开机启动

```bash
chmod +x /etc/init.d/nat6
/etc/init.d/nat6 enable
```

##### 修改内核参数

修改`/etc/sysctl.conf`,添加下列的配置

```bash
net.ipv6.conf.default.forwarding=2

net.ipv6.conf.all.forwarding=2

net.ipv6.conf.default.accept_ra=2

net.ipv6.conf.all.accept_ra=2
```

##### 修改防火墙

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810214619343.png)

原规则如下

```bash
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 53
iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 53
ip6tables -t nat -A POSTROUTING -o $(uci get network.wan6.ifname) -j MASQUERADE
```

自己需要再次修改的地方

* 第三行

    ```bash
    $(uci get network.wan6.ifname)
    ```

    这是wan6对应的物理网口,直接写死他

    ```bash
    ip6tables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
    ```

##### 重启路由器,测试是否成功

在电脑上面访问[IPv6 测试 (test-ipv6.com)](http://www.test-ipv6.com/)

如果和下图一样,说明配置成功了:

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810215142756.png)

### tor

配置tor的原因很简单,Tor Browser(基于FireFox)浏览器真不好用!

好多插件都不能正常工作,还没有内置的翻译,没垂直标签页,不好看等各种理由,使得我有这样一个想法:软路由上面配置tor网络,开一个代理端口,然后用edge配合插件,让onion后缀的网站都走tor开的那个代理端口

这样我就可以用edge访问互联网上面的全部网页了

(配置tor主要是为了zlibrary(世界最大的电子图书馆),DuckDuckGo(注重隐私的搜索引擎))

在国内使用tor,不仅仅需要代理,还需要配置网桥

#### 基础知识-网桥

什么是网桥?

> 信息来源:[什么是网桥？ | Tor Project | 支持](https://support.torproject.org/zh-CN/censorship/censorship-7/)
>
> 网桥指未在 Tor 公共目录里列出的 Tor 中继节点。

网桥的类型

* obfs4:让 Tor 流量看起来只是随机的数据，但网络审查严格的地区这方法可能无效。
* Snowflake:例如，通过 Snowflake 代理路由连接，让它看起来像是在使用视频通话。
* meek-azure:它将连接伪装成在访问 Microsoft 网站而不是使用 Tor。在网络审查严格的地区仍可使用，但速度通常很慢。

根据我的测试,obfs4在国内被完全阻断了(路由器端连接不上,连接路由器的电脑可以连接上),meek-azure也连接不上(连接路由器的电脑连接不上)

所以我选择了Snowflake作为我的网桥

#### 安装tor和Snowflake网桥

```bash
opkg install tor
opkg install snowflake-client
```

#### 配置tor

编辑`/etc/tor/torrc`

```bash
## 使用系统日志而不是 Tor 的日志文件
Log notice syslog

#设置日志输出
Log notice file /var/log/tor/debug.log

## 保存所有密钥等的目录。默认情况下，Unix 系统将其保存在 $HOME/.tor，Windows 系统将其保存在 Application Data\tor 中。
#DataDirectory /var/lib/tor

## 定义中央服务器的IP和端口，以侦听客户端连接
SocksPort 0.0.0.0:9050

## 使用Snowflake网桥,因为obfs4网桥在我这好像已经被GFW所拦截
UseBridges 1
ClientTransportPlugin snowflake exec /usr/bin/snowflake-client

## 使用Snowflake网桥
Bridge snowflake 192.0.2.3:80 2B280B23E1107BB62ABFC40DDCC8824814F80A72 fingerprint=2B280B23E1107BB62ABFC40DDCC8824814F80A72 url=https://snowflake-broker.torproject.net.global.prod.fastly.net/ front=cdn.sstatic.net ice=stun:stun.l.google.com:19302,stun:stun.antisip.com:3478,stun:stun.bluesip.net:3478,stun:stun.dus.net:3478,stun:stun.epygi.com:3478,stun:stun.sonetel.com:3478,stun:stun.uls.co.za:3478,stun:stun.voipgate.com:3478,stun:stun.voys.nl:3478 utls-imitate=hellorandomizedalpn

## 设置每隔900秒重新连接,以切换tor链路
NewCircuitPeriod 900
```

Snowflake网桥是在电脑上面,使用Tor Browser浏览器得到的

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810222226765.png)

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810222305729.png)

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810222344244.png)

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810222402528.png)

---

#### 设置开机启动tor

```bash
/etc/init.d/nat6 enable
```

#### 重启路由器,测试是否成功

ssh到路由器,输入以下命令查看9050端口的占用情况

```bash
netstat -anp|grep 9050
```

如下图显示,被tor所占用

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810222737481.png)

查看tor日志

```bash
tail -n 10 -f /var/log/tor/debug.log
```

如果看到了下图

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810223543813.png)

说明tor已经配置完成了

#### 下面是如何在edge中使用路由器上面的tor

##### 安装代理用插件

[SmartProxy - Chrome 应用商店 (google.com)](https://chrome.google.com/webstore/detail/smartproxy/jogcnplbkgkfdakgdenhlpcfhjioidoj)

##### 进入扩展的设置

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810223815981.png)

##### 设置代理服务器

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810224022923.png)

参考下图配置代理服务器

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810224049540.png)

设置好后记得点击`保存更改`

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810224141244.png)

##### 设置智能方案

关闭`始终开启`方案,关闭后记得点击`保存更改`

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810224350653.png)

设置`自动切换`方案

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810224609186.png)

参考下图配置规则,配置完,保存,然后记得再点外面那个`保存更改`

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810224715175.png)

##### 设置插件的代理模式为"自动切换"

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810224917707.png)

##### 测试插件是否设置正确

浏览器访问[DuckDuckGo (duckduckgogg42xjoc72x3sjasowoarfbgcmvfimaftt6twagswzczad.onion)](https://duckduckgogg42xjoc72x3sjasowoarfbgcmvfimaftt6twagswzczad.onion/)

如果看到下图,说明配置成功

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810225104488.png)

## MT3000

### wifi

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810225307631.png)

注意:访客wifi与其他wifi和lan口网络不是同一个网段的,不能相互访问

### ipv6

直接选择nat6

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810225440915.png)

### Samba和WebDav

先把usb存储设备分区,格式化再接到MT3000的usb口上面(注意:MT3000的usb3.0口没有做信号屏蔽,同时使用会造成2.4Ghz信号中断,我是用了个延长线让硬盘远离路由器解决的)

先在下图确认磁盘管理里面是否能找到硬盘,然后在`用户管理`里面添加一个用户(这是给samba用的),在`共享文件夹`中选择文件夹,最后在`文件服务`选择协议和端口,点击`应用`就完成了

![](https://cdn.jsdelivr.net/gh/a15355447898a/Blog-photos/image-20230810225919381.png)

