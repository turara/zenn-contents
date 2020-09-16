---
title: "Go言語でGOPATHの外に作ったプロジェクトでサブパッケージをインポートする"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Go,初心者]
published: true
---

Go言語に入門中の身です。A Tour of Goをやり終えたところです。
手元のお試し用のコードが増えるにつれて、`main.go`ファイルに収まらなくなり、ファイルを分割したい欲求にかられ始めた頃、壁にぶつかりました。

# ローカルファイルのインポートができない！

ソースコードを以下のように配置して……

```shell
zenn-first-article
├── main.go
└── zenn
    └── zenn.go
```

```go
// zenn.go
package zenn

// Article describes a zenn article meta
type Article struct {
	Slug      string
	Title     string
	Emoji     string
	Type      string
	Topics    []string
	Published bool
}
```

`main.go`から`./zenn/zenn`のようにインポートすると……

```go
// main.go
package main

import (
	"fmt"
	"./zenn/zenn"
)

func main() {
	fmt.Println("Hello, Zenn!")
	item := zenn.Article{}
	fmt.Printf("%v\n", item)
}
```


以下のようなエラーが発生。。

```
main.go:6:2: local import "./zenn/zenn" in non-local package
exit status 1
Process exiting with code: 1
```

# GOPATHとは一体……？

`golang import local package`などと検索してみても、いまいちピッタリはまる情報がヒットしません。どうにか得られた情報はこんな感じ

- 環境変数GOPATHにセットしたパスの下にすべてのプロジェクトを配置するのが基本
- ローカルパッケージをインポートするときも（$GOPATH/src以下からの）絶対パスを使う
- ローカルパッケージのインポートでも相対パスは非推奨
- バージョン1.11から導入されたgo modulesという仕組みがある
- go modulesを使うと、プロジェクトをGOPATHの外に配置できる

## go modulesを試してみる

「GOPATH（何も設定しなければ、~/goになる）の下に全部のプロジェクトを置く？！なにそれ！」
「ローカルパッケージの相対パスでのインポートは非推奨？！」
などとびっくりしつつ……

go modulesというのを使えばよいのね。と思いながら使い方を調べると、GitHubなどにホスティングされている外部パッケージを管理する仕組みのようで、自分で作成したローカルパッケージをインポートする方法がいまいちわからないという事態に。。

たしかに`$GOPATH/src/zenn-first-article`にプロジェクトのディレクトリを配置すれば、

```go
// main.go
package main

import (
	"fmt"
	"zenn-first-article/zenn"
)
```

のようにインポートできるようにはなるのですが、npmやbundlerのようなパッケージ管理ツールに慣れた身にとっては違和感がすごいです。

# 解決策

なんだかもんにょりしてツイートしたところ、親切な方が`go modules`の使い方を教えてくださいました！

@[tweet](https://twitter.com/turara_engeneer/status/1305708002385616896)

@[tweet](https://twitter.com/nobonobo/status/1305725079980920833)

なるほど！`go modules`を初期化するときに指定したモジュール名がポイントになるようです。
教えていただいた通り、以下のように初期化を行うことで`module名/パッケージ名`で無事インポートできるようになりました。

```shell
$ pwd
/Users/turara/Dev/Go/zenn-first-article
$ go mod init zenn-first-article // module名を指定
go: creating new go.mod: module zenn-first-article
$ cat go.mod
module zenn-first-article // module名がgo.modに記載される

go 1.15
```

```go
package main

import (
	"fmt"
	"zenn-first-article/zenn" // module名/パッケージ名
)
```

のぼのぼ📡さん、ありがとうございました！

# 環境

go version go1.15.2 darwin/amd6


# おまけ：Goの公開用パッケージの命名について

パッケージの命名について、はじめて知った面白い視点だったので紹介します。

@[tweet](https://twitter.com/turara_engeneer/status/1305713824012484738)