# GitHub Actions ワークフロー構文（日本語訳）

## ワークフローの YAML 構文について

ワークフローファイルは YAML 構文を使用し、ファイル拡張子は `.yml` または `.yaml` のいずれかである必要があります。YAML が初めてで詳しく学びたい場合は、「Learn YAML in Y minutes」を参照してください。

ワークフローファイルは、リポジトリの `.github/workflows` ディレクトリに保存する必要があります。

## `name`

ワークフローの名前です。GitHub は、リポジトリの「Actions」タブの下にワークフローの名前を表示します。`name` を省略すると、GitHub はリポジトリのルートを基準としたワークフローファイルのパスを表示します。

## `run-name`

ワークフローから生成されるワークフロー実行の名前です。GitHub は、リポジトリの「Actions」タブにあるワークフロー実行の一覧に、ワークフロー実行名を表示します。`run-name` が省略されている場合、または空白文字だけの場合、実行名はワークフロー実行のイベント固有の情報に設定されます。たとえば、`push` または `pull_request` イベントによってトリガーされたワークフローでは、コミットメッセージまたはプルリクエストのタイトルに設定されます。

この値には式を含めることができ、`github` および `inputs` コンテキストを参照できます。

### `run-name` の例

```yaml
run-name: Deploy to ${{ inputs.deploy_target }} by @${{ github.actor }}
```

## `on`

ワークフローを自動的にトリガーするには、`on` を使用して、どのイベントがワークフローの実行を引き起こせるかを定義します。利用可能なイベントの一覧については、「ワークフローをトリガーするイベント」を参照してください。

ワークフローをトリガーできる単一または複数のイベントを定義したり、時刻スケジュールを設定したりできます。また、特定のファイル、タグ、またはブランチの変更があった場合にのみワークフローを実行するよう制限することもできます。これらのオプションについては、以降のセクションで説明します。

### 単一のイベントを使用する

たとえば、次の `on` 値を持つワークフローは、ワークフローのリポジトリ内の任意のブランチに `push` が行われたときに実行されます。

```yaml
on: push
```

### 複数のイベントを使用する

単一のイベントまたは複数のイベントを指定できます。たとえば、次の `on` 値を持つワークフローは、リポジトリ内の任意のブランチに `push` が行われたとき、または誰かがリポジトリをフォークしたときに実行されます。

```yaml
on: [push, fork]
```

複数のイベントを指定した場合、ワークフローをトリガーするには、そのうち 1 つのイベントだけが発生すれば十分です。ワークフローの複数のトリガーイベントが同時に発生した場合は、複数のワークフロー実行がトリガーされます。

### アクティビティの種類を使用する

一部のイベントには、ワークフローをいつ実行するかをより細かく制御できるアクティビティの種類があります。ワークフロー実行をトリガーするイベントアクティビティの種類を定義するには、`on.<event_name>.types` を使用します。

たとえば、`issue_comment` イベントには、`created`、`edited`、`deleted` のアクティビティの種類があります。ワークフローが `label` イベントでトリガーされる場合、ラベルが作成、編集、または削除されるたびに実行されます。`label` イベントに `created` アクティビティの種類を指定した場合、ワークフローはラベルが作成されたときに実行されますが、ラベルが編集または削除されたときには実行されません。

```yaml
on:
  label:
    types:
      - created
```

複数のアクティビティの種類を指定した場合、ワークフローをトリガーするには、それらのイベントアクティビティの種類のうち 1 つだけが発生すれば十分です。ワークフローの複数のトリガーイベントアクティビティの種類が同時に発生した場合は、複数のワークフロー実行がトリガーされます。たとえば、次のワークフローは、イシューが開かれたとき、またはラベル付けされたときにトリガーされます。2 つのラベルを持つイシューが開かれた場合、3 つのワークフロー実行が開始されます。イシューが開かれたイベントに対して 1 つ、2 つのイシューラベル付けイベントに対して 2 つです。

```yaml
on:
  issues:
    types:
      - opened
      - labeled
```

各イベントとそのアクティビティの種類の詳細については、「ワークフローをトリガーするイベント」を参照してください。

### フィルターを使用する

一部のイベントには、ワークフローをいつ実行するかをより細かく制御できるフィルターがあります。

たとえば、`push` イベントには `branches` フィルターがあり、任意の `push` が発生したときではなく、`branches` フィルターに一致するブランチへの `push` が発生したときにのみワークフローを実行します。

```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'
```

### 複数のイベントでアクティビティの種類とフィルターを使用する

イベントにアクティビティの種類またはフィルターを指定し、ワークフローが複数のイベントでトリガーされる場合は、各イベントを個別に設定する必要があります。設定のないイベントも含め、すべてのイベントにコロン（`:`）を付ける必要があります。

たとえば、次の `on` 値を持つワークフローは、次の場合に実行されます。

- ラベルが作成された場合
- リポジトリ内の `main` ブランチに `push` が行われた場合
- GitHub Pages が有効なブランチに `push` が行われた場合

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

## `on.<event_name>.types`

ワークフロー実行をトリガーするアクティビティの種類を定義するには、`on.<event_name>.types` を使用します。ほとんどの GitHub イベントは、複数の種類のアクティビティによってトリガーされます。たとえば、`label` はラベルが作成、編集、または削除されたときにトリガーされます。`types` キーワードを使用すると、ワークフローの実行を引き起こすアクティビティを絞り込むことができます。1 つのアクティビティの種類だけが ウェブフックイベントをトリガーする場合、`types` キーワードは不要です。

イベントの種類の配列を使用できます。各イベントとそのアクティビティの種類の詳細については、「ワークフローをトリガーするイベント」を参照してください。

```yaml
on:
  label:
    types: [created, edited]
```

## `on.<pull_request|pull_request_target>.<branches|branches-ignore>`

`pull_request` および `pull_request_target` イベントを使用する場合、特定のブランチを対象とするプルリクエストに対してのみワークフローを実行するよう設定できます。

ブランチ名パターンを含めたい場合、またはブランチ名パターンを含めると同時に除外もしたい場合は、`branches` フィルターを使用します。ブランチ名パターンを除外するだけの場合は、`branches-ignore` フィルターを使用します。同じワークフロー内の同じイベントに対して、`branches` フィルターと `branches-ignore` フィルターの両方を使用することはできません。

`branches`/`branches-ignore` と `paths`/`paths-ignore` の両方を定義した場合、ワークフローは両方のフィルターが満たされたときにのみ実行されます。

`branches` および `branches-ignore` キーワードでは、複数のブランチ名に一致させるために、`*`、`**`、`+`、`?`、`!` などの文字を使用する glob パターンを受け入れます。名前にこれらの文字のいずれかが含まれており、リテラル一致を行いたい場合は、それぞれの特殊文字を `\` でエスケープする必要があります。glob パターンの詳細については、「GitHub Actions のワークフロー構文」を参照してください。

### 例: ブランチを含める

`branches` に定義されたパターンは、Git 参照 の名前に対して評価されます。たとえば、次のワークフローは、次を対象とするプルリクエストに対して `pull_request` イベントが発生するたびに実行されます。

- `main` という名前のブランチ（`refs/heads/main`）
- `mona/octocat` という名前のブランチ（`refs/heads/mona/octocat`）
- 名前が `releases/` で始まるブランチ。たとえば `releases/10`（`refs/heads/releases/10`）

```yaml
on:
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches:
      - main
      - 'mona/octocat'
      - 'releases/**'
```

ブランチフィルター、パスフィルター、またはコミットメッセージによってワークフローがスキップされた場合、そのワークフローに関連付けられたチェックは「Pending」状態のままになります。それらのチェックが成功することを必須としているプルリクエストは、マージがブロックされます。

### 例: ブランチを除外する

パターンが `branches-ignore` パターンに一致すると、ワークフローは実行されません。`branches-ignore` に定義されたパターンは、Git 参照 の名前に対して評価されます。たとえば、次のワークフローは、プルリクエストが次を対象としていない限り、`pull_request` イベントが発生するたびに実行されます。

- `mona/octocat` という名前のブランチ（`refs/heads/mona/octocat`）
- 名前が `releases/**-alpha` に一致するブランチ。たとえば `releases/beta/3-alpha`（`refs/heads/releases/beta/3-alpha`）

```yaml
on:
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches-ignore:
      - 'mona/octocat'
      - 'releases/**-alpha'
```

### 例: ブランチを含めるおよび除外する

単一のワークフロー内で同じイベントをフィルターするために、`branches` と `branches-ignore` を使用することはできません。単一のイベントに対してブランチパターンを含めると同時に除外もしたい場合は、`branches` フィルターと `!` 文字を使用して、どのブランチを除外するかを示します。

`!` 文字を含むブランチを定義する場合、`!` 文字を含まないブランチも少なくとも 1 つ定義する必要があります。ブランチを除外するだけの場合は、代わりに `branches-ignore` を使用します。

パターンを定義する順序は重要です。

- 肯定一致の後に一致する否定パターン（`!` が接頭辞として付いたもの）があると、その Git 参照 は除外されます。
- 否定一致の後に一致する肯定パターンがあると、その Git 参照 は再び含められます。

次のワークフローは、`releases/10` または `releases/beta/mona` を対象とするプルリクエストの `pull_request` イベントで実行されますが、`releases/10-alpha` または `releases/beta/3-alpha` を対象とするプルリクエストでは実行されません。これは、否定パターン `!releases/**-alpha` が肯定パターンの後に続いているためです。

```yaml
on:
  pull_request:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

## `on.push.<branches|tags|branches-ignore|tags-ignore>`

`push` イベントを使用する場合、特定のブランチまたはタグでワークフローを実行するよう設定できます。

ブランチ名パターンを含めたい場合、またはブランチ名パターンを含めると同時に除外もしたい場合は、`branches` フィルターを使用します。ブランチ名パターンを除外するだけの場合は、`branches-ignore` フィルターを使用します。同じワークフロー内の同じイベントに対して、`branches` フィルターと `branches-ignore` フィルターの両方を使用することはできません。

タグ名パターンを含めたい場合、またはタグ名パターンを含めると同時に除外もしたい場合は、`tags` フィルターを使用します。タグ名パターンを除外するだけの場合は、`tags-ignore` フィルターを使用します。同じワークフロー内の同じイベントに対して、`tags` フィルターと `tags-ignore` フィルターの両方を使用することはできません。

`tags`/`tags-ignore` のみ、または `branches`/`branches-ignore` のみを定義した場合、ワークフローは未定義の Git 参照 に影響するイベントでは実行されません。`tags`/`tags-ignore` も `branches`/`branches-ignore` も定義しない場合、ワークフローはブランチまたはタグのいずれかに影響するイベントで実行されます。`branches`/`branches-ignore` と `paths`/`paths-ignore` の両方を定義した場合、ワークフローは両方のフィルターが満たされたときにのみ実行されます。

`branches`、`branches-ignore`、`tags`、`tags-ignore` キーワードでは、複数のブランチ名またはタグ名に一致させるために、`*`、`**`、`+`、`?`、`!` などの文字を使用する glob パターンを受け入れます。名前にこれらの文字のいずれかが含まれており、リテラル一致を行いたい場合は、それぞれの特殊文字を `\` でエスケープする必要があります。glob パターンの詳細については、「GitHub Actions のワークフロー構文」を参照してください。

### 例: ブランチとタグを含める

`branches` および `tags` に定義されたパターンは、Git 参照 の名前に対して評価されます。たとえば、次のワークフローは、次への `push` イベントが発生するたびに実行されます。

- `main` という名前のブランチ（`refs/heads/main`）
- `mona/octocat` という名前のブランチ（`refs/heads/mona/octocat`）
- 名前が `releases/` で始まるブランチ。たとえば `releases/10`（`refs/heads/releases/10`）
- `v2` という名前のタグ（`refs/tags/v2`）
- 名前が `v1.` で始まるタグ。たとえば `v1.9.1`（`refs/tags/v1.9.1`）

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

### 例: ブランチとタグを除外する

パターンが `branches-ignore` または `tags-ignore` パターンに一致すると、ワークフローは実行されません。`branches` および `tags` に定義されたパターンは、Git 参照 の名前に対して評価されます。たとえば、次のワークフローは、`push` イベントが次に対するものでない限り、`push` イベントが発生するたびに実行されます。

- `mona/octocat` という名前のブランチ（`refs/heads/mona/octocat`）
- 名前が `releases/**-alpha` に一致するブランチ。たとえば `releases/beta/3-alpha`（`refs/heads/releases/beta/3-alpha`）
- `v2` という名前のタグ（`refs/tags/v2`）
- 名前が `v1.` で始まるタグ。たとえば `v1.9`（`refs/tags/v1.9`）

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

### 例: ブランチとタグを含めるおよび除外する

単一のワークフロー内で同じイベントをフィルターするために、`branches` と `branches-ignore` を使用することはできません。同様に、単一のワークフロー内で同じイベントをフィルターするために、`tags` と `tags-ignore` を使用することはできません。単一のイベントに対してブランチまたはタグのパターンを含めると同時に除外もしたい場合は、`branches` または `tags` フィルターと `!` 文字を使用して、どのブランチまたはタグを除外するかを示します。

`!` 文字を含むブランチを定義する場合、`!` 文字を含まないブランチも少なくとも 1 つ定義する必要があります。ブランチを除外するだけの場合は、代わりに `branches-ignore` を使用します。同様に、`!` 文字を含むタグを定義する場合、`!` 文字を含まないタグも少なくとも 1 つ定義する必要があります。タグを除外するだけの場合は、代わりに `tags-ignore` を使用します。

パターンを定義する順序は重要です。

- 肯定一致の後に一致する否定パターン（`!` が接頭辞として付いたもの）があると、その Git 参照 は除外されます。
- 否定一致の後に一致する肯定パターンがあると、その Git 参照 は再び含められます。

次のワークフローは、`releases/10` または `releases/beta/mona` へのプッシュで実行されますが、`releases/10-alpha` または `releases/beta/3-alpha` では実行されません。これは、否定パターン `!releases/**-alpha` が肯定パターンの後に続いているためです。

```yaml
on:
  push:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

## `on.<push|pull_request|pull_request_target>.<paths|paths-ignore>`

`push` および `pull_request` イベントを使用する場合、変更されたファイルパスに基づいてワークフローを実行するよう設定できます。タグのプッシュでは、パスフィルターは評価されません。

ファイルパスパターンを含めたい場合、またはファイルパスパターンを含めることと除外することの両方を行いたい場合は、`paths` フィルターを使用します。ファイルパスパターンを除外するだけでよい場合は、`paths-ignore` フィルターを使用します。1 つのワークフロー内の同じイベントに対して、`paths` フィルターと `paths-ignore` フィルターの両方を使用することはできません。単一のイベントでパスパターンの包含と除外の両方を行いたい場合は、除外するパスを示すために `!` 文字を前に付けた `paths` フィルターを使用します。

> **メモ**
>
> `paths` パターンを定義する順序は重要です。
>
> - 肯定一致の後に一致する否定パターン（`!` が前に付いたもの）があると、そのパスは除外されます。
> - 否定一致の後に一致する肯定パターンがあると、そのパスは再び含められます。

`branches`/`branches-ignore` と `paths`/`paths-ignore` の両方を定義した場合、ワークフローは両方のフィルターが満たされた場合にのみ実行されます。

`paths` および `paths-ignore` キーワードは、複数のパス名に一致させるために `*` および `**` ワイルドカード文字を使用する glob パターンを受け付けます。詳しくは GitHub Actions のワークフロー構文を参照してください。

### 例: パスを含める

`paths` フィルター内のパターンに少なくとも 1 つのパスが一致すると、ワークフローが実行されます。たとえば、次のワークフローは JavaScript ファイル（`.js`）をプッシュするたびに実行されます。

```yaml
on:
  push:
    paths:
      - '**.js'
```

パスフィルタリング、ブランチフィルタリング、またはコミットメッセージによってワークフローがスキップされた場合、そのワークフローに関連付けられたチェックは「Pending」状態のままになります。それらのチェックが成功することを必須としているプルリクエストは、マージがブロックされます。

### 例: パスを除外する

すべてのパス名が `paths-ignore` のパターンに一致する場合、ワークフローは実行されません。一部のパス名がパターンに一致していても、いずれかのパス名が `paths-ignore` のパターンに一致しない場合、ワークフローは実行されます。

次のパスフィルターを持つワークフローは、リポジトリのルートにある `docs` ディレクトリの外に少なくとも 1 つのファイルを含む `push` イベントでのみ実行されます。

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
```

### 例: パスを含め、除外する

単一のワークフロー内で、同じイベントをフィルターするために `paths` と `paths-ignore` を使用することはできません。単一のイベントでパスパターンの包含と除外の両方を行いたい場合は、除外するパスを示すために `!` 文字を前に付けた `paths` フィルターを使用します。

`!` 文字を含むパスを定義する場合は、`!` 文字を含まないパスも少なくとも 1 つ定義する必要があります。パスを除外するだけでよい場合は、代わりに `paths-ignore` を使用します。

`paths` パターンを定義する順序は重要です。

- 肯定一致の後に一致する否定パターン（`!` が前に付いたもの）があると、そのパスは除外されます。
- 否定一致の後に一致する肯定パターンがあると、そのパスは再び含められます。

この例は、`push` イベントに `sub-project` ディレクトリまたはそのサブディレクトリ内のファイルが含まれるたびに実行されます。ただし、そのファイルが `sub-project/docs` ディレクトリ内にある場合は除きます。たとえば、`sub-project/index.js` または `sub-project/src/index.js` を変更したプッシュはワークフロー実行をトリガーしますが、`sub-project/docs/readme.md` のみを変更したプッシュはトリガーしません。

```yaml
on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
```

### Git 差分の比較

> **メモ**
>
> 1,000 件を超えるコミットをプッシュした場合、またはタイムアウトにより GitHub が差分を生成しない場合、ワークフローは常に実行されます。

フィルターは、変更されたファイルを評価し、それらを `paths-ignore` または `paths` の一覧と照合することで、ワークフローを実行すべきかどうかを判定します。変更されたファイルがない場合、ワークフローは実行されません。

GitHub は、プッシュには 2 点差分、プルリクエストには 3 点差分を使用して、変更されたファイルの一覧を生成します。

- プルリクエスト: 3 点差分は、トピックブランチの最新バージョンと、そのトピックブランチがベースブランチと最後に同期されたコミットとの比較です。
- 既存のブランチへのプッシュ: 2 点差分は、head SHA と base SHA を直接比較します。
- 新しいブランチへのプッシュ: プッシュされた最も深いコミットの祖先の親に対する 2 点差分です。

> **メモ**
>
> 差分は 300 ファイルに制限されています。フィルターが返した最初の 300 ファイルに一致しない変更ファイルがある場合、ワークフローは実行されません。ワークフローが自動的に実行されるように、より具体的なフィルターを作成する必要がある場合があります。

詳しくは「プルリクエストでのブランチ比較について」を参照してください。

## `on.schedule`

`on.schedule` を使用して、ワークフローの時間スケジュールを定義できます。

POSIX cron 構文を使用して、特定の時刻にワークフローを実行するようスケジュールします。既定では、スケジュールされたワークフローは UTC で実行されます。タイムゾーンを考慮したスケジュール設定のために、IANA タイムゾーン文字列を使用して任意でタイムゾーンを指定できます。スケジュールされたワークフローは、既定のブランチ上の最新コミットで実行されます。スケジュールされたワークフローを実行できる最短間隔は 5 分に 1 回です。

> **メモ**
>
> 夏時間（DST）を採用しているタイムゾーンを `timezone` に設定したスケジュールでは、DST の春の時刻繰り上げ移行中、スキップされた時間帯のスケジュール済みワークフローは次の有効な時刻に進みます。たとえば、午前 2:30 のスケジュールは午前 3:00 に進みます。

Cron 構文には空白で区切られた 5 つのフィールドがあり、各フィールドは時間の単位を表します。

```text
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of the month (1 - 31)
│ │ │ ┌───────────── month (1 - 12 or JAN-DEC)
│ │ │ │ ┌───────────── day of the week (0 - 6 or SUN-SAT)
│ │ │ │ │
* * * * *
```

5 つのフィールドのいずれでも、次の演算子を使用できます。

| 演算子 | 説明 | 例 |
| --- | --- | --- |
| `*` | 任意の値 | `15 * * * *` は、毎日の毎時 15 分に実行されます。 |
| `,` | 値リストの区切り文字 | `2,10 4,5 * * *` は、毎日の 4 時台と 5 時台の 2 分および 10 分に実行されます。 |
| `-` | 値の範囲 | `30 4-6 * * *` は、4 時台、5 時台、6 時台の 30 分に実行されます。 |
| `/` | ステップ値 | `20/15 * * * *` は、20 分から 59 分まで、20 分、35 分、50 分のように 15 分ごとに実行されます。 |

この例は、America/New_York タイムゾーンで毎週月曜日から金曜日の午前 5:30 にワークフローの実行をトリガーします。

```yaml
on:
  schedule:
    - cron: '30 5 * * 1-5'
      timezone: "America/New_York"
```

1 つのワークフローは、複数の `schedule` イベントによってトリガーできます。ワークフローをトリガーした `schedule` イベントには、`github.event.schedule` コンテキストを通じてアクセスします。この例は、毎週月曜日から木曜日の 5:30 UTC、および火曜日と木曜日の 17:30 UTC にワークフローの実行をトリガーしますが、月曜日と水曜日には `Not on Monday or Wednesday` ステップをスキップします。

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

`schedule` イベントについて詳しくは、「ワークフローをトリガーするイベント」を参照してください。

## `on.workflow_call`

`on.workflow_call` を使用して、再利用可能なワークフローの入力と出力を定義します。呼び出されるワークフローで使用できるシークレットをマッピングすることもできます。再利用可能なワークフローについて詳しくは、「ワークフローの再利用」を参照してください。

## `on.workflow_call.inputs`

`workflow_call` キーワードを使用する場合、呼び出し元ワークフローから呼び出されるワークフローに渡される入力を任意で指定できます。`workflow_call` キーワードについて詳しくは、「ワークフローをトリガーするイベント」を参照してください。

利用可能な標準の入力パラメーターに加えて、`on.workflow_call.inputs` には `type` パラメーターが必要です。詳しくは `on.workflow_call.inputs.<input_id>.type` を参照してください。

`default` パラメーターが設定されていない場合、入力の既定値は、`boolean` では `false`、`number` では `0`、`string` では `""` です。

呼び出されるワークフロー内では、`inputs` コンテキストを使用して入力を参照できます。詳しくは「コンテキストリファレンス」を参照してください。

呼び出し元ワークフローが、呼び出されるワークフローで指定されていない入力を渡すと、エラーになります。

### `on.workflow_call.inputs` の例

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

詳しくは「ワークフローの再利用」を参照してください。

## `on.workflow_call.inputs.<input_id>.type`

`on.workflow_call` キーワードに入力が定義されている場合は必須です。このパラメーターの値は、入力のデータ型を指定する文字列です。これは `boolean`、`number`、または `string` のいずれかである必要があります。

## `on.workflow_call.outputs`

呼び出されるワークフローの出力のマップです。呼び出されるワークフローの出力は、呼び出し元ワークフロー内のすべての下流ジョブで使用できます。各出力には、識別子、任意の説明、および値があります。値は、呼び出されるワークフロー内のジョブからの出力の値に設定する必要があります。

次の例では、この再利用可能なワークフローに対して `workflow_output1` と `workflow_output2` という 2 つの出力が定義されています。これらは、どちらも `my_job` というジョブからの `job_output1` および `job_output2` という出力にマッピングされます。

### `on.workflow_call.outputs` の例

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

ジョブ出力の参照方法については、`jobs.<job_id>.outputs` を参照してください。詳しくは「ワークフローの再利用」を参照してください。

## `on.workflow_call.secrets`

呼び出されるワークフローで使用できるシークレットのマップです。

呼び出されるワークフロー内では、`secrets` コンテキストを使用してシークレットを参照できます。

> **メモ**
>
> シークレットを入れ子になった再利用可能なワークフローに渡す場合は、そのシークレットを渡すために `jobs.<job_id>.secrets` を再度使用する必要があります。詳しくは「ワークフローの再利用」を参照してください。

呼び出し元ワークフローが、呼び出されるワークフローで指定されていないシークレットを渡すと、エラーになります。

### `on.workflow_call.secrets` の例

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

## `on.workflow_call.secrets.<secret_id>`

シークレットに関連付ける文字列識別子です。

## `on.workflow_call.secrets.<secret_id>.required`

シークレットを指定する必要があるかどうかを示すブール値です。

## `on.workflow_run.<branches|branches-ignore>`

`workflow_run` イベントを使用する場合、ワークフローをトリガーするために、トリガー元ワークフローがどのブランチで実行されている必要があるかを指定できます。

`branches` および `branches-ignore` フィルターは、複数のブランチ名に一致させるために `*`、`**`、`+`、`?`、`!` などの文字を使用する glob パターンを受け付けます。名前にこれらの文字が含まれており、リテラル一致させたい場合は、これらの特殊文字をそれぞれ `\` でエスケープする必要があります。glob パターンについて詳しくは、GitHub Actions のワークフロー構文を参照してください。

たとえば、次のトリガーを持つワークフローは、`Build` という名前のワークフローが、名前が `releases/` で始まるブランチで実行された場合にのみ実行されます。

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [requested]
    branches:
      - 'releases/**'
```

次のトリガーを持つワークフローは、`Build` という名前のワークフローが `canary` という名前ではないブランチで実行された場合にのみ実行されます。

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [requested]
    branches-ignore:
      - "canary"
```

1 つのワークフロー内の同じイベントに対して、`branches` フィルターと `branches-ignore` フィルターの両方を使用することはできません。単一のイベントでブランチパターンの包含と除外の両方を行いたい場合は、除外するブランチを示すために `!` 文字とともに `branches` フィルターを使用します。

パターンを定義する順序は重要です。

- 肯定一致の後に一致する否定パターン（`!` が前に付いたもの）があると、そのブランチは除外されます。
- 否定一致の後に一致する肯定パターンがあると、そのブランチは再び含められます。

たとえば、次のトリガーを持つワークフローは、`Build` という名前のワークフローが `releases/10` または `releases/beta/mona` という名前のブランチで実行された場合に実行されますが、`releases/10-alpha`、`releases/beta/3-alpha`、または `main` では実行されません。

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [requested]
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

## `on.workflow_dispatch`

`workflow_dispatch` イベントを使用する場合、ワークフローに渡される入力を任意で指定できます。

このトリガーは、ワークフローファイルが既定のブランチ上にある場合にのみイベントを受信します。

## `on.workflow_dispatch.inputs`

トリガーされたワークフローは、`inputs` コンテキストで入力を受け取ります。詳しくは「コンテキスト」を参照してください。

> **メモ**
>
> ワークフローは、`github.event.inputs` コンテキストでも入力を受け取ります。`inputs` コンテキストと `github.event.inputs` コンテキストの情報は同一ですが、`inputs` コンテキストはブール値を文字列に変換せず、ブール値として保持します。`choice` 型は文字列に解決され、単一選択可能なオプションです。
> `inputs` の最上位プロパティの最大数は 25 です。
> `inputs` の最大ペイロードは 65,535 文字です。

### `on.workflow_dispatch.inputs` の例

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

## `on.workflow_dispatch.inputs.<input_id>.required`

入力を指定する必要があるかどうかを示すブール値です。

## `on.workflow_dispatch.inputs.<input_id>.type`

このパラメーターの値は、入力のデータ型を指定する文字列です。これは `boolean`、`choice`、`number`、`environment`、または `string` のいずれかである必要があります。

## `permissions`

`permissions` を使用すると、GITHUB_TOKEN に付与されるデフォルトの権限を変更し、必要に応じてアクセス権を追加または削除できます。これにより、必要最小限のアクセスだけを許可できます。詳しくは、ワークフローで認証に GITHUB_TOKEN を使用する方法を参照してください。

`permissions` は、ワークフロー内のすべてのジョブに適用するためのトップレベルのキーとして使用することも、特定のジョブ内で使用することもできます。特定のジョブ内に `permissions` キーを追加すると、そのジョブ内で GITHUB_TOKEN を使用するすべてのアクションと `run` コマンドに、指定したアクセス権が付与されます。詳しくは、`jobs.<job_id>.permissions` を参照してください。

組織の所有者は、リポジトリレベルで GITHUB_TOKEN の書き込みアクセスを制限できます。詳しくは、組織で GitHub Actions を無効化または制限する方法を参照してください。

ワークフローが `pull_request_target` イベントによってトリガーされた場合、公開フォークからトリガーされた場合でも、GITHUB_TOKEN にはリポジトリへの読み取りおよび書き込み権限が付与されます。詳しくは、ワークフローをトリガーするイベントを参照してください。

次の表に示す利用可能な各権限には、`read`（該当する場合）、`write`、または `none` のいずれかのアクセスレベルを割り当てることができます。`write` には `read` が含まれます。これらの権限のいずれかについてアクセス権を指定すると、指定されていないすべての権限は `none` に設定されます。

利用可能な権限と、それぞれがアクションに許可する操作の詳細は次のとおりです。

| 権限 | GITHUB_TOKEN を使用するアクションに許可される操作 |
| --- | --- |
| `actions` | GitHub Actions を操作します。たとえば、`actions: write` は、アクションがワークフロー実行をキャンセルすることを許可します。詳しくは、GitHub Apps に必要な権限を参照してください。 |
| `artifact-metadata` | アーティファクトのメタデータを操作します。たとえば、`artifact-metadata: write` は、アクションがビルドアーティファクトに代わってストレージレコードを作成することを許可します。詳しくは、アーティファクトメタデータの REST API エンドポイントを参照してください。 |
| `attestations` | アーティファクトの証明を操作します。たとえば、`attestations: write` は、アクションがビルドのアーティファクト証明を生成することを許可します。詳しくは、ビルドの来歴を確立するためにアーティファクト証明を使用する方法を参照してください。 |
| `checks` | チェック実行とチェックスイートを操作します。たとえば、`checks: write` は、アクションがチェック実行を作成することを許可します。詳しくは、GitHub Apps に必要な権限を参照してください。 |
| `code-quality` | コード品質を操作します。たとえば、`code-quality: write` は、アクションがコードカバレッジレポートをアップロードすることを許可します。詳しくは、GitHub Code Quality についてを参照してください。 |
| `contents` | リポジトリの内容を操作します。たとえば、`contents: read` は、アクションがコミットを一覧表示することを許可し、`contents: write` は、アクションがリリースを作成することを許可します。詳しくは、GitHub Apps に必要な権限を参照してください。 |
| `deployments` | デプロイを操作します。たとえば、`deployments: write` は、アクションが新しいデプロイを作成することを許可します。詳しくは、GitHub Apps に必要な権限を参照してください。 |
| `discussions` | GitHub Discussions を操作します。たとえば、`discussions: write` は、アクションがディスカッションを閉じる、または削除することを許可します。詳しくは、ディスカッションに GraphQL API を使用する方法を参照してください。 |
| `id-token` | OpenID Connect (OIDC) トークンを取得します。これには `id-token: write` が必要です。詳しくは、OpenID Connect を参照してください。 |
| `issues` | issue を操作します。たとえば、`issues: write` は、アクションが issue にコメントを追加することを許可します。詳しくは、GitHub Apps に必要な権限を参照してください。 |
| `models` | GitHub Models で AI 推論応答を生成します。たとえば、`models: read` は、アクションが GitHub Models 推論 API を使用することを許可します。AI モデルでプロトタイプを作成する方法を参照してください。 |
| `packages` | GitHub Packages を操作します。たとえば、`packages: write` は、アクションが GitHub Packages にパッケージをアップロードして公開することを許可します。詳しくは、GitHub Packages の権限についてを参照してください。 |
| `pages` | GitHub Pages を操作します。たとえば、`pages: write` は、アクションが GitHub Pages のビルドを要求することを許可します。詳しくは、GitHub Apps に必要な権限を参照してください。 |
| `pull-requests` | プルリクエストを操作します。たとえば、`pull-requests: write` は、アクションがプルリクエストにラベルを追加することを許可します。詳しくは、GitHub Apps に必要な権限を参照してください。 |
| `security-events` | GitHub のコードスキャンアラートを操作します。たとえば、`security-events: read` は、アクションがリポジトリのコードスキャンアラートを一覧表示することを許可し、`security-events: write` は、アクションがコードスキャンアラートの状態を更新することを許可します。詳しくは、「コードスキャンアラート」のリポジトリ権限を参照してください。<br><br>Dependabot アラートには、`vulnerability-alerts` 権限を使用してください。シークレットスキャンアラートはこの権限では読み取れず、GitHub App または個人用アクセストークンが必要です。詳しくは、「GitHub Apps に必要な権限」の「シークレットスキャンアラート」のリポジトリ権限を参照してください。 |
| `statuses` | コミットステータスを操作します。たとえば、`statuses:read` は、アクションが指定された参照のコミットステータスを一覧表示することを許可します。詳しくは、GitHub Apps に必要な権限を参照してください。 |
| `vulnerability-alerts` | Dependabot アラートを読み取ります。たとえば、`vulnerability-alerts: read` は、アクションがリポジトリの Dependabot アラートを一覧表示することを許可します。サポートされるのは `read` と `none` のみで、`write` は有効ではありません。`write-all` または `read-all` が使用されると、`vulnerability-alerts` は自動的に `read` として含まれます。詳しくは、「Dependabot alerts」のリポジトリ権限を参照してください。 |

### GITHUB_TOKEN スコープのアクセス権を定義する

`permissions` キー内で利用可能な権限の値として `read`、`write`、または `none` を指定することで、GITHUB_TOKEN が許可するアクセス権を定義できます。

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

これらの権限のいずれかについてアクセス権を指定すると、指定されていないすべての権限は `none` に設定されます。

利用可能なすべての権限に対して `read-all` または `write-all` のいずれかのアクセス権を定義するには、次の構文を使用できます。

```yaml
permissions: read-all
permissions: write-all
```

利用可能なすべての権限を無効にするには、次の構文を使用できます。

```yaml
permissions: {}
```

### フォークされたリポジトリで権限を変更する

フォークされたリポジトリに対しては、`permissions` キーを使用して読み取り権限を追加および削除できますが、通常は書き込みアクセスを付与できません。この動作の例外は、管理者ユーザーが GitHub Actions 設定で `Send write tokens to workflows from pull requests` オプションを選択している場合です。詳しくは、リポジトリの GitHub Actions 設定を管理する方法を参照してください。

### ワークフロージョブの権限が計算される仕組み

GITHUB_TOKEN の権限は、最初に Enterprise、組織、またはリポジトリのデフォルト設定に設定されます。これらのレベルのいずれかでデフォルトが制限付き権限に設定されている場合、その設定が該当するリポジトリに適用されます。たとえば、組織レベルで制限付きデフォルトを選択した場合、その組織内のすべてのリポジトリは、制限付き権限をデフォルトとして使用します。その後、権限はワークフローファイル内の構成に基づいて、まずワークフローレベルで、次にジョブレベルで調整されます。最後に、ワークフローがフォークされたリポジトリからの `pull_request_target` 以外の pull request イベントによってトリガーされ、`Send write tokens to workflows from pull requests` 設定が選択されていない場合、書き込み権限は読み取り専用に変更されるように調整されます。

### ワークフロー内のすべてのジョブに GITHUB_TOKEN 権限を設定する

ワークフローのトップレベルで権限を指定すると、その設定をワークフロー内のすべてのジョブに適用できます。

### 例: ワークフロー全体に GITHUB_TOKEN 権限を設定する

この例は、ワークフロー内のすべてのジョブに適用される GITHUB_TOKEN の権限を設定する方法を示しています。すべての権限に読み取りアクセスが付与されます。

```yaml
name: "My workflow"

on: [ push ]

permissions: read-all

jobs:
  ...
```

### フォークされたリポジトリで `permissions` キーを使用する

フォークされたリポジトリに対しては、`permissions` キーを使用して読み取り権限を追加および削除できますが、通常は書き込みアクセスを付与できません。この動作の例外は、管理者ユーザーが GitHub Actions 設定で `Send write tokens to workflows from pull requests` オプションを選択している場合です。詳しくは、リポジトリの GitHub Actions 設定を管理する方法を参照してください。

### Dependabot によってトリガーされたワークフロー実行の権限

Dependabot プルリクエストによってトリガーされたワークフロー実行は、フォークされたリポジトリからの実行であるかのように実行されるため、読み取り専用の GITHUB_TOKEN を使用します。これらのワークフロー実行は、どのシークレットにもアクセスできません。これらのワークフローを安全に保つための戦略については、安全な使用に関するリファレンスを参照してください。

## `env`

ワークフロー内のすべてのジョブのステップで使用できる変数のマップです。単一のジョブのステップ、または単一のステップでのみ使用できる変数を設定することもできます。詳しくは、`jobs.<job_id>.env` と `jobs.<job_id>.steps[*].env` を参照してください。

`env` マップ内の変数は、同じマップ内の他の変数に基づいて定義することはできません。

同じ名前で複数の環境変数が定義されている場合、GitHub は最も具体的な変数を使用します。たとえば、ステップで定義された環境変数は、そのステップの実行中、同じ名前のジョブおよびワークフローの環境変数を上書きします。ジョブに定義された環境変数は、そのジョブの実行中、同じ名前のワークフロー変数を上書きします。

### `env` の例

```yaml
env:
  SERVER: production
```

## `defaults`

`defaults` を使用して、ワークフロー内のすべてのジョブに適用されるデフォルト設定のマップを作成します。ジョブでのみ使用できるデフォルト設定を設定することもできます。詳しくは、`jobs.<job_id>.defaults` を参照してください。

同じ名前で複数のデフォルト設定が定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで定義された同じ名前のデフォルト設定を上書きします。

## `defaults.run`

`defaults.run` を使用すると、ワークフロー内のすべての `run` ステップに対して、デフォルトのシェルと作業ディレクトリのオプションを指定できます。ジョブでのみ使用できる `run` のデフォルト設定を設定することもできます。詳しくは、`jobs.<job_id>.defaults.run` を参照してください。このキーワードでは、コンテキストまたは式を使用できません。

同じ名前で複数のデフォルト設定が定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで定義された同じ名前のデフォルト設定を上書きします。

### 例: デフォルトのシェルと作業ディレクトリを設定する

```yaml
defaults:
  run:
    shell: bash
    working-directory: ./scripts
```

## `defaults.run.shell`

ステップのシェルを定義するには、`shell` を使用します。このキーワードは複数のコンテキストを参照できます。詳しくは、コンテキストを参照してください。

| サポートされるプラットフォーム | シェルパラメーター | 説明 | 内部で実行されるコマンド |
| --- | --- | --- | --- |
| Linux / macOS | `unspecified` | Windows 以外のプラットフォームでのデフォルトのシェルです。`bash` を明示的に指定した場合とは異なるコマンドが実行される点に注意してください。`bash` がパス内に見つからない場合、これは `sh` として扱われます。 | `bash -e {0}` |
| All | `bash` | Windows 以外のプラットフォームでのデフォルトのシェルで、`sh` へのフォールバックがあります。Windows で bash シェルを指定すると、Git for Windows に含まれる bash シェルが使用されます。 | `bash --noprofile --norc -eo pipefail {0}` |
| All | `pwsh` | PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | `pwsh -command ". '{0}'"` |
| All | `python` | `python` コマンドを実行します。 | `python {0}` |
| Linux / macOS | `sh` | シェルが指定されておらず、パス内に `bash` が見つからない場合の、Windows 以外のプラットフォーム向けのフォールバック動作です。 | `sh -e {0}` |
| Windows | `cmd` | GitHub はスクリプト名に拡張子 `.cmd` を追加し、`{0}` に代入します。 | `%ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}"".` |
| Windows | `pwsh` | これは Windows で使用されるデフォルトのシェルです。PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。セルフホスト Windows ランナーに PowerShell Core がインストールされていない場合は、代わりに PowerShell Desktop が使用されます。 | `pwsh -command ". '{0}'".` |
| Windows | `powershell` | PowerShell Desktop です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | `powershell -command ". '{0}'".` |

同じ名前で複数のデフォルト設定が定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで定義された同じ名前のデフォルト設定を上書きします。

## `defaults.run.working-directory`

ステップのシェルの作業ディレクトリを定義するには、`working-directory` を使用します。このキーワードは複数のコンテキストを参照できます。詳しくは、コンテキストを参照してください。

> **ヒント**
> 
> 割り当てた作業ディレクトリがランナー上に存在することを、そこでシェルを実行する前に確認してください。同じ名前で複数のデフォルト設定が定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで定義された同じ名前のデフォルト設定を上書きします。

## `concurrency`

`concurrency` を使用すると、同じ同時実行グループを使用する単一のジョブまたはワークフローだけが一度に実行されるようにできます。同時実行グループには、任意の文字列または式を指定できます。式では、`github`、`inputs`、`vars` コンテキストのみを使用できます。式について詳しくは、ワークフローとアクションで式を評価する方法を参照してください。

ジョブレベルで `concurrency` を指定することもできます。詳しくは、`jobs.<job_id>.concurrency` を参照してください。

これは、任意の時点で同時実行グループ内に、実行中のジョブまたはワークフローが最大 1 つしか存在できないことを意味します。同時実行するジョブまたはワークフローがキューに入れられたとき、リポジトリ内で同じ同時実行グループを使用している別のジョブまたはワークフローが進行中である場合、キューに入れられたジョブまたはワークフローは保留中になります。デフォルトでは、同じ同時実行グループ内に既存の保留中のジョブまたはワークフローがある場合、それはキャンセルされ、新しくキューに入れられたジョブまたはワークフローがその代わりになります。

同じ同時実行グループ内で現在実行中のジョブまたはワークフローもキャンセルするには、`cancel-in-progress: true` を指定します。同じ同時実行グループ内で現在実行中のジョブまたはワークフローを条件付きでキャンセルするには、許可されている式コンテキストのいずれかを使用した式として `cancel-in-progress` を指定できます。

複数の保留中のジョブまたはワークフロー実行が同じ同時実行グループ内で待機できるようにするには、省略可能な `queue` プロパティを使用します。`queue` プロパティは次の値を受け付けます。

`single`（デフォルト）: 同時実行グループ内で保留中にできるジョブまたはワークフロー実行は最大 1 つです。新しいジョブまたはワークフロー実行がキューに入れられると、同じグループ内の既存の保留中のジョブまたはワークフロー実行はキャンセルされ、置き換えられます。

`max`: 同時実行グループ内で最大 100 個のジョブまたはワークフロー実行を保留中にできます。キューがいっぱいになると、追加のジョブまたはワークフロー実行はキャンセルされます。

`queue: max` と `cancel-in-progress: true` の組み合わせは許可されず、ワークフロー検証エラーになります。

> **メモ**
> 
> 同時実行グループ名では大文字と小文字は区別されません。たとえば、`prod` と `Prod` は同じ同時実行グループとして扱われます。
> 同じ同時実行グループ内のジョブまたはワークフロー実行は、各ワークフローがディスパッチされた時刻ではなく、それぞれが同時実行グループの待機を開始した時刻に従って、先入れ先出し（FIFO）順に処理されます。ジョブまたは実行の実際の開始時刻は変動する可能性があるため、順序は保証されません。

### 例: 同時実行とデフォルトの動作を使用する

GitHub Actions のデフォルトの動作では、複数のジョブまたはワークフロー実行を同時に実行できます。`concurrency` キーワードを使用すると、ワークフロー実行の同時実行を制御できます。

たとえば、特定のブランチに対するワークフロー実行全体の同時実行を制限するために、トリガー条件を定義した直後に `concurrency` キーワードを使用できます。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

ワークフロー内のジョブの同時実行も、ジョブレベルで `concurrency` キーワードを使用することで制限できます。

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

### 例: 同時実行グループ

同時実行グループは、同じ同時実行キーを共有するワークフロー実行またはジョブの実行を管理および制限する方法を提供します。

同時実行キーは、ワークフローまたはジョブを同時実行グループにまとめるために使用されます。同時実行キーを定義すると、GitHub Actions は、そのキーを持つワークフローまたはジョブが常に 1 つだけ実行されるようにします。同じ同時実行キーで新しいワークフロー実行またはジョブが開始されると、GitHub Actions は、そのキーですでに実行中のワークフローまたはジョブをキャンセルします。同時実行キーには、ハードコードされた文字列を使用することも、コンテキスト変数を含む動的な式を使用することもできます。

ワークフロー内で同時実行条件を定義し、ワークフローまたはジョブを同時実行グループの一部にすることができます。

これは、ワークフロー実行またはジョブが開始されると、GitHub が同じ同時実行グループ内ですでに進行中のワークフロー実行またはジョブをキャンセルすることを意味します。これは、必要以上のリソースを消費したり競合を引き起こしたりする可能性のあるアクションを防ぐために、ステージング環境へのデプロイに使用されるものなど、特定のワークフローまたはジョブのセットの並列実行を防ぎたいシナリオで役立ちます。

この例では、`job-1` は `staging_environment` という名前の同時実行グループの一部です。つまり、`job-1` の新しい実行がトリガーされた場合、`staging_environment` 同時実行グループ内ですでに進行中の同じジョブの実行はキャンセルされます。

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: staging_environment
      cancel-in-progress: true
```

または、ワークフロー内で `concurrency: ci-${{ github.ref }}` のような動的な式を使用すると、ワークフローまたはジョブは、`ci-` に続いてワークフローをトリガーしたブランチまたはタグの参照を付けた名前の同時実行グループの一部になります。この例では、前の実行がまだ進行中の間に `main` ブランチへ新しいコミットがプッシュされると、前の実行はキャンセルされ、新しい実行が開始されます。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

### 例: 複数の保留中の実行をキューに入れる

デフォルトでは、同時実行グループ内で一度に保留中にできるジョブまたはワークフロー実行は 1 つだけです。複数の実行をキャンセルせずにキューに入れられるようにするには、`queue: max` を設定します。`queue: max` を使用すると、最大 100 個のジョブまたはワークフロー実行が同時実行グループ内で待機できます。キューがいっぱいになると、追加の実行はキャンセルされます。

たとえば、次のワークフローは本番環境へのデプロイをキューに入れ、それぞれの実行が同時実行グループの待機を開始した時刻に基づいて、1 つずつ順番に処理します。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: production-deploy
  queue: max
```

`queue: max` は `cancel-in-progress: true` と組み合わせることはできません。これは、進行中の実行の処理方法について、2 つのオプションが相反する動作を表すためです。

### 例: 同時実行を使用して進行中のジョブまたは実行をキャンセルする

GitHub Actions で同時実行を使用して進行中のジョブまたは実行をキャンセルするには、`cancel-in-progress` オプションを `true` に設定した `concurrency` キーを使用できます。

```yaml
concurrency:
  group: ${{ github.ref }}
  cancel-in-progress: true
```

この例では、特定の同時実行グループを定義しない場合、GitHub Actions はジョブまたはワークフローの進行中の実行をすべてキャンセルする点に注意してください。

### 例: フォールバック値を使用する

特定のイベントでのみ定義されるプロパティを使用してグループ名を作成する場合は、フォールバック値を使用できます。たとえば、`github.head_ref` は `pull_request` イベントでのみ定義されます。ワークフローが `pull_request` イベントに加えて他のイベントにも応答する場合、構文エラーを避けるためにフォールバックを指定する必要があります。次の同時実行グループは、`pull_request` イベントでのみ進行中のジョブまたは実行をキャンセルします。`github.head_ref` が未定義の場合、同時実行グループは実行 ID にフォールバックします。これは、その実行に対して一意であり、かつ必ず定義されています。

```yaml
concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
```

### 例: 現在のワークフローの進行中のジョブまたは実行だけをキャンセルする

同じリポジトリ内に複数のワークフローがある場合、他のワークフローの進行中のジョブまたは実行をキャンセルしないように、同時実行グループ名はワークフロー間で一意である必要があります。そうでない場合、以前から進行中または保留中のジョブは、ワークフローに関係なくキャンセルされます。

同じワークフローの進行中の実行だけをキャンセルするには、`github.workflow` プロパティを使用して同時実行グループを作成できます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### 例: 特定のブランチでのみ進行中のジョブをキャンセルする

特定のブランチでは進行中のジョブをキャンセルし、他のブランチではキャンセルしたくない場合は、`cancel-in-progress` で条件式を使用できます。たとえば、リリースブランチではなく開発ブランチで進行中のジョブをキャンセルしたい場合に、このようにできます。

リリースブランチで実行されていない場合に、同じワークフローの進行中の実行だけをキャンセルするには、`cancel-in-progress` を次のような式に設定できます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !contains(github.ref, 'release/')}}
```

この例では、`release/1.2.3` ブランチへの複数回のプッシュは、進行中の実行をキャンセルしません。`main` など別のブランチへのプッシュは、進行中の実行をキャンセルします。

## `jobs`

ワークフロー実行は 1 つ以上のジョブで構成され、既定では並列に実行されます。ジョブを順番に実行するには、`jobs.<job_id>.needs` キーワードを使用して他のジョブへの依存関係を定義できます。

各ジョブは、`runs-on` で指定されたランナー環境で実行されます。

ワークフローの使用制限内であれば、ジョブはいくつでも実行できます。詳しくは、「GitHub ホストランナーの課金と使用状況」および「セルフホストランナーの使用制限に関する Actions の制限」を参照してください。

ワークフロー実行内で実行されているジョブの一意の識別子を調べる必要がある場合は、GitHub API を使用できます。詳しくは、「GitHub Actions の REST API エンドポイント」を参照してください。

## `jobs.<job_id>`

`jobs.<job_id>` を使用して、ジョブに一意の識別子を付けます。キー `job_id` は文字列で、その値はジョブの構成データのマップです。`<job_id>` は、`jobs` オブジェクト内で一意の文字列に置き換える必要があります。`<job_id>` は英字または `_` で始まり、英数字、`-`、または `_` のみを含める必要があります。

### 例: ジョブを作成する

この例では、2 つのジョブが作成され、それぞれの `job_id` 値は `my_first_job` と `my_second_job` です。

```yaml
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```

## `jobs.<job_id>.name`

`jobs.<job_id>.name` を使用して、GitHub UI に表示されるジョブの名前を設定します。

## `jobs.<job_id>.permissions`

特定のジョブについて、`jobs.<job_id>.permissions` を使用すると、`GITHUB_TOKEN` に付与される既定の権限を変更し、必要に応じてアクセスを追加または削除して、必要最小限のアクセスだけを許可できます。詳しくは、「ワークフローでの認証に GITHUB_TOKEN を使用する」を参照してください。

ジョブ定義内で権限を指定することで、必要に応じてジョブごとに `GITHUB_TOKEN` の異なる権限セットを構成できます。別の方法として、ワークフロー内のすべてのジョブに対して権限を指定することもできます。ワークフローレベルで権限を定義する方法については、`permissions` を参照してください。

次の表に示す利用可能な各権限には、`read`（該当する場合）、`write`、または `none` のいずれかのアクセスレベルを割り当てることができます。`write` には `read` が含まれます。これらの権限のいずれかについてアクセスを指定すると、指定されていない権限はすべて `none` に設定されます。

利用可能な権限と、各権限でアクションが実行できることの詳細は次のとおりです。

| 権限 | GITHUB_TOKEN を使用するアクションに許可される操作 |
| --- | --- |
| actions | GitHub Actions を操作します。たとえば、`actions: write` はアクションがワークフロー実行をキャンセルすることを許可します。詳しくは、「GitHub Apps に必要な権限」を参照してください。 |
| artifact-metadata | アーティファクトのメタデータを操作します。たとえば、`artifact-metadata: write` はアクションがビルドアーティファクトの代わりにストレージレコードを作成することを許可します。詳しくは、「アーティファクトメタデータの REST API エンドポイント」を参照してください。 |
| attestations | アーティファクトの証明を操作します。たとえば、`attestations: write` はアクションがビルドのアーティファクト証明を生成することを許可します。詳しくは、「ビルドの来歴を確立するためにアーティファクト証明を使用する」を参照してください。 |
| checks | チェック実行とチェック スイートを操作します。たとえば、`checks: write` はアクションがチェック実行を作成することを許可します。詳しくは、「GitHub Apps に必要な権限」を参照してください。 |
| code-quality | コード品質を操作します。たとえば、`code-quality: write` はアクションがコードカバレッジレポートをアップロードすることを許可します。詳しくは、「GitHub Code Quality について」を参照してください。 |
| contents | リポジトリの内容を操作します。たとえば、`contents: read` はアクションがコミットを一覧表示することを許可し、`contents: write` はアクションがリリースを作成することを許可します。詳しくは、「GitHub Apps に必要な権限」を参照してください。 |
| deployments | デプロイを操作します。たとえば、`deployments: write` はアクションが新しいデプロイを作成することを許可します。詳しくは、「GitHub Apps に必要な権限」を参照してください。 |
| discussions | GitHub Discussions を操作します。たとえば、`discussions: write` はアクションがディスカッションを閉じたり削除したりすることを許可します。詳しくは、「ディスカッションに GraphQL API を使用する」を参照してください。 |
| id-token | OpenID Connect (OIDC) トークンを取得します。これには `id-token: write` が必要です。詳しくは、「OpenID Connect」を参照してください。 |
| issues | Issue を操作します。たとえば、`issues: write` はアクションが Issue にコメントを追加することを許可します。詳しくは、「GitHub Apps に必要な権限」を参照してください。 |
| models | GitHub Models で AI 推論レスポンスを生成します。たとえば、`models: read` はアクションが GitHub Models 推論 API を使用することを許可します。「AI モデルでプロトタイプを作成する」を参照してください。 |
| packages | GitHub Packages を操作します。たとえば、`packages: write` はアクションが GitHub Packages にパッケージをアップロードして公開することを許可します。詳しくは、「GitHub Packages の権限について」を参照してください。 |
| pages | GitHub Pages を操作します。たとえば、`pages: write` はアクションが GitHub Pages ビルドを要求することを許可します。詳しくは、「GitHub Apps に必要な権限」を参照してください。 |
| pull-requests | プルリクエストを操作します。たとえば、`pull-requests: write` はアクションがプルリクエストにラベルを追加することを許可します。詳しくは、「GitHub Apps に必要な権限」を参照してください。 |
| security-events | GitHub コードスキャンアラートを操作します。たとえば、`security-events: read` はアクションがリポジトリのコードスキャンアラートを一覧表示することを許可し、`security-events: write` はアクションがコードスキャンアラートの状態を更新することを許可します。詳しくは、「コードスキャンアラート」のリポジトリ権限を参照してください。<br><br>Dependabot アラートには、`vulnerability-alerts` 権限を使用してください。シークレットスキャンアラートはこの権限では読み取れず、GitHub App または個人用アクセストークンが必要です。詳しくは、「GitHub Apps に必要な権限」の「シークレットスキャンアラート」のリポジトリ権限を参照してください。 |
| statuses | コミットステータスを操作します。たとえば、`statuses:read` はアクションが特定の参照に対するコミットステータスを一覧表示することを許可します。詳しくは、「GitHub Apps に必要な権限」を参照してください。 |
| vulnerability-alerts | Dependabot アラートを読み取ります。たとえば、`vulnerability-alerts: read` はアクションがリポジトリの Dependabot アラートを一覧表示することを許可します。`read` と `none` のみがサポートされ、`write` は有効ではありません。`write-all` または `read-all` が使用されると、`vulnerability-alerts` は自動的に `read` として含まれます。詳しくは、「Dependabot アラート」のリポジトリ権限を参照してください。 |

### GITHUB_TOKEN スコープのアクセスを定義する

`permissions` キー内で利用可能な権限の値として `read`、`write`、または `none` を指定することで、`GITHUB_TOKEN` が許可するアクセスを定義できます。

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

これらの権限のいずれかについてアクセスを指定すると、指定されていない権限はすべて `none` に設定されます。

次の構文を使用して、利用可能なすべての権限に対して `read-all` または `write-all` のいずれかのアクセスを定義できます。

```yaml
permissions: read-all
permissions: write-all
```

次の構文を使用して、利用可能なすべての権限を無効にできます。

```yaml
permissions: {}
```

### フォークされたリポジトリで権限を変更する

`permissions` キーを使用して、フォークされたリポジトリの読み取り権限を追加および削除できますが、通常は書き込みアクセスを付与できません。この動作の例外は、管理者ユーザーが GitHub Actions 設定で「プルリクエストからのワークフローに書き込みトークンを送信する」オプションを選択している場合です。詳しくは、「リポジトリの GitHub Actions 設定を管理する」を参照してください。

### 例: ワークフロー内の 1 つのジョブに GITHUB_TOKEN 権限を設定する

この例では、`stale` という名前のジョブにのみ適用される `GITHUB_TOKEN` の権限を設定しています。`issues` 権限と `pull-requests` 権限には書き込みアクセスが付与されます。他のすべての権限にはアクセスがありません。

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

## `jobs.<job_id>.needs`

`jobs.<job_id>.needs` を使用して、このジョブが実行される前に正常に完了している必要があるジョブを指定します。これは文字列または文字列の配列にできます。ジョブが失敗またはスキップされた場合、そのジョブを必要とするすべてのジョブは、ジョブが続行されるようにする条件式を使用していない限りスキップされます。実行に互いを必要とする一連のジョブが含まれる場合、失敗またはスキップは、その失敗またはスキップの時点以降の依存関係チェーン内のすべてのジョブに適用されます。依存しているジョブが成功しなかった場合でもジョブを実行したい場合は、`jobs.<job_id>.if` で `always()` 条件式を使用します。

### 例: 依存ジョブの成功を必須にする

```yaml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

この例では、`job2` が開始する前に `job1` が正常に完了している必要があり、`job3` は `job1` と `job2` の両方が完了するのを待ちます。

この例のジョブは順番に実行されます。

```yaml
job1
job2
job3
```

### 例: 依存ジョブの成功を必須にしない

```yaml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    if: ${{ always() }}
    needs: [job1, job2]
```

この例では、`job3` は `always()` 条件式を使用しているため、`job1` と `job2` が完了した後、それらが成功したかどうかに関係なく常に実行されます。詳しくは、「ワークフローとアクションで式を評価する」を参照してください。

## `jobs.<job_id>.if`

`jobs.<job_id>.if` 条件を使用すると、条件が満たされない限りジョブが実行されないようにできます。サポートされている任意のコンテキストと式を使用して、条件を作成できます。このキーでサポートされるコンテキストについて詳しくは、「コンテキストのリファレンス」を参照してください。

> **メモ**
>
> `jobs.<job_id>.if` 条件は、`jobs.<job_id>.strategy.matrix` が適用される前に評価されます。

`if` 条件で式を使用する場合、GitHub Actions は `if` 条件を式として自動的に評価するため、任意で `${{ }}` 式構文を省略できます。ただし、この例外はすべての場所に適用されるわけではありません。

式が `!` で始まる場合は、`!` が YAML 形式で予約された表記であるため、常に `${{ }}` 式構文を使用するか、`''`、`""`、または `()` でエスケープする必要があります。例:

```yaml
if: ${{ ! startsWith(github.ref, 'refs/tags/') }}
```

詳しくは、「ワークフローとアクションで式を評価する」を参照してください。

### 例: 特定のリポジトリでのみジョブを実行する

この例では、`if` を使用して、`production-deploy` ジョブを実行できるタイミングを制御します。リポジトリの名前が `octo-repo-prod` で、`octo-org` 組織内にある場合にのみ実行されます。それ以外の場合、ジョブはスキップ済みとしてマークされます。

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

## `jobs.<job_id>.runs-on`

`jobs.<job_id>.runs-on` を使用して、ジョブを実行するマシンの種類を定義します。

宛先のマシンには、GitHub ホストランナー、大規模ランナー、またはセルフホストランナーを使用できます。

ランナーに割り当てられたラベル、ランナーグループのメンバーシップ、またはこれらの組み合わせに基づいて、ランナーを対象にできます。

`runs-on` は次の形式で指定できます。

- 単一の文字列
- 文字列を含む単一の変数
- 文字列、文字列を含む変数、またはその両方の組み合わせの配列
- `group` キーまたは `labels` キーを使用した `key: value` ペア

文字列または変数の配列を指定すると、ワークフローは、指定されたすべての `runs-on` 値に一致する任意のランナーで実行されます。たとえば、ここではジョブは `linux`、`x64`、`gpu` のラベルを持つセルフホストランナーでのみ実行されます。

```yaml
runs-on: [self-hosted, linux, x64, gpu]
```

詳しくは、「セルフホストランナーを選択する」を参照してください。

配列内で文字列と変数を混在させることができます。例:

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

複数のマシンでワークフローを実行したい場合は、`jobs.<job_id>.strategy` を使用します。

> **メモ**
>
> `self-hosted` のような単純な文字列を囲む引用符は不要ですが、`"${{ inputs.chosen-os }}"` のような式には引用符が必要です。

### GitHub ホストランナーを選択する

GitHub ホストランナーを使用する場合、各ジョブは `runs-on` で指定されたランナーイメージの新しいインスタンスで実行されます。

GitHub ホストランナーを使用している場合の `runs-on` の値は、ランナーラベルまたはランナーグループの名前です。標準 GitHub ホストランナーのラベルは、次の表に示されています。

詳しくは、「GitHub ホストランナー」を参照してください。

### パブリックリポジトリ用の標準 GitHub ホストランナー

パブリックリポジトリでは、次の表に示すワークフローラベルを使用するジョブは、関連付けられた仕様で実行されます。単一 CPU ランナーを除き、各 GitHub ホストランナーは GitHub によってホストされる新しい仮想マシン (VM) です。単一 CPU ランナーは、共有 VM 上のコンテナでホストされます。詳しくは、「GitHub ホストランナーのリファレンス」を参照してください。標準 GitHub ホストランナーの使用は、パブリックリポジトリでは無料かつ無制限です。

| 仮想マシン / コンテナ | プロセッサ (CPU) | メモリ (RAM) | ストレージ (SSD) | アーキテクチャ | ワークフローラベル |
| --- | --- | --- | --- | --- | --- |
| Linux | 1 | 5 GB | 14 GB | x64 | ubuntu-slim |
| Linux | 4 | 16 GB | 14 GB | x64 | ubuntu-latest, ubuntu-24.04, ubuntu-22.04 |
| Windows | 4 | 16 GB | 14 GB | x64 | windows-latest, windows-2025, windows-2025-vs2026, windows-2022 |
| Linux | 4 | 16 GB | 14 GB | arm64 | ubuntu-24.04-arm, ubuntu-22.04-arm |
| Windows | 4 | 16 GB | 14 GB | arm64 | windows-11-arm |
| macOS | 4 | 14 GB | 14 GB | Intel | macos-15-intel, macos-26-intel |
| macOS | 3 (M1) | 7 GB | 14 GB | arm64 | macos-latest, macos-14, macos-15, macos-26 |

### プライベートリポジトリ用の標準 GitHub ホストランナー

プライベートリポジトリでは、次の表に示すワークフローラベルを使用するジョブは、関連付けられた仕様の仮想マシンで実行されます。これらのランナーは GitHub アカウントに割り当てられた無料分の時間を使用し、その後は 1 分あたりの料金で課金されます。「Actions ランナーの料金」を参照してください。

| 仮想マシン | プロセッサ (CPU) | メモリ (RAM) | ストレージ (SSD) | アーキテクチャ | ワークフローラベル |
| --- | --- | --- | --- | --- | --- |
| Linux | 1 | 5 GB | 14 GB | x64 | ubuntu-slim |
| Linux | 2 | 8 GB | 14 GB | x64 | ubuntu-latest, ubuntu-24.04, ubuntu-22.04 |
| Windows | 2 | 8 GB | 14 GB | x64 | windows-latest, windows-2025, windows-2022 |
| Linux | 2 | 8 GB | 14 GB | arm64 | ubuntu-24.04-arm, ubuntu-22.04-arm |
| Windows | 2 | 8 GB | 14 GB | arm64 | windows-11-arm |
| macOS | 4 | 14 GB | 14 GB | Intel | macos-15-intel, macos-26-intel |
| macOS | 3 (M1) | 7 GB | 14 GB | arm64 | macos-latest, macos-14, macos-15, macos-26 |

標準 GitHub ホストランナーに加えて、GitHub は GitHub Team および GitHub Enterprise Cloud プランのお客様に、高度な機能を備えた各種の管理された仮想マシンを提供しています。たとえば、より多くのコアとディスク容量、GPU 搭載マシン、ARM 搭載マシンなどです。詳しくは、「大規模ランナー」を参照してください。

> **メモ**
>
> `-latest` ランナーイメージは GitHub が提供する最新の安定版イメージであり、オペレーティングシステムのベンダーから入手可能なオペレーティングシステムの最新バージョンではない場合があります。

> **警告**
>
> ベータ版および非推奨のイメージは、「現状有姿」、「すべての不具合を含む」および「提供可能な状態」で提供され、サービスレベル契約および保証の対象外です。ベータ版イメージはカスタマーサポートの対象外となる場合があります。

### 例: オペレーティングシステムを指定する

```yaml
runs-on: ubuntu-latest
```

詳しくは、「GitHub ホストランナー」を参照してください。

### セルフホストランナーを選択する

ジョブにセルフホストランナーを指定するには、ワークフローファイルで `runs-on` をセルフホストランナーのラベルで構成します。

セルフホストランナーには `self-hosted` ラベルが付いている場合があります。セルフホストランナーを設定すると、既定では `self-hosted` ラベルが含まれます。`self-hosted` ラベルが適用されないようにするには、`--no-default-labels` フラグを渡すことができます。ラベルを使用すると、オペレーティングシステムやアーキテクチャなど、ランナーのターゲット指定オプションを作成できます。`self-hosted` で始まり（これは先頭に記載する必要があります）、必要に応じて追加のラベルを含めるラベル配列を指定することをおすすめします。ラベルの配列を指定すると、指定したすべてのラベルを持つランナーにジョブがキューに入れられます。

> **メモ**
>
> Actions Runner Controller は `self-hosted` ラベルをサポートしていません。

### 例: ランナー選択にラベルを使用する

```yaml
runs-on: [self-hosted, linux]
```

詳しくは、「セルフホストランナー」および「ワークフローでセルフホストランナーを使用する」を参照してください。

### グループ内のランナーを選択する

`runs-on` を使用してランナーグループを対象にできるため、ジョブはそのグループのメンバーである任意のランナーで実行されます。より細かく制御するには、ランナーグループとラベルを組み合わせることもできます。

ランナーグループには、大規模ランナーまたはセルフホストランナーのみをメンバーとして含めることができます。

### 例: ジョブが実行される場所を制御するためにグループを使用する

この例では、Ubuntu ランナーが `ubuntu-runners` というグループに追加されています。`runs-on` キーは、ジョブを `ubuntu-runners` グループ内の利用可能な任意のランナーに送信します。

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

### 例: グループとラベルを組み合わせる

グループとラベルを組み合わせる場合、ジョブを実行する資格を得るには、ランナーが両方の要件を満たす必要があります。

この例では、`ubuntu-runners` というランナーグループに Ubuntu ランナーが含まれており、これらには `ubuntu-24.04-16core` ラベルも割り当てられています。`runs-on` キーは `group` と `labels` を組み合わせているため、ジョブは、グループ内にあり、一致するラベルも持つ利用可能な任意のランナーにルーティングされます。

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

## `jobs.<job_id>.snapshot`

`jobs.<job_id>.snapshot` を使用すると、カスタムイメージを生成できます。

「カスタムイメージの生成」に示されているように、文字列構文またはマッピング構文のいずれかを使用して、ジョブに `snapshot` キーワードを追加します。

`snapshot` キーワードを含む各ジョブは、個別のイメージを作成します。1 つのイメージまたはイメージバージョンだけを生成するには、すべてのワークフローステップを 1 つのジョブに含めます。`snapshot` キーワードを含むジョブが正常に実行されるたびに、そのイメージの新しいバージョンが作成されます。

詳しくは、「カスタムイメージの使用」を参照してください。

## `jobs.<job_id>.environment`

`jobs.<job_id>.environment` を使用して、ジョブが参照する環境を定義します。

環境は、環境名のみとして指定することも、`name` と `url` を持つ環境オブジェクトとして指定することもできます。この URL は、deployments API の `environment_url` に対応します。deployments API について詳しくは、「リポジトリ用 REST API エンドポイント」を参照してください。

> **メモ**
>
> 環境を参照するジョブがランナーに送信される前に、すべてのデプロイ保護ルールに合格する必要があります。詳しくは、「デプロイ用環境の管理」を参照してください。

### 例: 単一の環境名を使用する

```yaml
environment: staging_environment
```

### 例: 環境名と URL を使用する

```yaml
environment:
  name: production_environment
  url: https://github.com
```

`url` の値には式を使用できます。使用できる式コンテキスト: `github`、`inputs`、`vars`、`needs`、`strategy`、`matrix`、`job`、`runner`、`env`、`steps`。式について詳しくは、「ワークフローとアクションで式を評価する」を参照してください。

### 例: 出力を URL として使用する

```yaml
environment:
  name: production_environment
  url: ${{ steps.step_id.outputs.url_output }}
```

`name` の値には式を使用できます。使用できる式コンテキスト: `github`、`inputs`、`vars`、`needs`、`strategy`、`matrix`。式について詳しくは、「ワークフローとアクションで式を評価する」を参照してください。

### 例: 式を環境名として使用する

```yaml
environment:
  name: ${{ github.ref_name }}
```

### 例: デプロイを作成せずに環境を使用する

デプロイオブジェクトを作成せずに環境のシークレットと変数を使用するには、`deployment` を `false` に設定します。

```yaml
environment:
  name: testing
  deployment: false
```

`deployment: false` の設定は、カスタムデプロイ保護ルールと互換性がありません。詳しくは、「GitHub Actions を使用したデプロイ」を参照してください。

## `jobs.<job_id>.concurrency`

`jobs.<job_id>.concurrency` を使用すると、同じ同時実行グループを使用するジョブまたはワークフローが、一度に 1 つだけ実行されるようにできます。同時実行グループには、任意の文字列または式を指定できます。使用できる式コンテキスト: `github`、`inputs`、`vars`、`needs`、`strategy`、`matrix`。式について詳しくは、「ワークフローとアクションで式を評価する」を参照してください。

ワークフローレベルで同時実行を指定することもできます。詳しくは、`concurrency` を参照してください。

これは、同時実行グループ内で、実行中のジョブまたはワークフローが常に最大 1 つだけになることを意味します。同時実行するジョブまたはワークフローがキューに入ったとき、リポジトリ内で同じ同時実行グループを使用する別のジョブまたはワークフローが進行中の場合、キューに入ったジョブまたはワークフローは保留中になります。既定では、同じ同時実行グループ内に既存の保留中ジョブまたはワークフローがある場合、それはキャンセルされ、新しくキューに入ったジョブまたはワークフローが代わりに入ります。

同じ同時実行グループ内で現在実行中のジョブまたはワークフローもキャンセルするには、`cancel-in-progress: true` を指定します。同じ同時実行グループ内で現在実行中のジョブまたはワークフローを条件付きでキャンセルするには、使用可能な任意の式コンテキストを使って `cancel-in-progress` を式として指定できます。

同じ同時実行グループ内で、複数の保留中のジョブまたはワークフロー実行を待機できるようにするには、省略可能な `queue` プロパティを使用します。`queue` プロパティは、次の値を受け付けます。

`single`（既定）: 同時実行グループ内で保留中にできるジョブまたはワークフロー実行は最大 1 つです。新しいジョブまたはワークフロー実行がキューに入ると、同じグループ内の既存の保留中ジョブまたはワークフロー実行はキャンセルされ、置き換えられます。
`max`: 同時実行グループ内で最大 100 個のジョブまたはワークフロー実行を保留中にできます。キューが満杯になると、追加のジョブまたはワークフロー実行はキャンセルされます。
`queue: max` と `cancel-in-progress: true` の組み合わせは許可されておらず、ワークフロー検証エラーになります。

> **メモ**
>
> 同時実行グループ名では、大文字と小文字は区別されません。たとえば、`prod` と `Prod` は同じ同時実行グループとして扱われます。
> 同じ同時実行グループ内のジョブまたはワークフロー実行は、各ワークフローがディスパッチされた時刻ではなく、それぞれが同時実行グループで待機を開始した時刻に従って、先入れ先出し（FIFO）順に処理されます。ジョブまたは実行の実際の開始時刻は変動する可能性があるため、順序は保証されません。

### 例: 同時実行と既定の動作を使用する

GitHub Actions の既定の動作では、複数のジョブまたはワークフロー実行を同時に実行できます。`concurrency` キーワードを使用すると、ワークフロー実行の同時実行を制御できます。

たとえば、トリガー条件が定義されている場所の直後に `concurrency` キーワードを使用して、特定のブランチに対するワークフロー実行全体の同時実行を制限できます。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

ジョブレベルで `concurrency` キーワードを使用して、ワークフロー内のジョブの同時実行を制限することもできます。

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

### 例: 同時実行グループ

同時実行グループは、同じ同時実行キーを共有するワークフロー実行またはジョブの実行を管理し、制限する方法を提供します。

同時実行キーは、ワークフローまたはジョブを同時実行グループにまとめるために使用されます。同時実行キーを定義すると、GitHub Actions は、そのキーを持つワークフローまたはジョブが常に 1 つだけ実行されるようにします。同じ同時実行キーで新しいワークフロー実行またはジョブが開始されると、GitHub Actions は、そのキーですでに実行中のワークフローまたはジョブをキャンセルします。同時実行キーには、ハードコーディングされた文字列、またはコンテキスト変数を含む動的な式を指定できます。

ワークフロー内で同時実行条件を定義して、ワークフローまたはジョブが同時実行グループの一部になるようにできます。

これは、ワークフロー実行またはジョブが開始されると、GitHub が同じ同時実行グループ内ですでに進行中のワークフロー実行またはジョブをキャンセルすることを意味します。これは、ステージング環境へのデプロイに使用されるものなど、特定の一連のワークフローまたはジョブについて並列実行を防ぎ、競合を引き起こしたり必要以上にリソースを消費したりする可能性のあるアクションを防止したいシナリオで役立ちます。

この例では、`job-1` は `staging_environment` という名前の同時実行グループの一部です。つまり、`job-1` の新しい実行がトリガーされると、`staging_environment` 同時実行グループ内ですでに進行中の同じジョブの実行はすべてキャンセルされます。

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: staging_environment
      cancel-in-progress: true
```

または、ワークフローで `concurrency: ci-${{ github.ref }}` のような動的な式を使用すると、ワークフローまたはジョブは、ワークフローをトリガーしたブランチまたはタグの参照が `ci-` に続く名前の同時実行グループの一部になります。この例では、前の実行がまだ進行中の間に新しいコミットが `main` ブランチにプッシュされると、前の実行はキャンセルされ、新しい実行が開始されます。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

### 例: 複数の保留中実行をキューに入れる

既定では、同時実行グループ内で保留中にできるジョブまたはワークフロー実行は、一度に 1 つだけです。キャンセルする代わりに複数の実行をキューに入れられるようにするには、`queue: max` を設定します。`queue: max` では、最大 100 個のジョブまたはワークフロー実行を同時実行グループ内で待機させることができます。キューが満杯になると、追加の実行はキャンセルされます。

たとえば、次のワークフローは本番環境へのデプロイをキューに入れ、各実行が同時実行グループで待機を開始した時刻に基づく順序で、1 つずつ処理します。

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: production-deploy
  queue: max
```

`queue: max` は `cancel-in-progress: true` と組み合わせることができない点に注意してください。この 2 つのオプションは、進行中の実行の処理について矛盾する動作を表すためです。

### 例: 同時実行を使用して進行中のジョブまたは実行をキャンセルする

GitHub Actions で同時実行を使用して進行中のジョブまたは実行をキャンセルするには、`cancel-in-progress` オプションを `true` に設定して `concurrency` キーを使用できます。

```yaml
concurrency:
  group: ${{ github.ref }}
  cancel-in-progress: true
```

この例では、特定の同時実行グループを定義していない場合、GitHub Actions はそのジョブまたはワークフローの進行中の実行をすべてキャンセルする点に注意してください。

### 例: フォールバック値を使用する

特定のイベントに対してのみ定義されるプロパティを使ってグループ名を作成する場合は、フォールバック値を使用できます。たとえば、`github.head_ref` は `pull_request` イベントでのみ定義されます。ワークフローが `pull_request` イベントに加えて他のイベントにも応答する場合は、構文エラーを回避するためにフォールバックを指定する必要があります。次の同時実行グループは、`pull_request` イベントでのみ進行中のジョブまたは実行をキャンセルします。`github.head_ref` が未定義の場合、同時実行グループは実行 ID にフォールバックします。実行 ID は、その実行について一意であり、定義されていることが保証されています。

```yaml
concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
```

### 例: 現在のワークフローの進行中ジョブまたは実行のみをキャンセルする

同じリポジトリに複数のワークフローがある場合、他のワークフローの進行中ジョブまたは実行をキャンセルしないようにするには、同時実行グループ名はワークフロー間で一意である必要があります。そうしないと、ワークフローに関係なく、以前に進行中または保留中だったジョブはすべてキャンセルされます。

同じワークフローの進行中実行のみをキャンセルするには、`github.workflow` プロパティを使用して同時実行グループを作成できます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### 例: 特定のブランチでのみ進行中ジョブをキャンセルする

特定のブランチでは進行中ジョブをキャンセルし、他のブランチではキャンセルしたくない場合は、`cancel-in-progress` で条件式を使用できます。たとえば、開発ブランチでは進行中ジョブをキャンセルし、リリースブランチではキャンセルしたくない場合に、このようにできます。

リリースブランチで実行されていない場合にのみ同じワークフローの進行中実行をキャンセルするには、次のような式を `cancel-in-progress` に設定できます。

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !contains(github.ref, 'release/')}}
```

この例では、`release/1.2.3` ブランチへの複数のプッシュは、進行中の実行をキャンセルしません。`main` など別のブランチへのプッシュは、進行中の実行をキャンセルします。

## `jobs.<job_id>.outputs`

`jobs.<job_id>.outputs` を使用して、ジョブの出力のマップを作成できます。ジョブの出力は、このジョブに依存するすべての下流ジョブで利用できます。ジョブの依存関係の定義について詳しくは、`jobs.<job_id>.needs` を参照してください。

出力は、ジョブごとに最大 1 MB です。ワークフロー実行内のすべての出力の合計は、最大 50 MB です。サイズは UTF-16 エンコードに基づいて概算されます。

式を含むジョブ出力は、各ジョブの終了時にランナー上で評価されます。シークレットを含む出力はランナー上で秘匿化され、GitHub Actions には送信されません。

出力がシークレットを含む可能性があるためスキップされた場合、次の警告メッセージが表示されます: "Skip output {output.Key} since it may contain secret." シークレットの扱い方について詳しくは、「例: ジョブまたはワークフロー間でシークレットをマスクして渡す」を参照してください。

依存ジョブでジョブ出力を使用するには、`needs` コンテキストを使用できます。詳しくは、「コンテキストのリファレンス」を参照してください。

### 例: ジョブの出力を定義する

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

### マトリックスジョブでジョブ出力を使用する

マトリックスを使用すると、異なる名前の複数の出力を生成できます。マトリックスを使用する場合、ジョブ出力はマトリックス内のすべてのジョブから結合されます。

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
> Actions は、マトリックスジョブが実行される順序を保証しません。出力名が一意であることを確認してください。一意でない場合、最後に実行されたマトリックスジョブが出力値を上書きします。

## `jobs.<job_id>.env`

ジョブ内のすべてのステップで使用できる変数のマップです。ワークフロー全体または個々のステップに変数を設定できます。詳しくは、`env` と `jobs.<job_id>.steps[*].env` を参照してください。

同じ名前の環境変数が複数定義されている場合、GitHub は最も具体的な変数を使用します。たとえば、ステップで定義された環境変数は、そのステップの実行中、同じ名前のジョブおよびワークフローの環境変数をオーバーライドします。ジョブに定義された環境変数は、そのジョブの実行中、同じ名前のワークフロー変数をオーバーライドします。

### `jobs.<job_id>.env` の例

```yaml
jobs:
  job1:
    env:
      FIRST_NAME: Mona
```

## `jobs.<job_id>.defaults`

`jobs.<job_id>.defaults` を使用して、ジョブ内のすべてのステップに適用されるデフォルト設定のマップを作成します。ワークフロー全体にデフォルト設定を設定することもできます。詳しくは、`defaults` を参照してください。

同じ名前のデフォルト設定が複数定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで定義された同じ名前のデフォルト設定をオーバーライドします。

## `jobs.<job_id>.defaults.run`

`jobs.<job_id>.defaults.run` を使用して、ジョブ内のすべての `run` ステップに既定のシェルと作業ディレクトリを指定します。

ジョブ内のすべての `run` ステップに、既定のシェルと作業ディレクトリのオプションを指定できます。ワークフロー全体の `run` にデフォルト設定を設定することもできます。詳しくは、`defaults.run` を参照してください。

これらは、`jobs.<job_id>.defaults.run` レベルおよび `jobs.<job_id>.steps[*].run` レベルでオーバーライドできます。

同じ名前のデフォルト設定が複数定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで定義された同じ名前のデフォルト設定をオーバーライドします。

## `jobs.<job_id>.defaults.run.shell`

`shell` を使用して、ステップのシェルを定義します。このキーワードは、複数のコンテキストを参照できます。詳しくは、「コンテキスト」を参照してください。

| サポートされるプラットフォーム | シェルパラメーター | 説明 | 内部で実行されるコマンド |
| --- | --- | --- | --- |
| Linux / macOS | unspecified | Windows 以外のプラットフォームでの既定のシェルです。`bash` を明示的に指定した場合とは異なるコマンドが実行される点に注意してください。パスに `bash` が見つからない場合、これは `sh` として扱われます。 | bash -e {0} |
| All | bash | `sh` へのフォールバックを備えた、Windows 以外のプラットフォームでの既定のシェルです。Windows で `bash` シェルを指定すると、Git for Windows に含まれる `bash` シェルが使用されます。 | bash --noprofile --norc -eo pipefail {0} |
| All | pwsh | PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | pwsh -command ". '{0}'" |
| All | python | `python` コマンドを実行します。 | python {0} |
| Linux / macOS | sh | シェルが指定されておらず、パスに `bash` が見つからない場合の、Windows 以外のプラットフォームでのフォールバック動作です。 | sh -e {0} |
| Windows | cmd | GitHub はスクリプト名に拡張子 `.cmd` を追加し、`{0}` を置換します。 | %ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}"". |
| Windows | pwsh | これは Windows で使用される既定のシェルです。PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。セルフホスト Windows ランナーに PowerShell Core がインストールされていない場合は、代わりに PowerShell Desktop が使用されます。 | pwsh -command ". '{0}'". |
| Windows | powershell | PowerShell Desktop です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | powershell -command ". '{0}'". |

同じ名前のデフォルト設定が複数定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで定義された同じ名前のデフォルト設定をオーバーライドします。

## `jobs.<job_id>.defaults.run.working-directory`

`working-directory` を使用して、ステップのシェルの作業ディレクトリを定義します。このキーワードは、複数のコンテキストを参照できます。詳しくは、「コンテキスト」を参照してください。

> **ヒント**
>
> 割り当てる `working-directory` が、そこでシェルを実行する前にランナー上に存在していることを確認してください。同じ名前のデフォルト設定が複数定義されている場合、GitHub は最も具体的なデフォルト設定を使用します。たとえば、ジョブで定義されたデフォルト設定は、ワークフローで定義された同じ名前のデフォルト設定をオーバーライドします。

### 例: ジョブの既定の run ステップオプションを設定する

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: ./scripts
```

## `jobs.<job_id>.steps`

ジョブには、ステップと呼ばれる一連のタスクが含まれます。ステップでは、コマンドの実行、セットアップ タスクの実行、リポジトリ内のアクション、パブリック リポジトリ内のアクション、または Docker レジストリに公開されたアクションの実行ができます。すべてのステップがアクションを実行するわけではありませんが、すべてのアクションはステップとして実行されます。各ステップはランナー環境内の独自プロセスで実行され、ワークスペースとファイルシステムにアクセスできます。ステップは独自プロセスで実行されるため、環境変数への変更はステップ間で保持されません。GitHub には、ジョブを設定して完了するための組み込みステップが用意されています。

GitHub に表示されるチェックは最初の 1,000 件だけですが、ワークフローの使用制限内であれば、ステップ数に上限はありません。詳しくは GitHub ホステッド ランナーの課金と使用量、およびセルフホステッド ランナーの使用制限に関する Actions の制限を参照してください。

### `jobs.<job_id>.steps` の例

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

## `jobs.<job_id>.steps[*].id`

ステップの一意の識別子です。`id` を使って、コンテキスト内でステップを参照できます。詳しくはコンテキストのリファレンスを参照してください。

## `jobs.<job_id>.steps[*].if`

`if` 条件を使うと、条件が満たされない限りステップが実行されないようにできます。サポートされている任意のコンテキストと式を使って条件を作成できます。このキーでサポートされているコンテキストについて詳しくは、コンテキストのリファレンスを参照してください。

`if` 条件内で式を使う場合、GitHub Actions は `if` 条件を自動的に式として評価するため、必要に応じて `${{ }}` 式構文を省略できます。ただし、この例外はすべての場所に適用されるわけではありません。

式が `!` で始まる場合、`!` は YAML 形式の予約表記であるため、必ず `${{ }}` 式構文を使うか、`''`、`""`、または `()` でエスケープする必要があります。例:

```yaml
if: ${{ ! startsWith(github.ref, 'refs/tags/') }}
```

詳しくは、ワークフローとアクションで式を評価する方法を参照してください。

### コンテキストを使う例

このステップは、イベントの種類が `pull_request` で、イベント アクションが `unassigned` の場合にのみ実行されます。

```yaml
steps:
  - name: My first step
    if: ${{ github.event_name == 'pull_request' && github.event.action == 'unassigned' }}
    run: echo This event is a pull request that had an assignee removed.
```

### ステータス チェック関数を使う例

`my backup step` は、ジョブの前のステップが失敗した場合にのみ実行されます。詳しくは、ワークフローとアクションで式を評価する方法を参照してください。

```yaml
steps:
  - name: My first step
    uses: octo-org/action-name@main
  - name: My backup step
    if: ${{ failure() }}
    uses: actions/heroku@1.0.0
```

### シークレットを使う例

シークレットは `if:` 条件内で直接参照できません。代わりに、シークレットをジョブ レベルの環境変数として設定し、その環境変数を参照してジョブ内のステップを条件付きで実行することを検討してください。

シークレットが設定されていない場合、そのシークレットを参照する式（例の `${{ secrets.SuperSecret }}` など）の戻り値は空文字列になります。

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

詳しくは、コンテキストのリファレンスと GitHub Actions でのシークレットの使用を参照してください。

## `jobs.<job_id>.steps[*].name`

GitHub に表示されるステップの名前です。

## `jobs.<job_id>.steps[*].uses`

ジョブ内のステップの一部として実行するアクションを選択します。アクションとは、再利用可能なコード単位です。ワークフローと同じリポジトリで定義されたアクション、パブリック リポジトリのアクション、または公開済みの Docker コンテナ イメージ内のアクションを使用できます。

Git ref、SHA、または Docker タグを指定して、使用するアクションのバージョンを含めることを強くおすすめします。バージョンを指定しないと、アクションの所有者が更新を公開したときに、ワークフローが壊れたり、予期しない動作が発生したりする可能性があります。

リリース済みアクション バージョンのコミット SHA を使うことが、安定性とセキュリティの面で最も安全です。
アクションがメジャー バージョン タグを公開している場合は、互換性を維持したまま重要な修正とセキュリティ パッチを受け取れることが期待できます。ただし、この動作はアクションの作者の裁量に委ねられます。
アクションの既定ブランチを使うのは便利な場合がありますが、破壊的変更を含む新しいメジャー バージョンがリリースされると、ワークフローが壊れる可能性があります。
一部のアクションでは、`with` キーワードを使って設定する必要がある入力が求められます。必要な入力を確認するには、アクションの README ファイルを確認してください。

アクションは JavaScript ファイルまたは Docker コンテナのいずれかです。使用しているアクションが Docker コンテナの場合は、Linux 環境でジョブを実行する必要があります。詳しくは `runs-on` を参照してください。

### バージョン指定されたアクションを使う例

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

### パブリック アクションを使う例

```yaml
{owner}/{repo}@{ref}
```

パブリック GitHub リポジトリ内のブランチ、ref、または SHA を指定できます。

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

### サブディレクトリ内のパブリック アクションを使う例

```yaml
{owner}/{repo}/{path}@{ref}
```

パブリック GitHub リポジトリ内の特定のブランチ、ref、または SHA にあるサブディレクトリです。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/aws/ec2@main
```

### ワークフローと同じリポジトリ内のアクションを使う例

```yaml
./path/to/dir
```

ワークフローのリポジトリ内で、アクションを含むディレクトリへのパスです。アクションを使用する前に、リポジトリをチェックアウトする必要があります。

リポジトリ ファイル構造の例:

```text
|-- hello-world (repository)
|   |__ .github
|       └── workflows
|           └── my-first-workflow.yml
|       └── actions
|           |__ hello-world-action
|               └── action.yml
```

このパスは、既定の作業ディレクトリ（`github.workspace`、`$GITHUB_WORKSPACE`）からの相対パス（`./`）です。アクションがリポジトリをワークフローとは異なる場所にチェックアウトする場合、ローカル アクションに使用する相対パスを更新する必要があります。

ワークフロー ファイルの例:

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

### Docker Hub アクションを使う例

```yaml
docker://{image}:{tag}
```

Docker Hub に公開されている Docker イメージです。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://alpine:3.8
```

### GitHub Packages Container registry を使う例

```yaml
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

### Docker パブリック レジストリ アクションを使う例

```yaml
docker://{host}/{image}:{tag}
```

パブリック レジストリ内の Docker イメージです。この例では、`gcr.io` の Google Container Registry を使用します。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://gcr.io/cloud-builders/gradle
```

### ワークフローとは別のプライベート リポジトリ内のアクションを使う例

アクションが内部リポジトリにある場合、またはワークフローのリポジトリからのアクセスを許可するように構成されたプライベート リポジトリにある場合は、そのアクションを直接参照できます。詳しくは、リポジトリの GitHub Actions 設定の管理、およびリポジトリの GitHub Actions 設定の管理を参照してください。

アクションがアクセスを許可するように構成されたリポジトリ内にない場合は、リポジトリをチェックアウトして、ローカルでアクションを参照する必要があります。個人用アクセス トークンを生成し、そのトークンをシークレットとして追加します。次の例は、アクションを参照するためのこの方法を示しています。詳しくは、個人用アクセス トークンの管理、および GitHub Actions でのシークレットの使用を参照してください。

例の `PERSONAL_ACCESS_TOKEN` は、使用するシークレットの名前に置き換えてください。

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

または、個人用アクセス トークンの所有者が離脱した場合でもワークフローが実行され続けるように、個人用アクセス トークンの代わりに GitHub App を使用してください。詳しくは、GitHub Actions ワークフローで GitHub App を使って認証済み API 要求を行う方法を参照してください。

## `jobs.<job_id>.steps[*].run`

オペレーティング システムのシェルを使って、21,000 文字を超えないコマンド ライン プログラムを実行します。名前を指定しない場合、ステップ名は `run` コマンドで指定されたテキストが既定になります。

既定では、コマンドは非ログイン シェルを使って実行されます。別のシェルを選択し、コマンドの実行に使うシェルをカスタマイズできます。詳しくは `jobs.<job_id>.steps[*].shell` を参照してください。

各 `run` キーワードは、ランナー環境内の新しいプロセスとシェルを表します。複数行のコマンドを指定すると、各行は同じシェルで実行されます。例:

1 行のコマンド:

```yaml
- name: Install Dependencies
  run: npm install
```

複数行のコマンド:

```yaml
- name: Clean install dependencies and build
  run: |
    npm ci
    npm run build
```

## `jobs.<job_id>.steps[*].working-directory`

`working-directory` キーワードを使うと、コマンドを実行する作業ディレクトリを指定できます。

```yaml
- name: Clean temp directory
  run: rm -rf *
  working-directory: ./temp
```

または、ジョブ内のすべての `run` ステップ、もしくはワークフロー全体のすべての `run` ステップに対して、既定の作業ディレクトリを指定できます。詳しくは `defaults.run.working-directory` と `jobs.<job_id>.defaults.run.working-directory` を参照してください。

`run` ステップを使ってスクリプトを実行することもできます。詳しくは、ワークフローへのスクリプトの追加を参照してください。

## `jobs.<job_id>.steps[*].shell`

`shell` キーワードを使うと、ランナーのオペレーティング システムとジョブの既定のシェル設定を上書きできます。組み込みのシェル キーワードを使うことも、独自のシェル オプション セットを定義することもできます。内部で実行されるシェル コマンドは、`run` キーワードで指定されたコマンドを含む一時ファイルを実行します。

| サポートされるプラットフォーム | シェル パラメーター | 説明 | 内部で実行されるコマンド |
| --- | --- | --- | --- |
| Linux / macOS | unspecified | Windows 以外のプラットフォームにおける既定のシェルです。これは `bash` を明示的に指定した場合とは異なるコマンドを実行する点に注意してください。パス内に `bash` が見つからない場合、これは `sh` として扱われます。 | bash -e {0} |
| All | bash | Windows 以外のプラットフォームでの既定のシェルで、`sh` へのフォールバックがあります。Windows で `bash` シェルを指定すると、Git for Windows に含まれる `bash` シェルが使用されます。 | bash --noprofile --norc -eo pipefail {0} |
| All | pwsh | PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | pwsh -command ". '{0}'" |
| All | python | `python` コマンドを実行します。 | python {0} |
| Linux / macOS | sh | シェルが指定されておらず、パス内に `bash` が見つからない場合の、Windows 以外のプラットフォームにおけるフォールバック動作です。 | sh -e {0} |
| Windows | cmd | GitHub はスクリプト名に拡張子 `.cmd` を追加し、`{0}` に代入します。 | %ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}"". |
| Windows | pwsh | これは Windows で使用される既定のシェルです。PowerShell Core です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。セルフホステッド Windows ランナーに PowerShell Core がインストールされていない場合は、代わりに PowerShell Desktop が使用されます。 | pwsh -command ". '{0}'". |
| Windows | powershell | PowerShell Desktop です。GitHub はスクリプト名に拡張子 `.ps1` を追加します。 | powershell -command ". '{0}'". |

または、ジョブ内のすべての `run` ステップ、もしくはワークフロー全体のすべての `run` ステップに対して、既定のシェルを指定できます。詳しくは `defaults.run.shell` と `jobs.<job_id>.defaults.run.shell` を参照してください。

### Bash を使ってコマンドを実行する例

```yaml
steps:
  - name: Display the path
    shell: bash
    run: echo $PATH
```

### Windows cmd を使ってコマンドを実行する例

```yaml
steps:
  - name: Display the path
    shell: cmd
    run: echo %PATH%
```

### PowerShell Core を使ってコマンドを実行する例

```yaml
steps:
  - name: Display the path
    shell: pwsh
    run: echo ${env:PATH}
```

### PowerShell Desktop を使ってコマンドを実行する例

```yaml
steps:
  - name: Display the path
    shell: powershell
    run: echo ${env:PATH}
```

### インライン Python スクリプトを実行する例

```yaml
steps:
  - name: Display the path
    shell: python
    run: |
      import os
      print(os.environ['PATH'])
```

### カスタム シェル

`shell` の値は、`command [options] {0} [more_options]` を使ったテンプレート文字列に設定できます。GitHub は文字列のうち空白で区切られた最初の単語をコマンドとして解釈し、`{0}` の位置に一時スクリプトのファイル名を挿入します。

例:

```yaml
steps:
  - name: Display the environment variables and their values
    shell: perl {0}
    run: |
      print %ENV
```

この例の `perl` のように、使用されるコマンドはランナーにインストールされている必要があります。

GitHub ホステッド ランナーに含まれるソフトウェアについては、GitHub ホステッド ランナーを参照してください。

### 終了コードとエラー アクション設定

組み込みのシェル キーワードについては、GitHub ホステッド ランナーによって実行される次の既定値を提供しています。シェル スクリプトを実行するときは、これらのガイドラインを使用してください。

**`bash`/`sh`**

- 既定では、`sh` と `bash` の両方で `set -e` を使ってフェイルファスト動作が適用されます。`shell: bash` を指定した場合は、ゼロ以外の終了ステータスを生成するパイプラインから早期終了するように、`-o pipefail` も適用されます。
- シェル オプションにテンプレート文字列を指定することで、シェル パラメーターを完全に制御できます。たとえば、`bash {0}` です。
- `sh` に似たシェルは、スクリプトで最後に実行されたコマンドの終了コードで終了します。これはアクションの既定の動作でもあります。ランナーは、この終了コードに基づいてステップの状態を失敗または成功として報告します。

**`powershell`/`pwsh`**

- 可能な場合はフェイルファスト動作になります。`pwsh` と `powershell` の組み込みシェルでは、スクリプト内容の先頭に `$ErrorActionPreference = 'stop'` を追加します。
- アクションの状態にスクリプトの最後の終了コードが反映されるように、PowerShell スクリプトには `if ((Test-Path -LiteralPath variable:\LASTEXITCODE)) { exit $LASTEXITCODE }` を追加します。
- 必要に応じて、組み込みシェルを使わずに、`pwsh -File {0}` や `powershell -Command "& '{0}'"` のようなカスタム シェル オプションを指定することで、いつでもこの動作を無効にできます。

**`cmd`**

- 各エラー コードを確認して適切に応答するようにスクリプトを記述する以外に、フェイルファスト動作を完全に有効にする方法はないようです。既定ではその動作を実際に提供できないため、この動作をスクリプトに記述する必要があります。
- `cmd.exe` は最後に実行したプログラムのエラー レベルで終了し、そのエラー コードをランナーに返します。この動作は、以前の `sh` および `pwsh` の既定の動作と内部的に一貫しており、`cmd.exe` の既定でもあるため、この動作はそのまま維持されます。

## `jobs.<job_id>.steps[*].with`

アクションで定義された入力パラメーターのマップです。各入力パラメーターはキーと値のペアです。入力パラメーターは環境変数として設定されます。変数には `INPUT_` というプレフィックスが付き、大文字に変換されます。

Docker コンテナ用に定義された入力パラメーターは、`args` を使う必要があります。詳しくは `jobs.<job_id>.steps[*].with.args` を参照してください。

### `jobs.<job_id>.steps[*].with` の例

`hello_world` アクションで定義された 3 つの入力パラメーター（`first_name`、`middle_name`、`last_name`）を定義します。これらの入力変数は、`INPUT_FIRST_NAME`、`INPUT_MIDDLE_NAME`、`INPUT_LAST_NAME` 環境変数として `hello-world` アクションからアクセスできます。

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

## `jobs.<job_id>.steps[*].with.args`

Docker コンテナの入力を定義する文字列です。GitHub は、コンテナの起動時に `args` をコンテナの `ENTRYPOINT` に渡します。このパラメーターでは文字列の配列はサポートされていません。空白を含む単一の引数は、二重引用符 `""` で囲む必要があります。

### `jobs.<job_id>.steps[*].with.args` の例

```yaml
steps:
  - name: Explain why this job ran
    uses: octo-org/action-name@main
    with:
      entrypoint: /bin/echo
      args: The ${{ github.event_name }} event triggered this step.
```

`args` は Dockerfile の `CMD` 命令の代わりに使われます。Dockerfile で `CMD` を使う場合は、優先順に並べた次のガイドラインを使用してください。

- アクションの README に必須の引数を記載し、`CMD` 命令からは省略します。
- `args` を指定しなくてもアクションを使用できる既定値を使います。
- アクションが `--help` フラグまたは類似のものを公開している場合は、アクションが自己文書化されるように、それを既定値として使います。

## `jobs.<job_id>.steps[*].with.entrypoint`

Dockerfile の Docker `ENTRYPOINT` を上書きします。または、まだ指定されていない場合は設定します。シェル形式と exec 形式を持つ Docker `ENTRYPOINT` 命令とは異なり、`entrypoint` キーワードは、実行する実行可能ファイルを定義する単一の文字列だけを受け入れます。

### `jobs.<job_id>.steps[*].with.entrypoint` の例

```yaml
steps:
  - name: Run a custom command
    uses: octo-org/action-name@main
    with:
      entrypoint: /a/different/executable
```

`entrypoint` キーワードは Docker コンテナ アクションで使うことを意図していますが、入力を定義していない JavaScript アクションでも使用できます。

## `jobs.<job_id>.steps[*].env`

ランナー環境でステップが使用する変数を設定します。ワークフロー全体またはジョブに変数を設定することもできます。詳しくは `env` と `jobs.<job_id>.env` を参照してください。

同じ名前の環境変数が複数定義されている場合、GitHub は最も限定的な変数を使用します。たとえば、ステップで定義された環境変数は、そのステップの実行中、同じ名前のジョブおよびワークフローの環境変数を上書きします。ジョブに定義された環境変数は、そのジョブの実行中、同じ名前のワークフロー変数を上書きします。

公開アクションでは、README ファイルで想定される変数が指定されている場合があります。パスワードやトークンなどのシークレットまたは機密値を設定する場合は、`secrets` コンテキストを使用してシークレットを設定する必要があります。詳しくはコンテキストのリファレンスを参照してください。

### `jobs.<job_id>.steps[*].env` の例

```yaml
steps:
  - name: My first action
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      FIRST_NAME: Mona
      LAST_NAME: Octocat
```

## `jobs.<job_id>.steps[*].continue-on-error`

ステップが失敗したときにジョブが失敗するのを防ぎます。このステップが失敗してもジョブを成功として扱えるようにするには、`true` に設定します。

## `jobs.<job_id>.steps[*].timeout-minutes`

プロセスを強制終了するまでにステップを実行できる最大分数です。最大値: GitHub ホストランナーとセルフホストランナーのどちらも 360 です。

小数値はサポートされていません。`timeout-minutes` は正の整数である必要があります。

## `jobs.job_id.steps[].background`

ステップを非同期で実行し、ジョブがその完了を待たずに次のステップへ進むようにします。データベース、サーバー、監視タスクなど、ほかのステップと並行して実行する必要がある長時間実行プロセスには `background true` を使用します。後で `wait` または `wait-all` を使用してバックグラウンドステップと同期するか、`cancel` で停止します。

`run` または `uses` を使用するステップで `background` を使用できます。`wait` または `cancel` からバックグラウンドステップを参照するには、`id` を指定します。1 つのジョブで同時に実行できるバックグラウンドステップは最大 10 個です。追加のバックグラウンドステップは、空き枠ができるまでキューに入れられます。

バックグラウンドステップからの出力と環境の変更は、そのステップを含む `wait` または `wait-all` ステップを実行した後にのみ使用できます。バックグラウンドステップが失敗した場合、そのステップを含む次の `wait` または `wait-all` でジョブが失敗します（そのステップに `continue-on-error` が設定されている場合を除きます）。ジョブ後のクリーンアップの前には、暗黙的な `wait-all` が実行されます。

後続のステップの実行中に稼働し続けるサーバーやデータベースのような長時間実行プロセスを開始する細かな制御、`wait` または `cancel` による特定のステップの参照、またはバックグラウンド作業とほかのステップの交互実行が必要な場合は、`background` を使用します。一方、ジョブが続行する前にすべて完了すべき自己完結したステップのグループがある場合は、`parallel` の方が便利な省略表現です。

> **メモ**
>
> コンポジットアクション内のステップでは `background` を使用できません。コンポジットアクション自体はバックグラウンドステップとして実行できますが、その内部でバックグラウンドステップを宣言することはできません。

### 例: バックグラウンドでステップを実行する

```text
steps
  - name Start server
    id server
    run npm start
    background true

  - name Run tests against the server
    run npm test

  - name Wait for the server step to finish
    wait server
```

## `jobs.job_id.steps[].wait`

1 つ以上のバックグラウンドステップが完了するまでジョブを一時停止します。`wait` ステップ自体は作業を行わず、参照されたバックグラウンドステップが完了するまでブロックするだけです。単一のステップ `id` を文字列として指定するか、複数のステップ `id` を配列として指定します。

`wait` ステップが完了すると、参照されたバックグラウンドステップの出力を後続のステップで使用できるようになります。参照されたバックグラウンドステップが失敗した場合、`wait` ステップも失敗します。

> **メモ**
>
> `wait` ステップは常に実行され、`if` 条件はサポートしません。

### 例: 特定のバックグラウンドステップを待機する

```text
steps
  - name Build frontend
    id build-frontend
    run npm run buildfrontend
    background true

  - name Build backend
    id build-backend
    run npm run buildbackend
    background true

  - name Run linter while builds run
    run npm run lint

  - name Wait for both builds to finish
    wait [build-frontend, build-backend]

  - name Run tests
    run npm test
```

## `jobs.job_id.steps[].wait-all`

アクティブなすべてのバックグラウンドステップが完了するまでジョブを一時停止します。複数のバックグラウンドステップが実行中で、続行する前にすべてを完了させたい場合に便利です。`wait` と同様に、`wait-all` ステップは、待機対象のバックグラウンドステップのいずれかが失敗した場合、`continue-on-error` を `true` に設定していない限り失敗します。

`wait-all` キーワードは引数を取りません。

> **メモ**
>
> `wait-all` ステップは常に実行され、`if` 条件はサポートしません。

### 例: すべてのバックグラウンドステップを待機する

```text
steps
  - name Start database
    id db
    run docker run -d postgres15
    background true

  - name Start cache
    id cache
    run docker run -d redis7
    background true

  - name Run integration tests
    run npm run testintegration

  - name Wait for all services to stop
    wait-all
```

## `jobs.job_id.steps[].cancel`

実行中のバックグラウンドステップを正常終了させます。ランナーはステップのプロセスに終了シグナル（SIGTERM）を送信してクリーンアップできるようにし、短い猶予期間内に終了しない場合は強制的に停止（SIGKILL）します。`cancel` キーワードは、単一のバックグラウンドステップをその `id` で対象にします。

> **メモ**
>
> `cancel` ステップは常に実行され、`if` 条件はサポートしません。

### 例: バックグラウンドステップをキャンセルする

```text
steps
  - name Start long-running monitor
    id monitor
    run .scriptsmonitor.sh
    background true

  - name Run the main task
    run npm test

  - name Stop the monitor
    cancel monitor
```

## `jobs.job_id.steps[].parallel`

ステップのグループを同時に実行し、続行する前にそれらすべてが完了するまで待機します。`parallel` キーワードは、グループ内の各ステップをバックグラウンドステップとして実行し、グループの最後で暗黙的に待機するための省略表現です。同時に実行できる独立したステップのグループがあり、それらを個別に参照する必要がない場合に使用します。

複数のコンポーネントを一度にビルドする場合など、ジョブが先に進む前にすべて完了すべき自己完結したステップのグループがある場合は、`parallel` を使用します。後続のステップの実行中に稼働し続けるサーバーやデータベースのような長時間実行プロセスを開始する細かな制御、`wait` または `cancel` による特定のステップの参照、またはバックグラウンド作業とほかのステップの交互実行が必要な場合は、`background` を使用します。要するに、`parallel` はより限定的ですが「このグループを一度に実行する」場合にはより便利であり、`background` は汎用的な基本機能です。

グループ内の各ステップには、ほかのバックグラウンドステップと同じ 10 ステップの同時実行制限が適用されます。

> **メモ**
>
> コンポジットアクション内では `parallel` を使用できません。

### 例: ステップを並列に実行する

```text
steps
  - uses actionscheckout@v6

  - parallel
      - name Build frontend
        run npm run buildfrontend

      - name Build backend
        run npm run buildbackend

      - name Build docs
        run npm run builddocs

  - name Run tests after all builds complete
    run npm test
```

上記のグループは、各ステップに `background true` を指定して宣言し、その後に `wait` ステップを置くことと同等です。

## `jobs.<job_id>.timeout-minutes`

GitHub が自動的にジョブをキャンセルするまでにジョブを実行できる最大分数です。既定値: 360

タイムアウトがランナーのジョブ実行時間制限を超えている場合は、代わりに実行時間制限に達した時点でジョブがキャンセルされます。ジョブ実行時間制限について詳しくは、GitHub ホストランナーの課金と使用状況、およびセルフホストランナーの使用制限に関する Actions の制限を参照してください。

> **メモ**
>
> GITHUB_TOKEN は、ジョブが終了したとき、または最大 24 時間後に期限切れになります。セルフホストランナーでは、ジョブのタイムアウトが 24 時間を超える場合、トークンが制限要因になることがあります。GITHUB_TOKEN について詳しくは、ワークフローでの認証に GITHUB_TOKEN を使用する方法を参照してください。

## `jobs.<job_id>.strategy`

ジョブでマトリックス戦略を使用するには、`jobs.<job_id>.strategy` を使用します。マトリックス戦略を使用すると、単一のジョブ定義内で変数を使用し、その変数の組み合わせに基づいて複数のジョブ実行を自動的に作成できます。たとえば、マトリックス戦略を使用して、複数の言語バージョンや複数のオペレーティングシステムでコードをテストできます。詳しくは、ワークフローでジョブのバリエーションを実行する方法を参照してください。

## `jobs.<job_id>.strategy.matrix`

さまざまなジョブ構成のマトリックスを定義するには、`jobs.<job_id>.strategy.matrix` を使用します。詳しくは、ワークフローでジョブのバリエーションを実行する方法を参照してください。

マトリックスでは、1 回のワークフロー実行につき最大 256 個のジョブが生成されます。この制限は、GitHub ホストランナーとセルフホストランナーの両方に適用されます。

定義した変数は `matrix` コンテキストのプロパティになり、ワークフローファイルのほかの場所でそのプロパティを参照できます。この例では、`matrix.version` と `matrix.os` を使用して、ジョブが使用している `version` と `os` の現在値にアクセスできます。詳しくはコンテキストのリファレンスを参照してください。

既定では、GitHub はランナーの可用性に応じて、並列に実行されるジョブ数を最大化します。マトリックス内の変数の順序によって、ジョブが作成される順序が決まります。最初に定義した変数が、ワークフロー実行で最初に作成されるジョブになります。

### 単一次元のマトリックスを使用する

次のワークフローでは、変数 `version` を値 `[10, 12, 14]` で定義しています。このワークフローは、変数内の各値に対して 1 つずつ、合計 3 つのジョブを実行します。各ジョブは `matrix.version` コンテキストを通じて `version` 値にアクセスし、その値を `actions/setup-node` アクションに `node-version` として渡します。

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

### 多次元のマトリックスを使用する

複数の変数を指定して、多次元のマトリックスを作成します。ジョブは、変数のすべての可能な組み合わせごとに実行されます。

たとえば、次のワークフローでは 2 つの変数を指定しています。

- `os` 変数に指定された 2 つのオペレーティングシステム
- `version` 変数に指定された 3 つの Node.js バージョン

このワークフローは、`os` 変数と `version` 変数の各組み合わせに対して 1 つずつ、合計 6 つのジョブを実行します。各ジョブは `runs-on` の値を現在の `os` 値に設定し、現在の `version` 値を `actions/setup-node` アクションに渡します。

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

マトリックス内の変数構成は、オブジェクトの配列にすることができます。たとえば、次のマトリックスは対応するコンテキストを持つ 4 つのジョブを生成します。

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

マトリックス内の各ジョブは、次に示すように、`os` と `node` の値の独自の組み合わせを持ちます。

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

## `jobs.<job_id>.strategy.matrix.include`

`include` リスト内の各オブジェクトについて、そのオブジェクト内の `key:value` ペアは、元のマトリックス値のいずれも上書きしない場合に、各マトリックスの組み合わせへ追加されます。そのオブジェクトをどのマトリックスの組み合わせにも追加できない場合は、代わりに新しいマトリックスの組み合わせが作成されます。元のマトリックス値は上書きされませんが、追加されたマトリックス値は上書きされる可能性があることに注意してください。

### 例: 構成を展開する

たとえば、次のワークフローは、`os` と `node` の各組み合わせに対して 1 つずつ、合計 4 つのジョブを実行します。`os` の値が `windows-latest`、`node` の値が `16` のジョブが実行されると、そのジョブには値 `6` を持つ `npm` という追加の変数が含まれます。

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

### 例: 構成を追加する

たとえば、このマトリックスは、マトリックス内の `os` と `version` の各組み合わせに対して 1 つずつ、さらに `os` の値が `windows-latest`、`version` の値が `17` のジョブを加えた、合計 10 個のジョブを実行します。

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

マトリックス変数を指定しない場合、`include` 配下のすべての構成が実行されます。たとえば、次のワークフローでは、各 `include` エントリに対して 1 つずつ、合計 2 つのジョブが実行されます。これにより、完全に入力されたマトリックスがなくてもマトリックス戦略を活用できます。

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

## `jobs.<job_id>.strategy.matrix.exclude`

除外される構成は、除外されるためには部分一致するだけでかまいません。

すべての `include` の組み合わせは、`exclude` の後に処理されます。これにより、`include` を使用して、以前に除外された組み合わせを追加し直すことができます。

## `jobs.<job_id>.strategy.fail-fast`

ジョブの失敗をどのように処理するかは、`jobs.<job_id>.strategy.fail-fast` と `jobs.<job_id>.continue-on-error` で制御できます。

`jobs.<job_id>.strategy.fail-fast` はマトリックス全体に適用されます。`jobs.<job_id>.strategy.fail-fast` が `true` に設定されている場合、またはその式が `true` と評価される場合、マトリックス内のいずれかのジョブが失敗すると、GitHub はマトリックス内で進行中およびキューに入っているすべてのジョブをキャンセルします。このプロパティの既定値は `true` です。

`jobs.<job_id>.continue-on-error` は単一のジョブに適用されます。`jobs.<job_id>.continue-on-error` が `true` の場合、`jobs.<job_id>.continue-on-error: true` が指定されたジョブが失敗しても、マトリックス内のほかのジョブは実行を続けます。

`jobs.<job_id>.strategy.fail-fast` と `jobs.<job_id>.continue-on-error` は一緒に使用できます。たとえば、次のワークフローは 4 つのジョブを開始します。各ジョブでは、`continue-on-error` は `matrix.experimental` の値によって決まります。`continue-on-error: false` のジョブのいずれかが失敗すると、進行中またはキューに入っているすべてのジョブがキャンセルされます。`continue-on-error: true` のジョブが失敗しても、ほかのジョブには影響しません。

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

## `jobs.<job_id>.strategy.max-parallel`

既定では、GitHub はランナーの可用性に応じて、並列に実行されるジョブ数を最大化します。

## `jobs.<job_id>.continue-on-error`

`jobs.<job_id>.continue-on-error` は単一のジョブに適用されます。`jobs.<job_id>.continue-on-error` が `true` の場合、`jobs.<job_id>.continue-on-error: true` が指定されたジョブが失敗しても、マトリックス内のほかのジョブは実行を続けます。

ジョブが失敗したときにワークフロー実行が失敗するのを防ぎます。このジョブが失敗してもワークフロー実行を成功として扱えるようにするには、`true` に設定します。

### 例: 特定の失敗したマトリックスジョブでワークフロー実行が失敗しないようにする

ジョブマトリックス内の特定のジョブが失敗しても、ワークフロー実行を失敗させないようにできます。たとえば、`node` が `15` に設定された実験的なジョブだけが失敗しても、ワークフロー実行を失敗させないようにしたい場合です。

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

## `jobs.<job_id>.container`

> **メモ**
> 
> ワークフローで Docker コンテナアクション、ジョブコンテナ、またはサービスコンテナを使用する場合は、Linux ランナーを使用する必要があります。
> 
> GitHub ホストランナーを使用している場合は、Ubuntu ランナーを使用する必要があります。
> セルフホストランナーを使用している場合は、Linux マシンをランナーとして使用し、Docker がインストールされている必要があります。

`jobs.<job_id>.container` を使用すると、ジョブ内で、すでにコンテナを指定していない任意のステップを実行するためのコンテナを作成できます。スクリプトとコンテナアクションの両方を使用するステップがある場合、コンテナアクションは、同じネットワーク上で同じボリュームマウントを持つ兄弟コンテナとして実行されます。

コンテナを設定しない場合、コンテナ内で実行するよう設定されたアクションをステップが参照していない限り、すべてのステップは `runs-on` で指定されたホスト上で直接実行されます。

> **メモ**
> 
> コンテナ内の `run` ステップの既定のシェルは、`bash` ではなく `sh` です。これは `jobs.<job_id>.defaults.run` または `jobs.<job_id>.steps[*].shell` で上書きできます。

### 例: コンテナ内でジョブを実行する

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

コンテナイメージだけを指定する場合は、`image` キーワードを省略できます。

```yaml
jobs:
  container-test-job:
    runs-on: ubuntu-latest
    container: node:18
```

## `jobs.<job_id>.container.image`

`jobs.<job_id>.container.image` を使用すると、アクションを実行するコンテナとして使用する Docker イメージを定義できます。値には Docker Hub のイメージ名またはレジストリ名を指定できます。

> **メモ**
> 
> Docker Hub は通常、プッシュ操作とプル操作の両方にレート制限を課しており、これはセルフホストランナー上のジョブに影響します。ただし、GitHub と Docker の間の合意により、GitHub ホストランナーはこれらの制限の対象になりません。

## `jobs.<job_id>.container.credentials`

イメージのコンテナレジストリでイメージをプルするために認証が必要な場合は、`jobs.<job_id>.container.credentials` を使用して、ユーザー名とパスワードのマップを設定できます。認証情報は、`docker login` コマンドに指定する値と同じです。

### 例: コンテナレジストリの認証情報を定義する

```yaml
container:
  image: ghcr.io/owner/image
  credentials:
     username: ${{ github.actor }}
     password: ${{ secrets.github_token }}
```

## `jobs.<job_id>.container.env`

`jobs.<job_id>.container.env` を使用すると、コンテナ内の環境変数のマップを設定できます。

## `jobs.<job_id>.container.ports`

`jobs.<job_id>.container.ports` を使用すると、コンテナで公開するポートの配列を設定できます。

## `jobs.<job_id>.container.volumes`

`jobs.<job_id>.container.volumes` を使用すると、コンテナが使用するボリュームの配列を設定できます。ボリュームを使用して、サービス間またはジョブ内の他のステップとの間でデータを共有できます。名前付き Docker ボリューム、匿名 Docker ボリューム、またはホスト上のバインドマウントを指定できます。

ボリュームを指定するには、ソースパスと宛先パスを指定します。

`<source>:<destinationPath>`。

`<source>` はボリューム名またはホストマシン上の絶対パスで、`<destinationPath>` はコンテナ内の絶対パスです。

### 例: コンテナでボリュームをマウントする

```yaml
volumes:
  - my_docker_volume:/volume_mount
  - /data/my_data
  - /source/directory:/destination/directory
```

## `jobs.<job_id>.container.options`

`jobs.<job_id>.container.options` を使用すると、追加の Docker コンテナリソースオプションを構成できます。オプションの一覧については、`docker create` オプションを参照してください。

> **警告**
> 
> `--network` オプションと `--entrypoint` オプションはサポートされていません。

## `jobs.<job_id>.services`

> **メモ**
> 
> ワークフローで Docker コンテナアクション、ジョブコンテナ、またはサービスコンテナを使用する場合は、Linux ランナーを使用する必要があります。
> 
> GitHub ホストランナーを使用している場合は、Ubuntu ランナーを使用する必要があります。
> セルフホストランナーを使用している場合は、Linux マシンをランナーとして使用し、Docker がインストールされている必要があります。

ワークフロー内のジョブのサービスコンテナをホストするために使用します。サービスコンテナは、Redis のようなデータベースまたはキャッシュサービスを作成するのに役立ちます。ランナーは Docker ネットワークを自動的に作成し、サービスコンテナのライフサイクルを管理します。

ジョブをコンテナ内で実行するよう構成している場合、またはステップでコンテナアクションを使用している場合は、サービスまたはアクションにアクセスするためにポートをマップする必要はありません。Docker は、同じ Docker ユーザー定義ブリッジネットワーク上のコンテナ間ですべてのポートを自動的に公開します。サービスコンテナはホスト名で直接参照できます。ホスト名は、ワークフローでサービスに設定したラベル名に自動的にマップされます。

ジョブをランナーマシン上で直接実行するよう構成していて、ステップでコンテナアクションを使用していない場合は、必要な Docker サービスコンテナのポートを Docker ホスト（ランナーマシン）にマップする必要があります。`localhost` とマップされたポートを使用して、サービスコンテナにアクセスできます。

ネットワークサービスコンテナ間の違いについて詳しくは、「Docker サービスコンテナとの通信」を参照してください。

### 例: `localhost` を使用する

この例では、nginx と redis の 2 つのサービスを作成します。コンテナポートを指定し、ホストポートを指定しない場合、コンテナポートはホスト上の空きポートにランダムに割り当てられます。GitHub は、割り当てられたホストポートを `${{job.services.<service_name>.ports}}` コンテキストに設定します。この例では、`${{ job.services.nginx.ports['80'] }}` コンテキストと `${{ job.services.redis.ports['6379'] }}` コンテキストを使用して、サービスのホストポートにアクセスできます。

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

## `jobs.<job_id>.services.<service_id>.image`

アクションを実行するサービスコンテナとして使用する Docker イメージです。値には Docker Hub のイメージ名またはレジストリ名を指定できます。

`jobs.<job_id>.services.<service_id>.image` に空の文字列が割り当てられている場合、サービスは開始されません。これを使用して、次の例のように条件付きサービスを設定できます。

```yaml
services:
  nginx:
    image: ${{ options.nginx == true && 'nginx' || '' }}
```

## `jobs.<job_id>.services.<service_id>.credentials`

イメージのコンテナレジストリでイメージをプルするために認証が必要な場合は、`jobs.<job_id>.container.credentials` を使用して、ユーザー名とパスワードのマップを設定できます。認証情報は、`docker login` コマンドに指定する値と同じです。

### `jobs.<job_id>.services.<service_id>.credentials` の例

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

## `jobs.<job_id>.services.<service_id>.env`

サービスコンテナ内の環境変数のマップを設定します。

## `jobs.<job_id>.services.<service_id>.ports`

サービスコンテナで公開するポートの配列を設定します。

## `jobs.<job_id>.services.<service_id>.volumes`

サービスコンテナが使用するボリュームの配列を設定します。ボリュームを使用して、サービス間またはジョブ内の他のステップとの間でデータを共有できます。名前付き Docker ボリューム、匿名 Docker ボリューム、またはホスト上のバインドマウントを指定できます。

ボリュームを指定するには、ソースパスと宛先パスを指定します。

`<source>:<destinationPath>`。

`<source>` はボリューム名またはホストマシン上の絶対パスで、`<destinationPath>` はコンテナ内の絶対パスです。

### `jobs.<job_id>.services.<service_id>.volumes` の例

```yaml
volumes:
  - my_docker_volume:/volume_mount
  - /data/my_data
  - /source/directory:/destination/directory
```

## `jobs.<job_id>.services.<service_id>.options`

追加の Docker コンテナリソースオプションです。オプションの一覧については、`docker create` オプションを参照してください。

> **警告**
> 
> `--network` オプションはサポートされていません。

## `jobs.<job_id>.services.<service_id>.command`

Docker イメージの既定のコマンド（`CMD`）を上書きします。この値は、`docker create` コマンドでイメージ名の後に引数として渡されます。`entrypoint` も指定した場合、`command` はその `entrypoint` への引数を提供します。

### `jobs.<job_id>.services.<service_id>.command` の例

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

## `jobs.<job_id>.services.<service_id>.entrypoint`

Docker イメージの既定の `ENTRYPOINT` を上書きします。この値は、実行する実行可能ファイルを定義する単一の文字列です。イメージのエントリーポイントを完全に置き換える必要がある場合に使用します。`entrypoint` と `command` を組み合わせると、カスタムエントリーポイントに引数を渡すことができます。

### `jobs.<job_id>.services.<service_id>.entrypoint` の例

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

## `jobs.<job_id>.uses`

ジョブとして実行する再利用可能なワークフローファイルの場所とバージョンです。次のいずれかの構文を使用します。

`{owner}/{repo}/.github/workflows/{filename}@{ref}` は、パブリックリポジトリとプライベートリポジトリ内の再利用可能なワークフローに使用します。
`./.github/workflows/{filename}` は、同じリポジトリ内の再利用可能なワークフローに使用します。

最初のオプションでは、`{ref}` に SHA、リリースタグ、またはブランチ名を指定できます。リリースタグとブランチが同じ名前の場合、リリースタグがブランチ名より優先されます。安定性とセキュリティのためには、コミット SHA を使用するのが最も安全なオプションです。詳しくは、「安全な使用のリファレンス」を参照してください。

2 番目の構文オプション（`{owner}/{repo}` と `@{ref}` を含まないもの）を使用する場合、呼び出されるワークフローは呼び出し元ワークフローと同じコミットから取得されます。`refs/heads` や `refs/tags` などの ref プレフィックスは許可されていません。このキーワードでは、コンテキストや式を使用できません。

### `jobs.<job_id>.uses` の例

```yaml
jobs:
  call-workflow-1-in-local-repo:
    uses: octo-org/this-repo/.github/workflows/workflow-1.yml@172239021f7ba04fe7327647b213799853a9eb89
  call-workflow-2-in-local-repo:
    uses: ./.github/workflows/workflow-2.yml
  call-workflow-in-another-repo:
    uses: octo-org/another-repo/.github/workflows/workflow.yml@v1
```

詳しくは、「ワークフローの再利用」を参照してください。

## `jobs.<job_id>.with`

ジョブを使用して再利用可能なワークフローを呼び出す場合、`with` を使用して、呼び出されるワークフローに渡される入力のマップを指定できます。

渡す入力は、呼び出されるワークフローで定義されている入力仕様と一致している必要があります。

`jobs.<job_id>.steps[*].with` とは異なり、`jobs.<job_id>.with` で渡す入力は、呼び出されるワークフロー内で環境変数として使用できません。代わりに、`inputs` コンテキストを使用して入力を参照できます。

### `jobs.<job_id>.with` の例

```yaml
jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/called-workflow.yml@main
    with:
      username: mona
```

## `jobs.<job_id>.with.<input_id>`

入力の文字列識別子と入力の値で構成されるペアです。識別子は、呼び出されるワークフローの `on.workflow_call.inputs.<inputs_id>` で定義された入力の名前と一致している必要があります。値のデータ型は、呼び出されるワークフローの `on.workflow_call.inputs.<input_id>.type` で定義された型と一致している必要があります。

許可される式コンテキスト: `github`、`needs`。

## `jobs.<job_id>.secrets`

ジョブを使用して再利用可能なワークフローを呼び出す場合、`secrets` を使用して、呼び出されるワークフローに渡されるシークレットのマップを指定できます。

渡すシークレットは、呼び出されるワークフローで定義されている名前と一致している必要があります。

### `jobs.<job_id>.secrets` の例

```yaml
jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/called-workflow.yml@main
    secrets:
      access-token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
```

## `jobs.<job_id>.secrets.inherit`

`inherit` キーワードを使用すると、呼び出し元ワークフローのすべてのシークレットを、呼び出されるワークフローに渡すことができます。これには、呼び出し元ワークフローがアクセスできるすべてのシークレット、つまり組織、リポジトリ、環境のシークレットが含まれます。`inherit` キーワードは、同じ組織内のリポジトリ間、または同じ Enterprise 内の組織間でシークレットを渡すために使用できます。

### `jobs.<job_id>.secrets.inherit` の例

```yaml
on:
  workflow_dispatch:

jobs:
  pass-secrets-to-workflow:
    uses: ./.github/workflows/called-workflow.yml
    secrets: inherit
```

```yaml
on:
  workflow_call:

jobs:
  pass-secret-to-action:
    runs-on: ubuntu-latest
    steps:
      - name: Use a repo or org secret from the calling workflow.
        run: echo ${{ secrets.CALLING_WORKFLOW_SECRET }}
```

## `jobs.<job_id>.secrets.<secret_id>`

シークレットの文字列識別子とシークレットの値で構成されるペアです。識別子は、呼び出されるワークフローの `on.workflow_call.secrets.<secret_id>` で定義されたシークレットの名前と一致している必要があります。

許可される式コンテキスト: `github`、`needs`、`secrets`。

## フィルターパターンのチートシート

パス、ブランチ、タグのフィルターでは特殊文字を使用できます。

- `*`: 0 個以上の文字に一致しますが、`/` 文字には一致しません。たとえば、`Octo*` は `Octocat` に一致します。
- `**`: 任意の文字 0 個以上に一致します。
- `?`: 直前の文字 0 個または 1 個に一致します。
- `+`: 直前の文字 1 個以上に一致します。
- `[]`: 角かっこ内に列挙された、または範囲に含まれる英数字 1 文字に一致します。範囲には `a-z`、`A-Z`、`0-9` のみを含めることができます。たとえば、範囲 `[0-9a-z]` は任意の数字または小文字に一致します。たとえば、`[CB]at` は `Cat` または `Bat` に一致し、`[1-2]00` は `100` と `200` に一致します。
- `!`: パターンの先頭にある場合、以前の肯定パターンを否定します。最初の文字でない場合、特別な意味はありません。

`*`、`[`、`!` の文字は YAML の特殊文字です。パターンを `*`、`[`、または `!` で始める場合は、パターンを引用符で囲む必要があります。また、`[` や `]` を含むパターンをフローシーケンスで使用する場合も、パターンを引用符で囲む必要があります。

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

ブランチ、タグ、パスのフィルター構文について詳しくは、`on.<push>.<branches|tags>`、`on.<pull_request>.<branches|tags>`、`on.<push|pull_request>.paths` を参照してください。

### ブランチとタグに一致するパターン

| パターン | 説明 | 一致例 |
|---|---|---|
| `feature/*` | `*` ワイルドカードは任意の文字に一致しますが、スラッシュ（`/`）には一致しません。 | `feature/my-branch`<br>`feature/your-branch` |
| `feature/**` | `**` ワイルドカードは、ブランチ名とタグ名の中でスラッシュ（`/`）を含む任意の文字に一致します。 | `feature/beta-a/my-branch`<br>`feature/your-branch`<br>`feature/mona/the/octocat` |
| `main`<br><br>`releases/mona-the-octocat` | ブランチ名またはタグ名の正確な名前に一致します。 | `main`<br>`releases/mona-the-octocat` |
| `'*'` | スラッシュ（`/`）を含まないすべてのブランチ名とタグ名に一致します。`*` 文字は YAML の特殊文字です。パターンを `*` で始める場合は、引用符を使用する必要があります。 | `main`<br>`releases` |
| `'**'` | すべてのブランチ名とタグ名に一致します。これは、`branches` または `tags` フィルターを使用しない場合の既定の動作です。 | `all/the/branches`<br>`every/tag` |
| `'*feature'` | `*` 文字は YAML の特殊文字です。パターンを `*` で始める場合は、引用符を使用する必要があります。 | `mona-feature`<br>`feature`<br>`ver-10-feature` |
| `v2*` | `v2` で始まるブランチ名とタグ名に一致します。 | `v2`<br>`v2.0`<br>`v2.9` |
| `v[12].[0-9]+.[0-9]+` | メジャーバージョンが 1 または 2 のすべてのセマンティックバージョニングのブランチとタグに一致します。 | `v1.10.1`<br>`v2.0.0` |

### ファイルパスに一致するパターン

パスパターンはパス全体に一致する必要があり、リポジトリのルートから始まります。

| パターン | 一致内容の説明 | 一致例 |
|---|---|---|
| `'*'` | `*` ワイルドカードは任意の文字に一致しますが、スラッシュ（`/`）には一致しません。`*` 文字は YAML の特殊文字です。パターンを `*` で始める場合は、引用符を使用する必要があります。 | `README.md`<br>`server.rb` |
| `'*.jsx?'` | `?` 文字は、直前の文字 0 個または 1 個に一致します。 | `page.js`<br>`page.jsx` |
| `'**'` | `**` ワイルドカードはスラッシュ（`/`）を含む任意の文字に一致します。これは、パスフィルターを使用しない場合の既定の動作です。 | `all/the/files.md` |
| `'*.js'` | `*` ワイルドカードは任意の文字に一致しますが、スラッシュ（`/`）には一致しません。リポジトリのルートにあるすべての `.js` ファイルに一致します。 | `app.js`<br>`index.js` |
| `'**.js'` | リポジトリ内のすべての `.js` ファイルに一致します。 | `index.js`<br>`js/index.js`<br>`src/js/app.js` |
| `docs/*` | リポジトリのルートにある `docs` ディレクトリ直下のすべてのファイルにのみ一致します。 | `docs/README.md`<br>`docs/file.txt` |
| `docs/**` | リポジトリのルートにある `docs` ディレクトリとそのサブディレクトリ内の任意のファイルに一致します。 | `docs/README.md`<br>`docs/mona/octocat.txt` |
| `docs/**/*.md` | `docs` ディレクトリ内の任意の場所にある `.md` サフィックスを持つファイルに一致します。 | `docs/README.md`<br>`docs/mona/hello-world.md`<br>`docs/a/markdown/file.md` |
| `'**/docs/**'` | リポジトリ内の任意の場所にある `docs` ディレクトリ内の任意のファイルに一致します。 | `docs/hello.md`<br>`dir/docs/my-file.txt`<br>`space/docs/plan/space.doc` |
| `'**/README.md'` | リポジトリ内の任意の場所にある `README.md` ファイルに一致します。 | `README.md`<br>`js/README.md` |
| `'**/*src/**'` | リポジトリ内の任意の場所にある `src` サフィックスを持つフォルダー内の任意のファイルに一致します。 | `a/src/app.js`<br>`my-src/code/js/app.js` |
| `'**/*-post.md'` | リポジトリ内の任意の場所にある `-post.md` サフィックスを持つファイルに一致します。 | `my-post.md`<br>`path/their-post.md` |
| `'**/migrate-*.sql'` | リポジトリ内の任意の場所にある `migrate-` プレフィックスと `.sql` サフィックスを持つファイルに一致します。 | `migrate-10909.sql`<br>`db/migrate-v1.0.sql`<br>`db/sept/migrate-v1.sql` |
| `'*.md'`<br><br>`'!README.md'` | パターンの前に感嘆符（`!`）を使用すると、そのパターンが否定されます。ファイルがあるパターンに一致し、さらにファイル内で後に定義された否定パターンにも一致する場合、そのファイルは含まれません。 | `hello.md`<br><br>一致しない<br><br>`README.md`<br>`docs/hello.md` |
| `'*.md'`<br><br>`'!README.md'`<br><br>`README*` | パターンは順番にチェックされます。前のパターンを否定するパターンによって、ファイルパスが再び含められます。 | `hello.md`<br>`README.md`<br>`README.doc` |

