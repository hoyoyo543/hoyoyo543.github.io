---
title: “terraform”を開くとコンピュータが破損します。 ゴミ箱に入れる必要があります。
date: 2023-05-11 07:06:31
tags:
---

macでterraformを実行しようとすると、急にこの警告が出てくるようになりました。

{% asset_img "terraform-maruware.png" "“terraform”を開くとコンピュータが破損します。 ゴミ箱に入れる必要があります。" %}
terraformを再インストールすることで警告が出なくなりました。  
tfenvでterraformをインストールしていたので、
tfenv listでバージョンを確認して、1.2.3だったので
```
tfenv uninstall 1.2.3

tfenv install 1.2.3
```
で再インストールすることでエラーが表示されなくなりました。
