# 基礎編1-1 std::coutについて
> std::coutはどういう場面で使用しますか？

***
## ![](./img/KANAMI.png "KANAMI") <font color="Maroon">KANAMI</font>

お、いい質問だね！ `std::cout` は、C++を学ぶ上で最初に出会う、とっても大事な相棒だよ。

一言で言うと、**「コンソール（ターミナルやコマンドプロンプトのこと。黒い画面のやつね）に、文字や変数の内容を表示したいとき」** に使うものだよ。

君が使ってきた言語で言うと、Pythonの`print()`関数や、Fortranの`PRINT *`文や`WRITE(*,*)`文と、やってることは全く同じだと思ってOK！

### `std::cout`の正体

前のコードで書いた `std::cout << "..." << std::endl;` を分解してみようか。

* `std`：これは「スタンダード（Standard）」っていう**名前空間（namespace）** のこと。「C++の標準機能ですよ」っていう目印みたいなものかな。いろんな人が作った関数や変数の名前が被らないようにするための仕組みだね。
* `::`：スコープ解決演算子。`std`っていう名前空間の中にある`cout`を使いますよ、っていう意味。
* `cout`：これが本体。「character output」の略で、文字を出力するためのオブジェクト（もの）。
* `<<`：ストリーム挿入演算子。矢印の向きにデータが流れていくイメージを持って！ 右側にあるものを、左側にある`std::cout`に「流し込んで」表示させてる感じ。

### こんな場面で大活躍！

じゃあ具体的にどんな時に使うのかっていうと、主にこんな感じかな。

1.  **デバッグ（一番よく使う！）**
    プログラムが思った通りに動かない時、「この変数、今どんな値になってるんだろう？」って確認したいよね。そんな時に`std::cout`で変数の値を表示させるのは、最も原始的で、でも最も確実なデバッグ方法の一つだよ。
    ```cpp
    int a = 5;
    int b = 10;
    std::cout << "変数aの値は: " << a << std::endl;
    int c = a * b;
    std::cout << "計算結果cの値は: " << c << std::endl;
    ```

2.  **プログラムの進捗状況の表示**
    君がこれから作るHPCのプログラムみたいに、計算にすごく時間がかかるものだと、「今、ちゃんと動いてるかな？」「全体の何%くらい終わったかな？」って知りたくなるよね。ループの中で現在の回数を表示したりすると、安心できる。
    ```cpp
    for (int i = 0; i < 10000; ++i) {
        if (i % 1000 == 0) { // 1000回ごとに表示
            std::cout << "現在 " << i << "回目のループ処理中..." << std::endl;
        }
        // 何か重い計算処理
    }
    ```

3.  **ユーザーへのメッセージ表示**
    ユーザーに何かを入力してほしい時とか、操作方法を案内する時に使う。
    ```cpp
    std::cout << "あなたの名前を入力してください: ";
    // この後、ユーザーからの入力を受け取る std::cin を使うことが多い
    ```

4.  **計算結果の表示**
    もちろん、プログラムの最終的な答えを表示するためにも使うよ。最初のサンプルコードがまさにこれだね。

### HPCでの注意点

ただ、君の最終目標であるMPIを使った並列計算では、ちょっと注意が必要だよ。
もし、たくさんの計算ノード（プロセス）が一斉に`std::cout`を使うと、それぞれの出力がターミナル上でごちゃ混ぜになって、何が何だか分からなくなっちゃうんだ。

だから、HPCの世界では、
* ランク0番のマスタープロセスだけが`std::cout`で結果を表示するように制御する
* 各プロセスは、コンソールじゃなくて、それぞれ別のログファイルに結果を書き出す（これは `std::ofstream` っていうのを使うよ）

みたいな工夫をすることが多いかな。まあ、これはもっと先の話だから、今は「そういうこともあるんだな」くらいに思っておけば大丈夫！

こんな感じで、`std::cout`はC++プログラミングの基本中の基本。プログラムからの「声」を聞くための大事なツールだから、まずはこれを使いこなせるようになろう！ またいつでも聞いてね！
