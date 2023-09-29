---
title: phpのissetとemptyとis_nullの違い
date: 2023-05-11 10:55:45
tags:
---

issetは返り値に対してチェックを行うとエラーになる。
```
これは、Fatal errorになる。
isset(bar());
```
emptyとis_nullは返り値に対してチェックができる。
```
これは、OK.
empty(bar());
is_null(bar());
```

なので、!empty(bar())とかがあって、isset(bar())のほうが否定（反転）がとれてリーダブルコードじゃんとか思って変更するとエラーになるw
