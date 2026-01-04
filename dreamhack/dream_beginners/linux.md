# Linux Basic Commands

- `id` : 현재 유저의 유저 ID와 해당 유저가 속해있는 그룹 ID를 출력
  - 리눅스는 권한을 기반으로 파일을 읽고 쓰고 실행할 수 있기 때문에 주로 자신이 해당하는 권한을 가지고 있는지 확인하기 위해 사용
- `pwd` : 현재 작업 중인 디렉토리의 경로를 출력
- `ls` : 디렉토리의 내용을 출력
- `cd` : 작업 중인 디렉토리 변경
  - 절대 경로 : 루트 디렉토리 /를 시작으로 모든 경로를 적어서 표현하는 경로
  - 상대 경로 : 현재 디렉토리를 기준으로 상위 디렉토리 또는 하위 디렉토리로 뻗어 나가는 경로
    - ex. `cd..` : 현재 디렉토리에서 부모 디렉토리로 이동
    - 현재 디렉토리가 `/home/user/`일 때 `cd..`을 실행하면 `/home`으로 이동
```
 user@user-VirtualBox:~$ pwd
/home/user
user@user-VirtualBox:~$ cd /
user@user-VirtualBox:/$ pwd
/
user@user-VirtualBox:/$ cd /home/user
user@user-VirtualBox:~$ pwd
/home/user
user@user-VirtualBox:~$ cd ..
user@user-VirtualBox:/home$ pwd
/home
user@user-VirtualBox:/home$ cd ~
user@user-VirtualBox:~$ pwd
/home/user
user@user-VirtualBox:~$
```

