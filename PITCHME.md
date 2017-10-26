### Idris きほんのき

Yohei Oda

---

### whoami

- Scalaエンジニア（二年半）
- 仮想マシンとかGCとか好き
- HaskellはすごいH本を一通り読んだくらい
- Coq?😇

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
