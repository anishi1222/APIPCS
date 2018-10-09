# APIでのOAuth 2.0の使用

> このチュートリアルは現在作成中であり、完全ではありません。Early Adapterからのフィードバックをお待ちしております。

API Platform Cloud Serviceのゲートウェイは、OAuth 2.0クライアントとOAuth 2.0リソースサーバーの両方として機能できます。以下のユースケースを考えてみましょう。

1. OAuth 2.0クライアント：サービスバックエンドはOAuth 2.0で保護されており、フロントエンドAPIの要件を満たしていません。これは、バックエンドシステムは内部ユーザーのみが存在し、APIコンシューマに利用できないまたは適切ではない場合があります。
1. OAuth 2.0リソースサーバ：ゲートウェイは、Bearerトークンを要求して検証するOAuth 2.0リソースサーバとして機能します。これにより、APIコンシューマに基づいてAPIレベルでアクセス制御を維持する機能が提供されます。

ゲートウェイは、同じフロー内でOAuth 2.0クライアントとリソースサーバとして機能することも、基本認証を使用するサービスバックエンドのためにモダンな認証方式を提供するためのOAuth 2.0フロントエンドとして機能することもできます。

## 始める前に

API Platform Cloud Serviceの概念は熟知していて、APIの作成、デプロイ、呼び出しのチュートリアルを完了しました。

このチュートリアルでは、OAuth 2.0トークンを強制するAPIを作成/更新します。このような認定のためのAPIの設定には、複数の個人が関わっています。

1. **API Manager**：API実装を作成しました。 OAuth 2.0ポリシーを設定し、スコープの処理方法を選択します。
1. **Gateway Manager**：リソースサーバーとして機能できるようにゲートウェイのOAuth 2.0プロファイルをロードします。ゲートウェイがOAuth 2.0プロバイダでどのように設定されているかは、[OAuth 2.0の設定](../../../../configure/gateway/oauth/README.md)を参照してください。
1. **アイデンティティ管理者**：ゲートウェイまたは必要に応じて特定のAPIで使用するために必要なクライアントアプリケーションを作成します。

### 前提条件

- 利用可能なAPI Platform Cloud Service環境を有している必要があります。API Platform Cloud Service環境の調達の詳細については、[環境](../../../../../environments/README.md)を参照してください。

    > 注：リンクは常に新しいタブで開いてください（右クリック→*新しいタブでリンクを開く*）。そうすれば、タスクを完了後にリンク先のチュートリアルなどからこの演習ガイドに戻ってくることができます。

- Identity Cloud Service のIdentity Consoleへのアクセスとアプリケーションを作成できる権限が必要です。
- OAuth 2.0 *Resource Server*として構成されているゲートウェイがご使用の環境に少なくとも1つ配置されている必要があります。ゲートウェイがOAuth 2.0プロバイダでどのように設定されているかは、[OAuth 2.0の設定](../../../../configure/gateway/oauth/README.md)を参照してください。
- *API Manager*としてAPI Platform Cloud Serviceにアクセスでき、OAuth 2.0ポリシーを追加できるAPIがある必要があります。APIを作成する必要がある場合は、[APIポリシー実装の作成](../../../../manage/apis/create_api/README.md)を参照してAPIを作成してください。
- APIがデプロイ済みで呼び出し可能である必要があります。

## Identity Cloud Service（IDCS）にクライアントを登録する

OAuth 2.0の設定時の考慮点は多数ありますが、*オーディエンス*と*スコープ*を含むトークンがゲートウェイに到着することを覚えておく必要があります。

ゲートウェイでは、（[OAuth 2.0の設定](../../../../configure/gateway/oauth/README.md)で説明されている)1つのプロファイルを、ゲートウェイのロードバランサアドレスとしてオーディエンスにロード済みです。

リソースアプリケーションには4つのスコープ（作成、読み取り、更新、削除）がありますが、スコープには任意の名前を付けることができます。

今回は、クライアントがリソースサーバーとして振る舞うAPI Platform Cloud Serviceゲートウェイを使用してAPIを呼び出す想定のため、このクライアントは、*audience* + *scope* の形式でスコープを持つOAuth 2.0トークンを取得します。

OAuth 2.0リソースサーバー向けに作成したアプリのすべての機能をAPIクライアントから利用できるようにすることは望ましくないため、2つのスコープにしかアクセスできないクライアントをIDCS上に作成します。

### IDCSにログイン

1. *Identity Administrator*権限を持つユーザーを使用してOracle Cloud Domainにログインします。
1. ダッシュボードの右上にある*ユーザー*メニューをクリックします。
1. Identity Console ボタンをクリックします。

### API用のアプリケーションを作成する

1. サイドメニューから*Applications*を選択します。
1. *Add*をクリックします。
1. *Trusted Application*を選択します。
1. アプリケーションに*API Client*のような名前を付けます。この環境を他の人と共有している場合は、アプリケーションの名前を一意にします（例えばあなたの名前を付ける、など）。*Next*をクリックします。
1. *Configure this application as a client now*を選択します。
    1. *Client Credentials*、*Refresh Token*を選択します。
    1. *Trust Scope*の*Allowed scopes*を選択します。
    1. *Allowed Scopes*の下に*Add*をクリックします。
        1. APIGatewayリソースアプリを選択し、Create, Readスコープを選択します。
    1. *Next*をクリックし、残りのすべての画面をそのままにします。
    1. *Finish*をクリックします。

> *Cient ID*と*Client Secret*を表示するアプリケーションを確認します。これらの値を両方ともコピーし、後で使用できるようにお気に入りのテキストエディタに保存します。

*Activate*をクリックしてクライアントアプリケーションをアクティブ化します。

> 注：*client_secret*のセキュリティを維持できないクライアントを処理するための認証方法には種々あります。私たちは簡単にするためにこの方法を使用していますが、IDCSのドキュメントではもっと詳細を記載しています。

### アプリケーションのOAuth 2.0トークンをテストする

作成したアプリケーションのクライアント資格情報でOAuth 2.0トークンのエンドポイントを呼び出します。

#### 例（cURLを使用）

```bash
curl -u <client_id>:<client_secret) -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&scope=http://129.213.40.20:8011/Read" https://idcs-ba024b2844b3414aa15581833cd48889.identity.oraclecloud.com/oauth2/v1/token
```

以下の例のようにaccess_token値を受け取ります。

```json
{
    "access_token": "eyJ4NXQjUzI1NiI6Ijg4em5OWVFaMER2Njl6TUh2UXNiSzA4SmhLaTR6d0RsUUNUV25HbHhVVFUiLCJ4NXQiOiJJUmJtbEQtaFIzWDV2WXhma1N1M2pyenVoM1kiLCJraiL5TVBV9eE2S8nHO-CFXMhpdOjGk6Xb8Q2M23XGb8PCfSKHH6iQbG-MHvgnDeOMdYy0XkV-yNsKmeSHm4JhndAFzWuiUdX0sZ7nvZ2sbRBXFA6M7Lfg8YzL2m3LfLMObX5zPA",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

### トークンをデコードして内容を表示する

1. *access_token*の値を引用符なしでコピーします。
1. ブラウザでjwt.ioを開きます。
1. エンコードされたトークンを*Encoded*ボックスに貼り付けます。

以下の例のように、*Decoded*ボックスにペイロードデータが表示されるはずです。

```json
{
  "sub": "1bf4d3e599ec22a9f937a49c9a4c",
  "user.tenant.name": "idcs-ba0244839b3414da735440cd48889",
  "sub_mappingattr": "userName",
  "iss": "https://identity.oraclecloud.com/",
  "tok_type": "AT",
  "client_id": "1bf4d3e599ec4923543a4a49c4d4c",
  "aud": "http://10.10.10.1:8011/",
  "sub_type": "client",
  "scope": "Read",
  "client_tenantname": "idcs-ba027ab544aa1559373cd48889",
  "exp": 1529362611,
  "iat": 1529359011,
  "client_guid": "67981394d5548c4510",
  "client_name": "APIClient",
  "tenant": "idcs-ba04aa1558ddcd48889",
  "jti": "4a8e-4e06-ba29"
}
```

スコープが*Read*である点に注目してください。

### 他のスコープを試す

リソースアプリケーションはCreate、Read、Update、Deleteをサポートしていますが、API用に作成したクライアントは*Create*と*Read*だけ許可しています。

トークンを取得するために同じ呼び出しを試してください。*Read*スコープの代わりに、*Update*スコープを使用してみてください。

#### 例（cURLを使用）

```bash
curl -u <client_id>:<secret) -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&scope=http://10.10.10.1:8011/Update" https://idcs-ba024b2844b3414aa15581833cd48889.identity.oraclecloud.com/oauth2/v1/token
```

次のような結果が表示されます。

```json
{
    "error": "unauthorized_client",
    "error_description": "Client authorization failed."
}
```

これは、クライアントにはWriteスコープが許可されていないためです。クライアントは*Update*スコープのトークンを取得できません。

## APIにOAuth 2.0ポリシーを追加する

### API Platform Cloud Serviceにログイン

1. 選択した環境の説明に従って、ブラウザで管理ポータルのURLをアクセスします。

    > このURLは、選択した[環境](../../../../../environments/README.md)に基づいていますが、形式は `http(s)://<host>:<port>/apiplatform`です。

1. *API Manager*ロールを持ち、かつ該当APIを管理する権限を持つユーザーとしてログインします。

### APIを編集してOAuth 2.0ポリシーを追加する

1. APIのリストからAPIを選択します。
1. サイド・タブから*API実装*を選択します。
1. *使用可能なポリシー*のリストで*セキュリティ*を展開します。
1. *OAuth 2.0*を選択し、*適用*をクリックします。
    1. 別の名前を選択して説明を追加できますが、*APIリクエスト*の後に配置するようにしてください。*次*（右矢印）をクリックすると、ウィザードの次の画面に移動します。
1. *少なくとも1つ*を選択し、*Get*メソッドの*Read*を有効なスコープとして追加し、*適用*をクリックします。

*OAuth 2.0*ポリシーには、いくつかのスコープ適用オプションがあります。それが正当なユーザーであることを検証するか（Any）、ユーザーが少なくともスコープリストにある1つのスコープを持つか、スコープをメソッドに関連付けることができます。たとえば、ユーザーがPOSTを使用してAPIを呼び出す場合は、ユーザーに*Create*スコープが必要です。

上の例では、読み取り専用にしているので、ユーザーには読み取りスコープが必要です。このAPIはユーザーが他のスコープを持っていても他のスコープを受け入れません。

> このAPIを本当に*Read Only*にしたい場合は、*インターフェース・フィルタ*ポリシーも使って、**GET**メソッドのみが使用される条件を設定します。

> 注意：18.2.5でテストした結果、HTTPメソッドが制限されていても、Readを呼び出してPOSTは除外されませんでした。ただし、各メソッドに使用する必要があるスコープを定義する際には制限されています。それは、Interface Filteringのようなメソッドを制限するようではありません。現在レビュー中です。

APIを保存して再デプロイします。

### あなたのAPIをテストする

1. 作成したクライアントアプリケーションを使用して、Readスコープのトークンを取得します。
2. 復習が必要な場合は、アプリケーションの作成時の[例](アプリケーション用のoauth-token)を参照してください。
3. トークンを使用してAPIを呼び出します。

#### Readスコープの例（cURLを使用）

クライアントアプリケーションには、*Read*と*Create*というスコープがあります。最初に*Read*を使ってAPIをテストしましょう。

##### OAuth 2.0トークンの取得

```bash
curl -u <client_id>:<secret> -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&scope=http://129.213.40.20:8011/Read" https://idcs-ba024aaa1558b89.identity.oraclecloud.com/oauth2/v1/token
```

##### トークンを使用してAPIを呼び出す

```bash
curl -H "Authorization: Bearer eyJ4NXQjUzI1NiI6Ijg4em5OWVFaMER2Njl6TUh2UXNiSzA4SmhLaTR6d0RsUUNUV25HbHhVVFHgQo0K3rV5Ez5WnrajltwGCEgbJ_6y2mraFyb2sVb4qi4RzS7DrZV-4IdlaBXE5R1j-swNIzHcRjPBBtK7BDJCw55cCI7QRTBJSLgt2rs86zZDhfSYxsrVhQxBOzvIusYA" -H "Accept: application/json" http://129.213.40.20:8011/beers/
```

##### API呼び出しの結果の例

```json
[
    {
        "id": "1",
        "label": "Breakfast Stout",
        "brewery": "Founders Brewing Company",
        "location": "Michigan",
        "ibu": 60,
        "abv": 8.30
    },
    {
        "id": "2",
        "label": "90 Minute IPA",
        "brewery": "Dogfish Head Craft Brewery",
        "location": "Delaware",
        "ibu": 90,
        "abv": 9.00
    },
    {
        "id": "3",
        "label": "Heady Topper",
        "brewery": "The Alchemist Brewery",
        "location": "Vermont",
        "ibu": 75,
        "abv": 8.00
    }
]
```

#### スコープの作成の例（cURLを使用）

##### OAuth 2.0トークンの取得

```bash
curl -u <client_id>:<secret> -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&scope=http://129.213.40.20:8011/Create" https://idcs-ba024b2844b3414aa15581833cd48889.identity.oraclecloud.com/oauth2/v1/token
```

##### トークンを使用してAPIを呼び出す

```bash
 curl -H "Authorization: Bearer eyJ4NXQjUzI1NiI6Ijg4em5OWVFaMER2Njl6TUh2UXNiSzA4SmhLaTR6d0RsUUNUV25HbHhVVFUiLCJ4NXQiOiJJUmJtbEQtaFIzWDV2WXhma1N1M2pyenVoM1kiLCJraWQiOiJTSUdOSU5H5s-92MLRv_-VZYqlwJQE7BgP6DYCEBwxFTliS147aheipO7gY3-w3uGGSbVcziQ" http://129.213.40.20:8011/beers/1
 ```

##### APIコールのサンプル結果

```UnAuthorized to access the resource.```

> これは、*Create*は許可されたスコープであるものの、APIは*Read*のみを許可するためです。再度申し上げますが、APIを確実に*read only*にするために、*インターフェース・フィルタ*ポリシーも使用する必要があります。

### ログを確認する

ゲートウェイはAPI呼び出しに関する情報を記録しています。ログを確認してOAuth 2.0ポリシーの実際の動作を観察できます。

1. ゲートウェイノードをインストールしたOSユーザまたは他のユーザとして、ゲートウェイサーバにログインし、適切なディレクトリにアクセスします。
1. `<nodeInstallDir>/domain/gateway/gateway1/apics/logs`に移動します。
1. `apics.log`を開きます。

#### ログエントリの例

次の例のような結果がログに表示されます。

##### 成功したReadリクエスト

`
[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.enforcer.strategy.impl.RetrieveJWKPublicKeyStrategy] Key ID for Selecting Public Key ::SIGNING_KEY

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.enforcer.strategy.impl.RetrieveJWKPublicKeyStrategy] Public Key is loaded :: Algorithm RSA::FormatX.509

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.validator.impl.JWTTokenValidatorImpl] Signature is valid => true

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.claim.APICSJWTClaimsVerifier] -------Token Validity  Claim is Validated ------

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.claim.APICSJWTClaimsVerifier] -------Issuer Claim is Validated ------

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.claim.APICSJWTClaimsVerifier] -------Subject Claim is Validated ------

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.claim.APICSJWTClaimsVerifier] -------Mandatory Claims is Validated ------

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.claim.APICSJWTClaimsVerifier] --- All the claims are validated --- 

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.claim.APICSJWTClaimsVerifier] --- Token is Valid ---

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.validator.impl.JWTTokenValidatorImpl] Audience Claim is validated

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.validator.impl.JWTTokenValidatorImpl] Scopes from Access Token is Read

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.validator.impl.JWTTokenValidatorImpl] Scope Read is asserted based on the list of scope ['[Read]'] defined in the policy

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.validator.impl.JWTTokenValidatorImpl] OAuth JWT Access Token is valid

[06-18 10:26:27:INFO oracle.apiplatform.policies.oauth.OAuthRuntime] !!!!! OAuth Access Token is validated !!!!!!!
`

##### 拒否されたCreateリクエスト

> 注意：簡潔にするため、重複するログステートメントは削除しています。具体的には、*Audience Claim is validated*などのイベントを重複しないように編集しています。

`
...
[06-18 10:35:27:INFO oracle.apiplatform.policies.oauth.validator.impl.JWTTokenValidatorImpl] Scopes from Access Token is Create

[06-18 10:35:27:ERROR oracle.apiplatform.policies.oauth.validator.impl.JWTTokenValidatorImpl] Scope from access token ['[Create]'] is not present in the list of scopes ['[Read]'] defined in the policy

[06-18 10:35:27:ERROR oracle.apiplatform.policies.oauth.OAuthRuntime] Error Occured in enforcing an Access Token ==>oracle.apiplatform.policies.oauth.exception.ScopeNotValidException: Scope from access token ['[Create]'] is not present in the list of scopes ['[Read]'] defined in the policy

[06-18 10:35:27:ERROR oracle.apiplatform.policies.oauth.OAuthRuntime] UnAuthorized to access the resource.Scope from access token ['[Create]'] is not present in the list of scopes ['[Read]'] defined in the policy
`

##### 無効なトークン

トークンの一部文字を変更するなど、無効トークンを試すと、ログに次のようなメッセージが現れます。

`
[06-18 10:53:45:INFO oracle.apiplatform.policies.oauth.enforcer.strategy.impl.RetrieveJWKPublicKeyStrategy] Key ID for Selecting Public Key ::SIGNING_KEY

[06-18 10:53:45:INFO oracle.apiplatform.policies.oauth.enforcer.strategy.impl.RetrieveJWKPublicKeyStrategy] Public Key is loaded :: Algorithm RSA::FormatX.509

[06-18 10:53:45:INFO oracle.apiplatform.policies.oauth.validator.impl.JWTTokenValidatorImpl] Signature is valid => false

[06-18 10:53:45:ERROR oracle.apiplatform.policies.oauth.OAuthRuntime] Error Occured in enforcing an Access Token ==>oracle.apiplatform.policies.oauth.exception.OAuthAccessTokenException: Signature Validation Failed

[06-18 10:53:45:ERROR oracle.apiplatform.policies.oauth.OAuthRuntime] UnAuthorized to access the resource.Signature Validation Failed
`

## 結論

この演習では、次のことを学びました。

- APIでOAuth 2.0ポリシーを使用するクライアントアプリケーションを定義する方法
- APIでOAuth 2.0ポリシーを追加および設定する方法
