---
title: Eloquentのモック方法
date: 2025-04-14 09:37:14
tags:
---

MockreyでEloquentをモックする方法

# 動的プロパティへのアクセスをモック
$user->mail_address のような箇所をモックしたい場合、

shouldReceive('getAttribute')->with({モックしたいプロパティ名})とする  
(例)
```
$user->shouldReceive('getAttribute')
       ->with('mail_address')
       ->andReturn('foo@example.com');
```

# issetで動的プロパティを評価している箇所をモック
isset($user->mail_address) の様な箇所をモックしたい場合、
shouldReceive('offsetExists')->with({モックしたい評価されるプロパティ名})とする
(例)
```
$user->shouldReceive('offsetExists')->with('mail_address')->andReturnTrue();
```

# リレーションの箇所をモック
```
public function profile()
{
    return $this->belongsTo(profile::class);
}
```
のようなリレーションがUserモデルに定義されていて、$user->profile のような箇所をモックしたい場合。
動的プロパティと同様に、shouldReceive('getAttribute')->with({モックしたいリレーション名})とする。
```
$user->shouldReceive('getAttribute')
       ->with('profile')
       ->andReturn(new Profile());
```

参考URL  
https://github.com/unamu1229/test_laravel/commit/228069268bec9094c3238090ff2b82deab3d0a20
