# Defects4J リポジトリ概要

## 1. このリポジトリは何か

**Defects4J** は、実際のオープンソース Java プロジェクトから収集した不具合を、同じ条件で再現・検証できる形に整備したデータセットと、その実行・分析を支えるフレームワークです。主な目的は、ソフトウェア工学研究におけるデバッグ、プログラム修正、テスト生成、コードカバレッジ、ミューテーション解析などの実験を再現可能にすることです。[1]

このリポジトリには、単なるバグの一覧だけでなく、対象プロジェクトの取得、特定の不具合を含むバージョンへのチェックアウト、コンパイル、テスト実行、メタデータ照会、カバレッジ・ミューテーション解析などを行うコマンドライン基盤が含まれています。[1] [2]

> **要点:** Defects4J は「実在する Java 不具合のベンチマーク」と「その不具合を一定の手順で扱うための実験基盤」を一体で提供するリポジトリです。

## 2. 収録されている不具合

README の現行記載では、18 個のオープンソースプロジェクトから **有効な不具合 854 件** と **非推奨の不具合 10 件** を収録しています。[1]

| プロジェクト ID | 元プロジェクト | 有効な不具合数 |
|---|---|---:|
| Chart | jfreechart | 26 |
| Cli | commons-cli | 39 |
| Closure | closure-compiler | 174 |
| Codec | commons-codec | 18 |
| Collections | commons-collections | 28 |
| Compress | commons-compress | 47 |
| Csv | commons-csv | 16 |
| Gson | gson | 18 |
| JacksonCore | jackson-core | 26 |
| JacksonDatabind | jackson-databind | 110 |
| JacksonXml | jackson-dataformat-xml | 6 |
| Jsoup | jsoup | 93 |
| JxPath | commons-jxpath | 22 |
| Lang | commons-lang | 61 |
| Math | commons-math | 106 |
| Mockito | mockito | 38 |
| Time | joda-time | 26 |

不具合には、対応する課題管理システム上の報告、修正コミット、修正前後のリビジョン、修正を発生させるトリガーテストなどのメタデータがあります。修正は原則として単一コミットで行われ、リファクタリングや機能追加などの無関係な変更を取り除いて最小化されています。また、設定ファイル・ドキュメント・テストだけを変更するものではなく、ソースコードの変更によって修正された不具合が対象です。[1]

不具合を含む版と修正版は、それぞれ `<id>b` と `<id>f` で表されます。たとえば、`1b` は不具合を含む版、`1f` は修正版です。Java のバージョン変更によって再現できなくなった不具合は `deprecated-bugs.csv` に記録され、通常の対象からは除外されます。[1]

## 3. リポジトリの主要構成

| パス | 役割 |
|---|---|
| `framework/bin/` | `defects4j` CLI と、テスト・カバレッジ・ミューテーション解析などの実行スクリプト |
| `framework/core/` | プロジェクト、VCS、テスト、カバレッジ、クエリなどを扱う Perl モジュール |
| `framework/bug-mining/` | 不具合の収集・分析・パッチ最小化を行うためのスクリプト |
| `framework/projects/` | プロジェクト共通・個別のビルドやエクスポート設定 |
| `framework/test/` | フレームワークの動作確認用テストと利用例 |
| `framework/util/` | 不具合表、テスト表、依存関係、関連テストなどを生成・整備するユーティリティ |
| `project_repos/` | `init.sh` 実行時に取得される対象プロジェクトのリポジトリ。サイズと重複を避けるため、Git 管理対象には含まれない |
| `developer/` | Defects4J のコントリビューター向け資料 |
| `.github/workflows/` | CI 設定 |
| `cpanfile` | Perl 依存モジュールの定義 |
| `Dockerfile` / `docker-compose.yml` | コンテナ環境での利用を支援する設定 |

## 4. セットアップの流れ

公式 README に示されている基本的なセットアップは次のとおりです。[1]

```bash
git clone https://github.com/rjust/defects4j
cd defects4j
cpanm --installdeps .
./init.sh
export PATH="$PATH:/path/to/defects4j/framework/bin"
defects4j info -p Lang
```

`./init.sh` は、Git リポジトリに含まれていない対象プロジェクトや外部ライブラリを取得します。そのため、初期化にはネットワークアクセス、十分なディスク容量、各種ビルドツールが必要です。README が示す主な前提条件は **Java 11、Git 1.9 以上、Subversion 1.8 以上、Perl 5.0.12 以上、`cpanm`** です。[1]

## 5. 基本的な利用例

### プロジェクトと不具合の情報を確認する

```bash
defects4j info -p Lang
defects4j info -p Lang -b 1
```

### 不具合を含む版をチェックアウトする

```bash
defects4j checkout -p Lang -v 1b -w /tmp/lang_1_buggy
cd /tmp/lang_1_buggy
```

### コンパイルとテストを実行する

```bash
defects4j compile
defects4j test
```

### バージョン固有のメタデータを取得する

```bash
defects4j export -p tests.trigger
defects4j export -p classes.modified
defects4j export -p cp.test
```

`export` では、変更されたクラス、関連テスト、トリガーテスト、ソース・テストディレクトリ、コンパイル用・テスト用クラスパスなどを取得できます。`query` を利用すると、バグ ID、修正前後のコミット、課題 URL、変更クラスなど、プロジェクト単位のメタデータを CSV として出力できます。[1]

## 6. 提供される主な CLI コマンド

| コマンド | 用途 |
|---|---|
| `info` | プロジェクトまたは不具合の概要を表示 |
| `env` | Defects4J 実行環境を表示 |
| `checkout` | 不具合版または修正版を作業ディレクトリへ展開 |
| `compile` | ソースと開発者テストをコンパイル |
| `test` | テストメソッドまたはテストスイートを実行 |
| `mutation` | ミューテーション解析を実行 |
| `coverage` | コードカバレッジ解析を実行 |
| `monitor.test` | テスト実行時のクラスローダーを監視 |
| `bids` / `pids` | 不具合 ID または利用可能なプロジェクト ID を表示 |
| `export` | バージョン固有のプロパティを出力 |
| `query` | 不具合メタデータを問い合わせ、CSV に出力 |

## 7. 再現性に関する注意点

Defects4J の結果を再現する際は、環境差分を軽視できません。公式 README では、すべての不具合とトリガーテストを **Java 11** で検証しているため、異なる Java バージョンではテスト失敗や再現不能が発生する可能性があると説明されています。[1]

また、テストのタイムゾーンは `America/Los_Angeles` に固定されています。フレームワーク外で対象コードを扱う場合は、次のように `TZ` を設定することが推奨されています。

```bash
export TZ=America/Los_Angeles
```

壊れたテストや不安定なテストは Defects4J から除外されていますが、利用者の環境では依存ライブラリ、Java、OS、タイムゾーンなどの違いによって挙動が変わる可能性があります。研究結果を比較する場合は、実行環境、Defects4J のバージョン、対象プロジェクト、バグ ID、実行コマンドを記録しておくことが重要です。[1]

## 8. 想定される利用目的

このリポジトリは、次のような研究・学習用途に適しています。

| 利用目的 | Defects4J の使い方 |
|---|---|
| 自動プログラム修正 | 不具合版、トリガーテスト、修正版を使って修正候補を評価する |
| テスト生成 | 既存テスト、トリガーテスト、関連テストを基準に新しいテストの有効性を調べる |
| デバッグ支援 | 再現可能な失敗を対象に、原因箇所の特定手法を比較する |
| カバレッジ分析 | `coverage` や `export` で実行範囲と関連クラスを調べる |
| ミューテーション解析 | `mutation` でテストスイートの検出能力を評価する |
| ソフトウェア工学教育 | 実在プロジェクトの不具合を、失敗テストから修正まで追跡する |

## 9. このリポジトリを読むときのポイント

最初に `README.md` で対象プロジェクト、不具合 ID、セットアップ方法、CLI の使い方を確認し、次に `framework/bin/defects4j` と `framework/core/` でコマンドの実装を追うと、全体像を把握しやすくなります。個々のプロジェクトに関するビルド・テスト差分を確認したい場合は `framework/projects/` と `projects/` を参照し、フレームワーク自体の挙動を検証したい場合は `framework/test/` を利用します。

なお、対象プロジェクトのソースコードは clone 直後からすべて揃っているわけではありません。`project_repos/` は初期化処理で取得されるため、データセットを実際に checkout・compile・test する前に `cpanm --installdeps .` と `./init.sh` を実行する必要があります。[1]

## 参考資料

[1]: https://github.com/tonbiattack/defects4j/blob/master/README.md "Defects4J README"
[2]: https://github.com/tonbiattack/defects4j/tree/master/framework "Defects4J framework directory"
[3]: https://github.com/tonbiattack/defects4j/blob/master/framework/bin/defects4j "Defects4J command-line interface"
[4]: https://github.com/tonbiattack/defects4j/tree/master/framework/test "Defects4J framework tests"
