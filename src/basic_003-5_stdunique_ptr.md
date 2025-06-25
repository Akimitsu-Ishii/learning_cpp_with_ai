# 基礎編3-5 std::unique_ptr変数の宣言
> std::unique_ptr<MyData> ptr1 = std::make_unique<MyData>();
> この部分の書き方が良くわかりません。

***
## ![](./img/KANAMI.png "KANAMI") <font color="Maroon">KANAMI</font>

よし来た！そこ、絶対ギモンに思うと思ったよ！

`std::unique_ptr<MyData> ptr1 = std::make_unique<MyData>();`

この一行にモダンC++の美味しいところがギュッと詰まってるから、分解して見ていこう！

---

### 分解してみよう！

この行は `=` を境に、左側（LHS）と右側（RHS）に分けられる。

#### 左側： `std::unique_ptr<MyData> ptr1`

**意味：「`ptr1`という名前の変数を宣言します。その型は `std::unique_ptr<MyData>` です」**

* `ptr1`: これは単なる変数名だね。
* `std::unique_ptr<MyData>`: これが変数の「型」。
    * `std::unique_ptr`は「テンプレート」っていうC++の機能で出来てる。テンプレートっていうのは、型をカスタマイズできる設計図みたいなもの。
    * `<MyData>` の部分で、「**`MyData`型のオブジェクトを指すための、唯一の所有権を持つスマートポインタ**ですよ」と、型を具体的に指定しているんだ。
    * もし`int`を管理したいなら `std::unique_ptr<int>` になるし、`Player`クラスなら`std::unique_ptr<Player>`になる。

ここまでは、「`MyData`専用の`unique_ptr`型の変数`ptr1`を用意した」って感じだね。

#### 右側： `std::make_unique<MyData>()`

**意味：「`MyData`型のオブジェクトを1つ作って、それを管理する`unique_ptr`を生成してください」**

これが今回の主役！ C++14から導入された、`unique_ptr`を**安全かつ簡単に作るための専用のヘルパー関数**なんだ。

これもさらに分解しよう。

* `std::make_unique`: 「`unique_ptr`を作りますよ」っていう関数名。
* `<MyData>`: ここでもテンプレートが出てきた！「これから作るのは **`MyData`型のオブジェクト**ですよ」って、作るモノの型を教えてあげてる。
* `()`: 関数の呼び出し。
    * もし`MyData`のコンストラクタ（誕生の瞬間に呼ばれる関数）が引数を取るなら、この丸カッコ `()` の中に引数を書くんだ。
    * 例えば、前に作った`Player`クラスなら、こんな感じになる。
      ```cpp
      // Player("KANAMI", 200) を呼び出してオブジェクトを作り、unique_ptrで管理する
      auto ptr_player = std::make_unique<Player>("KANAMI", 200); 
      ```

つまり、`std::make_unique<MyData>()` は、内部でこっそり
1.  `new MyData()` を実行して、メモリ上に`MyData`オブジェクトを1つ確保する。
2.  その確保したメモリを管理するための `std::unique_ptr<MyData>` を作る。
3.  そして、その出来上がった `unique_ptr` を返してくれる。

この「オブジェクトの確保」と「それを管理する`unique_ptr`の作成」をワンセットでやってくれる、超便利な「作成キット」なんだ。

### なぜ `make_unique` を使うの？ 古い書き方との比較

C++11の時代（`make_unique`がまだ無かった頃）は、こう書いてたんだ。

```cpp
// C++11時代の古い書き方
std::unique_ptr<MyData> ptr1(new MyData());
```

これでも動くんだけど、問題点がいくつかある。
* `MyData`って2回も書いてて、ちょっと面倒。
* `new`を直接使ってるから、`delete`し忘れそうな雰囲気があって、ちょっと怖い。
* （上級者向けの話）ごく稀なケースで、例外が発生したときにメモリリークする可能性がある。

`std::make_unique`を使うと、これらの問題が全部解決する！
* `new`を隠蔽してくれるから、安全な感じがする。
* コードがスッキリする。
* 例外に対しても安全。

### KANAMIからのアドバイス：`auto`を使ってもっとモダンに！

さらに、`make_unique`を使うときは、`auto`キーワードと組み合わせるのが現代の主流だよ。

```cpp
// 超モダンな書き方！
auto ptr1 = std::make_unique<MyData>();
```

`auto`は、「右辺の式から、変数の型をコンパイラが自動で推測してね！」っていうおまじない。
`std::make_unique<MyData>()` が `std::unique_ptr<MyData>` を返すことはコンパイラには分かりきってるから、「型名は書くの省略するんで、よろしく！」ってできるんだ。これで、左辺の長ったらしい型名を書かなくて済む！

**結論として、C++14以降を使える環境なら、`new`を直接書く代わりに、常に`std::make_unique`と`auto`をセットで使うのが、現代的で安全で、イケてる書き方なんだ！**

この`auto`と`make_unique`の組み合わせは、本当にC++を書きやすくしてくれた革命的な機能の一つ。ぜひマスターして、君のコードをモダンなスタイルに進化させていこう！
