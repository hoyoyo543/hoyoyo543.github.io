---
title: 'systemd Failed to execute command: Permission denied'
date: 2021-03-20 13:33:38
tags:
---

CentOS8にてsystemdのtest.serviceを作成してみてsystemd start testを行ってみると実行に失敗、
/var/log/messagesに下記のエラーが表示されていました。
```
Mar 20 13:21:45 unamu systemd[1]: Started testscript.
Mar 20 13:21:45 unamu systemd[83619]: test.service: Failed to execute command: Permission denied
Mar 20 13:21:45 unamu systemd[83619]: test.service: Failed at step EXEC spawning /tmp/test: Permission denied
```
Permission deniedの表示からファイルのパーミッションを確認しましたが、問題無し。
SELinuxが邪魔をしていたようです。
有効になっているか確認
```
$ getenforce
Enforcing
```
Permissiveモードにします。
```
$ setenforce 0
```
その後、systemd start testで無事実行されました。

## 参考url
https://fujiyasu.hatenablog.com/entry/2016/08/05/094804