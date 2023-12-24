---
title: code-serverの構築手順
tags:
  - Linux
  - OpenSSL
  - RHEL
  - VSCode
  - code-server
private: false
updated_at: '2023-10-18T13:58:24+09:00'
id: 2229bda056d0f1ef3434
organization_url_name: null
slide: false
ignorePublish: false
---
`code-server`をインストールした際の手順を忘れないうちにdumpしておきます。
`code-server`はブラウザからアクセスして利用できるVSCodeのサーバアプリケーションです（※MS公式のソフトウェアではありません）。

Linuxマシン上のファイルを直接VSCodeから触る手段としては、VSCodeの「Remote - SSH」拡張を使用する手段もありますが、
こちらはWebブラウザから利用できるという点で、より手軽と言えます。また、クライアント機の性能が貧弱でVSCodeの動作が重い、といった場合にも利用するメリットがあると思います。

- Github

https://github.com/coder/code-server

- Document

https://coder.com/docs/code-server/latest

# 動作環境

- Red Hat Enterprise Linux release 8.6
- code-server version 4.7.0

# 手順

1. インストールスクリプトを利用してインストールしました。  
以下のコマンドを実行するだけです。

    ```bash
    curl -fsSL https://code-server.dev/install.sh | sh
    ```

1. systemdサービスとして登録されるので、以下で起動できます。

    ```bash
    # マシン起動時に自動起動
    systemctl enable code-server@[実行ユーザ]
    # code-serverの起動
    systemctl start code-server@[実行ユーザ]
    ```

1. デフォルトでは、localhostからのアクセスしか許可しないので、設定を変更します。

    ```diff_yaml:~/.config/code-server/config.yaml
    - bind-addr: 127.0.0.1:8080
    + bind-addr: 0.0.0.0:8080
    auth: password
    password: [初期パスワード]
    cert: false
    ```

    ```bash
    # code-serverの再起動
    systemctl restart code-server@[実行ユーザ]
    ```

1. ブラウザから`http://[マシンのIPアドレス]:8080/`にアクセスすると、code-serverにアクセスできます。  
アクセス時に、↑の設定ファイルに記載されたパスワードの入力が求められます。
    ![code-server.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/3f369877-b53d-ad1c-4f16-4957cb611ebe.jpeg)

# HTTPSで接続する

上記までの手順で、HTTP接続でcode-serverを利用できるようになっています。
が、実はこのままだと**MarkdownやAsciidocのプレビュー**がブラウザ上でできないという問題があります。

理由として、MarkdownやAsciidocのプレビューには**Service Worker**の有効化が必要とのことであり、
通常、HTTPS接続でしか**Service Worker**を有効化できないためです。

- 該当issue

https://github.com/coder/code-server/issues/4421

そのため、HTTPSで接続できるように設定を変更します。  
今回は、ローカルのサーバにcode-serverを構築しているので、オレオレルート証明書とサーバ証明書を作成する方法で実施します。

---

**※2023/10/18修正**
証明書の作成方法をよりシンプルにしました。

---

## オレオレルート証明書を作成
### ディレクトリ・ファイルの作成

最初はホームディレクトリにいるものとします。
```bash
mkdir certs
cd certs
```

### オレオレルート証明書の作成

```bash
# 秘密鍵の作成
openssl ecparam -out ca.key -name prime256v1 -genkey
# 証明書署名要求ファイルの作成
openssl req -new -sha256 -key ca.key -out ca.csr
```

いくつか入力項目がありますが、今回は以下のようにしました。
- Country Name (2 letter code) [XX]:JP
- State or Province Name (full name) []:Tokyo
- Locality Name (eg, city) [Default City]:
- Organization Name (eg, company) [Default Company Ltd]:
- Organizational Unit Name (eg, section) []:
- Common Name (eg, your name or your server's hostname) []:codeserver-ca
- Email Address []:

```bash
# オレオレルート証明書の作成
# 有効期限は実質無期限（100年）としておきます
openssl x509 -req -sha256 -days 36500 -in ca.csr -signkey ca.key -out ca.crt
```

## code-server用の証明書の作成

```bash
# 秘密鍵の作成
openssl ecparam -out codeserver.key -name prime256v1 -genkey
# 証明書署名要求ファイルの作成
openssl req -new -sha256 -key codeserver.key -out codeserver.csr
```

入力項目は以下のようにしました。
- Country Name (2 letter code) [XX]:JP
- State or Province Name (full name) []:Tokyo
- Locality Name (eg, city) [Default City]:
- Organization Name (eg, company) [Default Company Ltd]:
- Organizational Unit Name (eg, section) []:
- Common Name (eg, your name or your server's hostname) []:codeserver
- Email Address []:

```bash
# subjectAltNameでサーバのプライベートIPを指定
cat <<EOF > subjectnames.txt
> subjectAltName = IP:xx.xx.xx.xx
> EOF
```
```bash
# サーバ証明書の作成
# 有効期限は実質無期限（100年）としておきます
openssl x509 -req -in codeserver.csr -CA  ca.crt -CAkey ca.key -CAcreateserial -out codeserver.crt -days 36500 -sha256 -extfile subjectnames.txt
# 証明書の内容表示
openssl x509 -in codeserver.crt -text -noout
```

## code-serverのオプションに証明書と秘密鍵を指定

code-serverのユニットファイルを編集します。


```diff_ini:/usr/lib/systemd/system/code-server\@.service
[Unit]
Description=code-server
After=network.target

[Service]
Type=exec
- ExecStart=/usr/bin/code-server
+ ExecStart=/usr/bin/code-server --cert <ホームディレクトリパス>/certs/codeserver.crt --cert-key <ホームディレクトリパス>/certs/codeserver.key
Restart=always
User=%i

[Install]
WantedBy=default.target
```

code-serverを再起動します。

```bash
systemctl daemon-reload
systemctl restart code-server@[実行ユーザ]
```

## 証明書をブラウザに登録

ブラウザに登録するため、証明書の拡張子を.derに変換します。
```bash
cd certs
openssl x509 -outform der -in codeserver.crt -out codeserver.der
openssl x509 -outform der -in ca.crt -out ca.der
```

ファイルをクライアント機に転送して、インストールします。
Windowsの場合は、以下の証明書ストアに配置してください。
- codeserver.der → 「信頼された発行元」
- ca.der → 「信頼されたルート証明機関」

## ブラウザからHTTPSでアクセス
`https://[マシンのIPアドレス]:8080/`でアクセスできるようになっています。
これで、Service Workerを有効化できるようになり、code-server上でもMarkdownやAsciidocのプレビューができるようになります。
![code-server2.JPG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/70acfd8b-14e1-47a7-987a-970d0a01bc97.jpeg)

以上。
