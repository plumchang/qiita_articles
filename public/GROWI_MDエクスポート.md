---
title: GROWIでMarkdownファイルを一括エクスポートする
tags:
  - Python
  - Growi
private: false
updated_at: '2024-06-15T13:30:11+09:00'
id: ef078bdc9e6a18e3f2f4
organization_url_name: null
slide: false
ignorePublish: false
---

# 概要

[GROWI](https://growi.org/ja/)は、オープンソースのWiki開発ツールです。  
GROWIには、今のところ（2024/06時点）、記事を一括でMarkdownファイルにエクスポートする機能がありません。  
ただし、[データアーカイブ機能](https://docs.growi.org/ja/admin-guide/management-cookbook/export.html)を使うと、MongoDB Collectionsをzip形式でエクスポートできますので、本データを元にMarkdownファイルや添付ファイルを復元できます。

:::note
GROWIの以下のような機能群に魅力があり、以前、自分が所属するチームで利用していました。
- オープンソースのため、ローカル環境でホストできる
- Markdownで記事を書ける
- draw.ioと連携でき、記事内にダイアグラムを埋め込める
- HackMDと連携でき、複数人で記事の同時編集ができる

※ある時、社内のセキュリティ施策の関係で、GROWIから別ツールへ移行する必要が出たため、本記事に記載する方法でファイルにエクスポートしました。
:::


**前提**

動作確認済みのGROWIバージョン：v5.1.0

※記事執筆時点（2024/06）の最新バージョンは、v7.0.10でした。  

# Markdownファイル・添付ファイルのエクスポート

**1. [データアーカイブ機能](https://docs.growi.org/ja/admin-guide/management-cookbook/export.html)でzipファイルをエクスポートします。**  
エクスポート対象に以下を含めます。
- Pages
- Revisions
- Attachments
- AttachmentFiles.Files
- AttachmentFiles.Chunks

**2. zipを展開します。**

**3. Markdownファイルをエクスポートします。**

展開したzip内の`pages.json`、`revisons.json`、`attachments.json`を、以下のPythonスクリプトと同一ディレクトリに配置し、スクリプトを実行します。  
`./output`ディレクトリ以下に、Markdownファイルがエクスポートされます。  
- ファイル名は、「記事タイトル（スペースは_に変換）」+「.md」とします。
- 添付ファイルへのリンクは、「/.attachments/」+「添付ファイル名」に置換します。

```python:make_md_from_json.py
import json
import os
import re

# JSONファイルを読み込む
with open('pages.json', 'r') as f:
    pages = json.load(f)

with open('revisions.json', 'r') as f:
    revisions = json.load(f)

with open('attachments.json', 'r') as f:
    attachments = json.load(f)

# ページIDをキーとした辞書を作成
pages_dict = {page['_id']: page['path'] for page in pages}
revisions_dict = {revision['pageId']: revision['body'] for revision in revisions}
attachments_dict = {attachment['_id']: attachment['fileName'] for attachment in attachments}

# 各ページについてマークダウンファイルを出力
for page_id, path in pages_dict.items():
    if page_id in revisions_dict:
        # パスが'/'の場合は'index'に置き換える
        if path == '/':
            path = '/index'
        # ファイルパスを作成
        path = path.replace(' ', '_')  # スペースを_に置き換える
        file_path = './output' + path + '.md'
        # ディレクトリが存在しなければ作成
        os.makedirs(os.path.dirname(file_path), exist_ok=True)
        # 添付ファイルのリンクを書き換える
        body = revisions_dict[page_id]
        for attachment_id, filename in attachments_dict.items():
            body = re.sub(r'\(/attachment/' + attachment_id + r'\)', r'(/.attachments/' + filename + r')', body)
        # ファイルを書き込む
        with open(file_path, 'w') as f:
            f.write(body)
```

**4. 添付ファイルをエクスポートします。**

展開したzip内の`attachments.json`、`attachmentFiles.files.json`、`attachmentFiles.chunks.json`を、以下のPythonスクリプトと同一ディレクトリに配置し、スクリプトを実行します。  
`./output/.attachments`ディレクトリ以下に、添付ファイルがエクスポートされます。

```python:make_attachments_from_json.py
import json
import os
from collections import defaultdict
import base64

# JSONファイルを読み込む
with open('attachments.json', 'r') as f:
    attachments = json.load(f)

with open('attachmentFiles.files.json', 'r') as f:
    files = json.load(f)

with open('attachmentFiles.chunks.json', 'r') as f:
    chunks = json.load(f)

# ファイル名をキーとした辞書を作成
attachments_dict = {attachment['fileName']: attachment['_id'] for attachment in attachments}
files_dict = {file['filename']: file['_id'] for file in files}

# ファイルIDをキーとした辞書を作成し、チャンクを番号順に格納
chunks_dict = defaultdict(list)
for chunk in chunks:
    chunks_dict[chunk['files_id']].append((chunk['n'], chunk['data']))
for file_id in chunks_dict.keys():
    chunks_dict[file_id].sort()

# 各添付ファイルについてファイルを出力
for filename, attachment_id in attachments_dict.items():
    if filename in files_dict and files_dict[filename] in chunks_dict:
        # ファイルパスを作成
        file_path = './output/.attachments/' + filename
        # ディレクトリが存在しなければ作成
        os.makedirs(os.path.dirname(file_path), exist_ok=True)
        # ファイルを書き込む
        with open(file_path, 'wb') as f:
            for _, data in chunks_dict[files_dict[filename]]:
                f.write(base64.b64decode(data))
```
