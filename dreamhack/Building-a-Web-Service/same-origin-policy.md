# Same Origin Policy

## Same Origin Policy (SOP)
- 동일 출처 정책(SOP) : 한 웹 페이지가 다른 출처의 리소스에 마음대로 접근하지 못하도록 제한하는 정책 
- 필요한 이유
  - 브라우저는 인증 정보(쿠키 등)를 내부에 보관하고, 해당 사이트에 요청을 보낼 때 자동으로 이 정보를 포함함
  - 브라우저가 웹 리소스를 통해 간접적으로 타 사이트에 접근할 때도 쿠키를 함께 전송함
  - 악의적인 페이지가 클라이언트의 권한을 이용해 대상 사이트에 HTTP 요청을 보내고, HTTP 응답 정보를 획득하는 코드를 실행할 수 있음 (정보 유출이 생길 수 있음)
- SOP의 origin 구분 방법
  - origin = 프로토콜(Protocol, Scheme) + 포트(Port) + 호스트(Host)
  - 구성 요소가 모두 일치해야 동일한 오리진이라고 함 
  - 예시 : 기준 URL = `https://same-origin.com` (기본 포트 443)
    - `https://same-origin.com/frame.html` : same origin (경로만 다를 뿐, 3요소가 모두 일치함)
    - `http://same-origin.com/frame.html` : cross origin (프로토콜이 다름)
    - `https://cross.same-origin.com/...` : cross origin (호스트가 다름)
    - `https://same-origin.com:1234/` : cross origin (포트가 다름)

### 실습 
- `window.open(<경로>)` : 새로운 창을 띄우는 함수
- `<object>.location.href` : 객체가 가리키고 있는 URL 주소를 읽어옴

## Same Origin Policy 제한 완화 
- 웹 서비스의 기능 구현을 위해 다른 origin의 리소스를 사용해야 하는 경우가 있음
- 특정 조건 하에서 제한을 완화해줌
- 허용 되는 예외 태그
  - `<img>` : 다른 사이트의 이미지를 표시할 수 있음
  - `<style>` : 외부 CSS 파일을 적용할 수 있음
  - `<script>` : 외부 자바 스크립트 파일을 실행할 수 있음
- 동일 서비스 내에서도 서로 다른 호스트를 사용하는 경우, 브라우저는 이를 별개의 오리진으로 인식하여 데이터 공유를 차단함
  - 서브 도메인 분리 : `dreamhack.io`(메인), `mail.dreamhack.io`(메일), `blog.dreamhack.io`(블로그) 등은 모두 다른 오리진
  - 상호작용의 필요성 : 메인 페이지에서 메일 서비스의 데이터를 가져와 '읽지 않은 메일 개수'를 표시하려면 반드시 SOP 제한을 완화하여 리소스를 공유할 방법이 필요함
  - 공유방법 : **교차 출처 리소스 공유(Cross Origin Resource Sharing, CORS)**

### Cross-Origin Resource Sharing (CORS) 
- **교차 출처 리소스 공유 (Cross Origin Resource Sharing, CORS)** : HTTP 헤더에 기반하여 Cross Origin 간에 리소스를 공유하는 방법
- 발신측에서 특정 헤더를 설정해 요청하면, 수신측에서 정해진 규칙에 따라 데이터를 가져갈 수 있도록 허용 여부를 결정함
- 핵심 메커니즘 : **CORS Preflight (예비 요청)**
  - 브라우저는 교차 출처 요청 시, 실제 요청을 보내기 전 안전성을 확인하기 위해 OPTIONS 메소드를 이용한 예비 요청(Preflight)을 보냄
  - 질의 : 클라이언트는 Access-Control-Request-Method와 Access-Control-Request-Headers 헤더를 통해 어떤 메소드와 헤더를 사용할 것인지 서버에 미리 물어봄
  - 검증 : 서버는 허용 가능한 출처, 메소드, 헤더 정보를 응답 헤더에 실어 보냄
  - 결정: 브라우저는 서버의 응답이 자신의 요청 조건과 일치하는지 확인한 후, 비로소 실제 데이터 요청(예: POST)을 실행함
- 주요 CORS 응답 헤더
  | Header | 설명 |
  |--------|------|
  | Access-Control-Allow-Origin | 헤더 값에 해당하는 Origin에서 들어오는 요청만 처리 |
  | Access-Control-Allow-Methods | 헤더 값에 해당하는 메소드의 요청만 처리 |
  | Access-Control-Allow-Credentials | 쿠키 사용 여부를 판단합니다. 예시의 경우 쿠키의 사용을 허용 |
  | Access-Control-Allow-Headers | 헤더 값에 해당하는 헤더의 사용 가능 여부를 나타냄 |
  
#### 예시
  <브라우저에서 보내는 코드(JS)>
  ```
  xhr = new XMLHttpRequest();
  xhr.open('POST', 'https://theori.io/whoami');
  xhr.withCredentials = true;
  xhr.setRequestHeader('Content-Type','application/json');
  xhr.send('{"data":"WhoAmI"}');
  ```
  - `dreamhack.io` 사이트에서 `theori.io` 서버로 POST 요청을 보내겠다는 의미
  - cross-origin -> 브라우저가 먼저 확인 요청을 보냄

  <브라우저가 보내는 확인용 요청 = OPTIONS>
  ```
  OPTIONS /whoami HTTP/1.1
  Host: theori.io
  Access-Control-Request-Method: POST
  Access-Control-Request-Headers: content-type
  Origin: https://dreamhack.io
  ```
  - `theori.io`서버에게 `dreamhack.io`에서 POST + Content-Type 헤더로 요청 보내도 되냐고 묻는 것과 같음

  <서버의 답변>
  ```
  Access-Control-Allow-Origin: https://dreamhack.io
  Access-Control-Allow-Methods: POST, GET, OPTIONS
  Access-Control-Allow-Credentials: true
  Access-Control-Allow-Headers: Content-Type
  ```
  - Allow-Origin : dreamhack.io는 허용
  - Allow-Methods : POST, GET, OPTIONS 허용
  - Allow-Credentials : 쿠키 사용 허용
  - Allow-Headers : Content-Type 사용 허용  
  -> 이제 브라우저가 안심하고 진짜 POST 요청을 보냄

<과정 정리>  
  - (1) 브라우저 -> 서버에 OPTIONS로 확인
  - (2) 서버 -> "괜찮다"고 답
  - (3) 실제 POST 요청 전송

### JSON with Padding (JSONP) 
- **JSONP** : 스크립트 태그를 이용해 Cross-Origin 데이터를 가져오는 방법
  - `<script src="...">` : 자바 스크립트를 실행하는 장치 (인자로 JSON이 아닌 실행 가능한 자바스크립트 코드를 보내야 함) 
- 동작 원리
  - (1) 클라이언트 (함수 정의 및 요청)
    - 데이터를 처리할 콜백 함수(예: myCallback)를 미리 정의함
    - `<script>` 태그의 src 주소에 콜백 함수의 이름을 파라미터로 실어서 요청을 보냄 
    ```html
    <script>
    function myCallback(data){  // 함수 정의
      console.log(data.id)
    }
    </script>

    <script src="http://theori.io/whoami?callback=myCallback"></script>  // 데이터를 myCallback(...) 형태로 감싸서 보내달라는 의미 
    ```js 
  - (2) 서버 (데이터 감싸기)
    - 요청받은 데이터를 JSON 형태로 준비함
    - 전달받은 콜백 함수 이름으로 JSON 데이터를 감싸서 응답함
    ```
    myCallback({'id': 'dreamhack'});
    ```
    - 자바스크립트 코드
  - (3) 브라우저 (실행)
    - 응답받은 자바스크립트 코드(myCallback(...))를 실행함
    - 결과적으로 클라이언트가 정의해둔 함수가 호출되면서 데이터를 전달받게 됨
  - 과정 총 정리
    - 클라이언트 : callback=myCallback로 감싸서 달라고 함
    - 서버 : myCallback({'id':'dreamhack'}) 형태로 응답
    - 브라우저 : 그 코드를 실행
    - 결과 : myCallback 함수 안에서 데이터 처리됨 
- 현재는 거의 사용되지 않는 추세이며, 새로운 코드를 작성할 때는 보안성이 더 높은 CORS를 사용해야 함

##### CORS vs JSONP
| 구분   | JSONP         | CORS       |
| ---- | ------------- | ---------- |
| 방식   | `<script>` 우회 | HTTP 헤더 기반 |
| 데이터  | JS 코드         | 순수 JSON    |
| 보안   | 취약            | 상대적으로 안전   |
| 현대 웹 | 거의 안 씀        | 표준         |
