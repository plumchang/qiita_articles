---
title: Elasticsearch docsをビルドしてみた
tags:
  - Elasticsearch
  - ドキュメント
  - ビルド
private: false
updated_at: '2019-12-17T22:26:16+09:00'
id: 8bdb1577fe23b788f327
organization_url_name: fujitsu
slide: false
ignorePublish: false
---
# 動機
現在、Elasticsearchのドキュメントは日本語翻訳版がv5.4のものしかありません。最新リリースはv7.5。
[v5.4リファレンス](https://www.elastic.co/guide/jp/elasticsearch/reference/current/index.html)
[Elasticドキュメント](https://www.elastic.co/guide/index.html)
そこで、社内の成果発表ネタの意味も兼ねて、最新版の翻訳に貢献しようと思ったのが本記事の動機です。
ElasticsearchのドキュメントはAsciidocで書かれていて、以下にビルドツールも公開されていますので、
今回はそれを使ったビルドを試してみました。
https://github.com/elastic/docs

# 手順

### 1. ドキュメントビルドツールのソースのクローン

```
git clone https://github.com/elastic/docs.git
```
### 2. ビルド＆実行

```
cd docs/
./build_docs --doc README.asciidoc --open
```
※dockerをインストールしている必要があります。

ビルドが終わると、ブラウザが立ち上がり以下のようなページが開きます。
--doc にビルド対象を指定、また、--openオプションを指定することでビルド後に自動的にブラウザを立ち上げ、ページを表示します。
<img width="1194" alt="スクリーンショット 2019-12-17 19.58.16.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/782fe023-5a60-5fe4-3f82-cdb3e32ee100.png">

### 3. Elasticsearchソースのクローン
上でビルドしたのは本ビルドツールのチュートリアル用ページといった感じで、Elasticsearchやその他Elastic製品のドキュメントとは異なります。
次はElasticsearchのドキュメントをビルドしてみます。
ドキュメント全体のビルドなど後々のために、ドキュメントビルドツールのディレクトリ（docs）と同じ場所にクローンするのが良さそうです。

```
git clone https://github.com/elastic/elasticsearch.git
```

### 4. ビルド
今回はひとまず単一のページをビルドしてみます。

```
 ./build_docs --doc ../elasticsearch/docs/reference/getting-started.asciidoc --lenient --open
```

--lenient オプションを指定すると、別ファイルとの依存関係等によるエラーを無視して実行することができます。

以下のようなページが作成されました。
<img width="1195" alt="スクリーンショット 2019-12-17 22.02.29.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/d57c4fad-5f54-abda-bd7b-f53830ceaf4c.png">
<img width="1197" alt="スクリーンショット 2019-12-17 22.03.57.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/4301a155-904e-c370-3c85-cd4b41fe3d9a.png">

### 5. 翻訳
今回の主目的である翻訳をしてみました。
翻訳後のファイル(今回は getting-started.asciidoc)を差し替えて同様のビルドコマンドを実行。

```
 ./build_docs --doc ../elasticsearch/docs/reference/getting-started.asciidoc --lenient --open
```

翻訳後のページを作成できました。
<img width="1201" alt="スクリーンショット 2019-12-17 22.11.01.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/15864546-ef6b-2a5e-9b3b-d35dce1b020e.png">
<img width="1196" alt="スクリーンショット 2019-12-17 22.11.18.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/3394fd68-c3ac-375e-2312-a37f963761a3.png">

# おわりに
実のところ、Elasticsearchに上記翻訳のプルリクを投げたのですが、見事に弾かれました。下記参照。
https://github.com/elastic/elasticsearch/pull/49958
悲しいことに、現在ドキュメント翻訳のコミットは受け付けていない様子。。
ですが、勉強も兼ねてもう少し翻訳は続けていこうと思っています。

Advent Calenderの予定日を過ぎて投稿しているにも関わらずの薄い内容になってしまいました。:bow_tone1:
今回はページ単体のビルドを試しましたが、今後リファレンス全体のビルドも試すつもりです。

以上。
