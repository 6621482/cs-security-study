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

## Cross-Origin Resource Sharing (CORS)
- 웹 서비스의 기능 구현을 위해 다른 origin의 리소스를 사용해야 하는 경우가 있음
- 특정 조건 하에서 제한을 완화해줌
- 허용 되는 예외 태그
  - `<img>` : 다른 사이트의 이미지를 표시할 수 있음
  - `<style>` : 외부 CSS 파일을 적용할 수 있음
  - `<script>` : 외부 자바 스크립트 파일을 실행할 수 있음
  - 
