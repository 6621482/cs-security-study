# Docker
## Doker
- **Docker** : 애플리케이션과 그 실행에 필요한 환경(OS 설정, 라이브러리 등)을 하나로 묶어 어디서나 동일하게 실행할 수 있게 해주는 컨테이너 기반 플랫폼
- **도커 파일(Dockerfile)**
  - Docker Image를 빌드하는 데 단계적으로 필요한 명령어가 존재하는 파일 
  - 텍스트 파일
  - 어떤 환경에서, 무엇을 설치하고, 무엇을 실행할지 정의
- **도커 이미지(Docker Image)**
  - 도커 컨테이너의 전 단계
  - 컨테이너를 생성하고 실행하기 위한 모든 것을 포함 (파일, 환경 변수, 명령어 등)
  - 아직 실행되지 않은 상태
- **도커 컨테이너(Docker Container)**
  - 도커 이미지로부터 만들어진 실행 가능한 인스턴스 (실행 중인 이미지)

- **도커 레지스트리(Docker Registry)**
  - 도커 이미지를 저장하는 저장소

 ## 도커를 쓰는 이유
- 실행 환경 차이 문제 해결
  - 개발 환경과 실행 환경이 달라 발생하는 문제를 방지할 수 있음
  - Docker 이미지는 어디서 실행해도 동일한 환경을 보장
- 설치 및 실행 과정 단순화
  - Dockerfile과 이미지 하나로 빌드 -> 실행 가능
  - 환경 세팅 시간을 줄일 수 있음
 
## Dockerfile
```
FROM kalilinux/kali-rolling

ENV user testuser
ENV service_port 3333

RUN apt-get update
RUN apt-get install -y socat

RUN adduser $user

ADD ./deploy/secret /home/$user/secret
ADD ./deploy/app /home/$user/app

RUN chown -R root:$user /home/$user
RUN chown root:$user /home/$user/secret
RUN chown root:$user /home/$user/app

RUN chmod 755 /home/$user/app
RUN chmod 440 /home/$user/secret

WORKDIR /home/$user
USER $user
EXPOSE $service_port

CMD socat -T 30 TCP-LISTEN:$service_port,reuseaddr,fork EXEC:/home/$user/app
```
- 운영체제, 버전, 환경 변수, 파일 시스템, 사용자 등을 정의
- 파일명 : 확장자 없이 __Dockerfile__
  - `docker build` 실행 : 이름이 Dockerfile인 파일을 찾아 이미지를 빌드
- 도커 파일 구성 기본 형식
  ```
  # 주석
  명령어 인자
  ```
- Dockerfile 명령어
  - `FROM 이미지:태그`
    - Dockerfile은 `FROM` 명령어로 시작해야 함
    - 생성할 이미지의 기반이 되는 base 이미지를 지정
    - 보통 사용할 운영체제의 공식 이미지를 Dockerhub에서 가져옴
    - ex. `FROM kalilinux/kali-rolling`
  - `ENV 변수명 값` 또는 `ENV 변수명=값`
    - Dockerfile 내에서 사용하는 환경 변수를 지정
    - 파일 내에서 변수는 `$변수명` 또는 `${변수명}` 형태로 표현
    - 예시
      - (1) ENV로 변수 선언
        - `ENV PYTHON_VERSION 3.11.2` : 이 Dockerfile 안에서는 PYTHON_VERSION = 3.11.2 라고 기억
      - (2) 변수 사용
        - ex. `RUN echo $PYTHON_VERSION`
        - 실제 실행 결과 : 3.11.2
        - ex. `RUN echo /opt/python/$PYTHON_VERSION/bin`
        - 실제 실행 결과 : /opt/python/3.11.2/bin
  - `RUN 명령어` 또는 `RUN ["명령어", "인자1", "인자2"]`
    - 이미지를 빌드할 때 실행할 명령어를 작성
    - 필요한 패키지 설치, 파일 권한 설정 등의 작업 수행
  - `COPY <src> <dest>`
    - src 파일이나 디렉토리를 이미지 파일 시스템의 dest로 복사
  - `ADD <src> <dest>`
    - src 파일이나 디렉토리, URL을 이미지 파일 시스템의 dest로 복사
  - `WORKDIR 디렉토리`
    - Dockerfile 내의 명령을 수행할 작업 디렉토리를 지정 (리눅스의 `cd`와 유사)
  - `USER 사용자명|UID` 또는 `USER 사용자명|UID[:그룹명|GID]`
    - 명령을 수행할 사용자 혹은 그룹을 지정
  - `EXPOSE 포트` 또는 `EXPOSE 포트/프로토콜`
    - 컨테이너가 실행 중일 때 들어오는 네트워크 트래픽을 리슨할 포트와 프로토콜을 지정
    - 기본적으로 TCP가 지정됨
  - `ENTRYPOINT 명령어 인자1 인자2` 또는 `ENTRYPOINT ["명령어", "인자1", "인자2"]`
    - 컨테이너가 실행될 떄 수행할 명령어를 지정
    - ex. `ENTRYPOINT ["echo", "hello"]`
  - `CMD 명령어 인자1 인자2` 또는 `CMD ["명령어", "인자1", "인자2"]` 또는 `CMD ["인자1", "인자2"]`
    - 컨테이너가 실행될 때 수행할 명령어를 지정하거나, ENTRYPOINT 명령어에 인자를 전
