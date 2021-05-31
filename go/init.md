# Go 言語導入編

Go言語（golang） を使う予定になってしまったので、
それの準備について記載する

## 環境構築編
### IDE 準備

IDEについて
JetBrains 製のIDEを開発時につかっているので、
それでできないか検討

有名な話ですが、Go用の IDE も有償で当然あります。
https://www.jetbrains.com/ja-jp/go/

Intellij IDEA でもプラグインがあるらしい
https://pleiades.io/help/idea/go-plugin.html

### SDK

Go の開発には Go SDK が必要になるようです。
コンパイルしたり実行したりするのに必要で、  
IDE で開発す場合もダウンロードが必要です。
※どうやら Intellij の場合はプロジェクト作成時にIDE上でダウンロード可能だったようです

以下のサイトからダウンロードします。
https://golang.org/dl/

Windows、Mac、Linux用にパッケージがそろっているので、
必要なものをダウンロードします。  
今回は Windows 用の msi をダウンロードし、実行します。

`C:\Program Files\Go` にダウンロードしました。

### IDE からプロジェクト作成

Intellij から New Projects で Go を選択  
GOROOTでSDKを指定します。
インストールしてしまった `C:\Program Files\Go` を指定。

`go-sandbox` でプロジェクトを作成

次にコードを書いてコンパイル・ビルドして実行するところまでやりたい。
お作法は全くわからないですが、ひとまず他で私がやっているように `src/main` ディレクトリを作成し、  
その下に実行用のファイルを作成します。  

`main.go` という名前のファイルを作成しました。  
[New] -> [Go File] から [Sample Application] を選択  

書いたコードは以下
```go
package main

import "fmt"

func main() {
	fmt.Printf("Hello world!")
}
```

この状態で以下をおこない、実行します
- main メソッドにある IDE 上の「▶」を押すか
- [Run/Debug Configurations] で [Go Build]　を選択、「go build main.go」 を作成
- Terminal から 「go run src\main\main.go」 を実行   
  （Project Root ディレクトリ上から実行しているものとする）
- Terminal から 「go build src\main\main.go」 を実行  
  そうすると「main.exe」という実行ファイルが作成されるので、「main.exe」として実行する
  （Project Root ディレクトリ上から実行しているものとする）

無事に以下のように表示される(IDE から実行した場合の例)
```go
GOROOT=C:\Program Files\Go #gosetup
GOPATH=null #gosetup
"C:\Program Files\Go\bin\go.exe" build -o C:\Users\Ore\AppData\Local\Temp\___go_build_main_go.exe C:/Users/Ore/work/go/go-sandbox/src/main/main.go #gosetup
C:\Users\Ore\AppData\Local\Temp\___go_build_main_go.exe #gosetup
Hello world!
Process finished with exit code 0
```

### パッケージ管理ツールについて

Go のパッケージ管理ツールについて.
たとえば、Java の gradle, maven や Scala の sbt みたいなのを想像している  
こういったものはなく、go コマンドに組み込まれたサブコマンドをつかって管理するらしい  

2021 年現在(go 1.12 以降)では `go mod` を使うのが一般で、  
一部は慣例で `vendoring` を使っているものがあるらしい

ファイルのある場所で `go mod` コマンドを実行する.  
```shell
cd work\
go mod
```

## ライブラリ
### フレームワーク

Web API で Hello World したい  
フレームワークは何がいいかわからなかったので何があるかしらべるとことから

以下を参考にしました
https://qiita.com/yumin/items/5de33b068ead564ebcbf

[Gin](https://github.com/gin-gonic/gin) というのが一番使われてそう？
だいたいだと以下
- Gin
- Echo
- Martini
- Revel

### フレームワークのダウンロード

以下のコマンドで gin をダウンロードします
`go get github.com/gin-gonic/gin`

https://pleiades.io/help/go/create-a-project-with-go-modules-integration.html#working-with-go-modules

## 環境変数

- GOPATH: Go のコマンドやパッケージインストール先のホームディレクトリを指定する
- GOHOME: つかっているのか？？
- GOPRIVATE: private repos や Github Enterprise などから取得するための設定
  export GOPRIVATE="*.github.com"

## ディレクトリ構成について

参考にされるスタンダードなディレクトリ構成が、有志の方達の間で検討されているらしい。  
以下を参照
https://github.com/golang-standards/project-layout
