# ワークフローの YAML 構文について

ワークフローファイルでは YAML 構文を使用し、ファイル拡張子は `.yml` または `.yaml` である必要があります。YAML を初めて使う場合は、`Learn YAML in Y minutes` を参照してください。

ワークフローファイルは、リポジトリの `.github/workflows` ディレクトリに保存する必要があります。

## name

ワークフローの名前です。GitHub は、リポジトリの「Actions」タブでワークフローの名前を表示します。`name` を省略すると、GitHub はリポジトリのルートを基準にしたワークフローファイルのパスを表示します。

## run-name

ワークフローから生成されるワークフロー実行の名前です。GitHub は、リポジトリの「Actions」タブにあるワークフロー実行の一覧でワークフロー実行名を表示します。`run-name` が省略されている場合、または空白文字だけで構成されている場合、実行名にはそのワークフロー実行に対するイベント固有の情報が設定されます。たとえば、`push` または `pull_request` イベントによってトリガーされるワークフローでは、コミットメッセージまたは pull request のタイトルが設定されます。

この値には式を含めることができ、`github` コンテキストおよび `inputs` コンテキストを参照できます。

#### run-name の例

```yaml
run-name: Deploy to ${{ inputs.deploy_target }} by @${{ github.actor }}
```

## on

ワークフローを自動的にトリガーするには、`on` を使用して、どのイベントでワークフローを実行できるかを定義します。利用可能なイベントの一覧については、「ワークフローをトリガーするイベント」を参照してください。

ワークフローをトリガーできる単一または複数のイベントを定義したり、時間ベースのスケジュールを設定したりできます。また、特定のファイル、タグ、またはブランチの変更時にのみワークフローが実行されるように制限することもできます。これらのオプションについては、以降のセクションで説明します。

#### 単一のイベントを使う

たとえば、次の `on` 値を持つワークフローは、ワークフローのリポジトリ内の任意のブランチに対して `push` が行われたときに実行されます。

```yaml
on: push
```

#### 複数のイベントを使う

単一のイベントまたは複数のイベントを指定できます。たとえば、次の `on` 値を持つワークフローは、リポジトリ内の任意のブランチに対して `push` が行われたとき、または誰かがそのリポジトリを fork したときに実行されます。

```yaml
on: [push, fork]
```

複数のイベントを指定した場合、ワークフローをトリガーするにはそのうちの 1 つだけが発生すれば十分です。ワークフローに対する複数のトリガーイベントが同時に発生した場合は、複数のワークフロー実行がトリガーされます。

#### アクティビティタイプを使う

一部のイベントにはアクティビティタイプがあり、ワークフローをいつ実行するかをより細かく制御できます。ワークフロー実行をトリガーするイベントアクティビティの種類を定義するには、`on.<event_name>.types` を使用します。

たとえば、`issue_comment` イベントには `created`、`edited`、`deleted` のアクティビティタイプがあります。ワークフローが `label` イベントでトリガーされる場合、ラベルが作成、編集、削除されるたびに実行されます。`label` イベントに対して `created` アクティビティタイプを指定すると、ラベルが作成されたときにはワークフローが実行されますが、ラベルが編集または削除されたときには実行されません。

```yaml
on:
  label:
    types:
      - created
```

複数のアクティビティタイプを指定した場合、ワークフローをトリガーするにはそのうちの 1 つだけが発生すれば十分です。ワークフローに対する複数のトリガーアクティビティタイプが同時に発生した場合は、複数のワークフロー実行がトリガーされます。たとえば、次のワークフローは issue が open されたとき、または label が付けられたときにトリガーされます。2 つのラベルが付いた issue が open されると、3 つのワークフロー実行が開始されます。1 つは issue の opened イベントに対するもので、2 つは 2 件の issue labeled イベントに対するものです。

```yaml
on:
  issues:
    types:
      - opened
      - labeled
```

各イベントとそのアクティビティタイプの詳細については、「ワークフローをトリガーするイベント」を参照してください。

#### フィルターを使う

一部のイベントにはフィルターがあり、ワークフローをいつ実行するかをより細かく制御できます。

たとえば、`push` イベントには `branches` フィルターがあり、どの `push` でも実行するのではなく、`branches` フィルターに一致するブランチへの `push` が発生した場合にのみワークフローを実行します。

```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'
```

#### 複数イベントでアクティビティタイプとフィルターを使う

あるイベントに対してアクティビティタイプまたはフィルターを指定し、かつワークフローが複数のイベントでトリガーされる場合は、各イベントを個別に設定する必要があります。設定のないイベントを含め、すべてのイベントの末尾にコロン (`:`) を付ける必要があります。

たとえば、次の `on` 値を持つワークフローは、次の場合に実行されます。

- ラベルが作成されたとき
- リポジトリの `main` ブランチに `push` されたとき
- GitHub Pages が有効なブランチに `push` されたとき

```yaml
on:
  label:
    types:
      - created
  push:
    branches:
      - main
  page_build:
```

### on.<event_name>.types

ワークフロー実行をトリガーするアクティビティの種類を定義するには、`on.<event_name>.types` を使用します。ほとんどの GitHub イベントは、複数の種類のアクティビティによってトリガーされます。たとえば、`label` はラベルが作成、編集、削除されたときにトリガーされます。`types` キーワードを使うと、ワークフローを実行する原因となるアクティビティを絞り込めます。Webhook イベントをトリガーするアクティビティタイプが 1 つだけである場合、`types` キーワードは不要です。

イベントタイプの配列を使用できます。各イベントとそのアクティビティタイプの詳細については、「ワークフローをトリガーするイベント」を参照してください。

```yaml
on:
  label:
    types: [created, edited]
```

### on.<pull_request|pull_request_target>.<branches|branches-ignore>

`pull_request` イベントおよび `pull_request_target` イベントを使用する場合、特定のブランチを対象とする pull request に対してのみワークフローが実行されるよう設定できます。

ブランチ名パターンを含めたい場合、またはブランチ名パターンの包含と除外の両方を行いたい場合は `branches` フィルターを使用します。ブランチ名パターンを除外するだけの場合は `branches-ignore` フィルターを使用します。1 つのワークフロー内で同じイベントに対して `branches` と `branches-ignore` の両方を使用することはできません。

`branches`/`branches-ignore` と `paths`/`paths-ignore` の両方を定義した場合、ワークフローは両方のフィルター条件を満たしたときにのみ実行されます。

`branches` および `branches-ignore` キーワードは、`*`、`**`、`+`、`?`、`!` などの文字を使用した glob パターンを受け付け、複数のブランチ名に一致させることができます。名前にこれらの文字が含まれていて、それを文字どおりに一致させたい場合は、それぞれの特殊文字を `\` でエスケープする必要があります。glob パターンの詳細については、「GitHub Actions のワークフロー構文」を参照してください。

#### 例: ブランチを含める

`branches` で定義したパターンは Git ref の名前に対して評価されます。たとえば、次のワークフローは、pull request イベントが次のいずれかを対象としている場合に実行されます。

- `main` という名前のブランチ (`refs/heads/main`)
- `mona/octocat` という名前のブランチ (`refs/heads/mona/octocat`)
- `releases/` で始まる名前のブランチ。たとえば `releases/10` (`refs/heads/releases/10`)

```yaml
on:
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches:
      - main
      - 'mona/octocat'
      - 'releases/**'
```

ブランチフィルタリング、パスフィルタリング、またはコミットメッセージが原因でワークフローがスキップされた場合、そのワークフローに関連付けられたチェックは `Pending` 状態のままになります。それらのチェックの成功が必須である pull request はマージできなくなります。

#### 例: ブランチを除外する

パターンが `branches-ignore` パターンに一致した場合、ワークフローは実行されません。`branches-ignore` で定義したパターンは Git ref の名前に対して評価されます。たとえば、次のワークフローは、pull request イベントが次を対象としている場合を除き、実行されます。

- `mona/octocat` という名前のブランチ (`refs/heads/mona/octocat`)
- `releases/**-alpha` に一致する名前のブランチ。たとえば `releases/beta/3-alpha` (`refs/heads/releases/beta/3-alpha`)

```yaml
on:
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches-ignore:
      - 'mona/octocat'
      - 'releases/**-alpha'
```

#### 例: ブランチを含めて除外する

1 つのワークフロー内で、同じイベントを `branches` と `branches-ignore` の両方でフィルターすることはできません。単一のイベントについてブランチパターンを含めつつ除外もしたい場合は、`branches` フィルターと `!` 文字を組み合わせて、どのブランチを除外するかを示します。

`!` 文字を使ってブランチを定義する場合は、`!` を使わないブランチも少なくとも 1 つ定義する必要があります。ブランチを除外したいだけなら、代わりに `branches-ignore` を使用してください。

パターンを定義する順序は重要です。

- 正の一致の後に一致する否定パターン (`!` で始まる) があると、その Git ref は除外されます。
- 負の一致の後に一致する正のパターンがあると、その Git ref は再び含まれます。

次のワークフローは、`releases/10` または `releases/beta/mona` を対象とする pull request に対する `pull_request` イベントで実行されますが、`!releases/**-alpha` という否定パターンが正のパターンの後に続いているため、`releases/10-alpha` または `releases/beta/3-alpha` を対象とする pull request では実行されません。

```yaml
on:
  pull_request:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

### on.push.<branches|tags|branches-ignore|tags-ignore>

`push` イベントを使用する場合、特定のブランチまたはタグでワークフローを実行するよう設定できます。

ブランチ名パターンを含めたい場合、またはブランチ名パターンの包含と除外の両方を行いたい場合は `branches` フィルターを使用します。ブランチ名パターンを除外するだけの場合は `branches-ignore` フィルターを使用します。1 つのワークフロー内で同じイベントに対して `branches` と `branches-ignore` の両方を使用することはできません。

タグ名パターンを含めたい場合、またはタグ名パターンの包含と除外の両方を行いたい場合は `tags` フィルターを使用します。タグ名パターンを除外するだけの場合は `tags-ignore` フィルターを使用します。1 つのワークフロー内で同じイベントに対して `tags` と `tags-ignore` の両方を使用することはできません。

`tags`/`tags-ignore` のみ、または `branches`/`branches-ignore` のみを定義した場合、未定義の Git ref に影響するイベントではワークフローは実行されません。`tags`/`tags-ignore` と `branches`/`branches-ignore` のどちらも定義しない場合、ワークフローはブランチまたはタグのいずれかに影響するイベントで実行されます。`branches`/`branches-ignore` と `paths`/`paths-ignore` の両方を定義した場合、ワークフローは両方のフィルター条件を満たしたときにのみ実行されます。

`branches`、`branches-ignore`、`tags`、`tags-ignore` キーワードは、`*`、`**`、`+`、`?`、`!` などの文字を使用した glob パターンを受け付け、複数のブランチ名またはタグ名に一致させることができます。名前にこれらの文字が含まれていて、それを文字どおりに一致させたい場合は、それぞれの特殊文字を `\` でエスケープする必要があります。glob パターンの詳細については、「GitHub Actions のワークフロー構文」を参照してください。

#### 例: ブランチとタグを含める

`branches` と `tags` で定義したパターンは Git ref の名前に対して評価されます。たとえば、次のワークフローは、次のいずれかへの `push` イベントが発生したときに実行されます。

- `main` という名前のブランチ (`refs/heads/main`)
- `mona/octocat` という名前のブランチ (`refs/heads/mona/octocat`)
- `releases/` で始まる名前のブランチ。たとえば `releases/10` (`refs/heads/releases/10`)
- `v2` という名前のタグ (`refs/tags/v2`)
- `v1.` で始まる名前のタグ。たとえば `v1.9.1` (`refs/tags/v1.9.1`)

```yaml
on:
  push:
    # Sequence of patterns matched against refs/heads
    branches:
      - main
      - 'mona/octocat'
      - 'releases/**'
    # Sequence of patterns matched against refs/tags
    tags:
      - v2
      - v1.*
```

#### 例: ブランチとタグを除外する

パターンが `branches-ignore` または `tags-ignore` パターンに一致した場合、ワークフローは実行されません。`branches` と `tags` で定義したパターンは Git ref の名前に対して評価されます。たとえば、次のワークフローは、`push` イベントの対象が次の場合を除き、実行されます。

- `mona/octocat` という名前のブランチ (`refs/heads/mona/octocat`)
- `releases/**-alpha` に一致する名前のブランチ。たとえば `releases/beta/3-alpha` (`refs/heads/releases/beta/3-alpha`)
- `v2` という名前のタグ (`refs/tags/v2`)
- `v1.` で始まる名前のタグ。たとえば `v1.9` (`refs/tags/v1.9`)

```yaml
on:
  push:
    # Sequence of patterns matched against refs/heads
    branches-ignore:
      - 'mona/octocat'
      - 'releases/**-alpha'
    # Sequence of patterns matched against refs/tags
    tags-ignore:
      - v2
      - v1.*
```

#### 例: ブランチとタグを含めて除外する

1 つのワークフロー内で、同じイベントを `branches` と `branches-ignore` の両方でフィルターすることはできません。同様に、同じイベントを `tags` と `tags-ignore` の両方でフィルターすることもできません。単一のイベントについてブランチまたはタグのパターンを含めつつ除外もしたい場合は、`branches` または `tags` フィルターと `!` 文字を組み合わせて、どのブランチまたはタグを除外するかを示します。

`!` 文字を使ってブランチを定義する場合は、`!` を使わないブランチも少なくとも 1 つ定義する必要があります。ブランチを除外したいだけなら、代わりに `branches-ignore` を使用してください。同様に、`!` 文字を使ってタグを定義する場合は、`!` を使わないタグも少なくとも 1 つ定義する必要があります。タグを除外したいだけなら、代わりに `tags-ignore` を使用してください。

パターンを定義する順序は重要です。

- 正の一致の後に一致する否定パターン (`!` で始まる) があると、その Git ref は除外されます。
- 負の一致の後に一致する正のパターンがあると、その Git ref は再び含まれます。

次のワークフローは、`releases/10` または `releases/beta/mona` への `push` で実行されますが、`!releases/**-alpha` という否定パターンが正のパターンの後に続いているため、`releases/10-alpha` や `releases/beta/3-alpha` では実行されません。

```yaml
on:
  push:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

### on.<push|pull_request|pull_request_target>.<paths|paths-ignore>

`push` イベントおよび `pull_request` イベントを使用する場合、どのファイルパスが変更されたかに基づいてワークフローを実行するよう設定できます。パスフィルターはタグへの `push` では評価されません。

ファイルパスのパターンを含めたい場合、またはファイルパスのパターンの包含と除外の両方を行いたい場合は `paths` フィルターを使用します。ファイルパスのパターンを除外するだけの場合は `paths-ignore` フィルターを使用します。1 つのワークフロー内で同じイベントに対して `paths` と `paths-ignore` の両方を使用することはできません。単一のイベントについてパスパターンを含めつつ除外もしたい場合は、`!` 文字を前置した `paths` フィルターを使用して、どのパスを除外するかを示します。

> **メモ**
>
> `paths` パターンを定義する順序は重要です。
>
> - 正の一致の後に一致する否定パターン (`!` で始まる) があると、そのパスは除外されます。
> - 負の一致の後に一致する正のパターンがあると、そのパスは再び含まれます。
> - `branches`/`branches-ignore` と `paths`/`paths-ignore` の両方を定義した場合、ワークフローは両方のフィルター条件を満たしたときにのみ実行されます。

`paths` および `paths-ignore` キーワードは、`*` および `**` ワイルドカード文字を使用した glob パターンを受け付け、複数のパス名に一致させることができます。詳細については、「GitHub Actions のワークフロー構文」を参照してください。

#### 例: パスを含める

`paths` フィルター内のパターンに少なくとも 1 つのパスが一致した場合、ワークフローは実行されます。たとえば、次のワークフローは JavaScript ファイル (`.js`) を push するたびに実行されます。

```yaml
on:
  push:
    paths:
      - '**.js'
```

パスフィルタリング、ブランチフィルタリング、またはコミットメッセージが原因でワークフローがスキップされた場合、そのワークフローに関連付けられたチェックは `Pending` 状態のままになります。それらのチェックの成功が必須である pull request はマージできなくなります。

#### 例: パスを除外する

すべてのパス名が `paths-ignore` 内のパターンに一致する場合、ワークフローは実行されません。いくつかのパス名がそのパターンに一致していても、1 つでも `paths-ignore` 内のパターンに一致しないパス名があれば、ワークフローは実行されます。

次のパスフィルターを持つワークフローは、リポジトリのルートにある `docs` ディレクトリの外部に少なくとも 1 つのファイルが含まれる `push` イベントでのみ実行されます。

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
```

#### 例: パスを含めて除外する

1 つのワークフロー内で、同じイベントを `paths` と `paths-ignore` の両方でフィルターすることはできません。単一のイベントについてパスパターンを含めつつ除外もしたい場合は、`!` 文字を前置した `paths` フィルターを使用して、どのパスを除外するかを示します。

`!` 文字を使ってパスを定義する場合は、`!` を使わないパスも少なくとも 1 つ定義する必要があります。パスを除外したいだけなら、代わりに `paths-ignore` を使用してください。

`paths` パターンを定義する順序は重要です。

- 正の一致の後に一致する否定パターン (`!` で始まる) があると、そのパスは除外されます。
- 負の一致の後に一致する正のパターンがあると、そのパスは再び含まれます。

この例は、`sub-project/docs` ディレクトリ内のファイルを除き、`sub-project` ディレクトリまたはそのサブディレクトリ内のファイルが `push` イベントに含まれるたびに実行されます。たとえば、`sub-project/index.js` または `sub-project/src/index.js` を変更する push はワークフロー実行をトリガーしますが、`sub-project/docs/readme.md` だけを変更する push はトリガーしません。

```yaml
on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
```

#### Git diff の比較

> **メモ**
>
> 1,000 件を超えるコミットを push した場合、またはタイムアウトのために GitHub が diff を生成しない場合、ワークフローは常に実行されます。

フィルターは、変更されたファイルを評価し、それらを `paths-ignore` または `paths` の一覧に照らして判定することで、ワークフローを実行すべきかどうかを決定します。変更されたファイルがない場合、ワークフローは実行されません。

GitHub は、変更されたファイルの一覧を、push には two-dot diff、pull request には three-dot diff を使用して生成します。

- Pull request: three-dot diff は、トピックブランチの最新バージョンと、そのトピックブランチが最後にベースブランチと同期されたコミットとの比較です。
- 既存のブランチへの push: two-dot diff は、head SHA と base SHA を互いに直接比較します。
- 新しいブランチへの push: push された最も深いコミットの祖先の親に対する two-dot diff です。

> **メモ**
>
> diff は 300 ファイルに制限されています。フィルターが返す最初の 300 ファイルに一致しない変更ファイルがある場合、ワークフローは実行されません。ワークフローを自動的に実行するには、より具体的なフィルターを作成する必要がある場合があります。

詳細については、「pull request でのブランチ比較について」を参照してください。

### on.schedule

`on.schedule` を使用すると、ワークフローの時間スケジュールを定義できます。

POSIX cron 構文を使用して、特定の時刻にワークフローを実行するようスケジュールできます。既定では、スケジュールされたワークフローは UTC で実行されます。必要に応じて、IANA タイムゾーン文字列を使用してタイムゾーンを指定し、タイムゾーンを考慮したスケジューリングを行うこともできます。スケジュールされたワークフローは、デフォルトブランチ上の最新コミットで実行されます。スケジュールされたワークフローを実行できる最短間隔は 5 分ごとです。

> **メモ**
>
> `timezone` に夏時間 (DST) を採用するタイムゾーンを設定したスケジュールでは、DST の春の繰り上げ移行中に存在しない時間帯に設定されたワークフローは、次に有効な時刻へ進みます。たとえば、午前 2:30 のスケジュールは午前 3:00 に繰り上がります。

cron 構文はスペースで区切られた 5 つのフィールドで構成され、各フィールドは時間の単位を表します。

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of the month (1 - 31)
│ │ │ ┌───────────── month (1 - 12 or JAN-DEC)
│ │ │ │ ┌───────────── day of the week (0 - 6 or SUN-SAT)
│ │ │ │ │
* * * * *
```

5 つの各フィールドでは、次の演算子を使用できます。

| 演算子 | 説明 | 例 |
| --- | --- | --- |
| `*` | 任意の値 | `15 * * * *` は、毎日毎時 15 分に実行されます。 |
| `,` | 値リストの区切り | `2,10 4,5 * * *` は、毎日 4 時台と 5 時台の 2 分および 10 分に実行されます。 |
| `-` | 値の範囲 | `30 4-6 * * *` は、4 時台、5 時台、6 時台の 30 分に実行されます。 |
| `/` | ステップ値 | `20/15 * * * *` は、20 分から 59 分までの間で 15 分ごと (20 分、35 分、50 分) に実行されます。 |

この例では、`America/New_York` タイムゾーンで毎週月曜日から金曜日の午前 5:30 にワークフローが実行されます。

```yaml
on:
  schedule:
    - cron: '30 5 * * 1-5'
      timezone: "America/New_York"
```

1 つのワークフローを複数の `schedule` イベントでトリガーできます。ワークフローをトリガーした `schedule` イベントには `github.event.schedule` コンテキストからアクセスします。この例では、ワークフローは毎週月曜日と水曜日の 5:30 UTC、および毎週火曜日と木曜日の 5:30 UTC と 17:30 UTC に実行されますが、`Not on Monday or Wednesday` ステップは月曜日と水曜日にはスキップされます。

```yaml
on:
  schedule:
    - cron: '30 5 * * 1,3'
    - cron: '30 5,17 * * 2,4'

jobs:
  test_schedule:
    runs-on: ubuntu-latest
    steps:
      - name: Not on Monday or Wednesday
        if: github.event.schedule != '30 5 * * 1,3'
        run: echo "This step will be skipped on Monday and Wednesday"
      - name: Every time
        run: echo "This step will always run"
```

`schedule` イベントの詳細については、「ワークフローをトリガーするイベント」を参照してください。

### on.workflow_call

`on.workflow_call` を使用すると、再利用可能なワークフローの入力と出力を定義できます。また、呼び出されたワークフローで利用できるシークレットをマッピングすることもできます。再利用可能なワークフローの詳細については、「ワークフローの再利用」を参照してください。

### on.workflow_call.inputs

`workflow_call` キーワードを使用する場合、呼び出し元ワークフローから呼び出されたワークフローへ渡される入力を任意で指定できます。`workflow_call` キーワードの詳細については、「ワークフローをトリガーするイベント」を参照してください。

利用可能な標準入力パラメーターに加えて、`on.workflow_call.inputs` では `type` パラメーターが必須です。詳細については、`on.workflow_call.inputs.<input_id>.type` を参照してください。

`default` パラメーターが設定されていない場合、入力の既定値は `boolean` では `false`、`number` では `0`、`string` では `""` です。

呼び出されたワークフロー内では、`inputs` コンテキストを使用して入力を参照できます。詳細については、「Contexts reference」を参照してください。

呼び出し元ワークフローが、呼び出されたワークフローで指定されていない入力を渡した場合はエラーになります。

#### on.workflow_call.inputs の例

```yaml
on:
  workflow_call:
    inputs:
      username:
        description: 'A username passed from the caller workflow'
        default: 'john-doe'
        required: false
        type: string

jobs:
  print-username:
    runs-on: ubuntu-latest

    steps:
      - name: Print the input name to STDOUT
        run: echo The username is ${{ inputs.username }}
```

詳細については、「ワークフローの再利用」を参照してください。

### on.workflow_call.inputs.<input_id>.type

`on.workflow_call` キーワードに対して入力を定義する場合は必須です。このパラメーターの値は、入力のデータ型を指定する文字列です。指定できる値は `boolean`、`number`、`string` のいずれかです。

### on.workflow_call.outputs

呼び出されたワークフローの出力のマップです。呼び出されたワークフローの出力は、呼び出し元ワークフロー内のすべての下流ジョブで利用できます。各出力には識別子、省略可能な説明、および値があります。値には、呼び出されたワークフロー内のジョブの出力値を設定する必要があります。

次の例では、この再利用可能なワークフローに対して `workflow_output1` と `workflow_output2` という 2 つの出力が定義されています。これらは、`my_job` というジョブの `job_output1` と `job_output2` という出力にマッピングされています。

#### on.workflow_call.outputs の例

```yaml
on:
  workflow_call:
    # Map the workflow outputs to job outputs
    outputs:
      workflow_output1:
        description: "The first job output"
        value: ${{ jobs.my_job.outputs.job_output1 }}
      workflow_output2:
        description: "The second job output"
        value: ${{ jobs.my_job.outputs.job_output2 }}
```

ジョブ出力の参照方法については `jobs.<job_id>.outputs` を参照してください。詳細については、「ワークフローの再利用」を参照してください。

### on.workflow_call.secrets

呼び出されたワークフローで使用できるシークレットのマップです。

呼び出されたワークフロー内では、`secrets` コンテキストを使用してシークレットを参照できます。

> **メモ**
>
> シークレットを入れ子になった再利用可能ワークフローに渡す場合は、`jobs.<job_id>.secrets` を再度使用してシークレットを渡す必要があります。詳細については、「ワークフローの再利用」を参照してください。

呼び出し元ワークフローが、呼び出されたワークフローで指定されていないシークレットを渡した場合はエラーになります。

#### on.workflow_call.secrets の例

```yaml
on:
  workflow_call:
    secrets:
      access-token:
        description: 'A token passed from the caller workflow'
        required: false

jobs:

  pass-secret-to-action:
    runs-on: ubuntu-latest
    steps:
    # passing the secret to an action
      - name: Pass the received secret to an action
        uses: ./.github/actions/my-action
        with:
          token: ${{ secrets.access-token }}

  # passing the secret to a nested reusable workflow
  pass-secret-to-workflow:
    uses: ./.github/workflows/my-workflow
    secrets:
       token: ${{ secrets.access-token }}
```

### on.workflow_call.secrets.<secret_id>

シークレットに関連付ける文字列識別子です。

### on.workflow_call.secrets.<secret_id>.required

シークレットを指定する必要があるかどうかを示すブール値です。

### on.workflow_run.<branches|branches-ignore>

`workflow_run` イベントを使用する場合、どのブランチでトリガー元のワークフローが実行されたときにこのワークフローをトリガーするかを指定できます。

`branches` および `branches-ignore` フィルターは、`*`、`**`、`+`、`?`、`!` などの文字を使用した glob パターンを受け付け、複数のブランチ名に一致させることができます。名前にこれらの文字が含まれていて、それを文字どおりに一致させたい場合は、それぞれの特殊文字を `\` でエスケープする必要があります。glob パターンの詳細については、「GitHub Actions のワークフロー構文」を参照してください。

たとえば、次のトリガーを持つワークフローは、`Build` という名前のワークフローが `releases/` で始まる名前のブランチで実行された場合にのみ実行されます。

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [requested]
    branches:
      - 'releases/**'
```

次のトリガーを持つワークフローは、`Build` という名前のワークフローが `canary` ではないブランチで実行された場合にのみ実行されます。

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [requested]
    branches-ignore:
      - "canary"
```

1 つのワークフロー内で、同じイベントに対して `branches` と `branches-ignore` の両方を使用することはできません。単一のイベントについてブランチパターンを含めつつ除外もしたい場合は、`branches` フィルターと `!` 文字を組み合わせて、どのブランチを除外するかを示します。

パターンを定義する順序は重要です。

- 正の一致の後に一致する否定パターン (`!` で始まる) があると、そのブランチは除外されます。
- 負の一致の後に一致する正のパターンがあると、そのブランチは再び含まれます。

たとえば、次のトリガーを持つワークフローは、`Build` という名前のワークフローが `releases/10` または `releases/beta/mona` という名前のブランチで実行されたときには実行されますが、`releases/10-alpha`、`releases/beta/3-alpha`、または `main` では実行されません。

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [requested]
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

### on.workflow_dispatch

`workflow_dispatch` イベントを使用する場合、ワークフローに渡される入力を任意で指定できます。

このトリガーがイベントを受け取るのは、ワークフローファイルがデフォルトブランチ上にある場合だけです。

### on.workflow_dispatch.inputs

トリガーされたワークフローは、`inputs` コンテキストで入力を受け取ります。詳細については、「Contexts」を参照してください。

> **メモ**
>
> ワークフローは `github.event.inputs` コンテキストでも入力を受け取ります。`inputs` コンテキストと `github.event.inputs` コンテキストの情報は同一ですが、`inputs` コンテキストではブール値が文字列に変換されず、ブール値のまま保持される点が異なります。`choice` 型は文字列に解決され、単一選択のオプションです。
>
> 入力に指定できる最上位プロパティの最大数は 25 です。
>
> 入力の最大ペイロードは 65,535 文字です。

#### on.workflow_dispatch.inputs の例

```yaml
on:
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'
        required: true
        default: 'warning'
        type: choice
        options:
          - info
          - warning
          - debug
      print_tags:
        description: 'True to print to STDOUT'
        required: true
        type: boolean
      tags:
        description: 'Test scenario tags'
        required: true
        type: string
      environment:
        description: 'Environment to run tests against'
        type: environment
        required: true

jobs:
  print-tag:
    runs-on: ubuntu-latest
    if: ${{ inputs.print_tags }} 
    steps:
      - name: Print the input tag to STDOUT
        run: echo  The tags are ${{ inputs.tags }} 
```

### on.workflow_dispatch.inputs.<input_id>.required

入力を指定する必要があるかどうかを示すブール値です。

### on.workflow_dispatch.inputs.<input_id>.type

このパラメーターの値は、入力のデータ型を指定する文字列です。指定できる値は `boolean`、`choice`、`number`、`environment`、`string` のいずれかです。

## permissions

`permissions` を使うと、`GITHUB_TOKEN` に付与される既定の権限を変更できます。必要に応じてアクセス権を追加または削除し、必要最小限のアクセスのみを許可できます。詳細については、ワークフローで `GITHUB_TOKEN` を認証に使用する方法を参照してください。

`permissions` は、ワークフロー内のすべてのジョブに適用するトップレベルキーとして使用することも、特定のジョブ内で使用することもできます。特定のジョブ内に `permissions` キーを追加すると、そのジョブ内で `GITHUB_TOKEN` を使用するすべてのアクションと `run` コマンドに、指定したアクセス権が付与されます。詳細については、`jobs.<job_id>.permissions` を参照してください。

Organization の所有者は、リポジトリ レベルで `GITHUB_TOKEN` の書き込みアクセスを制限できます。詳細については、Organization の GitHub Actions を無効化または制限する方法を参照してください。

ワークフローが `pull_request_target` イベントによってトリガーされる場合、公開フォークからトリガーされた場合でも、`GITHUB_TOKEN` にはリポジトリへの読み取り/書き込み権限が付与されます。詳細については、ワークフローをトリガーするイベントを参照してください。

以下の表に示す各権限について、アクセス レベルとして `read`（該当する場合）、`write`、`none` のいずれかを割り当てることができます。`write` には `read` が含まれます。これらの権限のいずれかに対してアクセスを指定した場合、指定しなかった権限はすべて `none` に設定されます。

#### 利用可能な権限と、各権限でアクションに許可される操作

| 権限 | `GITHUB_TOKEN` を使用するアクションで可能になること |
| --- | --- |
| actions | GitHub Actions を操作します。たとえば、`actions: write` はアクションによるワークフロー実行のキャンセルを許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| artifact-metadata | artifact metadata を操作します。たとえば、`artifact-metadata: write` は、ビルド artifact を代表してアクションがストレージ レコードを作成することを許可します。詳細については、artifact metadata の REST API エンドポイントを参照してください。 |
| attestations | artifact attestations を操作します。たとえば、`attestations: write` は、ビルド用の artifact attestation をアクションが生成することを許可します。詳細については、ビルドの来歴を確立するための artifact attestations の使用を参照してください。 |
| checks | check runs と check suites を操作します。たとえば、`checks: write` はアクションによる check run の作成を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| code-quality | code quality を操作します。たとえば、`code-quality: write` はアクションによるコード カバレッジ レポートのアップロードを許可します。詳細については、GitHub Code Quality についてを参照してください。 |
| contents | リポジトリの内容を操作します。たとえば、`contents: read` はアクションによるコミット一覧の取得を許可し、`contents: write` はアクションによるリリースの作成を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| deployments | deployments を操作します。たとえば、`deployments: write` はアクションによる新しい deployment の作成を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| discussions | GitHub Discussions を操作します。たとえば、`discussions: write` はアクションによる discussion のクローズまたは削除を許可します。詳細については、Discussions 向け GraphQL API の使用を参照してください。 |
| id-token | OpenID Connect (OIDC) トークンを取得します。これには `id-token: write` が必要です。詳細については、OpenID Connect を参照してください。 |
| issues | issues を操作します。たとえば、`issues: write` はアクションによる issue へのコメント追加を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| models | GitHub Models で AI 推論レスポンスを生成します。たとえば、`models: read` はアクションによる GitHub Models 推論 API の使用を許可します。AI モデルでのプロトタイピングを参照してください。 |
| packages | GitHub Packages を操作します。たとえば、`packages: write` はアクションによる GitHub Packages へのパッケージのアップロードと公開を許可します。詳細については、GitHub Packages の権限についてを参照してください。 |
| pages | GitHub Pages を操作します。たとえば、`pages: write` はアクションによる GitHub Pages ビルド要求を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| pull-requests | pull requests を操作します。たとえば、`pull-requests: write` はアクションによる pull request へのラベル追加を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| security-events | GitHub code scanning alerts を操作します。たとえば、`security-events: read` はアクションによるリポジトリの code scanning alerts 一覧の取得を許可し、`security-events: write` はアクションによる code scanning alert の状態更新を許可します。詳細については、「Code scanning alerts」のリポジトリ権限を参照してください。 |
| statuses | commit statuses を操作します。たとえば、`statuses:read` はアクションによる特定の参照の commit statuses 一覧取得を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| vulnerability-alerts | Dependabot alerts を読み取ります。たとえば、`vulnerability-alerts: read` はアクションによるリポジトリの Dependabot alerts 一覧取得を許可します。サポートされるのは `read` と `none` のみで、`write` は無効です。`write-all` または `read-all` を使用すると、`vulnerability-alerts` は自動的に `read` として含まれます。詳細については、「Dependabot alerts」のリポジトリ権限を参照してください。 |

Dependabot alerts には `vulnerability-alerts` 権限を使用してください。secret scanning alerts はこの権限では読み取れず、GitHub App または個人用アクセス トークンが必要です。詳細については、「GitHub Apps に必要な権限」内の「Secret scanning alerts」のリポジトリ権限を参照してください。

#### GITHUB_TOKEN スコープのアクセス定義

`permissions` キー内で利用可能な権限の値として `read`、`write`、`none` を指定することで、`GITHUB_TOKEN` が許可するアクセスを定義できます。

```yaml
permissions:
  actions: read|write|none
  artifact-metadata: read|write|none
  attestations: read|write|none
  checks: read|write|none
  code-quality: read|write|none
  contents: read|write|none
  deployments: read|write|none
  id-token: write|none
  issues: read|write|none
  models: read|none
  discussions: read|write|none
  packages: read|write|none
  pages: read|write|none
  pull-requests: read|write|none

  security-events: read|write|none
  statuses: read|write|none
  vulnerability-alerts: read|none
```

これらの権限のいずれかに対してアクセスを指定した場合、指定しなかった権限はすべて `none` に設定されます。

利用可能なすべての権限に対して `read-all` または `write-all` のいずれかのアクセスを定義するには、次の構文を使用できます。

```yaml
permissions: read-all
permissions: write-all
```

利用可能なすべての権限を無効にするには、次の構文を使用できます。

```yaml
permissions: {}
```

#### フォークされたリポジトリでの権限変更

`permissions` キーを使用して、フォークされたリポジトリに対する読み取り権限を追加または削除できますが、通常は書き込みアクセスを付与できません。この動作の例外は、管理者ユーザーが GitHub Actions の設定で **Send write tokens to workflows from pull requests** オプションを選択している場合です。詳細については、リポジトリの GitHub Actions 設定の管理を参照してください。

#### ワークフロージョブの権限の計算方法

`GITHUB_TOKEN` の権限は、最初に enterprise、organization、または repository の既定設定に基づいて設定されます。これらいずれかのレベルで既定が制限付き権限に設定されている場合、その設定が該当するリポジトリに適用されます。たとえば、organization レベルで制限付きの既定を選択すると、その organization 内のすべてのリポジトリが既定として制限付き権限を使用します。その後、ワークフローファイル内の設定に基づいて権限が調整され、まずワークフローレベル、次にジョブレベルの順に適用されます。最後に、ワークフローがフォークされたリポジトリからの `pull_request_target` 以外の pull request イベントによってトリガーされ、かつ **Send write tokens to workflows from pull requests** 設定が選択されていない場合、権限は調整されて、書き込み権限はすべて読み取り専用に変更されます。

#### すべてのジョブに対する GITHUB_TOKEN 権限の設定

ワークフローのトップレベルで `permissions` を指定すると、その設定をワークフロー内のすべてのジョブに適用できます。

#### 例: ワークフロー全体の GITHUB_TOKEN 権限を設定する

この例では、ワークフロー内のすべてのジョブに適用される `GITHUB_TOKEN` の権限を設定しています。すべての権限に読み取りアクセスが付与されます。

```yaml
name: "My workflow"

on: [ push ]

permissions: read-all

jobs:
  ...
```

#### フォークされたリポジトリで `permissions` キーを使う

`permissions` キーを使用して、フォークされたリポジトリに対する読み取り権限を追加または削除できますが、通常は書き込みアクセスを付与できません。この動作の例外は、管理者ユーザーが GitHub Actions の設定で **Send write tokens to workflows from pull requests** オプションを選択している場合です。詳細については、リポジトリの GitHub Actions 設定の管理を参照してください。

#### Dependabot によってトリガーされたワークフロー実行の権限

Dependabot pull request によってトリガーされたワークフロー実行は、フォークされたリポジトリからの実行であるかのように動作するため、読み取り専用の `GITHUB_TOKEN` を使用します。これらのワークフロー実行は、いかなる secrets にもアクセスできません。これらのワークフローを安全に保つための戦略については、Secure use reference を参照してください。

## env

`env` は、ワークフロー内のすべてのジョブのステップで利用できる変数のマップです。また、単一のジョブのステップでのみ利用できる変数や、単一のステップでのみ利用できる変数を設定することもできます。詳細については、`jobs.<job_id>.env` と `jobs.<job_id>.steps[*].env` を参照してください。

`env` マップ内の変数を、同じマップ内のほかの変数を使って定義することはできません。

同じ名前の環境変数が複数定義されている場合、GitHub は最も具体的な変数を使用します。たとえば、ステップで定義された環境変数は、そのステップの実行中、同じ名前のジョブおよびワークフローの環境変数を上書きします。ジョブに対して定義された環境変数は、そのジョブの実行中、同じ名前のワークフロー変数を上書きします。

#### 例: `env`

```yaml
env:
  SERVER: production
```

## defaults

`defaults` を使うと、ワークフロー内のすべてのジョブに適用される既定設定のマップを作成できます。また、ジョブでのみ利用できる既定設定を指定することもできます。詳細については、`jobs.<job_id>.defaults` を参照してください。

同じ名前の既定設定が複数定義されている場合、GitHub は最も具体的な既定設定を使用します。たとえば、ジョブで定義された既定設定は、ワークフローで同じ名前で定義された既定設定を上書きします。

### `defaults.run`

`defaults.run` を使うと、ワークフロー内のすべての `run` ステップに対して、既定の `shell` と `working-directory` オプションを指定できます。また、ジョブでのみ利用できる `run` の既定設定を指定することもできます。詳細については、`jobs.<job_id>.defaults.run` を参照してください。このキーワードでは contexts や expressions は使用できません。

同じ名前の既定設定が複数定義されている場合、GitHub は最も具体的な既定設定を使用します。たとえば、ジョブで定義された既定設定は、ワークフローで同じ名前で定義された既定設定を上書きします。

#### 例: デフォルトのシェルと作業ディレクトリを設定する

```yaml
defaults:
  run:
    shell: bash
    working-directory: ./scripts
```

### `defaults.run.shell`

`shell` を使うと、ステップのシェルを定義できます。このキーワードでは複数の contexts を参照できます。詳細については、Contexts を参照してください。

| 対応プラットフォーム | `shell` パラメーター | 説明 | 内部で実行されるコマンド |
| --- | --- | --- | --- |
| Linux / macOS | unspecified | Windows 以外のプラットフォームでの既定のシェルです。なお、`bash` を明示的に指定した場合とは異なるコマンドが実行されます。`bash` がパス上に見つからない場合は、`sh` として扱われます。 | `bash -e {0}` |
| All | bash | Windows 以外のプラットフォームでの既定のシェルで、`sh` へのフォールバックがあります。Windows で `bash` シェルを指定した場合は、Git for Windows に含まれる `bash` シェルが使用されます。 | `bash --noprofile --norc -eo pipefail {0}` |
| All | pwsh | PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を付加します。 | `pwsh -command ". '{0}'"` |
| All | python | `python` コマンドを実行します。 | `python {0}` |
| Linux / macOS | sh | シェルが指定されておらず、かつ `bash` がパス上に見つからない場合の、Windows 以外のプラットフォームでのフォールバック動作です。 | `sh -e {0}` |
| Windows | cmd | GitHub はスクリプト名に拡張子 `.cmd` を付加し、`{0}` を置き換えます。 | `%ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}""` |
| Windows | pwsh | これは Windows で使用される既定のシェルです。PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を付加します。self-hosted の Windows runner に PowerShell Core がインストールされていない場合は、代わりに PowerShell Desktop が使用されます。 | `pwsh -command ". '{0}'".` |
| Windows | powershell | PowerShell Desktop です。GitHub はスクリプト名に拡張子 `.ps1` を付加します。 | `powershell -command ". '{0}'".` |

同じ名前の既定設定が複数定義されている場合、GitHub は最も具体的な既定設定を使用します。たとえば、ジョブで定義された既定設定は、ワークフローで同じ名前で定義された既定設定を上書きします。

### `defaults.run.working-directory`

`working-directory` を使うと、ステップのシェルに対する作業ディレクトリを定義できます。このキーワードでは複数の contexts を参照できます。詳細については、Contexts を参照してください。

> **ヒント**
>
> 割り当てる `working-directory` が、その中でシェルを実行する前に runner 上に存在していることを確認してください。同じ名前の既定設定が複数定義されている場合、GitHub は最も具体的な既定設定を使用します。たとえば、ジョブで定義された既定設定は、ワークフローで同じ名前で定義された既定設定を上書きします。

## concurrency

`concurrency` を使うと、同じ concurrency グループを使用するジョブまたはワークフローが、一度に 1 つだけ実行されるようにできます。concurrency グループには任意の文字列または expression を使用できます。expression で使用できるのは `github`、`inputs`、`vars` の contexts のみです。expressions の詳細については、ワークフローとアクションでの式の評価を参照してください。

ジョブ レベルで `concurrency` を指定することもできます。詳細については、`jobs.<job_id>.concurrency` を参照してください。

これは、ある concurrency グループ内で同時に実行中でいられるジョブまたはワークフローは最大 1 つだけであることを意味します。並行するジョブまたはワークフローがキューに追加されたとき、リポジトリ内で同じ concurrency グループを使用する別のジョブまたはワークフローが進行中であれば、キューに追加されたジョブまたはワークフローは保留状態になります。既定では、同じ concurrency グループ内に既に存在する保留中のジョブまたはワークフローはキャンセルされ、新しくキューに追加されたジョブまたはワークフローがその代わりになります。

同じ concurrency グループ内で現在実行中のジョブまたはワークフローもキャンセルするには、`cancel-in-progress: true` を指定します。同じ concurrency グループ内で現在実行中のジョブまたはワークフローを条件付きでキャンセルするには、許可されている expression contexts のいずれかを使った expression として `cancel-in-progress` を指定できます。

同じ concurrency グループ内で、保留中のジョブまたはワークフロー実行を 2 つ以上待機させたい場合は、任意の `queue` プロパティを使用します。`queue` プロパティには、次の値を指定できます。

- `single`（既定）: concurrency グループ内で保留状態にできるジョブまたはワークフロー実行は最大 1 つです。新しいジョブまたはワークフロー実行がキューに追加されると、同じグループ内の既存の保留中ジョブまたはワークフロー実行はキャンセルされて置き換えられます。
- `max`: concurrency グループ内で最大 100 件のジョブまたはワークフロー実行を保留状態にできます。キューが満杯になると、それ以上のジョブまたはワークフロー実行はキャンセルされます。

`queue: max` と `cancel-in-progress: true` の組み合わせは許可されておらず、ワークフローの検証エラーになります。

> **メモ**
>
> concurrency グループ名では大文字と小文字は区別されません。たとえば、`prod` と `Prod` は同じ concurrency グループとして扱われます。
> 同じ concurrency グループ内のジョブまたはワークフロー実行は、各ワークフローがディスパッチされた時刻ではなく、各ジョブまたは実行が concurrency グループで待機を開始した時刻に基づく先入れ先出し（FIFO）順で処理されます。実際の開始時刻はジョブまたは実行ごとに異なる場合があるため、順序は保証されません。

#### 例: `concurrency` と既定の動作を使う

GitHub Actions の既定の動作では、複数のジョブまたはワークフロー実行を並行して実行できます。`concurrency` キーワードを使うと、ワークフロー実行の並行性を制御できます。

たとえば、トリガー条件を定義した直後に `concurrency` キーワードを使用して、特定のブランチに対するワークフロー実行全体の並行性を制限できます。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

また、ジョブ レベルで `concurrency` キーワードを使用して、ワークフロー内のジョブの並行性を制限することもできます。

```yaml
on:
  push:
    branches:
      - main

jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: example-group
      cancel-in-progress: true
```

#### 例: concurrency グループ

concurrency グループは、同じ concurrency キーを共有するワークフロー実行またはジョブの実行を管理および制限する方法を提供します。

concurrency キーは、ワークフローまたはジョブを concurrency グループにまとめるために使用されます。concurrency キーを定義すると、そのキーを持つワークフローまたはジョブは、任意の時点で 1 つだけ実行されることが GitHub Actions によって保証されます。同じ concurrency キーを持つ新しいワークフロー実行またはジョブが開始されると、GitHub Actions は、そのキーですでに実行中のワークフローまたはジョブをキャンセルします。concurrency キーには固定文字列を使用することも、context 変数を含む動的 expression を使用することもできます。

ワークフロー内で concurrency 条件を定義し、そのワークフローまたはジョブを concurrency グループの一部にすることができます。

これは、ワークフロー実行またはジョブが開始されると、GitHub が同じ concurrency グループですでに進行中のワークフロー実行またはジョブをキャンセルすることを意味します。これは、staging 環境へのデプロイに使用されるワークフローやジョブのように、競合を引き起こしたり必要以上にリソースを消費したりする可能性があるアクションを防ぐために、特定の一連のワークフローまたはジョブの並列実行を防ぎたい場面で役立ちます。

この例では、`job-1` は `staging_environment` という名前の concurrency グループに属しています。これは、`job-1` の新しい実行がトリガーされた場合、その時点ですでに進行中である `staging_environment` concurrency グループ内の同じジョブの実行がキャンセルされることを意味します。

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: staging_environment
      cancel-in-progress: true
```

別の方法として、ワークフローで `concurrency: ci-${{ github.ref }}` のような動的 expression を使用すると、そのワークフローまたはジョブは、ワークフローをトリガーしたブランチまたはタグの参照の後ろに `ci-` を付けた名前の concurrency グループの一部になります。この例では、前回の実行がまだ進行中の間に `main` ブランチへ新しいコミットがプッシュされると、前回の実行がキャンセルされて新しい実行が開始されます。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

#### 例: 複数の保留中実行をキューに入れる

既定では、concurrency グループ内で保留状態にできるジョブまたはワークフロー実行は一度に 1 つだけです。キャンセルせずに複数の実行をキューに入れられるようにするには、`queue: max` を設定します。`queue: max` を使用すると、最大 100 件のジョブまたはワークフロー実行を concurrency グループ内で待機させることができ、キューが満杯になるとそれ以上の実行はキャンセルされます。

たとえば、次のワークフローは production 環境へのデプロイをキューに入れ、各実行が concurrency グループで待機を開始した順に、1 件ずつ処理します。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: production-deploy
  queue: max
```

`queue: max` は `cancel-in-progress: true` と組み合わせることができない点に注意してください。2 つのオプションは、進行中の実行の扱いとして相反する動作を表すためです。

#### 例: `concurrency` を使って進行中のジョブまたは実行をキャンセルする

GitHub Actions で `concurrency` を使って進行中のジョブまたは実行をキャンセルするには、`cancel-in-progress` オプションを `true` に設定した `concurrency` キーを使用できます。

```yaml
concurrency:
  group: ${{ github.ref }}
  cancel-in-progress: true
```

この例では、特定の concurrency グループを定義していないため、GitHub Actions はそのジョブまたはワークフローの進行中の実行をすべてキャンセルすることに注意してください。

#### 例: フォールバック値を使う

特定のイベントでのみ定義されるプロパティを使ってグループ名を構築する場合は、フォールバック値を使用できます。たとえば、`github.head_ref` は `pull_request` イベントでのみ定義されます。ワークフローが `pull_request` イベント以外にも応答する場合、構文エラーを避けるためにフォールバックを指定する必要があります。次の concurrency グループは、`pull_request` イベントでのみ進行中のジョブまたは実行をキャンセルします。`github.head_ref` が未定義の場合、concurrency グループは実行 ID にフォールバックします。これは、その実行に対して一意であり、かつ必ず定義されていることが保証されています。

```yaml
concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
```

#### 例: 現在のワークフローの進行中ジョブまたは実行だけをキャンセルする

同じリポジトリに複数のワークフローがある場合、ほかのワークフローの進行中ジョブまたは実行を誤ってキャンセルしないように、concurrency グループ名はワークフロー間で一意である必要があります。そうでない場合、ワークフローに関係なく、それ以前に進行中または保留中だったジョブがキャンセルされます。

同じワークフローの進行中実行だけをキャンセルするには、`github.workflow` プロパティを使って concurrency グループを構築できます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

#### 例: 特定のブランチでのみ進行中のジョブをキャンセルする

特定のブランチでは進行中のジョブをキャンセルし、ほかのブランチではキャンセルしたくない場合、`cancel-in-progress` と条件式を組み合わせて使用できます。たとえば、release ブランチではなく development ブランチで進行中ジョブをキャンセルしたい場合に、この方法を使えます。

release ブランチで実行している場合を除いて、同じワークフローの進行中実行だけをキャンセルするには、`cancel-in-progress` を次のような expression に設定できます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !contains(github.ref, 'release/')}}
```

この例では、`release/1.2.3` ブランチに対する複数回の push は、進行中の実行をキャンセルしません。`main` など別のブランチへの push は、進行中の実行をキャンセルします。

## jobs

1 回のワークフロー実行は 1 つ以上のジョブで構成され、既定ではそれらは並列に実行されます。ジョブを順番に実行するには、`jobs.<job_id>.needs` キーワードを使ってほかのジョブへの依存関係を定義できます。

各ジョブは、`runs-on` で指定された runner 環境で実行されます。

ワークフローの利用制限の範囲内であれば、ジョブ数に制限はありません。詳細については、GitHub-hosted runners の請求と使用量、および self-hosted runner の使用制限に関する Actions limits を参照してください。

ワークフロー実行内で実行中のジョブの一意識別子を確認する必要がある場合は、GitHub API を使用できます。詳細については、GitHub Actions の REST API エンドポイントを参照してください。

### `jobs.<job_id>`

`jobs.<job_id>` を使うと、ジョブに一意の識別子を付けられます。キー `job_id` は文字列で、その値はジョブの構成データのマップです。`<job_id>` は、`jobs` オブジェクト内で一意な文字列に置き換える必要があります。`<job_id>` は文字または `_` で始まる必要があり、英数字、`-`、`_` のみを含めることができます。

#### 例: ジョブを作成する

この例では、2 つのジョブが作成されており、その `job_id` の値は `my_first_job` と `my_second_job` です。

```yaml
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```

### `jobs.<job_id>.name`

`jobs.<job_id>.name` を使うと、GitHub UI に表示されるジョブ名を設定できます。

### `jobs.<job_id>.permissions`

特定のジョブでは、`jobs.<job_id>.permissions` を使って `GITHUB_TOKEN` に付与される既定の権限を変更できます。必要に応じてアクセス権を追加または削除し、必要最小限のアクセスのみを許可できます。詳細については、ワークフローで `GITHUB_TOKEN` を認証に使用する方法を参照してください。

ジョブ定義の中で権限を指定することで、必要に応じてジョブごとに異なる `GITHUB_TOKEN` 権限セットを構成できます。あるいは、ワークフロー内のすべてのジョブに対する権限を指定することもできます。ワークフローレベルでの権限定義については、`permissions` を参照してください。

以下の表に示す各権限について、アクセス レベルとして `read`（該当する場合）、`write`、`none` のいずれかを割り当てることができます。`write` には `read` が含まれます。これらの権限のいずれかに対してアクセスを指定した場合、指定しなかった権限はすべて `none` に設定されます。

#### 利用可能な権限と、各権限でアクションに許可される操作

| 権限 | `GITHUB_TOKEN` を使用するアクションで可能になること |
| --- | --- |
| actions | GitHub Actions を操作します。たとえば、`actions: write` はアクションによるワークフロー実行のキャンセルを許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| artifact-metadata | artifact metadata を操作します。たとえば、`artifact-metadata: write` は、ビルド artifact を代表してアクションがストレージ レコードを作成することを許可します。詳細については、artifact metadata の REST API エンドポイントを参照してください。 |
| attestations | artifact attestations を操作します。たとえば、`attestations: write` は、ビルド用の artifact attestation をアクションが生成することを許可します。詳細については、ビルドの来歴を確立するための artifact attestations の使用を参照してください。 |
| checks | check runs と check suites を操作します。たとえば、`checks: write` はアクションによる check run の作成を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| code-quality | code quality を操作します。たとえば、`code-quality: write` はアクションによるコード カバレッジ レポートのアップロードを許可します。詳細については、GitHub Code Quality についてを参照してください。 |
| contents | リポジトリの内容を操作します。たとえば、`contents: read` はアクションによるコミット一覧の取得を許可し、`contents: write` はアクションによるリリースの作成を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| deployments | deployments を操作します。たとえば、`deployments: write` はアクションによる新しい deployment の作成を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| discussions | GitHub Discussions を操作します。たとえば、`discussions: write` はアクションによる discussion のクローズまたは削除を許可します。詳細については、Discussions 向け GraphQL API の使用を参照してください。 |
| id-token | OpenID Connect (OIDC) トークンを取得します。これには `id-token: write` が必要です。詳細については、OpenID Connect を参照してください。 |
| issues | issues を操作します。たとえば、`issues: write` はアクションによる issue へのコメント追加を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| models | GitHub Models で AI 推論レスポンスを生成します。たとえば、`models: read` はアクションによる GitHub Models 推論 API の使用を許可します。AI モデルでのプロトタイピングを参照してください。 |
| packages | GitHub Packages を操作します。たとえば、`packages: write` はアクションによる GitHub Packages へのパッケージのアップロードと公開を許可します。詳細については、GitHub Packages の権限についてを参照してください。 |
| pages | GitHub Pages を操作します。たとえば、`pages: write` はアクションによる GitHub Pages ビルド要求を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| pull-requests | pull requests を操作します。たとえば、`pull-requests: write` はアクションによる pull request へのラベル追加を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| security-events | GitHub code scanning alerts を操作します。たとえば、`security-events: read` はアクションによるリポジトリの code scanning alerts 一覧の取得を許可し、`security-events: write` はアクションによる code scanning alert の状態更新を許可します。詳細については、「Code scanning alerts」のリポジトリ権限を参照してください。 |
| statuses | commit statuses を操作します。たとえば、`statuses:read` はアクションによる特定の参照の commit statuses 一覧取得を許可します。詳細については、GitHub Apps に必要な権限を参照してください。 |
| vulnerability-alerts | Dependabot alerts を読み取ります。たとえば、`vulnerability-alerts: read` はアクションによるリポジトリの Dependabot alerts 一覧取得を許可します。サポートされるのは `read` と `none` のみで、`write` は無効です。`write-all` または `read-all` を使用すると、`vulnerability-alerts` は自動的に `read` として含まれます。詳細については、「Dependabot alerts」のリポジトリ権限を参照してください。 |

Dependabot alerts には `vulnerability-alerts` 権限を使用してください。secret scanning alerts はこの権限では読み取れず、GitHub App または個人用アクセス トークンが必要です。詳細については、「GitHub Apps に必要な権限」内の「Secret scanning alerts」のリポジトリ権限を参照してください。

#### GITHUB_TOKEN スコープのアクセス定義

`permissions` キー内で利用可能な権限の値として `read`、`write`、`none` を指定することで、`GITHUB_TOKEN` が許可するアクセスを定義できます。

```yaml
permissions:
  actions: read|write|none
  artifact-metadata: read|write|none
  attestations: read|write|none
  checks: read|write|none
  code-quality: read|write|none
  contents: read|write|none
  deployments: read|write|none
  id-token: write|none
  issues: read|write|none
  models: read|none
  discussions: read|write|none
  packages: read|write|none
  pages: read|write|none
  pull-requests: read|write|none

  security-events: read|write|none
  statuses: read|write|none
  vulnerability-alerts: read|none
```

これらの権限のいずれかに対してアクセスを指定した場合、指定しなかった権限はすべて `none` に設定されます。

利用可能なすべての権限に対して `read-all` または `write-all` のいずれかのアクセスを定義するには、次の構文を使用できます。

```yaml
permissions: read-all
permissions: write-all
```

利用可能なすべての権限を無効にするには、次の構文を使用できます。

```yaml
permissions: {}
```

#### フォークされたリポジトリでの権限変更

`permissions` キーを使用して、フォークされたリポジトリに対する読み取り権限を追加または削除できますが、通常は書き込みアクセスを付与できません。この動作の例外は、管理者ユーザーが GitHub Actions の設定で **Send write tokens to workflows from pull requests** オプションを選択している場合です。詳細については、リポジトリの GitHub Actions 設定の管理を参照してください。

#### 例: ワークフロー内の 1 つのジョブに GITHUB_TOKEN 権限を設定する

この例では、`stale` という名前のジョブにのみ適用される `GITHUB_TOKEN` の権限を設定しています。`issues` と `pull-requests` の権限には書き込みアクセスが付与されます。それ以外のすべての権限にはアクセス権がありません。

```yaml
jobs:
  stale:
    runs-on: ubuntu-latest

    permissions:
      issues: write
      pull-requests: write

    steps:
      - uses: actions/stale@v10
```

### `jobs.<job_id>.needs`

`jobs.<job_id>.needs` を使うと、このジョブを実行する前に正常に完了していなければならないジョブを指定できます。文字列または文字列の配列を指定できます。ジョブが失敗またはスキップされた場合、それを必要とするすべてのジョブもスキップされます。ただし、そのジョブを継続させる条件式をジョブで使用している場合は例外です。実行に互いを必要とする一連のジョブが含まれている場合、失敗またはスキップは、失敗またはスキップが発生した時点以降の依存チェーン内のすべてのジョブに適用されます。依存しているジョブが成功しなかった場合でもジョブを実行したい場合は、`jobs.<job_id>.if` で `always()` 条件式を使用してください。

#### 例: 依存ジョブの成功を必須にする

```yaml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

この例では、`job1` が正常に完了してから `job2` が開始され、`job3` は `job1` と `job2` の両方が完了するまで待機します。

この例のジョブは順番に実行されます。

```
job1
job2
job3
```

#### 例: 依存ジョブの成功を必須にしない

```yaml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    if: ${{ always() }}
    needs: [job1, job2]
```

この例では、`job3` は `always()` 条件式を使用しているため、`job1` と `job2` が成功したかどうかに関係なく、両方の完了後に常に実行されます。詳細については、ワークフローとアクションでの式の評価を参照してください。

### `jobs.<job_id>.if`

`jobs.<job_id>.if` 条件を使うと、条件が満たされない限りジョブが実行されないようにできます。条件の作成には、サポートされている任意の context と expression を使用できます。このキーでどの contexts がサポートされているかについては、Contexts reference を参照してください。

> **メモ**
>
> `jobs.<job_id>.if` 条件は、`jobs.<job_id>.strategy.matrix` が適用される前に評価されます。

`if` 条件で expressions を使用する場合、GitHub Actions は `if` 条件を自動的に expression として評価するため、` ${{ }} ` の式構文を省略できます。ただし、この例外はあらゆる場所に適用されるわけではありません。

式が `!` で始まる場合、`!` は YAML 形式で予約された記法であるため、常に `${{ }}` 式構文を使用するか、`''`、`""`、`()` でエスケープする必要があります。たとえば次のようになります。

```yaml
if: ${{ ! startsWith(github.ref, 'refs/tags/') }}
```

詳細については、ワークフローとアクションでの式の評価を参照してください。

#### 例: 特定のリポジトリでのみジョブを実行する

この例では、`if` を使って `production-deploy` ジョブを実行できるタイミングを制御しています。このジョブは、リポジトリ名が `octo-org/octo-repo-prod` であり、かつ `octo-org` organization 内にある場合にのみ実行されます。それ以外の場合、このジョブはスキップとしてマークされます。

```yaml
name: example-workflow
on: [push]
jobs:
  production-deploy:
    if: github.repository == 'octo-org/octo-repo-prod'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-node@v4
        with:
          node-version: '14'
      - run: npm install -g bats
```
### jobs.<job_id>.runs-on

`jobs.<job_id>.runs-on` を使って、ジョブを実行するマシンの種類を定義します。

実行先のマシンには、GitHub ホスト型ランナー、larger runner、またはセルフホストランナーを指定できます。

割り当てられたラベル、所属しているグループ、またはその組み合わせに基づいてランナーを対象にできます。

`runs-on` は次の形式で指定できます。

- 単一の文字列。
- 文字列を含む単一の変数。
- 文字列、文字列を含む変数、またはその両方を組み合わせた配列。
- `group` または `labels` キーを使ったキーと値のペア。

文字列または変数の配列を指定した場合、ワークフローは指定したすべての `runs-on` 値に一致する任意のランナー上で実行されます。たとえば、次の例では、ジョブは `linux`、`x64`、`gpu` のラベルを持つセルフホストランナーでのみ実行されます。

```yaml
runs-on: [self-hosted, linux, x64, gpu]
```

詳細については、「セルフホストランナーを選択する」を参照してください。

配列の中では文字列と変数を混在させることができます。たとえば次のようになります。

```yaml
on:
  workflow_dispatch:
    inputs:
      chosen-os:
        required: true
        type: choice
        options:
        - Ubuntu
        - macOS

jobs:
  test:
    runs-on: [self-hosted, "${{ inputs.chosen-os }}"]
    steps:
    - run: echo Hello world!
```

ワークフローを複数のマシンで実行したい場合は、`jobs.<job_id>.strategy` を使います。

> **メモ**
>
> `self-hosted` のような単純な文字列では引用符は不要ですが、`"${{ inputs.chosen-os }}"` のような式では引用符が必要です。

#### GitHub ホスト型ランナーを選択する

GitHub ホスト型ランナーを使用する場合、各ジョブは `runs-on` で指定されたランナーイメージの新しいインスタンス上で実行されます。

GitHub ホスト型ランナーを使用している場合の `runs-on` の値は、ランナーラベルまたはランナーグループ名です。標準の GitHub ホスト型ランナーで使えるラベルは、次の表に示されています。

詳細については、「GitHub ホスト型ランナー」を参照してください。

#### パブリックリポジトリ向けの標準 GitHub ホスト型ランナー

パブリックリポジトリでは、次の表に示すワークフローラベルを使用するジョブは、対応する仕様で実行されます。単一 CPU のランナーを除き、各 GitHub ホスト型ランナーは GitHub がホストする新しい仮想マシン (VM) です。単一 CPU のランナーは共有 VM 上のコンテナーでホストされます。詳しくは「GitHub ホスト型ランナーのリファレンス」を参照してください。標準の GitHub ホスト型ランナーは、パブリックリポジトリでは無料で無制限に使用できます。

| 仮想マシン / コンテナー | プロセッサー (CPU) | メモリ (RAM) | ストレージ (SSD) | アーキテクチャ | ワークフローラベル |
| --- | --- | --- | --- | --- | --- |
| Linux | 1 | 5 GB | 14 GB | x64 | ubuntu-slim |
| Linux | 4 | 16 GB | 14 GB | x64 | ubuntu-latest, ubuntu-24.04, ubuntu-22.04 |
| Windows | 4 | 16 GB | 14 GB | x64 | windows-latest, windows-2025, windows-2025-vs2026, windows-2022 |
| Linux | 4 | 16 GB | 14 GB | arm64 | ubuntu-24.04-arm, ubuntu-22.04-arm |
| Windows | 4 | 16 GB | 14 GB | arm64 | windows-11-arm |
| macOS | 4 | 14 GB | 14 GB | Intel | macos-15-intel, macos-26-intel |
| macOS | 3 (M1) | 7 GB | 14 GB | arm64 | macos-latest, macos-14, macos-15, macos-26 |

#### プライベートリポジトリ向けの標準 GitHub ホスト型ランナー

プライベートリポジトリでは、次の表に示すワークフローラベルを使用するジョブは、対応する仕様の仮想マシン上で実行されます。これらのランナーでは、GitHub アカウントに割り当てられた無料分の分数が消費され、その後は 1 分あたりの料金で課金されます。詳しくは「Actions runner の料金」を参照してください。

| 仮想マシン | プロセッサー (CPU) | メモリ (RAM) | ストレージ (SSD) | アーキテクチャ | ワークフローラベル |
| --- | --- | --- | --- | --- | --- |
| Linux | 1 | 5 GB | 14 GB | x64 | ubuntu-slim |
| Linux | 2 | 8 GB | 14 GB | x64 | ubuntu-latest, ubuntu-24.04, ubuntu-22.04 |
| Windows | 2 | 8 GB | 14 GB | x64 | windows-latest, windows-2025, windows-2022 |
| Linux | 2 | 8 GB | 14 GB | arm64 | ubuntu-24.04-arm, ubuntu-22.04-arm |
| Windows | 2 | 8 GB | 14 GB | arm64 | windows-11-arm |
| macOS | 4 | 14 GB | 14 GB | Intel | macos-15-intel, macos-26-intel |
| macOS | 3 (M1) | 7 GB | 14 GB | arm64 | macos-latest, macos-14, macos-15, macos-26 |

標準の GitHub ホスト型ランナーに加えて、GitHub は GitHub Team および GitHub Enterprise Cloud プランの顧客向けに、高度な機能を備えたさまざまなマネージド仮想マシンを提供しています。たとえば、より多くのコアとディスク容量、GPU 搭載マシン、ARM 搭載マシンなどです。詳細については、「Larger runners」を参照してください。

> **メモ**
>
> `-latest` ランナーイメージは GitHub が提供する最新の安定版イメージであり、オペレーティングシステムのベンダーから提供される最新バージョンとは限りません。

> **警告**
>
> Beta および Deprecated Images は「現状のまま」、「あらゆる不具合を含んだ状態」で、「利用可能な範囲で」提供され、サービス レベル アグリーメントおよび保証の対象外です。Beta Images はカスタマーサポートの対象外となる場合があります。

#### 例: オペレーティングシステムを指定する

```yaml
runs-on: ubuntu-latest
```

詳細については、「GitHub ホスト型ランナー」を参照してください。

#### セルフホストランナーを選択する

ジョブにセルフホストランナーを指定するには、ワークフローファイルの `runs-on` をセルフホストランナーのラベルで設定します。

セルフホストランナーには `self-hosted` ラベルを付けることができます。セルフホストランナーをセットアップすると、既定では `self-hosted` ラベルが含まれます。`self-hosted` ラベルが適用されないようにするには、`--no-default-labels` フラグを渡します。ラベルは、オペレーティングシステムやアーキテクチャなど、ランナーのターゲティングオプションを作成するために使用できます。先頭を `self-hosted` にし（これは必ず最初に記載する必要があります）、その後に必要に応じて追加のラベルを含める配列を指定することをおすすめします。ラベルの配列を指定すると、ジョブは指定したすべてのラベルを持つランナーでキューに入れられます。

> **メモ**
>
> Actions Runner Controller は `self-hosted` ラベルをサポートしていません。

#### 例: ラベルを使ってランナーを選択する

```yaml
runs-on: [self-hosted, linux]
```

詳細については、「セルフホストランナー」および「ワークフローでセルフホストランナーを使う」を参照してください。

#### グループ内のランナーを選択する

`runs-on` を使ってランナーグループを対象にできるため、ジョブはそのグループのメンバーである任意のランナー上で実行されます。より細かく制御したい場合は、ランナーグループとラベルを組み合わせることもできます。

ランナーグループのメンバーにできるのは、larger runner またはセルフホストランナーのみです。

#### 例: グループを使ってジョブの実行場所を制御する

この例では、Ubuntu ランナーが `ubuntu-runners` というグループに追加されています。`runs-on` キーは、ジョブを `ubuntu-runners` グループ内の利用可能な任意のランナーに送ります。

```yaml
name: learn-github-actions
on: [push]
jobs:
  check-bats-version:
    runs-on: 
      group: ubuntu-runners
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-node@v4
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

#### 例: グループとラベルを組み合わせる

グループとラベルを組み合わせる場合、ランナーはジョブを実行する対象となるために両方の要件を満たしている必要があります。

この例では、`ubuntu-runners` というランナーグループに Ubuntu ランナーが登録されており、それらには `ubuntu-24.04-16core` ラベルも割り当てられています。`runs-on` キーは `group` と `labels` を組み合わせることで、そのグループ内にあり、かつ一致するラベルも持つ利用可能な任意のランナーにジョブをルーティングします。

```yaml
name: learn-github-actions
on: [push]
jobs:
  check-bats-version:
    runs-on:
      group: ubuntu-runners
      labels: ubuntu-24.04-16core
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-node@v4
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

### jobs.<job_id>.snapshot

`jobs.<job_id>.snapshot` を使ってカスタムイメージを生成できます。

「カスタムイメージを生成する」で示されているように、文字列構文またはマッピング構文のいずれかを使って、ジョブに `snapshot` キーワードを追加します。

`snapshot` キーワードを含む各ジョブは、それぞれ別個のイメージを作成します。イメージまたはイメージバージョンを 1 つだけ生成したい場合は、ワークフローのすべてのステップを 1 つのジョブに含めてください。`snapshot` キーワードを含むジョブが成功して実行されるたびに、そのイメージの新しいバージョンが作成されます。

### jobs.<job_id>.environment

`jobs.<job_id>.environment` を使って、ジョブが参照する environment を定義します。

environment は環境名だけで指定することも、`name` と `url` を持つ environment オブジェクトとして指定することもできます。この URL は Deployments API の `environment_url` に対応付けられます。Deployments API の詳細については、「リポジトリの REST API エンドポイント」を参照してください。

> **メモ**
>
> environment を参照するジョブがランナーに送信される前に、すべてのデプロイメント保護ルールに合格している必要があります。詳細については、「デプロイ用の environment を管理する」を参照してください。

#### 例: 単一の環境名を使う

```yaml
environment: staging_environment
```

#### 例: 環境名と URL を使う

```yaml
environment:
  name: production_environment
  url: https://github.com
```

`url` の値には式を使用できます。使用できる式コンテキストは `github`、`inputs`、`vars`、`needs`、`strategy`、`matrix`、`job`、`runner`、`env`、`steps` です。式の詳細については、「ワークフローとアクションで式を評価する」を参照してください。

#### 例: URL として output を使う

```yaml
environment:
  name: production_environment
  url: ${{ steps.step_id.outputs.url_output }}
```

`name` の値には式を使用できます。使用できる式コンテキストは `github`、`inputs`、`vars`、`needs`、`strategy`、`matrix` です。式の詳細については、「ワークフローとアクションで式を評価する」を参照してください。

#### 例: 環境名として式を使う

```yaml
environment:
  name: ${{ github.ref_name }}
```

#### 例: デプロイメントを作成せずに environment を使う

デプロイメントオブジェクトを作成せずに environment のシークレットと変数を使うには、`deployment` を `false` に設定します。

```yaml
environment:
  name: testing
  deployment: false
```

`deployment: false` の設定は、カスタムデプロイメント保護ルールとは互換性がありません。詳細については、「GitHub Actions でデプロイする」を参照してください。

### jobs.<job_id>.concurrency

`jobs.<job_id>.concurrency` を使うと、同じ concurrency グループを使うジョブまたはワークフローが、一度に 1 つだけ実行されるようにできます。concurrency グループには任意の文字列または式を使用できます。使用できる式コンテキストは `github`、`inputs`、`vars`、`needs`、`strategy`、`matrix` です。式の詳細については、「ワークフローとアクションで式を評価する」を参照してください。

ワークフローレベルでも concurrency を指定できます。詳細については、`concurrency` を参照してください。

これは、任意の時点で 1 つの concurrency グループに存在できる実行中のジョブまたはワークフローが最大 1 つであることを意味します。同時実行のジョブまたはワークフローがキューに入れられたとき、リポジトリ内で同じ concurrency グループを使う別のジョブまたはワークフローが進行中であれば、そのキューに入れられたジョブまたはワークフローは保留状態になります。既定では、同じ concurrency グループ内に既に存在する保留中のジョブまたはワークフローはキャンセルされ、新しくキューに入れられたジョブまたはワークフローがその代わりになります。

同じ concurrency グループ内で現在実行中のジョブまたはワークフローもキャンセルしたい場合は、`cancel-in-progress: true` を指定します。同じ concurrency グループ内の現在実行中のジョブまたはワークフローを条件付きでキャンセルしたい場合は、許可されている任意の式コンテキストを使う式として `cancel-in-progress` を指定できます。

同じ concurrency グループ内で複数の保留中ジョブまたはワークフロー実行を待機させたい場合は、省略可能な `queue` プロパティを使います。`queue` プロパティには次の値を指定できます。

- `single` (default): concurrency グループ内で保留にできるジョブまたはワークフロー実行は最大 1 つです。新しいジョブまたはワークフロー実行がキューに入ると、同じグループ内で既に保留中のジョブまたはワークフロー実行はキャンセルされて置き換えられます。
- `max`: concurrency グループ内で最大 100 件のジョブまたはワークフロー実行を保留にできます。キューがいっぱいになると、それ以上のジョブまたはワークフロー実行はキャンセルされます。

`queue: max` と `cancel-in-progress: true` の組み合わせは許可されておらず、ワークフローの検証エラーになります。

> **メモ**
>
> concurrency グループ名では大文字と小文字は区別されません。たとえば、`prod` と `Prod` は同じ concurrency グループとして扱われます。
>
> 同じ concurrency グループ内のジョブまたはワークフロー実行は、各ワークフローがディスパッチされた時刻ではなく、それぞれが concurrency グループでの待機を開始した時刻に基づく先入れ先出し (FIFO) の順序で処理されます。ジョブまたは実行の実際の開始時刻は変動する可能性があるため、順序は保証されません。

#### 例: concurrency と既定の動作を使う

GitHub Actions の既定の動作では、複数のジョブまたはワークフロー実行を同時に実行できます。`concurrency` キーワードを使うと、ワークフロー実行の同時実行を制御できます。

たとえば、特定のブランチに対するワークフロー実行全体の同時実行を制限するには、トリガー条件を定義した直後に `concurrency` キーワードを使用できます。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

`concurrency` キーワードをジョブレベルで使うことで、ワークフロー内のジョブの同時実行を制限することもできます。

```yaml
on:
  push:
    branches:
      - main

jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: example-group
      cancel-in-progress: true
```

#### 例: concurrency グループ

concurrency グループは、同じ concurrency キーを共有するワークフロー実行またはジョブの実行を管理し、制限する方法を提供します。

concurrency キーは、ワークフローまたはジョブを 1 つの concurrency グループにまとめるために使われます。concurrency キーを定義すると、GitHub Actions はそのキーを持つワークフローまたはジョブが任意の時点で 1 つだけ実行されるようにします。同じ concurrency キーで新しいワークフロー実行またはジョブが開始されると、GitHub Actions はそのキーを持って既に実行中のワークフローまたはジョブをキャンセルします。concurrency キーには固定文字列を使うことも、コンテキスト変数を含む動的な式を使うこともできます。

ワークフロー内で concurrency 条件を定義して、ワークフローまたはジョブを concurrency グループの一部にすることができます。

これは、ワークフロー実行またはジョブが開始されると、GitHub が同じ concurrency グループですでに進行中のワークフロー実行またはジョブをキャンセルすることを意味します。これは、ステージング環境へのデプロイに使われるワークフローやジョブのように、競合を引き起こしたり必要以上のリソースを消費したりする可能性のあるアクションを防ぐために、特定のワークフローまたはジョブ群の並列実行を防ぎたい場面で役立ちます。

この例では、`job-1` は `staging_environment` という名前の concurrency グループに属しています。つまり、`job-1` の新しい実行がトリガーされると、`staging_environment` concurrency グループ内ですでに進行中の同じジョブの実行はすべてキャンセルされます。

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: staging_environment
      cancel-in-progress: true
```

また、ワークフロー内で `concurrency: ci-${{ github.ref }}` のような動的式を使用すると、そのワークフローまたはジョブは、ワークフローをトリガーしたブランチまたはタグの参照の後ろに `ci-` が付いた名前の concurrency グループに属することになります。この例では、前回の実行がまだ進行中の間に `main` ブランチへ新しいコミットがプッシュされると、前回の実行がキャンセルされて新しい実行が開始されます。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

#### 例: 複数の保留中の実行をキューに入れる

既定では、concurrency グループ内で一度に保留にできるジョブまたはワークフロー実行は 1 つだけです。キャンセルせずに複数の実行をキューに入れられるようにするには、`queue: max` を設定します。`queue: max` を使うと、最大 100 件のジョブまたはワークフロー実行が concurrency グループで待機できます。キューがいっぱいになると、それ以上の実行はキャンセルされます。

たとえば、次のワークフローは本番環境へのデプロイをキューに入れ、各実行が concurrency グループで待機を開始した順に、1 件ずつ処理します。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: production-deploy
  queue: max
```

`queue: max` は `cancel-in-progress: true` と組み合わせられないことに注意してください。これは、進行中の実行の扱いについて 2 つのオプションが互いに矛盾する動作を表しているためです。

#### 例: concurrency を使って進行中のジョブまたは実行をキャンセルする

GitHub Actions で concurrency を使って進行中のジョブまたは実行をキャンセルするには、`concurrency` キーを使い、`cancel-in-progress` オプションを `true` に設定します。

```yaml
concurrency:
  group: ${{ github.ref }}
  cancel-in-progress: true
```

この例では、特定の concurrency グループを定義していないため、GitHub Actions はそのジョブまたはワークフローの進行中の実行をすべてキャンセルすることに注意してください。

#### 例: フォールバック値を使う

特定のイベントでのみ定義されるプロパティを使ってグループ名を組み立てる場合は、フォールバック値を使うことができます。たとえば、`github.head_ref` は `pull_request` イベントでのみ定義されます。ワークフローが `pull_request` イベント以外にも応答する場合は、構文エラーを避けるためにフォールバックを指定する必要があります。次の concurrency グループは、`pull_request` イベントでのみ進行中のジョブまたは実行をキャンセルします。`github.head_ref` が未定義の場合、concurrency グループは実行 ID にフォールバックします。実行 ID は、その実行に対して一意であり、かつ必ず定義されています。

```yaml
concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
```

#### 例: 現在のワークフローの進行中のジョブまたは実行だけをキャンセルする

同じリポジトリ内に複数のワークフローがある場合、他のワークフローの進行中ジョブまたは実行をキャンセルしないようにするには、ワークフロー間で concurrency グループ名を一意にする必要があります。そうしないと、ワークフローに関係なく、それ以前に進行中または保留中だったジョブはすべてキャンセルされます。

同じワークフローの進行中の実行だけをキャンセルするには、`github.workflow` プロパティを使って concurrency グループを組み立てます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

#### 例: 特定のブランチでのみ進行中のジョブをキャンセルする

あるブランチでは進行中のジョブをキャンセルし、別のブランチではキャンセルしたくない場合は、`cancel-in-progress` に条件式を使うことができます。たとえば、リリースブランチではなく開発ブランチで進行中のジョブをキャンセルしたい場合に利用できます。

リリースブランチ上で実行している場合を除き、同じワークフローの進行中の実行だけをキャンセルするには、`cancel-in-progress` を次のような式に設定できます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !contains(github.ref, 'release/')}}
```

この例では、`release/1.2.3` ブランチへの複数回のプッシュでは進行中の実行はキャンセルされません。`main` など別のブランチへのプッシュでは、進行中の実行がキャンセルされます。

### jobs.<job_id>.outputs

`jobs.<job_id>.outputs` を使うと、ジョブの outputs のマップを作成できます。ジョブ outputs は、このジョブに依存するすべての下流ジョブで利用できます。ジョブ依存関係の定義について詳しくは、`jobs.<job_id>.needs` を参照してください。

outputs のサイズは、ジョブごとに最大 1 MB です。ワークフロー実行全体でのすべての outputs の合計は最大 50 MB です。サイズは UTF-16 エンコーディングに基づいて概算されます。

式を含むジョブ outputs は、各ジョブの最後にランナー上で評価されます。シークレットを含む outputs はランナー上で秘匿化され、GitHub Actions には送信されません。

シークレットが含まれている可能性があるために output がスキップされた場合は、次の警告メッセージが表示されます: `"Skip output {output.Key} since it may contain secret."`。シークレットの扱いについては、「例: ジョブまたはワークフロー間でシークレットをマスクして渡す」を参照してください。

依存先のジョブでジョブ outputs を使うには、`needs` コンテキストを使用できます。詳細については、「Contexts リファレンス」を参照してください。

#### 例: ジョブの outputs を定義する

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    # Map a step output to a job output
    outputs:
      output1: ${{ steps.step1.outputs.test }}
      output2: ${{ steps.step2.outputs.test }}
    steps:
      - id: step1
        run: echo "test=hello" >> "$GITHUB_OUTPUT"
      - id: step2
        run: echo "test=world" >> "$GITHUB_OUTPUT"
  job2:
    runs-on: ubuntu-latest
    needs: job1
    steps:
      - env:
          OUTPUT1: ${{needs.job1.outputs.output1}}
          OUTPUT2: ${{needs.job1.outputs.output2}}
        run: echo "$OUTPUT1 $OUTPUT2"
```

#### マトリックスジョブでジョブ outputs を使う

マトリックスを使うと、異なる名前の複数の output を生成できます。マトリックスを使う場合、ジョブ outputs はマトリックス内のすべてのジョブから結合されます。

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    outputs:
      output_1: ${{ steps.gen_output.outputs.output_1 }}
      output_2: ${{ steps.gen_output.outputs.output_2 }}
      output_3: ${{ steps.gen_output.outputs.output_3 }}
    strategy:
      matrix:
        version: [1, 2, 3]
    steps:
      - name: Generate output
        id: gen_output
        run: |
          version="${{ matrix.version }}"
          echo "output_${version}=${version}" >> "$GITHUB_OUTPUT"
  job2:
    runs-on: ubuntu-latest
    needs: [job1]
    steps:
      # Will show
      # {
      #   "output_1": "1",
      #   "output_2": "2",
      #   "output_3": "3"
      # }
      - run: echo '${{ toJSON(needs.job1.outputs) }}'
```

> **警告**
>
> Actions は、マトリックスジョブが実行される順序を保証しません。output 名が一意であることを確認してください。そうしないと、最後に実行されたマトリックスジョブが output の値を上書きします。

### jobs.<job_id>.env

ジョブ内のすべてのステップで利用できる変数のマップです。ワークフロー全体または個々のステップに対して変数を設定できます。詳細については、`env` および `jobs.<job_id>.steps[*].env` を参照してください。

同じ名前の環境変数が複数定義されている場合、GitHub は最も具体的な変数を使用します。たとえば、ステップで定義された環境変数は、そのステップの実行中、同じ名前を持つジョブおよびワークフローの環境変数を上書きします。ジョブに対して定義された環境変数は、そのジョブの実行中、同じ名前を持つワークフロー変数を上書きします。

#### 例: jobs.<job_id>.env

```yaml
jobs:
  job1:
    env:
      FIRST_NAME: Mona
```

### jobs.<job_id>.defaults

`jobs.<job_id>.defaults` を使うと、ジョブ内のすべてのステップに適用されるデフォルト設定のマップを作成できます。ワークフロー全体に対するデフォルト設定を行うこともできます。詳細については、`defaults` を参照してください。

同じ名前のデフォルト設定が複数定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで同じ名前で定義されたデフォルト設定を上書きします。

### jobs.<job_id>.defaults.run

`jobs.<job_id>.defaults.run` を使うと、ジョブ内のすべての `run` ステップに対するデフォルトの `shell` と `working-directory` を指定できます。

ジョブ内のすべての `run` ステップに対して、デフォルトの `shell` オプションと `working-directory` オプションを指定できます。ワークフロー全体に対する `run` のデフォルト設定を行うこともできます。詳細については、`defaults.run` を参照してください。

これらは、`jobs.<job_id>.defaults.run` レベルおよび `jobs.<job_id>.steps[*].run` レベルで上書きできます。

同じ名前のデフォルト設定が複数定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで同じ名前で定義されたデフォルト設定を上書きします。

### jobs.<job_id>.defaults.run.shell

`shell` を使って、ステップで使用するシェルを定義します。このキーワードは複数のコンテキストを参照できます。詳細については、「Contexts」を参照してください。

| サポートされるプラットフォーム | `shell` パラメーター | 説明 | 内部で実行されるコマンド |
| --- | --- | --- | --- |
| Linux / macOS | unspecified | Windows 以外のプラットフォームでの既定のシェルです。`bash` を明示的に指定した場合とは異なるコマンドが実行されることに注意してください。`bash` がパス内で見つからない場合は、`sh` として扱われます。 | bash -e {0} |
| All | bash | Windows 以外のプラットフォームでの既定のシェルで、`sh` へのフォールバックがあります。Windows で `bash` シェルを指定した場合は、Git for Windows に含まれる `bash` シェルが使用されます。 | bash --noprofile --norc -eo pipefail {0} |
| All | pwsh | PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | pwsh -command ". '{0}'" |
| All | python | `python` コマンドを実行します。 | python {0} |
| Linux / macOS | sh | シェルが指定されておらず、かつ `bash` がパス内で見つからない場合の、Windows 以外のプラットフォームでのフォールバック動作です。 | sh -e {0} |
| Windows | cmd | GitHub はスクリプト名に拡張子 `.cmd` を追加し、`{0}` を置き換えます。 | %ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}"". |
| Windows | pwsh | これは Windows で使用される既定のシェルです。PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。セルフホストの Windows ランナーに PowerShell Core がインストールされていない場合は、代わりに PowerShell Desktop が使用されます。 | pwsh -command ". '{0}'". |
| Windows | powershell | PowerShell Desktop です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | powershell -command ". '{0}'". |

同じ名前のデフォルト設定が複数定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで同じ名前で定義されたデフォルト設定を上書きします。

### jobs.<job_id>.defaults.run.working-directory

`working-directory` を使って、ステップのシェルに対する作業ディレクトリを定義します。このキーワードは複数のコンテキストを参照できます。詳細については、「Contexts」を参照してください。

> **ヒント**
>
> 割り当てた `working-directory` が、その中でシェルを実行する前にランナー上に存在していることを確認してください。同じ名前のデフォルト設定が複数定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで同じ名前で定義されたデフォルト設定を上書きします。

#### 例: ジョブのデフォルト `run` ステップオプションを設定する

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: ./scripts
```
### jobs.<job_id>.steps

ジョブには、steps と呼ばれる一連のタスクが含まれます。ステップでは、コマンドの実行、セットアップ タスクの実行、または自分のリポジトリ、パブリック リポジトリ、Docker レジストリで公開されているアクションの実行を行えます。すべてのステップがアクションを実行するわけではありませんが、すべてのアクションはステップとして実行されます。各ステップは、ランナー環境内の独自のプロセスで実行され、ワークスペースとファイルシステムにアクセスできます。ステップは独自のプロセスで実行されるため、環境変数への変更はステップ間で保持されません。GitHub には、ジョブのセットアップと完了を行うための組み込みステップが用意されています。

GitHub に表示されるのは最初の 1,000 件のチェックのみですが、ワークフローの使用制限内である限り、ステップは無制限に実行できます。詳しくは、「GitHub-hosted runners の課金と使用状況」および「self-hosted runner の使用制限に関する Actions の制限」を参照してください。

#### 例: jobs.<job_id>.steps

```yaml
name: Greeting from Mona

on: push

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
      - name: Print a greeting
        env:
          MY_VAR: Hi there! My name is
          FIRST_NAME: Mona
          MIDDLE_NAME: The
          LAST_NAME: Octocat
        run: |
          echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```

### jobs.<job_id>.steps[*].id

ステップの一意の識別子です。`id` を使用して、コンテキスト内でそのステップを参照できます。詳しくは、「Contexts reference」を参照してください。

### jobs.<job_id>.steps[*].if

条件が満たされない限りステップが実行されないようにするには、`if` 条件を使用できます。サポートされている任意のコンテキストと式を使用して条件を作成できます。このキーでどのコンテキストがサポートされているかについて詳しくは、「Contexts reference」を参照してください。

`if` 条件で式を使用する場合、GitHub Actions が `if` 条件を自動的に式として評価するため、必要に応じて `${{ }}` 式構文を省略できます。ただし、この例外はあらゆる場所に適用されるわけではありません。

式が `!` で始まる場合は、`!` が YAML 形式で予約された記法であるため、常に `${{ }}` 式構文を使用するか、`''`、`""`、または `()` でエスケープする必要があります。例:

```yaml
if: ${{ ! startsWith(github.ref, 'refs/tags/') }}
```

詳しくは、「ワークフローとアクションで式を評価する」を参照してください。

#### 例: コンテキストの使用

このステップは、イベントの種類が `pull_request` で、イベントのアクションが `unassigned` の場合にのみ実行されます。

```yaml
steps:
  - name: My first step
    if: ${{ github.event_name == 'pull_request' && github.event.action == 'unassigned' }}
    run: echo This event is a pull request that had an assignee removed.
```

#### 例: ステータス チェック関数の使用

`my backup` ステップは、ジョブ内の前のステップが失敗した場合にのみ実行されます。詳しくは、「ワークフローとアクションで式を評価する」を参照してください。

```yaml
steps:
  - name: My first step
    uses: octo-org/action-name@main
  - name: My backup step
    if: ${{ failure() }}
    uses: actions/heroku@1.0.0
```

#### 例: シークレットの使用

シークレットは `if:` 条件内で直接参照できません。代わりに、シークレットをジョブ レベルの環境変数として設定し、その環境変数を参照してジョブ内のステップを条件付きで実行することを検討してください。

シークレットが設定されていない場合、シークレットを参照する式の戻り値は、（例の `${{ secrets.SuperSecret }}` のように）空文字列になります。

```yaml
name: Run a step if a secret has been set
on: push
jobs:
  my-jobname:
    runs-on: ubuntu-latest
    env:
      super_secret: ${{ secrets.SuperSecret }}
    steps:
      - if: ${{ env.super_secret != '' }}
        run: echo 'This step will only run if the secret has a value set.'
      - if: ${{ env.super_secret == '' }}
        run: echo 'This step will only run if the secret does not have a value set.'
```

詳しくは、「Contexts reference」および「GitHub Actions でシークレットを使用する」を参照してください。

### jobs.<job_id>.steps[*].name

GitHub 上で表示するステップの名前です。

### jobs.<job_id>.steps[*].uses

ジョブ内のステップの一部として実行するアクションを選択します。アクションは、再利用可能なコードの単位です。ワークフローと同じリポジトリ内で定義されたアクション、パブリック リポジトリ内のアクション、または公開済みの Docker コンテナー イメージ内のアクションを使用できます。

使用するアクションのバージョンは、Git ref、SHA、または Docker タグを指定して必ず含めることを強くお勧めします。バージョンを指定しないと、アクションの所有者が更新を公開したときに、ワークフローが壊れたり、予期しない動作が発生したりする可能性があります。

安定性とセキュリティの面では、リリース済みアクション バージョンのコミット SHA を使用するのが最も安全です。
メジャー バージョン タグを公開しているアクションでは、互換性を維持しながら、重大な修正やセキュリティ パッチを受け取ることが期待されます。この動作はアクションの作成者の裁量による点に注意してください。
アクションの既定のブランチを使用すると便利な場合がありますが、互換性のない変更を含む新しいメジャー バージョンがリリースされると、ワークフローが壊れる可能性があります。
一部のアクションでは、`with` キーワードを使用して設定しなければならない入力が必要です。必要な入力を確認するには、アクションの README ファイルを確認してください。

アクションは JavaScript ファイルまたは Docker コンテナーです。使用しているアクションが Docker コンテナーである場合は、Linux 環境でジョブを実行する必要があります。詳しくは、`runs-on` を参照してください。

#### 例: バージョンを指定したアクションの使用

```yaml
steps:
  # Reference a specific commit
  - uses: actions/checkout@8f4b7f84864484a7bf31766abe9204da3cbe65b3
  # Reference the major version of a release
  - uses: actions/checkout@v6
  # Reference a specific version
  - uses: actions/checkout@v6.2.0
  # Reference a branch
  - uses: actions/checkout@main
```

#### 例: パブリック アクションの使用

```
{owner}/{repo}@{ref}
```

パブリック GitHub リポジトリでは、ブランチ、ref、または SHA を指定できます。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        # Uses the default branch of a public repository
        uses: actions/heroku@main
      - name: My second step
        # Uses a specific version tag of a public repository
        uses: actions/aws@v2.0.1
```

#### 例: サブディレクトリ内のパブリック アクションの使用

```
{owner}/{repo}/{path}@{ref}
```

パブリック GitHub リポジトリ内の、特定のブランチ、ref、または SHA にあるサブディレクトリです。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/aws/ec2@main
```

#### 例: ワークフローと同じリポジトリ内のアクションの使用

```
./path/to/dir
```

ワークフローのリポジトリ内でアクションを含むディレクトリへのパスです。アクションを使用する前に、リポジトリをチェックアウトする必要があります。

#### 例: リポジトリのファイル構成

```
|-- hello-world (repository)
|   |__ .github
|       └── workflows
|           └── my-first-workflow.yml
|       └── actions
|           |__ hello-world-action
|               └── action.yml
```

パスは、既定の作業ディレクトリ（`github.workspace`、`$GITHUB_WORKSPACE`）からの相対パス（`./`）です。アクションがワークフローとは異なる場所にリポジトリをチェックアウトする場合、ローカル アクションに使用する相対パスを更新する必要があります。

#### 例: ワークフロー ファイル

```yaml
jobs:
  my_first_job:
    runs-on: ubuntu-latest
    steps:
      # This step checks out a copy of your repository.
      - name: My first step - check out repository
        uses: actions/checkout@v6
      # This step references the directory that contains the action.
      - name: Use local hello-world-action
        uses: ./.github/actions/hello-world-action
```

#### 例: Docker Hub アクションの使用

```
docker://{image}:{tag}
```

Docker Hub で公開されている Docker イメージです。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://alpine:3.8
```

#### 例: GitHub Packages Container registry の使用

```
docker://{host}/{image}:{tag}
```

GitHub Packages Container registry 内のパブリック Docker イメージです。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://ghcr.io/OWNER/IMAGE_NAME
```

#### 例: Docker パブリック レジストリ アクションの使用

```
docker://{host}/{image}:{tag}
```

パブリック レジストリ内の Docker イメージです。この例では、`gcr.io` の Google Container Registry を使用しています。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://gcr.io/cloud-builders/gradle
```

#### 例: ワークフローとは別のプライベート リポジトリ内のアクションの使用

アクションが内部リポジトリ内にある場合、またはワークフローのリポジトリからのアクセスを許可するよう構成されたプライベート リポジトリ内にある場合は、そのアクションを直接参照できます。詳しくは、「Managing GitHub Actions settings for a repository」および「Managing GitHub Actions settings for a repository」を参照してください。

アクションがアクセスを許可するよう構成されたリポジトリ内にない場合は、そのリポジトリをチェックアウトし、アクションをローカルで参照する必要があります。個人用アクセス トークンを生成し、そのトークンをシークレットとして追加してください。次の例は、アクションを参照するためのこの方法を示しています。詳しくは、「個人用アクセス トークンの管理」および「GitHub Actions でシークレットを使用する」を参照してください。

例内の `PERSONAL_ACCESS_TOKEN` は、シークレットの名前に置き換えてください。

```yaml
jobs:
  my_first_job:
    steps:
      - name: Check out repository
        uses: actions/checkout@v6
        with:
          repository: octocat/my-private-repo
          ref: v1.0
          token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
          path: ./.github/actions/my-private-repo
      - name: Run my action
        uses: ./.github/actions/my-private-repo/my-action
```

あるいは、個人用アクセス トークンの所有者が離任した場合でもワークフローが実行され続けるようにするため、個人用アクセス トークンの代わりに GitHub App を使用してください。詳しくは、「GitHub Actions ワークフローで GitHub App を使って認証された API リクエストを行う」を参照してください。

### jobs.<job_id>.steps[*].run

21,000 文字を超えないコマンドライン プログラムを、オペレーティング システムのシェルを使用して実行します。名前を指定しない場合、ステップ名の既定値は `run` コマンドで指定したテキストになります。

コマンドは既定で非ログイン シェルを使用して実行されます。別のシェルを選択したり、コマンドの実行に使用するシェルをカスタマイズしたりできます。詳しくは、`jobs.<job_id>.steps[*].shell` を参照してください。

各 `run` キーワードは、ランナー環境内で新しいプロセスとシェルを表します。複数行のコマンドを指定した場合、各行は同じシェルで実行されます。例:

#### 単一行のコマンド

```yaml
- name: Install Dependencies
  run: npm install
```

#### 複数行のコマンド

```yaml
- name: Clean install dependencies and build
  run: |
    npm ci
    npm run build
```

### jobs.<job_id>.steps[*].working-directory

`working-directory` キーワードを使用すると、コマンドを実行する作業ディレクトリを指定できます。

```yaml
- name: Clean temp directory
  run: rm -rf *
  working-directory: ./temp
```

または、ジョブ内のすべての `run` ステップ、またはワークフロー全体のすべての `run` ステップに対して、既定の作業ディレクトリを指定できます。詳しくは、`defaults.run.working-directory` および `jobs.<job_id>.defaults.run.working-directory` を参照してください。

`run` ステップを使用してスクリプトを実行することもできます。詳しくは、「ワークフローへのスクリプトの追加」を参照してください。

### jobs.<job_id>.steps[*].shell

`shell` キーワードを使用すると、ランナーのオペレーティング システムとジョブの既定値における既定のシェル設定を上書きできます。組み込みのシェル キーワードを使用することも、カスタムのシェル オプション セットを定義することもできます。内部で実行されるシェル コマンドは、`run` キーワードで指定されたコマンドを含む一時ファイルを実行します。

| 対応プラットフォーム | shell パラメーター | 説明 | 内部で実行されるコマンド |
| --- | --- | --- | --- |
| Linux / macOS | unspecified | Windows 以外のプラットフォームでの既定のシェルです。`bash` を明示的に指定した場合とは異なるコマンドが実行される点に注意してください。パス内で `bash` が見つからない場合は、`sh` として扱われます。 | `bash -e {0}` |
| All | bash | Windows 以外のプラットフォームでの既定のシェルで、`sh` へのフォールバックがあります。Windows で `bash` シェルを指定すると、Git for Windows に含まれる `bash` シェルが使用されます。 | `bash --noprofile --norc -eo pipefail {0}` |
| All | pwsh | PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | `pwsh -command ". '{0}'"` |
| All | python | `python` コマンドを実行します。 | `python {0}` |
| Linux / macOS | sh | シェルが指定されておらず、かつパス内で `bash` が見つからない場合の、Windows 以外のプラットフォームでのフォールバック動作です。 | `sh -e {0}` |
| Windows | cmd | GitHub はスクリプト名に拡張子 `.cmd` を追加し、`{0}` をそれに置き換えます。 | `%ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}"".` |
| Windows | pwsh | これは Windows で使用される既定のシェルです。PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。self-hosted Windows ランナーに PowerShell Core がインストールされていない場合は、代わりに PowerShell Desktop が使用されます。 | `pwsh -command ". '{0}'".` |
| Windows | powershell | PowerShell Desktop です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | `powershell -command ". '{0}'".` |

または、ジョブ内のすべての `run` ステップ、またはワークフロー全体のすべての `run` ステップに対して、既定のシェルを指定できます。詳しくは、`defaults.run.shell` および `jobs.<job_id>.defaults.run.shell` を参照してください。

#### 例: Bash を使用したコマンドの実行

```yaml
steps:
  - name: Display the path
    shell: bash
    run: echo $PATH
```

#### 例: Windows cmd を使用したコマンドの実行

```yaml
steps:
  - name: Display the path
    shell: cmd
    run: echo %PATH%
```

#### 例: PowerShell Core を使用したコマンドの実行

```yaml
steps:
  - name: Display the path
    shell: pwsh
    run: echo ${env:PATH}
```

#### 例: PowerShell Desktop を使用したコマンドの実行

```yaml
steps:
  - name: Display the path
    shell: powershell
    run: echo ${env:PATH}
```

#### 例: インライン Python スクリプトの実行

```yaml
steps:
  - name: Display the path
    shell: python
    run: |
      import os
      print(os.environ['PATH'])
```

#### カスタム シェル

`shell` の値は、`command [options] {0} [more_options]` を使用するテンプレート文字列に設定できます。GitHub は、この文字列の最初の空白区切りの単語をコマンドとして解釈し、`{0}` に一時スクリプトのファイル名を挿入します。

たとえば、次のようにします。

```yaml
steps:
  - name: Display the environment variables and their values
    shell: perl {0}
    run: |
      print %ENV
```

この例で使用している `perl` などのコマンドは、ランナーにインストールされている必要があります。

GitHub-hosted ランナーに含まれるソフトウェアについては、「GitHub-hosted runners」を参照してください。

#### 終了コードとエラー時のアクション設定

組み込みのシェル キーワードについては、GitHub-hosted ランナーで実行される次の既定値が提供されています。シェル スクリプトを実行するときは、これらのガイドラインを使用してください。

- `bash/sh`:
  
  既定では、`sh` と `bash` の両方で `set -e` を使用した fail-fast 動作が強制されます。`shell: bash` を指定した場合は、パイプラインが 0 以外の終了ステータスを生成したときに早期終了を強制するため、`-o pipefail` も適用されます。
  
  シェル オプションにテンプレート文字列を指定することで、シェル パラメーターを完全に制御できます。たとえば、`bash {0}` のように指定します。
  
  `sh` 系のシェルは、スクリプト内で最後に実行されたコマンドの終了コードで終了します。これはアクションの既定動作でもあります。ランナーは、この終了コードに基づいてステップの状態を fail または succeed として報告します。
- `powershell/pwsh`
  
  可能な場合は fail-fast 動作になります。組み込みシェルの `pwsh` と `powershell` では、スクリプト内容の先頭に `$ErrorActionPreference = 'stop'` が追加されます。
  
  アクションの状態がスクリプトの最後の終了コードを反映するように、PowerShell スクリプトには `if ((Test-Path -LiteralPath variable:\LASTEXITCODE)) { exit $LASTEXITCODE }` が追加されます。
  
  組み込みシェルを使用せず、必要に応じて `pwsh -File {0}` や `powershell -Command "& '{0}'"` のようなカスタム シェル オプションを指定することで、いつでもこの動作を回避できます。
- `cmd`
  
  fail-fast 動作を完全に有効にする方法は、各エラー コードを確認して適切に応答するようスクリプトを記述する以外にはないようです。既定ではその動作を実際には提供できないため、この動作をスクリプトに書き込む必要があります。
  
  `cmd.exe` は最後に実行したプログラムのエラー レベルで終了し、そのエラー コードをランナーに返します。この動作は、前述の `sh` および `pwsh` の既定動作と内部的に一貫しており、また `cmd.exe` の既定動作でもあるため、この動作は維持されます。

### jobs.<job_id>.steps[*].with

アクションで定義された入力パラメーターのマップです。各入力パラメーターはキーと値のペアです。入力パラメーターは環境変数として設定されます。変数には `INPUT_` というプレフィックスが付き、大文字に変換されます。

Docker コンテナー用に定義された入力パラメーターでは、`args` を使用する必要があります。詳しくは、`jobs.<job_id>.steps[*].with.args` を参照してください。

#### 例: jobs.<job_id>.steps[*].with

`hello_world` アクションで定義されている 3 つの入力パラメーター（`first_name`、`middle_name`、`last_name`）を定義します。これらの入力変数には、`hello-world` アクションから `INPUT_FIRST_NAME`、`INPUT_MIDDLE_NAME`、`INPUT_LAST_NAME` の環境変数としてアクセスできます。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/hello_world@main
        with:
          first_name: Mona
          middle_name: The
          last_name: Octocat
```

### jobs.<job_id>.steps[*].with.args

Docker コンテナーへの入力を定義する文字列です。GitHub は、コンテナーの起動時に `args` をコンテナーの `ENTRYPOINT` に渡します。このパラメーターでは文字列の配列はサポートされません。空白を含む単一の引数は、二重引用符 `""` で囲む必要があります。

#### 例: jobs.<job_id>.steps[*].with.args

```yaml
steps:
  - name: Explain why this job ran
    uses: octo-org/action-name@main
    with:
      entrypoint: /bin/echo
      args: The ${{ github.event_name }} event triggered this step.
```

`args` は Dockerfile の `CMD` 命令の代わりに使用されます。Dockerfile で `CMD` を使用する場合は、優先順に次のガイドラインを使用してください。

- アクションの README に必要な引数を記載し、`CMD` 命令からはそれらを省略します。
- `args` を一切指定しなくてもアクションを使用できるような既定値を使用します。
- アクションが `--help` フラグ、またはそれに類するものを公開している場合は、アクションが自己文書化されるよう、それを既定値として使用します。

### jobs.<job_id>.steps[*].with.entrypoint

Dockerfile 内の Docker `ENTRYPOINT` を上書きします。まだ指定されていない場合は設定します。シェル形式と exec 形式を持つ Docker の `ENTRYPOINT` 命令とは異なり、`entrypoint` キーワードは実行する実行可能ファイルを定義する単一の文字列のみを受け入れます。

#### 例: jobs.<job_id>.steps[*].with.entrypoint

```yaml
steps:
  - name: Run a custom command
    uses: octo-org/action-name@main
    with:
      entrypoint: /a/different/executable
```

`entrypoint` キーワードは Docker コンテナー アクションで使用することを意図していますが、入力をまったく定義していない JavaScript アクションでも使用できます。

### jobs.<job_id>.steps[*].env

ランナー環境でステップが使用する変数を設定します。ワークフロー全体またはジョブに対して変数を設定することもできます。詳しくは、`env` および `jobs.<job_id>.env` を参照してください。

同じ名前の環境変数が複数定義されている場合、GitHub は最も具体的な変数を使用します。たとえば、ステップで定義された環境変数は、そのステップの実行中、同じ名前のジョブおよびワークフローの環境変数より優先されます。ジョブに対して定義された環境変数は、そのジョブの実行中、同じ名前のワークフロー変数より優先されます。

パブリック アクションでは、README ファイルで期待される変数を指定している場合があります。パスワードやトークンのようなシークレットまたは機密性の高い値を設定する場合は、`secrets` コンテキストを使用してシークレットを設定する必要があります。詳しくは、「Contexts reference」を参照してください。

#### 例: jobs.<job_id>.steps[*].env

```yaml
steps:
  - name: My first action
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      FIRST_NAME: Mona
      LAST_NAME: Octocat
```

### jobs.<job_id>.steps[*].continue-on-error

ステップが失敗してもジョブが失敗しないようにします。このステップが失敗したときでもジョブを成功として通過させるには、`true` に設定します。

### jobs.<job_id>.steps[*].timeout-minutes

プロセスを強制終了するまでにステップを実行できる最大分数です。GitHub-hosted ランナーと self-hosted ランナーの両方で最大値は 360 です。

小数値はサポートされていません。`timeout-minutes` は正の整数でなければなりません。

### jobs.<job_id>.timeout-minutes

GitHub がジョブを自動的にキャンセルするまでに、そのジョブを実行できる最大分数です。既定値: 360。

タイムアウトがランナーのジョブ実行時間制限を超える場合、代わりに実行時間制限に達した時点でジョブがキャンセルされます。ジョブの実行時間制限について詳しくは、「GitHub-hosted runners の課金と使用状況」および「self-hosted runner の使用制限に関する Actions の制限」を参照してください。

> **メモ**
>
> `GITHUB_TOKEN` は、ジョブが終了したとき、または最大 24 時間後に期限切れになります。self-hosted ランナーでは、ジョブのタイムアウトが 24 時間を超える場合、トークンが制限要因になることがあります。`GITHUB_TOKEN` について詳しくは、「ワークフローで認証に GITHUB_TOKEN を使用する」を参照してください。

### jobs.<job_id>.strategy

`jobs.<job_id>.strategy` を使用すると、ジョブでマトリックス戦略を利用できます。マトリックス戦略を使うと、1 つのジョブ定義内で変数を使用して、それらの変数の組み合わせに基づく複数のジョブ実行を自動的に作成できます。たとえば、マトリックス戦略を使用して、複数の言語バージョンや複数のオペレーティングシステムでコードをテストできます。詳細については、「Running variations of jobs in a workflow」を参照してください。

### jobs.<job_id>.strategy.matrix

`jobs.<job_id>.strategy.matrix` を使用して、異なるジョブ構成のマトリックスを定義します。詳細については、「Running variations of jobs in a workflow」を参照してください。

1 回のワークフロー実行でマトリックスが生成できるジョブ数は最大 256 です。この上限は、GitHub-hosted runner と self-hosted runner の両方に適用されます。

定義した変数は `matrix` コンテキスト内のプロパティになり、ワークフローファイルの他の領域でそのプロパティを参照できます。この例では、`matrix.version` と `matrix.os` を使用して、そのジョブが使用している `version` と `os` の現在の値にアクセスできます。詳細については、「Contexts reference」を参照してください。

既定では、GitHub は runner の可用性に応じて並列実行されるジョブ数を最大化します。マトリックス内の変数の順序によって、ジョブが作成される順序が決まります。最初に定義した変数が、ワークフロー実行で最初に作成されるジョブになります。

#### 単一次元マトリックスの使用

次のワークフローでは、変数 `version` を値 `[10, 12, 14]` で定義しています。このワークフローでは、変数内の各値に対して 1 つずつ、合計 3 つのジョブが実行されます。各ジョブは `matrix.version` コンテキストを通じて `version` の値にアクセスし、その値を `actions/setup-node` アクションの `node-version` として渡します。

```yaml
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}
```

#### 多次元マトリックスの使用

複数の変数を指定して、多次元マトリックスを作成します。ジョブは、変数の取り得るすべての組み合わせごとに実行されます。

たとえば、次のワークフローでは 2 つの変数を指定しています。

- `os` 変数で指定された 2 つのオペレーティングシステム。
- `version` 変数で指定された 3 つの Node.js バージョン。

このワークフローでは、`os` と `version` 変数の各組み合わせに対して 1 つずつ、合計 6 つのジョブが実行されます。各ジョブは `runs-on` の値を現在の `os` の値に設定し、現在の `version` の値を `actions/setup-node` アクションに渡します。

```yaml
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [ubuntu-22.04, ubuntu-24.04]
        version: [10, 12, 14]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}
```

マトリックス内の変数構成は、オブジェクトの配列にすることもできます。たとえば、次のマトリックスは、対応するコンテキストを持つ 4 つのジョブを生成します。

```yaml
matrix:
  os:
    - ubuntu-latest
    - macos-latest
  node:
    - version: 14
    - version: 20
      env: NODE_OPTIONS=--openssl-legacy-provider
```

マトリックス内の各ジョブは、以下に示すように独自の `os` と `node` の組み合わせを持ちます。

```yaml
- matrix.os: ubuntu-latest
  matrix.node.version: 14
- matrix.os: ubuntu-latest
  matrix.node.version: 20
  matrix.node.env: NODE_OPTIONS=--openssl-legacy-provider
- matrix.os: macos-latest
  matrix.node.version: 14
- matrix.os: macos-latest
  matrix.node.version: 20
  matrix.node.env: NODE_OPTIONS=--openssl-legacy-provider
```

### jobs.<job_id>.strategy.matrix.include

`include` リスト内の各オブジェクトについて、そのオブジェクト内の key:value ペアは、いずれの key:value ペアも元のマトリックス値を上書きしない場合に限り、各マトリックス組み合わせに追加されます。オブジェクトをどのマトリックス組み合わせにも追加できない場合は、代わりに新しいマトリックス組み合わせが作成されます。元のマトリックス値は上書きされませんが、追加されたマトリックス値は上書きできる点に注意してください。

#### 例: 構成の拡張

たとえば、次のワークフローでは、`os` と `node` の各組み合わせに対して 1 つずつ、合計 4 つのジョブが実行されます。`os` の値が `windows-latest` で、`node` の値が `16` のジョブが実行されると、そのジョブには `npm` という追加の変数が値 `6` で含まれます。

```yaml
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [windows-latest, ubuntu-latest]
        node: [14, 16]
        include:
          - os: windows-latest
            node: 16
            npm: 6
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - if: ${{ matrix.npm }}
        run: npm install -g npm@${{ matrix.npm }}
      - run: npm --version
```

#### 例: 構成の追加

たとえば、このマトリックスでは、マトリックス内の `os` と `version` の各組み合わせに対して 1 つずつ、合計 10 個のジョブに加えて、`os` の値が `windows-latest`、`version` の値が `17` のジョブが 1 つ実行されます。

```yaml
jobs:
  example_matrix:
    strategy:
      matrix:
        os: [macos-latest, windows-latest, ubuntu-latest]
        version: [12, 14, 16]
        include:
          - os: windows-latest
            version: 17
```

マトリックス変数を何も指定しない場合は、`include` 配下のすべての構成が実行されます。たとえば、次のワークフローでは、各 `include` エントリに対して 1 つずつ、合計 2 つのジョブが実行されます。これにより、完全に展開されたマトリックスがなくても、マトリックス戦略を活用できます。

```yaml
jobs:
  includes_only:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include:
          - site: "production"
            datacenter: "site-a"
          - site: "staging"
            datacenter: "site-b"
```

### jobs.<job_id>.strategy.matrix.exclude

除外される構成は、部分一致であるだけで除外の対象になります。

すべての `include` の組み合わせは `exclude` の後に処理されます。これにより、以前に除外した組み合わせを `include` を使って再び追加できます。

### jobs.<job_id>.strategy.fail-fast

`jobs.<job_id>.strategy.fail-fast` と `jobs.<job_id>.continue-on-error` を使用して、ジョブ失敗の扱いを制御できます。

`jobs.<job_id>.strategy.fail-fast` はマトリックス全体に適用されます。`jobs.<job_id>.strategy.fail-fast` が `true` に設定されている場合、またはその式が `true` と評価される場合、マトリックス内のいずれかのジョブが失敗すると、GitHub は進行中およびキュー待ちのすべてのジョブをキャンセルします。このプロパティの既定値は `true` です。

`jobs.<job_id>.continue-on-error` は単一のジョブに適用されます。`jobs.<job_id>.continue-on-error` が `true` の場合、`jobs.<job_id>.continue-on-error: true` を持つジョブが失敗しても、マトリックス内の他のジョブは実行を継続します。

`jobs.<job_id>.strategy.fail-fast` と `jobs.<job_id>.continue-on-error` は組み合わせて使用できます。たとえば、次のワークフローは 4 つのジョブを開始します。各ジョブについて、`continue-on-error` は `matrix.experimental` の値によって決定されます。`continue-on-error: false` のジョブのいずれかが失敗した場合、進行中またはキュー待ちのすべてのジョブがキャンセルされます。`continue-on-error: true` のジョブが失敗した場合、他のジョブには影響しません。

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    continue-on-error: ${{ matrix.experimental }}
    strategy:
      fail-fast: true
      matrix:
        version: [6, 7, 8]
        experimental: [false]
        include:
          - version: 9
            experimental: true
```

### jobs.<job_id>.strategy.max-parallel

既定では、GitHub は runner の可用性に応じて並列実行されるジョブ数を最大化します。

### jobs.<job_id>.continue-on-error

`jobs.<job_id>.continue-on-error` は単一のジョブに適用されます。`jobs.<job_id>.continue-on-error` が `true` の場合、`jobs.<job_id>.continue-on-error: true` を持つジョブが失敗しても、マトリックス内の他のジョブは実行を継続します。

ジョブが失敗したときにワークフロー実行全体が失敗するのを防ぎます。このジョブが失敗してもワークフロー実行を成功として通過させるには、`true` に設定します。

#### 例: 特定の失敗するマトリックスジョブによってワークフロー実行が失敗しないようにする

ワークフロー実行全体を失敗させずに、ジョブマトリックス内の特定のジョブだけを失敗可能にできます。たとえば、`node` が `15` に設定された実験的なジョブだけを、ワークフロー実行を失敗させずに失敗可能にしたい場合です。

```yaml
runs-on: ${{ matrix.os }}
continue-on-error: ${{ matrix.experimental }}
strategy:
  fail-fast: false
  matrix:
    node: [13, 14]
    os: [macos-latest, ubuntu-latest]
    experimental: [false]
    include:
      - node: 15
        os: ubuntu-latest
        experimental: true
```

### jobs.<job_id>.container

> **メモ**
>
> ワークフローで Docker container actions、job containers、service containers を使用する場合は、Linux runner を使用する必要があります。
>
> - GitHub-hosted runners を使用する場合は、Ubuntu runner を使用する必要があります。
> - Self-hosted runners を使用する場合は、runner として Linux マシンを使用し、Docker がインストールされている必要があります。

`jobs.<job_id>.container` を使用すると、ジョブ内ですでにコンテナーを指定していないすべてのステップを実行するためのコンテナーを作成できます。スクリプトアクションとコンテナーアクションの両方を使用するステップがある場合、コンテナーアクションは同じネットワーク上で同じボリュームマウントを共有する兄弟コンテナーとして実行されます。

コンテナーを設定しない場合、コンテナー内で実行するよう構成されたアクションをステップが参照していない限り、すべてのステップは `runs-on` で指定されたホスト上で直接実行されます。

> **メモ**
>
> コンテナー内の `run` ステップの既定シェルは `bash` ではなく `sh` です。これは `jobs.<job_id>.defaults.run` または `jobs.<job_id>.steps[*].shell` で上書きできます。

#### 例: コンテナー内でのジョブの実行

```yaml
name: CI
on:
  push:
    branches: [ main ]
jobs:
  container-test-job:
    runs-on: ubuntu-latest
    container:
      image: node:18
      env:
        NODE_ENV: development
      ports:
        - 80
      volumes:
        - my_docker_volume:/volume_mount
      options: --cpus 1
    steps:
      - name: Check for dockerenv file
        run: (ls /.dockerenv && echo Found dockerenv) || (echo No dockerenv)
```

コンテナーイメージだけを指定する場合は、`image` キーワードを省略できます。

```yaml
jobs:
  container-test-job:
    runs-on: ubuntu-latest
    container: node:18
```

### jobs.<job_id>.container.image

`jobs.<job_id>.container.image` を使用して、アクションを実行するコンテナーとして使う Docker イメージを定義します。値には、Docker Hub のイメージ名またはレジストリ名を指定できます。

> **メモ**
>
> 通常、Docker Hub は `push` と `pull` の両方の操作にレート制限を課しており、self-hosted runner 上のジョブに影響します。ただし、GitHub と Docker の取り決めにより、GitHub-hosted runner はこれらの制限の対象になりません。

### jobs.<job_id>.container.credentials

イメージのコンテナーレジストリでイメージの `pull` に認証が必要な場合は、`jobs.<job_id>.container.credentials` を使用して、ユーザー名とパスワードのマップを設定できます。これらの資格情報は、`docker login` コマンドに指定する値と同じです。

#### 例: コンテナーレジストリの資格情報の定義

```yaml
container:
  image: ghcr.io/owner/image
  credentials:
     username: ${{ github.actor }}
     password: ${{ secrets.github_token }}
```

### jobs.<job_id>.container.env

`jobs.<job_id>.container.env` を使用して、コンテナー内の環境変数のマップを設定します。

### jobs.<job_id>.container.ports

`jobs.<job_id>.container.ports` を使用して、コンテナーで公開するポートの配列を設定します。

### jobs.<job_id>.container.volumes

`jobs.<job_id>.container.volumes` を使用して、コンテナーで使用するボリュームの配列を設定します。ボリュームを使うと、サービス間やジョブ内の他のステップとの間でデータを共有できます。名前付き Docker ボリューム、匿名 Docker ボリューム、またはホスト上の bind mount を指定できます。

ボリュームを指定するには、ソースと宛先パスを指定します。

`<source>:<destinationPath>`。

`<source>` はボリューム名またはホストマシン上の絶対パスで、`<destinationPath>` はコンテナー内の絶対パスです。

#### 例: コンテナーでのボリュームのマウント

```yaml
volumes:
  - my_docker_volume:/volume_mount
  - /data/my_data
  - /source/directory:/destination/directory
```

### jobs.<job_id>.container.options

`jobs.<job_id>.container.options` を使用して、Docker コンテナーの追加のリソースオプションを構成します。オプションの一覧については、「docker create options」を参照してください。

> **警告**
>
> `--network` オプションと `--entrypoint` オプションはサポートされていません。

### jobs.<job_id>.services

> **メモ**
>
> ワークフローで Docker container actions、job containers、service containers を使用する場合は、Linux runner を使用する必要があります。
>
> - GitHub-hosted runners を使用する場合は、Ubuntu runner を使用する必要があります。
> - Self-hosted runners を使用する場合は、runner として Linux マシンを使用し、Docker がインストールされている必要があります。

ワークフロー内のジョブのためにサービスコンテナーをホストするために使用します。サービスコンテナーは、データベースや Redis のようなキャッシュサービスを作成する場合に役立ちます。runner は Docker ネットワークを自動的に作成し、サービスコンテナーのライフサイクルを管理します。

ジョブをコンテナー内で実行するように構成している場合、またはステップでコンテナーアクションを使用している場合は、サービスまたはアクションにアクセスするためにポートをマッピングする必要はありません。Docker は、同じ Docker ユーザー定義ブリッジネットワーク上のコンテナー間ですべてのポートを自動的に公開します。サービスコンテナーは、そのホスト名を使って直接参照できます。ホスト名は、ワークフローでそのサービスに構成したラベル名に自動的にマッピングされます。

ジョブを runner マシン上で直接実行するように構成し、ステップでコンテナーアクションを使用していない場合は、必要な Docker サービスコンテナーのポートを Docker ホスト（runner マシン）にマッピングする必要があります。`localhost` とマッピングされたポートを使用してサービスコンテナーにアクセスできます。

ネットワークにおけるサービスコンテナーの違いの詳細については、「Communicating with Docker service containers」を参照してください。

#### 例: localhost の使用

この例では、`nginx` と `redis` の 2 つのサービスを作成します。コンテナーポートのみを指定し、ホストポートを指定しない場合、そのコンテナーポートはホスト上の空いているポートにランダムに割り当てられます。GitHub は、割り当てられたホストポートを `${{job.services.<service_name>.ports}}` コンテキストに設定します。この例では、`${{ job.services.nginx.ports['80'] }}` および `${{ job.services.redis.ports['6379'] }}` コンテキストを使用してサービスのホストポートにアクセスできます。

```yaml
services:
  nginx:
    image: nginx
    # Map port 8080 on the Docker host to port 80 on the nginx container
    ports:
      - 8080:80
  redis:
    image: redis
    # Map random free TCP port on Docker host to port 6379 on redis container
    ports:
      - 6379/tcp
steps:
  - run: |
      echo "Redis available on 127.0.0.1:${{ job.services.redis.ports['6379'] }}"
      echo "Nginx available on 127.0.0.1:${{ job.services.nginx.ports['80'] }}"
```

### jobs.<job_id>.services.<service_id>.image

アクションを実行するサービスコンテナーとして使用する Docker イメージです。値には、Docker Hub のイメージ名またはレジストリ名を指定できます。

`jobs.<job_id>.services.<service_id>.image` に空文字列を割り当てると、サービスは起動しません。これを利用して、次の例のように条件付きサービスを設定できます。

```yaml
services:
  nginx:
    image: ${{ options.nginx == true && 'nginx' || '' }}
```

### jobs.<job_id>.services.<service_id>.credentials

イメージのコンテナーレジストリでイメージの `pull` に認証が必要な場合は、`jobs.<job_id>.container.credentials` を使用して、ユーザー名とパスワードのマップを設定できます。これらの資格情報は、`docker login` コマンドに指定する値と同じです。

#### 例: jobs.<job_id>.services.<service_id>.credentials

```yaml
services:
  myservice1:
    image: ghcr.io/owner/myservice1
    credentials:
      username: ${{ github.actor }}
      password: ${{ secrets.github_token }}
  myservice2:
    image: dockerhub_org/myservice2
    credentials:
      username: ${{ secrets.DOCKER_USER }}
      password: ${{ secrets.DOCKER_PASSWORD }}
```

### jobs.<job_id>.services.<service_id>.env

サービスコンテナー内の環境変数のマップを設定します。

### jobs.<job_id>.services.<service_id>.ports

サービスコンテナーで公開するポートの配列を設定します。

### jobs.<job_id>.services.<service_id>.volumes

サービスコンテナーで使用するボリュームの配列を設定します。ボリュームを使うと、サービス間やジョブ内の他のステップとの間でデータを共有できます。名前付き Docker ボリューム、匿名 Docker ボリューム、またはホスト上の bind mount を指定できます。

ボリュームを指定するには、ソースと宛先パスを指定します。

`<source>:<destinationPath>`。

`<source>` はボリューム名またはホストマシン上の絶対パスで、`<destinationPath>` はコンテナー内の絶対パスです。

#### 例: jobs.<job_id>.services.<service_id>.volumes

```yaml
volumes:
  - my_docker_volume:/volume_mount
  - /data/my_data
  - /source/directory:/destination/directory
```

### jobs.<job_id>.services.<service_id>.options

追加の Docker コンテナーリソースオプションです。オプションの一覧については、「docker create options」を参照してください。

> **警告**
>
> `--network` オプションはサポートされていません。

### jobs.<job_id>.services.<service_id>.command

Docker イメージの既定コマンド（`CMD`）を上書きします。値は `docker create` コマンドのイメージ名の後ろに引数として渡されます。`entrypoint` も指定した場合、`command` はその `entrypoint` に対する引数を提供します。

#### 例: jobs.<job_id>.services.<service_id>.command

```yaml
services:
  mysql:
    image: mysql:8
    command: --sql_mode=STRICT_TRANS_TABLES --max_allowed_packet=512M
    env:
      MYSQL_ROOT_PASSWORD: test
    ports:
      - 3306:3306
```

### jobs.<job_id>.services.<service_id>.entrypoint

Docker イメージの既定の `ENTRYPOINT` を上書きします。値は、実行する実行可能ファイルを定義する単一の文字列です。イメージの `entrypoint` を完全に置き換える必要がある場合にこれを使用します。`entrypoint` と `command` を組み合わせて、カスタム `entrypoint` に引数を渡すことができます。

#### 例: jobs.<job_id>.services.<service_id>.entrypoint

```yaml
services:
  etcd:
    image: quay.io/coreos/etcd:v3.5.17
    entrypoint: etcd
    command: >-
      --listen-client-urls http://0.0.0.0:2379
      --advertise-client-urls http://0.0.0.0:2379
    ports:
      - 2379:2379
```

### jobs.<job_id>.uses

実行する再利用可能ワークフローファイルの場所とバージョンです。次のいずれかの構文を使用します。

- `{owner}/{repo}/.github/workflows/{filename}@{ref}` は、public リポジトリおよび private リポジトリ内の再利用可能ワークフローに使用します。
- `./.github/workflows/{filename}` は、同じリポジトリ内の再利用可能ワークフローに使用します。

最初のオプションでは、`{ref}` に SHA、リリースタグ、またはブランチ名を指定できます。リリースタグとブランチが同じ名前の場合は、リリースタグがブランチ名より優先されます。安定性とセキュリティの観点では、コミット SHA を使用するのが最も安全です。詳細については、「Secure use reference」を参照してください。

2 番目の構文オプション（`{owner}/{repo}` と `@{ref}` を含まない形式）を使用する場合、呼び出されるワークフローは呼び出し元ワークフローと同じコミットから取得されます。`refs/heads` や `refs/tags` のような ref プレフィックスは使用できません。このキーワードではコンテキストや式は使用できません。

#### 例: jobs.<job_id>.uses

```yaml
jobs:
  call-workflow-1-in-local-repo:
    uses: octo-org/this-repo/.github/workflows/workflow-1.yml@172239021f7ba04fe7327647b213799853a9eb89
  call-workflow-2-in-local-repo:
    uses: ./.github/workflows/workflow-2.yml
  call-workflow-in-another-repo:
    uses: octo-org/another-repo/.github/workflows/workflow.yml@v1
```

詳細については、「Reuse workflows」を参照してください。

### jobs.<job_id>.with

ジョブが再利用可能ワークフローを呼び出すために使用される場合、`with` を使って呼び出されるワークフローに渡す入力のマップを指定できます。

渡す入力はすべて、呼び出されるワークフローで定義されている入力仕様と一致している必要があります。

`jobs.<job_id>.steps[*].with` とは異なり、`jobs.<job_id>.with` で渡した入力は、呼び出されるワークフロー内で環境変数としては利用できません。代わりに、`inputs` コンテキストを使用して入力を参照できます。

#### 例: jobs.<job_id>.with

```yaml
jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/called-workflow.yml@main
    with:
      username: mona
```

### jobs.<job_id>.with.<input_id>

入力の文字列識別子とその値から成る組です。識別子は、呼び出されるワークフロー内の `on.workflow_call.inputs.<inputs_id>` で定義された入力名と一致している必要があります。値のデータ型は、呼び出されるワークフロー内の `on.workflow_call.inputs.<input_id>.type` で定義された型と一致している必要があります。

使用できる式コンテキストは `github` と `needs` です。

### jobs.<job_id>.secrets

ジョブが再利用可能ワークフローを呼び出すために使用される場合、`secrets` を使って呼び出されるワークフローに渡すシークレットのマップを指定できます。

渡すシークレットはすべて、呼び出されるワークフローで定義されている名前と一致している必要があります。

#### 例: jobs.<job_id>.secrets

```yaml
jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/called-workflow.yml@main
    secrets:
      access-token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
```

### jobs.<job_id>.secrets.inherit

`inherit` キーワードを使用すると、呼び出し元ワークフローのすべてのシークレットを呼び出されるワークフローに渡せます。これには、呼び出し元ワークフローがアクセスできるすべてのシークレット、つまり organization、repository、environment の各シークレットが含まれます。`inherit` キーワードは、同一 organization 内のリポジトリ間、または同一 enterprise 内の organization 間でシークレットを渡すために使用できます。

#### 例: jobs.<job_id>.secrets.inherit

```yaml
on:
  workflow_dispatch:

jobs:
  pass-secrets-to-workflow:
    uses: ./.github/workflows/called-workflow.yml
    secrets: inherit
on:
  workflow_call:

jobs:
  pass-secret-to-action:
    runs-on: ubuntu-latest
    steps:
      - name: Use a repo or org secret from the calling workflow.
        run: echo ${{ secrets.CALLING_WORKFLOW_SECRET }}
```

### jobs.<job_id>.secrets.<secret_id>

シークレットの文字列識別子とその値から成る組です。識別子は、呼び出されるワークフロー内の `on.workflow_call.secrets.<secret_id>` で定義されたシークレット名と一致している必要があります。

使用できる式コンテキストは `github`、`needs`、`secrets` です。

## フィルターパターン チートシート

パス、ブランチ、タグのフィルターでは、特殊文字を使用できます。

- `*`: 0 個以上の文字に一致しますが、`/` 文字には一致しません。たとえば、`Octo*` は `Octocat` に一致します。
- `**`: 任意の文字の 0 個以上に一致します。
- `?`: 直前の文字の 0 個または 1 個に一致します。
- `+`: 直前の文字の 1 個以上に一致します。
- `[]`: 角かっこ内に列挙された、または範囲に含まれる 1 つの英数字に一致します。範囲には `a-z`、`A-Z`、`0-9` のみを含められます。たとえば、`[0-9a-z]` という範囲は任意の数字または小文字に一致します。たとえば、`[CB]at` は `Cat` または `Bat` に一致し、`[1-2]00` は `100` と `200` に一致します。
- `!`: パターンの先頭に置くと、それ以前の肯定パターンを否定します。先頭文字でない場合は特別な意味を持ちません。

`*`、`[`、`!` は YAML では特殊文字です。パターンを `*`、`[`、`!` で始める場合は、パターンを引用符で囲む必要があります。また、`[` や `]` を含むパターンを flow sequence で使用する場合も、パターンを引用符で囲む必要があります。

```yaml
# Valid
paths:
  - '**/README.md'

# Invalid - creates a parse error that
# prevents your workflow from running.
paths:
  - **/README.md

# Valid
branches: [ main, 'release/v[0-9].[0-9]' ]

# Invalid - creates a parse error
branches: [ main, release/v[0-9].[0-9] ]
```

ブランチ、タグ、およびパスのフィルター構文の詳細については、`on.<push>.<branches|tags>`、`on.<pull_request>.<branches|tags>`、および `on.<push|pull_request>.paths` を参照してください。

### ブランチとタグに一致するパターン

| Pattern | 説明 | 一致例 |
| --- | --- | --- |
| `feature/*` | `*` ワイルドカードは任意の文字に一致しますが、スラッシュ (`/`) には一致しません。 | `feature/my-branch`<br>`feature/your-branch` |
| `feature/**` | `**` ワイルドカードは、ブランチ名およびタグ名ではスラッシュ (`/`) を含む任意の文字に一致します。 | `feature/beta-a/my-branch`<br>`feature/your-branch`<br>`feature/mona/the/octocat` |
| `main` | ブランチ名またはタグ名の完全一致に一致します。 | `main` |
| `releases/mona-the-octocat` | ブランチ名またはタグ名の完全一致に一致します。 | `releases/mona-the-octocat` |
| `'*'` | スラッシュ (`/`) を含まないすべてのブランチ名とタグ名に一致します。`*` 文字は YAML では特殊文字です。パターンを `*` で始める場合は、引用符を使用する必要があります。 | `main`<br>`releases` |
| `'**'` | すべてのブランチ名とタグ名に一致します。これは、`branches` または `tags` フィルターを使用しない場合の既定の動作です。 | `all/the/branches`<br>`every/tag` |
| `'*feature'` | `*` 文字は YAML では特殊文字です。パターンを `*` で始める場合は、引用符を使用する必要があります。 | `mona-feature`<br>`feature`<br>`ver-10-feature` |
| `v2*` | `v2` で始まるブランチ名およびタグ名に一致します。 | `v2`<br>`v2.0`<br>`v2.9` |
| `v[12].[0-9]+.[0-9]+` | メジャーバージョンが 1 または 2 のすべてのセマンティックバージョニングのブランチおよびタグに一致します。 | `v1.10.1`<br>`v2.0.0` |

### ファイルパスに一致するパターン

パスパターンはパス全体に一致する必要があり、リポジトリのルートから始まります。

| Pattern | 一致内容の説明 | 一致例 |
| --- | --- | --- |
| `'*'` | `*` ワイルドカードは任意の文字に一致しますが、スラッシュ (`/`) には一致しません。`*` 文字は YAML では特殊文字です。パターンを `*` で始める場合は、引用符を使用する必要があります。 | `README.md`<br>`server.rb` |
| `'*.jsx?'` | `?` 文字は直前の文字の 0 個または 1 個に一致します。 | `page.js`<br>`page.jsx` |
| `'**'` | `**` ワイルドカードは、スラッシュ (`/`) を含む任意の文字に一致します。これは、パスフィルターを使用しない場合の既定の動作です。 | `all/the/files.md` |
| `'*.js'` | `*` ワイルドカードは任意の文字に一致しますが、スラッシュ (`/`) には一致しません。リポジトリのルートにあるすべての `.js` ファイルに一致します。 | `app.js`<br>`index.js` |
| `'**.js'` | リポジトリ内のすべての `.js` ファイルに一致します。 | `index.js`<br>`js/index.js`<br>`src/js/app.js` |
| `docs/*` | リポジトリのルートにある `docs` ディレクトリ直下のファイルのみに一致します。 | `docs/README.md`<br>`docs/file.txt` |
| `docs/**` | リポジトリのルートにある `docs` ディレクトリおよびそのサブディレクトリ内の任意のファイルに一致します。 | `docs/README.md`<br>`docs/mona/octocat.txt` |
| `docs/**/*.md` | `docs` ディレクトリ内の任意の場所にある `.md` 接尾辞を持つファイルに一致します。 | `docs/README.md`<br>`docs/mona/hello-world.md`<br>`docs/a/markdown/file.md` |
| `'**/docs/**'` | リポジトリ内の任意の場所にある `docs` ディレクトリ内の任意のファイルに一致します。 | `docs/hello.md`<br>`dir/docs/my-file.txt`<br>`space/docs/plan/space.doc` |
| `'**/README.md'` | リポジトリ内の任意の場所にある `README.md` ファイルに一致します。 | `README.md`<br>`js/README.md` |
| `'**/*src/**'` | リポジトリ内の任意の場所で、`src` 接尾辞を持つフォルダー内の任意のファイルに一致します。 | `a/src/app.js`<br>`my-src/code/js/app.js` |
| `'**/*-post.md'` | リポジトリ内の任意の場所で、`-post.md` 接尾辞を持つファイルに一致します。 | `my-post.md`<br>`path/their-post.md` |
| `'**/migrate-*.sql'` | リポジトリ内の任意の場所で、`migrate-` 接頭辞と `.sql` 接尾辞を持つファイルに一致します。 | `migrate-10909.sql`<br>`db/migrate-v1.0.sql`<br>`db/sept/migrate-v1.sql` |
| `'*.md'`<br>`'!README.md'` | パターンの前に感嘆符 (`!`) を付けると、そのパターンは否定されます。ファイルがあるパターンに一致し、その後で定義された否定パターンにも一致する場合、そのファイルは含まれません。 | `hello.md`<br>一致しないもの<br>`README.md`<br>`docs/hello.md` |
| `'*.md'`<br>`'!README.md'`<br>`README*` | パターンは順番に評価されます。前のパターンを否定するパターンは、ファイルパスを再度含めます。 | `hello.md`<br>`README.md`<br>`README.doc` |
