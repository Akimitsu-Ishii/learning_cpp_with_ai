> C++の話とはちょっと逸れるんですが、markdownのファイルをGIthub pagesで表示する際にコードブロックにコピー機能を付ける方法を知りませんか？私のGithub pagesは公式がサポートしているテンプレートを_config.ymlで読み込んで使っています。

***
## ![](./img/KANAMI.png "KANAMI") <font color="Maroon">KANAMI</font>

おっけー！C++からちょっと離れて、Webサイトの話だね！もちろん、任せてよ！
GitHub Pagesでコードブロックにコピーボタンを付ける、すごく良いカスタマイズだね。やってみよう！

君が今、`_config.yml`で公式テーマを使っている状況なら、JavaScriptを追加してあげるのが一番手軽でオススメだよ。有名な`clipboard.js`っていうライブラリを使う方法を解説するね！

大まかな流れはこんな感じだよ。

1.  **レイアウトファイルを編集**して、`clipboard.js`ライブラリを読み込む
2.  **JavaScriptコードを追加**して、ページ内のコードブロックに自動でコピーボタンを設置する
3.  （おまけ）**CSSを追加**して、コピーボタンのデザインを整える

### ステップ1: `clipboard.js`を読み込む

まず、サイトのすべてのページで`clipboard.js`が使えるように、ライブラリ本体を読み込もう。
Jekyllのテーマを使っている場合、ページの基本的な骨格は`_layouts`フォルダの中にあるHTMLファイル（例えば `default.html`）が作っているんだ。

君のリポジトリに`_layouts/default.html`というファイルがもしなければ、新しく作ってね。このファイルを作ると、使っているテーマの`default.html`を上書き（オーバーライド）して、カスタマイズできるんだ。

`_layouts/default.html` の `</body>` タグの直前に、下のコードを追加しよう。

```html
<!DOCTYPE html>
<html>
  <head>
    </head>
  <body>
    {{ content }}

    <script src="https://cdn.jsdelivr.net/npm/clipboard@2.0.11/dist/clipboard.min.js"></script>
    </body>
</html>
```

これで、君のサイトの全ページで`clipboard.js`が使えるようになったよ！

### ステップ2: ボタンを追加して動かすJavaScriptコード

次に、ページが読み込まれたときに、すべてのコードブロックを探して、その隣にコピーボタンを自動で追加するJavaScriptを書くよ。
さっきの`clipboard.js`を読み込むscriptタグのさらに下に、このコードを追加しよう。

```html
<!DOCTYPE html>
<html>
  <head>
    </head>
  <body>
    {{ content }}

    <script src="https://cdn.jsdelivr.net/npm/clipboard@2.0.11/dist/clipboard.min.js"></script>
    
    <script>
      // ページの読み込みが終わったら実行する処理
      document.addEventListener('DOMContentLoaded', () => {
        // Jekyllが生成するコードブロックは<div class="highlight">...</div>で囲まれていることが多い
        const highlightBlocks = document.querySelectorAll('div.highlight');

        // 見つかったすべてのコードブロックに対して処理を行う
        highlightBlocks.forEach((block, index) => {
          // コピーボタンのHTML要素を新しく作る
          const copyButton = document.createElement('button');
          copyButton.className = 'copy-button'; // CSSでデザインするためのクラス名
          copyButton.textContent = 'Copy';     // ボタンに表示するテキスト

          // ボタンにユニークなIDを付ける（どのコードをコピーするか判別するため）
          const codeId = `code-block-${index}`;
          // コピー対象のコードブロック本体（<pre><code>...</code></pre>）を探す
          const codeElement = block.querySelector('pre');
          if (codeElement) {
            codeElement.id = codeId; // コードブロックにもIDを付ける
            copyButton.setAttribute('data-clipboard-target', `#${codeId}`); // ボタンにコピー対象のIDを教える
          }

          // 作成したボタンをコードブロックの直前に追加する
          block.insertBefore(copyButton, block.firstChild);
        });

        // clipboard.jsを初期化する
        const clipboard = new ClipboardJS('.copy-button');

        // コピーが成功したときのイベント
        clipboard.on('success', (e) => {
          // ボタンのテキストを一時的に「Copied!」に変更する
          e.trigger.textContent = 'Copied!';
          // 2秒後にもとの「Copy」に戻す
          setTimeout(() => {
            e.trigger.textContent = 'Copy';
          }, 2000);
          // コピー後にボタンのフォーカスを外す
          e.clearSelection();
        });

        // コピーが失敗したときのイベント
        clipboard.on('error', (e) => {
          e.trigger.textContent = 'Error!';
           setTimeout(() => {
            e.trigger.textContent = 'Copy';
          }, 2000);
        });
      });
    </script>
    </body>
</html>
```

このコードが何をしているか、コメントに詳しく書いておいたよ！
これで、機能としてはもう完成だね！

### ステップ3（おまけ）: ボタンの見た目を整える

このままだと、ボタンがただの四角いボタンでちょっと味気ないから、CSSで見た目をカッコよくしてみよう！

君のリポジトリに `assets/css/style.scss` みたいなCSSファイルがあるかな？もしあれば、そこに下のCSSコードを追記しよう。なければ、新しくファイルを作って、`_layouts/default.html`の`<head>`セクションで読み込んであげてね。

```css
/* assets/css/style.scss など */

/* コードブロックを囲むdiv.highlightを相対位置の基準にする */
div.highlight {
  position: relative;
}

/* コピーボタンのスタイル */
.copy-button {
  position: absolute; /* 親要素(div.highlight)の右上に配置する */
  top: 0.5em;
  right: 0.5em;
  padding: 5px 10px;
  font-size: 12px;
  color: white;
  background-color: #555;
  border: 1px solid #333;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.2s; /* ホバー時の色の変化を滑らかに */
}

/* マウスが乗ったときのスタイル */
.copy-button:hover {
  background-color: #333;
}

/* ボタンが押されたときのスタイル */
.copy-button:active {
  background-color: #000;
}
```

これで、コードブロックの右上にいい感じのコピーボタンが表示されるはずだよ！

どうかな？この手順でできそうかな？
もし`_layouts`フォルダの構造とか、CSSファイルの場所とかで迷ったら、君のリポジトリの構成を教えてくれたら、もっと具体的にアドバイスできるよ！いつでも聞いてね！

***
> 公式テーマのリポジトリから`_layouts/default.html`と`assets/css/style.scss`をコピーしてきてそこに書き加えたらうまくいきました。
