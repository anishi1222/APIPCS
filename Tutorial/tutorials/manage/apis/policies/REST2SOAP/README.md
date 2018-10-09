# *REST APIからSOAPサービス*ポリシー

REST to SOAPポリシーを使用すると、SOAPバックエンドサービス用のRESTフロントエンドやAPIを提供できます。これはSOAPスタイルのサービスがあるけれども、クライアントからRESTで操作できる必要がある、という一般的なユースケースです。このチュートリアルでは、SOAPサービス用のREST APIを作成するプロセスについて説明します。

>　注意：SOAPサービスを有している必要があります。既に有していれば、それを使ってください。このチュートリアルでは、公開されている単純な[Calculator Service](http://www.dneonline.com/calculator.asmx)を使用します。このサービスを使用して、いくつかの基本的な概念を演習できます。[Calculator Service](http://www.dneonline.com/calculator.asmx)にアクセスして、このチュートリアルで使用する**wsdl**ファイルをダウンロードできます。

## 始める前に

- 利用可能なAPI Platform Cloud Service環境を有している必要があります。API Platform Cloud Service環境の調達の詳細については、[環境](../../../../../environments/README.md)を参照してください。
- [サービスの作成](../../../services/create_service/README.md)と[APIの作成](../../../apis/create_api/README.md)のチュートリアルを完了し、サービスやAPIの作成を理解している前提とします。

## SOAPサービスのREST APIを提供するAPIの作成

### SOAPサービスの登録

[サービスの作成](../../../services/create_service/README.md)チュートリアルで学んだように、1つ以上のAPIで使用するサービス実装のエンドポイントを登録できます。サービスで使用する **.wsdl** を関連付ける必要があるため、APIで使用するためには事前にSOAPサービスを登録する必要があります。

- 以下のようにサービスを登録してください。
  - サービスに*CalculatorService*と名前をつけます。
  - バージョンと説明をいれます。
  - タイプは*SOAP*を選択します。
  - [Calculator Service](http://www.dneonline.com/calculator.asmx)からダウンロードした **.wsdl** を選択します。

    > 注意：サービスの作成を復習する必要がある場合は、[サービスの作成](../../../services/create_service/README.md)チュートリアルを参照してください。

### Calculator APIの作成

次に、SOAPサービス用のRESTインターフェイスを提供するAPI実装を作成します。

- 次のようにAPIを作成します。
  - APIに*Calculator*と名前をつけます。
  - バージョンと説明をいれます。
    - *Plan Manager*ロールを持っている場合は、プランを作成するオプションが表示される場合があります（無視もできます）。
- APIを作成した後、*API Request*と*Service Request*を次のように設定します。
  - APIリクエスト：エンドポイントを*calc*に設定します。
  - サービスリクエスト：[SOAPサービスの登録](#SOAPサービスの登録)で作成したSOAPサービスを選択します。

### REST API to SOAP Serviceポリシーの追加

この演習では、クライアントの要件はパラメータ*a*と*b*（2つの数値）を受け入れるサービスを提供することとします。このサービスは、Operationを1つのリソースとして受け入れる必要があります。 たとえば、2つの数値を加算する場合、クライアントはAPIを`http(s)://MyGatewayIP/calc/add?a=20&b=30`と呼び、次のような結果を期待しています。

```json
{
    "Sum": "50"
}
```

SOAP（over HTTP）サービスは、HTTP POSTを使用して、SOAP Envelopeで指定された操作で、特定のエンドポイントに対してを実行します。この場合、4つのOperation（加減乗除）をRESTスタイルのAPIに変換する必要がありますが、これらは実際にHTTPメソッドに変換せず、リソースを使ってOperationにマップします。では*REST API to SOAP Service*ポリシーを追加しましょう。

#### APIにポリシーを追加する

- *使用可能なポリシー*で*インタフェース管理*を展開し、マウスを*REST APIからSOAPサービス*の上に移動し、*適用*をクリックします。
  - 最初の画面で、名前とコメントを任意の値に設定できます。リクエスト/レスポンス・パイプラインの配置はデフォルトのままにして、ナビゲーションをクリックしてウィザードの次のページに移動します。
  - *変換構成*で、*サービスの選択*ボタンをクリックし、[SOAPサービスの登録](#SOAPサービスの登録)で作成したSOAPサービスを選択します。
  - 選択するとメソッドが設定され、1つ以上のメソッドを選択できます。現時点では、*Add*操作を選択するだけです。

#### メソッドから操作のマッピングを設定する

- メソッドを*GET*に更新します。この場合、メソッドやhttp動詞は実際に数学の演算に変換されません。
- 操作のパスはOperationと同じにしますが、小文字にします。例えば、*Add*オペレーションの場合は、Pathを*add*に設定します。マッピングは次のように設定する必要があります。
  - メソッド：GET
  - パス：add
  - 操作：Add

##### ペイロードマッピングの設定

Method to Operation Mappingが拡張可能であることに注意してください（右側の矢印）。最初のマッピング（Add）を展開すると、SOAPペイロードが次のように表示されます。

```xml
<Envelope xmlns="http://www.w3.org/2003/05/soap-envelope">
   <Body>
      <Add xmlns="http://tempuri.org/">
         <intA>${payload.Add.intA}</intA>
         <intB>${payload.Add.intB}</intB>
      </Add>
   </Body>
</Envelope>
```

*intA*と*intB*要素にフォーカスを当ててください。APIにはいくつかの変数があります。

- **ペイロード**：リクエストの本文
- **ヘッダー**：リクエストのhttpヘッダー
- **クエリ**：httpクエリのパラメータ

*ペイロード*はメッセージ本体であり、このポリシーを使用して、バックエンドのSOAPサービスと簡単にマップできます。対応するREST/JSONリクエストは次のようになります。

```json
{
    "Add":
    {
        "intA": 20,
        "intB": 30
    }
}
```

私たちのクライアントが上記の本文でPOSTリクエストを行うと予想された場合、これは期待通りに動作するはずです。しかし、クライアントの要求は、クエリパラメータを渡してGETでAPIを呼び出すので、ペイロードを次の通り更新する必要があります。

```xml
<Envelope xmlns="http://www.w3.org/2003/05/soap-envelope">
   <Body>
      <Add xmlns="http://tempuri.org/">
         <intA>${queries.a}</intA>
         <intB>${queries.b}</intB>
      </Add>
   </Body>
</Envelope>
```

クエリパラメータ*a*と*b*を使用していることに注意してください。よって、`http(s)://MyGatewayIP/calc/add?a=20&b=30`のようなリクエストが期待どおりに機能するはずです。

レスポンスペイロードについては、ペイロードマッピングの*SOAPからREST*タブを選択すると、次のような結果が表示されます。

```json
{
  "AddResponse":{
    "AddResult": "${payload.Body.AddResponse.AddResult}"
  }
}
```

これを次のように変更しましょう。

```json
{
    "Sum": "${payload.Body.AddResponse.AddResult}"
}
```

ペイロードを修正したので、*30*と*50*を加算するためにAPIを呼び出すときに、次のような結果が得られます。

```json
{
    "Sum": 50
}
```

*適用*と*保存*を必ずクリックしてください。

### APIのデプロイとテスト

復習が必要な場合は、[APIのデプロイ](../../deploy_api/README.md)チュートリアルを参照してください。

APIをデプロイ後、APIを呼び出します。この場合、パラメータ付きの単純なGETであるため、WebブラウザでAPIを呼び出すことができます。APIの呼び出しに関して復習が必要な場合は、 [APIの呼び出し](../../invoke_api)にアクセスしてください。

## 結論

このチュートリアルでは、次のことを学びました。

- SOAPサービス用のREST APIを作成する方法
- API実行変数の使い方
- レスポンスペイロードを作り直す方法

## その他

残りの操作(Add以外)のステップを実行し、差、積、および商に対するレスポンスペイロードの変更方法を実践してください。