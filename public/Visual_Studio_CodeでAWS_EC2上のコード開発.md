---
title: Visual Studio Code でAWS EC2上のコード開発
tags:
  - AWS
  - SSH
  - EC2
  - proxy
  - VisualStudioCode
private: false
updated_at: '2019-12-26T11:33:06+09:00'
id: 2cec583b09c25d8e943e
organization_url_name: fujitsu
slide: false
ignorePublish: false
---
# 概要
社内からAWSの踏み台EC2インスタンスを経由しデプロイ先のEC2インスタンスに接続するような環境で開発を行っています。社内の開発マシン上のVisual Studio Code でデプロイ先のEC2インスタンスを直接触ることができたので、備忘録として残します。

**環境**
　開発マシン（社内のプロキシ環境下）：Windows10
　踏み台EC2インスタンス：Linux
　デプロイ先EC2インスタンス：Linux

# 手順
　
<ol><li> 開発マシンにVisual Studio Code のインストール
    https://code.visualstudio.com/download
</li><li> Remote - Development(SSH) プラグインのインストール
    https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack
</li><li> 開発マシンでユーザー環境変数の設定
    HTTP_PROXY_USER 社内プロキシの認証ユーザー名
    HTTP_PROXY_PASSWORD 社内プロキシの認証パスワード
</li><li> .ssh/configを開発マシンの任意の場所に作成

```
Host 踏み台EC2インスタンスの名前（任意）
    HostName インスタンスのIPアドレス
    User ec2-user
    Port 22
    IdentityFile パス/pemキー（開発マシンのどこかに置いておく）
    # 社内プロキシを経由
    ProxyCommand C:/Program Files/Git/mingw64/bin/connect.exe -H proxy.example.com:port %h %p

Host デプロイ先EC2インスタンスの名前（任意）
    HostName インスタンスのIPアドレス
    User ec2-user
    Port 22
    IdentityFile パス/pemキー（開発マシンのどこかに置いておく）
    ProxyCommand ssh -W %h:%p 踏み台インスタンスの名前
```  
</li><li> VSCode から接続
作成した.ssh/configを選択して接続する。途中で、"Continue"のポップアップが出たら、Continueを選択して進める。
    ![無題.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/0c45426c-1b98-f36f-f7b5-e007d736b906.png)

接続できた。

![無題.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/320b29e4-d81b-6008-cb94-c5a5b4807811.png)

これでデプロイ先インスタンスのファイルを直接編集できるようになりました。

</li></ol>

## 追記（2019/12/26）
上記設定では、ec2-userの権限があるファイルやディレクトリのみ操作が可能です。
なので、root権限が必要なファイルの編集、作成、移動等はできません。（ダウンロードは権限がなくても可能。）
デプロイ先EC2インスタンスにrootで接続するには、以下のようにします。（セキュリティ面で問題がある場合もあるので自己責任で。）

1. ターミナルからデプロイ先EC2インスタンスにssh接続
2. sudo su - でrootユーザーに切り替え
3. authorized_keysのコピー  
  cp /home/ec2-user/.ssh/authorized_keys /root/.ssh/authorized_keys  
  #元のキーはバックアップしておく
4. service sshd reload でsshd再起動
5. 開発マシンの.ssh/configを編集  

```
Host デプロイ先EC2インスタンスの名前（任意）  
    HostName インスタンスのIPアドレス   
    User root ←　ここ
```

これで、root権限が必要なファイルもVisual Studio Code上から操作可能になります。

### 参考
https://qiita.com/songxun/items/7f1e3ddcc87bb74e80f1
https://norm-nois.com/blog/archives/2958
