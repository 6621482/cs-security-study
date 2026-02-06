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

## 템플릿 렌더링 기호 
- `render_template()` : HTML 템플릿에 존재하는 특수한 기호들을 규칙에 맞춰 렌더링해줌
- Flask는 내부적으로 Jinja2라는 템플릿 엔진을 사용함

### {{ 표현식 }}
- HTML 문서 안에 이 기호를 사용하면, Python 코드에서 전달받은 데이터나 간단한 연산 결과를 그 자리에 직접 삽입(출력)할 수 있음
  
- <app.py>
  ```
  from flask import Flask, render_template

  app = Flask(__name__)

  @app.route('/expression')
  def get_expression():
      x = 3
      y = 13
      z = 130
      arr = [3, 6, 9, 12]
      return render_template('expression.html', x=x, y=y, z=z, arr=arr)

  if __name__ == '__main__':
      app.run(host='0.0.0.0', port=31337)
  ```
- <templates/expression.html>
  ```
  <!DOCTYPE html>
  <html>
  <head>
    <title>Expression Usages</title>
  </head>
  <body>
    <h1>Expression Usages</h1>
    <p>x: {{ x }}</p>
    <p>x + 100: {{ x + 100 }}</p>
    <p>123 + 456: {{ 123 + 456 }}</p>
    <p>y + z: {{ y + z }}</p>
    <p>arr[1]: {{ arr[1] }}</p>
  </body>
  </html>
  ```
  - 변수출력 `{{ x }}` : 전달받은 변수 값을 그대로 출력 (결과 : 1)
  - 산술연산 `{{ x + 100 }}` :	변수와 숫자를 더한 결과 출력 (결과 : 113)
  - 숫자 연산 `{{ 123 + 456 }}` :	상수끼리의 연산 결과 출력 (결과 : 579)
  - 복수 변수 연산	`{{ y + z }}` :	두 변수를 더한 결과 출력 (결과 : 143)
  - 리스트 인덱싱	`{{ arr[1] }}`	리스트의 특정 인덱스 원소 출력 (결과 : 6)

- 데이터 전달 : render_template('파일명.html', 변수명=값) 형태로 데이터를 보냄
- 출력 : HTML에서는 {{ }}를 사용하여 Python의 변수나 연산 결과를 가져옴
- 결과 확인 : 브라우저는 템플릿 기호가 모두 해석된 최종 HTML만 전달받아 화면에 출력함

### {% 구문 %}
- 프로그램의 로직(조건, 반복, 상속 등)을 제어하기 위해 사용되는 기호

#### 조건문 
- 데이터의 참/거짓 여부에 따라 HTML 문서에 데이터를 다르게 삽입할 때 사용함
- 반드시 **{% endif %}** 로 끝나야 함  
1. if문
   ```
   {% if 조건 %}
     <!-- 조건이 참일 때 실행 -->
   {%endif %}
   ```
2. if-else문
   ```
   {% if 조건 %}
    <!-- 조건이 참일 때 실행 -->
   {% else %}
    <!-- 조건이 거짓일 때 실행 -->
   {% endif %}
   ```
3. if-elif-else문 (다중조건)    
   ```
   {% if 조건1 %}
    <!-- 조건1이 참일 때 실행 -->
   {% elif 조건2 %}
    <!-- 조건2가 참일 때 실행 -->
   {% else %}
    <!-- 모든 조건이 거짓일 때 실행 -->
   {% endif %}
   ```

#### 반복문 
- 서버 Python 코드에서 전달된 리스트, 딕셔너리와 같은 데이터를 순회하면서 HTML 콘텐츠를 동적으로 생성할 수 있음
```
<ul>
    {% for 아이템명 in 순회가능한데이터 %}
        <li>{{ 아이템명 }}</li>
    {% endfor %}
</ul>
```
- `<ul>` : 순서가 없는 목록(Unordered List)를 만드는 태그 (`<li>`와 함께 사용함) 
- `for 아이템명 in 순회가능한데이터` : `순회가능한데이터`는 순회할 대상, `아이템명`은 각 반복의 현재 요소
- `<li>` : 목록의 한 항목을 나타내는 태그(list item) -> 자동으로 들여쓰기 + 점 (또는 번호)을 붙여줌 
- `{% endfor %}` : 반복문의 끝

#### 템플릿 상속 구문 
- 여러 HTML 파일 간에 공통 템플릿을 공유하고 재사용할 수 있도록 돕는 기능
- 부모 템플릿과 자식 템플릿이 필요함
```
{% block 블록명 %} {% endblock %}
```
- 부모 템플릿과 자식 템플릿 모두에서 사용될 수 있음
- 부모 템플릿에서는 자식 템플릿에서 덮어쓸 부분을 정의함
- 자식 템플릿에서는 부모 템플릿에 반영할 데이터를 지정할 수 있음
```
{% extends '부모템플릿파일이름' %}
```
- 자식 템플릿에서 사용할 수 있음
- 부모 템프릿을 상속함 

### {# 주석 #}
- 최종 결과물에 출력되지 않는 주석이나 참고 정보를 작성할 때 사용됨
```
<!DOCTYPE html>
<html>
<head>
  <title>Flask 템플릿</title>
  {# 이곳은 사용자에게 표시되지 않는 주석입니다. #}
</head>
<body>
  <h1>템플릿 주석 예제</h1>

  {# 이곳도 사용자에게 표시되지 않는 주석입니다. #}
  <p>이 문장은 사용자에게 표시됩니다.</p>
</body>
</html>
```
- `<!--이것은 주석입니다.-->` : HTML에서 제공하는 주석
  - 브라우저의 웹 페이지 화면에는 안 보이지만 HTML 소스 코드에는 그대로 노출이 되는 정보임
- `{# 주석 #}` : 렌더링 과정에서 제거되기 때문에 최종 HTML 소스 코드에 노출되지 않음

## 이용자로부터 입력받기 
### URL 경로 매개변수를 통한 입력 처리 
- **URL 경로 매개변수(URL Path Parameter)** : 데이터를 URL 경로의 일부로 포함시켜 서버에 전달하는 방식
- URL 내부의 특정 위치가 변수(인자값) 역할을 함
- 사용자가 URL의 해당 위치에 다른 값을 입력하면, 서버는 그 값을 받아 서로 다른 데이터를 조회하거나 처리함
- 예시 : `https://example.com/products/123`
  - `/products` : 상품 목록을 나타내는 고정된 경로
  - `123` : 특정 상품의 ID를 나타내는 매개변수
  - 123 대신 40을 넣으면, 서버는 상품 ID가 40인 상품 정보를 찾아 화면에 보여줌
- 예시 : 경로 매개변수로 이름을 전달받아서 웹 페이지에 `Hello, {{ 이름 }}!` 형태로 출력하기
  <app.py>
  ```
  from flask import Flask, render_template

  app = Flask(__name__)

  @app.route('/welcome/<name>')
  def get_welcome_name(name):
      return render_template('result.html', name=name)

  if __name__ == '__main__':
      app.run(host='0.0.0.0', port=31337)
  ```
  - 변수 정의 : `@app.route` 데코레이터에서 경로 매개변수로 사용할 부분을 `<`와 `>`로 감사면 경로 매개변수로 정의할 수 있음 
  - 함수 인자로 받기 : 정의된 <name>을 해당 경로와 연결된 함수(get_welcome_name)의 인자로 넣어야 함수 내부 코드에서 사용할수 있음
  - 템플릿으로 전달 : `render_template('result.html', name=name)`을 통해 사용자로부터 받은 name 값을 HTML 파일로 넘겨줌
  
   <templates/result.html>
  ```
  <!DOCTYPE html>
  <html>
  <head>
    <title>URL Path Parameter Example</title>
  </head>
  <body>
    <h1>URL Path Parameter Example</h1>
    <h2>Hello, {{ name }}!</h2>
  </body>
  </html>
  ```
  - 전달받은 매개변수를 `<h2>Hello, {{ name }}!</h2>` 형태로 렌더링함 (표현식 사용) 
  - 서버에서 전달받은 name 변수의 실제 값을 해당 위치에 삽입하여 HTML을 생성함
- 요약
  - URL 형태 : `/welcome/mark`
  - Flask 추출 : 함수 인자 (name)
  - 주요 용도 : 특정 리소스 식별 (ID 등) 

### URL 쿼리 매개변수를 통한 입력 처리 
- **URL 쿼리 매개변수(URL Query Parameter)** : URL의 `?` 뒤에 오는 매개변수로, 이용자가 데이터를 서버로 전달하는 방식 중 하나
- 예시 : `http://example.com/search?query=flask&page=2`
  - `?` 뒤에 오는 `query`가 매개변수 키(Key)를 나타내고, `flask`가 매개변수 값(Value)을 의미함
  - `&`로 이어진 `page=2`는 또 다른 매개변수 키-값 쌍임
- 예시 : 쿼리 매개변수로 이름을 전달받아 웹 페이지에 `Hello, {{ 이름 }}!` 형태로 출력하기     
  <app.py>
  ```
  from flask import Flask, render_template, request

  app = Flask(__name__)

  @app.route('/welcome')
  def get_welcome():
      user_name = request.args.get('user_name')
      return render_template('result.html', name=user_name)

  if __name__ == '__main__':
      app.run(host='0.0.0.0', port=31337)
  ```
  - 쿼리 매개변수를 사용하려면 `request` 클래스가 필요함 (첫 줄에서 import 해줌)
  - `request.args` : 이용자로부터 전달받은 쿼리 매개변수들을 담고 있는 객체
    - `.get()` 메서드 : `user_name`이라는 매개변수를 가져와서 `user_name`이라는 변수에 대입함
  - 렌더링 : render_template() 함수는 templates/ 폴더 내의 HTML 파일을 찾아 변수를 매핑한 뒤 최종 HTML을 생성함
   
  <templates/result.html>
  ```
  <!DOCTYPE html>
  <html>
  <head>
    <title>URL Query Parameter Example</title>
  </head>
  <body>
    <h1>URL Query Parameter Example</h1>
    <h2>Hello, {{ name }}!</h2>
  </body>
  </html>
  ```
  - 앞서 전달받은 경로 매개변수를 `<h2>Hello, {{ name }}!</h2>` 형태로 렌더링함
- 요약
  - URL 형태 : `/welcome?user_name=yewon`
  - Flask 추출 : `request.args.get('key')`
  - 주요 용도 : 검색, 필터링, 정렬 등

### `<form>` 태그를 통한 POST 데이터 입력 처리 
- `<form>` 태그 : 이용자가 웹 브라우저 상에서 데이터를 입력한 후 **서버로 전송할 수 있는 양식(Form)** 을 만듦
- `<form>` 태그의 핵심 속성
  - `action` : 입력할 값을 전송할 URL을 지정함
  - `method` : 입력 값을 전송할 때 사용하는 HTTP 메세지의 메서드를 설정 (주로 GET, POST 사용)
#### `<form>` 입력 요소 태그
- `<input>` : 입력 양식을 만드는 태그
  - `<form>` 태그 단독으로는 입력 양식을 만들 수 없고, `<input>`태그를 함께 사용해야 입력 가능한 폼을 만들 수 있음
  - `<input>` 태그에서 자주 사용되는 속성
    - `type` : 입력 필드의 유형을 지정함
      - 유형 : `text` (텍스트를 입력받는 폼) / `number` (숫자만 입력받는 폼) / `password` (비밀번호를 입력받는 폼이어서 입력 값이 별표 기호로 표시되는 폼)
    - `name` : 서버로 전송될 때, 데이터의 키(Key)를 설정함
      - `<form>` 태그를 통해 서버로 전달되는 데이터는 키-값 쌍 형태로 전달됨
      - name 속성이 어떤 키로 전달될지를 설정함
    - `id` : 입력 폼의 고유 식별자
- `<button>` : HTML에서 클릭 가능한 버튼을 생성하는 데 사용하는 태그

##### `<form>` 태그 예시 - 텍스트 입력 폼 
```
<!DOCTYPE html>
<html>
<head>
  <title>Form Exercise</title>
</head>
<body>
  <h1>Form Exercise</h1>
  <form action="/submit" method="post">
    Name:
    <input type="text" id="username" name="username">
    <button>Submit</button>
  </form>
</body>
</html>
```
- `<form action="/submit" method="post">` : 제출한 데이터는 `/submit` 경로로 POST 메서드를 통해 전달됨
- `<input type="text" id="username" name="username">` : 텍스트를 입력할 수 있는 입력 양식, 입력값은 서버에 전달될 때 `username=입력값` 형태로 전달됨
- <button>Submit</button> : submit 버튼 생성

##### `<form>` 입력 요소 태그 예시 - 비밀번호 입력 폼 
```
<!DOCTYPE html>
<html>
<head>
  <title>Form Exercise</title>
</head>
<body>
  <h1>Form Exercise</h1>
  <form action="/login" method="post">
    Username:
    <input type="text" id="userid" name="userid">
    <br>
    Password:
    <input type="password" id="userpw" name="userpw">
    <button>Log In</button>
  </form>
</body>
</html>
```
- `<form action="/login" method="post">` : 제출한 데이터는 `/login` 경로로 POST 메서드를 통해 전달됨
- `<br>` : 줄바꿈

##### `<form>` 입력 요소 태그 예시 - 파일 업로드 폼 
```
<!DOCTYPE html>
<html>
<head>
  <title>Form Exercise</title>
</head>
<body>
  <h1>Form Exercise</h1>
  <form action="/upload" method="post">
    Choose a file:
    <input type="file" id="file" name="file">
    <button>Upload</button>
  </form>
</body>
</html>
```

#### 세 가지 방법 비교
| 구분 | URL 경로 매개변수 | URL 쿼리 매개변수 | `<form>` POST 데이터 |
|------|------------------|------------------|----------------------|
| 전달 형태 | URL 경로의 일부 (예: `/products/123`) | URL 끝에 `?key=value` 형태 | HTML 양식을 통해 서버로 전송 |
| Flask 추출법 | 함수 인자로 직접 전달받음 | `request.args.get()` 사용 | `request.form` 등을 통해 추출 (Body 데이터) |
| 주요 속성 | `< >`를 이용한 변수 정의 | 별도 정의 없이 호출 가능 | `action`(대상), `method`(방식) 설정 필요 |
| 주요 용도 | 특정 리소스 식별 (상품 ID 등) | 검색, 필터링, 간단한 옵션 전달 | 로그인, 게시글 작성 등 보안/대용량 데이터 |
- URL 경로 및 쿼리 매개변수: 데이터가 URL 부분에 포함되어 전송됨 -> 브라우저 주소창에 데이터가 그대로 노출됨
- <form> 태그 (POST 방식): 데이터가 HTTP 메시지의 바디(Body) 부분에 포함되어 전송됨 -> 주소창에 데이터가 드러나지 않음 

### `<form>` 태그를 통한 POST 데이터 입력 처리 - 서버에서 처리하기 
- `<form>` 태그로 전달받은 POST 데이터는 `request.form` 객체에 전달됨
- `request.form` 객체의 `.get()` 메서드를 사용하면 전달받은 데이터를 가져올 수 있음
  - ex. 전달받은 데이터 중 키가 username인 데이터를 사용하려면? `request.form.get('username')`으로 가져올 수 있음

