# gdb
- 리눅스의 대표적인 디버거
- ex. 실습예제
    ~~~cpp
    // Name: debugee.c
    // Compile: gcc -o debugee debugee.c -no-pie

    #include <stdio.h>
    int main(void) {
    int sum = 0;
    int val1 = 1;
    int val2 = 2;

    sum = val1 + val2;

    printf("1 + 2 = %d\n", sum);

    return 0;
    }
    ~~~

### entry
- 리눅스의 실행파일 형식 = ELF
  - ELF의 헤더중 __진입점__ 이라는 필드 존재(readelf로 확인 가능)
- gdb의 entry 명령어를 통해 진입점, rip 주소들을 확인 가능

### context
- 각종 메모리를 한눈에 파악하기 쉽게 해줌
- 네 가지 영역으로 구성됨
  - REGISTERS: 레지스터의 상태를 보여줌줌
  - DISASM: rip부터 여러 줄에 걸쳐 디스어셈블된 결과를 보여줌
  - STACK: rsp부터 여러 줄에 걸쳐 스택의 값들을 보여줌
  - BACKTRACE: 현재 rip에 도달할 때까지 어떤 함수들이 중첩되어 호출됐는지 보여줌

### break & continue
- break는 특정 주소에 중단점을 설정하는 기능
- continue는 중단된 프로그램을 이어서 실행함

### run
- 단순 실행

### disassembly
- 프로그램을 어셈블리 코드 단위로 실행하고, 결과를 보여줌
- gdb는 기계어를 디스어셈블하는 기능을 기본적으로 탑재함
- u, nearpc, pdisass 는 디스어셈블된 코드를 더 읽기 쉽게 해줌

### negative
- 관찰하고자 하는 함수의 중단점에 도달 시 ni, si 명령어로 한줄씩 실행하여 볼 수 있음
  - </u>ni는 서브루틴의 내부로 들어가지 않고 si는 들어감</u>

### step info

### examine
- x라는 명령어를 통해 특정 주소에서 원하는 길이만큼 데이터를 원하는 형식으로 인코딩하여 볼 수 있음

### telescope
- 특정 주소의 메모리 값들 보여주고 메모리가 참조하고 있는 주소를 재귀적으로 탐색하여 보여줌

### vmmap
- 가상 메모리의 레이아웃을 보여줌


# pwntools
- 특정 익스플로잇 코드를 파이프를 통해 서버로 보낼 수 있게 도와주는 도구