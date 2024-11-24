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

-   수동으로 트리거
-   default branch 에서만 동작

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

## needs

-   job 간에 종속성을 생성
-   하나의 job이 다른 job 또는 여러 job이 완료될 때 실행되도록 사용

## re-run

-   과거에 실행된 워크플로우를 재실행
-   트리거된 그 시점을 다시실행(수정된거 반영 x)

```yml
name: needs
on: push

jobs:
    job1:
        runs-on: ubuntu-latest
        steps:
            - name: echo
              run: echo "job1 done"
    job2:
        runs-on: ubuntu-latest
        needs: [job1] #job1 성공후 실행
        steps:
            - name: echo
              run: echo "job2 done"
    job3:
        runs-on: ubuntu-latest
        steps:
            - name: echo #exit 1 -> job 강제 실패
              run: |
                  echo "job3 failed"
                  exit 1
    job4:
        runs-on: ubuntu-latest
        needs: [job3] #3이 실패할꺼니까 이것도 당연히 실행 안됨  # 이럴때 사용하는게 re-run 과거에 실행된 워크플로우를 재실행, 성공 실패 여부와 상관없이 재실행 가능, 트리거된 그 시점(수정 반영 x)을 재실행
        steps: # 수정 파일 반영할꺼면 수정한 파일이 포함된 후 워크플로우를 실행해야
            - name: echo
              run: |
                  echo "job4 done"
```
