---
title: 記事テスト2
layout: page
---

# GitHub Pagesで記事サイトを作る完全ガイド(改訂版)

Mac標準のターミナルが使えるレベルの初心者向けに、GitHubアカウント作成からJekyllテーマ(Chirpy)でかっこよく記事サイトを公開するまでをまとめました。
**実際につまずきやすいポイントをあらかじめ回避する手順**になっています。後半に「記事の運用ルール」「複数のMacでの作業」「GitHub Desktop(GUIツール)を使う場合」も掲載しています。

---

## 全体の流れ

1. GitHubアカウント作成
2. Personal Access Token(PAT)の発行 ← 最初にここまでやっておくのがポイント
3. 必要なツールのインストール(Git, rbenv, Ruby)
4. リポジトリ作成とローカル環境準備
5. デザインテーマの導入(Chirpy)
6. サンプル記事の作成
7. ローカルで確認
8. GitHub Pagesへ公開
9. 記事の作成・編集の実践ルール
10. 複数のMacで運用する場合

---

# Part 1: 構築(ターミナルを使う手順)

## 1. GitHubアカウント作成

1. https://github.com/ にアクセス
2. 右上「Sign up」をクリック
3. メールアドレス → パスワード → ユーザー名を入力(このユーザー名がサイトURLに使われるので、シンプルで覚えやすいものを推奨)
4. 認証(パズル等)を完了し、メールに届いた確認コードを入力

## 2. Personal Access Token(PAT)を発行する

GitHubは2021年からパスワード認証を廃止しているため、ターミナルからpushする際は代わりに「トークン」を使います。**後で作り直す手間を省くため、最初に必要な権限をまとめて付けておきます。**

1. ログイン状態で https://github.com/settings/tokens にアクセス
2. 「Generate new token」→「Generate new token (classic)」を選択
3. 設定内容:
   - Note: `mac-terminal` など分かりやすい名前
   - Expiration: `90 days` など任意
   - スコープ(権限)は以下**両方**にチェック
     - **`repo`**(リポジトリの読み書き)
     - **`workflow`**(GitHub Actionsのワークフローファイルを更新するために必須。Chirpyはこれが無いとpushで弾かれます)
4. 「Generate token」をクリック
5. 表示されたトークン(`ghp_〜`で始まる文字列)を**その場でコピー**して、メモ帳などに一時保存しておく
   - このページを離れると二度と表示されません

## 3. 必要なツールのインストール

ターミナル(Launchpad → その他 → ターミナル)を開いて以下を実行します。

### Homebrewのインストール(未導入の場合)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Gitのインストール確認
```bash
git --version
```
バージョンが表示されなければ以下でインストール。
```bash
brew install git
```

### Git初期設定(初回のみ)
```bash
git config --global user.name "あなたの名前"
git config --global user.email "GitHub登録メールアドレス"
git config --global credential.helper osxkeychain
```
(最後の1行で、トークン入力を一度行えば以降は聞かれなくなります)

### rbenvでRubyを管理する(重要)

Macに `brew install ruby` で直接Rubyを入れると、**最新すぎるバージョン(例: 4.0系)がインストールされ、Jekyllテーマが要求する古いバージョン(3.1〜3.3系)と衝突してエラーになります。** そのため、プロジェクトごとにバージョンを切り替えられる `rbenv` を使います。

```bash
brew install rbenv ruby-build
echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc
source ~/.zshrc
```

Chirpyが安定して動くバージョンをインストールします(時間がかかります)。
```bash
rbenv install 3.2.3
```

インストールできたか確認:
```bash
rbenv versions
```
`3.2.3` が一覧に表示されればOKです(この時点ではまだ `* system` のままで問題ありません。プロジェクトフォルダごとに指定します)。

### Jekyll & Bundlerのインストール
```bash
gem install jekyll bundler
```

## 4. リポジトリ作成とローカル環境準備

### GitHub側でリポジトリ作成

1. GitHub右上の「+」→「New repository」
2. Repository name に `ユーザー名.github.io` と入力(例: `hiroshi.github.io`)
   - これにするとサイトが `https://ユーザー名.github.io` という短いURLになります
3. 「Public」を選択
4. **「Add a README file」には⚠️チェックを入れない**(後述のテーマ導入時に履歴の衝突(コンフリクト)が起きる原因になるため、空のリポジトリのまま作成します)
5. 「Create repository」

## 5. デザインテーマの導入(Chirpy)

かっこいいデザインで人気の「Chirpy」テーマを使います。ダークモード対応でブログ・技術記事向けの見た目です。

⚠️ 以前は `jekyll-theme-chirpy` リポジトリの `starter` ブランチを使う方法が案内されていましたが、**現在は `chirpy-starter` という専用リポジトリを直接使う方式に変わっています。** `tools/init.sh` のようなスクリプトは不要です。

### Chirpy Starterを取得
```bash
cd ~/Desktop
git clone https://github.com/cotes2020/chirpy-starter.git chirpy-site
cd chirpy-site
```

### このフォルダだけRuby 3.2.3を使うよう指定
```bash
rbenv local 3.2.3
ruby -v
```
`ruby 3.2.3` と表示されることを確認してください(違う場合はターミナルを再起動)。

### 取得元(remote)を自分のリポジトリに向け直す

cloneした直後は「コピー元(cotes2020側)」を指したままなので、pushする前に必ず自分のリポジトリに向け直します。**これを忘れると `403 Permission denied` になります。**

```bash
git remote set-url origin https://github.com/ユーザー名/ユーザー名.github.io.git
git remote -v
```
自分のリポジトリのURLが表示されればOKです。

### 依存パッケージをインストール
```bash
bundle install
```

### サイト基本情報を設定
```bash
open -e _config.yml
```
- `title:` サイトタイトル
- `tagline:` サブタイトル
- `url:` `https://ユーザー名.github.io`
- `github: username:` 自分のユーザー名

## 6. サンプル記事の作成

記事は `_posts` フォルダに、`年-月-日-タイトル.md` という名前で置きます。

```bash
open -e "_posts/$(date +%Y-%m-%d)-hello-world.md"
```

開いたファイルに以下を貼り付けます。

```markdown
---
title: はじめての記事
date: 2026-09-14 10:00:00 +0900
categories: [お知らせ]
tags: [ブログ開設]
---

このブログを始めました。研究や授業に関する内容を少しずつ書いていく予定です。

## これから書きたいこと

- ゲーミフィケーションを使った学習支援の研究メモ
- 授業で使っているツールの紹介
- 気になった技術のメモ
```

同じ要領であと2〜3記事作っておくと、公開したときにサイトらしく見えます。

## 7. ローカルで確認

```bash
bundle exec jekyll serve
```

ターミナルに表示される `http://127.0.0.1:4000` にブラウザでアクセスすると、実際の見た目を確認できます。`Ctrl + C` で停止します。

## 8. GitHub Pagesへ公開

### 変更をGitHubにアップロード(push)

```bash
git add .
git commit -m "Chirpyテーマ導入とサンプル記事追加"
git branch -M main
git push origin main
```

初回pushでは、Username / Password の入力を求められます。

- Username: GitHubのユーザー名
- Password: **手順2で発行したトークン**(`ghp_〜`。GitHubアカウントのパスワードではない)

これで `credential.helper osxkeychain` の設定により、次回以降は自動的に認証されます。

### Pages設定を有効化

1. GitHubのリポジトリページ → 「Settings」タブ
2. 左メニュー「Pages」
3. 「Build and deployment」の「Source」を **「GitHub Actions」** に設定

   もし候補として「GitHub Pages Jekyll」や「Static HTML」の「Configure」ボタンしか出てこない場合、それはまだリポジトリ側に `.github/workflows/pages-deploy.yml` が認識されていない状態です。一度 `.github/workflows/` フォルダがpushされているか確認してください。
   ```bash
   ls .github/workflows/
   git status
   ```
   もしそこに何かファイルが残っている(pushされていない)場合は、改めて追加してpushします。
   ```bash
   git add .github
   git commit -m "add pages workflow"
   git push origin main
   ```

4. リポジトリの「Actions」タブでビルドが緑のチェックになるのを待つ(数分)
5. 完了後、`https://ユーザー名.github.io` にアクセスすると公開されています

---

# Part 2: 運用(記事の作成・編集、複数PCでの作業)

## 9. 記事の作成・編集の実践ルール

### ファイル名のルール

新しい記事は常に `_posts` フォルダに追加していきます。ファイル名は **`YYYY-MM-DD-タイトル.md`** 形式が必須です(Jekyllの仕様)。この日付部分が記事の並び順を決めます。

```
2026-09-17-hello-world.md
2026-10-01-学会参加報告.md
```

### ファイル名のタイトル部分と、画面に表示されるタイトルは別物

| 項目 | どこで決まるか | 役割 |
|---|---|---|
| 記事の並び順 | ファイル名の日付部分 | 内部処理用 |
| URL | ファイル名のタイトル部分(スラッグ) | URLに使われる(例: `/posts/hello-world/`) |
| 画面に見えるタイトル | front matterの `title:` | 実際に表示される文字列 |

つまり、**ファイル名は英数字の簡単なものにしておいて、実際に見せたい見出しは `title:` に日本語で自由に書く**、という使い分けで問題ありません。ファイル名を毎回考えるのが面倒な場合は、日付+時刻だけにしてしまうと楽です。

```bash
open -e "_posts/$(date +%Y-%m-%d-%H%M)-post.md"
```

これを毎回打つのも面倒なら、`~/.zshrc` に以下を追記しておくと `newpost` と打つだけで新規記事ファイルが自動生成されて開きます(パスはご自身の `chirpy-site` の場所に合わせて調整してください)。

```bash
echo 'newpost() {
  open -e "$HOME/Desktop/chirpy-site/_posts/$(date +%Y-%m-%d-%H%M)-post.md"
}' >> ~/.zshrc
source ~/.zshrc
```

### 既存記事の編集

`_posts` フォルダの中の該当ファイルを直接編集するだけです。

```bash
cd chirpy-site
open -e _posts/2026-09-17-hello-world.md
```

編集後は、通常の記事追加時と同じくローカル確認 → commit → push の流れになります。

```bash
bundle exec jekyll serve   # ローカルで見た目確認(任意)
git add .
git commit -m "記事修正: 誤字を訂正"
git push origin main
```

### 未来日付の記事は公開されない

Jekyllはデフォルトで、`date:` が**未来の日時**になっている記事を公開しません(予約投稿扱い)。今すぐ公開したい場合は、日付が現在時刻以前になっているか確認してください。予約投稿機能として使いたい場合は逆に活用できます。

### 「日記っぽさ」を抑えたい場合

Chirpyは標準で記事に日付が大きく表示されるため、ブログというより日記のような印象になりがちです。記事ごとのfront matterに以下を追加すると、日付の表示だけを抑えられます(記録上のデータ自体は残ります)。

```markdown
---
title: 記事タイトル
date: 2026-09-17 10:00:00 +0900
no_date: true
---
```

さらに体系立てた見せ方にしたい場合は、`_posts`(日付必須の仕組み)ではなく、日付に縛られない「Collections」機能を使う方法や、そもそもJust the Docs等のドキュメント系テーマに切り替える方法もあります。気になる場合は別途相談してください。

## 10. 複数のMacで運用する場合

**同じマシンである必要はありません。** Gitでファイルを管理しているだけなので、GitHubさえ経由すれば自宅のMacでも持ち運びのMacBookでも同じように記事を書けます。

### 2台目のMacでの初期セットアップ

1台目(Part 1)で行った準備を、2台目でも一度だけ行います。

1. Homebrew, Git, rbenv, Ruby 3.2.3 のインストール(Part 1の手順2〜3と同じ)
2. Personal Access Tokenの発行(**新しい端末用に別途発行**するのが安全です。同じトークンを使い回すより端末ごとに分けておくと、紛失時に片方だけ無効化できます)
3. 自分のリポジトリをclone

```bash
git clone https://github.com/ユーザー名/ユーザー名.github.io.git
cd ユーザー名.github.io
rbenv local 3.2.3
bundle install
```

### 日常の運用フロー(複数端末を行き来する場合は徹底する)

複数端末で使う場合、**「作業を始める前に最新版を取り込む」「作業が終わったらすぐpushする」**を徹底するのが重要です。これを忘れると、片方の端末での変更がもう片方に反映されず、最悪コンフリクト(競合)が起きます。

**作業を始める前(必ず実行)**
```bash
cd ユーザー名.github.io
git pull origin main
```

**記事を書き終えたら(必ず実行)**
```bash
git add .
git commit -m "記事追加: ◯◯について"
git push origin main
```

「pullを忘れて古い状態のまま編集 → 別端末での変更とかぶってpushでエラー」というのが一番起きやすいトラブルです。作業を始める前は毎回 `git pull` する習慣をつけておくと安心です。

### もっと手軽にしたい場合:GitHubのWeb画面から直接編集

ちょっとした記事の追記や誤字修正だけなら、**Macを使わずブラウザだけで完結**させることもできます。

1. GitHubで自分のリポジトリを開く
2. `_posts` フォルダ → 既存記事を編集する場合はファイルを開いて鉛筆(編集)アイコン、新規作成の場合は「Add file」→「Create new file」
3. ファイル名とMarkdown本文をブラウザ上で直接入力・編集
4. 画面下部で **「Commit directly to the `main` branch」** が選ばれていることを確認(「Create a new branch...」を選ぶとPull Requestが作られるだけで、mainにマージするまで公開されません)
5. 「Commit changes」で保存 → 自動的にPagesが再ビルドされる

出先で少し直したいだけの時などはこちらが便利です。ただし反映まで数分かかることがあり、CDNキャッシュの影響で**ビルドが成功していてもブラウザ側に数分古い内容が表示され続ける**ことがあります。すぐに反映されない場合は、シークレットウィンドウで開くか、5〜10分待ってから再確認してください。

**Web画面で編集した後は、ローカル(Mac)側でも必ず `git pull` してから作業を再開してください。** そうしないと、ローカルの古い状態のまま上書きしてしまう可能性があります。

---

# Part 3: GitHub Desktop(GUIツール)を使う手順

コマンド入力が不安な場合は、GitHub公式GUIアプリ「GitHub Desktop」を使うとpush/pullなどをボタン操作で行えます。ただし **Jekyllのビルド(`bundle install`、`rbenv`でのRuby管理)はターミナルが必要** なため、完全にターミナル無しでは進められない点に注意してください(GitHub Desktopは主に「ファイルの変更をアップロードする」部分を担当します)。

## 1. GitHub Desktopのインストール

1. https://desktop.github.com/ にアクセス
2. 「Download for macOS」をクリックしてインストール
3. 起動後、「Sign in to GitHub.com」でアカウントにログイン(この方式ならPATの手動発行は不要です)

## 2. リポジトリをローカルに取得(Clone)

1. GitHub Desktop左上「File」→「Clone Repository」
2. 「GitHub.com」タブから、事前にGitHub側で作成した `ユーザー名.github.io` を選択
3. 保存先フォルダ(例: デスクトップ)を指定して「Clone」

## 3. Chirpyテーマの導入(ここだけターミナル)

Part 1の「5. デザインテーマの導入」と同様、`chirpy-starter` の取得・remote変更・`bundle install` はターミナルで一度だけ行います。GitHub Desktop用のクローン先フォルダと、`chirpy-starter` の中身を統合する場合は、GitHub Desktopでcloneしたフォルダの中に `chirpy-starter` の中身をコピーしてから作業してください。

## 4. 記事を追加・編集する

Finderで `_posts` フォルダにMarkdownファイルを作成・編集するだけでOKです(GitHub Desktopは編集機能自体は持ちません)。ファイル名や運用のルールはPart 2の「9. 記事の作成・編集の実践ルール」と同じです。

## 5. 変更をアップロード(push)

1. GitHub Desktopを開くと、変更されたファイルの一覧が自動表示される
2. 左下の「Summary」欄にコミットメッセージ(例: 「記事を追加」)を入力
3. 「Commit to main」をクリック
4. 上部の「Push origin」ボタンをクリック → GitHubにアップロード完了

## 6. 複数端末で使う場合

作業を始める前に、上部の「Fetch origin」→「Pull origin」で必ず最新の状態を取り込んでから編集を始めてください。ターミナル版の `git pull` に相当する操作です。

## 7. 公開確認

Push後、ブラウザでGitHubリポジトリの「Settings → Pages」で Source が「GitHub Actions」になっているか確認し、「Actions」タブを開いてビルドが完了すれば `https://ユーザー名.github.io` で確認できます。

---

## つまずきやすいポイント(実例ベース)

| 症状 | 原因・対処 |
|---|---|
| `git clone` で Username/Password を聞かれ認証エラー | パスワード認証は廃止済み。Personal Access Token(`repo`+`workflow`スコープ)を使う |
| `remote: Repository not found` | リポジトリ名の間違い、または未作成。`https://github.com/ユーザー名?tab=repositories` で実際の名前を確認 |
| `bash tools/init.sh` が無い | 旧方式の名残。新しい `chirpy-starter` リポジトリでは不要なステップなのでスキップ |
| `Could not find compatible versions`(Ruby関連) | Homebrewの最新Rubyが新しすぎる。`rbenv` で `3.2.3` 等を導入し `rbenv local` でプロジェクトごとに指定 |
| `rbenv versions` で `system` にしか `*` が付かない | `rbenv local 3.2.3` を **プロジェクトフォルダの中で** 実行できていない。`pwd` で場所を確認してから再実行 |
| `403 Permission denied`(push時) | remoteが本家(`cotes2020/chirpy-starter`等)を指したまま。`git remote set-url origin 自分のリポジトリURL` で向け直す |
| ページが真っ白/ユーザー名とリポジトリ名だけ表示される | Settings → Pages の Source が「Deploy from a branch」のまま。「GitHub Actions」に変更する |
| pushで `refusing to allow a Personal Access Token to create or update workflow` | トークンに `workflow` スコープが無い。トークン設定でチェックを追加し「Update token」 |
| push後も同じ認証エラーが続く | 古い認証情報がMacのキーチェーンに残っている。`security delete-internet-password -s github.com` で削除してから再push |
| `git push` で `[rejected] (fetch first)` | GitHub側リポジトリ作成時に「Add a README」をチェックし、ローカルの履歴と食い違っている。作成時にチェックを外すのが一番簡単。すでに発生した場合は `git pull origin main --allow-unrelated-histories --no-rebase` → README競合を `git checkout --ours README.md` で解決 → commit → push |
| Actionsタブに候補(Configure)しか出ない | `.github/workflows/` がまだpushされていない。`git status` で未追跡になっていないか確認しpush |
| Web画面で編集したのに反映されない | ①コミット時に「Create a new branch」を選んでPRのままになっていないか確認 ②ActionsがビルドしたコミットIDが最新のものと一致しているか確認 ③CDNキャッシュの可能性があるのでシークレットウィンドウで数分後に再確認 |
| 記事が公開されない・一覧に出ない | `date:` が未来日付になっている可能性(未来投稿は非公開がデフォルト)。今すぐ公開したいなら現在時刻以前にするか `_config.yml` に `future: true` を追加 |
| 複数端末で作業していて push が衝突する | 作業開始前に `git pull origin main` を徹底していないのが原因。習慣化することで回避できる |

---

## 次のステップ案

- 独自ドメインを設定する(`_config.yml`で設定可能)
- プロフィールページやカテゴリページを追加する
- 研究業績やCVページを別途作る
- 「日記っぽさ」が気になる場合は `no_date: true` の活用や、Collections機能・Just the Docs等のドキュメント系テーマへの切り替えを検討
- Ruby環境の構築に手間を感じる場合、Go製の「Hugo」やNode.js製の「Eleventy」など、より簡素なセットアップの静的サイトジェネレータへの乗り換えも選択肢

気になるところがあれば、その部分だけ深掘りして手順を詳しくします。
