# GitAction 추가기능

## Checkout

-   깃헙 레포지토리 코드 가져오기
-   테스트 및 빌드 작업 수행

## Context

-   워크플로우를 실행하는 환경정보 제공

## Filter

-   이벤트에 특정한 조건 걸기
-   branch, path, tag

### Branch Filter

### Path Filter

### Tag Filter

## Timeout

-   기본값 : 360분
-   Level
    -   job
    -   step
    -   timeout-minutes

## Cache

-   의존성 설치시간 단축

## Artifact

-   워크플로우 실행중 생성된 파일들 모음
-   동일한 워크플로우 내에서 job사이에 데이터 공유

## Output

-   한 job에서 생성된 데이터 or 결과물 동일한 job의 다른 step과 공유
-   key-value 형태로 저장
-   다른 잡에서 아웃풋을 사용하려면
    -   needs
    -   job level outputs

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

## matrix

-   변수 기반으로 여러 job을 실행하는 기능

## If condition

-   조건 걸기
-   Level
    -   job
    -   step

## startwith, endwith, contains

-   문자열에 대한 조건 검사 수행

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
