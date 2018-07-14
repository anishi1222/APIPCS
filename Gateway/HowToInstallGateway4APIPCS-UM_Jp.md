# Gateway installation procedure for User managed APIPCS

## 注意事項

- ノードは事前に準備済みという想定です。
- ここで示す手順は、以下の条件でインストールする場合を示しています。環境が異なる場合は、適宜読み替えてください。
- Windows環境にインストールする場合、OpenSSLとPythonを事前にインストールおよび構成しておく必要があります。

| 項目 | 条件 |
|:--|:--|
| インストール、実行User | oracle |
| JDK (JAVA_HOME)の展開先 | /u01/java |
| Gateway Installerの展開先 | /u01/gwinst |
| GatewayのInstall先 | /u01/apipcs/gw/install |

## ドキュメント

以下にあります。
> Installing a Gateway Node <br/>
> [https://docs.oracle.com/en/cloud/paas/api-platform-cloud-um/apfad/installing-gateway-node.html#GUID-6848A2B0-03D0-435B-B867-5D9FD80E595B](https://docs.oracle.com/en/cloud/paas/api-platform-cloud-um/apfad/installing-gateway-node.html#GUID-6848A2B0-03D0-435B-B867-5D9FD80E595B)

## 事前構成

1. Userの追加（oracle）
2. 作業ディレクトリの作成(/u01)
3. ホスト名の解決のための設定
    - /etc/hostsへの追加・編集
    - /etc/sysconfig/networkや/etc/hostnameの編集
4. 必要であれば、Swapfileを作成 (OPCの場合。Bare Metalであれば作成済み)
    ```bash
    # 4GBのSwapを作る
    dd if=/dev/zero of=/swapfile bs=1024K count=4096
    mkswap /swapfile
    swapon /swapfile
    # /etc/fstabに追加しておく
    ```

## 依存モジュールの展開および構成

1. /tmpにopcユーザーで以下を転送
    - Gateway Installer
    - JDK (Server JREで可。tar.gzのほうが使いやすい）
2. oracleユーザーで転送済みファイルを展開
    - Gateway Installer：/u01/gwinst/
    - JDK : /u01/java
3. ~oracle/.bash_profileを編集（環境変数の設定）
    ```bash
    export JAVA_HOME=/u01/java
    export PATH=$JAVA_HOME/bin:$PATH
    ```
4. 環境変数の反映
    ```bash
    . ~oracle/.bash_profile
    ```
5. JDKの乱数生成設定を変更（rpmでJDKをインストールしている場合、rootで作業実施する必要がある）
    - $JAVA_HOME/jre/lib/security/java.securityを開く
    - securerandom.source=file:/dev/random の記述を securerandom.source=file:/dev/urandom に変更する（randomをurandomに変更）
    ```bash
    securerandom.source=file:/dev/urandom
    ```

## Gatewayのインストール

1. gateway-props.jsonを編集する。Management Serviceでウィザードに従って構成することも可能。以下は例。
    ```json
    {
        "nodeInstallDir": "/u01/apipcs/gw/install",
        "installationArchiveLocation": "/u01/apipcs/gw/archives",
        "logicalGateway": "LGW02",
        "gatewayNodeName": "GWNode02",
        "managementServiceUrl": "https://xxx.xxx.xxx.xxx:443",
        "oauthProfileLocation": "/u01/apipcs/gw/security/OAuth2TokenLocalEnforcerConfig.xml",
        "listenIpAddress": "yyy.yyy.yyy.yyy",
        "publishAddress": "zzz.zzz.zzz.zzz",
        "managementServiceConnectionProxy": [],
        "nodeProxy": [],
        "gatewayExecutionMode": "Development",
        "gatewayAdminServerPort": "7501",
        "gatewayAdminServerSSLPort": "7502",
        "gatewayMServerPort": "8001",
        "gatewayMServerSSLPort": "8002",
        "heapSizeGb": "2",
        "maximumHeapSizeGb" : "2",
        "opatchesFolder": "/u01/patches",
        "logicalGatewayId": "101"
    }
    ```
2. Gatewayのインストール、ドメイン作成、起動
    ```bash
    # Actionパラメータを指定しない場合、デフォルトアクションはinstall-configure
    APIGateway -f gateway-props.json

    # 起動まで実施する場合は、install-configure-startを付ける
    APIGateway -f gateway-props.json -a install-configure-start

    # 起動だけしたい場合は、startを付ける
    APIGateway -f gateway-props.json -a start
    ```

## 論理Gatewayの作成

1. 論理Gatewayの作成（既にある論理GatewayへのNode追加であれば不要）
    ```bash
    APIGateway -f gateway-props.json -a creategateway
    ```
    このとき、以下の内容を尋ねてきます。
    - Gateway Nodeの管理ユーザー、パスワード(GatewayをホストするWebLogic Serverの管理者パスワード)
    - Gateway Managerのユーザー、パスワード (Gateway NodeおよびLogical Gatewayを管理するユーザー)

## 論理GatewayへのGateway Nodeの追加

1. Gateway Nodeの追加
    ```bash
    APIGateway -f gateway-props.json -a join
    ```
    このとき、以下の内容を尋ねてきます。
    - Gateway Nodeの管理ユーザー、パスワード
    - Gateway Managerのユーザー、パスワード (Gateway NodeおよびLogical Gatewayを管理するユーザー)
    - Gateway Runtime Userのユーザー、パスワード (GatewayとManagement Service間の通信時に使うユーザー)

## 論理Gatewayの作成、Nodeの追加時のユーザー、パスワードなど

認証方式はBasic認証です。
