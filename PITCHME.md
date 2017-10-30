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
  - 本(※)が出たのでバージョン1を出した
    - 今日の説明もこの本がベース
    - 興味があるかたはぜひ買ってみてください
  - プロダクションで使えるとかそういう話ではない

※ Type Driven Development with Idris

---

### What's idris?

- 依存型を扱うことができる

---

### 依存型

- 型と値が組み合わさったとても強い型
  - 型が値に依存する
- `types are first class`
  - 引数として渡したり計算したりできる
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

#### 型

```haskell
Vect : Nat -> Type -> Type
```

#### コンストラクタ

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
my_list_map f (x :: xs) = f x :: my_list_map f xs
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

### 関数のテストで面倒なこと

- 色々なパターンの入力に対処する必要がある
  1. 普通の値
  1. コーナーケース
  1. やってほしくないが型の都合上渡せてしまう値
- 更に入力値が複数あると組み合わせが爆発する

---

### 関数のテストで面倒なこと

- 呼び出し側のバリデーションは信用できない
- 明らかにダメそうな値が渡された時にどうするか？
  - 関数を実装する人の関心事ではない
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

- `List a` を `Vect n a` に変更
- 添字を `Fin n` (n より小さい自然数を表す型) で渡す
- 要素を必ず取り出せることを型で保証できる

---

### 関数に依存型を使うメリット

- あらかじめ関数が期待する値を型として表現できる
  - 複数の値の組み合わせも型の制約対象にできる
  - 期待する値が渡された時の振る舞いのテストに注力できる

---

### 関数に依存型を使うメリット

- 引数に渡す前のバリデーションを呼び出し側に強制できる
- 呼び出し側から見ると…
  - 関数が求める値をシグネチャだけで判断できる
  - 必要なバリデーションが明確

---

いい感じ😊

---

もうちょっと色々やってみましょう

---

### ここで問題

1. `zip: Vect n a -> Vect n b -> Vect n (a, b)` がある
1. `Vect n String` と `Vect m Int` がある
1. `n==m` は `True` である
1. `zip` を呼ぶことはできる？

---

- できない😭
- `==` の振る舞いは型として定義されているわけではない
  - 型チェッカーは「`n==m` だから `n と m は同じ` 」と判定できない
- 型チェッカーにもわかるように書いてみる
  - （実際は組み込みの関数が使えたりしますが…）

---

#### Natの値が同じであることを示す型

```haskell
data EqNat : (num1 : Nat) -> (num2 : Nat) -> Type where
     Same : (num : Nat) -> EqNat num num
```

- コンストラクタから以下のことが判断可能
  - `EqNat a b` 型の値が存在していれば、a と b は同じ値である

---

#### 2つの同じ値にそれぞれ1を足しても同じ値になる

```haskell
sameS : (k: Nat) -> (j: Nat) -> (eq : EqNat k j) -> EqNat (S k) (S j)
sameS j j (Same j) = Same(S j)
```

- `S k` は k より 1 大きい数
- `EqNat k j` が存在しているので k と j は同じ値として扱える

---

#### 2つの値が同じかどうか判定する関数

```haskell
checkEqNat : (num1 : Nat) -> (num2 : Nat) -> Maybe (EqNat num1 num2)
checkEqNat Z Z = Just (Same 0)
checkEqNat Z (S k) = Nothing
checkEqNat (S k) Z = Nothing
checkEqNat (S k) (S j) = case checkEqNat k j of
                              Nothing => Nothing
                              (Just eq) => Just (sameS _ _ eq)
```

- Z と Z は同じ(Zはゼロのこと)
- S k と Z は違う
- 2つの値を S k と S j と書けるなら k と j を調べる
  - k と j が同じなら `sameS` により S k と S j も同じ

---

#### 型を変換する関数

```haskell
exactLength : (len : Nat) -> (input : Vect m a) -> Maybe (Vect len a)
exactLength {m} len input = case checkEqNat m len of
                                 Nothing => Nothing
                                 Just (Same len) => Just input

```

- `len と m が等しい` 場合は `Vect m a` を `Vect len a` として扱えている
  - Idris が型レベルで m と len が同じだと判定できている

---

### まとめ
- Idrisは依存型を扱えるプログラミング言語
- 依存型では型が値に依存する
- 依存型を使うと、様々な事前条件や事後条件を関数の型で表せる
  - バリデーションが行われることを保証できる
  - 関数を書く側は変な値をテストする必要がない
- 型変換をするときは型チェッカーにわかるようにする必要がある

---

#### 個人的な感想
##### Idris のいいところ
　
- コンパイルが通ったときの信頼感がすごい
- すさまじく柔軟
  - 安全であれば割となんでもありなコンパイラ
  - たまになぜ動いているのかよくわからなくなる😇
- 依存型ならではの謎のテクニック
  - 返り値の型をパラメータ的に使ったりできる

---

#### 個人的な感想
##### Idris のつらいところ

- わかりづらいエラー
  - 型を見失う => 死
    - 途中で型がわからなくなって途方に暮れる
    - エラーが出始めると型のチェックすらできない
  - インデントを間違う => 死

---

#### まとめと感想
##### Idris のつらいところ

- 型チェッカーに怒られる => 死
  - とにかく厳密
    - `S a` と `a + 1` が違うくらいで激おこ
    - どう解決していいかよくわからない
- 依存型の実装/実装テクニックが独特
  - 思いつかない

---

#### まとめと感想
##### Idris のつらいところ

- 遅い
  - コンパイルも実行も遅い
    - Natがひどい
- CPUもぶんぶん回る   
- 普通のことが普通にできないときがある
  - 型の証明がちゃんと動いていないときがあるような…?
  - このへんは今後の改善に期待

---

### 参考文献
#### 公式

- [Type Driven Development with Idris](https://www.manning.com/books/type-driven-development-with-idris)
  - （実質的な）公式ガイドブック
  - 今回の資料はこの本ベースで作っています

- [Idris Tutorial](http://docs.idris-lang.org/en/latest/tutorial/index.html)
  - 公式チュートリアル

---

### 参考文献
#### 日本語

- [IdrisでWebアプリを書く](https://www.slideshare.net/tanakh/idrisweb)
  - 3年前の資料ですが面白いので最初に読むのにお勧め
  - この時の問題は現在もあまり解決されていない気が…

- [プログラミング言語 idris - wkwkesのやつ](http://wkwkes.hatenablog.com/entry/2016/12/17/000000)
  - tactic を使った証明の様子が紹介されています

---

### 参考文献
#### Haskellers 向け

- [Idris for Haskellers](https://github.com/idris-lang/Idris-dev/wiki/Idris-for-Haskellers)
  - Haskeller 向け

- [10 things Idris improved over Haskell](https://deque.blog/2017/06/14/10-things-idris-improved-over-haskell/)
  - Better Haskell としての Idris を知りたい人にお勧め

---
