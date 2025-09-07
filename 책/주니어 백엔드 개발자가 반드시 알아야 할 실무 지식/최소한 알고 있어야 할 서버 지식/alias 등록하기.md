# alias 등록하기
서버를 운영하면 자주 사용하는 명령어가 생긴다. 이럴때 alias를 사용하면 편해진다. 리눅스에서 alias는 명령어에 대한 별칭이다. alias는 2가지 방식으로 등록할 수 있다. 

- 현재 터미널 세션에만 적용하는 법칙
    - >$ alias cdweb = 'cd /var/www/html'

- 매번 alias를 설정하지 않고 자동으로 적용하기 (쉘 구성 파일에 alias 등록)
    - 홈 디렉토리에 위치한 .bashrc 파일 하단에 alias 설정 추가
    ```shell
    # .bashrc
    # Source global definitions
    if [ -f /etc/bashrc ]; then
                . /etc/bashrc
    fi
    # alias
    alias cdweb='cd /var/www/html' 
    ```

인자 없이 alias 명령어만 실행하면 현재 정의되어있는 모든 별칭을 볼 수 있다.
>$ alias
