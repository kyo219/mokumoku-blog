# mokumoku-blog

勉強記録を公開するブログです。[Quarto](https://quarto.org/) で書き（Python・Julia のコードを実行可能）、main に push すると GitHub Actions がビルドして GitHub Pages に公開します。

公開URL: https://kyo219.github.io/mokumoku-blog/

## 構成

```
_quarto.yml          サイト全体の設定（タイトル、ナビ、目次・コード表示など）
theme.scss           配色・フォント・余白
index.qmd            トップページ（記事一覧・カテゴリ絞り込み）
about.qmd            About ページ
posts/<slug>/        1記事 = 1フォルダ
  index.qmd          記事本文（Markdown + Python または Julia）
  *.png, *.csv など   その記事で使うファイル
posts/_template/        記事テンプレート（Python 用。_ で始まるフォルダはビルドされない）
posts/_template-julia/  記事テンプレート（Julia 用）
pyproject.toml       Python の依存関係（uv.lock で固定）
Project.toml         Julia の依存関係（Manifest.toml で固定）
.github/workflows/publish.yml   自動公開の設定
_site/               ビルド結果（git 管理外）
```

## 初回セットアップ

1. [Quarto](https://quarto.org/docs/get-started/)（CI と同じ 1.11.9 推奨）、[uv](https://docs.astral.sh/uv/)、[Julia](https://julialang.org/downloads/)（1.13）をインストール
2. Python と Julia の環境を作る

```sh
uv sync
julia --project=. -e 'using Pkg; Pkg.instantiate()'
```

Julia 記事の初回ビルド時、Quarto が実行用パッケージ（QuartoNotebookRunner）を自動で用意します。

## 記事の追加

```sh
cp -r posts/_template posts/my-new-post          # Python で書く記事
cp -r posts/_template-julia posts/my-julia-post  # Julia で書く記事
```

1. `posts/my-new-post/index.qmd` の `title`・`date`・`description`・`categories` を書き換える
2. 本文を書く。フォルダ名がそのまま URL になる（`/posts/my-new-post/`）
3. 画像や CSV は同じフォルダに置き、`![説明](image.png)`、`pd.read_csv("data.csv")` のように相対パスで参照
4. テンプレートは `draft: true`（下書き）になっている。公開するときはこの行を削除する

コードと出力の表示は、セル先頭の `#|` オプションで切り替えます。

- `#| echo: false` … コードを隠して結果だけ表示
- `#| output: false` … 結果を隠してコードだけ表示
- `#| include: false` … 実行するが何も表示しない
- `#| eval: false` … 実行せずコードだけ表示
- `#| code-fold: true` … コードを折りたたむ

Julia 記事は先頭の YAML に `engine: julia` を書きます（テンプレートに記載済み）。
[Latexify.jl](https://github.com/korsbo/Latexify.jl) の `latexify` / `@latexify` / `@latexdefine` の結果をセルの最後に置くと、数式として描画されます（例: `posts/julia-latexify/index.qmd`）。

## 記事の修正

- 本文の修正: `posts/<slug>/index.qmd` を編集する
- 更新日を表示したい場合: `date` は変えずに `date-modified: 2026-12-01` を追加する
- 非公開に戻す: `draft: true` を追加する（一覧・検索・サイトマップから消える）
- 記事を削除する: `posts/<slug>/` フォルダごと削除する
- サイト全体の見た目・設定: `theme.scss`、`_quarto.yml` を編集する
- Python パッケージを追加する: `uv add numpy`（`pyproject.toml` と `uv.lock` が更新される。両方コミットする）
- Julia パッケージを追加する: `julia --project=. -e 'using Pkg; Pkg.add("Plots")'`（`Project.toml` と `Manifest.toml` を両方コミットする）

## 手元での確認

```sh
uv run quarto preview
```

ブラウザが開き、ファイルを保存するたびに自動で更新されます（終了は Ctrl+C）。下書きもここでは表示されます。

公開時と同じ成果物を作る場合:

```sh
uv run quarto render      # _site/ に出力される
```

`uv run` を付けるのは、記事内の Python を `.venv` の環境で実行させるためです。Julia 記事はリポジトリ直下の `Project.toml` が自動で使われます。

## 反映（公開）方法

```sh
git add .
git commit -m "記事を追加: my-new-post"
git push origin main
```

push すると GitHub Actions（Publish site）が Python・Julia の実行 → ビルド → GitHub Pages への公開まで自動で行います。
進行状況と失敗時のログはリポジトリの **Actions** タブで確認できます。数分後に公開URLへ反映されます。

### 最初の1回だけ必要な設定

1. GitHub のリポジトリで **Settings → Pages → Build and deployment → Source** を **GitHub Actions** にする
2. main に push する（または **Actions → Publish site → Run workflow** で手動実行）

### 公開URLとパス

- 通常のリポジトリ: `https://<ユーザー名>.github.io/<リポジトリ名>/`
- `<ユーザー名>.github.io` という名前のリポジトリ: `https://<ユーザー名>.github.io/`

サイト内リンクは相対パスで出力されるため、リポジトリ名に合わせたパス設定は不要です。
リポジトリ名の変更や独自ドメインを使う場合は、`_quarto.yml` の `website.site-url` を実際の URL に変更してください。
