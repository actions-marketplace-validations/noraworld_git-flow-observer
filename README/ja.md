# git-flow observer

| [English](/README/en.md) | **日本語** |
| ------------------------ | --------- |

git-flow observer は、プロジェクトが git-flow として正しく運用されているかどうかをチェックする GitHub Actions 製の CI ツールです。



## 仕組み
ヘッドブランチ (多くの場合、`develop` ブランチ) がベースブランチ (多くの場合、`main` ブランチ) と比べて fast-forwarded かどうかをチェックします。ヘッドブランチが fast-forwarded だった場合は CI が通りますが、そうでなければ失敗します。

いくつかのケースを紹介します。

* 以下のような PR をマージしたあと、ベースブランチ (`main` ブランチ) に追加されたコミットをヘッドブランチ (`develop` ブランチ) にマージバックせずに新しい PR を作成した場合に失敗します
    * ベースブランチ (`main` ブランチ) にのみ存在するコミットがマージコミットのみの状態で、ヘッドブランチ (`develop` ブランチ) にマージする PR
    * 追加コミットのあるリリースブランチをベースブランチ (`main` ブランチ) にマージする PR
    * ホットフィックスブランチをベースブランチ (`main` ブランチ) にマージする PR
    * cherry-pick されたコミットがある PR
* ベースブランチ (`main` ブランチ) からヘッドブランチ (`develop` ブランチ) にマージする PR を作成した場合はスキップされます (マージバック PR) [^1]

[^1]: ワークフロー YAML 内で `if: github.head_ref != 'main' || github.base_ref != 'develop'` を設定していた場合のみ

失敗した場合はプロジェクトが git-flow に準拠していないことに気づけます。



## インストール方法
インストールは非常に簡単です。以下の YAML コードを `.github/workflows/git-flow-observer.yml` というディレクトリ構造とファイル名でリポジトリに追加するだけです。

```yaml
# .github/workflows/git-flow-observer.yml

name: "git-flow observer"

on: pull_request

jobs:
  git-flow-observer:
    if: github.head_ref != 'main' || github.base_ref != 'develop'
    runs-on: ubuntu-latest
    steps:
      - name: "Observe"
        uses: noraworld/git-flow-observer@v0.1.0
        with:
          head: "develop"
          base: "main"
```

必要に応じて上記 YAML コードの一部を次のように置き換えます。

| キー | 説明 | 必須かどうか | サンプル値 |
| --- | --- | :------: | --- |
| `jobs.git-flow-observer.if` | ジョブをスキップするときの条件を指定します | False | `github.head_ref != 'main' \|\| github.base_ref != 'develop'` |
| `jobs.git-flow-observer.steps[*].with.head` | fast-forwarded かどうかをチェックするブランチを指定します | True | `"develop"` |
| `jobs.git-flow-observer.steps[*].with.base` | 比較の対象となるブランチを指定します | True | `"main"` |

HINT: `"main"` の代わりに `"master"` を指定する必要があるかもしれません。

HINT: `if: github.head_ref != 'main' || github.base_ref != 'develop'` を設定すると、ベースブランチからヘッドブランチへの PR のときはスキップされます。これにより CI が失敗した場合の修復作業が楽になることがあります。

以上でインストール作業は完了です。



## よくある質問
### 失敗した場合はどうしたら良いですか?

1. ヘッドブランチを fast-forwarded にします

まず、ヘッドブランチを fast-forwarded にします。ほとんどの場合、ベースブランチからヘッドブランチにマージする PR を作りマージすることで解決できます。

![Merge from main into develop](/screenshots/merge_from_main_into_develop_pr.png)

2. ジョブを再実行する

その後、以下のスクリーンショットに従いジョブを再実行します。

![Failed CI details](/screenshots/failed_ci_details.png)

![Re-run jobs button](/screenshots/rerun_jobs_button.png)

![Re-run jobs dialog](/screenshots/rerun_jobs_dialog.png)

### PR をマージする前に CI が通っていることを必須にしたい場合はどうしたら良いですか?

リポジトリの設定ページから設定できます。

1. (リポジトリ内の) `Settings` > `Branches` にアクセスし、`Add rule` ボタンをクリックします

HINT: `Settings` はアカウントの設定ではなくリポジトリの設定です。

2. 設定したいブランチのブランチ名パターンを入力し、`Require status checks to pass before merging` を有効し、git-flow observer をステータスチェックリストに追加します

![Branch protection rule](/screenshots/branch_protection_rule.png)

3. ページ下部の `Create` ボタンをクリックします

4. 1 〜 3 の手順をすべてのブランチに対して繰り返します

HINT: ヘッドブランチとベースブランチの両方に設定することをおすすめします。



## ライセンス
本プロジェクト内のすべてのコードは MIT ライセンスに基づき使用することができます。詳細は [LICENSE](LICENSE) をご覧ください。
