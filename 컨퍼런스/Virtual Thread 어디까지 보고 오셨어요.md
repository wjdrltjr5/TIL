# Virtual Thread 어디까지 보고 오셨어요?

- [영상링크](https://www.youtube.com/watch?v=AuBHv8NOca4&list=PLdHtZnJh1KdYYFaaJCCis_swbqbYPrtYC&index=10)
- [다른 참고링크](https://techblog.woowahan.com/15398/)
- [다른 참고 영상](https://www.youtube.com/watch?v=BZMZIM-n4C0)

## Virtual Thread Overview
- JDK 21에 공식 도입
- JEP 425를 통해 제안됨
### 목표
- Thread-Per-Request 스타일의 확장성 확보 (동기 코드 스타일)
- Thread 클래스를 상속함으로써 최소한의 변경으로 기존 코드를 마이그레이션 지원
- 기존 도구를 이용한 쉬운 디버깅 및 프로파일링

높은 동시 처리량이 필요한 서버 개발을 쉽고 효율적으로 만드는 것을 도움 

### Virtual Thread & Spring
- Spring Boot 3.2부터 공식 지원
    - spring.thread.virtual.enable=true
- 여러 Executor들을 Virtual Thread Executor 로 생성

### 높은 Blocking I/O처리량
- Platform Thread 대비 압도적으로 높은 Blocking I/O 처리량

## Virtual Thread Scheduling & Context Switching
- Virtual Thread의 실행 & 중단을 위해서는 두 가지 요소 필요
    - Scheduling
    - Context Switching
- Virtual Thread 생성시 Platform Thread의 Queue에 저장
    - 하나의 Virtual Thread 중단시 다른 Virtual Thread를 Queue에서 꺼내서 실행

- Virtual Thread의 Scheduling & Context Switching 은 Platform Thread의 메모리 영역을 조작하는 것
    - Heap에 저장되어있던 Virtual Thead의 스택을 Platform Thread에 복제하고 Virtual Thread가 실행할 바이트 코드의 주소를 Platform Thread PC Register에 업데이트
    - 해당 과정은 Application Level에서만 발생
        - Platform Thread의 경우 커널레벨에서 실행 여러 부가비용 발생
            - 커널 모드 변경비용
            - CPU Core가 사용하는 Cache의 Reload 비용
        - 위 비용을 아낄 수 있어 빠른 속도로 수행 가능

### 정리
- Virtual Thread도 실행 & 중단 존재
- Virtual Thread는 스스로 코드를 실행할 능력이 없음
- Virtual Thread가 가지고 있는 작업(Runnable)을 Platform Thread가 실행할 수 있도록 메모리 공간을 조작함
- Virtual Thread는 Platform Thread들을 가진 ForkJoinPool의 Queue에 전달하여 스케줄링 됨
- 모든 과정은 Application Level에서 이루어 지기 때문에 빠르다.
- Virtual Thread의 실행이란 Platform Thread가 Runnable의 run() 메서드를 호출하는 것
- Virtual Thread의 중단은 Runnable 실행 중 멈추고 나중에 다시 이어서 실행된다는 뜻
    - 실행 중 중단이란 개념 Function & Coroutine

### Function & Coroutine
모두 코드의 재사용성과 구조화를 위해 사용되지만 두개는 실행흐름을 제어하는 방식에 차이가 있음
- Function : 호출되면 시작부터 끝까지 연속적으로 실행되며 중간에 멈추거나 상태를 저장하지 않음
    - start -> exec -> end
- Coroutine : 실행도중 여러 지점에서 멈췄다가 다시 이어서 실행할 수 있음
    - start -> exec -> pause -> resume -> end
    - 실행의 중단과 재개가 가능함

- Java는 Coroutine과 유사한 Continuation을 통해 함수의 실행 상태의 중단/재개를 할 수 있음
```java
public static final ContinuationScope scope = new ContinuationScope("SpringCamp 2025");

public static void main(String[] args){

    final var cont = new Continuation(Scope, new Runnable(){
        @Override
        public void run(){
            greeting();
        }
    })

    cont.run();
    System.out.println("virtual thread!");
    cont.run(); // 이전에 멈췄던 지점부터 다시 실행 재게 이때 힙에 저장되었던 스택 복원
} 

private static void greeting(){
    System.out.println("hello!");
    Continuation.yield(scope); // 더이상 함수는 진행되지 않고 이전 호출한 함수에 다음 내용 실행
                              // 중단시점에 지금까지의 메모리 스택을 stack chunk라는 객체로 힙에 저장후 스택내용 제거
    System.out.println("spring camp 2025!");
}

/* 실행결과
hello!
virtual thread!
spring camp 2025!
*/
```
- Thread의 stack frame들을 heap에 저장한 후 중단, 재개시 heap에서 stack frame들을 꺼내와 이어서 실행함

## Continuation & Virtual Thread
- Virtual Thread는 여러 상태를 가진다.
    - 실행 : start();
    - 중단 : park();
    - 재개 : unpark();

- Virtual Thread는 Continuation 과 실행한 Runnable를 가진 인스턴스이다.
- Virtual Thread의 실행은 ForkJoinPool에 Runnable 타입의 람다 runContinuation()을 등록하는 것
- Virtual Thread는 park(), unpark()를 통해 Continuation을 사용하여 실행의 중단과 재게를 한다.

## Socket I/O
- Socket I/O Blocking은 커널영역에서 발생
- Kernel영역은 Application 영역의 구현을 알지 못한다. 
    - JVM의 Virtual Thread를 알지 못함
    - OS Thread로만 핸들링 수행
- Blocking I/O가 발생하면 OS Thread 자체를 Blocking 시킴
    - OS Thread와 연결된 Platform Thread가 Blocking 되어버림
    - 해결을 위해 Socket 주요구현을 c, c++ 네이티브에서 자바 코드레벨로 이동
    - 논블로킹으로 동작하는 소켓 사용

- JDK13 이전 Socket 구현
    - AbstractPlainSocket 클래스에서 대부분의 기능 구현 및 재위임
        - SocketInputStream, SocketOutputStream 반환 여기서 커널 블로킹 발생

- JDK13에 추가된 NioSocketImpl 클래스 (기본으로 사용)
    - SocketImpl 클래스의 많은 구현이 Native Code -> Java Code로 이전됨
    - Virtual Thread에 대해 Non-blocking Socket 사용
        - InputStream , OutputStream 반환
    - 이후 버전들에 대해 여러 변경이 일어남

- JDK13 이전 읽기/쓰기 연산 흐름
    1. 읽기/쓰기
    2. Blocking Socket에 Native I/O 호출
    3. I/O 수행 & Blocking
    4. 읽기/쓰기 결과 반환

- 새로운 읽기/쓰기 연산 흐름
    1. 읽기/쓰기
    2. Non-blocking Socket에 Native I/O 호출
    3. I/O 수행 & Non-Blocking
    4. 즉시 읽기/쓰기 결과 반환
        - 재시도 필요 여부
            - 필요시 Socket 읽기/쓰기 준비여부 감시 등록 (감시는 Poller가 수행)
            - 준비 전 Virtual Thread park(); -- 중단
            - 준비되면 unpark() -- 재개

![alt text](<NioSocketImpl구조.png>)

- NativeDispatch
    - Socket에 대해 Native I/O 기능을 호출하기 위한 클래스
    - Socket I/O 처리를 위해 SocketDispatcher 클래스 사용
    - OS Socket은 Non-blocking Mode로 설정 가능 (JDK21의 NioSocketImpl Virtual Thread에 대해 언제나 이 모드로 설정)
    - 해당 작업의 결과 반환 표

|반환 값(int)|	의미|	재시도 필요 여부|
|----------|------|--------------|
|양수	|양수값 길이 만큼 Byte 읽기/쓰기 수행 완료|	X|
|EOF(-1)|	연결 종료	|X|
|UNAVAILABLE(-2)|	Socket의 Send/Recv 버퍼가 아직 준비 안됨|	O|
|INTERRUPTED(-3)|	System Call이 Interrupt 됨|	O|
|기타 음수|	에러 발생	|X|
    

- Poller (추상 클래스)
    - Selector API 구현체를 이용
    - 생성시 플랫폼 스레드를 생성하고 무한루프를 돌면서 준비상태를 감시함
    - NioSocketImpl이 감시 요청을 여기에 요청함
    - 감지 등록 후 Virtual Thread 중단
    - Socket 준비 시 Virtual Thread 재개
    - 읽기/쓰기 각 1개씩 Poller 객체 생성 및 전용 플랫폼 스레드로 실행

- Selector API
    - 등록된 Socket의 상태를 감지할 수 있는 API
    - 읽기/쓰기 등의 준비 상태 감지 가능
    - 준비된 Socket의 FileDescriptor ID를 반환
    - OS별 구현체 존재
        - Window : IOCP
        - Linux : Epoll
        - MacOS, BSD : Kqueue

## 정리
- Virtual Thread는 Continuation을 통해 Platform Thread의 메모리를 조작하여 자신의 Runnable을 실행 & 중단하게 한다.
    - Context Switching
- Virtual Thread는 Scheduling & Context Switching이 Application Level에서 수행되기 떄문에 매우 빠르다.
- Socket은 Non-blocking Socket과 Virtual Thread의 실행 & 중단을 이용해서 Kernel Level Blocking이 발생하지 않도록 변경되었다.
- 위 특징들을 이용하여 동기 코드 스타일로도 비동기 코드 수준의 처리량달성이 가능하다.