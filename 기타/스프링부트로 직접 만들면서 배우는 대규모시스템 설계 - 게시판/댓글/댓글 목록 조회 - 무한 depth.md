# 댓글 목록 무한 depth 조회

-   댓글 1

    -   댓글 2
        -   댓글 6
    -   댓글 4
        -   댓글 5

-   댓글 3
    -   댓글 7

데이터가 가변적이고 복잡

-   각 depth 만큼 컬럼을 만들기

    -   각 depth는 무한할 수 있어야 한는데
    -   depth만큼 컬럼의 개수를 생성 및 관리 필요 -> 테이블 구조 복잡

-   문자열 컬럼 1개로 경로 정보 만들기

    -   Path Enumeration (경로 열거) 방식
    -   |1depth | 2depth | 3depth | ...
    -   N depth = N \* 5의 문자.

    -   11111 1

        -   11111 22222 2
            -   11111 22222 66666 6
        -   11111 44444 4
            -   11111 44444 55555 5

    -   33333 3
        -   33333 77777 7

-   문자열이기 때문에 꼭 문자열로 표시할 필요 x
-   DB에서 문자열 순서 사용시 고려
-   DB collation : 문자열을 정렬하고 비교하는 방법을 정의하는 설정

    -   대소문자 구분, 악센트 구분, 언어별 정렬 순서 등을 포함
        -   데이터베이스/ 테이블/ 컬럼별 설정
    -   기본 설정
        -   utf8mb4_0900_ai_ci
        -   utf8mb4 : 각 문자 최대 4바이트 utf-8 지원
        -   0900 : 정렬 방식 버전
        -   ai = 악센트 비구분
        -   ci = 대소문자 비구분
    -   62개의 문자 (0-9, A-Z, a-z)를 사용하려면

        -   utf8mb4_bin
        -   ```sql
            --컬럼에 적용
            path varchar(25) character set utf8mb4 collate utf8mb4_bin not null,
            ```

    -   ```sql
          -- 적용 확인 방법
          select table_name, column_name, collation_name
          from information_schema.COLUMNS
          where table_schema = {스키마명} and table_name = {테이블명};
        ```

-   구현 방법
-   00a0z : parentPath

    -   00z0z 00000
    -   00z0z 00001
    -   00z0z 00002 : childrenTopPath
        -   00z0z 00002 00000 : descendantsTopPath
    -   00z0z 00003

-   00z0z 00003 삽입을 위해서는
-   부모 path를 prefix를 가지는 모든 자손 댓글 에서 가장큰 path를 찾는다. -> 00z0z 00002 00000 : descendantsTopPath
-   신규 댓글의 depth x 5 만큼 자른다. 2 x 5 = 10 -> 00z0z 00002

```sql
-- descendantsTopPath 구하기
select path from comment
            where article_id = {article_id}
            and path > {parentPath} -- parent 본인은 미포함 검색
            and path like {parentPath%}
      order by path desc limit 1;
```

-   대부분의 현대적인 데이터베이스 시스템에서는 기본키에 대해 생성되는 B-Tree 인덱스가 양방향(ascending, descending) 스캔을 지원

    -   즉 위 인덱스에 index (article_id asc, path asc)가 이미 있다면
    -   위 쿼리도 Backward index scan을 이용
    -   자세한건 explain으로 확인
        -   복합 인덱스의 경우 상황이 다를 수 있음

-   childrenTopPath + 1 해서 신규 댓글 작성

    -   62개의 문자 (0-9, A-Z, a-z)를 사용하므로 +1 시 이러한 문자 반영
    -   00000 ~ zzzzz
    -   62진수로 구하기

-   하위 댓글 없이 최초 생성
    -   descendantsTopPath 가 없다면
    -   parentPath + 00000
-   이미 zzzzz까지 라면 예외를 던지던 문자열을 늘리던

## 댓글 목록 조회 페이지

```sql
-- 목록 조회
select * from (
  select comment_id
  from comment
  where article_id = {article_id}
  order by path asc
  limit {limit} offset {offset}
) t left join comment on t.comment_id = comment.comment_id;

-- 카운트 조회
select count(*) from (
  select comment_id from comment where article_id = {article_id} limit {limit}
) t;

```

## 댓글 목록 조회 무한 스크롤

```sql
-- 1번 페이지
select * from comment_v2
  where article_id = {article_id}
  order by path asc limit {limit};

-- 그이후
-- 기준점 = last_path
select * from comment_v2
  where article_id = {article_id} and path > {last_path}
  order by path asc
  limit {limit};

```
