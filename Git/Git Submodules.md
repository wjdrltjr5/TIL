# Git Submodules

## 프로젝트 생성

-   private repo 생성
-   서브모듈을 적용하고 싶은 레포로 이동
-   서브모듈을 생성하고싶은 폴더로 이동

```shell
 git submodule add [git repo link] [submodule명]
```

-   생성후 .gitmodules 파일이 생김

```shell
git submodule init

git submodule update
```

## 사용시

1. 프로젝트 클론하기

```bash
git clone https://github.com/employment-rocket/rocket.git
```

진행하면 서브모듈 폴더는 생성이 되지만 내부에는 파일이 존재 하지 않음

1. 서브모듈 초기화 후 update 실시 (처음 1번만)

```bash
git submodule init

git submodule update
```

진행하면 서브모듈 폴더내 서브모듈이 가지고있는 파일 생성

`주의할점`

-   서브모듈 다른 repo이기 때문에 서브모듈 내부 파일 수정하고 코드베이스 repo를 push해도 반영되지 않음
-   서브모듈 수정 내용을 원격저장소에 반영하고 싶다면 따로 서브모듈에 해당하는 레포를 clone받아서 수정하고 push
-   서브모듈 repo를 따로 관리해주어야 함
-   원격저장소에 변경된 서브모듈 내용을 로컬에 반영하고 싶다면

```shell
git submodule update --remote
```
