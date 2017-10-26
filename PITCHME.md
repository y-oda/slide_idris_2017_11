### Idris きほんのき

Yohei Oda

---

### whoami

- Scalaエンジニア（二年半）
- JVMとかGCとか好き
- HaskellはすごいH本を一通り読んだくらい
- Coq??😇

---

### What's idris?

- Haskellベースの関数型言語
  - 文法はほぼHaskell
  - 実装もHaskellメイン
  - 使いやすいBetterHaskell的な側面
    - 正格評価がデフォルト
    - `:` と `::` が逆
    - ちょっと便利な関数が増えてる etc

---

### What's idris?

- 資料を書いている時点でバージョン1.1.1
  - 本(Type driven development with Idris)が出たのでバージョン1を出した
    - 今日の説明もこの本をベースにしているので、興味が出たらぜひ買って読んでみてください
    - 実はまだ半分くらいしか読めてない
  - プロダクションで使えるとかそういう話ではない

---

### What's idris?

- 依存型を扱うことができる

---

### 依存型

- 型と値が組み合わさったとても強い型
  - 型が値に依存する
- `types are first class`
  - 値や関数と同じように、引数として渡して計算したりできる
- 型を使った命題の証明ができる
  - Coq や Agda などが代表的
  - Idris は証明より型安全な（普通の）プログラミング路線（らしい）
  
---

### 依存型の例

- 長さを型に持つリスト Vect 型
  - `Vect n a`
    - n は要素の数、 a は要素の型

---

### Vect の型とコンストラクタ

- 型

```haskell
Vect : Nat -> Type -> Type
```

- コンストラクタ

```haskell
Nil : Vect 0 elem
    Empty vector

(::) : (x : elem) -> (xs : Vect len elem) -> Vect (S len) elem
    A non-empty vector of length S len, consisting of a head element and the rest of the list, of length len.
    infixr 7
```

---

### Vect の基本的な処理を書いてみます

---

### List の map

```haskell
my_list_map: (a->b) -> List a -> List b
my_list_map f [] = []
my_list_map f (x :: xs) = f x :: my_map f xs
```

---

### Vect の map

```haskell
my_vect_map: (a->b) -> Vect n a -> Vect n b
my_vect_map f [] = []
my_vect_map f (x :: xs) = f x :: my_vect_map f xs
```

- 「map関数を使っても要素の数は変わらない」ことが型で保証されている

---

### List の append

```haskell
my_list_append : List a -> List a -> List a
my_list_append [] ys = ys
my_list_append (x :: xs) ys = x :: my_list_append xs ys
```

---

### Vect の append

```haskell
my_vect_append : Vect n a -> Vect m a -> Vect (n + m) a
my_vect_append [] ys = ys
my_vect_append (x :: xs) ys = x :: my_vect_append xs ys
```

- 引数の2つの長さを合計した長さのVectが返ってくる

---

### Vect の zip

```haskell
my_vect_zip : Vect n a -> Vect n b -> Vect n (a, b)
my_vect_zip [] ys = []
my_vect_zip (x :: xs) (y :: ys) = (x, y) :: my_vect_zip xs ys
```

- 同じ長さの2つのVectにしか使うことができない
- 複数の引数をとる場合でもシグネチャだけで関係性を表現できる

---

### 関数をテストするときに面倒なこと

- 色々なパターンの入力に対処する必要がある
  1. 普通の値
  1. コーナーケース
  1. やってほしくないが型の都合上渡せてしまう値
- 更に入力値が複数あると組み合わせが爆発する
- 関数の呼び出し側が値をチェックしてくれているかは信用できない
- 最後のパターンは関数を実装する人の関心の対象外
  - わけのわからない値を入れてくる方が悪い😤

---

### リストの要素を添字で取り出す index 関数 を作る

```haskell
index: List a -> Integer -> a
```

- マイナスの数値が来た時どうする？
- リストのサイズより大きな値がきたらどうする？

---

### 型をちょっと強くする

```haskell
index: List a -> Nat -> a
```

- 添字を `Nat` (自然数を表す型) で渡すように変更
  - マイナスの数を渡すことはできない
- リストのサイズより大きな値がきたらどうする？

---

### 依存型を使う

```haskell
index: Vect n a -> Fin n -> a
```

- 添字を `Fin n` (n より小さい自然数を表す型) で渡す
- 要素を必ず取り出せることを型で約束できる

---

### その他の例

- リストをとるが空リストはダメという関数を書きたい時

```haskell
func: Vect (S n) a -> 返したい型
```

- `(S n)` は1以上なので、空リスト `Vect Z a` では型があわない

---

### 関数に依存型を使うメリット

- あらかじめ関数が期待する値を型として表現できる
  - 複数の値の組み合わせも型の制約の対象にできる
  - 関数を実装する人は期待する値が渡された時の振る舞いだけをテストすればよい
- 引数に渡す前のバリデーションを呼び出し側に強制できる
  - 呼び出し側もどのような値を求めているのかシグネチャだけで判断できる

---
