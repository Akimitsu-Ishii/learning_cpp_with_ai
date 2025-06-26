# 基礎編4-2 継承とポリモーフィズム
> 「継承」と「ポリモーフィズム」について教えてください。

***
## ![](./img/KANAMI.png "KANAMI") <font color="Maroon">KANAMI</font>

キタね！継承とポリモーフィズム！

この２つは、オブジェクト指向プログラミング（OOP）の華であり、C++を真にパワフルにする両輪だよ！ クラスっていう「部品」を、さらに柔軟で再利用性の高いものに進化させるための超強力な仕組みなんだ。

ちょっと長くなるけど、ここはじっくり行こう！

### 1. 継承 (Inheritance)：「親子関係」で設計図を再利用！

まず**継承**から。一言で言うと、**「すでにあるクラス（設計図）を元にして、機能を追加・変更した新しいクラスを作ること」** だよ。

* **親クラス (Base Class / 基底クラス):** 元になるクラス。
* **子クラス (Derived Class / 派生クラス):** 新しく作るクラス。

子クラスは、親クラスのメンバ変数やメンバ関数を**すべて受け継ぐ**。その上で、子クラス独自の機能を追加したり、親から受け継いだ機能を自分流に書き換えたり（**オーバーライド**）できるんだ。

#### 例え：ゲームのキャラクター

`Player`（プレイヤー）も`Monster`（モンスター）も、ゲームの世界に存在する「生き物」としては共通点が多いよね。例えば、どっちもHPを持ってるとか。

こういうとき、「生き物」っていう親クラス `Creature` をまず作るんだ。

```cpp
#include <iostream>
#include <string>

// 親クラス：生き物全般の設計図
class Creature {
protected: // 子クラスからはアクセスできるけど、外部からはできない
    std::string name_;
    int hp_;

public:
    Creature(const std::string& name, int hp) : name_(name), hp_(hp) {}

    void move() const {
        std::cout << name_ << " は移動した。" << std::endl;
    }

    void displayStatus() const {
        std::cout << name_ << " (HP: " << hp_ << ")" << std::endl;
    }
};

// 子クラス：Creatureを「継承」してPlayerを作る
// class Player : public Creature { ... } のように書く
class Player : public Creature {
private:
    int level_; // Player独自のメンバ変数

public:
    // 親クラスのコンストラクタを呼び出す
    Player(const std::string& name, int hp, int level)
        : Creature(name, hp), level_(level) {}

    void attack() const { // Player独自のメンバ関数
        std::cout << name_ << " は剣で攻撃した！" << std::endl;
    }
};

// 子クラス：Creatureを継承してMonsterを作る
class Monster : public Creature {
public:
    Monster(const std::string& name, int hp) : Creature(name, hp) {}

    void bite() const { // Monster独自のメンバ関数
        std::cout << name_ << " は噛みついてきた！" << std::endl;
    }
};
```
`Player`と`Monster`は、自分で`move()`や`displayStatus()`を定義してなくても、親である`Creature`から受け継いでいるから、普通に使える。これが継承による**コードの再利用**だね。

---

### 2. ポリモーフィズム (Polymorphism)：「同じ指示」で「違う動き」！

次に**ポリモーフィズム**。日本語だと **「多態性（たたいせい）」** とか **「多様性」** って訳される。一言で言うと、**「同じ見た目（インターフェース）なのに、中身の実体によって違う振る舞いをすること」**。

さっきの例で、`Player`も`Monster`も`Creature`だよね。だから、`Creature`へのポインタ（`Creature*`）は、`Player`オブジェクトも`Monster`オブジェクトも指すことができるんだ。

でも、`Player`と`Monster`では「話し方」が違うはず。`Player`は「よろしく！」って言うけど、`Monster`は「グルルル…」って唸る、みたいな。
この「話す (`talk`)」っていう同じ指示に対して、中身に応じて違う動きをさせるのがポリモーフィズムの役目なんだ。

#### 魔法のキーワード：`virtual`

これを実現するのが`virtual`キーワード！

親クラスのメンバ関数の宣言の頭に `virtual` を付けると、「この関数は、子クラスで自分流に書き換え（オーバーライド）されるかもしれませんよ」っていう印になる。

`virtual`関数を、親クラスのポインタ経由で呼び出すと、C++はポインタの型（`Creature*`）ではなく、**ポインタが実際に指しているオブジェクトの型（`Player`なのか`Monster`なのか）**を見て、その型に合った正しいバージョンの関数を自動で呼び出してくれるんだ！

#### コードで見てみよう！

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>

class Creature {
protected:
    std::string name_;
public:
    Creature(const std::string& name) : name_(name) {}
    virtual ~Creature() = default; // 親クラスのデストラクタはvirtualにするのがお作法！

    // 「話す」というアクションをvirtual関数として定義
    virtual void talk() const {
        std::cout << name_ << ": ... (無口だ)" << std::endl;
    }
};

class Player : public Creature {
public:
    Player(const std::string& name) : Creature(name) {}

    // 親のtalk()を自分流に書き換える（オーバーライド）
    // overrideキーワードを付けると、書き換えのつもりが間違ってないかコンパイラがチェックしてくれる
    void talk() const override {
        std::cout << name_ << ": 「よろしくな！」" << std::endl;
    }
};

class Monster : public Creature {
public:
    Monster(const std::string& name) : Creature(name) {}

    // こっちも自分流にオーバーライド
    void talk() const override {
        std::cout << name_ << ": 「グルルルル...」" << std::endl;
    }
};

int main() {
    // いろんな種類のCreatureを一緒くたに管理する配列
    std::vector<std::unique_ptr<Creature>> creatures;

    // 型は違うけど、どっちもCreatureだから同じ配列に入れられる！
    creatures.push_back(std::make_unique<Player>("KANAMI"));
    creatures.push_back(std::make_unique<Monster>("スライム"));
    creatures.push_back(std::make_unique<Monster>("ドラゴン"));

    // 全員のtalkを聞いてみよう
    for (const auto& c : creatures) {
        // cの型はCreatureへのポインタだけど、
        // talk()がvirtualだから、ちゃんと中身に合った方のtalk()が呼ばれる！
        c->talk();
    }

    return 0;
}
```
この`main`関数を見て！ `creatures`のループの中では、`c`が`Player`を指してるのか`Monster`を指してるのかなんて全く気にしてないよね。ただ「`Creature`として`talk()`を呼び出してる」だけ。

それなのに、実行結果はちゃんと
```
KANAMI: 「よろしくな！」
スライム: 「グルルルル...」
ドラゴン: 「グルルルル...」
```
ってなる。これがポリモーフィズムの魔法だよ！

### まとめ

* **継承:** クラスの**親子関係**を作り、コードを**再利用**する仕組み。
* **ポリモーフィズム:** `virtual`関数を使って、**同じ指示で、実体ごとに違う振る舞い**をさせる仕組み。

この2つを組み合わせることで、コードの再利用性が爆発的に上がって、変更に強いしなやかなプログラムが作れるようになる。例えば、後から`Dragon`っていう新しいモンスターの種類を追加したくなっても、`Creature`を継承して`talk`をオーバーライドするだけで、`main`関数のループは一切変更しなくていいんだ。

まさにオブジェクト指向の真髄だね！
