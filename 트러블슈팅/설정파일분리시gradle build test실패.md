# 설정 파일 분리시 gradle build중 test실패

기본 application설정파일이 없고 dev,prod 이렇게 두개의 설정파일만 존재할때 build test 단계에서 실패
test코드의 설정파일 필요한 곳에 @ActiveProfiles("프로필명")해주면 해결
