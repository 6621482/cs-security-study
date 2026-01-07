# Regular Expressions

## Regular Expressions
- **Regular Expressions** : 특정한 패턴으로 문자열을 표현하는 식
- 필요성
  - 로직을 수행하기 전에 사용자가 입력한 값을 검증하기 위해 사용 (정해진 형식의 문자열 검색 및 검증)
  - 긴 문자열에서 원하는 형식의 문자열을 검색하여 추출할 때 사용
    - 정규표현식에 부합하는 문자열 = 정규 표현식 또는 패턴에 '매치한다'고 표현 
  - 장점 : 검증 코드를 직접 작성하는 번거로움과 복잡함을 줄이고, 간단하게 형식을 표현하고 일치 여부를 확인할 수 있음
- 보안에서의 중요성
  - 공격 발생 지점 : 주로 사용자가 값을 자유롭게 입력할 수 있는 곳
  - 보안 원칙 : **Never Trust, Always Verify**
- 취약점과의 관계
  - 위험성 : 잘못된 정규 표현식으로 검증하면 올바르지 않은 데이터가 서버로 넘어가 보안 취약점으로 이어짐
 
## 정규 표현식 패턴
- 정규 표현식 형태 : `'패턴'` 또는 `/패턴/`
### 매치
- `문자 / 문자열`
  - 의미 : 해당 문자 혹은 문자열과 매치
  - 예시
    - dream -> hello <mark>dream</mark> 
- `.`
  - 의미 : 줄바꿈 문자를 제외한 모든 문자 중 하나 (모든 문자와 매치, 점 당 문자 하나)
  - 예시
    - a.c -> <mark>abc</mark>, <mark>a@c</mark>, ac, abbc
- `|`
  - 의미 : OR (앞 또는 뒤의 패턴과 매치)
  - 예시
    - cat|dog -> I have a <mark>cat</mark>, I like bird
- `[]`
  - 의미 : 대괄호 안 문자 중 하나
  - 예시
    - [aeiou] -> H<mark>i</mark>
- `[^ ]`
  - 의미 : ^ 뒤의 패턴을 제외한 나머지와 매치
  - 예시
    - [^aeiou] -> <mark>H</mark>i
- `^`
  - 의미 : 문자열의 시작
  - 예시
    - ^Hello -> <mark>Hello</mark> World, Say Hello
- `$`
  - 의미 : 문자열의 끝
  - 예시
    - $Hello -> Hello <mark>World</mark>, World peace
- `\`
  - 의미 : 특수 문자를 그대로 사용
  - 예시
    - \$100 -> Price is <mark>$100</mark>
- `[a-z] [A-Z] [0-9]`
  - 의미 : 문자 범위 중 하나 (두 문자 사이 범위의 문자와 매치)
  - 예시
    - [d-f] -> H<mark>99</mark>llo
- `\w`
  - 의미 : 문자 또는 숫자 또는 _ (`[A-Za-z0-9_]`)
  - 예시
    - \w -> <mark>ID_check</mark>:<mark>99</mark>
- `\d`
  - 의미 : 숫자 (`[0-9]`)
  - 예시
    - \d -> abc<mark>123</mark>
- `\s`
  - 의미 : 공백 문자 (`[\b\f\n\r\t\v]` 백스페이스(지우기), 페이지 넘김, 줄바꿈, 탭)
  - 예시
    - \s -> Hello<mark>(Tab)</mark>World

### 수량자 
- `*`
  - 의미 : 앞의 문자가 0번 또는 그 이상 반복됨 (반드시 앞 글자가 있어야 함)
  - 예시
    - ab*c -> <mark>ac</mark>, <mark>abc</mark>, <mark>abbc</mark>, adc
  - `.*` : 내용이 아예 없어도 되고, 엄청 길어도 됨
- `+`
  - 의미 : 앞의 문자가 1개 이상이면 매치
  - 예시
    - ab+c -> <mark>abc</mark>, <mark>abbbc</mark>, ac
  - `.+` : 최소한 한 글자는 무조건 있어야 함 
- `?`
  - 의미 : 앞의 문자가 0개 혹은 1개이면 매치
  - 예시
    - ab?c -> <mark>ac</mark>, <mark>abc</mark>, abbc
  - `.?` : 딱 한 글자가 오거나, 아예 없어야 함
- `수량자?`
  - 의미 : 게으른 수량자, 조건이 맞는 최소한의 구간만 찾고 멈춤 (앞의 수량자가 대상임)
  - 예시 `"사과", "배", "포도"`
    - ".*" -> <mark>"사과", "배", "포도"</mark>
    - ".*?" -> <mark>"사과"</mark>
- `{n}`
  - 의미 : 앞에 나온 문자가 정확히 n개이면 매치
  - 예시
    - a{3} -> <mark>aaa</mark>, aa, aaaa
- `{n,}`
  - 의미 : 앞에 나온 문자가 n개 이상이면 매치
  - 예시
    - a{2,} -> <mark>aa</mark>, <mark>aaa</mark>, <mark>aaaa</mark>, a
- {n1,n2}
  - 의미 : 앞에 나온 문자가 n1개 이상, n2개 이하면 매치
  - 주의 : 쉼표 뒤에 공백을 넣으면 안
  - 예시
    - a{2,3} -> <mark>aa</mark>, <mark>aaa</mark>, aaaa, a
      
- **정규 표현식 수량자의 성격**
  - 기본 : **탐욕적(greedy) 수량자**
    - 가능한 많은 문자를 매치
  - **게으른(lazy) 수량자**
    - 기존의 수량자에 ?를 붙인 형태
    - 가능한 적은 문자를 매

### 그룹화
- `()`
  - 의미 : ()로 감싼 부분을 그룹화하여 하나의 문자로 여김
  - 예시
    - dog+ -> <mark>dog</mark>, <mark>doggggg</mark>, dogdog
    - (dog)+ -> <mark>dog</mark>, <mark>dogdog</mark>

## 정규 표현식 플래그
- 검색의 옵션을 지정하는 역할
- 형식 : `/패턴/플래그`
### 플래그
- `g`
  - 의미 : global search (전역 검색, 매치하는 모든 문자/문자열을 검색)
  - 예시 `apple banana apple orange`
    - /apple/ -> <mark>apple</mark> banana apple orange
    - /apple/g -> <mark>apple</mark> banana <mark>apple</mark> orange
- `i`
  - 의미 : ignore case (대소문자 구분 없음)
  - 예시 `Apple is apple`
    - /apple/ -> Apple is <mark>apple</mark>
    - /apple/gi -> <mark>Apple</mark> is <mark>apple</mark>
- `m`
  - 의미 : multiline (여러 줄에서 검색)
  - 예시
    ```
    start line 1
    start line 2
    start line 3
    ```
    - /^start/g
      ```
      <mark>start</mark> line 1
      start line 2
      start line 3
      ```
    - /^start/gm
      ```
      <mark>start</mark> line 1
      <mark>start</mark> line 2
      <mark>start</mark> line 3
      ```
- `s`
  - 의미 : single lind(dotall) (메타문자 `.`가 개행문자도 포함)
  - 예시
    ```
    Start
    End
    ```
    - /Start.*End/ -> 매칭 실패 (Start 뒤에 줄바꿈이 있는데, .은 줄바꿈을 못 넘어가서 끊김)
    - /Start.*End/s -> 매칭 성공 (s 옵션 덕분에 .이 줄바꿈까지 포함해서 연결함) 
