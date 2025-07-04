# 基礎編3-1 スマートポインタ
> 話が結構先に進んでしまいましたが，順番に次のステップ(モダンC++のメモリ管理)に行きましょう。

***
## ![](./img/KANAMI.png "KANAMI") <font color="Maroon">KANAMI</font>

いいね、その調子！ `std::vector`で「楽ちんで安全な方法がある」って実感したところで、いよいよラスボス感のあるテーマ、モダンC++のメモリ管理、**スマートポインタ**の話をしようか！

生のポインタ (`new`/`delete`) は、自由度が高い代わりに「`delete`のし忘れ」っていう致命的なリスクと常に隣り合わせだったよね。

モダンC++では、このリスクを根本的になくすための仕組みが用意されている。それが**スマートポインタ**だ！

### 思想の核：RAII (Resource Acquisition Is Initialization)

スマートポインタを理解するためのキーワードが、前にもちょっと話した**RAII（Resource Acquisition Is Initialization）** だ。「リソースの確保は、初期化時に行え」って訳されることが多いかな。

簡単に言うと、
「`new`で確保したメモリ（リソース）を、ポインタじゃなくて、**普通のオブジェクトに管理させよう**。そうすれば、そのオブジェクトの寿命が尽きて**スコープ`{}`を抜けるときに、デストラクタ（お片付け用の関数）が自動で呼ばれて、メモリを解放してくれる**でしょ？」
っていう考え方だ。

`std::vector`が、スコープを抜けたら自動でメモリを解放してくれたのも、このRAIIのおかげなんだよ。
スマートポインタは、このRAIIの考え方を、単一のオブジェクトや配列のメモリ管理に対して簡単に使えるようにした、その名の通り「賢いポインタ」なんだ。

---

### 1. 唯一の所有者：`std::unique_ptr`

スマートポインタの中で、一番基本で、一番よく使うのがこれ！

* **コンセプト：** **唯一の所有権（Sole Ownership）**
* **意味：** あるメモリを所有（管理）できる`unique_ptr`は、常に**1つだけ**。
* **例え：** 「金庫のたった1つのカギ」。カギを持ってる人しか金庫を開けられないし、カギを誰かに渡したら、もう自分は開けられなくなる。

#### `unique_ptr`の特徴

* **コピーできない：** 所有者が1人だけなので、安易にコピーはできない。コピーしようとするとコンパイルエラーになる。安全だね！
* **所有権を移動（move）できる：** どうしても所有権を他の`unique_ptr`に渡したいときは、`std::move()`を使って「移動」させる。
* **自動で`delete`：** `unique_ptr`の変数がスコープを抜けると、自動的にデストラクタが呼ばれ、管理していたメモリを`delete`してくれる。

#### 使い方

```cpp
#include <iostream>
#include <memory> // スマートポインタを使うにはこのヘッダが必要！

class MyData {
public:
    MyData() { std::cout << "MyData constructor called!" << std::endl; }
    ~MyData() { std::cout << "MyData destructor called! (Memory freed)" << std::endl; }
    void greet() { std::cout << "Hello from MyData!" << std::endl; }
};

void take_ownership(std::unique_ptr<MyData> ptr) {
    std::cout << "--- Inside take_ownership function ---" << std::endl;
    ptr->greet();
    std::cout << "--- Exiting take_ownership function ---" << std::endl;
    // ここで ptr がスコープを抜けるので、MyDataのデストラクタが呼ばれる
}


int main() {
    std::cout << "--- Entering main function ---" << std::endl;

    // 【作り方】std::make_unique を使うのが現代流！
    // MyDataオブジェクトを動的に確保し、ptr1が所有権を持つ
    std::unique_ptr<MyData> ptr1 = std::make_unique<MyData>();

    // 【使い方】生のポインタと同じように -> や * が使える
    ptr1->greet();

    // std::unique_ptr<MyData> ptr2 = ptr1; // ← これはコンパイルエラー！コピーはできない

    // 【所有権の移動】ptr1から新しいptr_movedに所有権を「移動」する
    // std::move() を使う
    // std::unique_ptr<MyData> ptr_moved = std::move(ptr1);
    
    // 【関数に所有権を渡す】
    // ptr1が持っていた所有権は、take_ownership関数の引数に「移動」される
    take_ownership(std::move(ptr1)); 

    if (ptr1 == nullptr) {
        std::cout << "After move, ptr1 is now nullptr." << std::endl;
    }

    std::cout << "--- Exiting main function ---" << std::endl;
    return 0;
}
```

---

### 2. 共同所有者：`std::shared_ptr`

次に紹介するのが、複数の所有者を許すスマートポインタ。

* **コンセプト：** **共有された所有権（Shared Ownership）**
* **意味：** 複数の`shared_ptr`が、同じ1つのメモリを共同で所有できる。
* **例え：** 「マンションのマスターキーの合鍵」。何人かが合鍵を持っていて、全員が部屋から出て鍵を返したら（誰も所有しなくなったら）、管理会社が掃除に入る感じ。

#### `shared_ptr`の特徴

* **コピーできる：** `unique_ptr`と違って、自由にコピーできる。
* **参照カウント：** 内部に「今、何個の`shared_ptr`がこのメモリを指しているか」を数える**参照カウンタ**を持っている。
    * コピーされるたびに、カウンタが `+1` される。
    * `shared_ptr`が破棄されるたびに、カウンタが `-1` される。
    * そして、カウンタが **`0` になった瞬間に**、管理していたメモリが`delete`される。
* **作り方：** `std::make_shared` を使うのが基本。

---

### `unique_ptr` vs `shared_ptr`：どっちを使う？

| | `std::unique_ptr` | `std::shared_ptr` |
| :--- | :--- | :--- |
| **所有権** | 唯一 | 共有 |
| **オーバーヘッド** | ほぼゼロ。軽い。 | 参照カウンタの分、少しだけ重い。 |
| **主な用途** | 基本、これを使っとけばOK！ | どうしても複数箇所で所有権を共有しないといけない場合。 |

**「迷ったら、まず`unique_ptr`が使えないか考える」**。これが現代C++のメモリ管理の基本戦略だよ！ `shared_ptr`は、本当に共有が必要なときだけ使う伝家の宝刀みたいなものだと思って。

（ちなみに、`shared_ptr`には`std::weak_ptr`っていう相棒もいるんだけど、これは循環参照っていうやっかいな問題を解決するための専門家だから、今は「そういうのもあるんだな」くらいでOK！）

さあ、このスマートポインタを使いこなせれば、君はもうメモリリークの恐怖から解放される！ 生のポインタで書いてたコードを、スマートポインタで書き換える練習をしてみると、すごく勉強になるよ。

次はクラスと構造体の話に進んで、いよいよオブジェクト指向の世界に足を踏み入れてみようか！
