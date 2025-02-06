# 도커 기본 명령어

docker --help

## 로컬 이미지 조회

```shell
docker image ls (이미지명)
docker image ls nginx
```

## 컨테이너실행

```shell
docker run -d --name {컨테이너명} 이미지명

docker	run	-d	--name	컨테이너1	nginx
docker	run	-d	--name	컨테이너2	nginx
docker	run	-d	--name	컨테이너3	nginx

docker run 이미지명 (실행명령)
docker run --name 컨테이너1 nginx	cat	usr/share/nginx/html/index.html
```

### Docker Run 명령어 주요 옵션

| 옵션              | 설명                                                                                        | 예시                                                  |
| ----------------- | ------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `-d`, `--detach`  | 컨테이너를 백그라운드(detached) 모드로 실행                                                 | `docker run -d image_name`                            |
| `--name`          | 컨테이너에 사용자 지정 이름을 부여                                                          | `docker run --name my_container image_name`           |
| `-p`, `--publish` | 호스트와 컨테이너 간 포트를 매핑 왼쪽은 호스트 포트, 오른쪽은 컨테이너 포트                 | `docker run -p 8080:80 image_name`                    |
| `-v`, `--volume`  | 호스트와 컨테이너 간 디렉토리(볼륨)를 공유하거나 데이터를 영속화할 때 사용합니다.           | `docker run -v /host/path:/container/path image_name` |
| `-e`, `--env`     | 컨테이너 내부에 환경 변수를 설정할 때 사용합니다.                                           | `docker run -e "ENV_VAR=value" image_name`            |
| `--rm`            | 컨테이너 종료 후 자동으로 컨테이너를 삭제합니다.                                            | `docker run --rm image_name`                          |
| `-it`             | 터미널 인터랙티브 모드로 실행하며, `-i`(표준 입력 유지)와 `-t`(TTY 할당)를 함께 사용합니다. | `docker run -it image_name /bin/bash`                 |
| `--network`       | 컨테이너가 연결될 네트워크를 지정합니다.                                                    | `docker run --network my_network image_name`          |
| `--entrypoint`    | 컨테이너 시작 시 기본 ENTRYPOINT를 오버라이드(대체)합니다.                                  | `docker run --entrypoint /bin/sh image_name`          |
| `--restart`       | 컨테이너의 재시작 정책을 설정합니다. (예: `no`, `on-failure`, `always`, `unless-stopped`)   | `docker run --restart always image_name`              |
| `-w`, `--workdir` | 컨테이너 내 작업 디렉토리를 지정합니다.                                                     | `docker run -w /app image_name`                       |
| `--cpus`          | 컨테이너에 할당할 CPU의 최대 사용량을 제한합니다.                                           | `docker run --cpus="1.5" image_name`                  |
| `--memory`        | 컨테이너에 할당할 메모리의 최대 사용량을 제한합니다.                                        | `docker run --memory="512m" image_name`               |

### 추가 설명

-   **포트 매핑(-p / -P):**  
    호스트의 특정 포트로 접근하면 컨테이너 내의 지정된 포트로 트래픽이 전달됩니다.
-   **볼륨 마운트(-v):**  
    컨테이너가 종료되어도 데이터를 보존하거나, 호스트와 데이터를 공유할 때 유용합니다.
-   **재시작 정책(--restart):**  
    컨테이너가 예기치 않게 종료되었을 때 자동으로 재시작할지 여부를 결정합니다.

## 실행중인 컨테이너 리스트 조회

```shell
docker ps

docker ps -a
```

## 실행중인 컨테이너 삭제

```shell
docker  rm -f
docker	rm	-f	컨테이너1	컨테이너2	컨테이너3

```

## 세부정보 조회

```shell
docker image inspect 이미지명
docker image inspect nginx

docker container inspect 컨테이너명
docker container inspect 컨테이너1
```

## 컨테이너 로그 조회

```
docker logs (컨테이너명)
```
