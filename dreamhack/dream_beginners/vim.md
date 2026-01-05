# Vim

## Vim Modes
```mermaid
graph TD
    %% 노드 정의
    Start(터미널에서 Vim 실행) -->|vi| Normal

    subgraph Vim [Vim 에디터 내부]
        direction LR
        Normal[Normal Mode<br>일반 모드]
        Insert[Insert Mode<br>입력 모드]
        Command[Command Mode<br>명령 모드]

        %% 흐름 연결
        Normal -->|i, o, a...| Insert
        Insert -->|esc| Normal
        Normal -->|:| Command
        Command -->|esc| Normal
    end

    %% 가장 기본적인 스타일 적용 (에러 안 남)
    style Normal fill:#6c63ff,stroke:#333,stroke-width:2px,color:#fff
    style Insert fill:#758bfd,stroke:#333,color:#fff
    style Command fill:#758bfd,stroke:#333,color:#fff
    style Start fill:#fff,stroke:#333,stroke-dasharray: 5 5

