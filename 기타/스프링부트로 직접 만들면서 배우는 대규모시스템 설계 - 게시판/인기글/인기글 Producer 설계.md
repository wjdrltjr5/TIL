# 인기글 Producer 설계

-   게시글/댓글/좋아요/조회수 서비스는

    -   Kafka로 이벤트를 전달하는 Producer 역할

-   Producer는 정의된 이벤트를 Kafka로 잘 전달하면 된다.
    -   이벤트 전달을 안전하게 잘 처리할 수 있을까?
    -   Transactional Messaging

## Transactional Messaging

-   데이터의 일관성, 원자성 관리를 위해 트랜잭션 관리는 중요
-   Producer가 생산한 이벤트가 Kafka를 통해 Consumer로 전달되는 과정

    -   Producer는 비즈니스 로직을 수행 (게시글/댓글/좋아요/조회수 서비스 API)
    -   이벤트 데이터를 Kafka로 이벤트 데이터 전송
    -   Consumer는 Kafka에서 이벤트 데이터를 가져와서 처리

-   네트워크를 통한 복잡한 전송과정 필요

    -   네트워크는 언제든 순단될 수 있고 장애가 발생할 수도 있다.
    -   Kafka가 데이터 유실을 방지하기 위해 여러 방법 제공(신뢰할 수 있는 시스템)
    -   Consumer는 이벤트 처리를 정상적으로 모두 완료한 이후에 offset을 commit하면 데이터의 유실 없이 처리할 수 있다.

-   Producer가 Kafka에 이벤트를 전달하는 과정에 장애가 발생한다면(네트워크 순단, Kafka의 모든 Broker 장애)

    -   Producer는 Kafka로의 데이터 전송과 무관하게 정상 독작해야 한다.
    -   Kafka로 데이터 전송이 실패한다고 해서 Producer에 장애가 전파되면 안된다.
        -   게시글 정상생성 but 이벤트 Kafka 전달 실패
        -   신뢰할 수 있는 카프카로 전송전 실패시 데이터 유실
        -   데이터의 일관성 깨짐

-   해결 : 비즈니스 로직 수행과 이벤트 전송이 하나의 트랜잭션으로 처리

    -   꼭 실시간 처리 x
    -   이벤트 전송은 추후에 처리되어고 충분
    -   최종적으로 일관성 유지

-   일반적인 트랜잭션 범위 내에서 이벤트를 카프카로 전송하는 코드추가는 불가

    -   mysql과 Kafka는 서로 다른 시스템

-   두개의 다른 시스템을 단일 트랜잭션으로 묶는 방법
    -   Distributed Transaction
        -   분산 시스템에서 트랜잭션을 보장하기 위한 방법
    -   Transactional Messaging
        -   메시지 전송과 타 시스템 작업 간에 분산 트랜잭션을 보장하는 방법
        -   Two Phase Commit
        -   Transactional Outbox
        -   Transactional Log Tailing

### Two Phase Commit

-   분산 시스템에서 모든 시스템에 하나의 트랜잭션을 수행할 때,
    -   모든 시스템이 성공적으로 작업을 완료하면 commit
    -   하나라도 실패하면 rollback
-   두 단계로 나뉘어 수행된다.

    -   Prepare Phase(준비 단계)
        -   Coordinator(중개자)는 각 참여자에게 커밋할 준비가 되었는지 물어본다.
        -   각 참여자는 커밋할 준비가 되었는지 응답
    -   Commit Phase(커밋 단계)
        -   모든 참여자가 준비 완료 응답을 보내면 Coordinator는 모든 참여자에게 트랜잭션 커밋을 요청
        -   모든 참여자는 트랜잭션을 커밋

-   문제 사항

    -   모든 참여자의 응답을 기다려야 하기 때문에 지연이 길어질 수 있다.
    -   Coordinator 또는 참여자 장애가 발생하면 참여자들은 현재 상태를 모른채 대기할 수 있다.
    -   트랜잭션 복구 처리가 복잡해질 수 있다.

-   Transactional Messaging 달성하기에 적절하지 않음

### Transactional Outbox

-   이벤트 전송 작업을 일반적인 데이터베이스 트랜잭션에 포함 시킬 수는 없다.
-   하지만, 이벤트 전송 정보를 데이터베이스 트랜잭션에 포함하여 기록할 수는 있다.
    -   트랜잭션을 지원하는 데이터베이스에 Outbox 테이블을 생성하고
    -   서비스 로직 수행과 Outbox 테이블 이벤트 메시지 기록을 단일 트랜잭션으로 묶는다

1. 비즈니스 로직 수행 및 Outbox 테이블 이벤트 기록
    - Transaction start
    - 비즈니스 로직 수행
    - Outbox 테이블에 전송 할 이벤트 데이터 기록
    - Commit or abort
2. Outbox 테이블을 이용한 이벤트 전송 처리
    - Outbox테이블 미전송 이벤트 조회
    - 이벤트 전송
    - Outbox 테이블 전송 완료 처리

-   Two Phase Commit의 성능과 오류 처리에 대한 문제가 줄어든다.
-   데이터베이스 트랜잭션 커밋이 완료되었다면 outbox에 기록이 되기 때문에 유실되지 않는다 (나중에 보내더라도)
-   추가적인 Outbox 테이블 생성 및 관리가 필요하다
-   Outbox 테이블의 미전송 이벤트를 Message Broker로 전송하는 작업이 필요하다.
-   Message Broker로 이벤트를 전송하는 작업 방법
    -   이벤트 전송 작업을 처리하기 위한 시스템을 직접 구축
    -   Transactional Log Tailing Pattern 활용

#### Transactional Log Tailing

-   데이터베이스의 트랜잭션 로그를 추적 및 분석하는 방법

    -   각 DB는 각 트랜잭션의 변경사항을 로그로 기록
    -   Mysql : binlog
    -   PostgreSQL : WAL
    -   SQL Server : Transaction Log

-   이러한 로그를 읽어서 Message Broker에 이벤트를 전송해볼 수 있다.

    -   CDC(Change Data Capture)기술을 활용하여 데이터의 변경 사항을 다른 시스템에 전송
    -   변경 데이터 캡쳐(CDC) : 데이터 변경 사항을 추적하는 기술

-   Data Table의 변경 사항만 추적해도 충분하다면, Outbox Table은 없앨 수도 있다.
-   트랜잭션 로그를 추적하기 위해 CDC 기술을 활용 러닝커브 발생

## Transactional Messaging

-   Outbox Table의 필요성?

    -   Transaction Log Tailing 을 활용하면 Data Table의 변경 사항을 직접 추적할 수 있다.
    -   하지만, Data Table은 메시지 정보가 DB 변경 사항에 종족된 구조로 한정
    -   Outbox Table을 활용하면
        -   부가적인 테이블로 복잡도 및 관리비용은 늘어나지만,
        -   이벤트 정보를 더욱 구체적이고 명확하게 정의할 수 있다.

-   Transactional Log Tailing의 활용?
    -   러닝 커브 발생
    -   익숙한 스프링 부트를 활용하여 개발중
        -   직접 개발함으로 익숙한 환경에서 관리

outbox 테이블 컬럼
id, eventType, payload, shard_key(샤드 사용중 가정이라 필요), created_at
