# Neovim 設定

LazyVim をベースにした Neovim 設定です。

## 基本

- ベース: [LazyVim](https://github.com/LazyVim/LazyVim)
- plugin manager: [lazy.nvim](https://github.com/folke/lazy.nvim)
- leader key: `Space`
- local leader key: `\`

## ローカル設定

`lua/config/options.lua` では、tmux 内で Neovim を使っているときのカーソル形状を調整しています。

## Plugin

### nightfox.nvim

設定ファイル: `lua/plugins/colorscheme.lua`

- `EdenEast/nightfox.nvim` を追加しています。
- LazyVim の colorscheme を `nordfox` にしています。
- UI が描画される前に colorscheme を読み込むため、遅延読み込みせず優先度高めで読み込んでいます。

### lsp_signature.nvim

設定ファイル: `lua/plugins/lsp_signature.lua`

- 関数呼び出し中に引数やシグネチャのヒントを表示します。
- `InsertEnter` で読み込まれます。
- plugin のデフォルト設定を使っています。

### lualine.nvim

設定ファイル: `lua/plugins/lualine.lua`

- statusline を表示する plugin です。
- `nvim-web-devicons` を依存として使い、ファイル種別などのアイコン表示に対応しています。
- `require("lualine").setup()` をデフォルト設定で呼んでいます。

### nvim-autopairs

設定ファイル: `lua/plugins/nvim-autopairs.lua`

- LazyVim 標準の `mini.pairs` を無効化しています。
- 代わりに `windwp/nvim-autopairs` を使っています。
- `InsertEnter` で読み込まれます。
- 括弧やクォートなどのペア入力を補助します。

### snacks.nvim

設定ファイル: `lua/plugins/snacks.lua`

- LazyVim 標準の Snacks 統合を使っています。
- explorer picker が右側に出るように layout を変更しています。
- よく使う keymap:
  - `Ctrl-/`: project root で terminal を開く、または focus する
  - `Space f t`: project root で terminal を開く
  - `Space f T`: 現在の working directory で terminal を開く
  - `Space g g`: Lazygit を開く

### winresizer

設定ファイル: `lua/plugins/winresizer.lua`

- `simeji/winresizer` を追加しています。
- Neovim の window サイズ変更、移動、focus 切り替えを対話的に行うための plugin です。
- 基本操作:
  - `Ctrl-e`: resize mode を開始
  - `h`, `j`, `k`, `l`: window サイズを変更
  - `e`: resize / move / focus mode を切り替え
  - `Enter`: 確定して終了
  - `q`: キャンセル

## 補足

`lua/plugins/example.lua` は LazyVim のサンプル用ファイルです。先頭で空の spec を返しているため、実際には plugin を読み込んでいません。
