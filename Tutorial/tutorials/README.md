# API Platformチュートリアル

API Platformのチュートリアルへようこそ。ここでは、API Platform Cloud Serviceについて学び、デモする機会を提供するチュートリアルデザインのコレクションを作成しています。

## 環境

これらのチュートリアルは、一般にどのような環境でも利用できるように設計されています。ただし、使用している環境の管理者でない場合は、すべてのチュートリアルを実行するために必要な権限がない場合があります。

異なる環境の詳細と注意点については、[環境](../environments/README.md)を参照してください。

## チュートリアルのカタログ

### APIデザイン

チュートリアル|前提条件のチュートリアル
---  | ---
[APIの設計](./design/design_api)| なし

### API管理

チュートリアル|前提条件のチュートリアル
--- | ---
[APIポリシー実装の作成](./manage/apis/create_api)| [APIの設計](./design/design_api)
[APIのデプロイ](./manage/apis/deploy_api)| [APIポリシー実装の作成](./manage/apis/create_api)
[APIの呼び出し](./manage/apis/invoke_api)| [APIのデプロイ](./manage/apis/deploy_api)
[プランへのAPIの資格付与](./manage/apis/entitle_api) | [APIポリシー実装の作成](./manage/apis/create_api)、[APIプランの作成](./manage/plans/create_plan)
[プランの作成](./manage/plans/create_plan)| なし
[サービスの作成](./manage/services/create_service)| なし
[メソッド・マッピングポリシー](./manage/apis/policies/method_mapping)| [APIポリシー実装の作成](./manage/apis/create_api)、[サービスの作成](./manage/services/create_service)

### ゲートウェイ管理

チュートリアル|前提条件のチュートリアル
--- | ---
[ゲートウェイ上のアクセスGrantの管理](./manage/gateways/grants)| なし
