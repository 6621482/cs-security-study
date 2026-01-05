# Vim

## Vim 모드
## Vim Modes Overview

```text
                vim 실행
                  │
                  ▼
            ┌─────────────┐
            │ Normal Mode │
            │   (기본)    │
            └─────────────┘
             ▲      │
      ESC ───┘      │ i / a / o
                    ▼
            ┌─────────────┐
            │ Insert Mode │
            │   (입력)    │
            └─────────────┘
                    │
                  ESC
                    ▼
            ┌─────────────┐
            │ Normal Mode │
            └─────────────┘
                    │
                    │ :
                    ▼
            ┌─────────────┐
            │ Command Mode│
            │   (명령)    │
            └─────────────┘
                    │
                  Enter
                    ▼
            ┌─────────────┐
            │ Normal Mode │
            └─────────────┘
