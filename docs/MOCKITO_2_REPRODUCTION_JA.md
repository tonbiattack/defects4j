# Defects4JでMockito-2の実在バグを再現する手順

このドキュメントは、Defects4J 3.x を使って **Mockito-2 の buggy version（`2b`）をチェックアウトし、トリガーテスト3件の失敗を再現するまで**の手順と、実際に遭遇したエラー・対処内容を日本語でまとめたものです。

## 目的

Defects4J に収録されている Mockito-2 のバグを、以下の流れで再現します。

1. Defects4J の依存関係を整える
2. Defects4J が管理する対象OSSリポジトリを取得する
3. Mockito-2 の buggy version をチェックアウトする
4. トリガーテストを確認する
5. テストを実行し、期待どおり3件失敗することを確認する

今回確認したバグは、Mockito の待機検証で使用される負の duration に関するものです。

---

## 環境

今回の実行環境は以下です。

- Ubuntu 24.04系
- zsh
- Defects4J 3.x
- 作業ディレクトリ: `~/defects4j`

Defects4J は Perl 製のCLIを含むため、Javaだけでなく Perl モジュールも必要です。

---

## 1. Defects4J の配置

Defects4J のリポジトリを `~/defects4j` に置いている前提です。

```bash
cd ~/defects4j
```

Defects4J CLI は次の場所にあります。

```text
~/defects4j/framework/bin/defects4j
```

PATH が通っていない場合は、次のように直接実行できます。

```bash
./framework/bin/defects4j
```

毎回フルパスを書くのが面倒な場合は、zsh の PATH に追加します。

```bash
echo 'export PATH="$PATH:$HOME/defects4j/framework/bin"' >> ~/.zshrc
source ~/.zshrc
```

確認します。

```bash
which defects4j
```

---

## 2. Perl依存関係の不足

### String::Interpolate がない

最初に checkout を実行したところ、以下のエラーが発生しました。

```text
Can't locate String/Interpolate.pm in @INC
(you may need to install the String::Interpolate module)
```

これは Mockito の依存ではなく、**Defects4J 本体の Perl スクリプトが必要としているモジュール**です。

Defects4J の Perl 依存関係はルートの `cpanfile` に定義されています。

個別に入れる場合の例:

```bash
sudo cpan String::Interpolate
```

または `cpanm` を使います。

```bash
sudo apt install cpanminus
sudo cpanm String::Interpolate
```

### DBI がない

次に以下のエラーが発生しました。

```text
Can't locate DBI.pm in @INC
(you may need to install the DBI module)
```

Ubuntu では次でインストールできます。

```bash
sudo apt install libdbi-perl
```

`DBI` は Perl からデータベースへアクセスするための共通インターフェースで、Defects4J の `framework/core/DB.pm` などから利用されています。

### 依存をまとめて確認する場合

不足モジュールを1つずつ追加するより、Defects4J ルートにある `cpanfile` を基準に依存を入れる方が確実です。

```bash
cd ~/defects4j
cpanm --installdeps .
```

権限や環境によってはシステムPerlではなく local::lib などを利用してください。

---

## 3. apt update 時の GitHub CLI GPG 警告

`sudo apt update` 実行時、今回の環境では次の警告も発生しました。

```text
NO_PUBKEY 5612B36462313325
```

これは **Defects4J のエラーではなく、GitHub CLI のAPTリポジトリ署名鍵に関する問題**です。

そのため、Mockito-2 のバグそのものとは切り分けて考えます。

Defects4J のトラブルシュート時は、表示されたエラーが

- Defects4J本体
- Perl依存
- Java
- Git
- OSのAPT設定
- シェル設定

のどこに属するかを分けて見ると調査しやすくなります。

---

## 4. project_repos の取得

Defects4J は対象OSSの元リポジトリを `project_repos` 配下で利用します。

```bash
cd ~/defects4j/project_repos
./get_repos.sh
```

今回、約602MBのアーカイブが取得されました。

```text
100  602M  100  602M
```

途中で以下のような curl 警告が表示されました。

```text
Warning: Failed to get filetime
Warning: Illegal date format for -z, --time-cond
```

ただし、ダウンロード自体は100%完了しました。

取得後、アーカイブを展開します。

```bash
unzip -q -u defects4j-repos-v3.zip
```

---

## 5. Mockito-2 buggy version を checkout

Defects4J ルートへ移動します。

```bash
cd ~/defects4j
```

Mockito の Bug ID 2 の **buggy version** を checkout します。

```bash
defects4j checkout -p Mockito -v 2b -w work/Mockito-2b
```

オプションの意味は以下です。

| オプション | 意味 |
|---|---|
| `-p Mockito` | 対象プロジェクト |
| `-v 2b` | Bug ID 2 の buggy version |
| `-w work/Mockito-2b` | 作業ディレクトリ |

Defects4J では、

- `2b`: buggy version
- `2f`: fixed version

という命名になっています。

### 実際の checkout 結果

```text
Checking out 80452c7a to /home/teni2/defects4j/work/Mockito-2b............. OK
Init local repository...................................................... OK
Tag post-fix revision...................................................... OK
sed: -e expression #1, char 84: `s' コマンドが終了していません
Run post-checkout hook..................................................... OK
Excluding broken/flaky tests............................................... OK
Excluding broken/flaky tests............................................... OK
Excluding broken/flaky tests............................................... OK
Initialize fixed program version........................................... OK
Apply patch................................................................ OK
Initialize buggy program version........................................... OK
Diff 80452c7a:d30450fa..................................................... OK
Apply patch................................................................ OK
Tag pre-fix revision....................................................... OK
Check out program version: Mockito-2b...................................... OK
```

途中で `sed` の警告が出ていますが、最終的に

```text
Check out program version: Mockito-2b...................................... OK
```

まで完了しました。

この後の `export` と `test` も成功したため、今回の再現作業ではこの警告は致命的ではありませんでした。

ただし別のバグを扱う場合や checkout 後の処理に異常がある場合は、無視せず原因を確認した方が安全です。

---

## 6. checkout 先へ移動

作成された作業ディレクトリへ移動します。

```bash
cd ~/defects4j/work/Mockito-2b
```

この時点で Git のブランチ表示は次のようになりました。

```text
D4J_Mockito_2_BUGGY_VERSION
```

これにより、現在 buggy version を見ていることも確認できます。

---

## 7. トリガーテストの確認

Defects4J の **トリガーテスト（triggering test）** は、対象のバグを含む版では失敗し、修正版では成功する、その不具合を直接再現するテストです。

checkout した作業ディレクトリ内で次を実行します。

```bash
defects4j export -p tests.trigger
```

今回の結果:

```text
Running ant (export.tests.trigger)......................................... OK
org.mockito.internal.util.TimerTest::should_throw_friendly_reminder_exception_when_duration_is_negative
org.mockito.verification.NegativeDurationTest::should_throw_exception_when_duration_is_negative_for_timeout_method
org.mockito.verification.NegativeDurationTest::should_throw_exception_when_duration_is_negative_for_after_method
```

Mockito-2 では3件のトリガーテストが登録されています。

1. `Timer(-1)` が負の duration を拒否するか
2. `Mockito.timeout(-1)` が負の duration を拒否するか
3. `Mockito.after(-1)` が負の duration を拒否するか

---

## 8. export は Defects4J ルートでは実行しない

最初、次の場所で `export` を実行して失敗しました。

```text
~/defects4j
```

実行:

```bash
defects4j export -p tests.trigger
```

結果:

```text
Cannot open config file (/home/teni2/defects4j/.defects4j.config): No such file or directory
/home/teni2/defects4j is not a valid working directory!
```

原因は、`export` が **checkout 済みの Defects4J working directory を対象にするコマンド**だからです。

正しくは以下です。

```bash
cd ~/defects4j/work/Mockito-2b
defects4j export -p tests.trigger
```

checkout 済みディレクトリには Defects4J が管理する設定が生成されており、そのディレクトリを基準に `export` や `test` が動作します。

---

## 9. テスト実行

Mockito-2b のディレクトリ内で実行します。

```bash
defects4j test
```

実際の結果:

```text
Running ant (compile.tests)................................................ OK
Running ant (run.dev.tests)................................................ OK
Failing tests: 3
  - org.mockito.internal.util.TimerTest::should_throw_friendly_reminder_exception_when_duration_is_negative
  - org.mockito.verification.NegativeDurationTest::should_throw_exception_when_duration_is_negative_for_timeout_method
  - org.mockito.verification.NegativeDurationTest::should_throw_exception_when_duration_is_negative_for_after_method
```

`tests.trigger` で確認した3件と、実際に失敗した3件が一致しています。

したがって **Mockito-2 のバグ再現に成功**しています。

---

## 10. 今回の最終状態

```text
Mockito-2b
  ↓
D4J_Mockito_2_BUGGY_VERSION
  ↓
tests.trigger = 3件
  ↓
defects4j test
  ↓
Failing tests: 3
```

期待どおり buggy version だけでトリガーテストが失敗しているため、ここから実装を追って原因を調査できます。

---

## 11. 次に調査するポイント

Mockito-2 の場合、まず次を確認します。

```text
org.mockito.internal.util.Timer
```

特に確認するポイント:

- `durationMillis` がどこで保持されるか
- コンストラクタで負数チェックをしているか
- `isCounting()` などの時間判定がどうなっているか
- `Mockito.timeout()` と `Mockito.after()` から Timer までどう到達するか
- 3件のトリガーテストが、どの契約を期待しているか

バグ調査では、最初から fixed version を見るのではなく、

1. トリガーテストを読む
2. 失敗条件を理解する
3. 関係するコードを追う
4. 仮説を立てる
5. 最小修正を試す
6. トリガーテストを再実行する
7. 全テストを実行する
8. 最後に fixed version と比較する

という順序にすると、デバッグ練習として使いやすくなります。

---

## 12. よく使うコマンドまとめ

### buggy version の checkout

```bash
cd ~/defects4j
defects4j checkout -p Mockito -v 2b -w work/Mockito-2b
```

### 作業ディレクトリへ移動

```bash
cd ~/defects4j/work/Mockito-2b
```

### トリガーテスト確認

```bash
defects4j export -p tests.trigger
```

### テスト実行

```bash
defects4j test
```

### fixed version を別ディレクトリに checkout する場合

buggy version の調査が終わってから答え合わせに使います。

```bash
cd ~/defects4j
defects4j checkout -p Mockito -v 2f -w work/Mockito-2f
```

---

## 13. 今回遭遇した問題の一覧

| 症状 | 原因 | 対処 |
|---|---|---|
| `zsh: command not found: defects4j` | PATH 未設定 | `framework/bin` を PATH に追加、または直接実行 |
| `Can't locate String/Interpolate.pm` | Perl依存不足 | `String::Interpolate` をインストール |
| `Can't locate DBI.pm` | Perl依存不足 | `sudo apt install libdbi-perl` |
| `NO_PUBKEY 5612B36462313325` | GitHub CLI のAPT署名鍵 | Defects4Jとは切り分けてGitHub CLIの鍵を更新 |
| `defects4j-repos-v3.zip` がない | project_repos 未取得 | `project_repos/get_repos.sh` を実行 |
| `not a valid working directory` | Defects4Jルートで `export` を実行 | checkout 先へ移動して実行 |
| `sed ... s コマンドが終了していません` | checkout 中の hook 等で発生 | 今回は後続処理が正常だったため継続して確認 |
| `compinit ... _docker` | zsh の Docker completion 設定 | Defects4Jとは別問題として扱う |

---

## 14. zsh の `_docker` 警告について

シェル起動時に以下も表示されました。

```text
compinit:527: そのようなファイルやディレクトリはありません: /usr/share/zsh/vendor-completions/_docker
```

これは zsh の補完設定が Docker 用補完ファイルを参照している一方、対象ファイルが存在しない状態です。

**Mockito-2 の checkout/test には直接関係ありません。**

Defects4J の問題と混ぜず、必要であれば zsh/Docker 補完設定として別途修正します。

---

## まとめ

今回の実行で、Defects4J 上の Mockito-2 buggy version を正常に再現できました。

確認できたポイントは以下です。

- Mockito-2b の checkout に成功
- `D4J_Mockito_2_BUGGY_VERSION` を取得
- トリガーテスト3件を取得
- `defects4j test` で同じ3件が失敗
- Defects4J本体と、OS・Perl・zsh由来のエラーを切り分けて対処

これで、実在OSSバグを「再現 → 原因調査 → 最小修正 → 回帰確認 → fixed versionとの比較」という流れでデバッグするための土台ができました。
