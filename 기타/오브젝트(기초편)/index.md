# 오브젝트 기초편

-   [객체지향의 사실과 오해](https://github.com/wjdrltjr5/The-Essence-of-Object-Orientation)
-   [오브젝트](https://github.com/wjdrltjr5/Object)

두 책을 집필하신 조용호님의 강의 두책을 읽고나서 객체지향에 대해 많은걸 생각할 수 있는 좋은 기회를 얻었었기에 이 강의또한 바로 현질

[조용호님 블로그](https://eternity-object.tistory.com/)

[강의대시보드](https://www.inflearn.com/course/%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8-%EA%B8%B0%EC%B4%88%ED%8E%B8-%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5)

## 목차

-   [section 1 영화 예매 도메인](./section%201/1.%20영화%20예매%20도메인.md)
-   section 2 절차적인 설계 개선하기
    -   [2-1 절차적인 설계](./section%202/2-1%20절차적인%20설계%20.md)
    -   [2-2 변경과 의존성](./section%202/2-2%20변경과%20의존성.md)
    -   [2-3 데이터와 프로세스 통합하기](./section%202/2-3%20데이터와%20프로세스%20통합하기.md)
    -   [2-4 절차에서 객체로](./section%202/2-4%20절차에서%20객체로.md)
-   section 3 객체지향 기본 원칙
    -   [3-1 객체지향 설계 원칙](./section%203/3-1%20객체지향%20설계%20원칙.md)
    -   [3-2 책임 주도 설계](./section%203/3-2%20책임%20주도%20설계.md)
    -   [3-3 표현적 차이 줄이기](./section%203/3-3%20표현적%20차이%20줄이기.md)
-   section 4 책임 할당하기
    -   [4-1 정보와 책임 할당 - 정보 전문가](./section%204/4-1%20정보와%20책임%20할당%20-%20정보%20전문가.md)
    -   [4-2 설계 트레이드오프 - 창조자와 낮은 결합도](./section%204/4-2%20설계%20트레이드오프%20-%20창조자와%20낮은%20결합도.md)
    -   [4-3 설계 트레이드오프 - 높은 응집도](./section%204/4-3%20설계%20트레이드오프-%20높은%20응집도.md)
    -   [4-4 유연한 설계 - 다형성](./section%204/4-4%20유연한%20설계%20-%20다형성.md)
    -   [4-5 결합도 낮추기 - 변경 보호](./section%204/4-5%20결합도%20낮추기%20-%20변경%20보호.md)
-   section 5 객체지향 구현
    -   [5-1 객체 구현하기](section%205/5-1%20객체%20구현하기.md)
    -   [5-2 메시지와 메서드의 분리](section%205/5-2%20메시지와%20메서드의%20분리.md)
    -   [5-3 유연하고 일관적인 협력](section%205/5-3%20유연하고%20일관적인%20협력.md)
    -   [5-4 애플리케이션 객체 추가하기](./section%205/5-4%20애플리케이션%20객체%20추가하기.md)
-   section 6 변경과 설계
    -   [6-1 변경과 설계](./section%206/6-1%20변경과%20설계.md)
    -   [6-2 응집도](./section%206/6-2%20응집도.md)
    -   [6-3 결합도](./section%206/6-3%20결합도.md)
    -   [6-4 캡슐화](./section%206/6-4%20캡슐화.md)
    -   [6-5 설계 평가하기](./section%206/6-5%20설계%20평가하기.md)
    -   [6-6 중복 할인 정책 추가하기](./section%206/6-6%20중복%20할인%20정책%20추가하기.md)

## Q&A

-   [Entity를 바로 도메인객체?](https://www.inflearn.com/community/questions/1403467/%EB%8B%A4%ED%98%95%EC%84%B1%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%97%AD%ED%95%A0-%EB%94%94%EC%9E%90%EC%9D%B8%EA%B3%BC-%EA%B7%B8%EC%97%90-%EB%8C%80%EC%9D%91%EB%90%98%EB%8A%94-%EC%98%81%EC%86%8D%EC%84%B1-%EC%A0%80%EC%9E%A5%EC%86%8C%EC%97%90%EC%84%9C%EC%9D%98-%EB%AA%A8%EB%8D%B8-%EB%94%94%EC%9E%90%EC%9D%B8%EC%9D%98-%EA%B4%B4%EB%A6%AC)
-   [도메인 추출 방법](https://www.inflearn.com/community/questions/1429413/%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%B6%94%EC%B6%9C-%EB%B0%A9%EB%B2%95)
-   [추상화의 역할은 `현재알고있는 ` 변경을 캡슐화 해서 코드 수정으로 인해 받을 수 있는 영향을 최소화하는 것](https://www.inflearn.com/community/questions/1444730/%EC%95%88%EB%85%95%ED%95%98%EC%84%B8%EC%9A%94-%EA%B8%B0%EC%A1%B4%EC%9D%98-%EC%B6%94%EC%83%81%ED%99%94%EB%90%9C-%EC%97%AD%ED%95%A0%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%83%88%EB%A1%9C%EC%9A%B4-%ED%98%91%EB%A0%A5%EC%9E%90%EA%B0%80-%ED%95%84%EC%9A%94%ED%95%98%EA%B2%8C-%EB%90%98%EB%8A%94-%EA%B2%BD%EC%9A%B0%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%84%A4%EA%B3%84%ED%95%B4%EC%95%BC%ED%95%A0%EA%B9%8C%EC%9A%94)
-   [설계는 트레이드 오프](https://www.inflearn.com/community/questions/1335449/discountpolicy-%EA%B5%AC%ED%98%84-%EB%B0%8F-%EC%84%A4%EA%B3%84%EC%97%90-%EB%8C%80%ED%95%B4-%EA%B6%81%EA%B8%88%ED%95%9C-%EC%A0%90%EC%9D%B4-%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4)
