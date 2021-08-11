---
title: '【VS Code】エディター（タブ）を左右に移動するショートカットを設定する'
emoji: '↔'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [VSCode]
published: true
---

出来るだけキーボード上で操作を完結させたいあなたへ。

VS Code を使っていて、タブを左右に移動させたくなることがあります。
これまではマウスを使っていたのですが、コマンドを発見したのでキーボードショートカットを設定してみました。

# 結論

```json:keybindings.json
[
  ...
  {
    "key": "ctrl+cmd+[",
    "command": "workbench.action.moveEditorLeftInGroup"
  },
  {
    "key": "ctrl+cmd+]",
    "command": "workbench.action.moveEditorRightInGroup"
  },
  ...
]
```

# 手順

## 1. (Optional) コマンド名を調べる

コマンドパレットを開いて、

![](/images/vscode-move-editor-commands/command-palette.png)

関係しそうな単語を打ち込んでみます。今回は move left と入れてみました。

![](/images/vscode-move-editor-commands/cp-move-left.png)

ヒット！
"View: Move Editor Left" というコマンドが使えそうです。

どうやらデフォルトのショートカットとして、 `Command + K Shift + Command + <-` が設定されているようです。
使いにくいので、カスタムショートカットを設定することにします。

## 2. (Optional) 設定ファイル上のコマンド名を調べる

コマンドパレットから、デフォルトのショートカットが設定されているファイルを開きます。

![](/images/vscode-move-editor-commands/cp-default-keyboard-shortcuts.png)

先程調べたコマンド名をキャメルケースにしてファイル内検索します。

![](/images/vscode-move-editor-commands/cp-find-commands.png)

以下のコマンドで間違いなさそうです。

```json:Default Keybindings
  { "key": "cmd+k shift+cmd+left",  "command": "workbench.action.moveEditorLeftInGroup" },
  { "key": "cmd+k shift+cmd+right", "command": "workbench.action.moveEditorRightInGroup" },
```

:::message
大抵はコマンドのキャメルケース化で見つかりますが、見つからない場合はデフォルトショートカットで検索したり、GitHub の VS Code のリポジトリからコマンド名で検索をかけたりします。
ただし、ショートカットを設定できるコマンドが用意されていないこともあります。そのようなコマンドは、コマンドパレットや GUI からしか呼び出せません。
:::

## 3. カスタムショートカットを設定する

コマンドパレットから、ショートカットの設定ファイル（JSON）を開きます。

![](/images/vscode-move-editor-commands/cp-open-keyboard-shortcuts.png)

適当な場所に以下を挿入します。
今回は左右の移動にそれぞれ `Ctrl + Command + [` と `Ctrl + Command + ]` を割り当てました。

```json:keybindings.json
  {
    "key": "ctrl+cmd+[",
    "command": "workbench.action.moveEditorLeftInGroup"
  },
  {
    "key": "ctrl+cmd+]",
    "command": "workbench.action.moveEditorRightInGroup"
  },
```

# 環境

- macOS Big Sur 11.2.1
- VS Code 1.59.0
