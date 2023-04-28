---
title: ファイアウォールでpingの疎通確認を拒否する
date: 2021-02-23 20:43:19
tags:
---

CentOS8のfirewall-cmdでpingの疎通確認を拒否を試してみました。
pingはicmpのプロトコルです。

firewallの設定内容を確認します。
```
[root@localhost unamu]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: cockpit dhcpv6-client http ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

icmp-block-inversionがyesの時にicmp-blocksに記載されたICMP Typeを受け入れ、  
noの時はicmp-blocksに記載されたICMP Typeを拒否します。  
icmp-blocksに何も設定されていないので、icmp-block-inversionをyesにすることですべてのICMP Typeを拒否することで
pingを拒否できそうです。  

icmp-block-inversionをyesにする。
```
[root@localhost unamu]# firewall-cmd --add-icmp-block-inversion
success
[root@localhost unamu]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: yes
  interfaces: enp0s3
  sources:
  services: cockpit dhcpv6-client http ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```
icmp-block-inversionが yes になったので、pingを実行して拒否されるか確認します。
```
~/D/g/D/laradock ❯❯❯ ping 192.168.11.17                                            (git)-[master]
PING 192.168.11.17 (192.168.11.17): 56 data bytes
92 bytes from 192.168.11.17: Communication prohibited by filter
Vr HL TOS  Len   ID Flg  off TTL Pro  cks      Src      Dst
 4  5  00 5400 59d4   0 0000  40  01 896f 192.168.11.4  192.168.11.17

Request timeout for icmp_seq 0
92 bytes from 192.168.11.17: Communication prohibited by filter
Vr HL TOS  Len   ID Flg  off TTL Pro  cks      Src      Dst
 4  5  00 5400 9edf   0 0000  40  01 4464 192.168.11.4  192.168.11.17

Request timeout for icmp_seq 1
92 bytes from 192.168.11.17: Communication prohibited by filter
Vr HL TOS  Len   ID Flg  off TTL Pro  cks      Src      Dst
 4  5  00 5400 40eb   0 0000  40  01 a258 192.168.11.4  192.168.11.17

^C
--- 192.168.11.17 ping statistics ---
3 packets transmitted, 0 packets received, 100.0% packet loss
```
100.0% packet loss となっているので拒否できているようです。

icmp-block-inversionをnoに戻しておきます。
```
[root@localhost unamu]# firewall-cmd --remove-icmp-block-inversion
success
```

応答自体を返さない場合はtargetをDROPにすることで可能です、
targetの設定を反映するにはreloadが必要なようです。
```
[root@localhost unamu]# firewall-cmd --set-target=DROP --permanent
success
[root@localhost unamu]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: cockpit dhcpv6-client http ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[root@localhost unamu]# firewall-cmd --reload
success
[root@localhost unamu]# firewall-cmd --list-all
public (active)
  target: DROP
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: cockpit dhcpv6-client http ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

pingで応答が無いか確認
```
~/D/g/D/laradock ❯❯❯ ping 192.168.11.17                                            (git)-[master]
PING 192.168.11.17 (192.168.11.17): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3
^C
--- 192.168.11.17 ping statistics ---
5 packets transmitted, 0 packets received, 100.0% packet loss
```
AWSのサービスにpingを実行した時と同じ反応です、
これが応答が無い場合のリアクションなんですね。

元に戻しておきます。
```
[root@localhost unamu]# firewall-cmd --set-target=default --permanent
success
[root@localhost unamu]# firewall-cmd --reload
success
```

## 参考サイト
https://milestone-of-se.nesuke.com/sv-basic/linux-basic/firewall-cmd/
https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/security_guide/sec-managing_icmp_requests
https://milestone-of-se.nesuke.com/sv-basic/linux-basic/drop-ping-on-linux/




