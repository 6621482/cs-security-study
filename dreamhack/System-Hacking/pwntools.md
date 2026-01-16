# pwntools 
- Pwn(폰) : 특정 시스템이나 프로그램의 취약점을 이용해 제어권을 빼앗는 공격 행위
- pwntools : 공격 스크립트를 신속하게 작성할 수 있도록 도와주는 프레임워크
  - 파이썬 패키지 형태로 제공됨

## pwntools 프로세스 통신 기능
- 프로세스 간 통신 과정
  - 통신하려는 프로세스 사이에 연결을 맺고, 데이터를 주고받으며 통신하고, 연결을 종료하는 과정을 거침

### 연결을 맺는 함수
함수들이 성공적으로 실행되면 `pwnlib.tubes` 클래스를 반환함

- `process(<argv>, [**kwargs])` : 로컬에 위치한 프로그램을 실행하여 통신할 때 사용하는 함수
  - 인자
    - argv : 실행할 프로그램의 경로 또는 argv 리스트  
    - kwargs : 선택 인자 (env, cwd 등)  
  - 전달된 프로그램을 실행한 뒤 해당 프로그램과 연결을 맺어줌
  - 예시
    ```pwntools
    from pwn import *
    p = process("./test")
    ```
    ```pwntools
    from pwn import *
    p = process(["./test", "AAAA"], env={"LD_PRELOAD":"./libc.so.6"})
    ```
    - 연결 성공 : p에 pwnlib.tubes 클래스가 저장됨  

- `remote(<host>, <port>, [**kwargs])` : 네트워크로 연결된 원격 프로그램과 통신하기 위한 함수
  - 인자
    - host : 서버 주소(IP 또는 도메인)
    - port : 포트 번호(int)
    - kwargs : 선택 인자 (timeout, ssl = True, typ = 'tcp' 등)
  - 기본적으로 TCP 연결을 맺음 
  - 예제
    ```pwntools
    from pwn import *
    r = remote("test.com", 1337)
    ```

- `ssh(user=None, host=None, port=22, password=None, key=None, keyfile=None, timeout=None, **kwargs)` : SSH 서버에 접속하여 통신하기 위한 함수 
  - 인자
    - user : SSH 접속할 계정 이름
    - host : 접속 대상 IP 또는 도메인
    - 인증 관련 : password, key, keyfile (password 와 key/keyfile 중 하나만 있어도 됨) 
    - 연결 관련 : port, timeout
  - 예시
    ```
    from pwn import *
    s = ssh(user="user", host="10.10.10.10", port = 2222, password ="guest")
    ```

### 데이터 송수신 함수
- `recv([numb][timeout])` : 원격 프로세스, 소켓에서 받을 수 있는 만큼의 바이트를 읽는 함수
  - 대상 : process, remote 등으로 생성한 pwnlib.tubes 계열
  - 인자
    - numb : 최대 수신 바이트 수
    - timeout : 대기 시간
  - numb만큼 받지 못해도 당장 받을 수 있는 만큼 수신한 뒤 함수를 종료함 
  - 반환형 : bytes
  - 그 외 recv 계열 함수들
    - `recvline([keepends], [timeout])` : \n이 나올 때까지 수신함
    - `recvn(<numb>, [timeout])` : numb 바이트를 받을 때까지 블로킹
      - 정확히 numb 바이트를 받지 못하면 계속 기다림
    - `recvuntil(<delim>, [drop], [timeout])` : delim이 나올 때까지 계속 수신함 (delim도 포함)
      - ex. `data = p.recvuntil(b"hi")` : p가 b"hi"를 출력할 때까지 데이터를 수신하여 data에 저장
        - b"" : 바이털 리터럴 표기 
    - `recvall([timeout])` : EOF까지 전부 수신 

- `send(<data>)` : 데이터를 전송하기 위해 사용하는 함수
  - 대상 : process, remote 등으로 생성한 pwnlib.tubes 계열
  - 인자 : bytes 클래스
  - 개행(\n)이 자동으로 붙지 않음 
  - 반환값 없음
  - 그 외 send 계열 함수들
    - `sendline(<data>)` : \<data\> 뒤에 b"\n"을 자동으로 붙여 전송함
    - `sendafter(<delim>, <data>, [timeout])` : 먼저 \<delim\>이 나올 때까지 recvuntil로 기다린 뒤, \<data\>를 전송함
      - 내부적으로 `recvuntil(delim)`, `send(data)`를 실행 
    - `sendlineafter(<delim>, <data>, [timeout])` : 먼저 \<delim\>이 나올 때까지 기다린 뒤, \<data\> + b"\n"을 전송함
  - 예시
    ```python
    from pwn import*
    p = process("./test")

    p.send(b'a')   # ./test에 b'a'를 입력
    p.sendline(b'a)   # ./test에 b'a' + b'\n'을 입력
    p.sendafter(b'hello', b'a')   # ./test가 b'hello'를 출력하면, b'a'를 입력
    p.sendlineafter(b'hello', b'a')    # ./test가 b'hello'를 출력하면, b'a' + b'\n'을 입력
    ```
- `interactive([prompt])` : 현재 연결된 프로세스, 소켓과 사용자 터미널을 직접 연결하여, 이후 입출력을 사람이 직접 타이핑하여 상호작용하게 만듦
  - 키보드 입력 -> send
  - 프로그램 출력 -> recv -> 화면 출력
  - interactive() 이후에는 send(), recv() 사용하지 않음
  - 종료 : `ctrl + D` 입력

### 연결을 종료하는 함수
- 연결된 두 프로세스 중 어느 한 프로세스가 연결을 종료하면 해당 연결은 종료됨 
- `close()` : 연결을 종료하는 함수
  ```python
  from pwn import*
  p = proces("./test")
  p.close   # 실행되는 순간에 연결이 유지되고 있는 상태여야 함 
  ```

### 로그 출력
- `context.log_level' : 로그 출력의 상세도를 설정할수 있음
  ```python
  from pwn import *
  context.log_level = 'debug'

  p = process("./example")
  p.recvall()
  ```
  - 송수신 데이터를 모두 보고싶은 경우  `context.log_level = 'debug'`로 설정
  - 로그 상세도
    - `critical` : 치명적인 오류 외에 출력하지 않음
    - `error` : 에러 메세지만 출력
    - `warning`/`warn` : 경고 메세지 출력
    - `info` : 일반적인 상태 메세지 출력 (기본 값) 
    - `debug` : 송수신 데이터, 내부 변수 등 상세히 출력

## pwntools 편리한 기능들
- **Packing & Unpacking** : 데이터를 하나의 형태로 모아 포장하거나, 다시 분해하는 것
  - `p8()`, `p16()`, `p32()`, `p64()` : 숫자를 bytes 클래스로 패킹하는 함수
  - `u8()`. `u16()`, `u32()`, `u64()` : bytes 클래스를 숫자로 언패킹하는 함수
  - 예시
    ```python
    from pwn import *

    s8 = 0x41
    s16 = 0x4142
    s32 = 0x41424344
    s64 = 0x4142434445464748

    print(p8(s8))
    print(p16(s16))
    print(p32(s32))
    print(p64(s64))

    s8 = b"A"
    s16 = b"AB"
    s32 = b"ABCD"
    s64 = b"ABCDEFGH"

    print(hex(u8(s8)))
    print(hex(u16(s16)))
    print(hex(u32(s32)))
    print(hex(u64(s64)))
    ```
   
    ``` python
    b'A'
    b'BA'
    b'DCBA'
    b'HGFEDCBA'
    0x41
    0x4241
    0x44434241
    0x4847464544434241
    ```
  - 패킹 및 언패킹 함수는 기본적으로 리틀 엔디안으로 동작함
### GDB
- `gdb.attach(p)` : 해당 프로세스에 GDB가 붙어 실행이 준단도고, GDB 터미널이 열려 디버깅을 수행할 수 있는 상태가 됨
  - 인자 p : process()로 생성한 객체, remote()로 생성한 객체, 프로세스의 PID
  - 예시
    ```
    from pwn import *
    p = process("./test")
    gdb.attach(p)
    sleep(1)   # attach 이후 GDB가 성공적으로 붙을 수 있는 시간을 벌어줌
    ```
- Assemble & Disassemble
  - `context.arch` : 아키텍처 설정
    - ex. `context.arch` = "amd64"`   # x86-64 아키텍처
  - `asm()` : 어셈블, 어셈블리 언어를 기계어로 변환
  - `disasm()` : 디스어셈블, 기계어를 어셈블리 언어로 변환
  - 예시
    ```
    from pwn import *

    context.arch = "amd64" # x86-64 아키텍처로 설정

    machine_code = asm('mov eax, 0')  # 어셈블리 'mov eax, 0'를 기계어로 변환
    print(machine_code)
    assembly_code = disasm(machine_code)  # 기계어를 어셈블리어로 변환
    print(assembly_code)
    ```
    ```
    b'\xb8\x00\x00\x00\x00'
    0:   b8 00 00 00 00          mov    eax, 0x0
    ```
### ELF
- **ELF**(Executable and Linkable Format) : 리눅스 운영체제의 실행 파일 형식
- `ELF(<path>, [checksec])` : ELF 실행 파일을 파싱해서 심볼, 주소, 섹션, GOT/PLT 정보를 코드에서 바로 쓰게 해주는 객체
  - 인자
    - path : 분석할 ELF 실행 파일 경로
    - checksec : 보안 옵션 자동 출력 여부 
  - 반환형 : `pwnlib.elf.elf.ELF`
  - 예시
    ```
    from pwn import *

    e = ELF("./example")  # ELF 파일 열기
    print(hex(e.symbols['write']))  # 'write' 심볼의 주소
    print(hex(e.symbols.write)) # 'write' 심볼의 주소, 속성 접근
    ```
  - `pwnlib.elf.elf.ELF`클래스의 멤버 변수 
    - `symbols` : ELF 파일에 정의된 함수, 전역 변수 이름과 그 주소를 매핑해둔 테이블
      - doctdict 클래스 : pwntools 내부에 구현된 클래스, 기존의 딕셔너리처럼 사용 (속성으로 접근 가능)
        - 속성으로 접근 가능하다는 것은 `e.symbols["write"] == e.symbols.write`을 의미 
      - symbols에 들어있는 주소 = 가상 주소 
