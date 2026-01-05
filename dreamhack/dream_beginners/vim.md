# Vim

## Vim Modes
```mermaid
graph TD
    Start(터미널에서 Vim 실행) -->|vi| Normal

    subgraph Vim [Vim 에디터 내부]
        direction LR
        Normal[Normal Mode<br>일반 모드]
        Insert[Insert Mode<br>입력 모드]
        Command[Command Mode<br>명령 모드]

        Normal -->|i, o, a...| Insert
        Insert -->|esc| Normal
        Normal -->|:| Command
        Command -->|esc| Normal
    end

    %% 스타일 설정: 회색 배경, 얇은 테두리(1px), 검은 글씨
    style Normal fill:#e0e0e0,stroke:#333,stroke-width:1px,color:#000
    style Insert fill:#f5f5f5,stroke:#333,stroke-width:1px,color:#000
    style Command fill:#f5f5f5,stroke:#333,stroke-width:1px,color:#000
    style Start fill:#fff,stroke:#333,stroke-dasharray: 5 5,stroke-width:1px

