# 논블로킹 IO로 성능 더 높이기
가상 스레드와 고루틴과 같은 경량 스레드를 사용하면 IO중심 작업을 하는 서버의 처리량을 높일 수 있지만 경량 스레드 자체도 메모리를 사용하고 스케줄링이 필요하다.

경량 스레드가 많이질수록 더 많은 메모리를 사용하고 스케줄링에 더 많은시간을 사용하게 된다.

사용자가 폭발적으로 증가하면 어느 순간 경량 스레드로도 한계가 온다. 이때는 서버의 IO 구현 방식을 구조적으로 변경해야 한다. 

논블로킹 IO는 새로운 것이 아니다. 여기에 비동기 API를 곁들이면 덜 복잡한 코드로 높은 성능을 낼 수 있다. 실제로 Nginx, Netty, Node.js 등 서버에서 많이 사용하는 기술은 성능을 위해 논블로킹 IO를 사용한다.

## 논블로킹 IO 동작 개요
논블로킹 IO는 입출력이 끝날 떄 까지 스레드가 대기하지 않는다. 예로 다음 코드에서 channel.read()코드는 데이터를 읽을 때까지 대기하지 않는다. 읽을 데이터가 없으면 바로 0을 리턴한다 이는 데이터를 읽을 때까지 대기하는 블로킹 IO와는 동작 방식이 다르다.

```java
// channel : SocketChannel, buffer: ByteBuffer
int byteReads = channel.read(buffer); // 데이터를 읽을 때 까지 대기하지 않음
..// 읽은 데이터가 없어도 다음 코드 계속 실행
```

데이터를 조회했는지 여부에 상관없이 대기하지 않고 바로 다음 코드를 실행하므로 블로킹IO처럼 데이터를 조회했다는 가정하에 코드를 작성할 수 없다.

대신 루프 안에서 조회를 반복해서 호출한 뒤 데이터를 읽었을 때만 처리하는 방식으로 구현할 수 있다.

```java
// CPU 낭비가 심한 방식
while(true){
    int byteReads = channel.read(buffer);
    if(byteReads > 0){
        handleData(channel, buffer);
    } 
}
```
하지만 위 코드처럼 작성하면 CPU 낭비가 심하다. 읽은 데이터가 없어도 무한 루프가 실행되기 때문이다.

실제로 논블로킹IO를 사용할 때는 데이터 읽기를 바로 시도하기 보다는 어떤 연산을 수행할 수 있는지 확인하고 해당 연산을 실행하는 방식으로 구현한다.

- 실행 가능한 IO연산 목록을 구한다(실행 가능한 연산을 구할 때까지 대기)
- 위에서 구한 IO연산 목록을 차례대로 순회한다.
    - 각 IO연산을 처리한다.
- 이 과정을 반복한다.

위 방식으로 구현한 간단한 예제 코드
```java
Selector selector = Selector.open();

ServerSocketChannel serverSocket = ServerSocketChannel.open();
serverSocket.bind(new InetSocketAddress(7031));
serverSocket.configureBlocking(false); // 서버 소켓 비동기 설정

serverSocket.register(selector, SelectionKey.OP_ACCEPT); // 연결 연산 등록

while(true){
    selector.select(); // 가능한 IO 연산이 있을 때까지 대기
    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    Iterator<SelectionKey> iterator = selectedKeys.iterator();

    while(true){
        selector.select(); // 가능한 IO 연산이 있을 때까지 대기
        Set<SelectionKey> selectedKeys = selector.selectedKeys();
        Iterator<SelectionKey> iterator = selectedKeys.iterator();
        while(iterator.hasNext()){ // IO 연산 순회
            SelectionKey key = iterator.next();
            iterator.remove(); 
            if(key.isAcceptable()){ // 클라이언트 연결 처리 가능하면
                SocketChannel client = serverSocket.accept(); // 클라이언트 연결 처리
                client.configureBlocking(false); // 소켓 비동기 설정
                client.register(selector, SelectionKey.OP_READ); // 읽기 연산 등록
            }else if(key.isReadable()){ // 읽기 연산 가능하면
                SocketChannel channel = (SocketChannel) key.channel(); // 채널 구함
                int readBytes = channel.read(inBuffer); // 채널에 읽기 연산 실행
                if(readBytes == -1){
                    channel.close();
                }else{
                    inBuffer.flip();
                    outBuffer.put(inBuffer); // 출력 버퍼에 복사
                    inBuffer.clear();
                    outBuffer.flip();
                    channel.write(outBuffer); // 채널에 쓰기 연산 실행
                    outBuffer.clear();
                }
            }
        }
    }
}
```
위코드의 핵심은 Selector다 Selector.select() 메서드가 IO 처리가 가능한 연산이 존재할 때까지 대기한다.

이 메서드가 리턴하면 수행할 수 있는 연산이 존재하는 것이다. 실행 가능한 연산 목록은 Selector.selectedKeys() 로 조회하는데 이렇게 구한 SelectionKey를 이용해서 어떤 연산이 가능한지 확인하고 해당 연산을 수행한다. 

논블로킹 IO를 이용해서 구현한 서버는 블로킹 IO를 이용한 구현과 차이가 난다. 일반적으로 블로킹IO로 구현한 서버는 커넥션별로(또는 요청별로) 스레드를 할당한다. 동시 연결 클라이언트가 1000개면 클라이언트를 처리할 스레드를 1000개 생성한다.

반면에 논블로킹 IO는 클라이언트 수에 상관없이 소수의 스레드를 사용한다. 위 코드는 스레드 1개를 이용해서 여러 클라이언트의 요청을 처리한다. 

논블로킹 IO는 동시 접속하는 클라리언트가 증가해도 스레드의 개수는 일정하게 유지되므로 같은 메모리로 더 많은 클라이언트 연결을 처리할 수 있다.

> IO 멀티플렉싱 : IO 다중화는 단일 이벤트 루프에서 여러 IO작업을 처리하는 개념을 표현할 때 사용한다. 위 코드에서 논블로킹 IO와 Selector를 이용한 입출력 처리가 이에 해당 IO멀티 플렉싱을 사용함으로써 더 적은 자원으로 더 많은 클라이언트를 처리할 수 있어 대규모 트래픽을 처리해야 하는 서버를 구현할 때 사용한다.

논블로킹IO를 1개 스레드로 구현하면 동시성이 떨어진다. 논블로킹 IO에서 동시성을 높이기 위해서 사용하는 방법은 채널들을 N개 그룹으로 나누고 각 그룹마다 스레드를 생성하는 것이다.

보통 CPU 개수만큼 그룹을 나누고 각 그룹마다 입출력을 처리할 스레드를 할당한다.

## 리액터 패턴
리액터 패턴은 논블로킹 IO를 이용해서 구현할 때 사용하는 패턴 중 하나이다.

논블로킹 IO로 구현된 네트워크 프레임워크 문서를 읽다 보면 나오는 리액터 단어가 이 패턴을 의미한다.

리액터 패턴은 동시에 들어오는 여러 이벤트를 처리하기 위한 이벤트 처리 방법이다. 리액터 패턴은 크게 리액터와 핸들러 두 요소로 구성된다.

먼저 리액터는 이벤트가 발생할 때까지 대기하다가 이벤트가 발생하면 알맞은 핸들러에 이벤트를 전달한다. 이벤트를 받은 핸들러는 필요한 로직을 수행한다.

리액터는 다음과 유사한 형태를 갖는다.
```java
while(isRunning){
    List<Event> events = getEvents(); // 이벤트가 발생할 때까지 대기
    for(Event event : events){
        Handler handler = getHandler(event); // 이벤트를 처리할 핸들러 구함
        handler.handle(event)
    }
}
```

위 코드를 보면 리액터는 이벤트를 대기하고 핸들러에 전달하는 과정을 반복하는데, 그래서 리액터를 이벤트 루프라고도 한다.

앞서 사용했던 논블로킹 IO 예제 코드를 보면 ke 타입에 따라 처리가 달라지는 것을 보면 리액터 패턴과 완전히 처리 방식이 동일한 것을 알 수 있다.

실제로 논블로킹IO에 기반한 Netty, Nginx, Node.js 등의 프레임워크나 서버는 리액터 패턴을 적용하고 있다.

리액터 패턴에서 이벤트 루프는 단일 스레드로 실행된다. 멀티 코어를 가진 서버에서 단일 스레드만 사용하면 처리량을 최대한 낼 수 없다. 또 핸들러에서 CPU연산이다 블로킹을 유발하는 연산을 수행하면 그 시간만큼 전체 이벤트 처리 시간이 지연된다.

이런 한계를 보완하기 위해 핸들러나 블로킹 연산을 별도 스레드 풀에서 실행하기도 한다. Netty의 경우 여러개의 이벤트 루프를 생성해서 멀티 코어를 활용한다.

## 프레임워크 사용하기
줄 단위로 데이터를 수신하는 서버를 구현해야 한다고 가정 블로킹 IO일 경우 BufferReader를 사용해서 쉽게 줄 단위로 데이터를 읽을 수 있다.

```java
BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()), "UTF-8");

...
String line;
while((line = br.readLine()) != null){// 줄 단위로 쉽게 읽을 수 있음
    //line 처리
    
}
```

논블로킹 IO를 사용하면 처리가 복잡해진다. 데이터를 읽은 뒤 ₩n 문자가 있는지 확인하는 코드를 구현해야 한다.

₩n문자가 없는 경우 읽은 데이터를 별도 버퍼에 계속 누적하는 처리도 해야 한다. 또한  ₩n 문자가 여러 개 존재하는 경우도 처리해야 한다. 채널마다 누적 처리를 위한 버퍼도 관리해야 한다.

논 블로킹 IO API를 직접 사용하기 보다는 논 블로킹 IO를 보다 쉽게 구현할 수 있도록 도와주는 프레임워크를 사용하는 것은 선호한다.

예로 리액터 네티를 사용하면 아래 코드를 이용해서 줄 단위로 데이터를 주고받는 에코서버를 구현할 수 있다.
```java
DisposableServer server = TcpServer.create()
        .port(7031)
        .doOnConnection(conn -> 
                conn.addHandlerFirst(new LineBasedFrameDecoder(1024)) // 줄 단위 읽기 처리
        )
        .handler((in, out) -> {
            return in.receive()
                     .asString() // byte를 문자열로 변환
                     .doOnNext(line -> {
                        log.info("received: {}", line);
                     })
                     .flatMap(line ->
                        out.sendString(Mono.just(line + "\n"))//문자열 쓰기
                    );
        })
        .bindNow();

```
리액터 네티가 줄 단위 읽기와 문자열 변환 처리 기능을 제공하므로 저수준의 IO처리를 직접 구현하지 않아도 된다. 

물론 리액터 네티가 기반으로 하는 리액티브 API(스프링 리액터)를 익혀야 한다.

## 논블로킹/비동기 IO와 성능
실제로 성능이 훨씬 더 잘나온다.