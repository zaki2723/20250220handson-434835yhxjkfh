## Gitの基本的なワークフロー

Gitを使用してファイルを作成し、編集、コミット、プッシュするまでの流れを示します。

### 1. ファイルの作成
```sh
# 新しいファイルを作成
$ touch filename.txt
```

### 2. ファイルの編集（VSCodeを使用）
```sh
# ファイルをVSCodeで開く
$ code filename.txt
```

### 3. 変更をステージング（git add）
```sh
# 変更をステージに追加
$ git add filename.txt
```

### 4. コミットを作成（git commit）
```sh
# コミットメッセージをつけて変更を記録
$ git commit -m "ファイルを追加または編集"
```

### 5. リモートリポジトリへプッシュ（git push）
```sh
# 変更をリモートリポジトリに送信
$ git push origin main
```

> **注意:** ブランチ名 `main` は適宜変更してください。