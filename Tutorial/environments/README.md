# 環境

以下の方法を使って、API Platform Cloud Service環境を入手できます。Apiary Cloud Serviceには、利用可能なマルチテナントサービスが1つあります。ほとんどの機能を http://apiary.io の無料アカウントで実行できます。これらの環境は、現在Oracle Public Cloud上の単一のテナント・サービスであるAPI Platform Cloud Serviceにアクセスするためのものです。 API Platform Autonomousが利用可能になると、これらの指示の一部が変更される可能性があります。

## API Platform Cloud Serviceのロールに関する注意

API Platform Cloud Serviceにはアクションを実行するため、以下のような複数のロールがあります。

- API Manager：APIを作成および管理するユーザー
- Service Manager：サービスとサービスアカウントを作成および管理するユーザー
- Gateway Manager：ゲートウェイを作成および管理するユーザー
- Gateway Runtime User：ゲートウェイ・ノードによって使用されるアカウント
- Plan Manager：プランを作成および管理するユーザー
- アプリケーション開発者：APIを使用してアプリケーションを作成するユーザー
- 管理者：API Platform Cloud Serviceの全体管理を担当するユーザー

API Platform Cloud Serviceにアクセスするためのオプションを選択します。一部のインスタンスではユーザーが事前に作成される場合と、プロビジョニングの一部としてユーザーを作成する必要がある場合があることに注意してください。また、チュートリアルでは、上記ロールを簡単に表現することがあります。具体的には、「*APIマネージャ*としてAPI Platformにログインする」という感じです。そのため、このタスクの完了に必要なユーザーを知っておく必要があります。

Apiaryの場合、サービスにサインアップするときに作成するユーザー以外を考える必要はありません。

オプションを選択して環境にアクセスしたら、簡単に参照できるようにユーザーの資格情報をドキュメントに記載したほうがいいかもしれません。

## 使用する環境の選択

講師つきのワークショップに参加している場合は、利用環境を講師が指示します。

## GSE Demo Central

demo.oracle.comの下であなたにプロビジョニングされる環境。これは短期間利用可能な環境で、ゲートウェイを構成する必要がありますが、管理の自由度が高いため、API Platform Cloud Serviceをさらに深く理解することができます。

詳細は、[GSE Demo Central](./gseDemo.md)をご覧ください。

> 注：リンクは常に新しいタブで開いてください（右クリック→*新しいタブでリンクを開く*）。そうすれば、タスクを完了後にリンク先のチュートリアルなどからこの演習ガイドに戻ってくることができます。

## 内部環境オプション（Oracle従業員のみ）

Oracle従業員には、内部的に使用可能な環境オプションがいくつかあります（VPNでアクセスする必要があります）。

内部環境オプションについては、[内部環境](https://stbeehive.oracle.com/teamcollab/wiki/API+PLATFORM:Internal+Environments)を参照してください。
