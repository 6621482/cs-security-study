# Vim

## Vim Modes
```mermaid
graph TD
    %% 터미널 실행 노드: 흰색 배경, 실선 테두리
    Start(터미널에서 Vim 실행) -->|vim| Normal
    style Start fill:#fff,stroke:#333,stroke-width:1px,stroke-dasharray:none

    subgraph Vim [Vim 에디터 내부]
        direction LR
        %% 에디터 내부 배경: 아주 연한 회색
        style Vim fill:#f9f9f9,stroke:#333,stroke-width:1px

        Normal[Normal Mode<br>일반 모드]
        Insert[Insert Mode<br>입력 모드]
        Command[Command Mode<br>명령 모드]

        Normal -->|i, o, a...| Insert
        Insert -->|esc| Normal
        Normal -->|:| Command
        Command -->|esc| Normal
    end

    %% 모든 모드 스타일 통일: 진한 회색(#e0e0e0)
    style Normal fill:#e0e0e0,stroke:#333,stroke-width:1px,color:#000
    style Insert fill:#e0e0e0,stroke:#333,stroke-width:1px,color:#000
    style Command fill:#e0e0e0,stroke:#333,stroke-width:1px,color:#000
```

## Normal Mode (일반 모드)
- 처음 실행하면 일반 모드로 시작
- 방향키, Home, End 등의 키를 통해 커서 이동 가능

- 커서 이동
  - `h` : 왼쪽으로 이동
  - `j` : 아래로 이동
  - `k` : 위로 이동
  - `l` : 오른쪽으로 이동
    
  - `0` : 현재 **행의 처음**으로 이동
  - `$` : 현재 **행의 끝**으로 이동
    
  - `gg` : **첫 행**으로 이동
  - `G` : **마지막 행**으로 이동
    
- 삭제 / 복사 / 붙여넣기
  - `x` : 현재 커서 위치의 **글자 삭제** (Delete)
  - `X` : 현재 커서 **앞 글자 삭제** (Backspace)
    
  - `dd` : 현재 커서가 위치한 **행 삭제**
  - `yy` : 현재 커서가 위치한 **행 복사**
    
  - `p` : 복사/삭제한 내용을 **현재 행 이후에 붙여넣기**
  - `P` : 복사/삭제한 내용을 **현재 행 이전에 붙여넣기**
 
- 문자열 찾기
  - `/문자열` + `Enter` : 현재 커서 **이후에서 문자열 검색**
  - `n` : 다음 검색 결과로 이동
  - 
- 되돌리기
  - `u` : **이전 수정 사항 되돌리기**
