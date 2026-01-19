# Stack Canary

## Stack canary
- **Stack canary** : 함수의 프롤로그에서 스택 버퍼와 반환 주소 사이에 임의의 값을 삽입하고, 함수의 에필로그에서 해당 값의 변조를 확인하는 보호 기법 
- 작동 원리
  - 설정 : 프로그램이 함수를 호출할 때, 반환 주소 바로 앞에 **카나리**를 심어둠
  - 검사 : 해당 함수가 종료되고 리턴하기 직전에, 카나리 값이 처음 심어둔 값과 동일한지 확인
  - 탐지 : 만약 버퍼 오버플로우 공격이 발생하였다면, 반환 주소를 덮어쓰기 전에 그 앞에 있는 카나리 값이 먼저 변조됨
  - 대응 : 카나리 값이 변한 것을 감지하면 해킹 시도가 있다고 판단하여 프로그램을 즉시 강제 종료시킴
- 목적
  - 실행 흐름 조작 방지 : return address 를 조작하여 악성 코드가 있는 주소로 점프하는 것을 방지함
- 한계
  - 우회 가능성이 존재함 : 해커가 카나리 값을 알아내 똑같이 덮어쓰면 공격을 탐지할 수 없음

### 카나리 생성 과정 
- 프로세스가 시작될 때, TLS에 전역 변수로 저장되고, 각 함수마다 프롤로그와 에필로그에서 이 값을 참조함
  - TLS(Thread Local Storage) : 각 스레드마다 따로 갖는 전용 데이터 영역 
    - fs는 세그먼트 레지스터로, TLS의 base 주소를 저장함
    - `fs:0x28` : fs가 가리키는 TLS 시작주소로부터 0x28 바이트 떨어진 위치에 스레드 전용 스택 카나리 값이 있음
#### TLS 주소 파악 - GDB 활용 
- `fs`의 값은 특정 시스템 콜을 사용해야 조회 가능 (`info register fs`로 알 수 없음)
- `arch_prctl(int code, unsigned long addr)` : fs 값 설정 시 호출되는 시스템 콜
  - arch_prctl 함수가 호출되는 시점에 중단점 걸어서 확인 
  - `ARCH_SET_FS` : FS base(=TLS base)를 addr로 설정
  - `ARCH_GET_FS` : 현재 FS base를 받아옴
  - `arch_prctl(ARCH_SET_FS, addr)`를 호출하면 FS base = addr로 설정됨
- TLS 주소 찾는 과정 
  - (1) `catch syscall arch_prctl`
    ```
    catch syscall arch_prctl
    run
    ``
    - arch_prctl 시스템콜이 호출되는 순간에 멈추게 함 (FS base(TLS base)는 arch_prctl로 설정되기 때문)
  - (2) `init_tls()`에서 멈춤
    ```
    Catchpoint 1 (call to syscall arch_prctl), init_tls
    ```
    init_tls()는 glibc가 스레드 시작 시 TLS를 초기화하는 내부 함수
  - (3) 레지스터 값
    ```
    RDI = 0x1002
    RSI = 0x7ffff7d7f740
    ```
    - rdi = 첫 번째 인자 = code
    - rsi = 두 번째 인자 = addr
    - `rdi = 0x1002` : 0x1002는 ARCH_SET_FS 상수값
    - `rsi = 0x7ffff7d7f740` : TLS 베이스 주소 = 0x7ffff7d7f740 -> 카나리값 주소 = 0x7ffff7d7f740 + 0x28

#### 카나리 값 설정
- (1) `watch *(TLS + 0x28)`
  - watch : 해당 주소의 값이 변경되면 프로세스를 즉시 중단
- (2) continue 후 멈춘 위치 : `security_init`
  ```
  Old value = 0
  New value = 2005351680
  security_init () at rtld.c:870
  ```
  - 스택 카나리는 security_init()에서 처음 설정됨
  - 카나리의 첫 바이트는 `0x00`임 

  ### 카나리 우회
  - **무차별 대입(Brute Force)** : 모든 후보를 순서대로 전부 시도하는 방식
    - x64 아키텍처 : 8바이트 카나리 (첫 널 바이트를 제외하면 실제로는 7바이트) -> 256^7번 시도해야 함 
  - **TLS 접근** : 카나리는 TLS에 전역 변수로 저장되므로, 이 값을 읽거나 조작할 수 있으면 카나리를 우회할 수 있음 
  - **스택 카나리 릭** : 스택 카나리를 읽을 수 있는 취약점이 있다면, 이를 이용하여 카나리 검사를 우회할 수 있음 
