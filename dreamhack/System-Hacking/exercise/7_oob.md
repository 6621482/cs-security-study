## Dreamhack - Out of bound 

### 문제 유형
- 시스템 해킹
- out of bound 접근
- 인덱스 검증 부재를 이용한 메모리 참조

### 문제 요약
환경: Ubuntu 16.04, 32bit (i386), No PIE (주소 고정)  
전역 배열 command의 인덱스를 사용자 입력으로 그대로 사용하여 배열 범위를 벗어난 접근이 가능한 취약점이 존재함  
목표 : 이 취약점을 이용해 `command[idx]`가 전역 변수 name을 가리키도록 만들고, name에 저장한 `/bin/sh\x00` 문자열의 주소를 system()의 인자로 전달하여 쉘을 획득하는 것

### 코드 분석 포인트
- 전역 변수 위치 : name과 command는 전역 변수로 선언되어 BSS/Data 영역에 위치함
  - PIE가 꺼져있어 주소가 고정됨
- OOB 취약점 : `system(command[idx])`에서 idx가 음수이거나 10 이상이어도 그대로 참조함
  - command 배열 근처에 있는 name 변수의 값을 system 함수의 인자로 전달할 수 있음
- 배열 인덱싱 계산
  - command의 타입 : `char *`
  - 32비트 환경 : 포인터 크기 = 4바이트
  - `command[n]의 주소 : &command + (n * 4)`
- system 함수 인자에 대한 핵심 개념
  - `system()`의 인자는 문자열 자체가 아님
  - 문자열이 저장된 주소가 인자로 전달되어야 함

### 접근 과정
1. GDB를 통해 메모리 주소 확인
   - `p &name`으로 name 변수 주소 확인
   - `p &command`로 command 변수 주소 확인
   - 두 변수 사이의 거리 : 76바이트
2. 인덱스 계산
   - command는 4바이트 포인터 배열이므로, name에 접근하기 위한 인덱스는 76 / 4 = 19임
   - name에 "/bin/sh"를 입력하고 idx에 19를 넣으면 오류가 발생함
     - 이유 : `command[19]`는 name의 시작 부분에 있는 값을 읽어옴
     - name에 "/bin/sh"가 있다면, 첫 바이트인 "/bin"(0x6e69622f)을 읽어옴
     - system 함수는 0x6e69622f를 주소로 인식하고 참조하려다 segmentation fault가 발생함
3. payload 구성
   - system 함수에는 문자열의 주소가 전달되어야 함
   - name 변수 안에 실제 문자열과 그 문자열을 가리키는 주소를 함께 넣음
     - 입력 데이터(name) : "/bin/sh\x00" + name의 시작주소
   - 최종 인덱스 : 읽어야 할 주소 값 = name 안에 넣은 name 시작주소 (시작점에서 8바이트 뒤)
     - command와의 거리 : 76(기존거리) + 8바이트(문자열 길이) = 84바이트
     - 최종 인덱스 : 84 / 4 = 21
4. 익스플로잇 실행
   - name : b"/bin/sh\x00" + p32(name 주소)
   - idx : 21
   - `sendline()`을 통해 각각 전송
   - `command[21]`이 참조하는 곳에는 name주소가 들어있고, `system(name 주소)`가 실행되면서 쉘 획득 성공

### 어려웠던 점
- system 함수의 인자 전달 방식 오해
  - system("/bin/sh")과 같이 코드상에서 문자열을 그대로 써왔기 때문에 name에 문자열만 넣으면 될 줄 알았음
  - 어셈블리 레벨에서 함수는 값이 아닌 주소를 인자로 받는다는 점을 간과하여 문자열 값 자체가 주소로 해석되어 엉뚱한 메모리를 참조하여 에러가 발생했음
 
### 배운 점
- C 함수 인자는 값과 주소를 구분해야 함
- 전역 변수는 PIE가 꺼져 있으 경우 공격 대상이 됨
- 단순 OOB라도 메모리 배치에 따라 exploit이 가능함
- `"bin/sh"` 문자열 뒤에 반드시 `\x00`을 붙여야 시스템이 문자열의 끝을 인식하고 정상적으로 쉘을 실행함 

### 쓰인 개념
- Out of bound 개념
- x86 32비트 포인터 크기
- system(char *command) 호출 방식
- pwntools
  - p32, remote, interactive 

### 추가 내용
**Q. `system()` 함수가 인자로 받는 것은 무엇인가?**  
- `int system(const char *command);`
- 인자 : 널문자로 끝나는 C 문자열이 저장된 메모리의 주소 
