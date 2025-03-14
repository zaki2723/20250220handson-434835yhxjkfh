3-29 コマンドにエイリアスを付けよう。
コマンドにエイリアスを付けれる。エイリアスを付けることで、入力が楽になる。
$ git config --global alias.ci commit
$ git config --global alias.st status
$ git config --global alias.br branch
$ git config --global alias.co checkout
等

--globalを付けると、PC全体の設定になる。　~/.config/.git/config
--globalを付けないと、今自分がいるプロジェクトの下のproject/.git/configに設定が反映される。




3-30. バージョン管理しないファイルは無視しよう
.gitignoreにファイルを指定することで、そのファイルをgitのバージョン管理から外すことができる。

どういったものを指定する必要があるか。
パスワードなどの機密情報
winやmacで自動生成されるファイル

#から始まる行はコメント
指定したファイルを除外。　例:index.html
ルートディレクトリを指定。例:/root.html
ディレクトリ以下を除外。例:dir/ ディレクトリ名の後ろにスラッシュを入力。
/以外の文字列にマッチ｢*｣ 例:/*/*.css

touchコマンド→空ファイルを作成。











