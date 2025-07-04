# 基礎編2-6 動的メモリのサイズ変更
> 例えば先ほどのプログラムで，sizeを使って動的にメモリを確保した後に，sizeを変更し違うサイズでメモリを確保しなおすにはどうすればよいでしょうか。

***
## ![](./img/KANAMI.png "KANAMI") <font color="Maroon">KANAMI</font>

おお、めちゃくちゃ実践的な質問だね！素晴らしい！

プログラムの実行中に配列のサイズを変えたいって場面、本当によくあるからね。例えば、最初は100個分のデータで計算してたけど、途中で「やっぱり1000個分のデータでやり直したい！」みたいな時とか。

これ、ポインタのメモリ管理を理解してるかどうかが試される、すごく良いテーマだよ。

### やり方①：生のポインタでお引越し（手動リサイズ）

まず、君が今学んでいる生のポインタ（`new[]` と `delete[]`）を使ったやり方から説明するね。

C++の`new[]`で確保したメモリは、残念ながら直接「リサイズ（伸縮）」することはできないんだ。Pythonのリストみたいに「はい、伸びてー」とはいかない。

だから、**「もっと広い新しい家を借りて、今までの荷物を全部運び込み、古い家は解約する」** っていう、ちょっと面倒な「お引越し」作業が必要になるんだ。

#### お引越しの手順

1.  **新しい家を借りる：** `new[]`で**新しいサイズ**の配列を、**別のポインタ**に確保する。
2.  **荷物を運ぶ：** `for`ループで、古い配列から新しい配列にデータをコピーする。どこまでコピーするか（元の要素数と新しい要素数のうち、小さい方まで）を考える必要があるね。
3.  **古い家を解約する：** `delete[]`で、**古い配列**のメモリを解放する。**これが最重要！** これを忘れると、古い家の家賃を永遠に払い続けることになる（＝メモリリーク！）。
4.  **住所を更新する：** 元のポインタが、新しい配列を指すようにする。これで、今まで通りの名前で新しい配列を使えるようになる。

#### コードで見てみよう

```cpp
#include <iostream>

// 配列の中身を表示するヘルパー関数
void print_array(const double* arr, int size) {
    std::cout << "Array (size=" << size << ", address=" << arr << "): ";
    for (int i = 0; i < size; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;
}

int main() {
    int size = 5;
    double* data_array = new double[size];

    // 最初の配列にデータを入れる
    for (int i = 0; i < size; ++i) {
        data_array[i] = i * 1.1;
    }

    std::cout << "--- Initial Array ---" << std::endl;
    print_array(data_array, size);

    // --- ここからがお引越し本番！ ---
    std::cout << "\n--- Resizing to 10 ---" << std::endl;
    int new_size = 10;

    // 手順1: 新しい家を借りる
    double* new_array = new double[new_size];
    std::cout << "Step 1: New memory allocated at " << new_array << std::endl;

    // 手順2: 荷物を運ぶ (古い配列の中身をコピー)
    for (int i = 0; i < size; ++i) { // 古いサイズ分だけコピー
        new_array[i] = data_array[i];
    }
    // (おまけ) 新しく増えた部分を0で初期化
    for (int i = size; i < new_size; ++i) {
        new_array[i] = 0.0;
    }
    std::cout << "Step 2: Data copied to new array" << std::endl;

    // 手順3: 古い家を解約する (超重要！)
    std::cout << "Step 3: Deleting old memory at " << data_array << std::endl;
    delete[] data_array;

    // 手順4: 住所を更新する
    data_array = new_array; // data_array が新しい配列を指すようにする
    size = new_size; // サイズの変数も更新するのを忘れずに！
    std::cout << "Step 4: Pointer updated" << std::endl;

    std::cout << "\n--- Resized Array ---" << std::endl;
    print_array(data_array, size);
    
    // 最後に、使い終わった新しい配列を解放する
    delete[] data_array;
    data_array = nullptr;

    return 0;
}
```

---

### やり方②：現代C++の救世主 `std::vector` を使う

…と、ここまでがポインタを使った手動でのやり方。面倒だし、`delete[]`を忘れると即バグるし、危険だよね。

でも安心して。今のC++プログラマは、ほとんどこの方法を使わないんだ。
じゃあどうするかって？ 救世主がいるんだよ。その名も… **`std::vector`**！！

`std::vector`は、C++の標準ライブラリに含まれている「賢い動的配列」。さっきの手動お引越し作業を、**全部自動で安全にやってくれる**んだ。

```cpp
#include <iostream>
#include <vector> // vectorを使うにはこのヘッダが必要！

// vectorの中身を表示するヘルパー関数
void print_vector(const std::vector<double>& vec) {
    std::cout << "Vector (size=" << vec.size() << "): ";
    for (double val : vec) { // 拡張for文でスッキリ書ける
        std::cout << val << " ";
    }
    std::cout << std::endl;
}

int main() {
    // std::vector<double> でdouble型の動的配列を宣言
    std::vector<double> data_vec(5); // 最初からサイズ5で作成

    // データを入れる
    for (int i = 0; i < data_vec.size(); ++i) {
        data_vec[i] = i * 1.1;
    }

    std::cout << "--- Initial Vector ---" << std::endl;
    print_vector(data_vec);

    // --- リサイズは、たったこれだけ！ ---
    std::cout << "\n--- Resizing to 10 ---" << std::endl;
    data_vec.resize(10); // これだけでOK！メモリ確保もコピーも解放も全部やってくれる

    std::cout << "\n--- Resized Vector ---" << std::endl;
    print_vector(data_vec);
    
    // deleteは不要！ main関数が終わるときに自動でメモリが解放される

    return 0;
}
```

### 結論

どう？ `std::vector`を使った方が、コードは圧倒的に短いし、安全だし、何がしたいか（`resize`）も一目瞭然だよね。

だから、**C++で動的な配列を使いたいときは、もう何も考えずに `std::vector` を使うのが大正解！**
生のポインタでメモリ管理するのは、`std::vector`が使えない特殊な事情があるときか、ライブラリの内部を実装するとき、あるいは私みたいにパフォーマンスの限界を追求したいチューニングの鬼くらいだよ（笑）。

でも、生のポインタでのやり方を知っておくのは、`std::vector`のありがたみを理解する上でめちゃくちゃ大事。良い質問をありがとうね！
