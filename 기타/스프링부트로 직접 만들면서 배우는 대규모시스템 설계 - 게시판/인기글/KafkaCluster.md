# Kafka Cluster

-   Kafka
    -   분산 이벤트 스트리밍 플랫폼
    -   대규모 데이터를 실시간 처리
    -   여러 서비스 간에 대규모 이벤트를 생산하고 소비하며 통신
-   기존 API로 서버 통신시 장애가 발생한다면 전파 가능성 + 데이터 유실 가능성 있음 (직접 결합)
-   Producer 와 Consumer 사이에 Message Queue에 데이터 전송 위임 (간접 결합)
-   producer -> Message Broker(queue들) <- consumer(가져오기)
-   Kafka Broker : kafka에서 데이터를 중개 및 처리해주는 애플리케이션 실행 단위
-   Kafka Cluster
    -   Kafka Broker
        -   여러 topic에 대한 부하분산을 위해 여러 Broker에 분산
            -   물리적으로 partition 분산
            -   consumer는 partition 단위로 구독
            -   순서보장 필요시 동일판 partition으로 보내준다.
        -   하나의 브로커가 오류가 나더라도 나머지 브로커에 오류난 브로커의 복제본 follower가 있음
        -   Producer의 acks의 설정을 통해 복제여부 확인
        -   acks = 0 : udp느낌 브로커에 전달되었는지 확인 x 빠름
        -   acks = 1 : leader 전달되면 성공 follower 전달 안되면 장애시 유실가능성
        -   acks = all : leader와 모든 follower에 데이터 기록되면 성공 가장 안전, 지연될 수 있음
        -   더 자세한 내용은 강의 참고
-   Kafka 는 순서가 보장된 데이터 로그를 각 topic의 partition 단위로 Broker의 디스크에 저장
-   각 데이터는 고유한 offset을 가지고 consumer는 offset 기반으로 데이터를 읽어감

-   Zookeeper : 카프카의 메타데이터 관리(Broker, Topic, Partition Consumer Group, Offset)

    -   카프카 2.8 이후부터는 주키퍼 없이 더 간단하기 브로커 자체적으로 관리 가능
    -   KRaft모드

-   각각의 서비스가 토픽(article, comment, like, view)를 카프카로 전송(이벤트)

## 결론

-   Producer

    -   Kafka로 데이터를 보내는 클라이언트
    -   데이터를 생산 및 전송
    -   Topic 단위로 데이터 전송

-   Consumer

    -   Kafka에서 데이터를 읽는 클라이언트
    -   데이터를 소비 및 처리
    -   Topic단위로 구독하여 데이터 처리

-   Broker

    -   Kafka에서 데이터를 중개 및 처리해주는 애플리케이션 실행 단위
    -   Producer와 Consumer 사이에서 데이터를 주고 받는 역할

-   Kafka Cluster

    -   여러 개의 Broker 가 모여서 하나의 분산형 시스템을 구성한것
    -   대규모 데이터에 대해 고성능, 안전성, 확장성, 고가용성 등 지원
        -   데이터 복제, 분산처리, 장애복구 등

-   Topic

    -   데이터가 구분되는 논리적인 단위
    -   게시글 이벤트를 위한 article-topic
    -   댓글 이벤트를 위한 comment-topic

-   Partition

    -   토픽이 분산되는 단위
    -   각 topic은 여러 개의 partition으로 분산 저장 및 병렬 처리 된다.
    -   각 partition내에서 데이터가 순차적으로 기록되므로, partition간에는 순서가 보장되지 않는다.
    -   partition은 여러 broker에 분산되어 cluster의 확장성을 높인다.

-   Offset

    -   각 데이터에 대해 고유한 위치
        -   데이터는 각 Topic의 partition단위로 순차적으로 기록 기록된 데이터는 offset을 가진다
    -   Consumer Group은 각 그룹이 처리한 Offset을 관리한다.
        -   데이터를 어디까지 읽었는지

-   Consumer Group
    -   Consumer Group은 각 Topic의 Partition 단위로 Offset을 관리한다.
        -   인기글 서비스 위한 Consumer Group
        -   조회 최적화 서비스를 위한 Consumer Group
    -   Consumer Group내 Consumer들은 데이터를 중복해서 읽기 않을 수 있다.
    -   Consumer Group별로 데이터를 병렬로 처리할 수 있다.

`카프카 환경세팅 강의자료 참고`
