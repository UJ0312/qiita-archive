---
title: CloudWatch Logsのリソースポリシーのサイズ制限(文字数5120文字以内)に抵触した場合の対処方法
tags:
  - AWS
  - CloudWatch
  - ElastiCache
  - CloudFormation
  - インフラエンジニア
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## 事象

新規作成したElastiCacheで、エンジンログとスローログをCloudWatch Logsへ出力するためにCloudFormationを更新したところ、スタック更新が失敗しました。

この記事では、その原因と対処法を備忘録としてまとめます。


## エラー内容

CloudFormationのイベントログには、以下のエラーが表示されていました。

```text
Failed to enable log delivery for log type engine-log. Error: Request could not be completed due to internal error.
Failed to enable log delivery for log type slow-log. Error: Request could not be completed due to internal error.
```

## 原因 〜ポリシーの文字数制限だった〜

CloudWatch Logsのリソースポリシーには、5120文字の制限があります。

今回のケースでは既にリソースポリシー（Sid: AWSLogDeliveryWrite20150319）の文字数が**5211文字** に達しており、上限を超過していました。

そのため、ログ出力設定時にリソースポリシーへロググループを自動追加できず、結果としてログ出力およびCloudFormationのスタック更新が失敗していました。

https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/sfn-best-practices.html#bp-cwl

なお、CloudWatch logのリソースポリシーはAWSコンソールのIAMポリシーには表示されません。
AWS CloudShellなどでAWS CLIを使用することで、確認することができます。

```bash
aws logs describe-resource-policies
```


## 対処法

リソースポリシー（Sid: AWSLogDeliveryWrite20150319）の文字数を削減するため、
Resourceブロックに定義されているロググループの指定をワイルドカード（*）でまとめます。

**修正前**
```text
"arn:aws:logs:ap-northeast-1:アカウントID:log-group:/aws/elasticache/xxxxx/enginelog:log-stream:*",
"arn:aws:logs:ap-northeast-1:アカウントID:log-group:/aws/elasticache/yyyyy/slowlog:log-stream:*",
"arn:aws:logs:ap-northeast-1:アカウントID:log-group:/aws/elasticache/zzzzz/enginelog:log-stream:*",
...（省略）
```

**修正後**
```text
"arn:aws:logs:ap-northeast-1:アカウントID:log-group:*:*"
```

ワイルドカードを使用することで、ポリシーの記述を大幅に削減し、文字数制限内に収めることができます。


## 修正手順

CloudShellを使って既存のリソースポリシーを上書きします。

1. CloudShellを開き、ポリシーファイルを作成します

```bash
nano resource-policy.json
```

2. 以下の内容で保存します

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AWSLogDeliveryWrite",
            "Effect": "Allow",
            "Principal": {
                "Service": "delivery.logs.amazonaws.com"
            },
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:logs:ap-northeast-1:アカウントID:log-group:*:*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:SourceAccount": "アカウントID"
                },
                "ArnLike": {
                    "aws:SourceArn": "arn:aws:logs:ap-northeast-1:アカウントID:*"
                }
            }
        }
    ]
}
```

3. 以下のコマンドで既存のリソースポリシーを上書きします

```bash
aws logs put-resource-policy \
  --policy-name AWSLogDeliveryWrite20150319 \
  --policy-document file://resource-policy.json
```

この対応後、CloudFormationのスタック更新は正常に完了しました！


## まとめ

ElastiCacheのログ有効化時に、CloudWatch Logsのリソースポリシーの文字数制限が原因でCloudFormationが失敗した事例でした。

リソースポリシーをワイルドカードで整理して5120文字以内に収めることで、無事にCloudWatch Logsへログ出力できるようになりました。

文字数制限は、原因が分かれば意外とシンプルに対処できます。同じエラーにはまっている方の参考になれば幸いです。
