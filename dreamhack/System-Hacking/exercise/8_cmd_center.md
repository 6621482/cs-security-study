## Dreamhack - cmd center

### 문제 유형 
- Stack BOF
- Command Injection

### 문제 요약
사용자로부터 입력을 받는 center_name 변수(24바이트)에 대한 입력 길이 제한이 100바이트로 설정되어 있어, 할당된 크기를 초과하여 입력할 수 있는 취약점이 존재함  
프로그램은 `strncmp`를 통해 cmd_ip의 앞 8글자가 "ifconfig"인지 확인한 후, 맞다면 `system(cmd_ip)`를 실행함  
목표 : BOF를 통해 strncmp 조건을 만족시킬 수 있게 cmd_ip 변수를 덮어써서 쉘을 실행시키는 것  

### 코드 분석 포인트 
- 취약점: `read(0, center_name, 100);`
  - center_name은 24바이트지만 100바이트를 입력받기 때문에 스택의 다른 영역을 침범할 수 있음
- 공격 대상 : `char cmd_ip[256] = "ifconfig"`
  - `system()` 함수에 인자로 전달되는 변수이므로, 이 값을 조작하면 원하는 명령어를 실행할 수 있음
- 제약 조건 : `if( !strncmp(cmd_ip, "ifconfig", 8))`
  - cmd_ip를 덮어쓸 때, 반드시 앞부분은 "ifconfig"로 시작해야 함

### 접근 과정
1. GDB를 통해 메모리 구조 파악
   - `disass main`을 통해 center_name의 위치와 cmd_ip의 위치를 파악함
2. 오프셋 계산
   - center_name과 cmd_ip 사이의 거리 : 0x20
   - 32바이트의 더미를 채우면 cmd_ip에 도달함
3. 페이로드 구성
   - padding : b"A" * 32
   - 제약 조건을 우회하기 위해 "ifconfig" 문자열 포함
   - shell injection : `;`을 사용하여 앞의 명령어를 끝내고 쉘을 실행하도록 구성

### 어려웠던 점
- 오프셋 계산 시 실수가 있어 쉘을 획득하지 못했음
  - GDB를 활용하여 정확하게 계산해야 함

### 배운 점
- 명령어 연속 실행
  - 리눅스에서 세미콜론을 사용하면 여러 명령어를 한 줄에 연속으로 실행할 수 있음
- 정확한 오프셋 계산
  - rbp를 기준으로 한 상대 주소 차이를 계산하여 정확한 패딩 길이를 구하는 법을 익힘

### 쓰인 개념
- BOF
- Command Injection
- disassembly
- pwntools

### 추가 내용
***Q. strcmp와 strncmp 차이?**
- `strcmp`(String Compare) : 두 문자열이 완전이 동일한지 비교
  - 두 문자열의 첫 글자부터 널문자가 나올 때까지 한 글자씩 비교함
  - 두 문자열의 끝까지 비교하므로, 두 문자열의 길이가 다르거나 내용이 하나라도 다르면 다른 것으로 판단함
  - 취약점 : 비교할 문자열이 널로 제대로 끝나지 않은 경우 Buffer Over-read 취약점이 발생할 수 있음
- `strncmp`(String N Compare) : 두 문자열을 지정한 길이(n)까지만 비교함
  - n만큼만 비교하고 멈춤
  - 앞부분만 일치하면 같다고 판단함
  - 예시 (비밀번호 : "apple123")
    - `strncmp(input, "apple123", 5)`라고 작성하면, 사용자가 "apple"만 입력해도 통과되는 논리적 오류 발생
  - 보안상의 이점 : 비교할 최대 길이를 제한하므로, strcmp에서 발생할 수 있는 무분별한 메모리 접근을 막을 수 있음
    - 검사하는 길이를 잘못 설정하면 우회 가능성이 있음에 주의해야 함 
