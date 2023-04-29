# hoyoyo543.github.io

## 記事の作成
hexoが入っているnodeバージョンに変更
```
nodebrew use v12.13.1
```

記事を書くファイルを作成
```
cd blog
hexo new "〇〇〇"
```
作成されたファイルに記事を書く

githubにプッシュ
```
hexo deploy -g
```

最後に元に戻す
```
nodebrew use v19.4.0
```