## Dreamhack - Stack Canary Bypass & Shellcode

### 문제 유형
- 시스템 해킹
- 스택 버퍼 오버플로우
- stack canary 우회
- ASLR 우회
- shellcode injection

### 문제 요약
64비트 환경 시스템으로, 스택에 buf를 할당하고, 시작 시 buf의 주소를 printf로 노출해줌  
두 번의 입력 기회가 주어짐  
- `read(0, buf, ...)` : canary leak을 위한 입력  
- `gets(buf)` : 실제 오버플로우를 일으켜 리턴주소를 덮는 입력  
목표 : 랜덤하게 생성되는 카나리 값을 알아내어 보호 기법을 우회하고, 쉘코드를 실행하여 플래그 획득  

### 코드 분석 포인트 
- 스택 프레임 구조
  - disass main을 통해 몇 바이트 할당했는지 확인하고, 구조를 파악함
  - `buf(80) → padding(8) → Canary(8) → SFP(8) → RET(8)`
- canary leak
  - 카나리의 첫 바이트는 항상 `\x00`(NULL)
  - printf는 NULL을 만날 때까지 출력함
  - buf부터 카나리의 첫 바이트까지 꽉 채워 NULL을 덮어쓰면, printf가 카나리값까지 이어서 출력함
- ASLR 및 주소
  - 스택 주소는 매번 바뀌지만, 프로그램이 buf 주소를 알려주므로 이를 받아 ret에 덮어씌울 때 활용 가능

### 접근 과정
1. buf 주소 파싱
  - `recvuntil("buf: ")`로 대기 후, `recvline()`으로 주소 문자열 획득
  - 16진수 문자열이므로 int(addr, 16)을 사용하여 정수로 변환
2. canary leak
  - 카나리 첫 번째 주소까지 b"A"를 채워 널문자를 덮어씀
  - `recvuntil`로 내가 보낸 더미 값을 받고, 뒤따라오는 7바이트를 recv(7)로 수신(=카나리값)
  - `u64(b"\x00" + leaked)`를 사용하여 원래 카나리 값(8바이트 정수)으로 복구
3. payload 구성
  - `payload = [Shellcode] + [Padding] + [Canary] + [SFP] + [RET]`
  - shellcode : shellcraft를 사용함 
  - Padding 길이: 88 - len(shellcode)
  - RET: 아까 구해둔 buf_addr (쉘코드가 위치한 곳)로 덮어씌움
4. 전송 및 쉘 획득
  - `gets()`함수는 개행을 만나야 입력을 종료하므로 sendline 사용

### 어려웠던 점
- GDB 값을 본 카나리 값을 payload에서 사용하였음 (카나리는 실행마다 바뀌므로 반드시 leak 해야 함을 깨달음)
- 오프셋 계산 실수 (디스 어셈블을 통해 패딩 8바이트가 있다는 사실을 확인하였음)
- 데이터 파싱 방법을 제대로 몰랐음
  - buf 주소(문자열)은 int()를 사용해야 하고, 메모리에서 가져온 카나리(바이트)는 `u64()`를 써야 하는데 카나리에 int()를 사용함
  - 카나리를 가져올 때, 마지막 8바이트(첫 바이트는 널)을 얻기 위해 복잡한 연산을 시도함
    - AND 연산을 통해 가져오려고 했으나, 정확한 값을 가져오지 못했음
- 아키텍처 설정을 하지 않아 32비트 쉘코드가 생성되어 공격이 실패함

### 배운 점
- 카나리 우회 원리
  - 카나리의 NULL 바이트를 덮어쓰면 printf가 카나리를 유출한다는 원리를 이해함
- 서버가 주는 데이터가 문자열인지 raw byte인지 구별하고, 상황에 맞게 `int(x, 16)`과 `u64(x)`를 올바르게 사용해야 함
- GDB를 통해 실제 스택 프레임을 확인해야 정확한 오프셋을 알 수 있음

### 쓰인 개념
- canary
- BOF
- ASLR
- pwntools
  - u64, p64, shellcraft, asm, remote 등
 
### 추가 내용
**데이터 파싱: int() vs u64() 구분하기**  
기준 : 서버가 데이터를 어떤 형태로 보내주는가?  
- int() 함수를 사용해야 할 때
  - 서버가 데이터를 **문자열** 형태로 보내줄 때
  - 서버 동작
    - `printf("%p", ...)`
    - `printf("%x", ...)`
  - 예시
    ```c
    printf("Buf Addr: %p\n", buf); // 출력: 0x7fffffffe120
    printf("Count: %d\n", cnt);    // 출력: 100
    ```
    - 화면에 0x... 또는 숫자가 눈으로 명확히 보임 
    - 데이터타입이 bytes이지만, 내용은 아스키문자들로 구성되어있음
  - 변환 : 16진수 문자열을 정수로 변환 
    ```python
    # 수신 데이터: b"0x7fffffffe120\n"
    addr_str = r.recvline().strip() 
    addr_int = int(addr_str, 16)  # 16진수 문자열 -> 정수
    ```
- u64() 함수를 사용해야 할 때
  - 서버의 메모리 값을 **있는 그대로(Raw Bytes)** 가져왔을 때
  - 서버 동작
    - `read()`, `write()`
    - `printf("%s", ...)` -> memory leak 발생한 경
  - 예시
    ```c
    write(1, buf, 8);               // 메모리 8바이트를 그대로 전송
    printf("Input: %s\n", buf);     // %s는 NULL 전까지의 바이트를 그대로 출력
    ```
    - 화면에 출력된 값이 \x00\x1a\xf4... 처럼 깨진 글자나 이상한 특수문자로 보임
    - 메모리에 저장된 리틀 엔디안 형태의 바이트 배열임
  - 변환 : 바이트를 하나의 64비트 정수로 언패킹함
    ```python
    # 수신 데이터: b"\x20\xe1\xff\xff\xff\x7f\x00\x00" (사람 눈엔 이상해 보임)
    raw_bytes = r.recv(8)
    addr_int = u64(raw_bytes)     # 바이트 배열 -> 정수
    ```
