---
title: core dumpについて
date: 2020-12-06 18:10:17
tags:
---


php-fpmのログで下記のようなSIGSEGVのエラーがでており、原因を調査する場合にcore dumpという方法があるようなので試してみました。
```
WARNING: [pool www] child 17248 exited on signal 11 (SIGSEGV) 
```

## 検証環境
CentOS8

## core dumpとは
システムが異常終了した場合に、その時点のメモリの内容を記録したcoreファイルを吐き出すことを指すようです。

## coreファイルの出力先を確認

```
$ cat /proc/sys/kernel/core_pattern
core
```
coreとなっている場合、
プロセスの作業ディレクトリに作成されます。  
プロセスの作業ディレクトリは、  
ls -l /proc/{プロセスID}/cwd のリンク先になります。  
プロセスが終了すると /proc/{プロセスID} のディレクトリは無くなるので、  
確認する場合はプロセスが生きている間に確認する必要があります。  
プロセスを生きている間に確認することが難しい場合は、  
/proc/sys/kernel/core_patternに適当なディレクトリを作成することでcoreの出力先を指定できます。 
osを再起動すると元の設定にもどります。
```
echo '/tmp/core-%e-%p' > /proc/sys/kernel/core_pattern
```
%eは、プログラム名。%pはプロセスIDとして置き換わります。

### CentOS8の場合
```
$ cat /proc/sys/kernel/core_pattern
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h %e
```
となっており、パイブでわたされた、/usr/lib/systemd/systemd-coredumpが
/var/lib/systemd/coredumpのディレクトリ直下にコアダンプが出力されるようになっていました。
また、coredumpctlでcoreの情報を見たりできます。

最近作成されたcoreの一覧を表示。
```
$ coredumpctl list
TIME                            PID   UID   GID SIG COREFILE  EXE
Sun 2020-12-06 22:29:22 JST    3252  1000  1000  11 present   /usr/bin/bash
```
coredumpctl info に pidを指定して情報を見る。
```
$ coredumpctl info 3252
           PID: 3252 (segv.sh)
           UID: 1000 (unamu)
           GID: 1000 (unamu)
        Signal: 11 (SEGV)
     Timestamp: Sun 2020-12-06 22:29:21 JST (6min ago)
  Command Line: /bin/bash ./segv.sh
    Executable: /usr/bin/bash
 Control Group: /user.slice/user-1000.slice/user@1000.service/gnome-terminal-server.service
          Unit: user@1000.service
     User Unit: gnome-terminal-server.service
         Slice: user-1000.slice
     Owner UID: 1000 (unamu)
       Boot ID: 35a2b597dc5a4a6ebe37fd3a7e97bd73
    Machine ID: c4f822f021d5433ab143f054bca5b672
      Hostname: localhost.localdomain
       Storage: /var/lib/systemd/coredump/core.segv\x2esh.1000.35a2b597dc5a4a6ebe37fd3a7e97bd73.3252.1607261361000000.lz4
       Message: Process 3252 (segv.sh) of user 1000 dumped core.
                
                Stack trace of thread 3252:
                #0  0x00007fe849946d79 _int_malloc (libc.so.6)
                #1  0x00007fe84994850e malloc (libc.so.6)
                #2  0x00005625a34934c2 xmalloc (bash)

```


## coreファイルのファイルサイズの変更
```
$ cat /proc/self/limits
Limit                     Soft Limit           Hard Limit           Units     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            8388608              unlimited            bytes     
Max core file size        unlimited            unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             3140                 3140                 processes 
Max open files            1024                 262144               files     
Max locked memory         65536                65536                bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       3140                 3140                 signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         0                    0                    
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us  
```
の Max core file size の項目がCoreファイルサイズの設定になります。
softlimitが0だとcoreファイルが作成されません。

unlimitedに設定すると上限なしで作成してくれます。
```
$ ulimit -c unlimited
```
OSを再起動すると元の設定値に戻ります。

## SIGSEGVエラーを出して、coreファイルが作成されている確認する
下記のシェルスクリプトを実行して、SIGSEGVエラーを出す。  

segv.sh 
```
#!/bin/bash
function func {
    func
}
func
```

```
# ./segv.sh 
Segmentation fault (コアダンプ)
# ls /tmp
core-segv.sh-41856 
```
(コアダンプ)と表示されて出力を指定した先にcoreファイルが作成されています。

## coreファイルの中身を確認

gdb 実行プログラミング coreファイル でgdbでデバックできます。
```
# gdb /bin/bash /tmp/core-segv.sh-41856

GNU gdb (GDB) Red Hat Enterprise Linux 8.2-11.el8
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /bin/bash...Reading symbols from .gnu_debugdata for /usr/bin/bash...(no debugging symbols found)...done.
(no debugging symbols found)...done.

warning: core file may not match specified executable file.
[New LWP 41856]

warning: Loadable section ".note.gnu.property" outside of ELF segments
Core was generated by `/bin/bash ./segv.sh'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x000055b592ffb6db in expand_word_list_internal ()
Missing separate debuginfos, use: yum debuginfo-install bash-4.4.19-10.el8.x86_64
(gdb) 
```

gdbの対話モードになったら bt or backtrackでコアダンプされる前の状態をさかのぼって原因を調査できます。
```
(gdb) bt
#0  0x000055b592ffb6db in expand_word_list_internal ()
#1  0x000055b592fd2fa7 in execute_simple_command ()
#2  0x000055b592fd51a6 in execute_command_internal ()
#3  0x000055b592fd4ab5 in execute_command_internal ()
#4  0x000055b592fd7a79 in execute_function.isra ()
#5  0x000055b592fd4124 in execute_simple_command ()
#6  0x000055b592fd51a6 in execute_command_internal ()
#7  0x000055b592fd4ab5 in execute_command_internal ()
#8  0x000055b592fd7a79 in execute_function.isra ()
#9  0x000055b592fd4124 in execute_simple_command ()
```

## 参考url
https://www2.filewo.net/wordpress/2019/12/16/centos%E3%81%A7%E3%81%AF%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88%E3%81%A7%E3%82%B3%E3%82%A2%E3%83%80%E3%83%B3%E3%83%97%E3%81%8C%E5%90%90%E3%81%8B%E3%82%8C%E3%81%AA%E3%81%84%E3%81%AE%E3%81%A7/

https://rabbitfoot141.hatenablog.com/entry/2016/11/14/153101

https://rheb.hatenablog.com/entry/systemd-coredump

https://qiita.com/suzutsuki0220/items/aa84d7e2e8f37e867f3d

https://qiita.com/rarul/items/d33b664c8414f065e65e

### php-fpm
https://www.bit-hive.com/articles/20190206