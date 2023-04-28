---
title: CQRSとマテリアライズドビュー
date: 2020-04-07 23:55:41
tags:
---

検索の一覧ページでパフォーマンスが問題になることがありますが、
マテリアライズドビューを利用することでパフォーマンス改善ができます。  
mysqlではマテリアライズドビューの機能が無いので、
`マテリアライズドビュー`のようなSQL結果を保存するテーブルを作成し、`CQRS`で更新をおこない利用することでパフォーマンスを改善することができます。

（例）SNSでユーザーに多くの投稿があって最新の投稿内容について検索内容が一致するユーザーを一覧で表示する場合に、  
マテリアライズドビューをユーザーと最新の投稿の中間テーブルとすると
下記のような最新の投稿を特定するクエリが必要なくなることでパフォーマンス改善になる。  
```
left join posts as p2 on posts.created_at < p2.created_at where p2.id is null
```

CQRS マテリアライズドビュー 更新フロー図
![CQRSとマテリアライズドビューの更新フロー図](https://cdn.slideship.com/users/J5fayUq9zL8BBesWpreNDm/img/2018/04/8tDAoBRRmWD9QDHzt73qts.png)

### 参考url
https://slideship.com/users/@miyake/presentations/2018/04/5JgW9hp3kHDDZi6zzDRxbm/?p=8
http://nippondanji.blogspot.com/2015/06/rdb.html
https://blog.spacemarket.com/code/room-search-speed/
https://docs.microsoft.com/ja-jp/azure/architecture/patterns/materialized-view
http://qcontokyo.com/data_2016/pdf/B-2_2_JunichiKato.pdf
<img src="{% post_path CQRSとマテリアライズドビュー %}/cqrs-img.png" />
https://docs.microsoft.com/ja-jp/azure/architecture/patterns/materialized-view

# Laravel で参考のコードを書いてみました。
https://github.com/unamu1229/sampleapp/commit/948a6a6809ecf421629ddf18ca54bef44195e94c  

## 処理の流れ

CommandSelectionエンティティ（コマンド用エンティティ）が CommandSelectionRepositoryリポジトリ（コマンド用リポジトリ）で永続化される  
↓  
CommandSelectionRepositoryリポジトリがイベントを発生させる（新規作成時：PushCommandSelection 更新時：PutCommandSelection）  
※ドメインイベントではないので、リポジトリでイベントを発生させてよいのではないかと思いました。  
識別子からクエリ用のエンティティを作成する時も永続化されてからでなければ、最新の状態をとってこれないこともありますし。    
↓  
リスナーでイベントを受ける（イベントPushCommandSelectionの場合、PushQuerySelection。イベントPutCommandSelectionの場合、PutQuerySelection。）  
↓  
リスナーがQuerySelectionエンティティ（クエリ用のエンティティ）を作成し、QuerySelectionRepositoryリポジトリ（クエリ用リポジトリ）でマテリアライズドビュー的な永続化を行う。
