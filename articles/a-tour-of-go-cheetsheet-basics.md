---
title: "Go言語のチュートリアル A Tour of Go のチートシート その1 - Basics"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Go, 初心者]
published: true
---

Go言語入門の定番、チュートリアル [A Tour of Go](https://go-tour-jp.appspot.com/list) をやったので、復習を兼ねて内容をまとめておきます。

また、チュートリアル中、何度も前のページに戻って確認しなければならなかったので、これからやろうという方にはチートシートとしてお使いいただけます。

チュートリアルをやる代わりに、さらっと眺める用途にもどうぞ。

※ 本記事は[Qiitaの記事](https://qiita.com/turara/items/0a44c06e8915a28d94ab)を移植したものです。

# Packages, variables and functions

## パッケージとインポート・エクスポート

- プログラムはmainパッケージから実行される。
- importでパッケージを読み込む。
- 大文字で始まる名前はエクスポートされる。

```go
// ファイルの冒頭にpackage名を宣言する
package main

// パッケージのインポートをグループ化できる
// factored import statementと呼ばれる
import (
    "fmt"
    "math"
)

func main() {
    // Printf / Sqrt はそれぞれのパッケージからエクスポートされている
    fmt.Printf("Sqrt(9) is %g\n", math.Sqrt(9))

    // 小文字で始まる名前は外部に公開されないので、エラーが発生する
    fmt.Println(math.pi)
}
```

## 関数

- 関数はfuncで定義する。
- 引数の型は変数の後ろに書く。

```go
// ２つのintを受け取り、intを返す関数
func add(x int, y int) int {
    return x + y
}

// 型が同じ場合は、最後の型以外は省略できる
func addThree(x, y, z int) int {
    return x + y + z
}

// 複数の戻り値を返すことができる
func swap(x, y string) (string, string) {
    return y, x
}

// 戻り値に名前をつけることができる
func div(x, y int) (result int) {
    // result変数はこの時点で定義済み
    result = x / y
    // この場合、何も書かずにreturnできる（naked return）
    return
}
```

## 変数・定数

変数の宣言はパッケージまたは関数の中で利用できる

```go
// 変数はvarで宣言する
var x, y, z int
var s string

// initializerで初期化可能、型を省略できる
var i, j, b = 1, 2, true

// 定数はconstで宣言する
const K int = 10000
const Truth = true

func main() {
    // 関数内限定でvarの代わりに:=を使うことができる
    foo := "bar"
    c := 2 + 0.5i // 複素数型
}
```

## 基本型（組み込み型）

- bool
- string
- int int8 int16 int32 int64
- uint uint8 uint16 uint32 uint64
- byte（uint8の別名）
- rune（int32の別名）
- float32 float64
- complex64 complex128 （複素数型）

### ゼロ値

初期値なしで宣言すると、ゼロ値が入る

- 数値型: 0
- bool: false
- string: ""（空文字列）

### 型変換

明示的な型変換が必要

```go
var i int = 60
var f float64 = float64(i)
var u uint = uint(i)
```

# Flow control statements

## forループ

- C言語などと同じで、初期化; 条件式; 後処理を記述する
- 多言語のwhileループの代わりにGoではforを利用する

```go
sum := 0

// カッコは不要
for i := 0; i < 10; i++ {
    sum += i
}

// 初期化と後処理は省略可能
for ;sum < 1000; {
    sum += sum
}

// セミコロンも省略可能 -> whileループ
for sum < 10000 {
    sum += sum
}

// 条件式も省略可能 -> 無限ループ
for {
    if sum > 100000 {
        break
    }
}
```

## if

- forループと同じでカッコは不要
- 条件判定用の変数を初期化することができる

```go
// カッコ不要
if i > 3 {
    return i
}

// xのスコープはif-elseブロック内のみ
if x := i * j + 3; x < 100 {
    return x
} else {
    return x - 100
}

// compile error
return x + 100
```

## switch

- ひとつのcaseが実行されたら、残りのcaseは実行されないので、caseごとのbreakは不要
- ifと同じように条件判定用の変数を初期化することができる
- caseは定数である必要はない

```go
// os変数を初期化
switch os := runtime.GOOS; os {
case "darwin":
    fmt.Println("OS X.")
case "linux":
    fmt.Println("Linux.")
default:
    // freebsd, openbsd, plan9, windows...
    fmt.Printf("%s.\n", os)
}

// 条件のないswitchも可能（switch trueと同じ）
t := time.Now()
switch {
case t.Hour() < 12:
    fmt.Println("Good morning!")
case t.Hour() < 17:
    fmt.Println("Good afternoon.")
default:
    fmt.Println("Good evening.")
}
```

## defer

- deferに関数の実行を渡すと、関数の終わりまで評価が遅延される
- 最後に渡したdeferから順番に実行される（last-in-first-out）

```go
func hello() {
    defer fmt.Println("world")
    fmt.Println("hello")
}

func countDown() {
    for i := 0; i < 10; i++ {
        defer fmt.Print(i)
    }
    // 9876543210と表示される
}
```

# More types: structs, slices, and maps

## ポインタ

- ポインタは値のメモリアドレスを指す
- ポインタのゼロ値は`nil`

```go
// int型のポインタは *int型
var p *int

// &オペレータでポインタを引き出す
var i int = 50
p = &I

// *オペレータでポインタの指す変数を示す
j := *p // pからiの値を読みだす
*p = 40 // pを通してiに値を代入する
```

## struct

- struct（構造体）はフィールド（field）の集まり

```go
type Vertex struct {
    X int
    Y int
}

func test() {
    // Vertexリテラル: field順に値を渡す
    v := Vertex{1, 2}
    v.X = 5

    // ポインタを使う
    p := &v
    // ポインタを通してフィールドにアクセスする
    (*p).X = 6
    // 省略してp.Xとも書ける
    p.X = 6
}

// 初期化時にフィールドの名前を使うことができる
var (
    v1 = Vertex{Y: 2, X: 1} // フィールドの順番は関係なし
    v2 = Vertex{Y: 2}       // Xはゼロ値（0）をとる
    v3 = Vertex{}           // X: 0, Y: 0
    p = &Vertex{1, 2}       // *Vertex型
)
```

## array

- 配列は長さを指定して宣言する
- 配列の長さは型の一部分なので、配列のサイズを変えることはできない

```go
// int型の要素を持つ長さ10の配列の宣言
var a [10]int

// リテラル
primes := [6]int{2, 3, 5, 7, 11, 13}

var s [2]string
s[0] = "Hello"
s[1] = "World"
```

## slices

- 配列が柔軟になった可変長の型
- `a[1:4]`などの記法で配列からスライスを作ることができる
- スライスはデータを格納せず、元の配列の部分列を指し示す

```go
primes := [6]int{2, 3, 5, 7, 11, 13}

// int型の要素を持つスライスの宣言
var s1 []int

// primesの要素のうち、1番目から3番目の要素を含むスライス
s1 = primes[1:4] // {3, 5, 7}

// 下限と上限の値を省略するとデフォルトの値が使用される
// デフォルト値は下限: 0, 上限: スライスの長さ
// 以下は等価
primes[0:6]
primes[:6]
primes[0:]
primes[:]

// 同じ元となる配列を共有している場合、変更が反映される
s2 := primes[1:3] // {3, 5}
s2[0] = 59 // -> primes[1]とs1[0]も59になる

// リテラル
// [3]{true, true, false}の配列を作成して、それを参照する
b := []{true, true, false}
```

### 長さ（length）と容量（capacity）

- 長さ：スライスに含まれる要素数、len関数で取得可能
- 容量：スライスの最初の要素から数えた元となる配列の要素数、cap関数で取得可能

```go
a := [6]int{0, 1, 2, 3, 4, 5}

s1 := a[0:0]  // len(s1) -> 0, cap(s1) -> 6
s2 := a[0:4]  // len(s2) -> 4, cap(s2) -> 6
s3 := a[2:6]  // len(s3) -> 2, cap(s3) -> 4

// 容量が十分な場合、スライスを再度スライスすると長さが伸びる
s4 := s1[0:4] // len(s4) -> 4, cap(s4) -> 6

// スライスのゼロ値はnil、長さと容量は0になる
var s5 []int  // len(s5) -> 0, cap(s5) -> 0
```

### make関数

組み込みのmake関数を使用してスライスを作成することができる

```go
// 型と長さを引数にとる
a := make([]int, 2)    // len(a) -> 2, cap(a) -> 2, a: [0 0]
// 容量も引数に渡すことができる
b := make([]int, 0, 3) // len(b) -> 0, cap(b) -> 3, b: []
```

### 多次元スライス

スライスは任意の型を含むことができるので、スライスを含むスライスを作ることができる

```go
board := [][]int{
    []int{1, 2, 3},
    []int{4, 5, 6},
    []int{7, 8, 9},
}
```

### append関数

- append関数: `func append(s []T, vs ...T) []T`
- スライスへ新しい要素を追加することができる
- appendしたときに、**スライスの参照先の配列の容量を超えてしまう場合は、戻り値となるスライスはより大きいサイズの配列を作って参照する**

```go
var s[]int             // len(s) -> 0, cap(s) -> 0, s: []
s = append(s, 0)       // len(s) -> 1, cap(s) -> 1, s: [0]
s = append(s, 1)       // len(s) -> 2, cap(s) -> 2, s: [0 1]
s = append(s, 2, 3, 4) // len(s) -> 5, cap(s) -> 6, s: [0 1 2 3 4]
```

## map

- キーと値を関連づける
- ゼロ値はnil

```go
var m map[string]int
m = make(map[string]int)
m["likes"] = 10
m["stocks"] = 5

// リテラル
m = map[string]int{
    "likes": 10,
    "stocks": 5,
}

// 追加・更新
m["foo"] = 1

// 取得
likes := m["likes]

// 削除
delete(m, "likes")

// 存在確認、二つ目の戻り値は、値が存在すればtrue、存在しなければfalse
stocks, ok := m["stocks"]
```

## range

- slicesやmapをループするときに使うことができる

```go
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

// ループごとにindexとvalueを返してくれる
for i, v := range pow {
    fmt.Printf("2**%d = %d\n", i, v)
}

// 不要な場合はアンダースコアで捨てることができる
for i, _ := range pow
for _, value := range pow

// indexだけが欲しい場合は２つ目を省略しても良い
for i := range pow
```

## クロージャ

- 関数はクロージャ
- 関数は自身の外部から変数を参照できる

```go
func adder() func(int) int {
    sum := 0
    // sumへの参照を保持した無名関数
    return func(x int) int {
        sum += x
        return sum
    }
}
```

# 続きは次回！

次回の内容は……

- Methods and interfaces
- Concurrency

# 参考

[A Tour of Go](https://go-tour-jp.appspot.com/list)
