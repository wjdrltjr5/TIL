# 오라클 성능 고도화 원리와 해법 1

## 목차

### Chapter 1 오라클 아키텍처

현실적으로 DBMS 내부 아키텍처와 SQL 옵티마이저의 원리를 이해하지 않고서는 고성능의 전문 튜너가 아니더라도 데이터베이스 프로그램 개발자라면 기본적으로 실행계획 정도는 확인할 수 있어야 하고, 그러려면 기본적인 아키텍처에 대한 이해는 꼭 필요하다.

-   [기본 아키텍처](./Chapter%201/1.%20기본%20아키텍처.md)
-   [DB 버퍼 캐시](./Chapter%201/2.%20DB%20버퍼%20캐시.md)
-   [버퍼 Lock](./Chapter%201/3.%20버퍼%20Lock.md)
-   [Redo](./Chapter%201/4.%20Redo.md)
-   [Undo](./Chapter%201/5.%20Undo.md)
-   [문장수준 읽기 일관성](./Chapter%201/6.%20문장수준%20읽기%20일관성.md)
-   [Consistent vs Current 모드 읽기](./Chapter%201/7.%20Consistent%20vs.%20Current%20모드%20읽기.md)
-   [블록 클린아웃](./Chapter%201/8.%20블록%20클린아웃.md)
-   [Snapshot too old](./Chapter%201/9.%20Snapshot%20too%20old.md)
-   [대기 이벤트](./Chapter%201/1.10%20대기%20이벤트.md)
-   [Shared Pool](./Chapter%201/1.11%20Shared%20Pool.md)

### Chapter 2 트랜잭션과 Lock

-   [트랜잭션 동시성제어](./Chapter%202/1.%20트랜잭션%20동시성%20제어.md)
-   [트랜잭션 수준 읽기 일관성](./Chapter%202/2.%20트랜잭션%20수준%20읽기%20일관성.md)
-   [비관적 vs 낙관적 동시성 제어](./Chapter%202/3.%20비관적%20vs%20낙관정%20동시성제어.md)
-   [동시성 구현 사례](./Chapter%202/4.%20동시성%20구현%20사례.md)
-   [오라클 Lock](./Chapter%202/5.%20오라클%20Lock.md)

### Chapter 3 오라클 성능 관리

-   [Explain Plan](./Chapter%203/1.%20Explain%20Plan.md)
-   [AutoTrace](./Chapter%203/2.%20AutoTrace.md)
-   [SQL 트레이스](./Chapter%203/3.%20SQL%20트레이스.md)
-   [DBMS_XPLAN 패키지](./Chapter%203/4.%20DBMS_XPLAN%20패키지.md)
-   [V$SYSSTAT](./Chapter%203/5.%20V$SYSSTAT.md)
-   [V$SYSTEM_EVENT](./Chapter%203/6.%20V$SYSTEM_EVENT.md)
-   [Response Time Analysis 방법론과 OWI](./Chapter%203/7.%20Response%20Time%20Analysis%20방법론과%20OWI.md)
-   [Statspack AWR](./Chapter%203/8.%20Statspack%20%20AWR.md)
-   [ASH](./Chapter%203/9.%20ASH.md)
-   [V$SQL](./Chapter%203/10.%20V$SQL.md)
-   [End to End](./Chapter%203/11.%20End-To-End%20성능관리.md)

### Chapter 4 라이브러리 캐시 최적화 원리

-   [SQL과 옵티마이저](./Chapter%204/1.%20SQL과%20옵티마이저.md)
-   [SQL 처리과정](./Chapter%204/2.%20SQL%20처리과정.md)
-   [라이브러리 캐시 구조](./Chapter%204/3.%20라이브러리%20캐시%20구조.md)
-   [커서 공유](./Chapter%204/4.%20커서%20공유.md)
-   [바인드 변수의 중요성](./Chapter%204/5.%20바인드%20변수의%20중요성.md)
-   [바인드 변수의 부작용과 해법](./Chapter%204/6.%20바인드%20변수의%20부작용과%20해법.md)
-   [세션 커서 캐싱](./Chapter%204/7.%20세션%20커서%20캐싱.md)
-   [애플리케이션 커서 캐싱](./Chapter%204/8.%20애플리케이션%20커서%20캐싱.md)
-   [Static vs. Dynamic SQL](./Chapter%204/9.%20Static%20vs.%20Dynamic%20SQL.md)
-   [Dynamic SQL 사용기준](./Chapter%204/10.%20Dynamic%20SQL사용기준.md)
-   [Static SQL 구현을 위한 기법들](./Chapter%204/11.%20Static%20SQL%20구현을%20위한%20기법들.md)

### Chapter 5 데이터베이스 Call 최소화 원리

-   [Call 통계](./Chapter%205/1.%20Call%20통계.md)
-   [User Call vs Recursive Call](./Chapter%205/2.%20User%20Call%20vs%20Recursive%20Call.md)
-   [데이터베이스 Call이 성능에 미치는 영향](./Chapter%205/3.%20데이터베이스%20Call이%20성능에%20미치는%20영향.md)
-   [Array Processing 활용](./Chapter%205/4.%20Array%20Processing%20활용.md)
-   [Fetch Call 최소화](./Chapter%205/5.%20Fetch%20Call%20최소화.md)
-   [페이지 처리의 중요성](./Chapter%205/6.%20페이지%20처리의%20중요성.md)
-   [PL/SQL 함수의 특징과 성능 부하](./Chapter%205/7.%20PL_SQL%20함수의%20특징과%20성능%20부하.md)
-   [PL/SQL 함수 호출 부하 해소 방안](./Chapter%205/8.%20PL_SQL%20함수%20호출%20부하%20해소%20방안.md)

### Chapter 6 I/O 효율화 원리

-   [블록 단위 I/O](./Chapter%206/1.%20블록단위%20IO.md)
-   [Memory vs Disk](./Chapter%206/2.%20Memory%20vs%20Disk.md)
-   [Single vs MultiBlock I/O](./Chapter%206/3.%20Single%20vs%20Multiblock%20io.md)
-   [Prefetch](./Chapter%206/4.%20Prefetch.md)
-   [Direct Path I/O](./Chapter%206/5.%20Direct%20Path%20IO.md)
