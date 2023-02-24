# テストの作成を始める

プログラムの作成に進む前に、テストについて説明してから、最初のテストを作成してみましょう。 パッケージ テストでは、Go パッケージの自動テストがサポートされています。 テストは、コードが意図したとおりに動作することを確認するために重要です。 一般に、機能を確認するには、パッケージ内の機能ごとに少なくとも 1 つのテストが必要です。

コーディングのときは、テスト駆動開発 (TDD) アプローチを使用するのがよい方法です。 この方法では、最初にテストを記述します。 テスト対象のコードがまだ存在しないため、これらのテストが失敗することを確認します。 次に、テストを満たすコードを記述します。

### テスト ファイルを作成する <a href="#create-the-test-file" id="create-the-test-file"></a>

最初に、`bankcore` パッケージ用のすべてのテストを保持する Go ファイルを作成する必要があります。 テスト ファイルを作成するときは、ファイルの名前が `_test.go` で終わるようにする必要があります。 その前には任意のものを指定できますが、パターンとしては、テスト対象のファイルの名前を使用します。

また、記述するすべてのテストは、`Test` で始まる関数にする必要があります。 その後には、通常、`TestDeposit` のように、作成しているテストのわかりやすい名前を記述します。

`$GOPATH/src/bankcore/` の場所に移動し、次のような内容で `bank_test.go` という名前のファイルを作成します。

Goコピー

```go
package bank

import "testing"

func TestAccount(t *testing.T) {

}
```

ターミナルを開き、場所 `$GOPATH/src/bankcore/` にいることを確認します。 それから、次のコマンドを使用して、詳細モードでテストを実行します。

shコピー

```sh
go test -v
```

Go によって、テストを実行するためのすべての `*_test.go` ファイルが検索されるため、次の出力が表示されます。

出力コピー

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
PASS
ok      github.com/msft/bank    0.391s
```

### 失敗するテストを記述する <a href="#write-a-failing-test" id="write-a-failing-test"></a>

コードを記述する前に、TDD を使用して、失敗するテストを最初に作成してみましょう。 次のコードを使用して、`TestAccount` 関数を変更します。

Goコピー

```go
package bank

import "testing"

func TestAccount(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    if account.Name == "" {
        t.Error("can't create an Account object")
    }
}
```

まだ実装していない口座と顧客の構造体を導入しました。 `t.Error()` 関数を使用して、何かが想定したとおりに行われない場合にテストが失敗することを確認します。

また、テストには、口座オブジェクト (まだ存在しません) を作成するロジックがあることに注意してください。 しかし、この時点では、パッケージと対話する方法を設計しています。

&#x20;<mark style="background-color:red;">注意</mark>

<mark style="background-color:red;">1 行ずつ説明したくはないので、テスト用のコードを提供します。 ただし、メンタル モデルでは、少しずつ開始し、必要なだけの繰り返しを行う必要があります</mark>。

ここでは、繰り返しを 1 回だけ行います。つまり、テストを記述し、失敗することを確認して、テストを満たすコードを記述します。 自分でコーディングするときは、簡単なものから始めて、作業を進めながら複雑にしていく必要があります。

`go test -v` コマンドを実行すると、失敗したテストが出力に表示されるはずです。

出力コピー

```output
# github.com/msft/bank [github.com/msft/bank.test]
.\bank_test.go:6:13: undefined: Account
.\bank_test.go:7:13: undefined: Customer
FAIL    github.com/msft/bank [build failed]
```

ここではこのままにしておきましょう。 後でオンライン銀行システムのロジックを記述しながら、このテストを完了し、新しいテストを作成します。
