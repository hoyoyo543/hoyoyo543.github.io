---
title: AWS EKS にてno endpoints available for service kubernetes-dashboardのエラー対応
date: 2020-07-19 16:35:33
tags:
---

# エラーの出た場面

AWSのチュートリアル: Kubernetes ダッシュボード (ウェブ UI) のデプロイをやっている時に、  
「ウェブブラウザで以下のリンクを開いて、ダッシュボードエンドポイントにアクセスします。」という段階で下記urlにアクセスすると  
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login  

下記のエラーが画面に表示されダッシュボードエンドポイントにアクセスできなかった。
```
{
    kind: "Status",
    apiVersion: "v1",
    metadata: { },
    status: "Failure",
    message: "no endpoints available for service "kubernetes-dashboard"",
    reason: "ServiceUnavailable",
    code: 503
}
```

## 対応方法
下記のコマンドを打って、Namespace kubernetes-dashboard を対象とするFargateプロファイルを追加で作成する。
```
eksctl create fargateprofile \
    --name fp-kubernetes-dashboard \
    --namespace kubernetes-dashboard \
    --cluster {クラスター名}
```

## エラーの理由
https://dev.classmethod.jp/articles/fargate-for-eks-tutorial-kubernetes-dashboard/#toc-4  
の「「Fargate for EKS」の場合に必要な追加手順」項目を参照