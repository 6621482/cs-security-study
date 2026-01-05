# Vim

## Vim Modes
```mermaid
graph TD
    %% 터미널 실행 노드 스타일: 실선 테두리 적용
    Start(터미널에서 Vim 실행) -->|vi| Normal
    style Start fill:#fff,stroke:#333,stroke-width:1px,stroke-dasharray:none

    subgraph Vim [Vim 에디터 내부]
        direction LR
        %% subgraph 배경색 설정: 연한 회색
        style Vim fill:#f9f9f9,stroke:#333,stroke-width:1px

        Normal[Normal Mode<br>일반 모드]
        Insert[Insert Mode<br>입력 모드]
        Command[Command Mode<br>명령 모드]

        Normal -->|i, o, a...| Insert
        Insert -->|esc| Normal
        Normal -->|:| Command
        Command -->|esc| Normal
    end

    %% 노드 스타일: 회색 배경, 얇은 테두리(1px), 검은 글씨
    style Normal fill:#e0e0e0,stroke:#333,stroke-width:1px,color:#000
    style Insert fill:#f5f5f5,stroke:#333,stroke-width:1px,color:#000
    style Command fill:#f5f5f5,stroke:#333,stroke-width:1px,color:#000

