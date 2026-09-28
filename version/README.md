# LazyVim の依存環境と導入手順

このリポジトリの設定を別のMacで使うときに必要な環境をまとめています。
確認日: 2026-09-28。動作確認環境: macOS 26.6.2 / Apple Silicon（arm64）。

## 必要なツール

「確認済みバージョン」は今回動作した組み合わせです。最低バージョンや完全固定の指定ではありません。最低条件が確認できたものは「用途・条件」に記載しています。

| ツール | 確認済みバージョン | 用途・条件 | 導入元 |
| --- | --- | --- | --- |
| Neovim | 0.12.5（Homebrew: `0.12.5_1`） | エディタ本体。現在のロックに含まれる `nvim-treesitter` は **0.12.0以上**が必要。LuaJIT対応版を使う | Homebrew `neovim` |
| Git | Apple Git 2.50.1 | プラグインの取得。LazyVimの最低条件は2.19.0 | Xcode Command Line Tools |
| ripgrep（`rg`） | 15.2.0 | Snacksの全文検索、grug-farの検索・置換 | Homebrew `ripgrep` |
| fd | 10.5.0 | Snacksのファイル検索・エクスプローラー | Homebrew `fd` |
| Tree-sitter CLI | 0.27.0 | パーサーの生成・ビルド。現在のロックでは0.26.1以上。npm版ではなくHomebrew版を使う | Homebrew `tree-sitter-cli` |
| Cコンパイラ（`cc` / Clang） | Apple Clang 21.0.0 | Tree-sitterパーサーのコンパイル | Xcode Command Line Tools |
| curl | 8.7.1 | プラグイン用バイナリ・パーサーなどの取得 | macOS標準 |
| tar / gzip / unzip | bsdtar 3.5.3 / Apple gzip 479 / UnZip 6.00 | ダウンロードしたアーカイブの展開 | macOS標準 |
| Lazygit | 0.65.1 | `Space g g` から使うGit画面。Neovimの起動自体には必須ではない | Homebrew `lazygit` |

HomebrewがNeovimと一緒に導入するLuaJITなどのライブラリは、個別にインストールする必要はありません。

## プラグインと補助ツールのバージョン

プラグインの正確なコミットは [`../lazy-lock.json`](../lazy-lock.json) を参照してください。
今回の確認では、有効なプラグインすべての実際のコミットとロックの一致を確認しています。

- LazyVim: `v16.0.0`（`c10948c50b18fae7f256433afdef09e432410480`）
- lazy.nvim: `11.17.5`
- blink.cmp: `v1.10.2`。Apple Silicon用の配布済みバイナリを使用します。

次のツールはMasonが `~/.local/share/nvim/mason/` に導入します。Homebrewでの重複導入は不要です。

| Masonのパッケージ名 | 確認済みバージョン | 用途 |
| --- | --- | --- |
| `lua-language-server` | 3.19.1 | Luaの補完・診断・定義参照 |
| `stylua` | 2.5.2 | Luaのコード整形 |
| `shfmt` | 3.14.1 | シェルスクリプトの整形 |

Masonのツールは `lazy-lock.json` の固定対象ではありません。上記の版を明示して入れる場合は、Neovimで次を実行します。

```vim
:MasonInstall lua-language-server@3.19.1 stylua@2.5.2 shfmt@3.14.1
```

Tree-sitterのパーサーは、ロックされた `nvim-treesitter` の定義に従って導入します。今回、設定で要求される全パーサーの導入を確認しました。

## macOSへの導入

### 1. コマンドの準備

Homebrewが使える状態で、次を実行します。

```sh
xcode-select -p
git --version
brew install neovim ripgrep fd tree-sitter-cli lazygit
```

`xcode-select -p` が失敗する場合は `xcode-select --install` を実行して、Command Line Toolsのインストールを完了してください。Gitが2.19.0以上なら、そのまま使用できます。

Apple SiliconのHomebrewの標準パスは `/opt/homebrew/bin` です。コマンドが見つからない場合は、`~/.zprofile` に次の設定があるか確認します。

```sh
eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

### 2. この設定を読み込ませる

以下はリポジトリを `~/working/tools-config/nvim_git` に置いた場合の例です。実際の配置先に合わせて変更してください。
既存の `~/.config/nvim` がある場合は、先に別名でバックアップします。

```sh
mkdir -p "$HOME/.config"
ln -s "$HOME/working/tools-config/nvim_git" "$HOME/.config/nvim"
readlink "$HOME/.config/nvim"
```

### 3. 保存済みのプラグイン構成を復元する

初回起動では未導入の依存プラグインが取得され、ロックファイルが書き換わる場合があります。起動前にロックを一時保存します。

```sh
cd "$HOME/working/tools-config/nvim_git"
LAZYVIM_LOCK_BACKUP="$(mktemp)"
cp lazy-lock.json "$LAZYVIM_LOCK_BACKUP"
nvim
```

初回のプラグイン・Masonツール・パーサーの導入完了を待ってから `:qa` で終了します。取得・ビルド途中で終了しないでください。
続けて同じシェルで、元のロックに戻して復元します。

```sh
cp "$LAZYVIM_LOCK_BACKUP" lazy-lock.json
nvim --headless '+Lazy! restore' +qa
nvim
```

Neovimで `:TSUpdate` を実行し、パーサーの更新完了を待ちます。通常の再現用セットアップでは `:Lazy restore` を使います。`:Lazy sync` はプラグインの更新も含みます。

### 4. 動作確認

```sh
nvim --version
git --version
rg --version
fd --version
tree-sitter --version
lazygit --version
```

NeovimでLuaファイルを開き、次を確認します。

```vim
:checkhealth lazyvim mason nvim-treesitter blink.cmp
:LspInfo
:ConformInfo
:Mason
```

Luaファイルで `lua_ls` が接続され、`stylua` が使用可能になっていることを確認してください。

## ターミナルと追加依存の範囲

- True Color対応のターミナルを使用します。このMacにはGhostty 1.3.1が導入されています。
- アイコン表示にNerd Fontを指定する場合はv3以降を使用します。ターミナル側のフォント設定です。
- macOSのクリップボードには標準の `pbcopy` / `pbpaste` を使用します。
- `lazyvim.json` のExtrasは空です。`lua/plugins/example.lua` は無効なサンプルなので、その中の言語サーバーやツールを導入対象に含めません。
- 現在の構成はSnacks Pickerを使います。`fzf` の警告や、Masonの未使用言語向けランタイムの警告だけを理由に追加インストールする必要はありません。言語のExtrasを有効化したときに、その依存を確認します。

## 要件の確認元

- [LazyVimの基本要件](https://www.lazyvim.org/)
- [このロックで使うnvim-treesitterの要件](https://github.com/nvim-treesitter/nvim-treesitter/blob/4916d6592ede8c07973490d9322f187e07dfefac/README.md#requirements)
- [この版のLazyVimのLSP・Mason設定](https://github.com/LazyVim/LazyVim/blob/c10948c50b18fae7f256433afdef09e432410480/lua/lazyvim/plugins/lsp/init.lua)
