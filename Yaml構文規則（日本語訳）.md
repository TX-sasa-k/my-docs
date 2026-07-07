# GitHub Actions ワークフロー構文（日本語）

## ワークフロー向け YAML 構文について

ワークフローファイルは YAML 構文を使用し、拡張子は .yml または .yaml のいずれかである必要があります。 YAML が初めてで詳しく学びたい場合は、Learn YAML in Y minutes を参照してください。

ワークフローファイルはリポジトリの .github/ワークフロー ディレクトリに保存する必要があります。

## `name`

ワークフローの名前です。 GitHub displays the names of your ワークフロー under your repository's "Actions" tab. もし omit name, GitHub displays the ワークフロー file path relative to the root of the repository.

## `run-name`

The name for ワークフロー runs generated from the ワークフロー. GitHub displays the ワークフロー run name in the list of ワークフロー runs on your repository's "Actions" tab. If run-name is omitted or is only whitespace, then the run name is set to イベント-specific information for the ワークフロー run. 例えば、 for a ワークフロー triggered by a push or pull_request イベント, it is set as the commit message or the title of the pull request.

## この値には式を含めることができ、github および inputs コンテキストを参照できます。

### run-name の例

```yaml
run-name: Deploy to ${{ inputs.deploy_target }} by @${{ github.actor }}
```

## `on`

〜するには automatically trigger a ワークフロー, use on to define which イベント can cause the ワークフロー to run. For a list of available イベント, see Events that trigger ワークフロー.

次のことができます define single or multiple イベント that can trigger a ワークフロー, or set a time schedule. 次のことができます also restrict the execution of a ワークフロー to only occur for specific files, タグ, or ブランチ changes. These options are described in the following sections.

## 単一イベントの使用

例えば、 a ワークフロー with the following on value will run when a push is made to any ブランチ in the ワークフロー's repository:

```yaml
on: push
```

## 複数イベントの使用

次のことができます specify a single イベント or multiple イベント. 例えば、 a ワークフロー with the following on value will run when a push is made to any ブランチ in the repository or when someone forks the repository:

```yaml
on: [push, fork]
```

もし specify multiple イベント, only one of those イベント needs to occur to trigger your ワークフロー. If multiple triggering イベント for your ワークフロー occur at the same time, multiple ワークフロー runs will be triggered.

## アクティビティタイプの使用

Some イベント have activity types that give you more control over when your ワークフロー should run. Use on.<イベント_name>.types to define the type of イベント activity that will trigger a ワークフロー run.

例えば、 the issue_comment イベント has the created, edited, and deleted activity types. もし r ワークフロー triggers on the label イベント, it will run whenever a label is created, edited, or deleted. もし specify the created activity type for the label イベント, your ワークフロー will run when a label is created but not when a label is edited or deleted.

```yaml
on:
  label:
    types:
      - created
```

もし specify multiple activity types, only one of those イベント activity types needs to occur to trigger your ワークフロー. If multiple triggering イベント activity types for your ワークフロー occur at the same time, multiple ワークフロー runs will be triggered. 例えば、 the following ワークフロー triggers when an issue is opened or labeled. If an issue with two labels is opened, three ワークフロー runs will start: one for the issue opened イベント and two for the two issue labeled イベント.

```yaml
on:
  issues:
    types:
      - opened
      - labeled
```

詳細については about each イベント and their activity types, see Events that trigger ワークフロー.

## フィルターの使用

Some イベント have フィルターs that give you more control over when your ワークフロー should run.

例えば、 the push イベント has a ブランチ フィルター that causes your ワークフロー to run only when a push to a ブランチ that matches the ブランチ フィルター occurs, instead of when any push occurs.

```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'
```

## 複数イベントでのアクティビティタイプとフィルターの併用

もし specify activity types or フィルターs for an イベント and your ワークフロー triggers on multiple イベント, you must configure each イベント separately. You must append a colon (:) to all イベント, including イベント without configuration.

例えば、 a ワークフロー with the following on value will run when:

A label is created

A push is made to the main ブランチ in the repository

A push is made to a GitHub Pages-enabled ブランチ

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

on.<イベント_name>.types

Use on.<イベント_name>.types to define the type of activity that will trigger a ワークフロー run. Most GitHub イベント are triggered by more than one type of activity. 例えば、 the label is triggered when a label is created, edited, or deleted. The types keyword enables you to narrow down activity that causes the ワークフロー to run. 〜する場合、 only one activity type triggers a webhook イベント, the types keyword is unnecessary.

次のことができます use an array of イベント types. 詳細については about each イベント and their activity types, see Events that trigger ワークフロー.

```yaml
on:
  label:
    types: [created, edited]
```

on.<pull_request|pull_request_target>.<ブランチ|ブランチ-ignore>

〜する場合、 using the pull_request and pull_request_target イベント, you can configure a ワークフロー to run only for pull requests that target specific ブランチ.

Use the ブランチ フィルター when you want to include ブランチ name patterns or when you want to both include and exclude ブランチ names patterns. Use the ブランチ-ignore フィルター when you only want to exclude ブランチ name patterns. 次のことができますnot use both the ブランチ and ブランチ-ignore フィルターs for the same イベント in a ワークフロー.

もし define both ブランチ/ブランチ-ignore and paths/paths-ignore, the ワークフロー will only run when both フィルターs are satisfied.

The ブランチ and ブランチ-ignore keywords accept glob patterns that use characters like *, **, +, ?, ! and others to match more than one ブランチ name. If a name contains any of these characters and you want a literal match, you need to escape each of these special characters with \. 詳細については about glob patterns, see the Workflow syntax for GitHub Actions.

```yaml
Example: Including branches
```

The patterns defined in ブランチ are evaluated against the Git ref's name. 例えば、 the following ワークフロー would run whenever there is a pull_request イベント for a pull request targeting:

A ブランチ named main (refs/heads/main)

A ブランチ named mona/octocat (refs/heads/mona/octocat)

A ブランチ whose name starts with releases/, like releases/10 (refs/heads/releases/10)

```yaml
on:
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches:
      - main
      - 'mona/octocat'
      - 'releases/**'
```

If a ワークフロー is skipped due to ブランチ フィルターing, path フィルターing, or a commit message, then checks associated with that ワークフロー will remain in a "Pending" state. A pull request that requires those checks to be successful will be blocked from merging.

```yaml
Example: Excluding branches
```

〜する場合、 a pattern matches the ブランチ-ignore pattern, the ワークフロー will not run. The patterns defined in ブランチ-ignore are evaluated against the Git ref's name. 例えば、 the following ワークフロー would run whenever there is a pull_request イベント unless the pull request is targeting:

A ブランチ named mona/octocat (refs/heads/mona/octocat)

A ブランチ whose name matches releases/**-alpha, like releases/beta/3-alpha (refs/heads/releases/beta/3-alpha)

```yaml
on:
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches-ignore:
      - 'mona/octocat'
      - 'releases/**-alpha'
Example: Including and excluding branches
```

次のことができますnot use ブランチ and ブランチ-ignore to フィルター the same イベント in a single ワークフロー. もし want to both include and exclude ブランチ patterns for a single イベント, use the ブランチ フィルター along with the ! character to indicate which ブランチ should be excluded.

もし define a ブランチ with the ! character, you must also define at least one ブランチ without the ! character. もし only want to exclude ブランチ, use ブランチ-ignore instead.

The order that you define patterns matters.

A matching negative pattern (prefixed with !) after a positive match will exclude the Git ref.

A matching positive pattern after a negative match will include the Git ref again.

The following ワークフロー will run on pull_request イベント for pull requests that target releases/10 or releases/beta/mona, but not for pull requests that target releases/10-alpha or releases/beta/3-alpha because the negative pattern !releases/**-alpha follows the positive pattern.

```yaml
on:
  pull_request:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

on.push.<ブランチ|タグ|ブランチ-ignore|タグ-ignore>

〜する場合、 using the push イベント, you can configure a ワークフロー to run on specific ブランチ or タグ.

Use the ブランチ フィルター when you want to include ブランチ name patterns or when you want to both include and exclude ブランチ names patterns. Use the ブランチ-ignore フィルター when you only want to exclude ブランチ name patterns. 次のことができますnot use both the ブランチ and ブランチ-ignore フィルターs for the same イベント in a ワークフロー.

Use the タグ フィルター when you want to include タグ name patterns or when you want to both include and exclude タグ names patterns. Use the タグ-ignore フィルター when you only want to exclude タグ name patterns. 次のことができますnot use both the タグ and タグ-ignore フィルターs for the same イベント in a ワークフロー.

もし define only タグ/タグ-ignore or only ブランチ/ブランチ-ignore, the ワークフロー won't run for イベント affecting the undefined Git ref. もし define neither タグ/タグ-ignore or ブランチ/ブランチ-ignore, the ワークフロー will run for イベント affecting either ブランチ or タグ. もし define both ブランチ/ブランチ-ignore and paths/paths-ignore, the ワークフロー will only run when both フィルターs are satisfied.

The ブランチ, ブランチ-ignore, タグ, and タグ-ignore keywords accept glob patterns that use characters like *, **, +, ?, ! and others to match more than one ブランチ or タグ name. If a name contains any of these characters and you want a literal match, you need to escape each of these special characters with \. 詳細については about glob patterns, see the Workflow syntax for GitHub Actions.

```yaml
Example: Including branches and tags
```

The patterns defined in ブランチ and タグ are evaluated against the Git ref's name. 例えば、 the following ワークフロー would run whenever there is a push イベント to:

A ブランチ named main (refs/heads/main)

A ブランチ named mona/octocat (refs/heads/mona/octocat)

A ブランチ whose name starts with releases/, like releases/10 (refs/heads/releases/10)

A タグ named v2 (refs/タグ/v2)

A タグ whose name starts with v1., like v1.9.1 (refs/タグ/v1.9.1)

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
Example: Excluding branches and tags
```

〜する場合、 a pattern matches the ブランチ-ignore or タグ-ignore pattern, the ワークフロー will not run. The patterns defined in ブランチ and タグ are evaluated against the Git ref's name. 例えば、 the following ワークフロー would run whenever there is a push イベント, unless the push イベント is to:

A ブランチ named mona/octocat (refs/heads/mona/octocat)

A ブランチ whose name matches releases/**-alpha, like releases/beta/3-alpha (refs/heads/releases/beta/3-alpha)

A タグ named v2 (refs/タグ/v2)

A タグ whose name starts with v1., like v1.9 (refs/タグ/v1.9)

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
Example: Including and excluding branches and tags
```

次のことができます't use ブランチ and ブランチ-ignore to フィルター the same イベント in a single ワークフロー. Similarly, you can't use タグ and タグ-ignore to フィルター the same イベント in a single ワークフロー. もし want to both include and exclude ブランチ or タグ patterns for a single イベント, use the ブランチ or タグ フィルター along with the ! character to indicate which ブランチ or タグ should be excluded.

もし define a ブランチ with the ! character, you must also define at least one ブランチ without the ! character. もし only want to exclude ブランチ, use ブランチ-ignore instead. Similarly, if you define a タグ with the ! character, you must also define at least one タグ without the ! character. もし only want to exclude タグ, use タグ-ignore instead.

The order that you define patterns matters.

A matching negative pattern (prefixed with !) after a positive match will exclude the Git ref.

A matching positive pattern after a negative match will include the Git ref again.

The following ワークフロー will run on pushes to releases/10 or releases/beta/mona, but not on releases/10-alpha or releases/beta/3-alpha because the negative pattern !releases/**-alpha follows the positive pattern.

```yaml
on:
  push:
    branches:
      - 'releases/**'
      - '!releases/**-alpha'
```

on.<push|pull_request|pull_request_target>.<paths|paths-ignore>

〜する場合、 using the push and pull_request イベント, you can configure a ワークフロー to run based on what file paths are changed. Path フィルターs are not evaluated for pushes of タグ.

Use the paths フィルター when you want to include file path patterns or when you want to both include and exclude file path patterns. Use the paths-ignore フィルター when you only want to exclude file path patterns. 次のことができますnot use both the paths and paths-ignore フィルターs for the same イベント in a ワークフロー. もし want to both include and exclude path patterns for a single イベント, use the paths フィルター prefixed with the ! character to indicate which paths should be excluded.

## メモ

The order that you define paths patterns matters:

A matching negative pattern (prefixed with !) after a positive match will exclude the path.

A matching positive pattern after a negative match will include the path again.

もし define both ブランチ/ブランチ-ignore and paths/paths-ignore, the ワークフロー will only run when both フィルターs are satisfied.

The paths and paths-ignore keywords accept glob patterns that use the * and ** wildcard characters to match more than one path name. 詳細については, see the Workflow syntax for GitHub Actions.

```yaml
Example: Including paths
```

If at least one path matches a pattern in the paths フィルター, the ワークフロー runs. 例えば、 the following ワークフロー would run anytime you push a JavaScript file (.js).

```yaml
on:
  push:
    paths:
      - '**.js'
```

If a ワークフロー is skipped due to path フィルターing, ブランチ フィルターing, or a commit message, then checks associated with that ワークフロー will remain in a "Pending" state. A pull request that requires those checks to be successful will be blocked from merging.

```yaml
Example: Excluding paths
```

〜する場合、 all the path names match patterns in paths-ignore, the ワークフロー will not run. If any path names do not match patterns in paths-ignore, even if some path names match the patterns, the ワークフロー will run.

A ワークフロー with the following path フィルター will only run on push イベント that include at least one file outside the docs directory at the root of the repository.

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
Example: Including and excluding paths
```

次のことができますnot use paths and paths-ignore to フィルター the same イベント in a single ワークフロー. もし want to both include and exclude path patterns for a single イベント, use the paths フィルター prefixed with the ! character to indicate which paths should be excluded.

もし define a path with the ! character, you must also define at least one path without the ! character. もし only want to exclude paths, use paths-ignore instead.

The order that you define paths patterns matters:

A matching negative pattern (prefixed with !) after a positive match will exclude the path.

A matching positive pattern after a negative match will include the path again.

This example runs anytime the push イベント includes a file in the sub-project directory or its subdirectories, unless the file is in the sub-project/docs directory. 例えば、 a push that changed sub-project/index.js or sub-project/src/index.js will trigger a ワークフロー run, but a push changing only sub-project/docs/readme.md will not.

```yaml
on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
```

## Git diff comparisons

## メモ

もし push more than 1,000 commits, or if GitHub does not generate the diff due to a timeout, the ワークフロー will always run.

The フィルター determines if a ワークフロー should run by evaluating the changed files and running them against the paths-ignore or paths list. If there are no files changed, the ワークフロー will not run.

GitHub generates the list of changed files using two-dot diffs for pushes and three-dot diffs for pull requests:

Pull requests: Three-dot diffs are a comparison between the most recent version of the topic ブランチ and the commit where the topic ブランチ was last synced with the base ブランチ.

Pushes to existing ブランチ: A two-dot diff compares the head and base SHAs directly with each other.

Pushes to new ブランチ: A two-dot diff against the parent of the ancestor of the deepest commit pushed.

## メモ

Diffs are limited to 300 files. If there are files changed that aren't matched in the first 300 files returned by the フィルター, the ワークフロー will not run. You may need to create more specific フィルターs so that the ワークフロー will run automatically.

詳細については, see About comparing ブランチ in pull requests.

## `on.schedule`

次のことができます use on.schedule to define a time schedule for your ワークフロー.

Use POSIX cron syntax to schedule ワークフロー to run at specific times. By デフォルト, scheduled ワークフロー run in UTC. 次のことができます optionally specify a timezone using an IANA timezone string for timezone-aware scheduling. Scheduled ワークフロー run on the latest commit on the デフォルト ブランチ. The shortest interval you can run scheduled ワークフロー is once every 5 minutes.

## メモ

For schedules that set timezone to a time zone that observes daylight saving time (DST), during DST spring-forward transitions, scheduled ワークフロー in skipped hours advance to the next valid time. 例えば、 a 2:30 AM schedule advances to 3:00 AM.

Cron syntax has five fields separated by a space, and each field represents a unit of time.

┌───────────── minute (0 - 59)

│ ┌───────────── hour (0 - 23)

│ │ ┌───────────── day of the month (1 - 31)

│ │ │ ┌───────────── month (1 - 12 or JAN-DEC)

│ │ │ │ ┌───────────── day of the week (0 - 6 or SUN-SAT)

## │ │ │ │ │

## * * * * *

次のことができます use these operators in any of the five fields:

## Operator Description Example

* Any value 15 * * * * runs at every minute 15 of every hour of every day.

, Value list separator 2,10 4,5 * * * runs at minute 2 and 10 of the 4th and 5th hour of every day.

- Range of values 30 4-6 * * * runs at minute 30 of the 4th, 5th, and 6th hour.

/ Step values 20/15 * * * * runs every 15 minutes starting from minute 20 through 59 (minutes 20, 35, and 50).

This example triggers the ワークフロー to run at 5:30 AM in the America/New_York timezone every Monday through Friday:

```yaml
on:
  schedule:
    - cron: '30 5 * * 1-5'
      timezone: "America/New_York"
```

A single ワークフロー can be triggered by multiple schedule イベント. Access the schedule イベント that triggered the ワークフロー through the github.イベント.schedule コンテキスト. This example triggers the ワークフロー to run at 5:30 UTC every Monday-Thursday, and 17:30 UTC on Tuesdays and Thursdays, but skips the Not on Monday or Wednesday ステップ on Monday and Wednesday.

```yaml
on:
  schedule:
    - cron: '30 5 * * 1,3'
    - cron: '30 5,17 * * 2,4'
```

```yaml
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

詳細については about schedule イベント, see Events that trigger ワークフロー.

## `on.workflow_call`

Use on.ワークフロー_call to define the inputs and outputs for a reusable ワークフロー. 次のことができます also map the secrets that are available to the called ワークフロー. 詳細については on reusable ワークフロー, see Reuse ワークフロー.

## `on.workflow_call.inputs`

〜する場合、 using the ワークフロー_call keyword, you can optionally specify inputs that are passed to the called ワークフロー from the caller ワークフロー. 詳細については about the ワークフロー_call keyword, see Events that trigger ワークフロー.

In addition to the standard input parameters that are available, on.ワークフロー_call.inputs requires a type parameter. 詳細については, see on.ワークフロー_call.inputs.<input_id>.type.

If a デフォルト parameter is not set, the デフォルト value of the input is false for a boolean, 0 for a number, and "" for a string.

Within the called ワークフロー, you can use the inputs コンテキスト to refer to an input. 詳細については, see Contexts reference.

If a caller ワークフロー passes an input that is not specified in the called ワークフロー, this results in an error.

### Example of on.ワークフロー_call.inputs

```yaml
on:
  workflow_call:
    inputs:
      username:
        description: 'A username passed from the caller workflow'
        default: 'john-doe'
        required: false
        type: string
```

```yaml
jobs:
  print-username:
    runs-on: ubuntu-latest
```

```yaml
    steps:
      - name: Print the input name to STDOUT
        run: echo The username is ${{ inputs.username }}
```

詳細については, see Reuse ワークフロー.

on.ワークフロー_call.inputs.<input_id>.type

Required if input is defined for the on.ワークフロー_call keyword. The value of this parameter is a string specifying the data type of the input. This must be one of: boolean, number, or string.

## `on.workflow_call.outputs`

A map of outputs for a called ワークフロー. Called ワークフロー outputs are available to all downstream ジョブ in the caller ワークフロー. Each output has an identifier, an optional description, and a value. The value must be set to the value of an output from a ジョブ within the called ワークフロー.

In the example below, two outputs are defined for this reusable ワークフロー: ワークフロー_output1 and ワークフロー_output2. These are mapped to outputs called ジョブ_output1 and ジョブ_output2, both from a ジョブ called my_ジョブ.

### Example of on.ワークフロー_call.outputs

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

For information on how to reference a ジョブ output, see ジョブ.<ジョブ_id>.outputs. 詳細については, see Reuse ワークフロー.

## `on.workflow_call.secrets`

A map of the secrets that can be used in the called ワークフロー.

Within the called ワークフロー, you can use the secrets コンテキスト to refer to a secret.

## メモ

もし are passing the secret to a nested reusable ワークフロー, then you must use ジョブ.<ジョブ_id>.secrets again to pass the secret. 詳細については, see Reuse ワークフロー.

If a caller ワークフロー passes a secret that is not specified in the called ワークフロー, this results in an error.

### Example of on.ワークフロー_call.secrets

```yaml
on:
  workflow_call:
    secrets:
      access-token:
        description: 'A token passed from the caller workflow'
        required: false
```

```yaml
jobs:
```

```yaml
  pass-secret-to-action:
    runs-on: ubuntu-latest
    steps:
    # passing the secret to an action
      - name: Pass the received secret to an action
        uses: ./.github/actions/my-action
        with:
          token: ${{ secrets.access-token }}
```

```yaml
  # passing the secret to a nested reusable workflow
  pass-secret-to-workflow:
    uses: ./.github/workflows/my-workflow
    secrets:
       token: ${{ secrets.access-token }}
```

on.ワークフロー_call.secrets.<secret_id>

A string identifier to associate with the secret.

on.ワークフロー_call.secrets.<secret_id>.required

A boolean specifying whether the secret must be supplied.

on.ワークフロー_run.<ブランチ|ブランチ-ignore>

〜する場合、 using the ワークフロー_run イベント, you can specify what ブランチ the triggering ワークフロー must run on in order to trigger your ワークフロー.

The ブランチ and ブランチ-ignore フィルターs accept glob patterns that use characters like *, **, +, ?, ! and others to match more than one ブランチ name. If a name contains any of these characters and you want a literal match, you need to escape each of these special characters with \. 詳細については about glob patterns, see the Workflow syntax for GitHub Actions.

例えば、 a ワークフロー with the following trigger will only run when the ワークフロー named Build runs on a ブランチ whose name starts with releases/:

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [requested]
    branches:
      - 'releases/**'
```

A ワークフロー with the following trigger will only run when the ワークフロー named Build runs on a ブランチ that is not named canary:

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types: [requested]
    branches-ignore:
      - "canary"
```

次のことができますnot use both the ブランチ and ブランチ-ignore フィルターs for the same イベント in a ワークフロー. もし want to both include and exclude ブランチ patterns for a single イベント, use the ブランチ フィルター along with the ! character to indicate which ブランチ should be excluded.

The order that you define patterns matters.

A matching negative pattern (prefixed with !) after a positive match will exclude the ブランチ.

A matching positive pattern after a negative match will include the ブランチ again.

例えば、 a ワークフロー with the following trigger will run when the ワークフロー named Build runs on a ブランチ that is named releases/10 or releases/beta/mona but will not releases/10-alpha, releases/beta/3-alpha, or main.

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

〜する場合、 using the ワークフロー_dispatch イベント, you can optionally specify inputs that are passed to the ワークフロー.

This trigger only receives イベント when the ワークフロー file is on the デフォルト ブランチ.

## `on.workflow_dispatch.inputs`

The triggered ワークフロー receives the inputs in the inputs コンテキスト. 詳細については, see Contexts.

## メモ

The ワークフロー will also receive the inputs in the github.イベント.inputs コンテキスト. The information in the inputs コンテキスト and github.イベント.inputs コンテキスト is identical except that the inputs コンテキスト preserves Boolean values as Booleans instead of converting them to strings. The choice type resolves to a string and is a single selectable option.

The maximum number of top-level properties for inputs is 25 .

The maximum payload for inputs is 65,535 characters.

### Example of on.ワークフロー_dispatch.inputs

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
```

```yaml
jobs:
  print-tag:
    runs-on: ubuntu-latest
    if: ${{ inputs.print_tags }}
    steps:
      - name: Print the input tag to STDOUT
        run: echo  The tags are ${{ inputs.tags }}
```

on.ワークフロー_dispatch.inputs.<input_id>.required

A boolean specifying whether the input must be supplied.

on.ワークフロー_dispatch.inputs.<input_id>.type

The value of this parameter is a string specifying the data type of the input. This must be one of: boolean, choice, number, environment or string.

## `permissions`

次のことができます use 権限 to modify the デフォルト 権限 granted to the GITHUB_TOKEN, adding or removing access as required, so that you only allow the minimum required access. 詳細については, see Use GITHUB_TOKEN for authentication in ワークフロー.

次のことができます use 権限 either as a top-level key, to apply to all ジョブ in the ワークフロー, or within specific ジョブ. 〜する場合、 you add the 権限 key within a specific ジョブ, all actions and run commands within that ジョブ that use the GITHUB_TOKEN gain the access rights you specify. 詳細については, see ジョブ.<ジョブ_id>.権限.

Owners of an organization can restrict write access for the GITHUB_TOKEN at the repository level. 詳細については, see Disabling or limiting GitHub Actions for your organization.

〜する場合、 a ワークフロー is triggered by the pull_request_target イベント, the GITHUB_TOKEN is granted read/write repository permission, even when it is triggered from a public fork. 詳細については, see Events that trigger ワークフロー.

For each of the available 権限, shown in the table below, you can assign one of the access levels: read (if applicable), write, or none. write includes read. もし specify the access for any of these 権限, all of those that are not specified are set to none.

Available 権限 and details of what each allows an action to do:

## Permission Allows an action using GITHUB_TOKEN to

actions Work with GitHub Actions. 例えば、 actions: write permits an action to cancel a ワークフロー run. 詳細については, see Permissions required for GitHub Apps.

artifact-metadata Work with artifact metadata. 例えば、 artifact-metadata: write permits an action to create storage records on behalf of a build artifact. 詳細については, see REST API endpoints for artifact metadata.

attestations Work with artifact attestations. 例えば、 attestations: write permits an action to generate an artifact attestation for a build. 詳細については, see Using artifact attestations to establish provenance for builds

checks Work with check runs and check suites. 例えば、 checks: write permits an action to create a check run. 詳細については, see Permissions required for GitHub Apps.

code-quality Work with code quality. 例えば、 code-quality: write permits an action to upload code coverage reports. 詳細については, see About GitHub Code Quality.

contents Work with the contents of the repository. 例えば、 contents: read permits an action to list the commits, and contents: write allows the action to create a release. 詳細については, see Permissions required for GitHub Apps.

deployments Work with deployments. 例えば、 deployments: write permits an action to create a new deployment. 詳細については, see Permissions required for GitHub Apps.

discussions Work with GitHub Discussions. 例えば、 discussions: write permits an action to close or delete a discussion. 詳細については, see Using the GraphQL API for Discussions.

id-token Fetch an OpenID Connect (OIDC) token. This requires id-token: write. 詳細については, see OpenID Connect

issues Work with issues. 例えば、 issues: write permits an action to add a comment to an issue. 詳細については, see Permissions required for GitHub Apps.

models Generate AI inference responses with GitHub Models. 例えば、 models: read permits an action to use the GitHub Models inference API. See Prototyping with AI models.

packages Work with GitHub Packages. 例えば、 packages: write permits an action to upload and publish packages on GitHub Packages. 詳細については, see About 権限 for GitHub Packages.

pages Work with GitHub Pages. 例えば、 pages: write permits an action to request a GitHub Pages build. 詳細については, see Permissions required for GitHub Apps.

pull-requests Work with pull requests. 例えば、 pull-requests: write permits an action to add a label to a pull request. 詳細については, see Permissions required for GitHub Apps.

security-イベント Work with GitHub code scanning alerts. 例えば、 security-イベント: read permits an action to list the code scanning alerts for the repository, and security-イベント: write allows an action to update the status of a code scanning alert. 詳細については, see Repository 権限 for "Code scanning alerts".

For Dependabot alerts, use the vulnerability-alerts permission. Secret scanning alerts cannot be read with this permission and require a GitHub App or a personal access token. 詳細については, see Repository 権限 for "Secret scanning alerts" in "Permissions required for GitHub Apps."

statuses Work with commit statuses. 例えば、 statuses:read permits an action to list the commit statuses for a given reference. 詳細については, see Permissions required for GitHub Apps.

vulnerability-alerts Read Dependabot alerts. 例えば、 vulnerability-alerts: read permits an action to list Dependabot alerts for the repository. Only read and none are supported; write is not valid. 〜する場合、 write-all or read-all is used, vulnerability-alerts is automatically included as read. 詳細については, see Repository 権限 for "Dependabot alerts".

## Defining access for the GITHUB_TOKEN scopes

次のことができます define the access that the GITHUB_TOKEN will permit by specifying read, write, or none as the value of the available 権限 within the 権限 key.

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
```

```yaml
  security-events: read|write|none
  statuses: read|write|none
  vulnerability-alerts: read|none
```

もし specify the access for any of these 権限, all of those that are not specified are set to none.

次のことができます use the following syntax to define one of read-all or write-all access for all of the available 権限:

```yaml
permissions: read-all
permissions: write-all
```

次のことができます use the following syntax to disable 権限 for all of the available 権限:

```yaml
permissions: {}
```

## Changing the 権限 in a forked repository

次のことができます use the 権限 key to add and remove read 権限 for forked repositories, but typically you can't grant write access. The exception to this behavior is where an admin user has selected the Send write tokens to ワークフロー from pull requests option in the GitHub Actions settings. 詳細については, see Managing GitHub Actions settings for a repository.

## How 権限 are calculated for a ワークフロー ジョブ

The 権限 for the GITHUB_TOKEN are initially set to the デフォルト setting for the enterprise, organization, or repository. If the デフォルト is set to the restricted 権限 at any of these levels then this will apply to the relevant repositories. 例えば、 if you choose the restricted デフォルト at the organization level then all repositories in that organization will use the restricted 権限 as the デフォルト. The 権限 are then adjusted based on any configuration within the ワークフロー file, first at the ワークフロー level and then at the ジョブ level. Finally, if the ワークフロー was triggered by a pull request イベント other than pull_request_target from a forked repository, and the Send write tokens to ワークフロー from pull requests setting is not selected, the 権限 are adjusted to change any write 権限 to read only.

## Setting the GITHUB_TOKEN 権限 for all ジョブ in a ワークフロー

次のことができます specify 権限 at the top level of a ワークフロー, so that the setting applies to all ジョブ in the ワークフロー.

```yaml
Example: Setting the GITHUB_TOKEN permissions for an entire workflow
```

This example shows 権限 being set for the GITHUB_TOKEN that will apply to all ジョブ in the ワークフロー. All 権限 are granted read access.

```yaml
name: "My workflow"
```

```yaml
on: [ push ]
```

```yaml
permissions: read-all
```

```yaml
jobs:
```

## `...`

## Using the 権限 key for forked repositories

次のことができます use the 権限 key to add and remove read 権限 for forked repositories, but typically you can't grant write access. The exception to this behavior is where an admin user has selected the Send write tokens to ワークフロー from pull requests option in the GitHub Actions settings. 詳細については, see Managing GitHub Actions settings for a repository.

## Permissions for ワークフロー runs triggered by Dependabot

Workflow runs triggered by Dependabot pull requests run as if they are from a forked repository, and therefore use a read-only GITHUB_TOKEN. These ワークフロー runs cannot access any secrets. For information about strategies to keep these ワークフロー secure, see Secure use reference.

## `env`

A map of variables that are available to the ステップ of all ジョブ in the ワークフロー. 次のことができます also set variables that are only available to the ステップ of a single ジョブ or to a single ステップ. 詳細については, see ジョブ.<ジョブ_id>.env and ジョブ.<ジョブ_id>.ステップ[*].env.

Variables in the env map cannot be defined in terms of other variables in the map.

〜する場合、 more than one environment variable is defined with the same name, GitHub uses the most specific variable. 例えば、 an environment variable defined in a ステップ will override ジョブ and ワークフロー environment variables with the same name, while the ステップ executes. An environment variable defined for a ジョブ will override a ワークフロー variable with the same name, while the ジョブ executes.

### Example of env

```yaml
env:
  SERVER: production
```

## `defaults`

Use デフォルトs to create a map of デフォルト settings that will apply to all ジョブ in the ワークフロー. 次のことができます also set デフォルト settings that are only available to a ジョブ. 詳細については, see ジョブ.<ジョブ_id>.デフォルトs.

〜する場合、 more than one デフォルト setting is defined with the same name, GitHub uses the most specific デフォルト setting. 例えば、 a デフォルト setting defined in a ジョブ will override a デフォルト setting that has the same name defined in a ワークフロー.

## `defaults.run`

次のことができます use デフォルトs.run to provide デフォルト shell and working-directory options for all run ステップ in a ワークフロー. 次のことができます also set デフォルト settings for run that are only available to a ジョブ. 詳細については, see ジョブ.<ジョブ_id>.デフォルトs.run. 次のことができますnot use コンテキストs or 式s in this keyword.

〜する場合、 more than one デフォルト setting is defined with the same name, GitHub uses the most specific デフォルト setting. 例えば、 a デフォルト setting defined in a ジョブ will override a デフォルト setting that has the same name defined in a ワークフロー.

```yaml
Example: Set the default shell and working directory
defaults:
  run:
    shell: bash
    working-directory: ./scripts
```

## `defaults.run.shell`

Use shell to define the shell for a ステップ. This keyword can reference several コンテキストs. 詳細については, see Contexts.

## Supported platform shell parameter Description Command run internally

Linux / macOS unspecified The デフォルト shell on non-Windows platforms. 注意: this runs a different command to when bash is specified explicitly. If bash is not found in the path, this is treated as sh. bash -e {0}

All bash The デフォルト shell on non-Windows platforms with a fallback to sh. 〜する場合、 specifying a bash shell on Windows, the bash shell included with Git for Windows is used. bash --noprofile --norc -eo pipefail {0}

All pwsh The PowerShell Core. GitHub appends the extension .ps1 to your script name. pwsh -command ". '{0}'"

All python Executes the python command. python {0}

Linux / macOS sh The fallback behavior for non-Windows platforms if no shell is provided and bash is not found in the path. sh -e {0}

Windows cmd GitHub appends the extension .cmd to your script name and substitutes for {0}. %ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}"".

Windows pwsh This is the デフォルト shell used on Windows. The PowerShell Core. GitHub appends the extension .ps1 to your script name. もし r self-hosted Windows runner does not have PowerShell Core installed, then PowerShell Desktop is used instead. pwsh -command ". '{0}'".

Windows powershell The PowerShell Desktop. GitHub appends the extension .ps1 to your script name. powershell -command ". '{0}'".

〜する場合、 more than one デフォルト setting is defined with the same name, GitHub uses the most specific デフォルト setting. 例えば、 a デフォルト setting defined in a ジョブ will override a デフォルト setting that has the same name defined in a ワークフロー.

## `defaults.run.working-directory`

Use working-directory to define the working directory for the shell for a ステップ. This keyword can reference several コンテキストs. 詳細については, see Contexts.

## ヒント

Ensure the working-directory you assign exists on the runner before you run your shell in it. 〜する場合、 more than one デフォルト setting is defined with the same name, GitHub uses the most specific デフォルト setting. 例えば、 a デフォルト setting defined in a ジョブ will override a デフォルト setting that has the same name defined in a ワークフロー.

## `concurrency`

Use 同時実行制御 to ensure that only a single ジョブ or ワークフロー using the same 同時実行制御 group will run at a time. A 同時実行制御 group can be any string or 式. The 式 can only use github, inputs and vars コンテキストs. 詳細については about 式s, see Evaluate 式s in ワークフロー and actions.

次のことができます also specify 同時実行制御 at the ジョブ level. 詳細については, see ジョブ.<ジョブ_id>.同時実行制御.

This means that there can be at most one running ジョブ or ワークフロー in a 同時実行制御 group at any time. 〜する場合、 a concurrent ジョブ or ワークフロー is queued, if another ジョブ or ワークフロー using the same 同時実行制御 group in the repository is in progress, the queued ジョブ or ワークフロー will be pending. By デフォルト, any existing pending ジョブ or ワークフロー in the same 同時実行制御 group will be canceled and the new queued ジョブ or ワークフロー will take its place.

〜するには also cancel any currently running ジョブ or ワークフロー in the same 同時実行制御 group, specify cancel-in-progress: true. 〜するには conditionally cancel currently running ジョブ or ワークフロー in the same 同時実行制御 group, you can specify cancel-in-progress as an 式 with any of the allowed 式 コンテキストs.

〜するには allow more than one pending ジョブ or ワークフロー run to wait in the same 同時実行制御 group, use the optional queue property. The queue property accepts the following values:

single (デフォルト): At most one ジョブ or ワークフロー run can be pending in the 同時実行制御 group. 〜する場合、 a new ジョブ or ワークフロー run is queued, any existing pending ジョブ or ワークフロー run in the same group is canceled and replaced.

```yaml
max: Up to 100 jobs or workflow runs can be pending in the concurrency group. When the queue is full, any additional jobs or workflow runs are canceled.
```

The combination of queue: max and cancel-in-progress: true is not allowed and will result in a ワークフロー validation error.

## メモ

The 同時実行制御 group name is case insensitive. 例えば、 prod and Prod will be treated as the same 同時実行制御 group.

Jobs or ワークフロー runs in the same 同時実行制御 group are processed in first-in-first-out (FIFO) order according to the time each one started waiting on the 同時実行制御 group, not the time each ワークフロー was dispatched. Since the actual start time of a ジョブ or run may vary, ordering is not guaranteed.

```yaml
Example: Using concurrency and the default behavior
```

The デフォルト behavior of GitHub Actions is to allow multiple ジョブ or ワークフロー runs to run concurrently. The 同時実行制御 keyword allows you to control the 同時実行制御 of ワークフロー runs.

例えば、 you can use the 同時実行制御 keyword immediately after where trigger conditions are defined to limit the 同時実行制御 of entire ワークフロー runs for a specific ブランチ:

```yaml
on:
  push:
    branches:
      - main
```

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

次のことができます also limit the 同時実行制御 of ジョブ within a ワークフロー by using the 同時実行制御 keyword at the ジョブ level:

```yaml
on:
  push:
    branches:
      - main
```

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: example-group
      cancel-in-progress: true
Example: Concurrency groups
```

Concurrency groups provide a way to manage and limit the execution of ワークフロー runs or ジョブ that share the same 同時実行制御 key.

The 同時実行制御 key is used to group ワークフロー or ジョブ together into a 同時実行制御 group. 〜する場合、 you define a 同時実行制御 key, GitHub Actions ensures that only one ワークフロー or ジョブ with that key runs at any given time. If a new ワークフロー run or ジョブ starts with the same 同時実行制御 key, GitHub Actions will cancel any ワークフロー or ジョブ already running with that key. The 同時実行制御 key can be a hard-coded string, or it can be a dynamic 式 that includes コンテキスト variables.

It is possible to define 同時実行制御 conditions in your ワークフロー so that the ワークフロー or ジョブ is part of a 同時実行制御 group.

This means that when a ワークフロー run or ジョブ starts, GitHub will cancel any ワークフロー runs or ジョブ that are already in progress in the same 同時実行制御 group. This is useful in scenarios where you want to prイベント parallel runs for a certain set of a ワークフロー or ジョブ, such as the ones used for deployments to a sタグing environment, in order to prイベント actions that could cause conflicts or consume more resources than necessary.

In this example, ジョブ-1 is part of a 同時実行制御 group named sタグing_environment. This means that if a new run of ジョブ-1 is triggered, any runs of the same ジョブ in the sタグing_environment 同時実行制御 group that are already in progress will be cancelled.

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: staging_environment
      cancel-in-progress: true
Alternatively, using a dynamic expression such as concurrency: ci-${{ github.ref }} in your workflow means that the workflow or job would be part of a concurrency group named ci- followed by the reference of the branch or tag that triggered the workflow. In this example, if a new commit is pushed to the main branch while a previous run is still in progress, the previous run will be cancelled and the new one will start:
```

```yaml
on:
  push:
    branches:
      - main
```

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
Example: Queueing multiple pending runs
```

By デフォルト, only one ジョブ or ワークフロー run can be pending in a 同時実行制御 group at a time. 〜するには allow multiple runs to queue instead of being canceled, set queue: max. With queue: max, up to 100 ジョブ or ワークフロー runs can wait in the 同時実行制御 group; once the queue is full, any additional runs are canceled.

例えば、 the following ワークフロー queues deployments to the production environment, processing them one at a time in order based on when each run started waiting on the 同時実行制御 group:

```yaml
on:
  push:
    branches:
      - main
```

```yaml
concurrency:
  group: production-deploy
  queue: max
```

注意: queue: max cannot be combined with cancel-in-progress: true, because the two options describe conflicting behaviors for handling in-progress runs.

```yaml
Example: Using concurrency to cancel any in-progress job or run
```

〜するには use 同時実行制御 to cancel any in-progress ジョブ or run in GitHub Actions, you can use the 同時実行制御 key with the cancel-in-progress option set to true:

```yaml
concurrency:
  group: ${{ github.ref }}
  cancel-in-progress: true
```

注意: in this example, without defining a particular 同時実行制御 group, GitHub Actions will cancel any in-progress run of the ジョブ or ワークフロー.

```yaml
Example: Using a fallback value
```

もし build the group name with a property that is only defined for specific イベント, you can use a fallback value. 例えば、 github.head_ref is only defined on pull_request イベント. もし r ワークフロー responds to other イベント in addition to pull_request イベント, you will need to provide a fallback to avoid a syntax error. The following 同時実行制御 group cancels in-progress ジョブ or runs on pull_request イベント only; if github.head_ref is undefined, the 同時実行制御 group will fallback to the run ID, which is guaranteed to be both unique and defined for the run.

```yaml
concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
Example: Only cancel in-progress jobs or runs for the current workflow
```

もし have multiple ワークフロー in the same repository, 同時実行制御 group names must be unique across ワークフロー to avoid canceling in-progress ジョブ or runs from other ワークフロー. Otherwise, any previously in-progress or pending ジョブ will be canceled, regardless of the ワークフロー.

〜するには only cancel in-progress runs of the same ワークフロー, you can use the github.ワークフロー property to build the 同時実行制御 group:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
Example: Only cancel in-progress jobs on specific branches
```

もし would like to cancel in-progress ジョブ on certain ブランチ but not on others, you can use conditional 式s with cancel-in-progress. 例えば、 you can do this if you would like to cancel in-progress ジョブ on development ブランチ but not on release ブランチ.

〜するには only cancel in-progress runs of the same ワークフロー when not running on a release ブランチ, you can set cancel-in-progress to an 式 similar to the following:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !contains(github.ref, 'release/')}}
```

In this example, multiple pushes to a release/1.2.3 ブランチ would not cancel in-progress runs. Pushes to another ブランチ, such as main, would cancel in-progress runs.

## `jobs`

A ワークフロー run is made up of one or more ジョブ, which run in parallel by デフォルト. 〜するには run ジョブ sequentially, you can define dependencies on other ジョブ using the ジョブ.<ジョブ_id>.needs keyword.

Each ジョブ runs in a runner environment specified by runs-on.

次のことができます run an unlimited number of ジョブ as long as you are within the ワークフロー usage limits. 詳細については, see Billing and usage for GitHub-hosted runners and Actions limits for self-hosted runner usage limits.

もし need to find the unique identifier of a ジョブ running in a ワークフロー run, you can use the GitHub API. 詳細については, see REST API endpoints for GitHub Actions.

ジョブ.<ジョブ_id>

Use ジョブ.<ジョブ_id> to give your ジョブ a unique identifier. The key ジョブ_id is a string and its value is a map of the ジョブ's configuration data. You must replace <ジョブ_id> with a string that is unique to the ジョブ object. The <ジョブ_id> must start with a letter or _ and contain only alphanumeric characters, -, or _.

```yaml
Example: Creating jobs
```

In this example, two ジョブ have been created, and their ジョブ_id values are my_first_ジョブ and my_second_ジョブ.

```yaml
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```

ジョブ.<ジョブ_id>.name

Use ジョブ.<ジョブ_id>.name to set a name for the ジョブ, which is displayed in the GitHub UI.

ジョブ.<ジョブ_id>.権限

For a specific ジョブ, you can use ジョブ.<ジョブ_id>.権限 to modify the デフォルト 権限 granted to the GITHUB_TOKEN, adding or removing access as required, so that you only allow the minimum required access. 詳細については, see Use GITHUB_TOKEN for authentication in ワークフロー.

By specifying the permission within a ジョブ definition, you can configure a different set of 権限 for the GITHUB_TOKEN for each ジョブ, if required. Alternatively, you can specify the 権限 for all ジョブ in the ワークフロー. For information on defining 権限 at the ワークフロー level, see 権限.

For each of the available 権限, shown in the table below, you can assign one of the access levels: read (if applicable), write, or none. write includes read. もし specify the access for any of these 権限, all of those that are not specified are set to none.

Available 権限 and details of what each allows an action to do:

## Permission Allows an action using GITHUB_TOKEN to

actions Work with GitHub Actions. 例えば、 actions: write permits an action to cancel a ワークフロー run. 詳細については, see Permissions required for GitHub Apps.

artifact-metadata Work with artifact metadata. 例えば、 artifact-metadata: write permits an action to create storage records on behalf of a build artifact. 詳細については, see REST API endpoints for artifact metadata.

attestations Work with artifact attestations. 例えば、 attestations: write permits an action to generate an artifact attestation for a build. 詳細については, see Using artifact attestations to establish provenance for builds

checks Work with check runs and check suites. 例えば、 checks: write permits an action to create a check run. 詳細については, see Permissions required for GitHub Apps.

code-quality Work with code quality. 例えば、 code-quality: write permits an action to upload code coverage reports. 詳細については, see About GitHub Code Quality.

contents Work with the contents of the repository. 例えば、 contents: read permits an action to list the commits, and contents: write allows the action to create a release. 詳細については, see Permissions required for GitHub Apps.

deployments Work with deployments. 例えば、 deployments: write permits an action to create a new deployment. 詳細については, see Permissions required for GitHub Apps.

discussions Work with GitHub Discussions. 例えば、 discussions: write permits an action to close or delete a discussion. 詳細については, see Using the GraphQL API for Discussions.

id-token Fetch an OpenID Connect (OIDC) token. This requires id-token: write. 詳細については, see OpenID Connect

issues Work with issues. 例えば、 issues: write permits an action to add a comment to an issue. 詳細については, see Permissions required for GitHub Apps.

models Generate AI inference responses with GitHub Models. 例えば、 models: read permits an action to use the GitHub Models inference API. See Prototyping with AI models.

packages Work with GitHub Packages. 例えば、 packages: write permits an action to upload and publish packages on GitHub Packages. 詳細については, see About 権限 for GitHub Packages.

pages Work with GitHub Pages. 例えば、 pages: write permits an action to request a GitHub Pages build. 詳細については, see Permissions required for GitHub Apps.

pull-requests Work with pull requests. 例えば、 pull-requests: write permits an action to add a label to a pull request. 詳細については, see Permissions required for GitHub Apps.

security-イベント Work with GitHub code scanning alerts. 例えば、 security-イベント: read permits an action to list the code scanning alerts for the repository, and security-イベント: write allows an action to update the status of a code scanning alert. 詳細については, see Repository 権限 for "Code scanning alerts".

For Dependabot alerts, use the vulnerability-alerts permission. Secret scanning alerts cannot be read with this permission and require a GitHub App or a personal access token. 詳細については, see Repository 権限 for "Secret scanning alerts" in "Permissions required for GitHub Apps."

statuses Work with commit statuses. 例えば、 statuses:read permits an action to list the commit statuses for a given reference. 詳細については, see Permissions required for GitHub Apps.

vulnerability-alerts Read Dependabot alerts. 例えば、 vulnerability-alerts: read permits an action to list Dependabot alerts for the repository. Only read and none are supported; write is not valid. 〜する場合、 write-all or read-all is used, vulnerability-alerts is automatically included as read. 詳細については, see Repository 権限 for "Dependabot alerts".

## Defining access for the GITHUB_TOKEN scopes

次のことができます define the access that the GITHUB_TOKEN will permit by specifying read, write, or none as the value of the available 権限 within the 権限 key.

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
```

```yaml
  security-events: read|write|none
  statuses: read|write|none
  vulnerability-alerts: read|none
```

もし specify the access for any of these 権限, all of those that are not specified are set to none.

次のことができます use the following syntax to define one of read-all or write-all access for all of the available 権限:

```yaml
permissions: read-all
permissions: write-all
```

次のことができます use the following syntax to disable 権限 for all of the available 権限:

```yaml
permissions: {}
```

## Changing the 権限 in a forked repository

次のことができます use the 権限 key to add and remove read 権限 for forked repositories, but typically you can't grant write access. The exception to this behavior is where an admin user has selected the Send write tokens to ワークフロー from pull requests option in the GitHub Actions settings. 詳細については, see Managing GitHub Actions settings for a repository.

```yaml
Example: Setting the GITHUB_TOKEN permissions for one job in a workflow
```

This example shows 権限 being set for the GITHUB_TOKEN that will only apply to the ジョブ named stale. Write access is granted for the issues and pull-requests 権限. All other 権限 will have no access.

```yaml
jobs:
  stale:
    runs-on: ubuntu-latest
```

```yaml
    permissions:
      issues: write
      pull-requests: write
```

```yaml
    steps:
      - uses: actions/stale@v10
```

ジョブ.<ジョブ_id>.needs

Use ジョブ.<ジョブ_id>.needs to identify any ジョブ that must complete successfully before this ジョブ will run. It can be a string or array of strings. If a ジョブ fails or is skipped, all ジョブ that need it are skipped unless the ジョブ use a conditional 式 that causes the ジョブ to continue. If a run contains a series of ジョブ that need each other, a failure or skip applies to all ジョブ in the dependency chain from the point of failure or skip onwards. もし would like a ジョブ to run even if a ジョブ it is dependent on did not succeed, use the always() conditional 式 in ジョブ.<ジョブ_id>.if.

```yaml
Example: Requiring successful dependent jobs
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

In this example, ジョブ1 must complete successfully before ジョブ2 begins, and ジョブ3 waits for both ジョブ1 and ジョブ2 to complete.

The ジョブ in this example run sequentially:

```yaml
job1
job2
job3
Example: Not requiring successful dependent jobs
jobs:
  job1:
  job2:
    needs: job1
  job3:
    if: ${{ always() }}
    needs: [job1, job2]
```

In this example, ジョブ3 uses the always() conditional 式 so that it always runs after ジョブ1 and ジョブ2 have completed, regardless of whether they were successful. 詳細については, see Evaluate 式s in ワークフロー and actions.

ジョブ.<ジョブ_id>.if

次のことができます use the ジョブ.<ジョブ_id>.if conditional to prイベント a ジョブ from running unless a condition is met. 次のことができます use any supported コンテキスト and 式 to create a conditional. 詳細については on which コンテキストs are supported in this key, see Contexts reference.

## メモ

The ジョブ.<ジョブ_id>.if condition is evaluated before ジョブ.<ジョブ_id>.strategy.matrix is applied.

```yaml
When you use expressions in an if conditional, you can, optionally, omit the ${{ }} expression syntax because GitHub Actions automatically evaluates the if conditional as an expression. However, this exception does not apply everywhere.
```

```yaml
You must always use the ${{ }} expression syntax or escape with '', "", or () when the expression starts with !, since ! is reserved notation in YAML format. For example:
```

```yaml
if: ${{ ! startsWith(github.ref, 'refs/tags/') }}
```

詳細については, see Evaluate 式s in ワークフロー and actions.

```yaml
Example: Only run job for specific repository
```

This example uses if to control when the production-deploy ジョブ can run. It will only run if the repository is named octo-repo-prod and is within the octo-org organization. Otherwise, the ジョブ will be marked as skipped.

```yaml
YAML
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

ジョブ.<ジョブ_id>.runs-on

Use ジョブ.<ジョブ_id>.runs-on to define the type of machine to run the ジョブ on.

The destination machine can be either a GitHub-hosted runner, larger runner, or a self-hosted runner.

次のことができます target runners based on the labels assigned to them, or their group membership, or a combination of these.

次のことができます provide runs-on as:

A single string

A single variable containing a string

An array of strings, variables containing strings, or a combination of both

A key: value pair using the group or labels keys

もし specify an array of strings or variables, your ワークフロー will execute on any runner that matches all of the specified runs-on values. 例えば、 here the ジョブ will only run on a self-hosted runner that has the labels linux, x64, and gpu:

```yaml
runs-on: [self-hosted, linux, x64, gpu]
```

詳細については, see Choosing self-hosted runners.

次のことができます mix strings and variables in an array. For example:

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
```

```yaml
jobs:
  test:
    runs-on: [self-hosted, "${{ inputs.chosen-os }}"]
    steps:
    - run: echo Hello world!
```

もし would like to run your ワークフロー on multiple machines, use ジョブ.<ジョブ_id>.strategy.

## メモ

```yaml
Quotation marks are not required around simple strings like self-hosted, but they are required for expressions like "${{ inputs.chosen-os }}".
```

Choosing GitHub-hosted runners

もし use a GitHub-hosted runner, each ジョブ runs in a fresh instance of a runner image specified by runs-on.

The value for runs-on, when you are using a GitHub-hosted runner, is a runner label or the name of a runner group. The labels for the standard GitHub-hosted runners are shown in the following tables.

詳細については, see GitHub-hosted runners.

Standard GitHub-hosted runners for public repositories

For public repositories, ジョブ using the ワークフロー labels shown in the table below will run with the associated specifications. With the exception of single-CPU runners, each GitHub-hosted runner is a new virtual machine (VM) hosted by GitHub. Single-CPU runners are hosted in a container on a shared VM—see GitHub-hosted runners reference. Use of the standard GitHub-hosted runners is free and unlimited on public repositories.

Virtual machine / container Processor (CPU) Memory (RAM) Storage (SSD) Architecture Workflow label

## Linux 1 5 GB 14 GB x64 ubuntu-slim

Linux 4 16 GB 14 GB x64 ubuntu-latest, ubuntu-24.04, ubuntu-22.04

## Windows 4 16 GB 14 GB x64 windows-latest, windows-2025, windows-2025-vs2026, windows-2022

Linux 4 16 GB 14 GB arm64 ubuntu-24.04-arm, ubuntu-22.04-arm

## Windows 4 16 GB 14 GB arm64 windows-11-arm

## macOS 4 14 GB 14 GB Intel macos-15-intel, macos-26-intel

macOS 3 (M1) 7 GB 14 GB arm64 macos-latest, macos-14, macos-15, macos-26

Standard GitHub-hosted runners for private repositories

For private repositories, ジョブ using the ワークフロー labels shown in the table below will run on virtual machines with the associated specifications. These runners use your GitHub account's allotment of free minutes, and are then charged at the per minute rates. See Actions runner pricing.

Virtual Machine Processor (CPU) Memory (RAM) Storage (SSD) Architecture Workflow label

## Linux 1 5 GB 14 GB x64 ubuntu-slim

Linux 2 8 GB 14 GB x64 ubuntu-latest, ubuntu-24.04, ubuntu-22.04

## Windows 2 8 GB 14 GB x64 windows-latest, windows-2025, windows-2022

Linux 2 8 GB 14 GB arm64 ubuntu-24.04-arm, ubuntu-22.04-arm

## Windows 2 8 GB 14 GB arm64 windows-11-arm

## macOS 4 14 GB 14 GB Intel macos-15-intel, macos-26-intel

macOS 3 (M1) 7 GB 14 GB arm64 macos-latest, macos-14, macos-15, macos-26

In addition to the standard GitHub-hosted runners, GitHub offers customers on GitHub Team and GitHub Enterprise Cloud plans a range of managed virtual machines with advanced features - for example, more cores and disk space, GPU-powered machines, and ARM-powered machines. 詳細については, see Larger runners.

## メモ

The -latest runner images are the latest stable images that GitHub provides, and might not be the most recent version of the operating system available from the operating system vendor.

## 警告

Beta and Deprecated Images are provided "as-is", "with all faults" and "as available" and are excluded from the service level agreement and warranty. Beta Images may not be covered by customer support.

```yaml
Example: Specifying an operating system
runs-on: ubuntu-latest
```

詳細については, see GitHub-hosted runners.

## Choosing self-hosted runners

〜するには specify a self-hosted runner for your ジョブ, configure runs-on in your ワークフロー file with self-hosted runner labels.

Self-hosted runners may have the self-hosted label. 〜する場合、 setting up a self-hosted runner, by デフォルト we will include the label self-hosted. You may pass in the --no-デフォルト-labels flag to prイベント the self-hosted label from being applied. Labels can be used to create targeting options for runners, such as operating system or architecture, we recommend providing an array of labels that begins with self-hosted (this must be listed first) and then includes additional labels as needed. 〜する場合、 you specify an array of labels, ジョブ will be queued on runners that have all the labels that you specify.

## メモ

Actions Runner Controller does not support the self-hosted label.

```yaml
Example: Using labels for runner selection
runs-on: [self-hosted, linux]
```

詳細については, see Self-hosted runners and Using self-hosted runners in a ワークフロー.

## Choosing runners in a group

次のことができます use runs-on to target runner groups, so that the ジョブ will execute on any runner that is a member of that group. For more granular control, you can also combine runner groups with labels.

Runner groups can only have larger runners or self-hosted runners as members.

```yaml
Example: Using groups to control where jobs are run
```

In this example, Ubuntu runners have been added to a group called ubuntu-runners. The runs-on key sends the ジョブ to any available runner in the ubuntu-runners group:

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
Example: Combining groups and labels
```

〜する場合、 you combine groups and labels, the runner must meet both requirements to be eligible to run the ジョブ.

In this example, a runner group called ubuntu-runners is populated with Ubuntu runners, which have also been assigned the label ubuntu-24.04-16core. The runs-on key combines group and labels so that the ジョブ is routed to any available runner within the group that also has a matching label:

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

ジョブ.<ジョブ_id>.snapshot

次のことができます use ジョブ.<ジョブ_id>.snapshot to generate a custom image.

Add the snapshot keyword to the ジョブ, using either the string syntax or mapping syntax as shown in Generating a custom image.

Each ジョブ that includes the snapshot keyword creates a separate image. 〜するには generate only one image or image version, include all ワークフロー ステップ in a single ジョブ. Each successful run of a ジョブ that includes the snapshot keyword creates a new version of that image.

詳細については, see Using custom images.

ジョブ.<ジョブ_id>.environment

Use ジョブ.<ジョブ_id>.environment to define the environment that the ジョブ references.

次のことができます provide the environment as only the environment name, or as an environment object with the name and url. The URL maps to environment_url in the deployments API. 詳細については about the deployments API, see REST API endpoints for repositories.

## メモ

All deployment protection rules must pass before a ジョブ referencing the environment is sent to a runner. 詳細については, see Managing environments for deployment.

```yaml
Example: Using a single environment name
environment: staging_environment
Example: Using environment name and URL
environment:
  name: production_environment
  url: https://github.com
```

The value of url can be an 式. Allowed 式 コンテキストs: github, inputs, vars, needs, strategy, matrix, ジョブ, runner, env, and ステップ. 詳細については about 式s, see Evaluate 式s in ワークフロー and actions.

```yaml
Example: Using output as URL
environment:
  name: production_environment
  url: ${{ steps.step_id.outputs.url_output }}
```

The value of name can be an 式. Allowed 式 コンテキストs: github, inputs, vars, needs, strategy, and matrix. 詳細については about 式s, see Evaluate 式s in ワークフロー and actions.

```yaml
Example: Using an expression as environment name
environment:
  name: ${{ github.ref_name }}
Example: Using an environment without creating a deployment
```

Set deployment to false to use an environment's secrets and variables without creating a deployment object.

```yaml
environment:
  name: testing
  deployment: false
```

Setting deployment: false is not compatible with custom deployment protection rules. 詳細については, see Deploying with GitHub Actions.

ジョブ.<ジョブ_id>.同時実行制御

次のことができます use ジョブ.<ジョブ_id>.同時実行制御 to ensure that only a single ジョブ or ワークフロー using the same 同時実行制御 group will run at a time. A 同時実行制御 group can be any string or 式. Allowed 式 コンテキストs: github, inputs, vars, needs, strategy, and matrix. 詳細については about 式s, see Evaluate 式s in ワークフロー and actions.

次のことができます also specify 同時実行制御 at the ワークフロー level. 詳細については, see 同時実行制御.

This means that there can be at most one running ジョブ or ワークフロー in a 同時実行制御 group at any time. 〜する場合、 a concurrent ジョブ or ワークフロー is queued, if another ジョブ or ワークフロー using the same 同時実行制御 group in the repository is in progress, the queued ジョブ or ワークフロー will be pending. By デフォルト, any existing pending ジョブ or ワークフロー in the same 同時実行制御 group will be canceled and the new queued ジョブ or ワークフロー will take its place.

〜するには also cancel any currently running ジョブ or ワークフロー in the same 同時実行制御 group, specify cancel-in-progress: true. 〜するには conditionally cancel currently running ジョブ or ワークフロー in the same 同時実行制御 group, you can specify cancel-in-progress as an 式 with any of the allowed 式 コンテキストs.

〜するには allow more than one pending ジョブ or ワークフロー run to wait in the same 同時実行制御 group, use the optional queue property. The queue property accepts the following values:

single (デフォルト): At most one ジョブ or ワークフロー run can be pending in the 同時実行制御 group. 〜する場合、 a new ジョブ or ワークフロー run is queued, any existing pending ジョブ or ワークフロー run in the same group is canceled and replaced.

```yaml
max: Up to 100 jobs or workflow runs can be pending in the concurrency group. When the queue is full, any additional jobs or workflow runs are canceled.
```

The combination of queue: max and cancel-in-progress: true is not allowed and will result in a ワークフロー validation error.

## メモ

The 同時実行制御 group name is case insensitive. 例えば、 prod and Prod will be treated as the same 同時実行制御 group.

Jobs or ワークフロー runs in the same 同時実行制御 group are processed in first-in-first-out (FIFO) order according to the time each one started waiting on the 同時実行制御 group, not the time each ワークフロー was dispatched. Since the actual start time of a ジョブ or run may vary, ordering is not guaranteed.

```yaml
Example: Using concurrency and the default behavior
```

The デフォルト behavior of GitHub Actions is to allow multiple ジョブ or ワークフロー runs to run concurrently. The 同時実行制御 keyword allows you to control the 同時実行制御 of ワークフロー runs.

例えば、 you can use the 同時実行制御 keyword immediately after where trigger conditions are defined to limit the 同時実行制御 of entire ワークフロー runs for a specific ブランチ:

```yaml
on:
  push:
    branches:
      - main
```

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

次のことができます also limit the 同時実行制御 of ジョブ within a ワークフロー by using the 同時実行制御 keyword at the ジョブ level:

```yaml
on:
  push:
    branches:
      - main
```

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: example-group
      cancel-in-progress: true
Example: Concurrency groups
```

Concurrency groups provide a way to manage and limit the execution of ワークフロー runs or ジョブ that share the same 同時実行制御 key.

The 同時実行制御 key is used to group ワークフロー or ジョブ together into a 同時実行制御 group. 〜する場合、 you define a 同時実行制御 key, GitHub Actions ensures that only one ワークフロー or ジョブ with that key runs at any given time. If a new ワークフロー run or ジョブ starts with the same 同時実行制御 key, GitHub Actions will cancel any ワークフロー or ジョブ already running with that key. The 同時実行制御 key can be a hard-coded string, or it can be a dynamic 式 that includes コンテキスト variables.

It is possible to define 同時実行制御 conditions in your ワークフロー so that the ワークフロー or ジョブ is part of a 同時実行制御 group.

This means that when a ワークフロー run or ジョブ starts, GitHub will cancel any ワークフロー runs or ジョブ that are already in progress in the same 同時実行制御 group. This is useful in scenarios where you want to prイベント parallel runs for a certain set of a ワークフロー or ジョブ, such as the ones used for deployments to a sタグing environment, in order to prイベント actions that could cause conflicts or consume more resources than necessary.

In this example, ジョブ-1 is part of a 同時実行制御 group named sタグing_environment. This means that if a new run of ジョブ-1 is triggered, any runs of the same ジョブ in the sタグing_environment 同時実行制御 group that are already in progress will be cancelled.

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    concurrency:
      group: staging_environment
      cancel-in-progress: true
Alternatively, using a dynamic expression such as concurrency: ci-${{ github.ref }} in your workflow means that the workflow or job would be part of a concurrency group named ci- followed by the reference of the branch or tag that triggered the workflow. In this example, if a new commit is pushed to the main branch while a previous run is still in progress, the previous run will be cancelled and the new one will start:
```

```yaml
on:
  push:
    branches:
      - main
```

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
Example: Queueing multiple pending runs
```

By デフォルト, only one ジョブ or ワークフロー run can be pending in a 同時実行制御 group at a time. 〜するには allow multiple runs to queue instead of being canceled, set queue: max. With queue: max, up to 100 ジョブ or ワークフロー runs can wait in the 同時実行制御 group; once the queue is full, any additional runs are canceled.

例えば、 the following ワークフロー queues deployments to the production environment, processing them one at a time in order based on when each run started waiting on the 同時実行制御 group:

```yaml
on:
  push:
    branches:
      - main
```

```yaml
concurrency:
  group: production-deploy
  queue: max
```

注意: queue: max cannot be combined with cancel-in-progress: true, because the two options describe conflicting behaviors for handling in-progress runs.

```yaml
Example: Using concurrency to cancel any in-progress job or run
```

〜するには use 同時実行制御 to cancel any in-progress ジョブ or run in GitHub Actions, you can use the 同時実行制御 key with the cancel-in-progress option set to true:

```yaml
concurrency:
  group: ${{ github.ref }}
  cancel-in-progress: true
```

注意: in this example, without defining a particular 同時実行制御 group, GitHub Actions will cancel any in-progress run of the ジョブ or ワークフロー.

```yaml
Example: Using a fallback value
```

もし build the group name with a property that is only defined for specific イベント, you can use a fallback value. 例えば、 github.head_ref is only defined on pull_request イベント. もし r ワークフロー responds to other イベント in addition to pull_request イベント, you will need to provide a fallback to avoid a syntax error. The following 同時実行制御 group cancels in-progress ジョブ or runs on pull_request イベント only; if github.head_ref is undefined, the 同時実行制御 group will fallback to the run ID, which is guaranteed to be both unique and defined for the run.

```yaml
concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
Example: Only cancel in-progress jobs or runs for the current workflow
```

もし have multiple ワークフロー in the same repository, 同時実行制御 group names must be unique across ワークフロー to avoid canceling in-progress ジョブ or runs from other ワークフロー. Otherwise, any previously in-progress or pending ジョブ will be canceled, regardless of the ワークフロー.

〜するには only cancel in-progress runs of the same ワークフロー, you can use the github.ワークフロー property to build the 同時実行制御 group:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
Example: Only cancel in-progress jobs on specific branches
```

もし would like to cancel in-progress ジョブ on certain ブランチ but not on others, you can use conditional 式s with cancel-in-progress. 例えば、 you can do this if you would like to cancel in-progress ジョブ on development ブランチ but not on release ブランチ.

〜するには only cancel in-progress runs of the same ワークフロー when not running on a release ブランチ, you can set cancel-in-progress to an 式 similar to the following:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !contains(github.ref, 'release/')}}
```

In this example, multiple pushes to a release/1.2.3 ブランチ would not cancel in-progress runs. Pushes to another ブランチ, such as main, would cancel in-progress runs.

ジョブ.<ジョブ_id>.outputs

次のことができます use ジョブ.<ジョブ_id>.outputs to create a map of outputs for a ジョブ. Job outputs are available to all downstream ジョブ that depend on this ジョブ. 詳細については on defining ジョブ dependencies, see ジョブ.<ジョブ_id>.needs.

Outputs can be a maximum of 1 MB per ジョブ. The total of all outputs in a ワークフロー run can be a maximum of 50 MB. Size is approximated based on UTF-16 encoding.

Job outputs containing 式s are evaluated on the runner at the end of each ジョブ. Outputs containing secrets are redacted on the runner and not sent to GitHub Actions.

If an output is skipped because it may contain a secret, you will see the following warning message: "Skip output {output.Key} since it may contain secret." 詳細については on how to handle secrets, please refer to the 例: Masking and passing a secret between ジョブ or ワークフロー.

〜するには use ジョブ outputs in a dependent ジョブ, you can use the needs コンテキスト. 詳細については, see Contexts reference.

```yaml
Example: Defining outputs for a job
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

## Using Job Outputs in a Matrix Job

Matrices can be used to generate multiple outputs of different names. 〜する場合、 using a matrix, ジョブ outputs will be combined from all ジョブ inside the matrix.

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

## 警告

Actions does not guarantee the order that matrix ジョブ will run in. Ensure that the output name is unique, otherwise the last matrix ジョブ that runs will override the output value.

ジョブ.<ジョブ_id>.env

A map of variables that are available to all ステップ in the ジョブ. 次のことができます set variables for the entire ワークフロー or an individual ステップ. 詳細については, see env and ジョブ.<ジョブ_id>.ステップ[*].env.

〜する場合、 more than one environment variable is defined with the same name, GitHub uses the most specific variable. 例えば、 an environment variable defined in a ステップ will override ジョブ and ワークフロー environment variables with the same name, while the ステップ executes. An environment variable defined for a ジョブ will override a ワークフロー variable with the same name, while the ジョブ executes.

### Example of ジョブ.<ジョブ_id>.env

```yaml
jobs:
  job1:
    env:
      FIRST_NAME: Mona
```

ジョブ.<ジョブ_id>.デフォルトs

Use ジョブ.<ジョブ_id>.デフォルトs to create a map of デフォルト settings that will apply to all ステップ in the ジョブ. 次のことができます also set デフォルト settings for the entire ワークフロー. 詳細については, see デフォルトs.

〜する場合、 more than one デフォルト setting is defined with the same name, GitHub uses the most specific デフォルト setting. 例えば、 a デフォルト setting defined in a ジョブ will override a デフォルト setting that has the same name defined in a ワークフロー.

ジョブ.<ジョブ_id>.デフォルトs.run

Use ジョブ.<ジョブ_id>.デフォルトs.run to provide デフォルト shell and working-directory to all run ステップ in the ジョブ.

次のことができます provide デフォルト shell and working-directory options for all run ステップ in a ジョブ. 次のことができます also set デフォルト settings for run for the entire ワークフロー. 詳細については, see デフォルトs.run.

These can be overridden at the ジョブ.<ジョブ_id>.デフォルトs.run and ジョブ.<ジョブ_id>.ステップ[*].run levels.

〜する場合、 more than one デフォルト setting is defined with the same name, GitHub uses the most specific デフォルト setting. 例えば、 a デフォルト setting defined in a ジョブ will override a デフォルト setting that has the same name defined in a ワークフロー.

ジョブ.<ジョブ_id>.デフォルトs.run.shell

Use shell to define the shell for a ステップ. This keyword can reference several コンテキストs. 詳細については, see Contexts.

## Supported platform shell parameter Description Command run internally

Linux / macOS unspecified The デフォルト shell on non-Windows platforms. 注意: this runs a different command to when bash is specified explicitly. If bash is not found in the path, this is treated as sh. bash -e {0}

All bash The デフォルト shell on non-Windows platforms with a fallback to sh. 〜する場合、 specifying a bash shell on Windows, the bash shell included with Git for Windows is used. bash --noprofile --norc -eo pipefail {0}

All pwsh The PowerShell Core. GitHub appends the extension .ps1 to your script name. pwsh -command ". '{0}'"

All python Executes the python command. python {0}

Linux / macOS sh The fallback behavior for non-Windows platforms if no shell is provided and bash is not found in the path. sh -e {0}

Windows cmd GitHub appends the extension .cmd to your script name and substitutes for {0}. %ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}"".

Windows pwsh This is the デフォルト shell used on Windows. The PowerShell Core. GitHub appends the extension .ps1 to your script name. もし r self-hosted Windows runner does not have PowerShell Core installed, then PowerShell Desktop is used instead. pwsh -command ". '{0}'".

Windows powershell The PowerShell Desktop. GitHub appends the extension .ps1 to your script name. powershell -command ". '{0}'".

〜する場合、 more than one デフォルト setting is defined with the same name, GitHub uses the most specific デフォルト setting. 例えば、 a デフォルト setting defined in a ジョブ will override a デフォルト setting that has the same name defined in a ワークフロー.

ジョブ.<ジョブ_id>.デフォルトs.run.working-directory

Use working-directory to define the working directory for the shell for a ステップ. This keyword can reference several コンテキストs. 詳細については, see Contexts.

## ヒント

Ensure the working-directory you assign exists on the runner before you run your shell in it. 〜する場合、 more than one デフォルト setting is defined with the same name, GitHub uses the most specific デフォルト setting. 例えば、 a デフォルト setting defined in a ジョブ will override a デフォルト setting that has the same name defined in a ワークフロー.

```yaml
Example: Setting default run step options for a job
jobs:
  job1:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: ./scripts
```

ジョブ.<ジョブ_id>.ステップ

A ジョブ contains a sequence of tasks called ステップ. Steps can run commands, run setup tasks, or run an action in your repository, a public repository, or an action published in a Docker registry. Not all ステップ run actions, but all actions run as a ステップ. Each ステップ runs in its own process in the runner environment and has access to the workspace and filesystem. Because ステップ run in their own process, changes to environment variables are not preserved between ステップ. GitHub provides built-in ステップ to set up and complete a ジョブ.

GitHub only displays the first 1,000 checks, however, you can run an unlimited number of ステップ as long as you are within the ワークフロー usage limits. 詳細については, see Billing and usage for GitHub-hosted runners and Actions limits for self-hosted runner usage limits.

### Example of ジョブ.<ジョブ_id>.ステップ

```yaml
name: Greeting from Mona
```

```yaml
on: push
```

```yaml
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

ジョブ.<ジョブ_id>.ステップ[*].id

A unique identifier for the ステップ. 次のことができます use the id to reference the ステップ in コンテキストs. 詳細については, see Contexts reference.

ジョブ.<ジョブ_id>.ステップ[*].if

次のことができます use the if conditional to prイベント a ステップ from running unless a condition is met. 次のことができます use any supported コンテキスト and 式 to create a conditional. 詳細については on which コンテキストs are supported in this key, see Contexts reference.

```yaml
When you use expressions in an if conditional, you can, optionally, omit the ${{ }} expression syntax because GitHub Actions automatically evaluates the if conditional as an expression. However, this exception does not apply everywhere.
```

```yaml
You must always use the ${{ }} expression syntax or escape with '', "", or () when the expression starts with !, since ! is reserved notation in YAML format. For example:
```

```yaml
if: ${{ ! startsWith(github.ref, 'refs/tags/') }}
```

詳細については, see Evaluate 式s in ワークフロー and actions.

```yaml
Example: Using contexts
```

This ステップ only runs when the イベント type is a pull_request and the イベント action is unassigned.

```yaml
steps:
  - name: My first step
    if: ${{ github.event_name == 'pull_request' && github.event.action == 'unassigned' }}
    run: echo This event is a pull request that had an assignee removed.
Example: Using status check functions
```

The my backup ステップ only runs when the previous ステップ of a ジョブ fails. 詳細については, see Evaluate 式s in ワークフロー and actions.

```yaml
steps:
  - name: My first step
    uses: octo-org/action-name@main
  - name: My backup step
    if: ${{ failure() }}
    uses: actions/heroku@1.0.0
Example: Using secrets
```

Secrets cannot be directly referenced in if: conditionals. Instead, consider setting secrets as ジョブ-level environment variables, then referencing the environment variables to conditionally run ステップ in the ジョブ.

```yaml
If a secret has not been set, the return value of an expression referencing the secret (such as ${{ secrets.SuperSecret }} in the example) will be an empty string.
```

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

詳細については, see Contexts reference and Using secrets in GitHub Actions.

ジョブ.<ジョブ_id>.ステップ[*].name

A name for your ステップ to display on GitHub.

ジョブ.<ジョブ_id>.ステップ[*].uses

Selects an action to run as part of a ステップ in your ジョブ. An action is a reusable unit of code. 次のことができます use an action defined in the same repository as the ワークフロー, a public repository, or in a published Docker container image.

We strongly recommend that you include the version of the action you are using by specifying a Git ref, SHA, or Docker タグ. もし don't specify a version, it could break your ワークフロー or cause unexpected behavior when the action owner publishes an update.

Using the commit SHA of a released action version is the safest for stability and security.

If the action publishes major version タグ, you should expect to receive critical fixes and security patches while still retaining compatibility. 注意: this behavior is at the discretion of the action's author.

Using the デフォルト ブランチ of an action may be convenient, but if someone releases a new major version with a breaking change, your ワークフロー could break.

Some actions require inputs that you must set using the with keyword. Review the action's README file to determine the inputs required.

Actions are either JavaScript files or Docker containers. If the action you're using is a Docker container you must run the ジョブ in a Linux environment. For more details, see runs-on.

```yaml
Example: Using versioned actions
steps:
  # Reference a specific commit
  - uses: actions/checkout@8f4b7f84864484a7bf31766abe9204da3cbe65b3
  # Reference the major version of a release
  - uses: actions/checkout@v6
  # Reference a specific version
  - uses: actions/checkout@v6.2.0
  # Reference a branch
  - uses: actions/checkout@main
Example: Using a public action
```

{owner}/{repo}@{ref}

次のことができます specify a ブランチ, ref, or SHA in a public GitHub repository.

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
Example: Using a public action in a subdirectory
```

{owner}/{repo}/{path}@{ref}

A subdirectory in a public GitHub repository at a specific ブランチ, ref, or SHA.

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/aws/ec2@main
Example: Using an action in the same repository as the workflow
```

./path/to/dir

The path to the directory that contains the action in your ワークフロー's repository. You must check out your repository before using the action.

### Example repository file structure:

|-- hello-world (repository)

| |__ .github

## | └── ワークフロー

| └── my-first-ワークフロー.yml

## | └── actions

## | |__ hello-world-action

| └── action.yml

The path is relative (./) to the デフォルト working directory (github.workspace, $GITHUB_WORKSPACE). If the action checks out the repository to a location different than the ワークフロー, the relative path used for local actions must be updated.

### Example ワークフロー file:

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
Example: Using a Docker Hub action
docker://{image}:{tag}
```

A Docker image published on Docker Hub.

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://alpine:3.8
Example: Using the GitHub Packages Container registry
docker://{host}/{image}:{tag}
```

A public Docker image in the GitHub Packages Container registry.

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://ghcr.io/OWNER/IMAGE_NAME
Example: Using a Docker public registry action
docker://{host}/{image}:{tag}
```

A Docker image in a public registry. This example uses the Google Container Registry at gcr.io.

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://gcr.io/cloud-builders/gradle
Example: Using an action inside a different private repository than the workflow
```

If the action is in an internal repository, or in a private repository configured to allow access from your ワークフロー's repository, you can reference the action directly. 詳細については, see Managing GitHub Actions settings for a repository and Managing GitHub Actions settings for a repository.

If the action isn't in a repository configured to allow access, you need to check out the repository and reference the action locally. Generate a personal access token and add the token as a secret. The following example shows this method for referencing an action. 詳細については, see Managing your personal access tokens and Using secrets in GitHub Actions.

Replace PERSONAL_ACCESS_TOKEN in the example with the name of your secret.

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

Alternatively, use a GitHub App instead of a personal access token in order to ensure your ワークフロー continues to run even if the personal access token owner leaves. 詳細については, see Making authenticated API requests with a GitHub App in a GitHub Actions ワークフロー.

ジョブ.<ジョブ_id>.ステップ[*].run

Runs command-line programs that do not exceed 21,000 characters using the operating system's shell. もし do not provide a name, the ステップ name will デフォルト to the text specified in the run command.

Commands run using non-login shells by デフォルト. 次のことができます choose a different shell and customize the shell used to run commands. 詳細については, see ジョブ.<ジョブ_id>.ステップ[*].shell.

Each run keyword represents a new process and shell in the runner environment. 〜する場合、 you provide multi-line commands, each line runs in the same shell. For example:

A single-line command:

```yaml
- name: Install Dependencies
  run: npm install
```

A multi-line command:

```yaml
- name: Clean install dependencies and build
  run: |
    npm ci
    npm run build
```

ジョブ.<ジョブ_id>.ステップ[*].working-directory

Using the working-directory keyword, you can specify the working directory of where to run the command.

```yaml
- name: Clean temp directory
  run: rm -rf *
  working-directory: ./temp
```

Alternatively, you can specify a デフォルト working directory for all run ステップ in a ジョブ, or for all run ステップ in the entire ワークフロー. 詳細については, see デフォルトs.run.working-directory and ジョブ.<ジョブ_id>.デフォルトs.run.working-directory.

次のことができます also use a run ステップ to run a script. 詳細については, see Adding scripts to your ワークフロー.

ジョブ.<ジョブ_id>.ステップ[*].shell

次のことができます override the デフォルト shell settings in the runner's operating system and the ジョブ's デフォルト using the shell keyword. 次のことができます use built-in shell keywords, or you can define a custom set of shell options. The shell command that is run internally executes a temporary file that contains the commands specified in the run keyword.

## Supported platform shell parameter Description Command run internally

Linux / macOS unspecified The デフォルト shell on non-Windows platforms. 注意: this runs a different command to when bash is specified explicitly. If bash is not found in the path, this is treated as sh. bash -e {0}

All bash The デフォルト shell on non-Windows platforms with a fallback to sh. 〜する場合、 specifying a bash shell on Windows, the bash shell included with Git for Windows is used. bash --noprofile --norc -eo pipefail {0}

All pwsh The PowerShell Core. GitHub appends the extension .ps1 to your script name. pwsh -command ". '{0}'"

All python Executes the python command. python {0}

Linux / macOS sh The fallback behavior for non-Windows platforms if no shell is provided and bash is not found in the path. sh -e {0}

Windows cmd GitHub appends the extension .cmd to your script name and substitutes for {0}. %ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}"".

Windows pwsh This is the デフォルト shell used on Windows. The PowerShell Core. GitHub appends the extension .ps1 to your script name. もし r self-hosted Windows runner does not have PowerShell Core installed, then PowerShell Desktop is used instead. pwsh -command ". '{0}'".

Windows powershell The PowerShell Desktop. GitHub appends the extension .ps1 to your script name. powershell -command ". '{0}'".

Alternatively, you can specify a デフォルト shell for all run ステップ in a ジョブ, or for all run ステップ in the entire ワークフロー. 詳細については, see デフォルトs.run.shell and ジョブ.<ジョブ_id>.デフォルトs.run.shell.

```yaml
Example: Running a command using Bash
steps:
  - name: Display the path
    shell: bash
    run: echo $PATH
Example: Running a command using Windows cmd
steps:
  - name: Display the path
    shell: cmd
    run: echo %PATH%
Example: Running a command using PowerShell Core
steps:
  - name: Display the path
    shell: pwsh
    run: echo ${env:PATH}
Example: Using PowerShell Desktop to run a command
steps:
  - name: Display the path
    shell: powershell
    run: echo ${env:PATH}
Example: Running an inline Python script
steps:
  - name: Display the path
    shell: python
    run: |
      import os
      print(os.environ['PATH'])
```

## Custom shell

次のことができます set the shell value to a template string using command [options] {0} [more_options]. GitHub interprets the first whitespace-delimited word of the string as the command, and inserts the file name for the temporary script at {0}.

For example:

```yaml
steps:
  - name: Display the environment variables and their values
    shell: perl {0}
    run: |
      print %ENV
```

The command used, perl in this example, must be installed on the runner.

For information about the software included on GitHub-hosted runners, see GitHub-hosted runners.

## Exit codes and error action preference

For built-in shell keywords, we provide the following デフォルトs that are executed by GitHub-hosted runners. You should use these guidelines when running shell scripts.

bash/sh:

By デフォルト, fail-fast behavior is enforced using set -e for both sh and bash. 〜する場合、 shell: bash is specified, -o pipefail is also applied to enforce early exit from pipelines that generate a non-zero exit status.

次のことができます take full control over shell parameters by providing a template string to the shell options. 例えば、 bash {0}.

sh-like shells exit with the exit code of the last command executed in a script, which is also the デフォルト behavior for actions. The runner will report the status of the ステップ as fail/succeed based on this exit code.

powershell/pwsh

Fail-fast behavior when possible. For pwsh and powershell built-in shell, we will prepend $ErrorActionPreference = 'stop' to script contents.

We append if ((Test-Path -LiteralPath variable:\LASTEXITCODE)) { exit $LASTEXITCODE } to powershell scripts so action statuses reflect the script's last exit code.

Users can always opt out by not using the built-in shell, and providing a custom shell option like: pwsh -File {0}, or powershell -Command "& '{0}'", depending on need.

## `cmd`

There doesn't seem to be a way to fully opt into fail-fast behavior other than writing your script to check each error code and respond accordingly. Because we can't actually provide that behavior by デフォルト, you need to write this behavior into your script.

cmd.exe will exit with the error level of the last program it executed, and it will return the error code to the runner. This behavior is internally consistent with the previous sh and pwsh デフォルト behavior and is the cmd.exe デフォルト, so this behavior remains intact.

ジョブ.<ジョブ_id>.ステップ[*].with

A map of the input parameters defined by the action. Each input parameter is a key/value pair. Input parameters are set as environment variables. The variable is prefixed with INPUT_ and converted to upper case.

Input parameters defined for a Docker container must use args. 詳細については, see ジョブ.<ジョブ_id>.ステップ[*].with.args.

### Example of ジョブ.<ジョブ_id>.ステップ[*].with

Defines the three input parameters (first_name, middle_name, and last_name) defined by the hello_world action. These input variables will be accessible to the hello-world action as INPUT_FIRST_NAME, INPUT_MIDDLE_NAME, and INPUT_LAST_NAME environment variables.

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

ジョブ.<ジョブ_id>.ステップ[*].with.args

A string that defines the inputs for a Docker container. GitHub passes the args to the container's ENTRYPOINT when the container starts up. An array of strings is not supported by this parameter. A single argument that includes spaces should be surrounded by double quotes "".

### Example of ジョブ.<ジョブ_id>.ステップ[*].with.args

```yaml
steps:
  - name: Explain why this job ran
    uses: octo-org/action-name@main
    with:
      entrypoint: /bin/echo
      args: The ${{ github.event_name }} event triggered this step.
```

The args are used in place of the CMD instruction in a Dockerfile. もし use CMD in your Dockerfile, use the guidelines ordered by preference:

Document required arguments in the action's README and omit them from the CMD instruction.

Use デフォルトs that allow using the action without specifying any args.

If the action exposes a --help flag, or something similar, use that as the デフォルト to make your action self-documenting.

ジョブ.<ジョブ_id>.ステップ[*].with.entrypoint

Overrides the Docker ENTRYPOINT in the Dockerfile, or sets it if one wasn't already specified. Unlike the Docker ENTRYPOINT instruction which has a shell and exec form, entrypoint keyword accepts only a single string defining the executable to be run.

### Example of ジョブ.<ジョブ_id>.ステップ[*].with.entrypoint

```yaml
steps:
  - name: Run a custom command
    uses: octo-org/action-name@main
    with:
      entrypoint: /a/different/executable
```

The entrypoint keyword is meant to be used with Docker container actions, but you can also use it with JavaScript actions that don't define any inputs.

ジョブ.<ジョブ_id>.ステップ[*].env

Sets variables for ステップ to use in the runner environment. 次のことができます also set variables for the entire ワークフロー or a ジョブ. 詳細については, see env and ジョブ.<ジョブ_id>.env.

〜する場合、 more than one environment variable is defined with the same name, GitHub uses the most specific variable. 例えば、 an environment variable defined in a ステップ will override ジョブ and ワークフロー environment variables with the same name, while the ステップ executes. An environment variable defined for a ジョブ will override a ワークフロー variable with the same name, while the ジョブ executes.

Public actions may specify expected variables in the README file. もし are setting a secret or sensitive value, such as a password or token, you must set secrets using the secrets コンテキスト. 詳細については, see Contexts reference.

### Example of ジョブ.<ジョブ_id>.ステップ[*].env

```yaml
steps:
  - name: My first action
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      FIRST_NAME: Mona
      LAST_NAME: Octocat
```

ジョブ.<ジョブ_id>.ステップ[*].continue-on-error

Prイベント a ジョブ from failing when a ステップ fails. Set to true to allow a ジョブ to pass when this ステップ fails.

ジョブ.<ジョブ_id>.ステップ[*].timeout-minutes

The maximum number of minutes to run the ステップ before killing the process. Maximum: 360 for both GitHub-hosted and self-hosted runners.

Fractional values are not supported. timeout-minutes must be a positive integer.

ジョブ.<ジョブ_id>.timeout-minutes

The maximum number of minutes to let a ジョブ run before GitHub automatically cancels it. Default: 360

If the timeout exceeds the ジョブ execution time limit for the runner, the ジョブ will be canceled when the execution time limit is met instead. 詳細については about ジョブ execution time limits, see Billing and usage for GitHub-hosted runners and Actions limits for self-hosted runner usage limits.

## メモ

The GITHUB_TOKEN expires when a ジョブ finishes or after a maximum of 24 hours. For self-hosted runners, the token may be the limiting factor if the ジョブ timeout is greater than 24 hours. 詳細については on the GITHUB_TOKEN, see Use GITHUB_TOKEN for authentication in ワークフロー.

ジョブ.<ジョブ_id>.strategy

Use ジョブ.<ジョブ_id>.strategy to use a matrix strategy for your ジョブ. A matrix strategy lets you use variables in a single ジョブ definition to automatically create multiple ジョブ runs that are based on the combinations of the variables. 例えば、 you can use a matrix strategy to test your code in multiple versions of a language or on multiple operating systems. 詳細については, see Running variations of ジョブ in a ワークフロー.

ジョブ.<ジョブ_id>.strategy.matrix

Use ジョブ.<ジョブ_id>.strategy.matrix to define a matrix of different ジョブ configurations. 詳細については, see Running variations of ジョブ in a ワークフロー.

A matrix will generate a maximum of 256 ジョブ per ワークフロー run. This limit applies to both GitHub-hosted and self-hosted runners.

The variables that you define become properties in the matrix コンテキスト, and you can reference the property in other areas of your ワークフロー file. In this example, you can use matrix.version and matrix.os to access the current value of version and os that the ジョブ is using. 詳細については, see Contexts reference.

By デフォルト, GitHub will maximize the number of ジョブ run in parallel depending on runner availability. The order of the variables in the matrix determines the order in which the ジョブ are created. The first variable you define will be the first ジョブ that is created in your ワークフロー run.

## Using a single-dimension matrix

The following ワークフロー defines the variable version with the values [10, 12, 14]. The ワークフロー will run three ジョブ, one for each value in the variable. Each ジョブ will access the version value through the matrix.version コンテキスト and pass the value as node-version to the actions/setup-node action.

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

## Using a multi-dimensional matrix

Specify multiple variables to create a multi-dimensional matrix. A ジョブ will run for each possible combination of the variables.

例えば、 the following ワークフロー specifies two variables:

## Two operating systems specified in the os variable

Three Node.js versions specified in the version variable

The ワークフロー will run six ジョブ, one for each combination of the os and version variables. Each ジョブ will set the runs-on value to the current os value and will pass the current version value to the actions/setup-node action.

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

A variable configuration in a matrix can be an array of objects. 例えば、 the following matrix produces 4 ジョブ with corresponding コンテキストs.

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

Each ジョブ in the matrix will have its own combination of os and node values, as shown below.

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

ジョブ.<ジョブ_id>.strategy.matrix.include

For each object in the include list, the key:value pairs in the object will be added to each of the matrix combinations if none of the key:value pairs overwrite any of the original matrix values. If the object cannot be added to any of the matrix combinations, a new matrix combination will be created instead. 注意: the original matrix values will not be overwritten, but added matrix values can be overwritten.

```yaml
Example: Expanding configurations
```

例えば、 the following ワークフロー will run four ジョブ, one for each combination of os and node. 〜する場合、 the ジョブ for the os value of windows-latest and node value of 16 runs, an additional variable called npm with the value of 6 will be included in the ジョブ.

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
Example: Adding configurations
```

例えば、 this matrix will run 10 ジョブ, one for each combination of os and version in the matrix, plus a ジョブ for the os value of windows-latest and version value of 17.

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

もし don't specify any matrix variables, all configurations under include will run. 例えば、 the following ワークフロー would run two ジョブ, one for each include entry. This lets you take advanタグe of the matrix strategy without having a fully populated matrix.

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

ジョブ.<ジョブ_id>.strategy.matrix.exclude

An excluded configuration only has to be a partial match for it to be excluded.

All include combinations are processed after exclude. This allows you to use include to add back combinations that were previously excluded.

ジョブ.<ジョブ_id>.strategy.fail-fast

次のことができます control how ジョブ failures are handled with ジョブ.<ジョブ_id>.strategy.fail-fast and ジョブ.<ジョブ_id>.continue-on-error.

ジョブ.<ジョブ_id>.strategy.fail-fast applies to the entire matrix. If ジョブ.<ジョブ_id>.strategy.fail-fast is set to true or its 式 evaluates to true, GitHub will cancel all in-progress and queued ジョブ in the matrix if any ジョブ in the matrix fails. This property デフォルトs to true.

ジョブ.<ジョブ_id>.continue-on-error applies to a single ジョブ. If ジョブ.<ジョブ_id>.continue-on-error is true, other ジョブ in the matrix will continue running even if the ジョブ with ジョブ.<ジョブ_id>.continue-on-error: true fails.

次のことができます use ジョブ.<ジョブ_id>.strategy.fail-fast and ジョブ.<ジョブ_id>.continue-on-error together. 例えば、 the following ワークフロー will start four ジョブ. For each ジョブ, continue-on-error is determined by the value of matrix.experimental. If any of the ジョブ with continue-on-error: false fail, all ジョブ that are in progress or queued will be cancelled. If the ジョブ with continue-on-error: true fails, the other ジョブ will not be affected.

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

ジョブ.<ジョブ_id>.strategy.max-parallel

By デフォルト, GitHub will maximize the number of ジョブ run in parallel depending on runner availability.

ジョブ.<ジョブ_id>.continue-on-error

ジョブ.<ジョブ_id>.continue-on-error applies to a single ジョブ. If ジョブ.<ジョブ_id>.continue-on-error is true, other ジョブ in the matrix will continue running even if the ジョブ with ジョブ.<ジョブ_id>.continue-on-error: true fails.

Prイベント a ワークフロー run from failing when a ジョブ fails. Set to true to allow a ワークフロー run to pass when this ジョブ fails.

```yaml
Example: Preventing a specific failing matrix job from failing a workflow run
```

次のことができます allow specific ジョブ in a ジョブ matrix to fail without failing the ワークフロー run. 例えば、 if you wanted to only allow an experimental ジョブ with node set to 15 to fail without failing the ワークフロー run.

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

ジョブ.<ジョブ_id>.container

## メモ

もし r ワークフロー use Docker container actions, ジョブ containers, or service containers, then you must use a Linux runner:

もし are using GitHub-hosted runners, you must use an Ubuntu runner.

もし are using self-hosted runners, you must use a Linux machine as your runner and Docker must be installed.

Use ジョブ.<ジョブ_id>.container to create a container to run any ステップ in a ジョブ that don't already specify a container. もし have ステップ that use both script and container actions, the container actions will run as sibling containers on the same network with the same volume mounts.

もし do not set a container, all ステップ will run directly on the host specified by runs-on unless a ステップ refers to an action configured to run in a container.

## メモ

The デフォルト shell for run ステップ inside a container is sh instead of bash. This can be overridden with ジョブ.<ジョブ_id>.デフォルトs.run or ジョブ.<ジョブ_id>.ステップ[*].shell.

```yaml
Example: Running a job within a container
```

```yaml
YAML
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

〜する場合、 you only specify a container image, you can omit the image keyword.

```yaml
jobs:
  container-test-job:
    runs-on: ubuntu-latest
    container: node:18
```

ジョブ.<ジョブ_id>.container.image

Use ジョブ.<ジョブ_id>.container.image to define the Docker image to use as the container to run the action. The value can be the Docker Hub image name or a registry name.

## メモ

Docker Hub normally imposes rate limits on both push and pull operations which will affect ジョブ on self-hosted runners. However, GitHub-hosted runners are not subject to these limits based on an agreement between GitHub and Docker.

ジョブ.<ジョブ_id>.container.credentials

If the image's container registry requires authentication to pull the image, you can use ジョブ.<ジョブ_id>.container.credentials to set a map of the username and password. The credentials are the same values that you would provide to the docker login command.

```yaml
Example: Defining credentials for a container registry
container:
  image: ghcr.io/owner/image
  credentials:
     username: ${{ github.actor }}
     password: ${{ secrets.github_token }}
```

ジョブ.<ジョブ_id>.container.env

Use ジョブ.<ジョブ_id>.container.env to set a map of environment variables in the container.

ジョブ.<ジョブ_id>.container.ports

Use ジョブ.<ジョブ_id>.container.ports to set an array of ports to expose on the container.

ジョブ.<ジョブ_id>.container.volumes

Use ジョブ.<ジョブ_id>.container.volumes to set an array of volumes for the container to use. 次のことができます use volumes to share data between services or other ステップ in a ジョブ. 次のことができます specify named Docker volumes, anonymous Docker volumes, or bind mounts on the host.

〜するには specify a volume, you specify the source and destination path:

<source>:<destinationPath>.

The <source> is a volume name or an absolute path on the host machine, and <destinationPath> is an absolute path in the container.

```yaml
Example: Mounting volumes in a container
volumes:
  - my_docker_volume:/volume_mount
  - /data/my_data
  - /source/directory:/destination/directory
```

ジョブ.<ジョブ_id>.container.options

Use ジョブ.<ジョブ_id>.container.options to configure additional Docker container resource options. For a list of options, see docker create options.

## 警告

The --network and --entrypoint options are not supported.

ジョブ.<ジョブ_id>.services

## メモ

もし r ワークフロー use Docker container actions, ジョブ containers, or service containers, then you must use a Linux runner:

もし are using GitHub-hosted runners, you must use an Ubuntu runner.

もし are using self-hosted runners, you must use a Linux machine as your runner and Docker must be installed.

Used to host service containers for a ジョブ in a ワークフロー. Service containers are useful for creating databases or cache services like Redis. The runner automatically creates a Docker network and manages the life cycle of the service containers.

もし configure your ジョブ to run in a container, or your ステップ uses container actions, you don't need to map ports to access the service or action. Docker automatically exposes all ports between containers on the same Docker user-defined bridge network. 次のことができます directly reference the service container by its hostname. The hostname is automatically mapped to the label name you configure for the service in the ワークフロー.

もし configure the ジョブ to run directly on the runner machine and your ステップ doesn't use a container action, you must map any required Docker service container ports to the Docker host (the runner machine). 次のことができます access the service container using localhost and the mapped port.

詳細については about the differences between networking service containers, see Communicating with Docker service containers.

```yaml
Example: Using localhost
This example creates two services: nginx and redis. When you specify the container port but not the host port, the container port is randomly assigned to a free port on the host. GitHub sets the assigned host port in the ${{job.services.<service_name>.ports}} context. In this example, you can access the service host ports using the ${{ job.services.nginx.ports['80'] }} and ${{ job.services.redis.ports['6379'] }} contexts.
```

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

ジョブ.<ジョブ_id>.services.<service_id>.image

The Docker image to use as the service container to run the action. The value can be the Docker Hub image name or a registry name.

If ジョブ.<ジョブ_id>.services.<service_id>.image is assigned an empty string, the service will not start. 次のことができます use this to set up conditional services, similar to the following example.

```yaml
services:
  nginx:
    image: ${{ options.nginx == true && 'nginx' || '' }}
```

ジョブ.<ジョブ_id>.services.<service_id>.credentials

If the image's container registry requires authentication to pull the image, you can use ジョブ.<ジョブ_id>.container.credentials to set a map of the username and password. The credentials are the same values that you would provide to the docker login command.

### Example of ジョブ.<ジョブ_id>.services.<service_id>.credentials

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

ジョブ.<ジョブ_id>.services.<service_id>.env

Sets a map of environment variables in the service container.

ジョブ.<ジョブ_id>.services.<service_id>.ports

Sets an array of ports to expose on the service container.

ジョブ.<ジョブ_id>.services.<service_id>.volumes

Sets an array of volumes for the service container to use. 次のことができます use volumes to share data between services or other ステップ in a ジョブ. 次のことができます specify named Docker volumes, anonymous Docker volumes, or bind mounts on the host.

〜するには specify a volume, you specify the source and destination path:

<source>:<destinationPath>.

The <source> is a volume name or an absolute path on the host machine, and <destinationPath> is an absolute path in the container.

### Example of ジョブ.<ジョブ_id>.services.<service_id>.volumes

```yaml
volumes:
  - my_docker_volume:/volume_mount
  - /data/my_data
  - /source/directory:/destination/directory
```

ジョブ.<ジョブ_id>.services.<service_id>.options

Additional Docker container resource options. For a list of options, see docker create options.

## 警告

The --network option is not supported.

ジョブ.<ジョブ_id>.services.<service_id>.command

Overrides the Docker image's デフォルト command (CMD). The value is passed as arguments after the image name in the docker create command. もし also specify entrypoint, command provides the arguments to that entrypoint.

### Example of ジョブ.<ジョブ_id>.services.<service_id>.command

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

ジョブ.<ジョブ_id>.services.<service_id>.entrypoint

Overrides the Docker image's デフォルト ENTRYPOINT. The value is a single string defining the executable to run. Use this when you need to replace the image's entrypoint entirely. 次のことができます combine entrypoint with command to pass arguments to the custom entrypoint.

### Example of ジョブ.<ジョブ_id>.services.<service_id>.entrypoint

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

ジョブ.<ジョブ_id>.uses

The location and version of a reusable ワークフロー file to run as a ジョブ. Use one of the following syntaxes:

{owner}/{repo}/.github/ワークフロー/{filename}@{ref} for reusable ワークフロー in public and private repositories.

./.github/ワークフロー/{filename} for reusable ワークフロー in the same repository.

In the first option, {ref} can be a SHA, a release タグ, or a ブランチ name. If a release タグ and a ブランチ have the same name, the release タグ takes precedence over the ブランチ name. Using the commit SHA is the safest option for stability and security. 詳細については, see Secure use reference.

もし use the second syntax option (without {owner}/{repo} and @{ref}) the called ワークフロー is from the same commit as the caller ワークフロー. Ref prefixes such as refs/heads and refs/タグ are not allowed. 次のことができますnot use コンテキストs or 式s in this keyword.

### Example of ジョブ.<ジョブ_id>.uses

```yaml
jobs:
  call-workflow-1-in-local-repo:
    uses: octo-org/this-repo/.github/workflows/workflow-1.yml@172239021f7ba04fe7327647b213799853a9eb89
  call-workflow-2-in-local-repo:
    uses: ./.github/workflows/workflow-2.yml
  call-workflow-in-another-repo:
    uses: octo-org/another-repo/.github/workflows/workflow.yml@v1
```

詳細については, see Reuse ワークフロー.

ジョブ.<ジョブ_id>.with

〜する場合、 a ジョブ is used to call a reusable ワークフロー, you can use with to provide a map of inputs that are passed to the called ワークフロー.

Any inputs that you pass must match the input specifications defined in the called ワークフロー.

Unlike ジョブ.<ジョブ_id>.ステップ[*].with, the inputs you pass with ジョブ.<ジョブ_id>.with are not available as environment variables in the called ワークフロー. Instead, you can reference the inputs by using the inputs コンテキスト.

### Example of ジョブ.<ジョブ_id>.with

```yaml
jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/called-workflow.yml@main
    with:
      username: mona
```

ジョブ.<ジョブ_id>.with.<input_id>

A pair consisting of a string identifier for the input and the value of the input. The identifier must match the name of an input defined by on.ワークフロー_call.inputs.<inputs_id> in the called ワークフロー. The data type of the value must match the type defined by on.ワークフロー_call.inputs.<input_id>.type in the called ワークフロー.

Allowed 式 コンテキストs: github, and needs.

ジョブ.<ジョブ_id>.secrets

〜する場合、 a ジョブ is used to call a reusable ワークフロー, you can use secrets to provide a map of secrets that are passed to the called ワークフロー.

Any secrets that you pass must match the names defined in the called ワークフロー.

### Example of ジョブ.<ジョブ_id>.secrets

```yaml
jobs:
  call-workflow:
    uses: octo-org/example-repo/.github/workflows/called-workflow.yml@main
    secrets:
      access-token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
```

ジョブ.<ジョブ_id>.secrets.inherit

Use the inherit keyword to pass all the calling ワークフロー's secrets to the called ワークフロー. This includes all secrets the calling ワークフロー has access to, namely organization, repository, and environment secrets. The inherit keyword can be used to pass secrets across repositories within the same organization, or across organizations within the same enterprise.

### Example of ジョブ.<ジョブ_id>.secrets.inherit

```yaml
on:
  workflow_dispatch:
```

```yaml
jobs:
  pass-secrets-to-workflow:
    uses: ./.github/workflows/called-workflow.yml
    secrets: inherit
on:
  workflow_call:
```

```yaml
jobs:
  pass-secret-to-action:
    runs-on: ubuntu-latest
    steps:
      - name: Use a repo or org secret from the calling workflow.
        run: echo ${{ secrets.CALLING_WORKFLOW_SECRET }}
```

ジョブ.<ジョブ_id>.secrets.<secret_id>

A pair consisting of a string identifier for the secret and the value of the secret. The identifier must match the name of a secret defined by on.ワークフロー_call.secrets.<secret_id> in the called ワークフロー.

Allowed 式 コンテキストs: github, needs, and secrets.

## Filter pattern cheat sheet

次のことができます use special characters in path, ブランチ, and タグ フィルターs.

*: Matches zero or more characters, but does not match the / character. 例えば、 Octo* matches Octocat.

**: Matches zero or more of any character.

?: Matches zero or one of the preceding character.

+: Matches one or more of the preceding character.

[] Matches one alphanumeric character listed in the brackets or included in ranges. Ranges can only include a-z, A-Z, and 0-9. 例えば、 the range[0-9a-z] matches any digit or lowercase letter. 例えば、 [CB]at matches Cat or Bat and [1-2]00 matches 100 and 200.

!: At the start of a pattern makes it negate previous positive patterns. It has no special meaning if not the first character.

The characters *, [, and ! are special characters in YAML. もし start a pattern with *, [, or !, you must enclose the pattern in quotes. Also, if you use a flow sequence with a pattern containing [ and/or ], the pattern must be enclosed in quotes.

## # Valid

```yaml
paths:
  - '**/README.md'
```

## # Invalid - creates a parse error that

# prイベント your ワークフロー from running.

```yaml
paths:
  - **/README.md
```

## # Valid

```yaml
branches: [ main, 'release/v[0-9].[0-9]' ]
```

## # Invalid - creates a parse error

```yaml
branches: [ main, release/v[0-9].[0-9] ]
```

詳細については about ブランチ, タグ, and path フィルター syntax, see on.<push>.<ブランチ|タグ>, on.<pull_request>.<ブランチ|タグ>, and on.<push|pull_request>.paths.

## ブランチとタグに一致させるパターン

## Pattern Description Example matches

feature/* The * wildcard matches any character, but does not match slash (/). feature/my-ブランチ

feature/your-ブランチ

feature/** The ** wildcard matches any character including slash (/) in ブランチ and タグ names. feature/beta-a/my-ブランチ

feature/your-ブランチ

feature/mona/the/octocat

## `main`

releases/mona-the-octocat Matches the exact name of a ブランチ or タグ name. main

releases/mona-the-octocat

'*' Matches all ブランチ and タグ names that don't contain a slash (/). The * character is a special character in YAML. 〜する場合、 you start a pattern with *, you must use quotes. main

## `releases`

'**' Matches all ブランチ and タグ names. This is the デフォルト behavior when you don't use a ブランチ or タグ フィルター. all/the/ブランチ

every/タグ

'*feature' The * character is a special character in YAML. 〜する場合、 you start a pattern with *, you must use quotes. mona-feature

## `feature`

## `ver-10-feature`

v2* Matches ブランチ and タグ names that start with v2. v2

## `v2.0`

## `v2.9`

v[12].[0-9]+.[0-9]+ Matches all semantic versioning ブランチ and タグ with major version 1 or 2. v1.10.1

## `v2.0.0`

## ファイルパスに一致させるパターン

Path patterns must match the whole path, and start from the repository's root.

## パターン 一致内容の説明 一致例

'*' The * wildcard matches any character, but does not match slash (/). The * character is a special character in YAML. 〜する場合、 you start a pattern with *, you must use quotes. README.md

## `server.rb`

'*.jsx?' The ? character matches zero or one of the preceding character. page.js

## `page.jsx`

'**' The ** wildcard matches any character including slash (/). This is the デフォルト behavior when you don't use a path フィルター. all/the/files.md

'*.js' The * wildcard matches any character, but does not match slash (/). Matches all .js files at the root of the repository. app.js

## `index.js`

'**.js' Matches all .js files in the repository. index.js

js/index.js

src/js/app.js

docs/* All files within the root of the docs directory only, at the root of the repository. docs/README.md

docs/file.txt

docs/** Any files in the docs directory and its subdirectories at the root of the repository. docs/README.md

docs/mona/octocat.txt

docs/**/*.md A file with a .md suffix anywhere in the docs directory. docs/README.md

docs/mona/hello-world.md

docs/a/markdown/file.md

'**/docs/**' Any files in a docs directory anywhere in the repository. docs/hello.md

dir/docs/my-file.txt

space/docs/plan/space.doc

'**/README.md' A README.md file anywhere in the repository. README.md

js/README.md

'**/*src/**' Any file in a folder with a src suffix anywhere in the repository. a/src/app.js

my-src/code/js/app.js

'**/*-post.md' A file with the suffix -post.md anywhere in the repository. my-post.md

path/their-post.md

'**/migrate-*.sql' A file with the prefix migrate- and suffix .sql anywhere in the repository. migrate-10909.sql

db/migrate-v1.0.sql

db/sept/migrate-v1.sql

'*.md'

'!README.md' Using an exclamation mark (!) in front of a pattern negates it. 〜する場合、 a file matches a pattern and also matches a negative pattern defined later in the file, the file will not be included. hello.md

## 一致しない例

## `README.md`

docs/hello.md

'*.md'

'!README.md'

README* Patterns are checked sequentially. A pattern that negates a previous pattern will re-include file paths. hello.md

## `README.md`

## `README.doc`

## ヘルプとサポート
