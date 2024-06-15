---
title: ケース別"git reset"
tags:
  - Git
private: false
updated_at: '2024-06-15T22:11:15+09:00'
id: f4d42d5431d5a7c24872
organization_url_name: null
slide: false
ignorePublish: false
---
# 1. リモートにプッシュしていない最新のコミットが1つあり、そのコミットを取り消したい

![01_gitreset_local1.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/ab45bcd7-32a9-1b12-c6d2-e370c6143dd1.png)

```sh
# 現在の変更はステージングしたまま
git reset --soft HEAD~1
# 現在の変更のステージングは外す
git reset --mixed HEAD~1
# 現在の変更は破棄する
git reset --hard HEAD~1
```

# 2. リモートにプッシュしていない最新のコミットが2つあり、そのうちの最後のコミットだけ取り消したい
最新のコミットを取り消すので、1.と同じ

![02_gitreset_local1.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/8fe25c91-230a-c8f2-4be2-b4fa4b89d2c0.png)

```sh
# 現在の変更はステージングしたまま
git reset --soft HEAD~1
# 現在の変更のステージングは外す
git reset --mixed HEAD~1
# 現在の変更は破棄する
git reset --hard HEAD~1
```

# 3. リモートにプッシュしていない最新のコミットが2つあり、2つともコミットを取り消したい

![03_gitreset_local2.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/74b65a5f-b79a-d25d-5fe2-744fc0d52945.png)

```sh
# 現在の変更はステージングしたまま
git reset --soft HEAD~2
# 現在の変更のステージングは外す
git reset --mixed HEAD~2
# 現在の変更は破棄する
git reset --hard HEAD~2
```

# 4. リモートにプッシュ済みの最新のコミットを取り消したい

![04_gitreset_remote1.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/9407cd3b-1374-1436-7d4b-5f54cce5e698.png)

```sh
# ローカルの最新のコミットを取り消す（現在の変更は破棄する）
git reset --hard HEAD~1
# リモートにフォースプッシュ
git push -f origin
```

:::note warn
リモートの履歴を改変するので、共同作業者がいる場合は事前にコミュケーションを取るなど、注意が必要
:::

**※コミット履歴を残す場合は`git revert`**
最新のコミットを受け消す、新しいコミット履歴が作成される。

![04_gitreset_revert.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/a3803c5a-64d5-ebd6-c1fe-280edae49348.png)

```sh
git revert HEAD
git push origin
```

# 5. ローカルのコミット履歴に関わらず、リモートの最新コミットの状態に戻したい

![05_gitreset_remote.drawio.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/432480/9d21e970-5985-b733-9767-3e4628c516e8.png)

```sh
# 指定したリモートブランチの最新コミットに戻す（現在の変更は破棄する）
git reset --hard origin/ブランチ名
```
