# オレオレ証明書でのSSLの設定仕方

## オレオレ証明書の作成

1. mkcertのインストール
    - Windows
    ```
    winget install mkcert
    ```
    - macOS
    ``` 
    brew install mkcert 
    ```
1. ローカルCA（認証局）のインストール
    ```
    mkcert -install
    ```
1. IPアドレスを指定したサーバー証明書を作成し、任意の場所に配置してください
    ```
    mkcert localhost 127.0.0.1 192.168.X.X(SSLで接続したいサーバーのIPアドレス)
    ```

## nginxをインストール・SSLの設定

1. nginxをインストールします
    - Windows
        - [こちら](https://nginx.org/en/download.html)からnginx/Windowsをダウンロード
        - 解凍してドライブ直下に配置(例:C:\nginx-1.26.2\)
    - macOS
        - ターミナルで以下を実行
        ```
        brew install nginx
        ```
1. nginxをテスト起動
    - Windows ターミナルで配置したnginxのフォルダをカレントフォルダにして以下を実行
    ```
    start nginx
    ```
    - macOS ターミナルで以下を実行
    ```
    nginx
    ```
1. http://localhost にアクセスしてnginxが起動しているのを確認します
1. テスト起動したnginxを停止します
    - Windows
    ```
    taskkill /f /im nginx.exe
    ```
    - macOS
    ```
    nginx -s stop
    ```
1. nginxの設定をエディタで開きます
    - Windows ※ 上記のnginxフォルダからの相対パスです
    ```
    code .\conf\nginx.conf
    ```
    - macOS ※ AppleシリコンのMacの例です。IntelシリコンのMacの場合パスが異なると思います
    ```
    code /opt/homebrew/etc/nginx/nginx.conf
    ```
1. 以下のような編集し保存します。
    ```
    events {}
    http {
        # HTTPからHTTPSへのリダイレクト
        server {
            listen 80;
            return 301 https://$host$request_uri;
        }
        server {
            listen 443 ssl; # 標準ポートを設定することが外部からの接続を許可

            # 先ほどIPアドレス指定して生成したサーバー証明書を指定する
            ssl_certificate '/path/to/localhost+2.pem';
            ssl_certificate_key '/path/to/localhost+2-key.pem';;
  
            location / {
                proxy_pass http://localhost:XXXXX(リバースプロキシするURL);
            }
        }
    }
    ```
1. nginxを起動します。サーバーとしてアクセスできるように確認のポップアップが表示されると思いますので許可してください
    - Windows ※ カレントフォルダがnginxのフォルダであるこt
    ```
    start nginx
    ```
    - macOS
    ```
    nginx -t # このコマンドで設定ファイルを検証できます。問題なければ以降を実行しましょう
    nginx
    ```
1. スマホデバイスで https://192.168.X.X(SSLで接続したいサーバーのIPアドレス) にアクセスしてプライバシーが保護されていない旨の表示がされることを確認します(まだオレオレ証明書を信頼状態にしていないので)

## CAルート証明書を送る

1. ターミナルで以下を実行しCAルート証明書の場所を確認します ※ mkcertをインストール済みの想定です
    ```
    mkcert -CAROOT
    ```
1. ターミナルで以下を実行し、CAルート証明書をコピーし`.crt`の拡張子にします
    ```
    cd (出力されたパス)
    ls # rootCA.pemがあることの確認
    cp rootCA.pem rootCA_name.crt # nameの部分はコンピューター名などわかりやすいので良いです
    ```
1. 作成したcrtファイルをスマホデバイスにメールなどで送ります。

## スマホデバイスでcrtファイルをインストールする

### iOS

1. 受け取ったcrtファイルをダウンロードします。ブラウザからインストールする際はChromeなどでなく **Safari** 出ないと上手く行きません
1. 設定アプリ > 一般 > VPNとデバイス管理 よりダウンロードしたルート証明書をインストールします (確認したのはiOS15)
1. 設定アプリ > 一般 > 情報 > 証明書信頼設定 > インストールしたルート証明書を信頼状態にします (確認したのはiOS15)
1. スマホのブラウザで https://192.168.X.X(SSLで接続したいサーバーのIPアドレス) にアクセスしてプライバシーの警告が表示されることなくアクセスできればOKです。

### Android

1. 受け取ったcrtファイルをダウンロードします。
1. 設定アプリ > セキュリティとプライバシー > その他のセキュリティとプライバシー > 暗号化と認証情報 > 証明書のインストール > CA 証明書 > ダウンロードしたcrtファイルを選択してインストール (Android14のPixel 6aのケース)
1. スマホのブラウザで https://192.168.X.X(SSLで接続したいサーバーのIPアドレス) にアクセスしてプライバシーの警告が表示されることなくアクセスできればOKです。

