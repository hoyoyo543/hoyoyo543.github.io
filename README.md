# hoyoyo543.github.io

## ブランチ
master は gihub.ioに公開されるブランチで、hexo deploy -g をした時に更新されるので、ソースコードの管理ブランチはsourceブランチで行う

## 記事の作成

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