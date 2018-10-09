# OAuth 2.0の設定

> このチュートリアルは現在作成中であり、完全ではありません。Early Adapterからのフィードバックをお待ちしております。

API Platform Cloud Serviceのゲートウェイは、OAuth 2.0クライアントとOAuth 2.0リソースサーバーの両方として機能できます。次のユースケースを考えてみましょう。

1. OAuth 2.0クライアント：サービスバックエンドはOAuth 2.0で保護されており、フロントエンドAPIの要件を満たしていません。これは、バックエンドシステムは内部ユーザーのみが存在し、APIコンシューマに利用できないまたは適切ではない場合があります。
1. OAuth 2.0リソースサーバ：ゲートウェイは、Bearerトークンを要求して検証するOAuth 2.0リソースサーバとして機能します。これにより、APIコンシューマに基づいてAPIレベルでアクセス制御を維持する機能が提供されます。

ゲートウェイは、同じフロー内でOAuth 2.0クライアントとリソースサーバとして機能することも、基本認証を使用するサービスバックエンドのためにモダンな認証方式を提供するためのOAuth 2.0フロントエンドとして機能することもできます。

## 始める前に

> お気に入りのテキストエディタを開きます。 **特殊文字を入れられないエディタであることを確認してください**。この演習中に、後で使用するための値を取得します。ゲートウェイにロードする*OAuth 2.0 Profile*ファイルを編集する場合もあります。

お気に入りのRESTクライアントを選択します。この演習ではcURLを使用した例を提供しますが、REST呼び出しに必要な属性を提供する場合もあります。選択した特定のRESTクライアントで属性の設定方法を理解しておく必要があります。

この演習には、リクエストがあらかじめ設定されているPostmanコレクション（RESTクライアントの場合）も用意しています。コレクションでは変数を使用します。この演習では、コレクションを正しく動作させるために変数を更新する必要があります。

### 前提条件

- API Platform Cloud Service環境がある前提とします。API Platform Cloud Service環境の調達の詳細については、[環境](../../../../environments/README.md)を参照してください。

  > 注：リンクは常に新しいタブで開いてください（右クリック→*新しいタブでリンクを開く*）。そうすれば、タスクを完了後にリンク先のチュートリアルなどからこの演習ガイドに戻ってくることができます。

- Identity Cloud Serviceの*アイデンティティ・コンソール*にアクセスする権限があり、かつ
  - ユーザー作成の権限がある
  - アプリケーション作成権限がある
- 利用環境に少なくとも1つのゲートウェイが配置されていて、かつ
  - ゲートウェイを管理するWeblogicドメインとゲートウェイノードを実行するマシンのOSへのアクセス資格情報がある
  - *ゲートウェイ・マネージャー*ロールを持つユーザーの資格情報および、プロファイルをインストールしようとしているゲートウェイの*ゲートウェイの管理*権限がある
  - プロファイルをインストールしようとしているゲートウェイが使用する*ゲートウェイ・ランタイム*ロールを持つユーザーの資格情報がある
  - `APIPCSAUTO_<name of your instance>`アプリケーションの*Client ID*と*Client Secret*がある
- ゲートウェイノードをまだ配置していない場合は、[ゲートウェイノードのインストール](../gateway_node/README.md)を参考にしてゲートウェイノードをインストールできます。

## Identity Cloud Service (IDCS)へのOAuth 2.0リソースサーバーとクライアントの登録

### IDCSにログイン

1. *Identity Administrator*権限を持つユーザーを使用してOracle Cloud Domainにログインします。
1. ダッシュボードの右上にある*ユーザー*メニューをクリックします。
1. *アイデンティティ・コンソール*ボタンをクリックします。

#### ゲートウェイのアプリケーションを作成する

1. サイドメニューから*Applications*を選択します。
1. *Add*をクリックします。
1. *Trusted Application*を選択します。
1. アプリケーションに*API Gateway*のような名前を付けます。この環境を他の人と共有している場合は、アプリケーションの名前を一意にします。たとえば、あなたの名前を付けます。*Next*をクリックします。

  > リソースアプリケーションを作成し、そのリソースアプリケーションを使用する1つまたは複数のクライアントアプリケーションを作成するのがベストプラクティスです。リソースとともにクライアントインターフェイスを作成することもできます。この演習では、私たちは両方を作成し、テストのためにクライアントアプリケーションを使用します。
  >
  > 今後、このゲートウェイを活用するAPIコンシューマー向けのクライアントアプリケーションを追加できます。

1. *Configure this application as a client now*を選択します。
    1. *Resource Owner*、*Client Credentials*、*Refresh Token*を選択します。
    1. *Introspect*を選択します。
    1. *Trust Scope*の*All Resources*を選択します。
    1. *Next*をクリックします。
1. *Configure this application as a resource server now*を選択します。
    1. *Is Refresh Token Allowed*を選択します。
    1. *Primary Audience*を設定します。これはどんなテキストでもかまいません。複数の点を一致させるだけです。ゲートウェイのロードバランサアドレスを使用することがベストプラクティスです。
        - 他の人と環境を共有していて、ゲートウェイに固有の*ロードバランサアドレス*がない場合は、固有のゲートウェイ名を使用します。
        - 独自のゲートウェイで独自の資格を使用している場合は、*ロードバランサアドレス*を使用できます。
        - audienceの最後に `/`をつけてください。
    1. （Create, Read, Update, Delete）にAllowed Scopesを追加します。
        - アプリケーションの目的に基づいて、任意のスコープに名前を付けることができます。
        - この時点では、単にゲートウェイをリソースサーバーとして設定していますが、将来、APIをスコープで区別できます。この演習では、基本的なCRUD機能をテストするように設定します。
    1. *Next*をクリックし、*Web Tier Policy*タブでは*Skip for later*を選択します。
    1. *Next*をクリックし、*Finish*をクリックします。
    1. アプリケーションを使用する前に*Activate*化しておいてください。

  > *Cient ID*と*Client Secret*を表示するアプリケーションを確認します。これらの値を両方ともコピーし、後で使用できるようにお気に入りのテキストエディタに保存します。
  > 異なるクライアントを作成することができます。たとえば、読み取りのみ可能、レコードを変更できないクライアントは、*Read*スコープのみ持っています。上記では、レコードの作成と読み取りは可能ですが、レコードの更新や削除はできないクライアントとしてテストします。

## OAuth 2.0トークンについて

トークンへのアクセスとトークンの解釈をテストしましょう。

### すべてのスコープでトークンを取得する

1. 以下の属性でトークン・エンドポイントを呼び出します。
    - URL： `https://<Identity Domain> .identity.oraclecloud.com/oauth2/v1/token`（*Oracle Cloud Account*にサインインする際に表示されます）
    - メソッド：POST
    - 基本認証
      - Username：あなたが作成したアプリケーションの*Client ID*
      - Password：あなたが作成したアプリケーションの*Client Secret*
    - ヘッダー
      - `Content-type：application/x-www-form-urlencoded`
    - Form
      - `grant_type = client_credentials`
      - `scope = urn：opc：idm：__ myscopes__`

#### トークンを取得する例（cURLを利用した例）

  > 注：値は公開のために変更されています。トークンサンプルはレンダリングされません。

`curl -u 79b0673:442af -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&scope=urn:opc:idm:__myscopes__" https://idcs-ba024b2844b3414aa15581833cd48889.identity.oraclecloud.com/oauth2/v1/token`

次のような結果が表示されるはずです。

```json
{
    "access_token": "eyJ4NXQjUzI1NiI6Ijg4em5OWVFaMER2Njl6TUh2UXNiSzA4SmhLaTR6d0RsUUNUV25HbHhVVFUiLCJ4NXQiOiJJUmJtbEQtaFIzWDV2WXhma1N1M2pyenVoM1kiLCJraWQiOiJTSUdOSU5HX0tFWSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiI3OWIwMDBkNzU0OTc0ZjA4Yjg2OThjNDI3MzJkZjY3MyIsInVzZXIudGVuYW50Lm5hbWyg5IiwianRpIjoiMDUyZDY5NWItMWYxZS00NTFmLTk3OGEtMmQ2MmM3YzBjMzA0In0.a8LW7GHF8uIpRRy0VHV8tDUerA2O6SIKEfdLp3tijBzpfqYbcn1E8X1qCv5nvKO1uDvBa0GjxixFyPkzs2OBlLfBsxx0D0C67GmaBzOjgzGX1Ah_rKK7x4_9lhlLKJtRnMos6PhP0_2wk1L64Yzc1yd64xN9Uhf30hew",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

マシンに `jq`がインストールされている場合は、cURLコマンドを更新して、次のステップに必要な*access_token*の値だけを選択することができます。

#### トークンの値だけを取得する例

`curl -u 79j73:4fg-450xx01af -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&scope=urn:opc:idm:__myscopes__" https://idcs-ba024b28xx414aa147291833cd48889.identity.oraclecloud.com/oauth2/v1/token | jq -r '.access_token'`

次のような結果が表示されるはずです。
`eyJ4NXQjUzI1NiI6Ijg4em5OWVFaMER2Njl6TUh2UXNiSzA4SmhLaTR6d0RsUUNUV25HbHhVVFUiLCJ4NXQiOiJJUmJtbEQtaFIzWDV2WXhma1N1M2pyenVoM1kiLCJraWQiOiJTSUdOSU5HX0tFWSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiI3OWIwMDBkNzU0OTc0ZjA4Yjg2OThjNDI3MzJkZjY3MyIsInVzZXIudGVuYW50Lm5hbWUiOiJpZGNzLWJhMDI0YjI4NDRiMzQxNGFhMTU1ODE4MzNjZDQ4ODg5Iiwic3ViX21hcHBpbmdhdHdYjI4NDRiMzQxNGFhMTU1ODE4MzNjZDQ4ODg5IiwianRpIjoiY2ZmYzZmZGQtMTA5OS00NDg2LTlmYWMtOGIzMDJjN2VmNjA1In0.gMsusYorDC2BdXLlXh-x9bARZ-8H0p9SgOZHx-RuLDeKXH2vMdFnk3EPtllEJYJLlxoZv5U4kouXx32th1Oq0ubXwrJyTLIVwMpUK6ucjcerKGBHA8SMCvuLzhIEQ2Tf5qPFJA5_livj3vZZt6Pnt0lr4q2Q8-g-E2hZatZAlrkViL00Kf08SGJIvUnZhydMGa3chSIkgjrgnjyw2JxLck4aSKlA1XRZayDkdT8oNQV1OkLGZKPg`

これは*access_token*の値のみであることに注意してください。

## トークンの解釈

トークンを表示する簡単な方法は、ブラウザを開き、`http://jwt.io`に入力することです。

*access_token*の値を *PASTE A TOKEN HERE*と記述されている**Encoded**ボックスに貼り付けます。これは、上記の2番目の出力例と同様に、access_tokenの値だけです。

**PAYLOAD：DATA**の結果は、次のように表示されるはずです。

```json
{
  "sub": "76649b000dvd32drf673",
  "user.tenant.name": "idcs-ba0248889",
  "sub_mappingattr": "userName",
  "iss": "https://identity.oraclecloud.com/",
  "tok_type": "AT",
  "client_id": "79b09f673",
  "user_isAdmin": false,
  "aud": [
    "urn:opc:lbaas:logicalguid=idcs-ba024bb9",
    "https://idcs-ba0249.identity.oraclecloud.com"
  ],
  "sub_type": "client",
  "clientAppRoles": [
    "Authenticated Client"
  ],
  "scope": "urn:opc:idm:t.security.client",
  "client_tenantname": "idcs-ba024bb889",
  "exp": 1529342878,
  "iat": 1529339278,
  "client_guid": "370f740",
  "client_name": "APIGateway",
  "tenant": "idcs-ba9400ab3475889",
  "jti": "1099-4486-9fac"
}
```

後で使用するために、この出力から **iss** の値をコピーして保存します。

## OAuth 2.0プロファイルの作成とゲートウェイへのロード

**OAuth 2.0 Resource Server**として機能するには、**OAuth 2.0 Authorization Server**のOAuth 2.0プロファイルを作成する必要があります。この場合はIDCSですが、JSON Web Token（JWT）を生成する際にRFC7519に準拠する他のサーバーを使用することもできます。

OAuth 2.0トークンを検証するために、IDCSとAPI Gatewayとの間に信頼関係を確立します。 JSON Web Token（JWT）は、次の方法で保護されています。

- トークンを取得および使用する場合は、SSL通信のみ使用できます。これにより、攻撃者は移動中のトークンにアクセスできなくなります。
- クライアントがアクセス権を持たないリソース（ReadではなくWrite）のスコープを要求した場合、IDCSはそのクライアントにトークンを提供しません。
- IDCSはJSON Web Keyを使用してトークンに署名します。鍵はゲートウェイに登録され、ゲートウェイはそれを使用してトークンを検証します。
  - クライアントが許可されているスコープを要求し、トークンのスコープを変更しようとすると、署名チェックで無効なトークンとして、APIゲートウェイは要求を拒否します。

### IDCSからJWKを取得する

OAuth 2.0プロファイルを作成する準備として、IDCS環境からJSON Web Key（JWK）を取得する必要があります。これを行うには、OAuth 2.0トークンを取得してから、SigningCertエンドポイントを呼び出す必要があります。

1. 有効なJSON Webトークンを取得します。トークンは1分後に期限切れになるように設定されているため、新しいトークンを要求する必要があります。リフレッシャーが必要な場合は、前述の手順を確認してください。
2. SigningCertエンドポイントを呼び出してキーを取得します

- URL： `https://<Your IDCS Domain>.identity.oraclecloud.com/admin/v1/SigningCert/jwk`
- メソッド： `GET`
- Headers：
  - `Authorization: Bearer <前のステップで取得したトークン値>`
  - `Accept: application/scim+json,application/json`

#### 例（cURLを使用）

##### OAuth 2.0トークンの取得

```bash
curl -u 79b000d754974f08b8698c42732df673:442eda5f-b8ae-44e8-996b-450dd1c301af -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&scope=urn:opc:idm:__myscopes__" https://idcs-ba024889.identity.oraclecloud.com/oauth2/v1/token
{"access_token":"eyJ4NXQjUzI1NiI6Ijg4em5OWVFaMER2Njl6TUh2UXNiSzA4SmhLaTR6d0RsUUNUV25HbHhVVFUiLCJ4NXQiOiJJUmJtbEQtaFIzWDV2WXhma1N1M2pyenVoM1kiLCJraWQiOiJTSUdOSU5HX0tFWSIsImFsZyI6IlJTMjU2In0.ey8gD09Fauu9SKqVZCX4vR57KyIRUKrtEVc9jvRPu1ykdc0YP0mScavW-0J1JYsPNlyBavXlvwjaSOYEBWG1TCz_W_DbGIU9GqjdIIiIC-KIqEoBRLs3aWddi5NZj-gBOscmKcgqxZh1lkSMuhyxwqdScNpZ0D0C67GmaBzOjgzGX1Ah_rKK7x4_9lhlLKJtRnMos6PhP0_2wk1L64Yzc1yd64xN9Uhf30hew","token_type":"Bearer","expires_in":3600}
```

##### トークンを使用してSigning Certのエンドポイントを呼び出す

```bash
curl -H "Authorization: Bearer eyJ4NXQxs6KWNVEXv9bhQcv7zfRLi6T3ueu9fbEknqc7p2ppLNPRBGIK52mIG39luTMuLdpY_0L6k-Xd9IRPSKR-dN1gwR8poY0_x3uYXjYiKRZ8D_jD-T2bMzbLsgL78rh7ui1b1cMz6fdk-K8aVmlLbNJniLy39-s2RiaeK37VYS5zkhe4W9bkewCCukBPc09rHSD8ougX4-WPs2YF57h_c-uxhnnuKRuuUS6iqwGdHLdNfUUVj2ihM4VlxHnQq9V1RxogxKYQ029ijWhy3tzFFMHyoUt-Y-r-8h5LULnKoHcuDf7xKAB5ILdFYg" -H "Accept: application/scim+json,application/json" https://idcs-ba024b2844b3414aa15581833cd48889.identity.oraclecloud.com/admin/v1/SigningCert/jwk
```

##### SigningCertの結果例

>注：サンプルは変更されており、実際の結果とはもっと完全になります。

```json
{
    "keys": [
        {
            "kty": "RSA",
            "x5t#S256": "88znNYlxUTU",
            "e": "AQAB",
            "x5t": "IRbmlD-hR3X5vYxfkSu3jrzuh3Y",
            "kid": "SIGNING_KEY",
            "x5c": [
                "MIIDXz+2Z3FR0CWdpx+dT91JmalvB01QlcCjdXv4SGKXmF",
                "MIIDdDCCAlygAwIBAgIGAVw4Ns68MA0GCSqGSIb3DQEBCwUAMFcxEzARBgoJkiaJk/IsZAEZFgNjb20xFjAUBgoJkiaTAPBgNVHQ8BAf8EBQMDB9gAMA0GCSqGSIb3DQEBCwUAA4IBAQBKcaopdLvr5+EstU3zjpPoZh4XnKvJuD+1J8LVDei"
            ],
            "key_ops": [
                "encrypt",
                "sign"
            ],
            "alg": "RS256",
            "n": "lIOuSLMNto9t5P2YmGAqXCW_I19vBpzjjDkfoNh1o7ZEbS5VjKW3uaq9yuVTmYhrIMj1k_WOQGxN3QFTmQ"
        }
    ]
}
```

返された完全な値をOAuth 2.0プロファイルで使用するので、必ず取得しておいてください。

### OAuth 2.0プロファイルXMLファイルの完成

OAuth 2.0プロファイルには、次の値を設定します：

- Issuer：あなたがトークンで受け取った **iss** の値
- Audience：IDCSのリソースアプリケーションに定義された **Audience** の値
- useFormat：**JWKFormatPubKey**
- `<JWKFormatPubKey>`*返された完全なキーの値*`</JWKFormatPubKey>`を含む

> 注意：ドキュメント内のサンプルには **PEMFormatPubKey** および **X509FormatPubKey** がありますが、 **JWKFormatPubKey** は記述されていません。これらのサンプルを使って作成しようとしている場合、今回は **JWKFormatPubKey** を使用するため、前2者のサンプルは削除してください。

以下はOAuth 2.0プロファイルのサンプルです。

```xml
<OAuth2TokenLocalEnforcerConfig>
	<Name>DEFAULT</Name>
	<Issuer>https://identity.oraclecloud.com/</Issuer>
        <Audience>http://129.213.40.20:8011/</Audience>
	<AudienceRestrictionFromConfig>true</AudienceRestrictionFromConfig>
	<!-- useFormat has 2 values  PEMFormatPubKey, X509FormatPubKey -->
	<PublicCertLocation useFormat='JWKFormatPubKey'>
		<JWKFormatPubKey>{"keys":[{"kty":"RSA","x5t#S256":"88znNYQZ0Dv69nGlxUTU","e":"AQAB","x5t":"IRbmlD-hR3X5vYxfkSu3jrzuh3Y","kid":"SIGNING_KEY","x5c":["MIIDXzCCAkegAwIBAgIGAWEwsA2SMA0GCSqGSIb3DQEBCwUAMFcxEzARBgoJkiaJk/IsZAEZFgNjb20xFjAUBgoJkiaJk/IsZAEZFgZvcmFjbGUxFTATBgoJkiaJk/IsZAEZFgVjbG91ZDERMA8GA1UEAxMIQ2xvdWQ5Q0EwHhcNMTgwMTI2MDQxODE5WhcNMjgwMTI2MDQxODE5WjBWMR+eckE3WvNtZKOwTfO3wTnli/IrauKfLJPOsIGnYy4lohr8k1iuJuroNh1bAY2ZJidoo2zz/pTRRBYGJ00Q+NwrnSUVpHk2qV0N9e6KX9v5jL98SdNjRjoQQ8CQGokNRvwIDAQABo0YwRDASBgNVHRMBAf8ECDAGAQH/AgEAMB0GA1UdDgQWBBRs+bdtrcF6S9hWxzjj2hnZyxS6RTAPBgNVHQ8BAf8EBQMDB9gAMA0GCSqGSIb3DQEBCwUAA4IBAQBKcaopdLvr5+EstU3zjpPoZh4XnKvJuD+1J8L1XGkA0brP0sAwg_h-kN4zZqRr8Mb1FwvV1NJpwXAiFDe0Tq_KXui5Fn8U-a4IqMcadoNAc9v6VCFTmQ"}]}</JWKFormatPubKey>
  </PublicCertLocation>
</OAuth2TokenLocalEnforcerConfig>
```

### OAuth 2.0プロファイルのゲートウェイへのロード

OAuth 2.0プロファイルの準備が完了したら、ゲートウェイにロードする必要があります。

1. ゲートウェイを実行しているマシンに（SSH）をログインします。
1. ゲートウェイの*gateway-props.json*ファイルを開きます。
1. *gateway-props.json*ファイルで次の設定を確認します。
    1. nodeInstallDir
    1. managementServerHost
    1. managementServerPort
    1. oauthProfileLocation
1. OAuth 2.0プロファイルファイルをoauthProfileLocationで指定したディレクトリに配置します。この設定がない場合は、ディレクトリを作成して、設定をgateway-props.jsonファイルに追加します。

> 注意：nodeInstallDirの下にOAuth 2.0プロファイル・ディレクトリを追加しないでください。ノードを再インストールすると、ディレクトリが消去され、OAuth 2.0プロファイルファイルが削除されるためです。

終了後、次のコマンドを実行します。

`./APIGateway -f gateway-props.json -a updateoauthprofile`

> 注意：上記の例では、gateway-props.jsonファイルはゲートウェイ用の構成ファイルの名前であり、インストーラと同じディレクトリにあることを前提としています。これが異なる場合は、適宜変更する必要があります。

*gateway-props.json*ファイルの名前が異なる名前のインストールの例をいくつか示します。

#### 例（gateway-props-gse.json）

```json
{
    "logicalGatewayId" : "100",
    "managementServiceUrl" : "https://<instanceName>-<IdentityDomain>.apiplatform.ocp.oraclecloud.com:443",
    "idcsUrl" : "https://idcs-ba0d48889.identity.oraclecloud.com/oauth2/v1/token",
    "requestScope" : "https://2890D9.apiplatform.ocp.oraclecloud.com:443.apiplatform",
    "gatewayNodeName" : "GSE Dev 1 Node 1",
    "gatewayNodeDescription" : "GSE Dev 1 Node 1",
    "listenIpAddress" : "10.0.0.2",
    "publishAddress" : "10.0.0.2",
    "nodeInstallDir" : "/u01/gw",
    "oauthProfileLocation" : "/u01/oauth/SampleOAuth.xml",
    "gatewayExecutionMode" : "Development",
    "heapSizeGb" : "2",
    "maximumHeapSizeGb" : "4",
    "gatewayMServerPort" : "8011",
    "gatewayMServerSSLPort" : "9022",
    "nodeManagerPort" : "5556",
    "coherencePort" : "8088",
    "gatewayDBPort" : "1527",
    "gatewayAdminServerPort" : "8001",
    "gatewayAdminServerSSLPort" : "9021"
}
```

#### プロファイルをロードする例

```text
[opc@api-gw-n-1 ~]$ ./APIGateway -f gateway-props-gse.json -a updateoauthprofile
Please enter user name for weblogic domain,representing the gateway node:
weblogic
Password:
2018-06-18 19:56:23,775 INFO Initiating validation checks.
2018-06-18 19:56:23,835 INFO validation complete
2018-06-18 19:56:23,835 INFO Install action logs are located in /u01/gw/logs
2018-06-18 19:56:23,836 INFO Logging to file /u01/gw/logs/main.log
2018-06-18 19:56:23,836 INFO Outcomes of operations will be accumulated in /u01/gw/logs/status.log
Please enter gateway manager user:
api-gateway-manager                  
Password:
Please enter gateway manager client id:
<client_id>
Please enter gateway manager client secret:
<secret>
Please enter gateway manager runtime user:
api-gateway-runtime
Password:
Please enter gateway manager runtime client id:
<client_id>
Please enter gateway manager runtime client secret:
<secret>
2018-06-18 19:58:05,461 INFO Performing update oauth profile step.
2018-06-18 19:58:35,317 INFO Update oauth profile step complete. Status = UPDATE_EXECUTED .Please refer log file for details.
2018-06-18 19:58:35,318 INFO Execution complete.
```

インストーラを実行すると、 `<nodeInstallDir>`/logsにいくつかのログが生成されます。

##### main.log

```text
2018-06-18 19:56:23,836 INFO Logging to file /u01/gw/logs/main.log
2018-06-18 19:58:05,461 INFO Performing update oauth profile step.
2018-06-18 19:58:35,317 INFO Update oauth profile step complete. Status = UPDATE_EXECUTED .Please refer log file fo
r details.
2018-06-18 19:58:35,318 INFO Execution complete.
```

##### updateOAuthProfile.log

```text
DEBUG:root:trying to fetch IDCS access token
DEBUG:root:Performing dev env SSL related workarounds
DEBUG:root:Calling https://idcs-ba024b83768889.identity.oraclecloud.com/oauth2/v1/token to fetch 
access token ...
DEBUG:root: Invoked access token call
DEBUG:root:Fetched Ok response
DEBUG:root:IDCS Access token fetch successful
DEBUG:root:Trying update oauthprofile = /u01/oauth/SampleOAuth.xml
DEBUG:root:Calling REST API to updateoauth profile on url = http://10.0.0.2:8011/apiplatform/gatewaynode/v1/securit
y/profile
DEBUG:root:Invoked REST API to update oauth profile.
DEBUG:root:Fetched OK response from REST API.
DEBUG:root:
INFO:main:Update oauth profile step complete. Status = UPDATE_EXECUTED .Please refer log file for details.
INFO:main:Execution complete.
```

## 次のステップ

### OAuth 2.0トークンを検証＆実施するためのAPIの更新

OAuth 2.0トークンを検証＆実施するようにAPIを設定する方法については、[APIでのOAuth 2.0の使用](../../../manage/apis/policies/oauth/README.md)を参照してください。