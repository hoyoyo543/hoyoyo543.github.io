---
title: >-
  Error: reading ELBv2 Load Balancer 〇〇〇 ValidationError: 〇〇〇 is not a valid
  load balancer ARN status code: 400, request id: 〇〇〇
date: 2023-05-11 07:24:39
tags:
---

terraform applyを行うと、このようなエラーがでました。  
```
Error: reading ELBv2 Load Balancer (arn:aws:elasticloadbalancing:〇〇〇): ValidationError: 'arn:aws:elasticloadbalancing:〇〇〇' is not a valid load balancer ARN status code: 400, request id: 〇〇〇
```
原因は、私がterraformのデプロイ先のawsアカウントを、以前のものとは別の新しいawsアカウントに新たにデプロイを作成しようとしている為でした。  
terraformが以前のアカウントで作成したelbを確認しようとしているけど、新しいawsアカウントにそのリソースが無いので、status code 400でエラーになっているのだと思います。  
ですので、ローカルのterraform.tfstateを削除して、以前の構築した状態を削除すると新しいawsアカウントにデプロイができるようになりました。
