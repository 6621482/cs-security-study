## Dreamhack - Return to Library

### 문제 유형
- 시스템 해킹
- 스택 버퍼 오버플로우
- canary, NX 우회
- 공격 기법 : RTL, ROP

### 문제 요약
환경 : 64비트 리눅스 환경  
버퍼는 0x30크기지만, read 함수로 최대 0x100 바이트까지 입력을 받아 스택 버퍼 오버플로우가 발생함  
입력 기회: 두 번 (1차: Canary Leak용, 2차: Exploit용)  
목표 : 랜덤한 카니라 값을 유출하여 우회하고, NX비트로 인해 쉘코드를 쓸 수 없는 상황에서 system 함수를 호출해 쉘을 획득하는 것

### 코드 분석 포인트
- 컴파일 옵션 (`-no-pie`) 확인
  - 바이너리 코드 영역과 데이터 영역의 주소가 고정됨
  - ASLR이 켜져 있어도 바이너리 내부 주소는 변하지 않으므로 ROP 가젯과 전역 변수 주소를 얻을 수 있음
- 취약점 위치 파악
  - `char buf[0x30]`으로 48바이트가 할당되었지만 `read(0, buf, 0x100)`으로 256바이트까지 입력받음
  - 스택 버퍼 오버플로우 취약점이 존재함 (리턴주소 덮어쓰기 가능)
- 카나리 우회
  - 카나리의 첫 바이트는 항상 `\x00`임
  - buf부터 카나리 직전까지 쓰레기 값으로 채우고, 카나리의 첫 바이트를 다른 값으로 덮어쓰면 printf("%s", buf)가 카나리 값까지 이어서 출력함

### 접근 과정
1. gdb로 스택 프레임 구조 파악
   - `disass main`을 통해 스텍 프레임 구조를 파악함
     - 구조 : `buf(0x30) + padding(0x8) + canary(0x8) + SFP(0x8) + RET`
   - system@plt (함수 plt 주소), "/bin/sh" (문자열 주소)
2. 가젯 주소 파악
   - `pop rdi; ret`, `ret` 주소 확인
3. canary leak
   - 첫 번째 read 입력해서 버퍼(0x30) + 패딩(0x8) + 카나리 첫 바이트(1) = 총 0x39 바이트의 데이터를 보냄
   - recv로 받아온 데이터에서 카나리값 복구
4. exploit payload 구성
   - 구조: `더미(0x38)` + `카나리(0x8)` + `SFP(0x8)` + <mark>`[ret 주소]`</mark> + `[pop rdi; ret 주소]` + `["/bin/sh" 주소]` + `[system@plt 주소]`
   - 체인 구성 이유
     - x86-64 System V ABI에 따라 함수의 첫 번째 인자는 RDI 레지스터로 전달됨
     - system("/bin/sh") 호출을 위해서는 "/bin/sh" 주소를 RDI에 먼저 로드해야 함
     - 이를 위해 `pop rdi ; ret` 가젯을 사용하여 스택에서 "/bin/sh" 주소를 꺼내 RDI에 저장
     - 이후 `ret`를 통해 system@plt로 흐름을 이동시킴
     - system 함수 진입 시 스택은 16바이트 정렬되어 있어야 하므로 정렬 보정을 위해 의미 없는 `ret` 가젯을 system 호출 직전에 추가함
5. 전송 및 쉘 획득
    - recvline()을 이용하여 payload를 전송하고, 쉘을 획득함

### 어려웠던 점
- `p system`을 했을 때 나오는 libc 실제 주소와 plt 주소 중 무엇을 써야 하는지 헷갈림
- `p binsh` 입력 시 `unkown type`에러가 발생함
  - (char *)로 캐스팅하거나 search 명령어를 사용하면 됨
- ROP 체인이 논리적으로 맞는데 오류가 생김 
  - 처음 만든 페이로드로 공격했을 때 `Segmentation Falut`가 발생함
  - 원인 : `system`함수 내부의 `movaps` 명령어
    - `movaps` : 스택 주소가 16바이트 단위로 정렬되어 있어야 함 -> ROP 실행 과정에서 이 정렬이 깨짐
  - 해결 : 아무 기능이 없는 ret 가젯을 system 함수 호출 직전에 하나 추가해야 함 

### 배운 점
- 페이로드를 구성할 때, CPU 명령어의 제약 조건까지 고려하여 ret 가젯으로 스택 길이를 맞춰야 함
- 쓰레기 값과 ret의 차이
  - 패딩을 채우기 위해 `b"A" * 0x8`과 같은 값을 넣으면 잘못된 주소로 점프하여 크래시가 발생할 수 있음
  - 반면, `ret`가젯을 넣으면 실행 흐름을 유지하면서 스택만 이동시킬 수 있음
- `read + printf("%s")` 조합은 정보 유출 취약점이 될 수 있음

### 쓰인 개념
- ROP
- stack canary 구조 
- pop rdi ; ret 가젯
- System V AMD64 calling convention (함수 진입 시점에 rsp는 반드시 16바이트 정렬이어야 함)
- pwntools

### 추가 내용
**Q. ROP 가젯 찾는 명령어?**  
- 모든 ROP 가젯 출력
  ```
  ROPgadget --binary <파일 경로>
  ```
  - 예시 : `ROPgadget --binary ./rtl`
- 특정 문자열을 포함하는 가젯 검색
  ```
  ROPgadget --binary <파일_경로> --re "<검색문자열>"
  ```
  - 예시 : `ROPgadget --binary ./rtl --re "pop rdi"`
- 정확히 특정 가젯만 검색
  ```
  ROPgadget --binary <파일_경로> --only "<가젯>"
  ```
  - 예시 : `ROPgadget --binary rtl --only "ret"`
 
**Q. ROP `system()`으로 점프할 때 rsp가 16바이트 정렬이 아니면 segmentation falut가 나는 이유?**  
- `systme()` 내부의 `movaps` 명령어 때문
- `movaps`
  - 16바이트 정렬된 메모리 주소만 접근 가능
