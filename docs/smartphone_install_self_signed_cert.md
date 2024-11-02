# スマホでオレオレ証明書を信頼状態にする

## 前提条件

- 登場するコマンドはVSCodeやCursorエディタをインストールしてターミナルで実行しています

## 対象のコンピューターからCAルート証明書を送る

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
1. スマホのブラウザで https://192.168.X.X(CAルート証明書のコンピューターのIPアドレス) にアクセスしてプライバシーの警告が表示されることなくアクセスできればOKです。

### Android

1. 受け取ったcrtファイルをダウンロードします。
1. 設定アプリ > セキュリティとプライバシー > その他のセキュリティとプライバシー > 暗号化と認証情報 > 証明書のインストール > CA 証明書 > ダウンロードしたcrtファイルを選択してインストール (Android14のPixel 6aのケース)
1. スマホのブラウザで https://192.168.X.X(CAルート証明書のコンピューターのIPアドレス) にアクセスしてプライバシーの警告が表示されることなくアクセスできればOKです。