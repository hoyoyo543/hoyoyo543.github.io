---
title: python リスト内の＊について
date: 2020-06-16 08:10:53
tags:
---

```
def test(x):
    return x * 2


a = [[3,2,1],[5,4,6],[5,4,6]]
a = [*map(lambda x: test(x), a)]
print(a)

# print出力
[[3, 2, 1, 3, 2, 1], [5, 4, 6, 5, 4, 6], [5, 4, 6, 5, 4, 6]]
```

このようなコードがあって、`[*map(`の部分が何をしているのか分からなかったので調べました。

## 結論
map()の返り値のリストを引数展開して、リストを定義している。

## 詳細
リストの前に*をつけると引数展開される
```
print(*[1, 2, 3])
# print出力
1 2 3
```

## 参考url
http://pixelbeat.jp/variable_length_arguments_and_argument_unpacking/


## 他の [ ]系 の変則的な書き方
### Numpy Boolean Index

Numpy arrayのIndexにBooleanのリストを渡すと、リストのTrueの位置と一致する値だけにできる。
```
import numpy as np
test = np.array([1000, 2000, None])
print(test[[False, True, True]])

# print出力
[2000 None]
```
### 参考url
https://qiita.com/junjis0203/items/4136f9ae4f07c452ceb6#boolean-index%E3%81%A8%E5%80%A4%E8%A8%AD%E5%AE%9A

