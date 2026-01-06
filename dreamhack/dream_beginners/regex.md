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
    - [d-f] -> H99llo
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
