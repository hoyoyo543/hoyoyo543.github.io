---
title: python moduleのimport fromのルートとなる起点を指定したい
date: 2020-06-21 16:51:06
tags:
---

# 実行スクリプトのディレクトリ階層に関係なくfrom importをさせたい
pythonを実行するスクリプトのディレクトリ階層によってfrom importが失敗することがあります。  
実行するスクリプトのディレクトリ階層の位置に関係なくfrom importをできるようにする方法を調べました。

## 例
このようなディレクトリ構成で普段は、python main.pyからitem_service.py item.py が呼び出され実行される内容だとします。
```
├── main.py
├── models
│   └── item.py
└──services
    └── item_service.py
```
下記のようにitem_service.pyはmain.pyをルートとしてmodels/item.pyをfrom importしています。
```
item_service.py

from models.item import Item

class ItemServiceModel:
    def search():
        ...

if __name__ == '__main__':
    ItemServiceModel.search()
```

この場合に python item_service.py をすると  
ModuleNotFoundError: No module named 'models'
のようなエラーが出てimportに失敗します。  

## なぜ失敗するか
https://docs.python.org/ja/3/tutorial/modules.html  
html#the-module-search-path
公式ドキュメントによると、「入力されたスクリプトのあるディレクトリ」がモジュールの探索パスになるということです。  
python main.pyでは成功し、python item_service.pyでは失敗するようになることの理解はできました。

## 入力されたスクリプトのあるディレクトリ構成位置に関係なくfrom importさせたい
先程の公式ドキュメントの同じところに
「PYTHONPATH (ディレクトリ名のリスト。シェル変数の PATH と同じ構文)。」がモジュールの探索パス（import from の起点ルート）になると書いています。
なので、.zshrc や .bshrc に
 ```
 export PYTHONPATH=$PYTHONPATH:{main.pyのあるディレクトリの絶対パス}
 ```
 とすれば、python item_service.pyでもfrom importが成功するようになりました。