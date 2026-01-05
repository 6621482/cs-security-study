# Vim

## Vim 모드
```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#ff8a65', 'edgeLabelBackground':'#ffffff', 'tertiaryColor': '#f4f7f6'}}}%%
graph TD
    %% 스타일 정의 (GitHub 호환을 위해 그림자 제거)
    classDef st fill:#f9f9f9,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5,rx:10,ry:10;
    classDef nm fill:#6c63ff,stroke:#4a47a3,stroke-width:3px,color:#fff,rx:15,ry:15;
    classDef om fill:#758bfd,stroke:#5a6bcf,stroke-width:2px,color:#fff,rx:15,ry:15;

    Start(터미널에서 Vim 실행):::st -->|vi| Normal
    
    subgraph Vim_Editor [Vim 에디터 내부]
        direction LR
        Normal[<b>Normal Mode</b><br>일반 모드]:::nm
        Insert[<b>Insert Mode</b><br>입력 모드]:::om
        Command[<b>Command Mode</b><br>명령 모드]:::om

        %% Transitions
        Normal -->|i, o, a...| Insert
        Insert -->|esc| Normal
        Normal -->|:| Command
        Command -->|esc| Normal
    end

    %% 연결선 스타일
    linkStyle default stroke:#5a6bcf,stroke-width:2px,fill:none,interpolate:basis;
