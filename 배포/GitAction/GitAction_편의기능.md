# GitAction 추가기능

## Checkout

-   깃헙 레포지토리 코드 가져오기
-   테스트 및 빌드 작업 수행
-   기본적으로 actions/checkout은 트리거된 이벤트에 따라 브랜치 또는 병합된 상태의 코드를 가져옴

```yml
name: checkout # 깃헙 레포지토리를 가져와 작업을 수행 마켓플레이스에 정의된 공식 액션
on: workflow_dispatch

jobs:
    no-checkout: #실패하는 테스트 왜? checkout action 을 하지 않아 레포지토리를 가져오지 않음
        runs-on: ubuntu-latest
        steps:
            - name: check file list
              run: cat README.md

    checkout: # check out으로 repository를 가져옴
        runs-on: ubuntu-latest
        steps:
            - name: use checkout action
              uses: actions/checkout@v4
            - name: check file list
              run: cat README.md
```

## Context

-   워크플로우를 실행하는 환경정보 제공

```yml
name: context # context 워크플로우를 실행하는 환경에 대한 정보 더 유연한 워크플로우를 만들기 위해 dev에 push 받으면 workflow에서 1번job main에 push시 2번job 이런식으로
on: workflow_dispatch

jobs:
    context:
        runs-on: ubuntu-latest
        steps:
            - name: github context
              run: echo '${{ toJson(github)}}' # object 객체 json으로 변환   가지고 있는 정보를 보여줌
            - name: check github context
              run: |
                  echo ${{ github.repository }}
                  echo ${{ github.event_name}}
```

## Filter

-   이벤트에 특정한 조건 걸기
-   branch, path, tag

### Branch Filter

```yml
name: branch-filter
on:
    push:
        branches: ["dev"] #필터 사용

jobs:
    branch-filter:
        runs-on: ubuntu-latest
        steps:
            - name: echo
              run: echo hello
```

### Path Filter

```yml
# 특정 경로 파일이 변경될때 실행 ex my-app이라는 디렉토리 내에 파일이 업데이트 될때만 실행한다던가
name: path-filter
on:
    push:
        paths:
            - ".github/workflows/part1/*"
            - "!.github/workflows/part1/push.yml" # !는 제외 설정

jobs:
    path-filter:
        runs-on: ubuntu-latest
        steps:
            - name: echo hello
              run: echo hello
```

### Tag Filter

```yml
# 특정 태그에서 실행 ex) v1.0.0  push 에서만 가능
name: tag-filter
on:
    push:
        tags:
            - "v[0-9]+.[0-9]+.[0-9]"
jobs:
    tag-filter:
        runs-on: ubuntu-latest
        steps:
            - name: echo
              run: echo hello
```

## Timeout

-   기본값 : 360분
-   Level
    -   job
    -   step
    -   timeout-minutes

```yml
#timeout 무한루프 방지를 위한 timeout default 360분
name: timeout
on: push

jobs:
    timeout:
        runs-on: ubuntu-latest
        timeout-minutes: 1 #전체
        steps:
            - name: loop
              run: |
                  count=0
                  while true; do
                    echo "seconds : $count"
                    count=$(( count+1 ))
                    sleep 1
                  done
              timeout-minutes: 1 # setp 단계 time out
            - name: echo
              run: echo hello
```

## Cache

-   의존성 설치시간 단축

```yml
# cache 자주 사용하는 데이터를 빠르게 불러올 수 있도록 저장하는 기능 , 의존성 설치 시간 단축, 깃헙 마켓플레이스 공식 액션=
name: cache
on:
    push:
        paths:
            - "my-app/**"

jobs:
    cache:
        runs-on: ubuntu-latest
        steps:
            - name: checkout
              uses: actions/checkout@v4
            - name: setup-node
              uses: actions/setup-node@v3 # 18버전의 노드 인스톨
              with:
                  node-version: 18
            - name: Cache Node.js modules # with path key restore-keys input 값으로 삽입
              uses: actions/cache@v3
              with:
                  path: ~/.npm # 캐싱할 경로 (노드 모듈 지정)
                  key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }} # 캐시 키지정 운영체제와해시값으로 키값 설정
                  restore-keys:
                      | # 키가 정확한 일치가 없을경우 가장 가까운 캐시값 사용
                      ${{ runner.os }}-node-
            - name: Install dependencies
              run: |
                  cd my-app
                  npm ci
            - name: npm build
              run: |
                  cd my-app
                  npm run build
```

## Artifact

-   워크플로우 실행중 생성된 파일들 모음
-   동일한 워크플로우 내에서 job사이에 데이터 공유

```yml
name: artifact # 동일한 워크플로 내에서 job 사이에 데이터 공유 워크 플로우가 종료된 후에도, 데이터 유지 다운로드 가능 공식 액션 upload, download
on: push

jobs:
    upload-artifact:
        runs-on: ubuntu-latest
        steps:
            - name: echo
              run: echo hello-world > hello.txt
            - name: upload artifact
              uses: actions/upload-artifact@v3
              with:
                  name: artifact-test
                  path: ./hello.txt

    download-artifact:
        runs-on: ubuntu-latest
        needs: [upload-artifact]
        steps:
            - name: download artifact
              uses: actions/download-artifact@v3
              with:
                  name: artifact-test
                  path: ./
            - name: check
              run: cat hello.txt
```

## Output

-   한 job에서 생성된 데이터 or 결과물 동일한 job의 다른 step과 공유
-   key-value 형태로 저장
-   다른 잡에서 아웃풋을 사용하려면
    -   needs
    -   job level outputs

```yml
# 한 job에서 생성한 데이터를 동일한 job의 step 또는 다른 job과 데이터 공유  단순한 값 전달할 때 사용(파일 공유는 artifact사용) key-value 형식
name: output
on: push

jobs:
    create_output:
        runs-on: ubuntu-latest
        outputs:
            test: ${{ steps.check-output.outputs.test}}
        steps:
            - name: echo output
              id: check-output
              run: |
                  echo "test=hello" >> $GITHUB_OUTPUT
            - name: check output
              run: |
                  echo ${{ steps.check-output.outputs.test}}

    get-output:
        needs: [create_output]
        runs-on: ubuntu-latest
        steps:
            - name: get output
              run: |
                  echo ${{ needs.create_output.outputs.test}}
```

## environment variables

-   step 이나 job에서 사용할 수 있는 환경 변수
-   key-value 형태로 데이터 저장
-   동일한 job에서만 데이터 공유
-   Level
    -   workflow
    -   job
    -   step
-   step >> job >> workflow 더 세부적일수록 우선순위 높음

## environment

-   repo나 Organizations에 미리 정의가능 시크릿도 가능
-   Level
    -   organization
    -   repository
    -   environment
    -   environment >> repository >> organization 더 세부적일수록 우선순위 높음

```yml
# 특정 환경에서만 사용 가능한 환경 변수와 시크릿 관리
# 레포지토리에서 정의할 수 있음
# 환경 변수와 시크릿은 organization, repository, environment(가장 작은 범위) 3단계 레벨로 나뉨
name: environment
on: push

jobs:
    get-env:
        runs-on: ubuntu-latest
        steps: # vars 환경변수 사용 , secrets 시크릿 사용
            - name: check env & secret
              run: |
                  echo ${{ vars.level }}
                  echo ${{ secrets.key }}

    get-env-dev:
        runs-on: ubuntu-latest
        environment: dev
        steps:
            - name: check env & secret
              run: |
                  echo ${{ vars.level }}
                  echo ${{ secrets.key }}
```

```yml
# secret 민감한 데이터를 안전하게 저장해서 워크플로우에서 사용 민감 정보를 코드와 분리하여 사용 API key, 암호, 인증 토큰
# 깃헙에서 암호화, 로그에 기록되지 않고, 출력 시 마스킹, 접근제한
name: secret
on: push

jobs:
    get-secrets:
        runs-on: ubuntu-latest
        steps:
            - name: get secrets
              run: echo ${{ secrets.level }}
```

## matrix

-   변수 기반으로 여러 job을 실행하는 기능

```yml
# 변수 기반으로 여러 job 을 실행하는 기능
# matrix를 사용해서 하나의 잡을 구성하면, 여러 개의 잡을 실행하도록 할 수 있음
# 예시 os 별로 사용 하기 window러너, linux러너,  max러너
# 하나의 잡을 구성해서 같은 코드베이스에서 각 버전별로 테스트 구성
name: matrix
on: push

jobs:
    get-martix:
        strategy:
            matrix:
                os: [windows-latest, ubuntu-latest] # 가능한 조합으로 모두 실행
                version: [12, 14]
        runs-on: ${{ matrix.os }}
        steps:
            - name: check matrix
              run: |
                  echo ${{ matrix.os }}
                  echo ${{ matrix.version }}
```

## If condition

-   조건 걸기
-   Level
    -   job
    -   step

```yml
# 특정 조건이 충족될때 실행되도록 하는 데 사용
# filter는 workflow 트리거를 세밀하게 제어 ex push branchfilter
# ifcondition은 workflow가 트리거된 이후 job과 step 세밀하게 제어
name: if-1
on:
    push:
    workflow_dispatch:

jobs:
    job1:
        runs-on: ubuntu-latest #job level
        if: github.event_name == 'push'
        steps:
            - name: get event name
              run: echo ${{ github.event_name }}

    job2:
        runs-on: ubuntu-latest #job level
        if: github.event_name != 'push'
        steps:
            - name: get event name
              run: echo ${{ github.event_name }}

    job3:
        runs-on: ubuntu-latest
        steps:
            - name: get event name
              if: github.event_name == 'push'
              run: echo "PUSH"
            - name: get event name
              if: github.event_name != 'push'
              run: echo "NOT PUSH"
```

```yml
# 특정 job과 step을 강제로 실행 하는 if always()
name: if-2
on: push

jobs:
    job1:
        runs-on: ubuntu-latest
        steps:
            - name: exit 1
              run: exit 1
            - name: echo
              run: echo hello

    job2:
        needs: [job1]
        if: always()
        runs-on: ubuntu-latest
        steps:
            - name: echo
              run: echo hello
```

## startwith, endwith, contains

-   문자열에 대한 조건 검사 수행

```yml
# 문자열 처리 함수
# startWith, endWith, contatins(이친구는)
name: string-function
on: push

jobs:
    string-function:
        runs-on: ubuntu-latest
        steps:
            - name: startWith
              if: startsWith('github actions','git')
              run: echo "git"
            - name: startWith
              if: startsWith('github actions','test')
              run: echo "test"

            - name: endWith
              if: endsWith('github actions','ions')
              run: echo "ions"
            - name: endsWith
              if: endsWith('github actions','test')
              run: echo "gtestit"

            - name: contains
              if: contains('github actions','act')
              run: echo "contains act"
            - name: contains
              if: contains('github actions','git')
              run: echo "contains git"
```

## Concurrency

-   workflow 동시성을 제어
-   group
    -   서로 충돌하지 않게 작업하려는 작업 그룹을 식별하는 문자열
    -   두 개의 워크플로우 실행이 동일한 group값이라면 concurrency설정에 따라 관리
-   cancel-in-progress
    -   동일한 그룹에 속한 새 워크플로우가 시작될 때 아직 실행중인 워크플로우를 취소할 지 여부 결정
    -   cancel-in-progress:true 이전 워크플로우 취소
    -   cancel-in-progress:false 이전 워크플로우 종료할떄까지 대기
-   Level
    -   workflow
    -   job
-   배포 워크플로우가 여러개 동시에 실행되는 경우 중복 실행 방지
