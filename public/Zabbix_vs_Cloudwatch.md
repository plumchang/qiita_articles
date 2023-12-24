---
title: Zabbix vs. Cloudwatch
tags:
  - AWS
  - EC2
  - zabbix
  - CloudWatch
  - ECS
private: false
updated_at: '2019-06-07T16:59:39+09:00'
id: 72d892fe7ac4a8290964
organization_url_name: fujitsu
slide: false
ignorePublish: false
---
#はじめに
AWSのサービスを監視する方法として、「Zabbix」か「Cloudwatch」かを検討する機会があったため、比較メモとしてこの記事を投稿します。
「Zabbix」で監視可能なサービス（EC2、ECS）をメインに比較してみました。
#全体
||Zabbix|Cloudwatch|
|:---|:---|:--|
|形態|OSS|サービス|
|料金|タダ|課金あり|
|監視可能なサービス|EC2, ECS|色々※|
|ダッシュボード|〇|〇|
|マネージャー管理<br>手動スケーリングなど|必要|不要|
※[Cloudwatchで監視できるサービス](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html)

#EC2
||Zabbix|Cloudwatch(※エージェントなし)|Cloudwatch(※エージェント利用)|
|:---|:---|:---|:--|
|メトリクスの種類|多※1|少|多※2|
|ログ監視|任意のファイル|×|任意のファイル|
|トリガー・アラーム|〇|△|△|

※1 [Zabbixのメトリクス](https://www.zabbix.com/documentation/4.0/manual/config/items/itemtypes/zabbix_agent)
※2 [EC2 Cloudwatchのメトリクス](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html)

ログ監視について補足です。

ファイル名指定は、ZabbixはWebコンソールからファイル名指定で監視できますが、CloudwatchではEC2インスタンス側で出力するログファイルを指定します。
また、文字列パターン等を設定してのアラーム発行はZabbix,Cloudwatchともに可能ですが、Cloudwatchでは対象のログが表示されないというデメリットがあります。

#ECS(Fargate)
||Zabbix|Cloudwatch|
|:---|:---|:--|
|メトリクスの種類|多|少※|
|ログ監視|任意のファイル|標準出力、標準エラーのみ|
|サービス監視|×|〇|
|タスク監視|〇|×|
|コンテナ監視|×|×|
|トリガー・アラーム|〇|〇|

※[ECS Cloudwatchのメトリクス](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/cloudwatch-metrics.html)

コンテナ監視をしたい場合、
EC2起動タイプのタスクであれば、[cAdvisor](https://github.com/google/cadvisor)を利用できます。
Fargateでのコンテナ監視はおそらく不可能…

#まとめ
EC2、ECSでログメインの監視をするのであれば、Webコンソールだけで操作が完結するZabbixを使うと良いでしょう。
Cloudwatchでは、障害時に原因のログが表示されないので使い勝手の点で劣ると思います。

それ以外のサービスも合わせて監視したいのであれば、Cloudwatchを選ぶことになります。
Zabbixには、任意スクリプトのリターン値をメトリクスとして取得できる機能があるので
AWS APIを駆使すれば、EC2、ECS以外のサービスも含めてZabbixで監視も可能です。
