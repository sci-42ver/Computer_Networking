# config the modem and the router
- I use the modem  and router TL-XDR3039
- how to [get](https://www.right.com.cn/forum/thread-7936266-1-1.html) the administrator account and the related password
```bash
# http://192.168.1.1/usr=CMCCAdmin&psw=aDm8H%25MdA&cmd=1&telnet.gch
$ telnet 192.168.1.1
# CMCCAdmin ; aDm8H%MdA
~ $ sidbg 1 DB decry /userconfig/cfg/db_user_cfg.xml
~ $ tftp -l /tmp/debug-decry-cfg -p 192.168.0.100
# failed
~ $ vi /tmp/debug-decry-cfg
# search 1. IGD.AU1 the corresponding User,Pass is the administrator account and password
...
<DM name="User" val="foo"/>       
<DM name="Pass" val="bar"/>
...
# search 2. IGD.WD1.WCD1.WCPPP1 the corresponding UserName,Password is the PPPOE account and password https://www.tp-link.com/us/support/faq/410/
<DM name="UserName" val="foo"/>
<DM name="Password" val="bar"/>
```
- then config the modem, better ["新建WAN连接"](https://www.youtube.com/watch?v=1cVIXdmPQMA) ([doc](https://www.cnblogs.com/yaoyue68/p/16815152.html)) instead of modifying/deleting the original ["2_INTERNET_R_VID_..."](https://ipw.cn/doc/ipv6/user/enable_ipv6.html)
  1. allow WAN
  2. the rest see the figure in the above doc link and "2_INTERNET_R_VID_..." cfg in the modem.
  then use one wire to collect LAN *binded* on the modem to the WAN in the router.
  - "\_R\_" [meaning](https://zhuanlan.zhihu.com/p/146528034?utm_id=0) -> relay.
  - different mode like 2_INTERNET_R_VID_... [meaning](https://sspai.com/post/78387)
- the router uses [method 1](https://resource.tp-link.com.cn/pc/docCenter/showDoc?id=1655112591200293)
 (the 2 changes in each method are enough to work)
 - TODO sometimes I connect directly to the modem, I will get the ipv6 but sometimes not.
   - maybe influenced by 1_TR..._R_VID...
## tftp [diff](https://www.geeksforgeeks.org/difference-between-ftp-and-tftp/) ftp
- ftp [config](https://www.geeksforgeeks.org/how-to-setup-and-configure-an-ftp-server-in-linux-2/)
```bash
$ ftp localhost
```
- TODO [config](https://www.right.com.cn/forum/forum.php?mod=redirect&goto=findpost&ptid=7936266&pid=19283742) tftp
## v2ray
- [better](https://baiyunju.cc/7256) use ASIS to avoid the heavy [DNS lookup](https://xtls.github.io/en/document/level-1/routing-lv1-part2.html#_8-%E6%98%8E%E4%BF%AE%E6%A0%88%E9%81%93%E3%80%81%E6%9A%97%E6%B8%A1%E9%99%88%E4%BB%93)
  but I uses geoip, so better "IPIfNonMatch"
  - TODO [detailed](https://github.com/v2ray/discussion/issues/770)
- [flush](https://tecadmin.net/flush-dns-cache-ubuntu/) dns with [preparation](https://superuser.com/a/1427312)
- notice the ws is not necessary [with tls](https://guide.v2fly.org/en_US/advanced/wss_and_web.html#client-side-configuration) which can be viewed in windows v2rayn exported config if using one subscription.
  Also see this [example](https://github.com/v2fly/v2ray-examples/blob/4cc09a4977169a1c55f668217934da8e0208967a/VMess-Websocket-TLS/config_client.json#L53)
- Also see real config [json](https://bitbucket.org/anom_mony/automatic_command/src/3348b916ae847dd11d4745f6efb2df88a13de4c5/arch_linux_init/net_conf/config.json?at=master)
# result
- it seems to have one public ipv6 addr now by [test](https://www.test-ipv6.com/)
  > Your IPv6 address on the public Internet appears to be 2409:8a20:120:c620::1000
```bash
$ ifconfig
...
wlp4s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      inet 192.168.0.101  netmask 255.255.255.0  broadcast 192.168.0.255
      inet6 2409:8a20:120:c620:6329:6dac:3f52:772e  prefixlen 64  scopeid 0x0<global>
      inet6 2409:8a20:120:c620::1000  prefixlen 128  scopeid 0x0<global>
      inet6 fe80::7c02:594d:1cc0:fde4  prefixlen 64  scopeid 0x20<link>
# https://www.cyberciti.biz/faq/check-for-ipv6-support-in-linux-kernel/
$ ping -c 2 -6 www.cyberciti.biz
PING www.cyberciti.biz(2606:4700:10::6816:3fa6 (2606:4700:10::6816:3fa6)) 56 data bytes
64 bytes from 2606:4700:10::6816:3fa6 (2606:4700:10::6816:3fa6): icmp_seq=1 ttl=53 time=234 ms
64 bytes from 2606:4700:10::6816:3fa6 (2606:4700:10::6816:3fa6): icmp_seq=2 ttl=53 time=234 ms

--- www.cyberciti.biz ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 233.607/234.004/234.402/0.397 ms
# https://serverfault.com/a/217266
$ telnet -6 2401:c080:1000:4720:5400:04ff:fea2:3610 12345
Trying 2401:c080:1000:4720:5400:4ff:fea2:3610...
Connected to 2401:c080:1000:4720:5400:04ff:fea2:3610.
Escape character is '^]'.
# check dns https://unix.stackexchange.com/a/233540/568529
$ dig +short @8.8.8.8 AAAA www.cyberciti.biz
2606:4700:10::6816:3fa6
2606:4700:10::ac43:1803
2606:4700:10::6816:3ea6
# https://superuser.com/a/1664770 probably the server doesn't allow ping.
$ ping -c 2 -6 2401:c080:1000:4720:5400:04ff:fea2:3610
PING 2401:c080:1000:4720:5400:04ff:fea2:3610(2401:c080:1000:4720:5400:4ff:fea2:3610) 56 data bytes

--- 2401:c080:1000:4720:5400:04ff:fea2:3610 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1026ms
```
- Also see [this](https://dnschecker.org/ipv6-whois-lookup.php)
  - here use [AS](https://hackertarget.com/as-ip-lookup/) where the location is not necessary to be same as the geolocation
    AS shown by `whois fe80::7c02:594d:1cc0:fde4` -> "Unknown AS number or IP network. Please upgrade this program."
```bash
$ ifconfig
...
wlp4s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.101  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 2409:8a20:121:3850::1001  prefixlen 128  scopeid 0x0<global>
        inet6 2409:8a20:121:3850:d45e:bf9e:5af3:b425  prefixlen 64  scopeid 0x0<global>
...
$ whois 2409:8a20:121:3850::1001
...
irt:            IRT-CHINAMOBILE-CN
address:        China Mobile Communications Corporation
address:        29, Jinrong Ave., Xicheng District, Beijing, 100032
e-mail:         abuse@chinamobile.com
abuse-mailbox:  abuse@chinamobile.com
admin-c:        CT74-AP
tech-c:         CT74-AP
auth:           # Filtered
remarks:        abuse@chinamobile.com was validated on 2023-08-02
mnt-by:         MAINT-CN-CMCC
last-modified:  2023-08-02T01:32:37Z
source:         APNIC
...
$ dig +short @8.8.8.8 AAAA www.cyberciti.biz      
2606:4700:10::ac43:1803
2606:4700:10::6816:3fa6
2606:4700:10::6816:3ea6
# TODO ipv6 same for dig and ping, but not for ipv4, see the following.
$ ping -c 2 -6 www.cyberciti.biz -v
ping: sock4.fd: -1 (socktype: 0), sock6.fd: 3 (socktype: SOCK_DGRAM), hints.ai_family: AF_INET6

ai->ai_family: AF_INET6, ai->ai_canonname: 'www.cyberciti.biz'
PING www.cyberciti.biz(2606:4700:10::6816:3ea6) 56 data bytes
[hervey_arch ~/Computer_Networking]$ whois 2606:4700:10::ac43:1803
OrgName:        Cloudflare, Inc.
OrgId:          CLOUD14
Address:        101 Townsend Street
City:           San Francisco
StateProv:      CA
PostalCode:     94107
Country:        US
RegDate:        2010-07-09
Updated:        2021-07-01
Ref:            https://rdap.arin.net/registry/entity/CLOUD14

################# ipv4
$ whois 112.2.79.103 # by https://www.test-ipv6.com/ ipv4 result
irt:            IRT-CHINAMOBILE2-CN
address:        China Mobile Communications Corporation
address:        29, Jinrong Ave., Xicheng District, Beijing, 100032
e-mail:         abuse@chinamobile.com
abuse-mailbox:  abuse@chinamobile.com
admin-c:        ct74-AP
tech-c:         CT74-AP
auth:           # Filtered
remarks:        abuse@chinamobile.com was validated on 2023-08-02
mnt-by:         MAINT-CN-CMCC
last-modified:  2023-08-02T01:32:37Z
source:         APNIC

$ whois 192.168.0.101 # this is local addr
OrgName:        Internet Assigned Numbers Authority
OrgId:          IANA
Address:        12025 Waterfront Drive
Address:        Suite 300
City:           Los Angeles
StateProv:      CA
PostalCode:     90292
Country:        US
RegDate:        
Updated:        2012-08-31
Ref:            https://rdap.arin.net/registry/entity/IANA

$ cat /etc/resolv.conf 
# Generated by NetworkManager
#nameserver 202.119.248.66
nameserver 8.8.8.8
nameserver 2001:4860:4860::8888
$ dig +short @8.8.8.8 A www.baidu.com       
www.a.shifen.com.
36.155.132.55
36.155.132.31
# TODO why different between ping , dig based on resolv.conf and traceroute
[hervey_arch ~/Computer_Networking]$ ping baidu.com       
PING baidu.com (39.156.66.10) 56(84) bytes of data.
$ traceroute baidu.com
traceroute to baidu.com (110.242.68.66), 30 hops max, 60 byte packets
$ whois 39.156.66.10 # same as above public addr by https://www.test-ipv6.com/
irt:            IRT-CHINAMOBILE-CN
address:        China Mobile Communications Corporation
address:        29, Jinrong Ave., Xicheng District, Beijing, 100032
e-mail:         abuse@chinamobile.com
abuse-mailbox:  abuse@chinamobile.com
admin-c:        CT74-AP
tech-c:         CT74-AP
auth:           # Filtered
remarks:        abuse@chinamobile.com was validated on 2023-08-02
mnt-by:         MAINT-CN-CMCC
last-modified:  2023-08-02T01:32:37Z
source:         APNIC

```