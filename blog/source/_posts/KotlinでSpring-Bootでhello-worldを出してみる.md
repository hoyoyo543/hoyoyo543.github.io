---
title: KotlinでSpring Bootでhello worldを出してみる
date: 2021-05-16 10:33:16
tags:
---

# KotlinのSpring Bootでhello worldを出してみました。

## 前提条件
- JDKがインストール済み
- Intellij IDEAもインストール済み

## Spring Bootのフレームワークをダウンロード

[Spring Initializr](https://start.spring.io/)のから設定してダウンロード

### 設定の参考url
https://qiita.com/kawasaki_dev/items/1a188878eb6928880256#%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E9%9B%9B%E5%BD%A2%E3%82%92%E4%BD%9C%E6%88%90

## GradleをインストールしてSpring Bootを起動してみる
Intellij IDEAからダウンロードしたSpring BootのフレームワークをOpenする。

そうすると右下にGradleをインストールしますかのようなポップアップが現れるので、そこからインストールをしてもらう。

GradleがインストールされるとProjectパネルに.gradelとbuildのディレクトリが作成され、main関数の横にrunボタンが表示されるのでrunボタンから起動してWEBアプリケーションを実行します。  
{% asset_img kotlin-spring-boot.png "KotlinでSpring Bootでhello worldを出してみる" %}

### Spring BootはWebサーバーが内包されている
PHPですとnginxやapacheなどのWebサーバを用意する必要がありますが、Spring Bootのフレームワーク自体にWebサーバが内包されているのでmain関数の横にrunボタンを押すだけでwebアプリケーションを起動できます、便利ですね。  
ちなみに、AWSのLambdaのようなものをサーバレスといいますが、このような「WEBサーバ」の用意が不要なこともサーバレスというようです。

## hello worldを表示するコード
https://github.com/unamu1229/spring-boot-demo/pull/2/files
のファイルを追加します。  
再度main関数をrunすると http://localhost:8080/ にhello worldと表示されました。


## 参考url
### Spring Initializr
https://qiita.com/kawasaki_dev/items/1a188878eb6928880256#%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E9%9B%9B%E5%BD%A2%E3%82%92%E4%BD%9C%E6%88%90
### サーバレス
https://qiita.com/kannkyo/items/c3d25553fc505150bfdf#31-%E3%82%B5%E3%83%BC%E3%83%90%E3%83%AC%E3%82%B9%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3
### Controller
https://spring.pleiades.io/guides/tutorials/spring-boot-kotlin/

https://blog.qbist.co.jp/?p=2654