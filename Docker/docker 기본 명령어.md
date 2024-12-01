# 도커 기본 명령어

docker --help

로컬 이미지 조회

```shell
docker image ls (이미지명)
docker image ls nginx
```

컨테이너실행

```shell
docker run -d --name {컨테이너명} 이미지명

docker	run	-d	--name	컨테이너1	nginx
docker	run	-d	--name	컨테이너2	nginx
docker	run	-d	--name	컨테이너3	nginx

docker run 이미지명 (실행명령)
docker run --name 컨테이너1 nginx	cat	usr/share/nginx/html/index.html
```

실행중인 컨테이너 리스트 조회

```shell
docker ps

docker ps -a
```

실행중인 컨테이너 삭제

```shell
docker  rm -f
docker	rm	-f	컨테이너1	컨테이너2	컨테이너3

```

세부정보 조회

```shell
docker image inspect 이미지명
docker image inspect nginx

docker container inspect 컨테이너명
docker container inspect 컨테이너1
```

컨테이너 로그 조회

```
docker logs (컨테이너명)
```
