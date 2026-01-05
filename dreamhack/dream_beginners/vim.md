# Vim

## Vim Modes
```mermaid
graph TD
    %% 터미널 실행 노드: 흰색 배경, 실선 테두리
    Start(터미널에서 Vim 실행) -->|vi| Normal
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

