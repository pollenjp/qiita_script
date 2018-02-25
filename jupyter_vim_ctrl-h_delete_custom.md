# < Ctrl-h > delete カスタム (jupyter-vim-binding)

Jupyter上でVim操作を扱えるプラグインである <a href="https://github.com/lambdalisue/jupyter-vim-binding">jupyter-vim-binding</a>。使われている方も多いと思われます。
しかし、普段VimでInsertモードの状態で一つ前の文字を消したい(Backspace)ときに標準ではJupyter上で機能しません。
自分はブラウザの機能が動作します。（履歴の表示など）

この症状を解決するためにキーのカスタムをしましたので以下に載せて置きます。

# 準備
なお、`~/.jupyter/custom/`のディレクトリが存在しない方は新たに作成してください。

```
$ mkdir -p ~/.jupyter/custom
```

# コード
そして、`~/.jupyter/custom/custom.js`に新たに以下のコードを書きます。

```
// ~/.jupyter/custom/custom.js

require([
    'nbextensions/vim_binding/vim_binding',
], function() {
  /***** start *****/
  // 挿入モードで<Ctrl-h>でBackspaceを実行
  CodeMirror.Vim.defineAction('[i]<C-h>', function(cm) {
    // カーソルの位置を{行, 文字数}で返す
    var head = cm.getCursor();
    CodeMirror.Vim.handleKey(cm, '<Esc>');      // Inesrt Modeから出る

    // 現在のカーソルがある行の文字数
    var last = cm.getLine(head.line).length;

    CodeMirror.Vim.handleKey(cm, 'x');          // 最後の行を削除する
    if (head.ch === last){                  // カーソルが最後の文字か
      CodeMirror.Vim.handleKey(cm, 'a');        // 後ろに挿入
    } else {
      CodeMirror.Vim.handleKey(cm, 'i');        // 前に挿入
    }
  });
  CodeMirror.Vim.mapCommand("<C-h>", "action", "[i]<C-h>", {}, { "context": "insert" });
  // ref
  //   https://github.com/lambdalisue/jupyter-vim-binding/wiki/Customization
  /***** end *****/
});
```

ただ、上のコードでは一度Escによって挿入モードを出ているので、もし<Ctrl-h>で連続して消してUndo("u")する場合は一文字ずつもどされる。

※ Backspaceのキーマップを見つけられなかったので知っている方は教えてください。おねがいします。

