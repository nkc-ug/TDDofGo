# 銀行のコア パッケージを作成する

テスト ファイルで実行される基本プロジェクトができたので、前のユニットの機能と要件を実装するコードの記述を始めましょう。 エラー、構造体、メソッドなど、説明したいくつかのトピックに戻ります。

`$GOPATH/src/bankcore/bank.go` ファイルを開き、`Hello()` 関数を削除して、オンライン銀行システムのコア ロジックの作成を始めます。

### 顧客と口座の構造体を作成する <a href="#create-structures-for-customers-and-accounts" id="create-structures-for-customers-and-accounts"></a>

最初に、銀行の顧客になりたい人の名前、住所、電話番号を格納する `Customer` 構造体を作成します。 また、`Account` データ用の構造体も必要です。 1 人の顧客が複数の口座を持つことができるため、顧客情報を口座オブジェクトに埋め込みます。 基本的に、`TestAccount` テストで定義したものを作成しましょう。

必要な構造体は次のコード例のようになります。

Goコピー

```go
package bank

// Customer ...
type Customer struct {
    Name    string
    Address string
    Phone   string
}

// Account ...
type Account struct {
    Customer
    Number  int32
    Balance float64
}
```

ターミナルで `go test -v` コマンドを実行すると、今度はテストが成功することがわかります。

出力コピー

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
PASS
ok      github.com/msft/bank    0.094s
```

`Customer` と `Account` の構造体を実装したので、このテストは合格します。 構造体ができたので、最初のバージョンの銀行で必要な機能を追加するためのメソッドを作成してみましょう。 これらの機能には、預金、引き出し、送金などがあります。

### 預金メソッドを実装する <a href="#implement-the-deposit-method" id="implement-the-deposit-method"></a>

最初に、口座にお金を追加できるメソッドが必要です。 ただし、その前に、`bank_test.go` ファイルに `TestDeposit` 関数を作成してみましょう。

Goコピー

```go
func TestDeposit(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    account.Deposit(10)

    if account.Balance != 10 {
        t.Error("balance is not being updated after a deposit")
    }
}
```

`go test -v` を実行すると、失敗したテストが出力に表示されるはずです。

出力コピー

```output
# github.com/msft/bank [github.com/msft/bank.test]
./bank_test.go:32:9: account.Deposit undefined (type Account has no field or method Deposit)
FAIL    github.com/msft/bank [build failed]
```

前のテストを満たすため、受け取った金額が 0 未満の場合にエラーを返す `Deposit` メソッドを `Account` 構造体に作成してみましょう。 それ以外の場合は、受け取った金額をそのまま口座の残高に追加します。

`Deposit` メソッドには、次のコードを使用します。

Goコピー

```go
// Deposit ...
func (a *Account) Deposit(amount float64) error {
    if amount <= 0 {
        return errors.New("the amount to deposit should be greater than zero")
    }

    a.Balance += amount
    return nil
}
```

`go test -v` を実行すると、テストが成功していることがわかります。

出力コピー

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
=== RUN   TestDeposit
--- PASS: TestDeposit (0.00s)
PASS
ok      github.com/msft/bank    0.193s
```

次のように、マイナスの金額を預金しようとするとエラーが発生することを確認するテストを作成することもできます。

Goコピー

```go
func TestDepositInvalid(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    if err := account.Deposit(-10); err == nil {
        t.Error("only positive numbers should be allowed to deposit")
    }
}
```

`go test -v` コマンドを実行すると、テストが成功していることがわかります。

出力コピー

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
=== RUN   TestDeposit
--- PASS: TestDeposit (0.00s)
=== RUN   TestDepositInvalid
--- PASS: TestDepositInvalid (0.00s)
PASS
ok      github.com/msft/bank    0.197s
```

&#x20;<mark style="background-color:red;">注意</mark>

<mark style="background-color:red;">ここからは、メソッドごとに 1 つのテスト ケースを記述します。 ただし、予想されるシナリオと予想されないシナリオの両方に対応できるよう、満足できるだけの数のテストをプログラムに記述する必要があります。 たとえば、この場合は、エラー処理ロジックをテストします</mark>。

### 引き出しメソッドを実装する <a href="#implement-the-withdraw-method" id="implement-the-withdraw-method"></a>

引き出し機能を記述する前に、テストを作成してみましょう。

Goコピー

```go
func TestWithdraw(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    account.Deposit(10)
    account.Withdraw(10)

    if account.Balance != 0 {
        t.Error("balance is not being updated after withdraw")
    }
}
```

`go test -v` コマンドを実行すると、失敗したテストが出力に表示されるはずです。

出力コピー

```output
# github.com/msft/bank [github.com/msft/bank.test]
./bank_test.go:67:9: account.Withdraw undefined (type Account has no field or method Withdraw)
FAIL    github.com/msft/bank [build failed]
```

`Withdraw` メソッドのロジックを実装してみます。ここでは、パラメーターとして受け取った金額だけ口座の残高を減らします。 前に行ったように、受け取った値が 0 より大きく、口座の残高が十分であることを検証する必要があります。

`Withdraw` メソッドには、次のコードを使用します。

Goコピー

```go
// Withdraw ...
func (a *Account) Withdraw(amount float64) error {
    if amount <= 0 {
        return errors.New("the amount to withdraw should be greater than zero")
    }

    if a.Balance < amount {
        return errors.New("the amount to withdraw should be greater than the account's balance")
    }

    a.Balance -= amount
    return nil
}
```

`go test -v` コマンドを実行すると、テストが成功していることがわかります。

出力コピー

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
=== RUN   TestDeposit
--- PASS: TestDeposit (0.00s)
=== RUN   TestDepositInvalid
--- PASS: TestDepositInvalid (0.00s)
=== RUN   TestWithdraw
--- PASS: TestWithdraw (0.00s)
PASS
ok      github.com/msft/bank    0.250s
```

### 明細メソッドを実装する <a href="#implement-the-statement-method" id="implement-the-statement-method"></a>

口座名、番号、残高を含む明細を出力するメソッドを作成してみましょう。 ただし、まず、`TestStatement` 関数を作成します。

Goコピー

```go
func TestStatement(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    account.Deposit(100)
    statement := account.Statement()
    if statement != "1001 - John - 100" {
        t.Error("statement doesn't have the proper format")
    }
}
```

`go test -v` を実行すると、失敗したテストが出力に表示されるはずです。

出力コピー

```output
# github.com/msft/bank [github.com/msft/bank.test]
./bank_test.go:86:22: account.Statement undefined (type Account has no field or method Statement)
FAIL    github.com/msft/bank [build failed]
```

文字列を返す `Statement` メソッドを記述してみましょう。 (後の課題では、このメソッドを上書きする必要があります。)次のコードを使用します。

Goコピー

```go
// Statement ...
func (a *Account) Statement() string {
    return fmt.Sprintf("%v - %v - %v", a.Number, a.Name, a.Balance)
}
```

`go test -v` を実行すると、テストが成功していることがわかります。

出力コピー

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
=== RUN   TestDeposit
--- PASS: TestDeposit (0.00s)
=== RUN   TestDepositInvalid
--- PASS: TestDepositInvalid (0.00s)
=== RUN   TestWithdraw
--- PASS: TestWithdraw (0.00s)
=== RUN   TestStatement
--- PASS: TestStatement (0.00s)
PASS
ok      github.com/msft/bank    0.328s
```

次のセクションに進み、`Statement` メソッドを公開するための Web API を記述してみましょう!
