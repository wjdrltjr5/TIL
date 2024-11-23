# GitAction 이벤트

## push

push 되면 발생하는 이벤트

```yml
name: push-workflow
on: push
jobs:
    push-job:
        runs-on: ubuntu-latest
        steps:
            - name: step1
              run: echo "hello world"
            - name: step2
              run: | # | 멀티라인 커맨드 사용 하기위해
                  echo "hello world"
                  echo "github action"
```

## pull request

pull request는 세가지 타입이 있음 명시하지 않으면 3타입 모두 작동

-   opened
-   synchronize
-   reopened

```yml
name: pull-request-workflow
on:
    pull_request: # on pull_request 로만 작성하면 opened, synchronize, reopened
        types: [opened] # activity type
jobs:
    pull-request-job:
        runs-on: ubuntu-latest
        steps:
            - name: step1
              run: echo "hello world"
            - name: step2
              run: |
                  echo "hello world"
                  echo "github action"
```

## issue

이슈 는 default branch 에서만 작동함
pull request와 마찬가지로 타입을 정의하지 않으면 모든 타입에 작동
세밀하게 제어하기 위해 activity type 을 명시해야 함

```yaml
name: issues-workflow
on:
    issues:
        types: [opened]
jobs:
    issues-job:
        runs-on: ubuntu-latest
        steps:
            - name: step1
              run: echo "hello world"
            - name: step2
              run: |
                  echo "hello world"
                  echo "github action"
```

## issue comment

default branch 에서만 동작 이슈뿐 아니라 pull-request 코멘드도 동작

```yaml
name: issue-comment-workflow
on: issue_comment

jobs:
    pr-comment:
        if: ${{ github.event.issue.pull_request }}
        runs-on: ubuntu-latest
        steps:
            - name: pr comment
              run: echo ${{ github.event.issue.pull_request }}

    issue-comment:
        if: ${{ !github.event.issue.pull_request }}
        runs-on: ubuntu-latest
        steps:
            - name: issue comment
              run: echo ${{ github.event.issue.pull_request }}
```

## workflow dispatch

수동 실행

```yaml
name: workflow-dispatch
on:
    workflow_dispatch:
        inputs: # 인풋값 사용
            name:
                description: "set name"
                required: true
                default: "github-actions"
                type: string

            environment:
                description: "set env"
                required: true
                default: "dev"
                type: choice
                options:
                    - dev
                    - qa
                    - prod

jobs:
    workflow-dispatch-job:
        runs-on: ubuntu-latest
        steps:
            - name: step1
              run: echo hello world
            - name: step2
              run: |
                  echo hello world
                  echo github action
            - name: echo inputs
              run: |
                  echo ${{ inputs.name }}
                  echo ${{ inputs.environment }}
```

## schedule

```yml
name: schedule-workflow
on:
    schedule:
        - cron: "15 * * * *"

jobs:
    schedule-job:
        runs-on: ubuntu-latest
        steps:
            - name: schedule test
              run: echo hello world
```
