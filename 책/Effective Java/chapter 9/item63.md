## 문자열 연결은 느리니 주의하라

---

문자열 연결 연산자 + 는 여러 문자열을 하나로 합쳐주는 편리한 수단이다. 그런데 한 줄 짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현을 만들때라면 괜찮지만 본격적으로 사용한다면 성능 저하를 감내하기 어렵다

문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다.

문자열은 불변이라서 두 문자열을 연결한경우 양쪽의 내용을 모두 복사해야 하므로 성능 저하는 피할수 없는 결과다.

성능을 포기하고 싶지 않다면 StringBuilder를 사용하자.

```java
public String statement(){
  StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
  for(int i = 0; i < numItems(); i++){
    b.append(lineForItem(i));
  }
  return b.toString();
}
```

---

성능에 신경써야 한다면 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자 대신 StringBuilder 의 appent메서드를 사용하라. 문자배열을 사용하거나. 문자열을 연결하지 않고 하나씩 처리하는 방법도 있다.
