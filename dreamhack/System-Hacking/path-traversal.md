# Path Traversal

## Path Traversal
### 리눅스 경로
- **절대 경로(Absolute Path)**
  - 루트 디렉토리('/')부터 파일에 이를 때까지 거쳐야 하는 디렉토리 명을 모두 연결하여 구성
  - 각 디렉토리는 '/'로 구분되고, 끝에 대상 파일의 이름을 추가하여 완성함
  - 한 파일에 대응되는 절대 경로는 유일함 (그 파일만의 고유한 값)
  - 현재 사용자가 어떤 디렉토리에 있더라도, 절대 경로를 사용하면 대상 파일을 가리킬 수 있음
  - 예시
    ```text
    /
    ├── home
    │   ├── theori
    │   │   ├── documents
    │   │   │   └── report.txt
    │   │   └── downloads
    │   │       └── tool.zip
    │   └── guest
    │       └── notes.txt
    ├── bin
    │   └── ls
    └── etc
        └── passwd
    ```
    - `/home/theori/documents/report.txt`
    - `/etc/passwd`
      
- **상대 경로(Relative Path)** : 현재 디렉토리를 기준으로 다른 파일에 이르는 경로를 상대적으로 표현한 것
  - `..` : 이전 디렉토리
  - `.` : 현재 디렉토리
  - 한 파일을 가리키는 상대 경로의 수는 무한함
  - - 예시
    ```text
    /
    ├── home
    │   ├── theori
    │   │   ├── documents
    │   │   │   └── report.txt
    │   │   └── downloads
    │   │       └── tool.zip
    │   └── guest
    │       └── notes.txt
    ├── bin
    │   └── ls
    └── etc
        └── passwd
    ```
    - 현재 위치 : `/home/theori`
      - report.txt : `documents/report.txt`
    - 현재 위치 : `/home/theori/downloads`
      - report.txt : ../documents/report.txt

### Path Traversal 
- **Path Traversal** : 권한 없는 경로에 프로세스가 접근할 수 있는 취약점
  - 권한 : 리눅스에서의 권한(rwx)이 아니라, 서비스 로직에서의 권한
    - 서비스 로직에서의 권한 : 서비스가 허용한 경로, 리소스 범위 내에서만 접근해야 한다는 논리적 제한 
  - 서버의 중요한 데이터를 공격자에게 노출시키는 취약점 
