---
title: >-
  Error: creating ELBv2 Listener 〇〇〇: UnsupportedCertificate: The certificate
  〇〇〇 must have a fully-qualified domain name, a supported signature, and a
  supported key size.
date: 2023-05-11 08:09:40
tags:
---

terraform でACMをALBのリスナーに設定する下記のようなコードの一部があり、
```
〜〜〜

resource "aws_lb_listener" "https" {
    load_balancer_arn = aws_lb.alb.arn
    port = "443"
    protocol = "HTTPS"
    certificate_arn = aws_acm_certificate.aws-qualification.arn
    ssl_policy = "ELBSecurityPolicy-2016-08"
    default_action {
        type = "forward"
        target_group_arn = aws_lb_target_group.ec2.arn
    }
}

〜〜〜
```
terraform applyをすると、下記のようなエラーが表示されました。
```
Error: creating ELBv2 Listener 〇〇〇: UnsupportedCertificate: The certificate
  〇〇〇 must have a fully-qualified domain name, a supported signature, and a
  supported key size.
│ 	status code: 400, request id: 〇〇〇
│
│   with aws_lb_listener.https,
│   on alb.tf line 71, in resource "aws_lb_listener" "https":
│   71: resource "aws_lb_listener" "https" {
```
原因はエラーメッセージの内容とは異なり、作成されたACMの検証が完了できていない為でした。  
ですので、`depends_on`を利用して、先程のコードの一部を下記のように更新してALBのリスナーの作成がACMの検証済みとなることを待つようにしました。

```
〜〜〜

resource "aws_lb_listener" "https" {
    load_balancer_arn = aws_lb.alb.arn
    port = "443"
    protocol = "HTTPS"
    certificate_arn = aws_acm_certificate.aws-qualification.arn
    ssl_policy = "ELBSecurityPolicy-2016-08"
    default_action {
        type = "forward"
        target_group_arn = aws_lb_target_group.ec2.arn
    }
    
    depends_on = [aws_acm_certificate_validation.standby]
}

resource "aws_acm_certificate_validation" "standby" {
    certificate_arn = aws_acm_certificate.aws-qualification.arn
}

〜〜〜
```
そうして、terraform applyをすると、下記のようなメッセージが出力されACMの検証済みとなることを待つようになります。
```
aws_acm_certificate_validation.standby: Creating...
aws_acm_certificate_validation.standby: Still creating... [10s elapsed]
aws_acm_certificate_validation.standby: Still creating... [20s elapsed]
```
待っている間に、作成されたACMの`ドメイン、タイプ、CNAME名、CNAME値`の内容で、Route53の対象のホストゾーンにレコードを作成する。  
そうすると、検証が成功し、terraform applyの処理が最後まで進む。
