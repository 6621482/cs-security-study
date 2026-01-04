# Linux Basic Commands

- `id` : 현재 유저의 유저 ID와 해당 유저가 속해있는 그룹 ID를 출력
  - 리눅스는 권한을 기반으로 파일을 읽고 쓰고 실행할 수 있기 때문에 주로 자신이 해당하는 권한을 가지고 있는지 확인하기 위해 사용
- `pwd` (Print Working Directory) : 현재 작업 중인 디렉토리의 경로를 출력
- `ls` (List) : 디렉토리의 내용을 출력
  - `ls ..` : 상위 디렉토리의 파일 목록 출
- `cd` (Change Directory) : 작업 중인 디렉토리 변경
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
user@user-VirtualBox:/home$ pwd그냥 
/home
user@user-VirtualBox:/home$ cd ~
user@user-VirtualBox:~$ pwd
/home/user
user@user-VirtualBox:~$
```
- `mkdir` (Make Directory) : 디렉토리 생성
  - ex. `mkdir new_dir`명렁어로 new_dir 디렉토리를 생성하면, 현재 디렉토리에 새롭게 추가됨
- `touch` : 비어 있는 새로운 파일을 만드는 데 사용
  - 앞서 생성한 new_dir 디렉토리로 이동 후 `ls -l`을 실행해보면 아무런 파일도 존재하지 않음
  - 이때 `touch new_file` 명령어를 실행한 후 `ls -l`을 실행하면, new_file 파일이 생성됨
-`mv` (Move) : 파일이나 디렉토리의 위치를 옮기거나 이름을 변경할 때 사용
  - ex. `mv new_file old_file` : new_file을 old_file로 변경
  - ex. `mv old_file ..` : old_file을 상위 디렉토리로 이동
- `rm` (Remove) : 파일을 삭제하는 명령어
  - 디렉터리는 `rm`명령어로 삭제되지 않음
  - `rm -r` : 디렉토리와 그 안의 모든 파일을 재귀적으로 삭제
- `cat` : 파일의 내용 출력
  - 형식 : `cat 파일경로` ex. cat /etc/passwd
- `file` : 파일의 유형 출력
  - 형식 : `file 파일경로`
- `echo` : 셸에 유저가 입력한 텍스트를 출력
  - `echo` 명령문 끝에 `> 파일명`을 이어붙여 실행하면 `파일명`을 이름으로 가지는 파일을 생성하고, `echo` 뒤에 입력한 내용을 파일 내용으로 저장
```
user@user-VirtualBox:~/new_dir$ echo "Hello world!"
Hello world!
user@user-VirtualBox:~/new_dir$
```
- `cp` (Copy) : 파일을 복사하는 명령어
  - 디렉토리를 복사할 때는 플래그를 붙인 형태인 `cp -r` 사용
  - ex. `cp hello world` : hello 파일을 world라는 이름으로 복사
- `grep` : 전체에서 특정 문자열을 찾을 때 사용
  - 형식 : `grep 문자열 파일`
  - 문자열이 포함된 행 전체를 출력
- `man` (Manual) : 특정 명렁어의 매뉴얼을 보여주는 명령어
  - 매뉴얼은 명렁어 사용법, 옵션, 예제 등 유용한 정보를 담고 있음
- `curl` (client URL) : 서버에 데이터를 보내거나 서버로부터 데이터를 받는 전송 명령어
  - 형식 : `curl [옵션] URL`
    - 옵션 1) `-o file` : 전송 받은 데이터를 파일에 저장
    - 옵션 2) `-i` : 결과 값에 HTTP 응답 헤더를 포함
    - 옵션 3) `X "method"` : HTTP 요청 메소드를 지정
    - 옵션 4) `-d "key=value"` : HTTP POST 메소드로 데이터를 전송
  - HTTP, HTTPS, FTP 등 다양한 프로토콜 지원
  - curl을 이용한 명령어 실행 결과 전송
 


### 추가 내용
- **HTTP 통신**
  - 기본 구조 : [요청(Request)] -> [응답(Response)]
  - 요청 : 요청라인 | 요청헤더 | (빈 줄) | 요청바디
    - 요청 메서드 : GET (정보 얻음) & POST (정보 보냄)
    - 헤더 : 부가정보 (형식, 데이터 길이 등)
    - 바디 : 진짜 내용
    > 실제 형태 :
    > POST /login HTTP/1.1
    > Host: example.com
    > Content-Type: application/x-www-form-urlencoded
    >
    > id=admin&pw=1234

  
  - 응답 : 상태라인 | 응답헤더 | (빈 줄) | 응답바디
