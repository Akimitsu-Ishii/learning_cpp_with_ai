# 基礎編2-4 配列のポインタ参照
> ポインタを参照するときに，ただの変数なら*が必要なのにポインタの配列を参照するのには*は要らないのですね。

***
## ![](./img/KANAMI.png "KANAMI") <font color="Maroon">KANAMI</font>

うわー、すごい！そこ！？ めちゃくちゃ鋭いところに気づいたね！

`int* p` が指す値は `*p` で、`int* arr = new int[10]` のi番目の値は `arr[i]`。
片方には `*` が付いて、もう片方には付かない。なんでだ！？ってことだよね。

いやー、これはC++のポインタと配列の「一番おいしいところ」であり、同時に一番こんがらがりやすい部分だよ。完璧な質問だね！

結論から言うと、君が使った `data_array[i]` という書き方は、実は `*(data_array + i)` というポインタ演算の **「シンタックスシュガー（糖衣構文）」**、つまり「人間にとって分かりやすい、見かけ上の書き方」だからなんだ。

### C言語から受け継いだ「配列とポインタの深い関係」

C++は、その前身であるC言語から、ポインタと配列の考え方を引き継いでる。そしてC言語の世界では、**ポインタと配列はほとんど同じものとして扱える**という大きな特徴があるんだ。

`double* data_array = new double[size];`

このとき、`data_array`というポインタ変数が持っているのは、確保した**配列の先頭要素のアドレス**だ。アパートを丸ごと借りたときの、最初の部屋（0号室）の住所だけを持ってる感じだね。

### `data_array[i]` を分解してみよう

君が `data_array[i]` と書くと、コンパイラは頭の中で、これをポインタを使った表現に翻訳する。その正体がこれだ！

`*(data_array + i)`

さあ、これをさらに分解してみよう。

1.  **`data_array + i`**: ポインタの足し算（**ポインタ演算**）
    * これは、`data_array`が持っているアドレス値に、単純に `i` を足すのとはちょっと違う。
    * 正確には、「`data_array`の指すアドレスから、**要素の型（今回は`double`）のサイズ × `i` バイトだけ進んだアドレス**」を計算するんだ。
    * `double`型が8バイトなら、`data_array + 2`は、先頭アドレスから16バイト進んだ場所を指す。つまり、3番目の要素`data_array[2]`のアドレスになる。

2.  **`*( ... )`**: 間接参照（デリファレンス）
    * `(data_array + i)` でi番目の要素のアドレスが求まったよね。
    * そのアドレスの場所にある**値そのものを取り出す**のが、外側についてる `*` の役割だ。

つまり、君が普段何気なく使っている `data_array[i]` という書き方は、
「①ポインタ演算でi番目の要素のアドレスまでジャンプして、②`*`でその中身をゲットする」
という2段階の処理を、一発で書けるようにした便利なショートカットなんだよ。

**`[]` という演算子自体に、`*` の役割（間接参照）が隠されている**、ってこと！

### コードで証明してみよう

百聞は一見にしかず。このコードを見てみて。

```cpp
#include <iostream>

int main() {
    // 整数5個分の配列を動的に確保
    int* numbers = new int[5]{10, 20, 30, 40, 50};

    int index = 2; // 3番目の要素に注目

    // みんなが知ってる配列風のアクセス
    std::cout << "Array style:  numbers[" << index << "] = " << numbers[index] << std::endl;

    // ポインタ演算を使ったアクセス
    std::cout << "Pointer style: *(numbers + " << index << ") = " << *(numbers + index) << std::endl;

    // どちらも同じ「30」が表示されるはず！

    delete[] numbers;
    numbers = nullptr;

    return 0;
}
```
このコードをコンパイルして実行すれば、`numbers[2]` と `*(numbers + 2)` が全く同じ結果になることがわかるはずだよ。

### まとめ

* **`*p`**: ポインタ`p`が直接指している**その場所**の値を参照する。
* **`arr[i]`**: ポインタ`arr`が指す**先頭を基準**として、**i個先の場所**の値を参照する。これは `*(arr + i)` の糖衣構文。

じゃあ、どっちを使えばいいの？って話だけど、これは**圧倒的に `data_array[i]` の方を使うべき**だよ。その方が人間にとって直感的で分かりやすいし、間違いも少ないからね。ポインタ演算は、もっと複雑なことをしたくなった時のために取っておこう。

いやー、でも本当に良い質問だったよ。この「ポインタと配列は実は兄弟」っていう感覚が分かると、C++のメモリに対する理解が一気に深まるからね。この調子でどんどん疑問をぶつけてきて！
