# 댓글 목록 2depth 조회

-   댓글 1
    -   댓글 2
    -   댓글 4
-   댓글 3

    -   댓글 5

-   상위 댓글은 항상 하위 댓글보다 먼저 생성되고
-   하위 댓글은 상위 댓글 별 순서대로 생성된다.
-   상위 댓글에 의해 순서가 명확하게 정해진다.
-   Secondary index(article_id asc,parent_comment_id asc, comment_id asc);
-   shard key = article_id이기에 단일 샤드에서 게시글별 댓글 목록 조회

## 댓글 목록 조회 페이지

-   N번 페이지에서 M개의 댓글 조회

```sql
select * from (
  select comment_id from comment
                    where article_id = {article_id}
                    order by parent_comment_id asc, comment_id asc
                    limit {limit} offset {offset}
)t left join comment on t.comment_id = comment_id;
```

-   댓글 개수 조회

```sql
select count(*) from (
  select comment_id from comment where article_id = {article_id} limit {limit}
)
```

## 댓글 목록 조회 무한스크롤

-   기준점이 2개
    -   parent_comment_id, comment_id 두 개의 컬럼이 정렬에 사용되기 떄문
    -   parent_comment_id가 last_parent_comment_id 보다 크면
        -   다음 데이터 조회
    -   parent_comment_id가 last_parent_comment_id 보다 같으면
        -   comment_id 비교

```sql
-- 1번 페이지
select * from comment
        where article_id = {article_id}
        order by parent_comment_id asc, comment_id asc
        limit {limit};

-- 그 이후 페이지
select * from comment
        where article_id = {article_id} and (
          parent_comment_id > {last_parent_comment_id} or
          (parent_comment_id = {last_parent_comment_id} and comment_id > {last_comment_id})
        )
        order by parent_comment_id asc, comment_id asc
        limit {limit};
```
