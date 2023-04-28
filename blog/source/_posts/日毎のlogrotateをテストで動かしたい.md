---
title: 日毎のlogrotateをテストで動かしたい
date: 2021-03-27 18:18:32
tags:
---

# 日毎のlogrotateをテストで動かしたい

AmazonLinuxで日毎の設定のlogrotateの動作確認をしたいと思い。  
対象のファイルの日付をtouchで昨日にして、  
/usr/sbin/logrotate /etc/logrotate.conf を実行してもログファイルが作成されませんでした。

調べてみると  
/var/lib/logrotate/logrotate.status  
というファイルに前回のログローテートを行った時刻を記載しているとのこと。

/var/lib/logrotate/logrotate.status に先程logrotateコマンドをした日時が記録されていたので、  
その日時を昨日に変更して、再度 /usr/sbin/logrotate /etc/logrotate.conf を実行すると

無事ログファイルがローテートされました。

## tips

調べている際に、logrotateは/etc/cron.dailyで実行されていて。  
cron.dailyの実行は/etc/anacrontabで設定されているということを知りました。

## 参考サイト
https://www.khstasaba.com/?p=958
https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/deployment_guide/ch-automating_system_tasks#s2-configuring-anacron-jobs