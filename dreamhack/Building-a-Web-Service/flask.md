# Flask

## 웹 서버
- **웹 서버(Web Server)** : 요청을 받으면 응답을 돌려주는 프로그램
  - 브라우저의 요청을 처리해서 알맞은 웹 페이지를 반환하도록 논리 흐름이 구현된 프로그램
    
### 웹 서버 개발
1. 프레임워크 없이 직접 만드는 경우
   - (1) 네트워크 연결 : 브라우저와 통신하기 위해 TCP 소켓을 맺음
   - (2) 데이터 수신: 소켓을 통해 들어오는 원시 데이터를 받음
   - (3) 프로토콜 검사: 받은 데이터가 올바른 HTTP 요청인지 규격에 맞게 검사함
   - (4) 정보 추출: 어떤 페이지(/home 등)를 원하는지 경로를 찾아냄
   - (5) 파일 처리: 서버 컴퓨터의 저장소에서 해당 HTML 파일을 찾아서 엶
   - (6) 메모리 로드: 파일 내용을 읽어 메모리에 임시로 담음
   - (7) 응답 구성: HTTP 표준 규격에 맞춰 응답 메시지 형식을 만듦
   - (8) 데이터 포함: 준비된 HTML 내용을 응답 메시지에 넣음
   - (9) 전송: 최종 완성된 데이터를 다시 브라우저로 보냄
2. 웹 프레임워크를 사용하는 경우

### 웹 프레임워크
> Don't reinvent the wheel
- 정의 : 웹 개발을 위해 반복적이거나 기본적으로 필요한 기능과 뼈대를 미리 만들어 제공하는 개발 환경
- 사용 이유 : 불필요한 과정에 신경쓰지 않고, 오직 웹 서버의 논리 흐름 설계에 집중할 수 있어서 생산적인 개발이 가능해짐

## Flask
- Flask : Python을 기반으로 하는 웹 프레임워크
  ```
  flask-app
  ├── app.py
  └── templates
      └── simple_page.html
  ```
  - flask-app : 최상위 디렉토리
  - flask-app/app.py : 웹 서버 파일
  - flask-app/templates : HTML 문서들을 담는 디렉토리 (별도로 Flask 프레임워크에서 설정하지 않는 이상 반드시 templates라는 이름으로 존재해야 하는 디렉토리)
  - flask-app/templates/simple_page.html : HTML 문서 파일

- `$ python3 app.py` : app.py를 python으로 실행하면 Flask 웹 서버가 구동됨
  - `Running On all addresses (0.0.0.0)` : Flask 서버가 모든 공인 IP로부터 요청을 받아들인다는 의미
    - 이 문구가 뜨면 다른 외부 네트워크의 이용자가 본 웹 서버로 접속할 수 있음
    - `0.0.0.0` : 네트워크 상의 모든 IOv4 주소 
  - `Running on http://127.0.0.1:31337` : 로컬호스트에서도 웹 서버에 접근할 수 있다는 의미
    - 웹 서비스가 31337포트로 열려 있으며, URL 주소인 `http://127.0.0.1:31337`로 접근할 수 있음
    - 웹 서버가 실행 중인 현재 컴퓨터에 접근할 수 있음
    - `127.0.0.1` : 로컬 호스트, 컴퓨터 자기 자신을 가리키는 루프백 주소
  - `Running on http://10.0.2.15:31337` : 웹 서버의 특정 네트워크 인터페이스에 할당된 로컬 IP 주소
    - 로컬 네트워크 내에서도 본 웹 서버에 접근할 수 있음 

### app.py 분석
```
from flask import Flask, render_template          # Flask 웹 프레임워크 모듈을 불러옴

app = Flask(__name__)

@app.route('/simple_page')                        # /simple_page 경로로의 요청이 들어오면,
def get_simple_page():                            # get_simple_page()를 실행함
    return render_template('simple_page.html')    # HTML 문서인 simple_page.html을 반환함

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=31337)           # 웹 서버를 구동함
```
- `from flask import Flask, render_template`
  - Python에서 `flask` 모듈로부터 `Flask` 클래스와 `render_template` 함수를 가져옴
  - `Flask` : Flask 애플리케이션을 생성하는 데 사용되는 클래스
  - `render_template` : HTML 파일을 렌더링하여 클라이언트에게 반환하는 함수
- `app = Flask(__name__)`
  - `Flask` 클래스의 인스턴스를 생성하여 app 변수에 할당함
- `@app.route('/simple_page')`
  - Python의 데코레이터 기능을 활용한 코드
  - `/simple_page` 경로로 들어오는 HTTP 요청을 바로 아래 위치한 함수인 `get_simple_page()`로 연결함
- ```
  def get_simple_page():
      return render_template('simple_page.html')
  ```
  - 데코레이터 `@app.route('/simple_page')`에 의해, `/simple_page` 경로로 요청이 들어오면 `get_simple_page()`가 실행됨
  - `get_simple_page()` : simple_page.html이라는 이름의 HTML 파일을 `render_template()`로 렌더링하여 이용자에게 반환함
- ```
  if __name__ == '__main__':
      app.run(host='0.0.0.0', port=31337)
  ```
  - Flask 웹 서버를 실행함
  - `host` 인자를 `'0.0.0.0'`으로 설정함으로써 외부의 모든 이용자가 본 웹 서버에 접근할 수 있도록 설정함
  - `port` 인자를 `31337`로 설정하여 31337번 포트에 웹 서비스가 열리도록 지정함

## 템플릿 렌더링 
- **템플릿 렌더링(Template Rendering)** : 서버의 Python 코드에서 생성한 변수나 값을 HTML 코드에 삽입하여 최종 HTML 문서를 만드는 과정
- **HTML 템플릿** : 뼈대인 HTML 코드 (코드 안에 실제 데이터가 채워지기 전, 기본적인 뼈대만 갖춘 상태의 HTML 문서)
- **최종 HTML 문서** : HTML 템플릿에 실제 데이터가 채워져서 최종적으로 완성된 HTML 문서

### 템플릿 렌더링 기호 
