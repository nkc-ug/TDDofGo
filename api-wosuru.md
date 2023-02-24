# 銀行 API を作成する

オンライン銀行のコア ロジックができたので、ブラウザーから (またはコマンド ラインからでも) テストする Web API を作成してみましょう。 ここでは、データベースを使用してデータを保持しないので、すべての口座をメモリに格納するためのグローバル変数を作成する必要があります。

また、このガイドがあまり長くならないように、テスト部分はスキップします。 できれば、コア パッケージを作成したときと同じアプローチに従い、コードの前にテストを作成してください。

### メモリ内の口座を設定する <a href="#set-up-an-account-in-memory" id="set-up-an-account-in-memory"></a>

データベースを使用してデータを保持する代わりに、プログラムの開始時に作成されるメモリ マップを口座用に使用します。 また、口座番号を使用して口座情報にアクセスするために、マップを使用します。

`$GOPATH/src/bankapi/main.go` ファイルに移動し、グローバル変数 `accounts` を作成して口座でそれを初期化するための、次のコードを追加します。 (このコードは、前にテストを作成したときのものと似ています。)

Goコピー

```go
package main

import (
    "github.com/msft/bank"
)

var accounts = map[float64]*bank.Account{}

func main() {
    accounts[1001] = &bank.Account{
        Customer: bank.Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number: 1001,
    }
}
```

`$GOPATH/src/bankapi/` の場所にいることを保証します。 `go run main.go` でプログラムを実行し、エラーが発生しないことを確認します。 このプログラムでは今はまだ何も行われないので、Web API を作成するロジックを追加してみましょう。

### 明細メソッドを公開する <a href="#expose-the-statement-method" id="expose-the-statement-method"></a>

前のモジュールで見たように、Go で Web API を作成するのは簡単です。 引き続き `net/http` パッケージを使用します。 また、`HandleFunc` 関数と `ListenAndServe` 関数を使用してエンドポイントを公開し、サーバーを起動します。 `HandleFunc` 関数には、公開する URL パスの名前と、そのエンドポイント用のロジックを備えている関数の名前が必要です。

まず、口座の明細を出力する機能を公開します。 `main.go` に次の関数をコピーして貼り付けます。

Goコピー

```go
func statement(w http.ResponseWriter, req *http.Request) {
    numberqs := req.URL.Query().Get("number")

    if numberqs == "" {
        fmt.Fprintf(w, "Account number is missing!")
        return
    }

    if number, err := strconv.ParseFloat(numberqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid account number!")
    } else {
        account, ok := accounts[number]
        if !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", number)
        } else {
            fmt.Fprintf(w, account.Statement())
        }
    }
}
```

`statement` 関数で最初に注目する点は、ブラウザーに応答を書き戻すために、オブジェクトを受け取っていることです (`w http.ResponseWriter`)。 また、HTTP 要求の情報にアクセスするために、要求オブジェクトを受け取っています (`req *http.Request`)。

次に、`req.URL.Query().Get()` 関数を使用して、クエリ文字列からパラメーターを読み取っていることに注意してください。 このパラメーターは、HTTP の呼び出しを通じて送信する口座番号です。 その値を使用して、口座マップにアクセスし、情報を取得します。

ユーザーからデータを取得しているので、クラッシュを防ぐために検証を組み込む必要があります。 口座番号が有効であることがわかったら、`Statement()` メソッドを呼び出し、返された文字列をブラウザーに出力します (`fmt.Fprintf(w, account.Statement())`)。

ここで、`main()` 関数を次のように変更します。

Goコピー

```go
func main() {
    accounts[1001] = &bank.Account{
        Customer: bank.Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number: 1001,
    }

    http.HandleFunc("/statement", statement)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```

プログラムを実行して (`go run main.go`) エラーや出力が表示されない場合は、正常に動作しています。 Web ブラウザーを開き、URL `http://localhost:8000/statement?number=1001` を入力するか、プログラムの実行中に別のシェルで次のコマンドを実行します。

shコピー

```sh
curl http://localhost:8000/statement?number=1001
```

次の出力が表示されます。

出力コピー

```output
1001 - John - 0
```

### 預金メソッドを公開する <a href="#expose-the-deposit-method" id="expose-the-deposit-method"></a>

引き続き同じ方法を使用して、預金メソッドを公開してみましょう。 この例では、メモリ内にある口座にお金を追加します。 `Deposit()` メソッドを呼び出すたびに、残高が増加します。

メイン プログラムで、次のような `deposit()` 関数を追加します。 この関数で、クエリ文字列から口座番号を取得し、その口座が `accounts` マップに存在することを確認して、預金する金額が有効な値であることを検証してから、`Deposit()` メソッドを呼び出します。

Goコピー

```go
func deposit(w http.ResponseWriter, req *http.Request) {
    numberqs := req.URL.Query().Get("number")
    amountqs := req.URL.Query().Get("amount")

    if numberqs == "" {
        fmt.Fprintf(w, "Account number is missing!")
        return
    }

    if number, err := strconv.ParseFloat(numberqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid account number!")
    } else if amount, err := strconv.ParseFloat(amountqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid amount number!")
    } else {
        account, ok := accounts[number]
        if !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", number)
        } else {
            err := account.Deposit(amount)
            if err != nil {
                fmt.Fprintf(w, "%v", err)
            } else {
                fmt.Fprintf(w, account.Statement())
            }
        }
    }
}
```

この関数でも同様のアプローチに従って、ユーザーから受け取ったデータを取得して検証していることに注意してください。 また、`if` ステートメント内で直接変数を宣言して使用しています。 最後に、口座に若干の資金を追加した後、明細を出力して口座の新しい残高を確認します。

ここで、`deposit` 関数を呼び出す `/deposit` エンドポイントを公開する必要があります。 `main()` 関数を次のように変更します。

Goコピー

```go
func main() {
    accounts[1001] = &bank.Account{
        Customer: bank.Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number: 1001,
    }

    http.HandleFunc("/statement", statement)
    http.HandleFunc("/deposit", deposit)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```

プログラムを実行して (`go run main.go`) エラーや出力が表示されない場合は、正常に動作しています。 Web ブラウザーを開き、URL `http://localhost:8000/deposit?number=1001&amount=100` を入力するか、プログラムの実行中に別のシェルで次のコマンドを実行します。

shコピー

```sh
curl "http://localhost:8000/deposit?number=1001&amount=100"
```

次の出力が表示されます。

出力コピー

```output
1001 - John - 100
```

同じ呼び出しを複数回行うと、口座の残高が増え続けます。 実行時にメモリ内の `accounts` マップが更新されていることを確認してみてください。 プログラムを停止すると、行ったすべての預金は失われますが、この初期バージョンではそれは予想されることです。

### 引き出しメソッドを公開する <a href="#expose-the-withdraw-method" id="expose-the-withdraw-method"></a>

最後に、口座からお金を引き出すメソッドを公開してみましょう。 ここでも、最初にメイン プログラムで `withdraw` 関数を作成します。 この関数で、口座番号の情報を検証し、引き出しを行い、コア パッケージから受け取ったすべてのエラーを出力します。 メイン プログラムに次の関数を追加します。

Goコピー

```go
func withdraw(w http.ResponseWriter, req *http.Request) {
    numberqs := req.URL.Query().Get("number")
    amountqs := req.URL.Query().Get("amount")

    if numberqs == "" {
        fmt.Fprintf(w, "Account number is missing!")
        return
    }

    if number, err := strconv.ParseFloat(numberqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid account number!")
    } else if amount, err := strconv.ParseFloat(amountqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid amount number!")
    } else {
        account, ok := accounts[number]
        if !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", number)
        } else {
            err := account.Withdraw(amount)
            if err != nil {
                fmt.Fprintf(w, "%v", err)
            } else {
                fmt.Fprintf(w, account.Statement())
            }
        }
    }
}
```

次に、`main()` 関数に `/withdraw` エンドポイントを追加して、`withdraw()` 関数のロジックを公開します。 `main()` 関数を次のように変更します。

Goコピー

```go
func main() {
    accounts[1001] = &bank.Account{
        Customer: bank.Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number: 1001,
    }

    http.HandleFunc("/statement", statement)
    http.HandleFunc("/deposit", deposit)
    http.HandleFunc("/withdraw", withdraw)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```

プログラムを実行して (`go run main.go`) エラーや出力が表示されない場合は、正常に動作しています。 Web ブラウザーを開き、URL `http://localhost:8000/withdraw?number=1001&amount=100` を入力するか、プログラムの実行中に別のシェルで次のコマンドを実行します。

shコピー

```sh
curl "http://localhost:8000/withdraw?number=1001&amount=100"
```

次の出力が表示されます。

出力コピー

```output
the amount to withdraw should be greater than the account's balance
```

発生するエラーは、コア パッケージからのものであることに注意してください。 プログラムを開始すると、口座の残高はゼロになります。 そのため、お金を引き出すことはできません。 `/deposit` エンドポイントを数回呼び出して資金を追加し、`/withdraw` エンドポイントをもう一度呼び出して、正常に動作していることを確認します。

shコピー

```sh
curl "http://localhost:8000/deposit?number=1001&amount=100"
curl "http://localhost:8000/deposit?number=1001&amount=100"
curl "http://localhost:8000/deposit?number=1001&amount=100"
curl "http://localhost:8000/withdraw?number=1001&amount=100"
```

次の出力が表示されます。

出力コピー

```output
1001 - John - 200
```

これで完了です。 一から構築したパッケージの機能を公開するための Web API を作成しました。 次のセクションに進み、練習を続けてください。 今度は、独自の解答を作成するという課題があります。
