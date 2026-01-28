# ReLocation Read-Only (RELFRO)
- Lazy Binding : 실행 효율을 위해 함수가 실제로 호출될 때 주소를 찾아 GOT에 적어주는 방식
  - 문제점 : 실행 중에 GOT에 주소를 써야 하기 때문에 GOT 영역에 쓰기 권한이 있음 -> GOT Overwrite가 가능해짐
- RELRO : 권한 최소화 전략
  - 데이터 세그먼트 중 실행 중에 더 이상 수정될 필요가 없는 영역(GOT, .init_array, .fini_array 등)을 **Read-Only**로 바꿔버리는 보안 메커니즘
  - 적용 범위에 따른 분류
    - Partial RELRO
    - Full RELRO
      
## Partial RELRO
- `.init_array`, `.fini_array` 같은 특정 섹션만 읽기 전용으로 만듦
  - `.init_array` & `.fini_array` : 프로그램 시작과 종료 시 실행되는 함수의 주소를 담고 있음 
- 바인딩 방식 : Lazy Binding (필요할 때 주소 찾음)
  - 함수 주소가 담기는 GOT 영역은 쓰기 권한이 남아있음
- GOT 권한 : rw-
- 보안 : GOT Overwrite 공격에 취약할 수 있음

### .got / .got.plt - Partial RELRO
Partial RELRO 환경에서는 효율성과 보안을 절충하기 위해 GOT 영역을 두 개로 분리함 
- `.got` (읽기 전용)
  - 바이너리가 실행되는 시점에 이미 주소가 확정되는 변수나 데이터들을 담음 (전역변수 등) 
  - 바인딩 방식 : Now Binding (실행 직후 주소가 바로 채워짐)
  - 권한 : 실행 중에는 값이 변할 일이 없으므로 쓰기 권한이 부여되지 않음
  - 보안 : 변조 불가 (안전) 
- `.got.plt` (읽기/쓰기 가능)
  - 실행 중에 라이브러리 함수가 호출될 때 주소를 찾는 변수들을 담음 (printf, system 등)
  - 바인딩 방식: Lazy Binding (함수가 처음 호출될 때 주소를 계산해서 이곳에 적음)
  - 실행 중에 함수 주소를 써넣어야(Write) 하므로 쓰기 권한이 부여됨
  - 보안 : GOT Overwrite 공격 가능

## Full RELRO
- 바인딩 방식 : Immediate Binding (시작할 때 주소 찾음)
  - 프로그램이 실행될 때 라이브러리 함수의 주소를 미리 다 찾아서 GOT에 적음
- GOT 권한 : r--
- 보안 : 실행 중에 GOT를 수정할 수 없으므로 GOT Overwrite 공격을 근본적으로 차단함

### .got.plt - Full RELRO
- 실행 중에 바인딩(lazy binding)되는 변수의 경우 해당 변수에 쓰기 권한이 필요함
- Full Relro를 적용하면 쓰기 권한을 제거해야 하므로 lazy binding 대신 바이너리가 실행되는 시점에 바인딩 되는 방식(now binding)만 사용할 수 있음
- 기존 `.got.plt`에 저장되어 있던 lazy binding 변수는 now binding 변수로 바뀌게 되어 `.got` 영역에만 저장됨 (이 GOT에는 쓰기 권한이 부여되지 않음) 

## RELRO 우회
- Partial RELRO 우회 : GOT Overwrite
  - 공격 지점: .got.plt 영역
  - Lazy Binding 방식을 사용하므로, 실행 중에 라이브러리 함수 주소를 써넣어야 하는 `.got.plt` 섹션에 쓰기 권한이 부여되어 있음
  - 공격자는 이 영역에 저장된 특정 함수(예: printf, scanf)의 주소를 악성 코드나 system 함수의 주소로 덮어쓰는 GOT Overwrite 공격을 수행할 수 있음
- Full RELRO 우회: Hook Overwrite
  - Full RELRO가 적용되면 바이너리 로딩 시점에 모든 함수 주소를 미리 바인딩하고, GOT 전체를 읽기 전용으로 바꿔 GOT Overwrite가 불가능함
  - 공격 지점: 라이브러리(libc) 내부에 존재하는 Hook(훅) 변수
    - 대표적인 Hook: `__malloc_hook`, `__free_hook` 등
    - 이 Hook들은 본래 디버깅을 위해 만들어진 함수 포인터
    - malloc 함수가 실행될 때, 코드 시작 부분에서 `__malloc_hook`이 존재하는지 확인하고 있다면 이를 먼저 호출함
    - 이 Hook 변수들이 `libc.so` 내의 쓰기 가능한 영역에 위치함
  - 공격자가 라이브러리가 로드된 주소(Libc Base)를 알아낼 수 있다면, 이 Hook 변수를 조작하여 malloc이나 free가 호출될 때 원하는 실행 흐름을 가로채는 Hook Overwrite 공격을 수행할 수 있음 
