---
title: "Zenn CLIの執筆準備をAppleScriptで自動化する"
emoji: "⚡️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Zenn,Mac,AppleScript]
published: true
---

# ワンクリックで執筆開始したい……！

[Zennを使ってみて感じたこと](https://zenn.dev/d0ne1s/articles/12c997e1858a6d3da4bc)

こちらの記事でダンさんが

>エディターで新規ウィンドウを開く → 記事のディレクトリに移動する → 記事執筆開始のコマンドを叩くを毎回やるのはちょっとしんどそう。automatorとかalfredとかでいい感じに自動化できると良いのだが、、

ということを言われていたので、AppleScriptを使って少しだけ自動化してみました。

ワンクリック……とまではいきませんが、SpotlightやAlfredeで数文字タイプすれば、執筆環境（ターミナル・エディタ・ブラウザ）を開くことができるようになりました。

# デモ

@[tweet](https://twitter.com/turara_engeneer/status/1306577317280448512)

SpotlightからAppleScriptのアプリケーションを起動して、ダイアログに記事のslugを入力すると、以下が実行されます。

[^1]: GIFには1.5MBの制限があるようで、小さいGIFしか

1. ターミナルが開いて新規記事を作成
2. 記事プレビュー用のコマンドを実行
3. Safariを開いてプレビューページを表示
4. VSCodeで記事管理用のリポジトリと新規作成した記事マークダウンファイルを開く

# 事前準備

公式の手順にしたがって、GitHub連携とZenn CLIのインストールを済ませます。

- [GitHubリポジトリでZennのコンテンツを管理する](https://zenn.dev/zenn/articles/connect-to-github)
- [Zenn CLIをインストールする](https://zenn.dev/zenn/articles/install-zenn-cli)
- [4ステップでGithubからZennに記事を公開してみる(トラブルシューティング付き)](https://zenn.dev/ohbashunsuke/articles/20200917001-deploy-with-github)


# ソースコード

サンプルのスクリプトファイルは[こちら](https://github.com/turara/zenn-contents/blob/master/samples/zenn-new-article-with-applescript/zenn-new-article-sample.scpt)からダウンロードできます。

以下では、ハンドラーごとに簡単に解説していきます。

## ダイアログを表示して入力を受け取る

ダイアログから記事のslugを受け取ります。

```applescript
on getInput()
	display dialog "Input slug of new article" with title "Zenn new article" default answer ""
	return text returned of result
end getInput
```

## ターミナルで実行するコマンドを組み立てる

ターミナルを開いたあと、下記を実行するためのコマンドを組み立てます。

- 新規記事の作成
- エディタをオープン
- Zenn CLIのプレビューの準備

VSCode以外のエディタを使用している方は、`code ...`の部分をお使いのエディタに合わせて変えてみてください。

```applescript
# 引数にリポジトリのパスとslugを受け取る
on buildScript(zennRepo, slug)
  # 一旦リストに流し込む
	set scriptList to {}

	set end of scriptList to ("cd " & zennRepo)
	set end of scriptList to " && "
	set end of scriptList to "npx zenn new:article"

  # slugが空の時は飛ばす
	if not (slug = "") then
		set end of scriptList to (" --slug " & slug)
	end if

	set end of scriptList to " && "

	set end of scriptList to "code ."

	if not (slug = "") then
		set end of scriptList to " && "
		set end of scriptList to ("code articles/" & slug & ".md")
	end if

	set end of scriptList to " && "
	set end of scriptList to "npx zenn preview"

  # リストを文字列にして返す
	return scriptList as string
end buildScript
```

完成したコマンドはこんな感じ。

```shell
cd ~/Zenn/zenn-contents && npx zenn new:article --slug zenn-new-article && code . && code articles/zenn-new-article.md && npx zenn preview
```


## Safariのwindowをスクリーンの左半分に合わせる

これは必須ではないですが、やっておくと便利です。

もちろんエディタのwindowをスクリーンの右半分に配置したかったのですが、残念ながらVSCodeはAppleScriptからwindowの位置やサイズを変更できないようです。なにかやり方をご存知の方がいらっしゃったら是非教えてください！

```applescript
on setSafariBoundsToHalfScreen()
	tell application "Finder"
		set b to bounds of window of desktop
		set w to third item of b
		set h to last item in b
	end tell

	tell application "Safari"
		activate
		set bounds of first window to {0, 0, w / 2, h}
	end tell
end setSafariBoundsToHalfScreen
```

## ここまでのハンドラを組み合わせる

ここまでで紹介したハンドラ（関数）を呼び出していきます。

zennRepoには、Zennの記事管理用のリポジトリのパスを指定してください。

```applescript
# 自分の環境に合わせてリポジトリのパスを設定
set zennRepo to "~/Zenn/zenn-contents"

# slugの入力を受け付けて、ターミナルで実行するコマンドの文字列を作成する
set slug to getInput()
set scriptString to buildScript(zennRepo, slug)

# ターミナルを開いてコマンドを実行する
tell application "Terminal"
	do script scriptString
end tell

# Zenn CLIの npx zenn previewが起動するのを待つために2秒ストップ
delay 2

# Safariを開いて、windowを調整する
setSafariBoundsToHalfScreen()

# Safariでプレビュー用のページを表示する
tell application "Safari"
	activate
	open location "http://localhost:8000"
end tell
```

# スクリプトをアプリケーションとして保存する

以上のコードをご自分の環境に合わせて修正したあと、「スクリプトエディタ」アプリ内に貼り付け、実行してみてください。

スクリプトエディタを初めて使用するときには、コンピュータの制御に関する許可を求められる場合があります。その場合は、セキュリティとプライバシー > アクセシビリティに、スクリプトエディタを追加します。

![](https://storage.googleapis.com/zenn-user-upload/fpx9ikq1mtiafk9r9eje4oqy4tld)

スクリプトエディタ上での実行がうまくいったら、スクリプトを保存します。

このとき、アプリケーションとして保存することで、ダブルクリックやスポットライトから起動することができるようになります。

![](https://storage.googleapis.com/zenn-user-upload/jb42m455k66ys3ba4wxfllz4zzl6)

ダブルクリックやスポットライトから起動すると、初回は以下のような許可を求めるダイアログがいくつか出てくるので、OKを押していけば大丈夫です。

![](https://storage.googleapis.com/zenn-user-upload/oty60rega7qvtsd8d4hwf2he3v3w)

誤って「許可しない」を押してしまった場合は、 セキュリティとプライバシー > オートメーションというところから設定の変更ができます。

![](https://storage.googleapis.com/zenn-user-upload/x6eb315fgmp7fsi2btw517e0ow45)

# おわりに

ZennはGithubで記事の変更履歴が管理できたり、自分の好きなエディタで執筆ができる点がとてもいいですよね。

この記事がZennでの執筆の助けになれば幸いです。

Happy Zenn writing!!

# 参考

[【Mac】Apple Scriptのdisplay dialogの練習がてらじゃんけんアプリを作ってみる](https://qiita.com/soh19/items/200f5881ed1fc87665dd)
[鳶嶋工房 / AppleScript / Tips / 文字列の追加・削除](http://tonbi.jp/AppleScript/Tips/String/AddDelete.html)