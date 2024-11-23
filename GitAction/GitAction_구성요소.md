# GitAction 구성요소

## Workflow (워크플로)

-   GitHub Actions에서 실행할 작업의 정의를 포함하는 YAML 파일입니다.
-   .github/workflows/ 디렉터리에 저장됩니다.
-   예: deploy.yml, test.yml

```yaml

name: CI Workflow
    on: [push, pull_request] # 이벤트 트리거 정의

jobs:
    build: # 하나의 Job 정의
      runs-on: ubuntu-latest
        steps:
          - name: Checkout repository
            uses: actions/checkout@v3

          - name: Run a script
            run: echo "Hello, GitHub Actions!"
```

## Events (이벤트)

-   워크플로를 트리거하는 조건입니다. 예를 들어, 코드가 푸시되거나 PR(Pull Request)이 열릴 때, 또는 특정 시간에 실행할 수 있습니다.
-   주요 이벤트:
    -   push: 브랜치에 커밋이 푸시되었을 때 실행.
    -   pull_request: PR이 열리거나 업데이트될 때 실행.
    -   schedule: 특정 시간에 실행.
    -   workflow_dispatch: 사용자가 수동으로 실행.

```yaml
on:
  push:
    branches: - main
  pull_request:
     branches: - main


```

## Jobs (잡)

-   하나의 워크플로 안에서 실행되는 독립적인 작업 단위입니다.
-   병렬 실행 가능하며, 특정 순서로 실행하도록 구성할 수도 있습니다.

```yaml
코드 복사
  jobs:
    build:
      runs-on: ubuntu-latest
        steps:
          - name: Install dependencies
             run: npm install

    test:
        runs-on: ubuntu-latest
          steps:
            - name: Run tests
               run: npm test`
```

## Steps (스텝)

-   Job 내부에서 실행되는 작업의 단위입니다.
-   여러 Step으로 Job을 구성합니다.
-   명령어를 실행하거나, 액션을 사용할 수 있습니다.

## Actions (액션)

-   GitHub Actions에서 제공하는 재사용 가능한 작업입니다.
-   예: actions/checkout, actions/setup-node.
-   자체 정의한 사용자 액션(Custom Actions)도 만들 수 있습니다.
