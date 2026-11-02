# WezTerm Config Studio

WezTermの見た目をブラウザーで調整し、`wezterm.lua` とStarship用の配色設定を生成するツールです。React・TypeScript・Tailwind CSS・Viteで構築しています。

[ホスト版を開く](https://wezterm-config-studio.jadebrink16.chatgpt.site)（所有者限定・サインインが必要）

## できること

- **12種類の配色プリセット**：Tokyo Night、Catppuccin、Nord、Dracula、Gruvbox、Ocean、Sakura、Ember、Forest、Mono、Lavender Light、Paper
- プリセットで本文・カーソル・タイトルバー・タブ・ステータス文字・プロンプトを一括配色
- フォント、文字サイズ、余白、カーソル形状、タブ表示を調整
- 本文・タブの背景透過と、タイトルバーの連動／独立した不透明度設定
- ディレクトリ／Gitブランチのプロンプト文字色・背景色を個別に指定
- 任意のディレクトリ／Git情報をWezTermの右ステータス欄に表示
- Luaのプレビュー、コピー、`wezterm.lua` ダウンロード
- Starship配色設定の表示・コピー

プリセットを選んでもフォントや不透明度は変更しません。色を個別編集すると「custom」と表示し、プリセットを再選択すると全体配色を戻せます。

## ローカル起動

Node.js 22 LTSとnpmを推奨します。

```sh
git clone https://github.com/crewzusa0905/wezterm-config-studio.git
cd wezterm-config-studio
npm ci
npm run dev
```

ターミナルに表示されるURL（通常 `http://localhost:5173/`）を開きます。非公開リポジトリのcloneにはGitHub認証が必要です。

```sh
npm run build
```

ビルドはTypeScript検査の後にViteを実行し、`dist/` を生成します。専用のテスト／lintスクリプトはありません。`node_modules/` と `dist/` は生成物としてGit管理から除外しています。

## 設定の適用

### WezTerm

1. プリセットを選び、プレビューで見た目を調整します。
2. **Export configuration** で `wezterm.lua` を保存します。
3. 既存設定をバックアップし、ファイルを `~/.config/wezterm/wezterm.lua` に配置します。
4. 選択したフォントをインストールし、WezTermで設定を再読み込みします。

設定ファイルを `~/.config/wezterm.lua` に置く構成ではなく、上記のサブディレクトリを使用してください。別の優先設定ファイルや起動時の `--config-file` がある場合は、WezTermが実際に読み込むファイルも確認してください。

ブラウザーからローカルの設定ファイルを書き換えたり、フォントをインストールしたりすることはありません。

### Starshipプロンプト

WezTermのLuaとシェルのプロンプトは別の設定です。プロンプトの配色にはStarshipの導入とシェル初期化が必要です。

1. [Starshipの導入手順](https://starship.rs/guide/)に従ってインストールします。
2. **Prompt colors → Copy Starship colors** を選びます。
3. コピーした各モジュールの設定を `~/.config/starship.toml` にマージします。同じ `[directory]` などのセクションを重複させず、既存セクションの項目を更新してください。
4. zshの場合、`~/.zshrc` の末尾で初期化します。

```sh
export STARSHIP_CONFIG="$HOME/.config/starship.toml"
if command -v starship >/dev/null 2>&1; then
  eval "$(starship init zsh)"
fi
```

新しいタブを開くと初期化が反映されます。ディレクトリとGit情報だけに絞りたい場合は、`starship.toml` の**セクション外の先頭部分**に次を設定します。

```toml
format = '$directory$git_branch$git_status$line_break$character'
```

GitブランチはGitリポジトリ内で表示されます。サイトが出力するStarship設定は配色のみで、この表示形式や斜めの区切り文字を自動設定するものではありません。

## プレビューと実際の表示の違い

- ターミナルはブラウザー上の近似表示です。ディレクトリ・ブランチ・OSなどはサンプルで、利用者の環境を読み取りません。
- 選択フォントがブラウザーにない場合は代替フォントで表示します。
- タイトルバーの透過はOSとWezTermのバージョン・描画方式に依存します。出力するアルファ値がネイティブタイトルバーで反映される保証はありません。macOSのシステムメニューバーは対象外です。
- 必要に応じて[統合ウィンドウボタン](https://wezterm.org/config/lua/config/window_decorations.html)を使う構成を別途調整してください。サイトの出力は標準で `use_fancy_tab_bar = false` です。
- 右ステータス欄のGit情報は、ローカルpaneとPATH上のGitを使用します。SSH先のGit状態には対応しません。`*` は追跡済みファイルの変更を示し、未追跡ファイルは含みません。
- 右ステータス欄はタブバーが表示されている場合に見えます。ディレクトリ取得にはシェル連携が役立ちます。
- 編集内容の永続保存機能はありません。必要な設定はコピーまたはダウンロードしてください。

## 日本語フォント

日本語が崩れる場合、[フォールバックフォント](https://wezterm.org/config/fonts.html)を設定できます。macOSでは、生成Luaの `config.font` を例えば次のように置き換えます。

```lua
config.font = wezterm.font_with_fallback {
  'JetBrains Mono',
  { family = 'Hiragino Kaku Gothic ProN', weight = 'Light' },
  'Symbols Nerd Font Mono',
  'Noto Color Emoji',
}
config.line_height = 1.15
```

他のOSでは、インストール済みの日本語フォント名に変更してください。この調整はサイトの標準出力には含まれません。

## 構成

| ファイル | 内容 |
| --- | --- |
| `src/main.tsx` | プリセット、プレビュー、Lua／Starship生成、操作UI |
| `src/style.css` | Tailwind CSS読み込みと画面スタイル |
| `vite.config.ts` | ViteとTailwind CSSの連携 |
| `.openai/hosting.json` | SitesのプロジェクトIDと静的配信先 |
| `package-lock.json` | 依存関係の固定 |

ブラウザー内で完結する静的アプリです。バックエンド、データベース、APIキーは不要です。`dist/` を静的ホスティングへ配置できます。既存のホスト版は所有者限定のまま運用しています。

実際の個人用WezTerm／Starship設定は、別リポジトリ [terminal-dotfiles](https://github.com/crewzusa0905/terminal-dotfiles) で管理しています。
