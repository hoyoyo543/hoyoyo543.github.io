---
title: Hexoで画像を表示する
date: 2021-05-16 12:20:37
tags:
---

_config.ymlをpost_asset_folder: trueにする。

hexo new {記事title}の時に source/_postsディレクトリ以下に{記事tile}のディレクトリができるので、
そのディレクトリ内に画像を配置。
```
{% asset_img 画像名 記事名 %}
```
のタグで表示できます。

画像名や記事名にスペースがふくまれる場合、ダブルクォーテーションで囲ってください。
```
{% asset_img "画像名" "記事名" %}
```

## 参考url
https://hexo.io/docs/asset-folders#Tag-Plugins-For-Relative-Path-Referencing