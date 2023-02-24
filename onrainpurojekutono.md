# オンライン銀行プロジェクトの概要

### 機能と要件を定義する <a href="#define-the-features-and-requirements" id="define-the-features-and-requirements"></a>

ここで構築しようとしているオンライン銀行は概念実証であり、これにより、銀行プログラムの実現可能性を判断できます。 この最初の繰り返しでは、コア パッケージとのやり取りは CLI プログラムを介して行われます。 ユーザー インターフェイスは作成せず、データベースにデータは保持しません。 顧客からの取引明細の表示には、エンドポイントを公開します。

このオンライン銀行システムでは次のことを行います。

* 顧客が口座を作成できるようにします。
* 顧客がお金を引き出すことができるようにします。
* 顧客が別の口座に送金できるようにします。
* 顧客データと最終残高を含む取引明細を提供します。
* 取引明細を印刷するための Web API を、エンドポイントを通して公開します。

このプログラムは一緒に構築していくので、今は細部についてはあまり気にしないでください。

### 初期プロジェクト ファイルを作成する <a href="#create-the-initial-project-files" id="create-the-initial-project-files"></a>

プログラムに必要なファイルの初期セットを作成してみましょう。 すべての銀行のコア ロジックと、若干の顧客と預金や送金などのアクションでシステムを初期化するための `main` プログラム用に、Go パッケージを作成します。 さらに、この `main` プログラムでは、取引明細用のエンドポイントを公開するための Web API サーバーを起動します。

`$GOPATH` ディレクトリに次のファイル構造を作成してみましょう。

出力コピー

```output
$GOPATH/
  src/
    bankcore/
      go.mod
      bank.go
    bankapi/
      go.mod
      main.go
```

次に、適切なファイルのコードの記述にのみ集中できるように、`bankapi` メインプログラムから `bankcore` パッケージを呼び出せることを確認する `Hello World!` プログラムの記述を始めましょう。

次のコード スニペットをコピーして `src/bankcore/bank.go` に貼り付けます。

Goコピー

```go
package bank

func Hello() string {
    return "Hey! I'm working!"
}
```

ここでは、Go モジュールを使用します。 `src/bankcore/go.mod` で、後で参照できるように、次の内容を追加して、このパッケージに適切な名前を付けます。

Goコピー

```go
module github.com/msft/bank

go 1.14
```

次に、`bankcore` パッケージを呼び出す次のコードを、`src/bankapi/main.go` に追加します。

Goコピー

```go
package main

import (
    "fmt"

    "github.com/msft/bank"
)

func main() {
    fmt.Println(bank.Hello())
}
```

`src/bankapi/go.mod` では、次のようにローカル環境の `bankcore` パッケージ ファイルを参照する必要があります。

Goコピー

```go
module bankapi

go 1.14

require (
    github.com/msft/bank v0.0.1
)

replace github.com/msft/bank => ../bankcore
```

すべてが機能していることを確認するため、`$GOPATH/src/bankapi/` ディレクトリでターミナルを開き、次のコマンドを実行します。

shコピー

```sh
go run main.go
```

次の出力が表示されます。

出力コピー

```output
Hey! I'm working!
```

この出力により、プロジェクト ファイルが意図したとおりに正しく設定されていることを確認できます。 次に、オンライン銀行システムの初期機能セットを実装するコードの記述を始めます
