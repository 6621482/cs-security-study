## Dreamhack - Type Error

### 문제 유형
- 시스템 해킹
- Integer Underflow
- Logic Bug

### 문제 요약
사용자로부터 버퍼의 크기를 입력받고, 그 크기만큼 데이터를 입력받는 프로그램  
SIGSEGV(메모리 참조 오류) 시그널이 발생하면 쉘을 실행 시켜주는 핸들러가 등록되어 있음  
목표 : 의도적으로 프로그램을 고장내는 것  

### 코드 분석 포인트
- 시그널 핸들러 등록
  - `signal(SIGSEGV, get_shell);`
  - segment falut가 발생하면 쉘을 획득할 수 있음
- 취약점
  ```C
  // 검증
  if (size > 256 || size < 0) { ... exit(0); }
  // read 
  read(0, buf, size - 1);
  ```
  - size에 0을 입력하면 검증은 통과하지만, read 함수 내부에서는 -1이 됨
  - read의 길이 인자는 `size_t`(unsigned int)이므로 -1은 매우 큰 양수로 해석됨 (Interger Underflow)

### 접근 과정
1. size에 0을 넣으면 read 함수의 마지막 인자에 interger underflow가 발생함을 파악
2. scanf로 정수를 받으므로 문자열 "0"을 입력하여 검증 로직 통과
3. 페이로드 구성
   - 버퍼크기보다 훨씬 많은 데이터를 보내 리턴 주소를 침범하여 SIFSEGV를 유발함
4. 쉘 획득
   - cat flag를 통해 flag값 확인

### 어려웠던 점
- scanf의 입력 형식을 몰랐음
  - 처음에 p32(0)으로 바이너리 데이터를 보냈는데 에러가 발생함
  - p32(0)대신 문자열 b"0"을 보내 해결함
- 프로그램이 죽지 않고 정상종료됨
  - size 검증은 통과했지만 페이로드 길이가 애매하게 짧아서 중요 메모리를 건드리지 못하고 정상 종료됨
  - 페이로드 길이를 늘려 확실하게 메모리 오류를 유발함

### 배운 점 
- `scanf()`와 `read()`의 인자 차이
  - `scanf("%d")`는 문자열로 된 숫자를 인자로 받음
  - `read()`는 바이트 그 자체를 읽음
  - 입력 함수에 따라 데이터를 보내는 방식을 다르게 해야 함
- int와 size_t 사이의 형변환 과정에서 발생하는 underflow가 취약점이 될 수 있음

### 쓰인 개념
- Integer Underflow
- Buffoer Overflow
- signal
- Type error

### 추가 내용 
**Q. `signal()`이 무슨 함수??**
- `signal(int signum, void (*handler)(int))` : 특정 시그널이 발생했을 때 실행할 함수(핸들러)를 등록하는 함수
  - `signum` : 처리할 시그널 번호
    - SIGSEGV : 잘못된 메모리 접근 (세그멘테이션 폴트)
    - SIGALRM : alarm()에 의해 시간 초과
    - SIGINT : Ctrl+C
  - 일반적인 프로그램을 SIGSEGV 발생 시 OS에 의해 강제 종료됨 (기본 동작)
  - 이 문제의 경우, `signal(SIGSEGV, get_shell);`를 통해 강제 종료가 아닌 get_shell 함수가 실행되도록 함

**Q. `scanf`가 입력으로 받는 것은 무엇인가?**
- scanf("%d", ...) : 입력 스트림에서 아스키 문자열 형태의 숫자를 찾아 정수로 변환함
- p32(0)은 바이너리 값 0x00을 보냄
  - scanf는 이를 유효한 숫자 문자열로 인식하지 못해 매칭이 실패함
