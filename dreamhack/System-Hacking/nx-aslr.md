# NX & ASLR

## NX
- **No-eXecute (NX)** : 실행에 사용되는 메모리 영역과 쓰기에 사용되는 메모리 영역을 분리하는 보호 기법
- 목적 : 특정 메모리 영역에 쓰기와 실행 권한이 동시에 부여될 경우 발생하는 보안 취약점을 막기 위함
  - 코드 영역에 쓰기 권한 존재 : 공격자가 기존 코드를 수정하여 원하는 악성코드가 실행되도록 조작
  - 스택/데이터 영역에 실행 권한 존재 : 쉘코드를 해당 영역에 입력한 후 반환 주소를 조작
- 적용 조건 : CPU가 NX를 지원해야 하고, 컴파일러 옵션을 통해 바이너리에 적용 가능
- 확인 방법 
  - gdb의 `vmmap` 사용
    - NX 적용 : 코드 영역 외의 다른 영역에는 실행 권한이 없음
    - NX 미적용 : 스택 영역 등에 rwx권한이 모두 있어 쉘코드 실행 가능
  - `checksec` 사용
    - 예시
      ```
      $ checksec ./nx
      [*] '/home/dreamhack/nx'
      Arch:     amd64-64-little
      RELRO:    Partial RELRO
      Stack:    Canary found
      NX:       NX enabled
      PIE:      No PIE (0x400000)
      ```

## ASLR
- **Address Space Layout Randomization (ASLR)** : 바이너리가 실행될 때마다 스택, 힙, 공유 라이브러리 등을 임의의 주소에 할당하는 보호 기법
- 목적 : 공격자가 메모리 주소를 예측하지 못하게 막는 것
- 확인 방법
  - `cat /proc/sys/kernel/randomize_va_space`
    - No ASLR(0) : ASLR을 적용하지 않음
    - Conservative Randomization(1) : 스택, 라이브러리, vdso 등
    - Conservative Randomization + brk(2) : (1)의 영역과 brk로 할당한 영역 
- 특징
  - 하위 12비트 고정 : 주소의 맨 뒤 12바이트는 page 단위 주소 매핑 때문에 안 바뀜
  - offset 고정 : libc 안에서 함수끼리의 거리는 항상 같음
  - `실제 주소 = 베이스 주소(랜덤) + 오프셋(고정)`
