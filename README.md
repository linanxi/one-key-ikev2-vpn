# 一键搭建适用于Ubuntu/CentOS的IKEV2/L2TP的VPN

------
[![Author](https://img.shields.io/badge/author-%40quericy-blue.svg)](https://quericy.me)   [![Platform](https://img.shields.io/badge/Platform-%20Ubuntu%2CCentOS%20-green.svg)]()  [![GitHub stars](https://img.shields.io/github/stars/quericy/one-key-ikev2-vpn.svg)](https://github.com/quericy/one-key-ikev2-vpn/stargazers)  [![GitHub license](https://img.shields.io/badge/license-GPLv3-yellowgreen.svg)](https://raw.githubusercontent.com/quericy/one-key-ikev2-vpn/master/LICENSE)

使用bash脚本一键搭建Ikev2的vpn服务端.

特性
=============
> * 服务端要求：Ubuntu或者CentOS-6/7或者Debian
> * 客户端：
 - iOS/OSX=>ikev1,ikev2
 - Andriod=>ikev1
 - WindowsPhone=>ikev2
 - 其他Windows平台=>ikev2
> * 可使用自己的私钥和根证书，也可自动生成
> * 证书可绑定域名或ip
> * 要是图方便可一路回车

最近更新
==========
> - 添加SSL证书自动申请自动更新并应用于IKEv2的解决方案,详见这篇博文:[SSL证书自动更新并应用到IKEv2, Nginx](https://quericy.me/blog/860/) ;
> - 添加对CentOS7的firewall防火墙的支持;
> - 使用ip address替换已被废弃的ifconfig;
> - 生成单独的sysctl配置文件/etc/sysctl.d/10-ipsec.conf单独加载，用于开启ipv4转发(如以后卸载或需要关闭net.ipv4.ip_forward,请记得删除此文件);
> - 升级strongswan版本到5.5.1,解决iOS9和iOS10的兼容性问题(感谢[caasiu](https://github.com/caasiu)的提醒[#21](https://github.com/quericy/one-key-ikev2-vpn/issues/21));
> - 添加导入SSL证书的支持,安装时可选使用证书颁发机构签发的证书还是生成自签名证书;

服务端安装说明
==========
1. 下载脚本:
    ```shell
    wget --no-check-certificate https://raw.githubusercontent.com/quericy/one-key-ikev2-vpn/master/one-key-ikev2.sh
    ```
    * 注:如需使用其他分支的脚本,请将上述url中的master修改为分支名称,各分支区别详见本页的[分支说明](#分支说明)节点

2. 运行脚本：
    ```shell
    chmod +x one-key-ikev2.sh
    bash one-key-ikev2.sh
    ```

3. 等待自动配置部分内容后，选择vps类型（OpenVZ还是Xen、KVM），**选错将无法成功连接，请务必核实服务器的类型**。输入服务器ip或者绑定的域名(连接vpn时服务器地址将需要与此保持一致,如果是导入泛域名证书这里需要写`*.域名`的形式)；

4. 选择使用使用证书颁发机构签发的SSL证书还是生成自签名证书：

    - 如果选择no,`使用自签名证书`（客户端如果使用IkeV2方式连接，将需要导入生成的证书并信任）则需要填写证书的相关信息(C,O,CN)，为空将使用默认值(default value)，确认无误后按任意键继续,后续安装过程中会出现输入两次pkcs12证书的密码的提示(可以设置为空)

    - 如果选择yes，`使用SSL证书`（如果证书是被信任的，后续步骤客户端将无需导入证书）请在继续下一步之前，将以下文件按提示命名并放在**脚本相同的目录下**（SSL证书详细配置和自动续期方案可见[https://quericy.me/blog/860/](https://quericy.me/blog/860/) ）：
        1. **ca.cert.pem** 证书颁发机构的CA，比如Let‘s Encrypt的证书,或者其他链证书；
        2. **server.cert.pem** 签发的域名证书；
        3. **server.pem** 签发域名证书时用的私钥；

5. 是否使用SNAT规则(可选).默认为不使用.使用前请确保服务器具有不变的**静态公网ip**,可提升防火墙对数据包的处理速度.如果服务器网络设置了NAT(如AWS的弹性ip机制),则填写网卡连接接口的ip地址(参见[KinonC](https://github.com/KinonC)提供的方案:[#36](https://github.com/quericy/one-key-ikev2-vpn/issues/36)).

6. 防火墙配置.默认配置iptables(如果使用的是firewall(如CentOS7)请选择yes自动配置firewall,将无视SNAT并跳过后续的补充网卡接口步骤).补充网卡接口信息,为空则使用默认值(Xen、KVM默认使用eth0,OpenVZ默认使用venet0).如果服务器使用其他公网接口需要在此指定接口名称,**填写错误VPN连接后将无法访问外网**)

7. 看到install Complete字样即表示安装完成。默认用户名密码将以黄字显示，可根据提示自行修改配置文件中的用户名密码,多用户则在配置文件中按格式一行一个(多用户时用户名不能使用%any),保存并重启服务生效。

8. 将提示信息中的证书文件ca.cert.pem拷贝到客户端，修改后缀名为.cer后导入。ios设备使用Ikev1无需导入证书，而是需要在连接时输入共享密钥，共享密钥即是提示信息中的黄字PSK.

客户端配置说明
=====
* 连接的服务器地址和证书保持一致,即取决于签发证书ca.cert.pem时使用的是ip还是域名;
 
* **Android/iOS/OSX** 可使用ikeV1,认证方式为用户名+密码+预共享密钥(PSK);

* **iOS/OSX/Windows7+/WindowsPhone8.1+/Linux** 均可使用IkeV2,认证方式为用户名+密码。`使用SSL证书`则无需导入证书；`使用自签名证书`则需要先导入证书才能连接,可将ca.cert.pem更改后缀名作为邮件附件发送给客户端,手机端也可通过浏览器导入,其中:
 * **iOS/OSX** 的远程ID和服务器地址保持一致,用户鉴定选择"用户名".如果通过浏览器导入,将证书放在可访问的远程外链上,并在**系统浏览器**(Safari)中访问外链地址.OSX证书需要设置为始终信任(添加方法见**[#58](https://github.com/quericy/one-key-ikev2-vpn/issues/58)**中[JiaHaoGong](https://github.com/JiaHaoGong)的截图);
 * **Windows PC** 系统导入证书需要导入到**"本地计算机"**的"受信任的根证书颁发机构",以"当前用户"的导入方式是无效的.推荐运行mmc添加本地计算机的证书管理单元来操作;
 * **WindowsPhone8.1** 登录时的用户名需要带上域信息,即wp"关于"页面的设备名称\用户名,也可以使用%any %any : EAP "密码"进行任意用户名登录,但指定了就不能添加其他用户名了.
 * **WindowsPhone10** ~~的vpn还存在bug(截至10586.164),ikeV2方式可连接但系统流量不会走vpn,只能等微软解决.~~ (截至14393.5 ,此bug已经得到修复,现在WP10已经可以正常使用IkeV2.)
 * **Windows10** 也存在此bug,部分Win10系统连接后ip不变,没有自动添加路由表,使用以下方法可解决(本方法由 bigbigfish 童鞋提供):
    * 手动关闭vpn的split tunneling功能(在远程网络上使用默认网关);
    * 也可使用powershell修改,进入CMD窗口,运行如下命令:
    ```powershell
    powershell    #进入ps控制台
    get-vpnconnection    #检查vpn连接的设置（包括vpn连接的名称）
    set-vpnconnection "vpn连接名称" -splittunneling $false    #关闭split tunneling
    get-vpnconnection   #检查修改结果
    exit   #退出ps控制台
    ```

卸载方式
===
1. 进入脚本所在目录的strongswan文件夹执行:
    ```bash
    make uninstall
    ```

2. 删除脚本所在目录的相关文件(one-key-ikev2.sh,strongswan.tar.gz,strongswan文件夹,my_key文件夹).

3. 卸载后记得检查iptables配置.

分支说明
==========
* [master](https://github.com/quericy/one-key-ikev2-vpn/tree/master)分支:经过测试的相对稳定的版本;
* dev-debian分支:~~Debian6/7测试分支,该脚本由[bestoa](https://github.com/bestoa)修改提供~~.主分支已提供对Debian的支持,该分支已废弃:[#59](https://github.com/quericy/one-key-ikev2-vpn/issues/59);
* [dev](https://github.com/quericy/one-key-ikev2-vpn/tree/dev)分支:开发分支,使用最新版本的strongswan,未进过充分测试,用于尝试和添加一些新的功能,未来可能添加对L2TP的兼容支持,以及对ipv6的支持;

部分问题解决方案
======
* ipsec启动问题：服务器重启后默认ipsec不会自启动，请命令手动开启,或添加/usr/local/sbin/ipsec start到自启动脚本文件中(如rc.local等)：
    ```bash
    ipsec start
    ```

* ipsec常用指令:
    ```bash
    ipsec start   #启动服务
    ipsec stop    #关闭服务
    ipsec restart #重启服务
    ipsec reload  #重新读取
    ipsec status  #查看状态
    ipsec --help  #查看帮助
    ```

* 可连接但是无法访问网络：
    - 检查iptables是否正常启用,检查iptables规则是否与其他地方冲突,或根据服务器防火墙的实际情况手动修改配置。
    - 检查sysctl是否开启ip_forward:
        1. 打开sysctl文件:`vim /etc/sysctl.conf`                
        2. 修改net.ipv4.ip_forward=1后保存并关闭文件    
        3. 使用以下指令刷新sysctl：`sysctl -p`
        4. 如执行后正常回显则表示生效。如显示错误信息，请重新打开/etc/syctl并根据错误信息对应部分用#号注释，保存后再刷新sysctl直至不会报错为止。

* 如果之前使用的自签名证书，后改用SSL证书，部分客户端可能需要卸载之前安装的自签名证书,否则可能会报`Ike凭证不可接受`的错误:
    * iOS：设置-通用，删除证书对应的描述文件即可；
    * Windows：Win+R,运行mmc打开Microsoft管理控制台,文件->添加管理单元,添加证书管理单元(必须选计算机账户),展开受信任的根证书颁发机构,找到对应的自签名证书,右键删除即可;
    * Windows Phone:暂时没有找到可以卸载证书的方法(除非越狱),目前只能重置来解决此问题;

* * *

如有其他疑问请戳本人博客：[https://quericy.me/blog/699](https://quericy.me/blog/699)




# 目录

1. 安装说明
2. 特性
3. 最近更新
4. 服务端安装说明
5. 客户端配置说明
6. 卸载方式
7. 分支说明
8. 部分问题解决方案
9. PS
10. PS2

# 安装说明

用法很简单：
总结成一句话就是：**除了类型要选对以外，其他的一路回车就好了**23333

# 特性

> - 服务端要求：Ubuntu或者CentOS-6/7或者Debian
> - 客户端：
>   - iOS/OSX=>ikev1,ikev2
>   - Andriod=>ikev1
>   - WindowsPhone=>ikev2
>   - 其他Windows平台=>ikev2
> - 可使用自己的私钥和根证书，也可自动生成
> - 证书可绑定域名或ip
> - 要是图方便可一路回车

# 最近更新

> - 添加SSL证书自动申请自动更新并应用于IKEv2的解决方案,详见这篇博文:SSL证书自动更新并应用到IKEv2, Nginx

配置方法：

更新到 ikev2

```
#! /bin/bash
cert_file="/home/user/.acme.sh/yourdomain/yourdomain.cer"
key_file="/home/user/.acme.sh/yourdomain/yourdomain.key"

sudo cp -f $cert_file /usr/local/etc/ipsec.d/certs/server.cert.pem
sudo cp -f $key_file /usr/local/etc/ipsec.d/private/server.pem
sudo cp -f $cert_file /usr/local/etc/ipsec.d/certs/client.cert.pem
sudo cp -f $key_file /usr/local/etc/ipsec.d/private/client.pem
sudo /usr/local/sbin/ipsec restart
```

编写定时任务:

```
59 02 1 * * bash /your/path/to/ipsec.sh > /dev/null

每个月1日的凌晨2点59分就会替换证书并重启IPsec服务.也可以根据自己的需求自行调整参数.
```

更新到nginx

```
#! /bin/bash
sudo cp -f /home/user/.acme.sh/yourdomain/fullchain.cer /path/to/nginx/ssl/fullchain.pem
sudo cp -f /home/user/.acme.sh/yourdomain/yourdomain.key /path/to/nginx/ssl/privatekey.pem
sudo cp -f /home/user/.acme.sh/yourdomain_ecc/fullchain.cer /path/to/nginx/ssl/fullchain.pem
sudo cp -f /home/user/.acme.sh/yourdomain_ecc/yourdomain.key /path/to/nginx/ssl/privatekey.pem
sudo service nginx restart
```



> - 添加对CentOS7的firewall防火墙的支持;
> - 使用ip address替换已被废弃的ifconfig;
> - 生成单独的sysctl配置文件/etc/sysctl.d/10-ipsec.conf单独加载，用于开启ipv4转发(如以后卸载或需要关闭net.ipv4.ip_forward,请记得删除此文件);
> - 升级strongswan版本到5.5.1,解决iOS9和iOS10的兼容性问题(感谢[caasiu](https://github.com/caasiu)的提醒[#21](https://github.com/quericy/one-key-ikev2-vpn/issues/21));
> - 添加导入SSL证书的支持,安装时可选使用证书颁发机构签发的证书还是生成自签名证书;

# 服务端安装说明

1. 下载脚本:

   ​

   ```
   wget --no-check-certificate https://raw.githubusercontent.com/quericy/one-key-ikev2-vpn/master/one-key-ikev2.sh
   ```

   - 注:如需使用其他分支的脚本,请将上述url中的master修改为分支名称,各分支区别详见本页的分支说明节点

2. 运行脚本：

   ​

   ```
   chmod +x one-key-ikev2.sh
   bash one-key-ikev2.sh
   ```

3. 等待自动配置部分内容后，选择vps类型（OpenVZ还是Xen、KVM），**选错将无法成功连接，请务必核实服务器的类型**。输入服务器ip或者绑定的域名(连接vpn时服务器地址将需要与此保持一致,如果是导入泛域名证书这里需要写`*.域名`的形式)；

   ```
   在 ip or domain 后面添加你的域名
   ```

   ​

4. 选择使用使用证书颁发机构签发的SSL证书还是生成自签名证书：

   - 如果选择no,`使用自签名证书`（客户端如果使用IkeV2方式连接，将需要导入生成的证书并信任）则需要填写证书的相关信息(C,O,CN)，为空将使用默认值(default value)，确认无误后按任意键继续,后续安装过程中会出现输入两次pkcs12证书的密码的提示(可以设置为空)

   - 如果选择yes，

     ```
     使用SSL证书
     ```

     （如果证书是被信任的，后续步骤客户端将无需导入证书）请在继续下一步之前，将以下文件按提示命名并放在

     脚本相同的目录下

     （SSL证书详细配置和自动续期方案可见）：

     1. **ca.cert.pem** 证书颁发机构的CA，比如Let‘s Encrypt的证书,或者其他链证书；
     2. **server.cert.pem** 签发的域名证书；
     3. **server.pem** 签发域名证书时用的私钥；

5. 是否使用SNAT规则(可选).默认为不使用.使用前请确保服务器具有不变的**静态公网ip**,可提升防火墙对数据包的处理速度.如果服务器网络设置了NAT(如AWS的弹性ip机制),则填写网卡连接接口的ip地址(参见[KinonC](https://github.com/KinonC)提供的方案:[#36](https://github.com/quericy/one-key-ikev2-vpn/issues/36)).

6. 防火墙配置.默认配置iptables(如果使用的是firewall(如CentOS7)请选择yes自动配置firewall,将无视SNAT并跳过后续的补充网卡接口步骤).补充网卡接口信息,为空则使用默认值(Xen、KVM默认使用eth0,OpenVZ默认使用venet0).如果服务器使用其他公网接口需要在此指定接口名称,**填写错误VPN连接后将无法访问外网**)

7. 看到install Complete字样即表示安装完成。默认用户名密码将以黄字显示，可根据提示自行修改配置文件中的用户名密码,多用户则在配置文件中按格式一行一个(多用户时用户名不能使用%any),保存并重启服务生效。

8. 将提示信息中的证书文件ca.cert.pem拷贝到客户端，修改后缀名为.cer后导入。ios设备使用Ikev1无需导入证书，而是需要在连接时输入共享密钥，共享密钥即是提示信息中的黄字PSK.
   [![ikev2_VPN_install](https://dn-quericyblog.qbox.me/wp-content/uploads/2015/06/ikev2_VPN_install-150x150.jpg)](https://dn-quericyblog.qbox.me/wp-content/uploads/2015/06/ikev2_VPN_install.jpg)

# 客户端配置说明

- 连接的服务器地址和证书保持一致,即取决于签发证书ca.cert.pem时使用的是ip还是域名;

- **Android/iOS/OSX** 可使用ikeV1,认证方式为用户名+密码+预共享密钥(PSK);

- iOS/OSX/Windows7+/WindowsPhone8.1+/Linux

   均可使用IkeV2,认证方式为用户名+密码。

  ```
  使用SSL证书
  ```

  则无需导入证书；

  ```
  使用自签名证书
  ```

  则需要先导入证书才能连接,可将ca.cert.pem更改后缀名作为邮件附件发送给客户端,手机端也可通过浏览器导入,其中:

  - **iOS/OSX** 的远程ID和服务器地址保持一致,用户鉴定选择”用户名”.如果通过浏览器导入,将证书放在可访问的远程外链上,并在**系统浏览器**(Safari)中访问外链地址;

  - **Windows PC** 系统导入证书需要导入到**“本地计算机”**的”受信任的根证书颁发机构”,以”当前用户”的导入方式是无效的.推荐运行mmc添加本地计算机的证书管理单元来操作;

  - **WindowsPhone8.1** 登录时的用户名需要带上域信息,即wp”关于”页面的设备名称\用户名,也可以使用%any %any : EAP “密码”进行任意用户名登录,但指定了就不能添加其他用户名了.

  - **WindowsPhone10** ~~的vpn还存在bug(截至10586.164),ikeV2方式可连接但系统流量不会走vpn,只能等微软解决.~~ (截至14393.5 ,此bug已经得到修复,现在WP10已经可以正常使用IkeV2.)

  - Windows10

     也存在此bug,部分Win10系统连接后ip不变,没有自动添加路由表,使用以下方法可解决(本方法由 bigbigfish 童鞋提供):

    - 手动关闭vpn的split tunneling功能(在远程网络上使用默认网关);

    - 也可使用powershell修改,进入CMD窗口,运行如下命令:

      ​

      ```
      powershell    #进入ps控制台
      get-vpnconnection    #检查vpn连接的设置（包括vpn连接的名称）
      set-vpnconnection "vpn连接名称" -splittunneling $false    #关闭split tunneling
      get-vpnconnection   #检查修改结果
      exit   #退出ps控制台
      ```

# 卸载方式

1. 进入脚本所在目录的strongswan文件夹执行:

   ​

   ```
   make uninstall
   ```

2. 删除脚本所在目录的相关文件(one-key-ikev2.sh,strongswan.tar.gz,strongswan文件夹,my_key文件夹).

3. 卸载后记得检查iptables配置.

# 分支说明

- [master](https://github.com/quericy/one-key-ikev2-vpn/tree/master)分支:经过测试的相对稳定的版本;
- [dev-debian](https://github.com/quericy/one-key-ikev2-vpn/tree/dev-debian)分支:Debian6/7测试分支,该脚本由[bestoa](https://github.com/bestoa)修改提供;
- [dev](https://github.com/quericy/one-key-ikev2-vpn/tree/dev)分支:开发分支,使用最新版本的strongswan,未进过充分测试,用于尝试和添加一些新的功能,未来可能添加对L2TP的兼容支持,以及对ipv6的支持;

# 部分问题解决方案

- ipsec启动问题：服务器重启后默认ipsec不会自启动，请命令手动开启,或添加/usr/local/sbin/ipsec start到自启动脚本文件中(如rc.local等)：

  ​

  ```
  ipsec start
  ```

- ipsec常用指令:

  ​

  ```
  ipsec start   #启动服务
  ipsec stop    #关闭服务
  ipsec restart #重启服务
  ipsec reload  #重新读取
  ipsec status  #查看状态
  ipsec --help  #查看帮助
  ```

- 可连接但是无法访问网络：

  - 检查iptables是否正常启用,检查iptables规则是否与其他地方冲突,或根据服务器防火墙的实际情况手动修改配置。
  - 检查sysctl是否开启ip_forward:
    1. 打开sysctl文件:`vim /etc/sysctl.conf`
    2. 修改net.ipv4.ip_forward=1后保存并关闭文件
    3. 使用以下指令刷新sysctl：`sysctl -p`
    4. 如执行后正常回显则表示生效。如显示错误信息，请重新打开/etc/syctl并根据错误信息对应部分用#号注释，保存后再刷新sysctl直至不会报错为止。

- 如果之前使用的自签名证书，后改用SSL证书，部分客户端可能需要卸载之前安装的自签名证书,否则可能会报

  ```
  Ike凭证不可接受
  ```

  的错误:

  - iOS：设置-通用，删除证书对应的描述文件即可；
  - Windows：Win+R,运行mmc打开Microsoft管理控制台,文件->添加管理单元,添加证书管理单元(必须选计算机账户),展开受信任的根证书颁发机构,找到对应的自签名证书,右键删除即可;
  - Windows Phone:暂时没有找到可以卸载证书的方法(除非越狱),目前只能重置来解决此问题;

- 要注意一下阿里云要配置一下网络安全组规则

  主要开放 500 和 4500 端口

- 如果服务器网站无法访问配置一下firewalld防火墙配置（点击这里跳过去)

  - 启动： systemctl start firewalld

  - 添加

    firewall-cmd –zone=public –add-port=80/tcp –permanent    （–permanent永久生效，没有此参数重启后失效）

    重新载入

    firewall-cmd –reload

- 可能需要命令

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```



* 如果使用公司网络或小区网络， 可尝试先使用4G调试通过。





### 启用日志

```
vi /usr/local/etc/strongswan.d/charon-logging.conf 
```

```
charon {
    filelog {

        /var/log/strongswan.log {
            append = yes
            default = 1
            flush_line = yes
            ike_name = yes
            time_format = %b %e %T
        }
```





### 系统环境：

本文中搭建VPN服务器使用的操作系统为阿里云ECS的centos7.2版本

### 软件安装：

安装strongswan和xl2tpd:

```
yum install strongswan xl2tpd
```

### 软件配置：

IPsec配置

/usr/local/etc/ipsec.conf 如下：

```
config setup
conn %default
	ikelifetime=60m
	keylife=20m
	rekeymargin=3m
	keyingtries=1
conn l2tp
	keyexchange=ikev1 # IKE版本
	left=<服务器外网ip>
	leftsubnet=0.0.0.0/0
	leftprotoport=17/1701
	authby=secret
	leftfirewall=no
	right=%any
	rightprotoport=17/%any
	type=transport
	auto=add
```
 /usr/local/etc/ipsec.secrets 如下
 
```
# ipsec.secrets - strongSwan IPsec secrets file
: PSK "xxx"  # 自己随意填写密钥
```

L2TP配置

/etc/xl2tpd/xl2tpd.conf 如下：

```
[lns default]
ip range = 10.31.3.2-10.31.3.254
local ip = 10.31.3.1
require chap = yes
refuse pap = yes
require authentication = yes
name = LinuxVPNserver
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```
PPP配置
/etc/ppp/options.xl2tpd 如下：

```
ipcp-accept-local
ipcp-accept-remote
ms-dns  8.8.8.8
noccp
auth
crtscts
idle 1800
mtu 1410
mru 1410
nodefaultroute
debug
lock
proxyarp
connect-delay 5000
```
/etc/ppp/chap-secrets 如下：
```
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
username        *       password                *
```

```
iptables -t nat -A POSTROUTING -s 10.31.3.0/24 -o eth0 -j MASQUERADE
systemctl start strongswan.service
systemctl start xl2tpd.service
```





## 兼容 L2TP

### [#](https://linsir.org/post/how_to_install_IPSec_IKEV2_base_on_strongswan_with_CentOS7#%E5%AE%89%E8%A3%85-xl2tpd) 安装 xl2tpd

```
yum install xl2tpd

```

如果提示找不到安装包，我们还可以手动下载安装。

```
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/x/xl2tpd-1.3.6-8.el7.x86_64.rpm
rpm -ihv xl2tpd-1.3.6-8.el7.x86_64.rpm

```

### [#](https://linsir.org/post/how_to_install_IPSec_IKEV2_base_on_strongswan_with_CentOS7#etcstrongswanipsecconf) /usr/local/etc/ipsec.conf

在/usr/local/etc/ipsec.conf最后添加



```

```



```
conn iOS_cert
    keyexchange=ikev1
    fragmentation=yes
    left=%defaultroute
    leftauth=pubkey
    leftsubnet=0.0.0.0/0
    leftcert=server.cert.pem
    right=%any
    rightauth=pubkey
    rightauth2=xauth
    rightsourceip=10.31.2.0/24
    rightcert=client.cert.pem
    auto=add


conn l2tp
    keyexchange=ikev1 #IKE协议版本
    left=%defaultroute #服务器IP，自己修改为你自己的
    leftsubnet=0.0.0.0/0
    leftprotoport=17/1701
    authby=secret #PSK认证
    leftfirewall=no #不允许strongSwan更改防火墙规则
    right=%any
    rightprotoport=17/%any
    type=transport # ipsec transport mode
    auto=add
```

### [#](https://linsir.org/post/how_to_install_IPSec_IKEV2_base_on_strongswan_with_CentOS7#xl2tpdconf) xl2tpd.conf

vim /etc/xl2tpd/xl2tpd.conf

```
[global]
ipsec saref = no
#listen-addr = 172.18.48.31
port =1701


[lns default]
ip range = 10.31.3.2-10.31.3.254
local ip = 10.31.3.1
assign ip = yes
require chap = yes
refuse pap = yes
require authentication = yes
name = xl2tpd
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```

这里注意下 ip range不要跟上面的strongswan冲突了，不然会部分上不了网。

### [#](https://linsir.org/post/how_to_install_IPSec_IKEV2_base_on_strongswan_with_CentOS7#optionsxl2tpd) options.xl2tpd

```
vim /etc/ppp/options.xl2tpd 


ipcp-accept-local
ipcp-accept-remote
ms-dns  8.8.8.8
ms-dns  8.8.4.4
noccp
auth
crtscts
idle 1800
mtu 1460
mru 1460
nodefaultroute
debug
lock
proxyarp
connect-delay 5000

```



## iptables 配置

```
vi /etc/sysconfig/iptables-config
```

```
iptables -L -n --line-numbers
```



```
删除FORWARD第三行 iptables -D FORWARD 3
```


