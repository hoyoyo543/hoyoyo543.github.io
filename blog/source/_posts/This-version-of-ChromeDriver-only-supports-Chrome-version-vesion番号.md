---
title: 'This version of ChromeDriver only supports Chrome version {vesion番号}'
date: 2020-05-04 17:08:26
tags:
---
# エラー This version of ChromeDriver only supports Chrome version 〇〇
SeleniumのWebdriverでChromeを動かしていると`This version of ChromeDriver only supports Chrome version 81`とエラーメッセージが表示されました。  
chromeのバージョンが自動アップデートされて、webdriverとchromeのバージョンが合わなくなったので表示されたようです。  
ですので普段利用するchromeとは別に、seleniumで利用するchromeをインストールしてchromeの自動アップデートを無効にする方法を調べたのですが、個別にアップデートを抑制する方法が見つからず。  

## ChromiunだとChromeと同じ機能で自動アップデートされない
そんな時に、`Chromium`だとChromeとほぼ同じ機能で自動アップデートされないという情報を見つけたので、
Webdriverのversionに対応する、Chromiumをダウンロードして解決しました。  

## Chromiumの過去バージョンのダウンロードurl
https://chromium.woolyss.com/  
