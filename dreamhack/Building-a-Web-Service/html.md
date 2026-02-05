# HTML

## HyperText Markup Language (HTML)
- **HTML** : 웹 페이지를 구성하는 언어
  - 웹 페이지에 존재하는 텍스트, 상자, 입력 폽, 버튼, 하이퍼 링크, 이미지 등 웹 페이지에 존재하는 구성 요소의 위치와 구성을 정의하고, 브라우저가 이를 해석하여 시각적으로 보여줌 
- 웹 사이트 접속 과정
  - 요청 : 사용자가 브라우저 주소창에 naver.com을 입력하면, 브라우저가 해당 서버에 페이지를 보여달라고 요청함
  - 응답 : 서버는 요청을 확인하고 HTML 문서를 브라우저에게 보내줌
  - 해석 : 브라우저는 전달받은 복잡한 텍스트(HTML)를 읽어서 우리가 보는 예쁜 화면으로 바꿔서 보여줌

## HTML 문법
### HTML 요소
- HTML 요소 : 웹 페이지를 구성하는 기본 단위
- ex. 텍스트, 상자, 입력 폼, 버튼, 하이퍼링크, 이미지 등
#### HTML 요소 구조 - 태그
- 태그 : `<태그이름>내용</태그이름>`
- HTML 요소 = 여는 태그 + 내용 + 닫는 태그
- 여는 태그(Start Tag) : `<태그이름>`의 형태로, 한 요소의 시작을 나타냄
- 내용(Content) : 요소에 들어갈 내용 (ex. 텍스트)
- 닫는 태그(End Tag) : `</태그이름>`의 형태로, 요소의 끝을 나타냄
- 태그 종류
  - `<h1>` : 페이지나 섹션의 제목을 표시함 (내용 텍스트가 크고 굵은 글씨로 표시됨)
    - ex. `<h1>Title of the document~</h1>`
      - 결과 : <h1>Title of the document~</h1>
  - `<p>` : 문단을 나타내는 태그로, 내용 텍스트를 그룹화하여 단락으로 표시함
    - ex. `<p>This is the first paragraph</p>`
      - 결과 : <p>This is the first paragraph</p>
  - `<strong>` : 텍스트를 굵음체로 강조할 때 사용됨
    - ex. `<strong>Text to be emphasized.</strong>`
      - 결과 : <strong>Text to be emphasized.</strong>
  - `<a>` : 다른 페이지나 URL로 이동하는 하이퍼링크를 만듦
    - ex. `<a href="https://dreamhack.io">Go to the Dreamhack website</a>`
      - 결과 : <a href="https://dreamhack.io">Go to the Dreamhack website</a>
  - `<img>` : 웹 페이지에 이미지를 삽입함
    - ex. `<img src="love_logo.jpg">`  
      - 결과 :  
          <img width="30" height="30" alt="image" src="https://github.com/user-attachments/assets/58ef1798-1457-42f4-a0e6-33399ead5c83" />
  - `<button>` : 클릭 가능한 버튼을 만듦
    - ex. `<button type="button">Click here</button>`
      - 결과 : <button type="button">Click here</button>
#### HTML 요소 구조 - 속성 
- 속성(Attribute) : 태그에 추가 정보를 제공하거나 특정 동작을 지정하기 위해 사용됨
- 구조 : `<태그이름 키1="값" 키2="값2" 키3="값3">내용</태그이름>
- 키-값 쌍의 형태로 작성되며, 여는 태그 안에 위치함
- 여러 개의 속성이 존재할 수 있음

#### 유의해야할 점 
- 태그는 중첩될 수 있지만, 순서를 지켜야 함 
- 속성은 여는 태그 안에 작성됨
- HTML은 대소문자를 구분하지 않음 (소문자로 작성하는 것이 권장됨)
- 일부 태그는 닫는 태그가 존재하지 않음

### HTML 기본 문서 구조 
```html
<!DOCTYPE html>
<html>
<head>
  <title>My First Page</title> 
</head>
<body>
  <h1>Welcome to HTML!</h1>
  <p>Hello, World!</p>
</body>
</html>
```
- `<!DOCTYPE html>` : HTML 문서의 유형을 브라우저에게 알려주는 선언문
  - HTML 태그가 아니며, 브라우저가 문서를 해석할 때 어떤 버전의 HTML을 기준으로 해야 하는지 정보를 제공하는 역할을 맡음
- `<html>` : HTML 문서의 시작을 알림
  - HTML 문서의 끝을 나타내는 `</html>`과 짝을 이룸
  - 모든 HTML 문서는 `<html>` 태그를 포함해야 함
- `<head>` : HTML 문서의 메타데이터를 작성하는 부분 (HTML5 생략할 수 있음)
  - 메타데이터 : 데이터에 대한 데이터 (데이터의 특성, 구조, 용도 등 추가적인 정보를 제공하는 데이터)
- `<title>` : 웹 페이지에 보이는 제목을 정의하는 태그 
  - `<head>` 내에 존재해야 함
- `<body>` : HTML의 본문을 정의하는 부분
  - 모든 HTML 문서는 `<body>` 태그를 포함해야 함
