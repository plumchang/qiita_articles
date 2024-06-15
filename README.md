# qiita_articles

## 投稿手順メモ

1. Git pull

    ```bash
    git pull origin
    ```

2. Qiita同期

    ```bash
    npx qiita pull --force
    ```

3. 記事作成

    ```bash
    npx qiita new 記事のファイルのベース名
    ```

4. 記事執筆

5. プレビュー

    ```bash
    npx qiita preview
    ```

6. 投稿

    ```bash
    git add .
    git commit
    git push origin
    ```