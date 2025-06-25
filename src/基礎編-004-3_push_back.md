## 質問
> push_backはどのような機能ですか？

***
<h2><img src="image/KANAMI.png">

<font color="Maroon">KANAMI</font></h2>

お、`push_back`に目を付けたね！
`std::vector`を使いこなす上で、一番基本で一番よく使うメンバ関数だよ！

一言で言うと、`push_back`は **「`std::vector`の末尾に新しい要素を1つ追加する」** ためのメンバ関数だ。

### `std::vector`は「魔法のショッピングカート」

`std::vector`は、**「自動で大きさが変わる魔法のショッピングカート」**みたいなものだと思って。

`push_back`は、そのカートに**「商品をポンと1つ放り込む」**アクションそのもの。
カートがいっぱいになったら？ 大丈夫、カートが勝手にちょっと大きいサイズに伸びてくれるから、君は気にせずどんどん商品を放り込めるんだ。

この「勝手に伸びてくれる」っていうのが、`std::vector`のすごいところで、内部では前に話した「お引越し」みたいなメモリの再確保を全部自動でやってくれてるんだよ。私たちはただ`push_back`を呼ぶだけでいい。めちゃくちゃ便利でしょ！

### 簡単なコードで見てみよう

```cpp
#include <iostream>
#include <vector>
#include <string>

// ベクタの中身をきれいに表示するヘルパー関数
void print_vector(const std::vector<std::string>& vec) {
    std::cout << "Vector (size=" << vec.size() << "): [ ";
    for (const auto& item : vec) {
        std::cout << item << " ";
    }
    std::cout << "]" << std::endl;
}

int main() {
    // string（文字列）を格納する空のベクタを作成
    std::vector<std::string> items;

    std::cout << "--- 初期状態 ---" << std::endl;
    print_vector(items);

    std::cout << "\n--- push_back を使って要素を追加 ---" << std::endl;
    
    std::cout << "Adding 'Apple'..." << std::endl;
    items.push_back("Apple"); // 末尾に "Apple" を追加
    print_vector(items);

    std::cout << "Adding 'Banana'..." << std::endl;
    items.push_back("Banana"); // 末尾に "Banana" を追加
    print_vector(items);

    std::cout << "Adding 'Cherry'..." << std::endl;
    items.push_back("Cherry"); // 末尾に "Cherry" を追加
    print_vector(items);

    return 0;
}
```

実行結果を見ると、`push_back`を呼ぶたびにベクタの`size`が1つずつ増えて、末尾に新しい要素が追加されていくのがわかるよね。

### さっきのポリモーフィズムの例では…

`creatures.push_back(std::make_unique<Player>("KANAMI"));`

このコードは、

1.  `std::make_unique<Player>("KANAMI")` で、`Player`オブジェクトを指す`unique_ptr`をまず作る。
2.  そして、その出来上がった`unique_ptr`を、`creatures`という`vector`の末尾に`push_back`で追加している。

`Player`は`Creature`を継承してるから、`unique_ptr<Player>`は`vector<unique_ptr<Creature>>`に格納できる、ってわけだね。

### `push_back`の仲間たち

`std::vector`には、`push_back`以外にも便利なメンバ関数がたくさんあるよ。

* `pop_back()`: `push_back`の逆。**末尾の要素を1つ削除**する。
* `size()`: 現在の**要素数を取得**する。
* `empty()`: ベクタが**空かどうか**を `true`/`false` で返す。
* `clear()`: **全要素を削除**して、ベクタを空っぽにする。
* `resize(N)`: ベクタのサイズを強制的にN個にする。

この `push_back` は、`std::vector` を使う上で避けては通れない、基本にして最強の武器の一つだよ。要素数が事前に分からないデータを扱うときに、本当に頼りになる相棒さ。どんどん使って、この便利さを体に染み込ませてね！