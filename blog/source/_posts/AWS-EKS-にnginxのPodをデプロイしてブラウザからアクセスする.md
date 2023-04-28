---
title: AWS EKS にnginxのPodをデプロイしてブラウザからアクセスする
date: 2020-07-26 14:18:47
tags:
---

Docker/Kubernetes実践コンテナ開発入門の本を読んでいて5.10.1のIngressを通じたアクセスをAmazon EKSで行う場合はどのようになるのか調べました。

## 利用しているdocker image
gihyodocker/echo : Hollo Docker!!と表示するwebサーバー
gihyodocker/nginx-proxy : echoに対してリバースプロキシする


## 手順

- 1 eksctlでcrusterを作成する
https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started-eksctl.html
※Fargetポッドがデプロイされる

- 2 ALB Ingress Controllerを作成する
https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/alb-ingress.html

- 3 ReplicaSet Service Ingress を作成する
ブラウザでIngressのアドレスにアクセスする

### ReplicaSet
同じ仕様のPodを複数作成する
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: echo-summer
  labels:
    app: echo
    release: summer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: echo
      release: summer
  template:
    metadata:
      labels:
        app: echo
        release: summer
    spec:
      containers:
      - name: nginx
        image: gihyodocker/nginx:latest
        env:
        - name: BACKEND_HOST
          value: localhost:8080
        ports:
        - containerPort: 80
      - name: echo
        image: gihyodocker/echo:latest
        ports:
        - containerPort: 8080
```

### Service
Podの集合に対して、アクセス経路を作成する。typeをNodePortにすることでグローバルなポートを開ける。OSI参照モデルのレイヤー４までしか扱えない。

```
apiVersion: v1
kind: Service
metadata:
  name: "echo"
spec:
  type: NodePort
  selector:
    app: echo
    release: summer
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```

### Ingress
ServiceのKudernetesクラスタ外への公開。  
VirtualHostやパスベースでのHTTP、HTTPSベースでのルーティングが可能。  
Ingressが作成されると、EKSのALB Ingress ControllerがALBと必要なAWSサポートリソースを作成してくれる。

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: "echo"
  annotations:
    kubernetes.io/ingress.class: alb # Ingress が ALB Ingress Controllerを使用することを設定
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip # Serviceでアクセス経路が作成されたPodにトラフィックをルーティングすることを設定
  labels:
    app: echo
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: "echo"
              servicePort: 80
```