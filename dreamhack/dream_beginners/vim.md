# Vim

## Vim 모드
```mermaid
graph TD
    Start(터미널에서 Vim 실행) -->|vi| Normal
    
    subgraph Vim_Editor [Vim 에디터 내부]
        direction LR
        Normal[<b>Normal Mode</b><br>일반 모드]
        Insert[<b>Insert Mode</b><br>입력 모드]
        Command[<b>Command Mode</b><br>명령 모드]

        %% Transitions
        Normal -->|i, o, a...| Insert
        Insert -->|esc| Normal
        Normal -->|:| Command
        Command -->|esc| Normal
    end

    style Normal fill:#6c63ff,stroke:#333,stroke-width:2px,color:#fff
    style Insert fill:#758bfd,stroke:#333,color:#fff
    style Command fill:#758bfd,stroke:#333,color:#fff
    style Start fill:#fff,stroke:#333,color:#000,stroke-dasharray: 5 5
