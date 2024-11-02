# 家庭内LANのVOICEVOXサーバーの構築手順

## 必要な機器

- 家庭内のLAN環境
- Windows又はmacOSのPC、或いはLinuxサーバー
    - Linuxサーバーでの検証はしていないのでWindows又はmacOSの手順を参考に実施してください

## 前提条件

- 登場するコマンドはVSCodeやCursorエディタをインストールしてターミナルで実行しています

## VOICEVOXサーバーを立てる

1. [こちら](https://github.com/VOICEVOX/voicevox_engine?tab=readme-ov-file#%E3%83%80%E3%82%A6%E3%83%B3%E3%83%AD%E3%83%BC%E3%83%89) のダウンロード先のリンクから自分の環境にあったVOICEVOXエンジンをダウンロードしてください
1. ダウンロードしたファイルを解凍します。
    - 7zipで圧縮されているので7zipで解凍できる環境がない人は、7zipの解凍するためのツールをダウンロードして解凍しましょう
        - 例) Windows [7zip](https://7-zip.org/download.html)
        - 例) macOS [@ntkgcj](https://qiita.com/ntkgcj)さんの[7z ファイルを解凍する 【mac】](https://qiita.com/ntkgcj/items/afe4863c40680d72a755)を参考にさせて頂きました🙇
1. 解凍したフォルダを任意の場所に配置します
1. ターミナルで解凍したフォルダをカレントフォルダにして以下を動作確認のために実行します
    - Windows
    ```
    .\run.exe --host localhost
    ```
    - macOS
    ```
    ./run --host localhost
    ```
1. http://localhost:50021 にアクセスしてVOICEVOXのサーバーが立ち上がっていることを確認できたら、ターミナルでCtrl + Cを押してVOICEVOXサーバーを終了します。
1. ブラウザとRobotVRMのサイト以外からアクセスできるのはセキュリティ上あまり良くないので、RobotVRMアプリのVOICEVOX設定の説明の記載の内容に従って以下を実行してVOICEVOXサーバーをサイトはRobotVRMのサイトからのみのアクセスを許容します
    - Windows
    ```
    .\run.exe --host localhost --allow_origin https://(RobotVRMアプリのVOICEVOXの設定に表示されているURL)
    ```
    - macOS
    ```
    ./run --host localhost --allow_origin https://(RobotVRMアプリのVOICEVOXの設定に表示されているURL)
    ```

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
    mkcert localhost 127.0.0.1 192.168.X.X(VOICEVOXサーバーのIPアドレス)
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
                proxy_pass http://localhost:50021;
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
1. スマホデバイスで https://192.168.X.X(VOICEVOXサーバーのIPアドレス) にアクセスしてプライバシーが保護されていない旨の表示がされることを確認します(まだオレオレ証明書を信頼状態にしていないので)

## スマホデバイスでオレオレ証明書を信頼状態にする

1. VOICEVOXサーバーのCAルート証明書をスマホデバイスに送って信頼状態にします。[スマホでオレオレ証明書を信頼状態にする](./smartphone_install_self_signed_cert.md) の手順参照
1. RobotVRMアプリのVOICEVOXの設定で"VOICEVOXサーバーURL"に https://192.168.x.x(VOICEVOXサーバーのIPアドレス) を入力して"ボイスを視聴する"ボタンを押して音声が出れば動作確認完了です。 