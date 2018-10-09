# APIの設計

Oracle API Platform Cloud Serviceの一部であるOracle Apiaryを使用すると、API BlueprintまたはSwagger/OpenAPIのいずれかを使用してAPIを設計できます。この演習では、設計プロセスについて説明します。serviceTickets APIを手早く作成するため、事前作成済みのAPI Blueprintを利用します。

## 始める前に

- Apiaryのアカウントを持っている必要があります。`http://apiary.io`で無料の開発者アカウントにサインアップできます。

## APIデザインの作成

### Apiaryへのログイン

ブラウザで`http://apiary.io`にアクセスしてサインインします。登録時にサンプルAPIが表示される場合があります。

### 新規APIプロジェクトの作成

左上のプロジェクト名をクリックし、*Create New API Project*を選択します。

### 新規APIプロジェクトの命名

デモのシナリオに沿ってAPIを*ticketService*と命名しますが、実際は好きな名前を自由に選ぶことができます。このガイドでは*ticketService*をプロジェクト名として使いますので、自分が設定した名称をを覚えておいてください！

1. デフォルト値*Personal API*のままでも良いです。
2. あなたがApiaryのチームアカウントのメンバーである場合、APIを非公開にするオプションがあります。
3. API BlueprintまたはSwaggerを選択できます。この例では、API Blueprintを使用しています。

テンプレートとして作成された*Polls API*を使用してAPIが作成されていることがわかります。これは、作成の手引きとして提供されているものです。

  > リファレンスとして、[ticketService API Blueprint](./ticketService.apib)を使用してください。手作業ですべてを入力する必要はありません。エディタで開き、コピーしてApiaryエディタの*Polls*サンプルに貼り付けることが可能です。

この演習では、API Blueprintの各ステップについて説明しませんが、少し時間を使って左側のAPI Blueprintと右側の自動的に生成されたドキュメントを比較してみましょう。Blueprint の一部を変更して、右側に反映された変更を確認することができます。

*Save*をクリックしてください。

### もっと詳しく知る

Apiaryデザインエディタは、APIの設計、共同作業、契約（Contract、つまりインターフェース）の理解のための豊富な機能を提供しています。詳細は以下のリソースをご覧ください。

- [Apiary Editor](https://help.apiary.io/tools/apiary-editor/)
- [インタラクティブ・ドキュメント](https://help.apiary.io/tools/interactive-documentation/)

## モックサーバーを使用したデザインのプロトタイプ作成

Apiaryでデザインを作成すると、自動的にインタラクティブなドキュメントが作成されますが、モックサーバーも作成されます。モックサーバーは、使用可能な動作するエンドポイントを提供するので、デザインのプロトタイピングに役立ちます。

  > 注意：デザインを保存したことを確認してください。

*Save*をクリックするたびに、Apiaryはデザインに基づいてモックサーバーを生成します。インタラクティブなドキュメントを使用して、デザインをテストすることができます。

### Apiaryコンソールでのテスト

1. インタラクティブなドキュメントの*Tickets Collection*で、*List All Tickets*をクリックします。
2. *Mock Server*が選択されていることを確認し、言語を選択してください。

    > さまざまな言語に切り替えて、各言語でのAPI呼び出し例を見ることができます。

3. チェック終了後、*Try*ボタンをクリックしてください。
4. パラメータやヘッダーなどを設定するオプションが表示されますが、今回は何も設定せず、*Mock Server*が選択されていることを確認後、*Call Resource*ボタンをクリックしてください。
5. リソースの呼び出し後、下にスクロールしてモックサーバーからの応答を確認します。

### おまけ

お気に入りのRESTクライアントを使用して、*Example*と*Console*ウィンドウに表示されるモックサーバーのエンドポイントを呼び出します。

### もっと詳しく知る

Apiary Mock Serverは、チームがAPIデザインをテストできる強力な機能です。常に正しいAPIを開発していることを確保できます。以下のリソースで詳しく学んでください：

- [モックサーバー](https://help.apiary.io/tools/mock-server/)

## 結論

この演習では、次のことを学びました。

- Oracle ApiaryでAPIを設計する方法
- インタラクティブドキュメントがAPIデザインからどのように生成されるか
- APIデザインから自動的に生成されるモックサーバーの使用方法

### このチュートリアルのビデオウォークスルー

[![Designing an API](http://img.youtube.com/vi/Lotj263xUgs/0.jpg)](http://www.youtube.com/watch?v=Lotj263xUgs "Designing an API")