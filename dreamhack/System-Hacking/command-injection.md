# Command Injection

## Command Injection
- **Injection** : 악의적인 데이터를 프로그램에 입력하여 이를 시스템 명령어, 코드, 데이터베이스 쿼리 등으로 실행되게 하는 기법
- **Command Injection** : 사용자의 입력을 시스템 명령어로 실행되게 하는 것
- 발생 : 명령어를 실행하는 함수에 사용자가 임의의 인자를 전달할 수 있을 때
- `system()` : 쉘을 호출해서 명령어를 전달
  - 내부 동작 : `system("명령어")` → `do_system` → `/bin/sh -c "명령어"` 실행
  - 위험성 : 인자로 들어가는 내용에 사용자가 입력을 더할 수 있으면, 쉘의 메타 문자를 이용해 악의적인 명령어를 함께 실행시킬 수 있음
    - 메타문자
      | 메타 문자 | 설명 | Example |
      |---------|------|---------|
      | `$` | 쉘 환경변수 | ```bash
      $ echo $PWD
      /home/theori
      ``` |
      | `&&` | 이전 명령어가 성공하면 다음 명령어 실행 | ```bash
      $ echo hello && echo theori
      hello
      theori
      ``` |
      | `;` | 명령어 구분자 (앞 명령 성공 여부와 무관) | ```bash
      $ echo hello ; echo theori
      hello
      theori
      ``` |
      | `\|` | 명령어 파이핑 (앞 출력 → 뒤 입력) | ```bash
      $ echo id | /bin/sh
      uid=1001(theori) gid=1001(theori) groups=1001(theori)
      ``` |
      | `*` | 와일드카드 (여러 문자열 매칭) | ```bash
      $ echo .*
      . ..
      ``` |
      | `` ` `` | 명령어 치환 (명령 결과를 문자열로 사용) | ```bash
      $ echo `echo hellotheori`
      hellotheori
      ``` |
      - <mark>&&</mark>, <mark>;</mark>, <mark>|</mark> : 여러 명령어를 연속으로 실행시킬 수 있음
      - `ping [user-input]` 실행 시 악의적인 사용자가 `a; /bin/sh`를 전달하면 `ping a; /bin/sh`가 명령어로 전달되어 ping을 실행하고 sh를 실행함
         

       
