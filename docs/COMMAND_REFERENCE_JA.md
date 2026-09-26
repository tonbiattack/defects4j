# Defects4Jで実行したコマンドの意味

このドキュメントでは、Mockito-2 のバグ再現で実際に使用したコマンドについて、**何をしているのか、各オプションが何を意味するのか、どのディレクトリで実行するのか**を詳しく説明します。

手順そのものは [MOCKITO_2_REPRODUCTION_JA.md](./MOCKITO_2_REPRODUCTION_JA.md) を参照してください。

---

## 1. `cd ~/defects4j`

```bash
cd ~/defects4j
```

### 意味

`cd` は **change directory** の略で、現在の作業ディレクトリを変更します。

`~` は現在ログインしているユーザーのホームディレクトリを表します。

今回の環境では、

```text
~
↓
/home/teni2
```

なので、

```text
~/defects4j
↓
/home/teni2/defects4j
```

という意味になります。

### なぜ実行するのか

Defects4J 本体のルートディレクトリへ移動するためです。

Mockito の checkout 先を

```text
work/Mockito-2b
```

のような相対パスで指定する場合、このディレクトリを基準に作られます。

---

## 2. `./framework/bin/defects4j`

```bash
./framework/bin/defects4j
```

### 意味

`./` は **現在のディレクトリを起点に実行する**ことを意味します。

つまり、

```text
./framework/bin/defects4j
```

は、

```text
現在のディレクトリ
  └─ framework
      └─ bin
          └─ defects4j
```

という実行ファイルを直接起動しています。

### なぜ `defects4j` だけでは動かなかったのか

シェルはコマンドを入力すると、`PATH` 環境変数に登録されているディレクトリから実行ファイルを探します。

PATH に

```text
~/defects4j/framework/bin
```

が登録されていなかったため、

```bash
defects4j
```

だけでは、

```text
zsh: command not found: defects4j
```

となりました。

---

## 3. `echo 'export PATH=...' >> ~/.zshrc`

```bash
echo 'export PATH="$PATH:$HOME/defects4j/framework/bin"' >> ~/.zshrc
```

### コマンド全体の目的

Defects4J の CLI ディレクトリを、zsh の PATH に恒久的に追加します。

### `echo`

`echo` は文字列を標準出力へ出力するコマンドです。

例えば、

```bash
echo hello
```

なら、

```text
hello
```

と表示されます。

### `export PATH=...`

`PATH` は、シェルが実行コマンドを探すディレクトリ一覧です。

```bash
export PATH="$PATH:$HOME/defects4j/framework/bin"
```

では、

- 既存の PATH: `$PATH`
- Defects4J の CLI: `$HOME/defects4j/framework/bin`

を連結しています。

### `$HOME`

ログインユーザーのホームディレクトリです。

今回なら、

```text
$HOME
↓
/home/teni2
```

です。

### `>> ~/.zshrc`

`>>` は **ファイル末尾へ追記するリダイレクト**です。

したがって、

```bash
echo '...' >> ~/.zshrc
```

は、文字列を画面へ表示する代わりに `.zshrc` の末尾へ書き込みます。

`.zshrc` は zsh 起動時に読み込まれる設定ファイルなので、次回以降も PATH 設定が有効になります。

---

## 4. `source ~/.zshrc`

```bash
source ~/.zshrc
```

### 意味

現在起動中の zsh に `.zshrc` の内容を再読み込みさせます。

通常、`.zshrc` を編集しても、現在のシェルには自動反映されません。

`source` を使うことで、ターミナルを開き直さずに新しい設定を適用できます。

### 今回の用途

先ほど追加した、

```bash
export PATH="$PATH:$HOME/defects4j/framework/bin"
```

を現在のシェルにも反映します。

---

## 5. `which defects4j`

```bash
which defects4j
```

### 意味

シェルが `defects4j` と入力されたときに、どの実行ファイルを使うか確認します。

正常なら例えば、

```text
/home/teni2/defects4j/framework/bin/defects4j
```

と表示されます。

### 何を確認しているのか

PATH 設定が正しく反映されたかを確認しています。

---

## 6. `sudo apt update`

```bash
sudo apt update
```

### `sudo`

管理者権限でコマンドを実行します。

### `apt`

Ubuntu / Debian 系でパッケージを管理するコマンドです。

### `update`

インストール済みパッケージそのものを更新するコマンドではありません。

APT が参照する各リポジトリから、**最新のパッケージ一覧を取得する処理**です。

つまり、

```text
sudo apt update
```

は、

```text
インターネット上のAPTリポジトリ
        ↓
利用可能なパッケージ一覧を更新
        ↓
ローカルのAPTキャッシュ
```

という処理です。

### `upgrade` との違い

```bash
sudo apt update
```

は一覧更新です。

一方、

```bash
sudo apt upgrade
```

は、一覧をもとに実際のパッケージを更新します。

---

## 7. `sudo apt install cpanminus`

```bash
sudo apt install cpanminus
```

### 意味

Ubuntu のAPTから `cpanminus` をインストールします。

`cpanminus` は Perl モジュールをインストールするためのツールで、実行コマンド名は `cpanm` です。

### 関係

```text
APT
 ↓
cpanminus パッケージをインストール
 ↓
cpanm コマンドが使える
 ↓
Perlモジュールをインストールできる
```

---

## 8. `sudo cpan String::Interpolate`

```bash
sudo cpan String::Interpolate
```

### 意味

CPAN から Perl モジュール `String::Interpolate` をインストールします。

### CPANとは

**Comprehensive Perl Archive Network** の略です。

Java の Maven Central や JavaScript の npm registry に近い位置づけで、Perl ライブラリを配布する仕組みです。

### 今回なぜ必要だったのか

Defects4J の Perl コードが、

```perl
use String::Interpolate;
```

相当の依存を持っているためです。

モジュールがない状態では Perl が、

```text
Can't locate String/Interpolate.pm in @INC
```

とエラーにします。

---

## 9. `sudo cpanm String::Interpolate`

```bash
sudo cpanm String::Interpolate
```

### 意味

`cpan` の代わりに `cpanm` を使って、同じ Perl モジュールをインストールします。

### cpan と cpanm の違い

どちらも CPAN のモジュールをインストールできます。

`cpanm` は一般に、

- 出力が簡潔
- 操作がシンプル
- 自動化しやすい

という特徴があります。

Defects4J の README でも `cpanm` が前提ツールとして扱われています。

---

## 10. `sudo apt install libdbi-perl`

```bash
sudo apt install libdbi-perl
```

### 意味

Ubuntu のパッケージとして Perl の DBI モジュールをインストールします。

### DBIとは

**Database Interface** の略です。

Perl からデータベースを扱うための共通インターフェースです。

Java でいう JDBC に少し近い役割です。

### 今回必要だった理由

Defects4J 内部の、

```text
framework/core/DB.pm
```

などが DBI を利用しているためです。

---

## 11. `cpanm --installdeps .`

```bash
cpanm --installdeps .
```

### 意味

現在のディレクトリにある Perl プロジェクトの依存関係をまとめてインストールします。

### `--installdeps`

プロジェクト本体をインストールするのではなく、**依存モジュールだけをインストールする**指定です。

### `.`

現在のディレクトリを意味します。

Defects4J ルートで実行すると、

```text
~/defects4j/cpanfile
```

をもとに必要な Perl モジュールを解決します。

---

## 12. `cd ~/defects4j/project_repos`

```bash
cd ~/defects4j/project_repos
```

### 意味

Defects4J が各OSSの元リポジトリを管理するディレクトリへ移動します。

Defects4J 本体と、バグ対象となる各OSSの Git データは役割が異なります。

概念的には、

```text
defects4j/
├─ framework/       Defects4J本体
├─ project_repos/   対象OSSの元リポジトリ群
└─ work/            実際に調査するcheckout先
```

という構成です。

---

## 13. `./get_repos.sh`

```bash
./get_repos.sh
```

### 意味

`project_repos` 用に用意されたシェルスクリプトを実行します。

### `./`

現在のディレクトリにあるファイルを直接実行する指定です。

### 何をするスクリプトか

Defects4J で利用する対象OSSリポジトリ群を取得します。

今回の実行では、

```text
defects4j-repos-v3.zip
```

という約602MBのアーカイブがダウンロードされました。

このリポジトリデータを利用して、後の `defects4j checkout` が各バグ版を再構築します。

---

## 14. `unzip -q -u defects4j-repos-v3.zip`

```bash
unzip -q -u defects4j-repos-v3.zip
```

### `unzip`

ZIP アーカイブを展開するコマンドです。

### `-q`

**quiet** の略です。

通常の詳細な展開ログを抑制します。

### `-u`

**update** の意味です。

既存ファイルがある場合、アーカイブ側の方が新しいファイルを更新します。

### 対象ファイル

```text
defects4j-repos-v3.zip
```

を現在のディレクトリへ展開します。

---

## 15. `defects4j checkout -p Mockito -v 2b -w work/Mockito-2b`

```bash
defects4j checkout -p Mockito -v 2b -w work/Mockito-2b
```

Mockito-2 の再現で最も重要なコマンドです。

### `defects4j`

Defects4J の CLI 本体です。

### `checkout`

指定したプロジェクト・バグ番号・バージョンを、調査可能な作業ディレクトリへ展開します。

Git の `git checkout` に似た名前ですが、単純なブランチ切り替えではありません。

Defects4J が保持するメタデータを使って、

- 対象リポジトリの準備
- 固定版・バグ版の特定
- パッチ適用
- broken/flaky test の除外
- Defects4J 用設定生成

などを行い、再現可能な working directory を作ります。

### `-p Mockito`

`-p` は **project** の指定です。

```text
-p Mockito
```

によって、Defects4J が管理している Mockito プロジェクトを対象にします。

### `-v 2b`

`-v` は **version** を指定します。

```text
2
```

は Bug ID 2、

```text
b
```

は buggy version を表します。

したがって、

```text
2b
```

は、

> Mockito の Bug ID 2 の修正前バージョン

という意味です。

固定版なら、

```text
2f
```

です。

`f` は fixed version を表します。

### `-w work/Mockito-2b`

`-w` は **working directory** の指定です。

```text
work/Mockito-2b
```

へ対象コードを展開します。

Defects4J ルートで実行した場合、

```text
~/defects4j/work/Mockito-2b
```

になります。

### コマンド全体を日本語にすると

> Defects4J が管理している Mockito の Bug ID 2 の修正前版を、work/Mockito-2b に作業用として展開する。

という意味です。

---

## 16. `cd ~/defects4j/work/Mockito-2b`

```bash
cd ~/defects4j/work/Mockito-2b
```

### 意味

先ほど checkout した Mockito-2 の修正前版へ移動します。

### なぜ移動が必要なのか

`defects4j test` や `defects4j export` は、現在のディレクトリにある Defects4J 用設定を参照します。

そのため、

```text
~/defects4j
```

ではなく、

```text
~/defects4j/work/Mockito-2b
```

で実行する必要があります。

---

## 17. `defects4j export -p tests.trigger`

```bash
defects4j export -p tests.trigger
```

### `export`

現在 checkout しているバグについて、Defects4J が保持するメタデータを出力するサブコマンドです。

### `-p`

ここでの `-p` は checkout の `-p` と意味が異なります。

checkout では project の指定でしたが、export では **property** の指定です。

### `tests.trigger`

対象バグのトリガーテスト一覧を表すプロパティです。

トリガーテストとは、

> buggy version では失敗し、fixed version では成功する、そのバグを直接露呈させるテスト

です。

### 今回の出力

```text
org.mockito.internal.util.TimerTest::should_throw_friendly_reminder_exception_when_duration_is_negative

org.mockito.verification.NegativeDurationTest::should_throw_exception_when_duration_is_negative_for_timeout_method

org.mockito.verification.NegativeDurationTest::should_throw_exception_when_duration_is_negative_for_after_method
```

### コマンド全体を日本語にすると

> 現在 checkout しているバグを直接再現するテスト一覧を表示する。

という意味です。

---

## 18. `defects4j test`

```bash
defects4j test
```

### 意味

現在 checkout しているプロジェクトのテストを Defects4J のルールに従って実行します。

Mockito-2 では内部的に Ant が使用され、

```text
Running ant (compile.tests)
Running ant (run.dev.tests)
```

と表示されました。

### Defects4J経由で実行する意味

単に `ant test` や `mvn test` を実行するのではなく、Defects4J が管理する、

- 対象テスト
- classpath
- broken test の除外
- flaky test の除外
- プロジェクト固有設定

などを反映した状態でテストできます。

再現実験では、できるだけ `defects4j test` を使う方が確実です。

### 今回の結果

```text
Failing tests: 3
```

となり、`tests.trigger` と同じ3件が失敗しました。

これによって Mockito-2 のバグが正しく再現できたと判断できます。

---

## 19. `defects4j checkout -p Mockito -v 2f -w work/Mockito-2f`

```bash
defects4j checkout -p Mockito -v 2f -w work/Mockito-2f
```

### 意味

Mockito-2 の **fixed version** を別ディレクトリへ checkout します。

```text
2b = Bug ID 2 の buggy version
2f = Bug ID 2 の fixed version
```

です。

### なぜ別ディレクトリにするのか

修正前版を保持したまま、修正版と比較できるようにするためです。

例えば、

```text
work/
├─ Mockito-2b
└─ Mockito-2f
```

と置けば、

- buggy version の原因調査
- fixed version の答え合わせ
- diff の確認

を並行して行えます。

---

## 20. `git:(D4J_Mockito_2_BUGGY_VERSION)` の意味

zsh のプロンプトに、

```text
Mockito-2b git:(D4J_Mockito_2_BUGGY_VERSION)
```

と表示されました。

これはコマンドではありませんが、重要な情報です。

現在の Git リポジトリが、

```text
D4J_Mockito_2_BUGGY_VERSION
```

という Defects4J が用意した参照位置にいることを表します。

したがって、今見ているソースコードが修正前版であることを確認できます。

---

## 21. `.defects4j.config` の役割

Defects4J ルートで、

```bash
defects4j export -p tests.trigger
```

を実行すると、

```text
Cannot open config file (.../.defects4j.config)
... is not a valid working directory!
```

となりました。

### 理由

`checkout` で作られた working directory には、Defects4J が対象プロジェクトを認識するための設定があります。

概念的には、

```text
work/Mockito-2b/
├─ .defects4j.config
├─ src/...
└─ ...
```

という状態です。

`defects4j export` や `defects4j test` は、この設定を見て、

- どのプロジェクトか
- どのバグ番号か
- buggy/fixed のどちらか
- ビルド方法

などを判断します。

そのため、Defects4J 本体のルートで実行しても対象が特定できません。

---

## 22. `Failing tests: 3` は失敗ではなく「再現成功」

通常の開発では、

```text
Failing tests: 3
```

は悪い結果です。

しかし Defects4J の buggy version を調査している場合は意味が違います。

今回の目的は、

> 既知のバグが存在する状態を再現する

ことです。

したがって、

```text
tests.trigger
↓
3件
```

に対して、

```text
defects4j test
↓
同じ3件が失敗
```

となれば、意図どおりの状態です。

つまり、

```text
テスト失敗
≠ 環境構築失敗

既知のトリガーテストが期待どおり失敗
= バグ再現成功
```

です。

---

## 23. コマンドの実行場所まとめ

| コマンド | 実行場所 |
|---|---|
| `defects4j checkout ...` | `~/defects4j` |
| `./get_repos.sh` | `~/defects4j/project_repos` |
| `unzip ...` | `~/defects4j/project_repos` |
| `defects4j export ...` | `~/defects4j/work/Mockito-2b` |
| `defects4j test` | `~/defects4j/work/Mockito-2b` |
| `cpanm --installdeps .` | `~/defects4j` |

---

## 24. 全体の流れ

実際の処理関係を簡略化すると以下です。

```text
Defects4J本体
~/defects4j
      │
      ├─ project_repos/get_repos.sh
      │          │
      │          └─ 対象OSSの元リポジトリ群を取得
      │
      └─ defects4j checkout
                 │
                 └─ work/Mockito-2b を生成
                            │
                            ├─ defects4j export -p tests.trigger
                            │       └─ バグ再現テストを確認
                            │
                            └─ defects4j test
                                    └─ 実際にバグを再現
```

---

## 25. 最小の再現コマンド

環境構築済みであれば、Mockito-2 の再現に必要な中心コマンドは以下だけです。

```bash
cd ~/defects4j

defects4j checkout \
  -p Mockito \
  -v 2b \
  -w work/Mockito-2b

cd work/Mockito-2b

defects4j export -p tests.trigger

defects4j test
```

意味を一文ずつ書くと、

```text
cd ~/defects4j
→ Defects4J本体へ移動

defects4j checkout -p Mockito -v 2b -w work/Mockito-2b
→ Mockito Bug 2 の修正前版を作業ディレクトリへ展開

cd work/Mockito-2b
→ 調査対象へ移動

defects4j export -p tests.trigger
→ このバグを直接再現するテストを確認

defects4j test
→ テストを実行し、既知のバグが再現することを確認
```

この5段階を理解しておけば、Mockito 以外の Defects4J プロジェクトでも同じ考え方で調査できます。
