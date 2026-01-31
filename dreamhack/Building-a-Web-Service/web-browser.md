# Web Browser

## Web Browser
- **Web Browser** : 웹 서버와 통신하고, 웹 문서를 해석해 보여주는 클라이언트 프로그램
- 역할
  - 통신 대행 : 사용자 대신 HTTP 통신을 수행하여 필요한 리소스를 요청함
  - 리소스 시각화 : 서버로부터 받은 HTML, CSS, Javascript 같은 코드(리소스)를 해석하여 우리가 보는 화면으로 만들어줌(렌더링)
  - 보안 및 편의 : HTTP 연결 확인, 쿠키 관리, 개발자 도구 등 웹 사용과 개발에 필요한 다양한 기능을 제공함
- 이용자가 주소창에 `example.io`를 입력했을 때 웹 브라우저가 하는 기본 동작 과정 
  - URL 분석: 주소창에 입력한 주소(example.io)가 어떤 형식인지 분석
  - DNS 요청: 문자로 된 주소를 컴퓨터가 이해할 수 있는 IP 주소로 바꾸기 위해 DNS 서버에 물어봄
  - HTTP 요청: 찾아낸 IP 주소의 서버에 "페이지 정보를 달라"고 요청을 보냄
  - HTTP 응답 수신: 서버가 보내주는 리소스(데이터)를 받음
  - 렌더링: 받은 데이터를 바탕으로 화면을 그려 사용자에게 보여줌

## URL
- **URL(Uniform Resource Locator)** : 웹에 있는 리소스의 위치를 표현하는 문자열
- 브라우저로 특정 웹 리소스에 접근할 때는 URL을 사용하여 이를 서버에게 요청함 
- 구성 : Scheme, Authority, Path, Query, Fragment 등  
  `foo`://`example.com:8042`/`over/there`?`name=ferret`#`nose`
  - Scheme (`foo`) : 웹 서버와 어떤 프로토콜로 통신할 지 나타냄
  - Host (`example.com`) : Authority의 일부로, 접속할 웹 서버의 주소에 대한 정보를 가지고 있음
  - Port (`8042`) :  Authority의 일부로, 접속할 웹 서버의 포트에 대한 정보를 가지고 있음
  - Path (`over/there`) : 접근할 웹 서버의 리소스 경로로, '/'로 구분됨
  - Query (`name=ferret`) : 웹 서버에 전달하는 파라미터로, URL에서 ? 뒤에 위치함
  - Fragment (`nose`) : 메인 리소스 내의 특정 하위 리소스(ex. 페이지 내 특정 위치)를 식별하기 위한 정보로, # 뒤에 위치함

### Domain name
- Host : 웹 브라우저가 접속할 서버의 주소를 나타내며, Domain name이나 IP address 값을 가질 수 있음
- 네트워크 통신에서 장치를 식별하는 IP주소 (ex. 93.184.216.34)는 불규칙한 숫자로 되어 있어 외우기 어려우므로 기억하기 쉬운 도메인 이름을 정의하여 IP 대신 사용함
- 도메인 이름의 구성
  - URL 전체 구조 중에서 Authority의 Host 부분에 해당함
- **DNS(Domain Name Server)** : 사람이 입력한 도메인 이름을 IP 주소로 변환해줌
  - 작동 원리 : 브라우저가 DNS에 도메인 이름을 질의하면, DNS는 해당 도메인에 연결된 IP 주소를 응답함
  - 확인 방법 : 리눅스 터미널에서 `nslookup` 명령어를 사용하여 특정 도메인의 IP 주소를 확인할 수 있음
    ```
    $ nslookup example.com
    Server:		8.8.8.8
    Address:	8.8.8.8#53

    Non-authoritative answer:
    Name:	example.com
    Address: 93.184.216.34
    ```

### 웹 렌더링
- **Web rendering** : 서버로부터 받은 리소스를 이용자에게 시각화하는 행위
- 서버의 응답을 받은 웹 브라우저는 리소스의 타입을 확인하고, 적절한 방식으로 이용자에게 전달
- 예시 : 서버로부터 HTML과 CSS를 받음
  - 브라우저가 HTML을 파싱하고 CSS를 적용하여 이용자에게 보여줌 
- 웹 렌더링은 웹 렌더링 엔진에 의해 이루어짐
  - 브라우저별로 사용하는 엔진이 다름 
    - Safari : webkit
    - Chrome : blink
    - Firefox : Gecko
  - 엔진 목적 : HTML를 파싱하고 시각화하여 사용자에게 보여주기 
