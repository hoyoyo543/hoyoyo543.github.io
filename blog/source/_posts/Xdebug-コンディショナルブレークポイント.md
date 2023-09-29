---
title: Xdebug コンディショナルブレークポイント
date: 2023-08-01 07:07:55
tags:
---

コンディショナルブレークポイント(Conditional Breakpoint)の機能を使えば、特定の条件の時のみ、ブレークポイントで止めることができる。  

PHPStormの場合、ブレークポイントを設定してそれを右クリックするとConditionの入力エリアが出てくるので、そこにブレークポイントを止めたい場合の条件を設定する。  

(例) $valueがbarの時のみブレークポイントで止める。  
{% asset_img "Conditional_Breakpoint.png" "Xdebug コンディショナルブレークポイント" %}
