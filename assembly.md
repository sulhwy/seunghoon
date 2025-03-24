# x86 어셈블리어
- 0과1로 되어있는 기계어를 다루기 쉽게 도와주는 통역사임

## 어셈블리 언어
- 다양한 ISA가 있듯이 x64어셈블리어, ARM 어셈블리어가 있음

### x64 어셈블리 언어
- 구조
  - __명령어(opcode)__ 와 __피연산자(operand)__ 로 구성됨
![Alt text](/ass.png)
#### 명령어들
1. 데이터 이동 = mov, lea
2. 산술 연산 = inc, dec, add, sub
3. 논리 연산 = and, or, xor, not
4. 비교 = cmp, test
5. 분기 = jmp, je, jg
6. 스택 = push, pop
7. 프로시져 = call, ret, leave
8. 시스템 콜 = syscall

#### 피연산자
- 상수, 레지스터, 메모리 3가지가 올 수 있음
- 메모리 피연산자는 []로 둘러쌓인 것으로 표현되고 앞에 크기 지정자 'TYPE PTR'이 추가될 수 있음
  - TYPE에는 BYTE, WORD, DWORD, QWORD가 올 수 있으며 각각 1바이트, 2바이트, 4바이트, 8바이트의 크기를 지정함
  - ex.
    - QWORD PTR[0x8048000] => 0x8048000의 데이터를 8바이트만큼 참조
    - WORD PTR[rax] => rax가 가리키는 주소에서 데이터를 2바이트만큼 참조

### x86-64 어셈블리 명령어

#### 1. 데이터 이동
- <u>어떤 값을 레지스터나 메모리에 옮기도록 지시함</u>
  - ex. mov는 주소에 들어있는 값을 대입, lea는 주소를 대입
    - mov dst, src : src에 들어있는 값을 dst에 대입
      - mov QWORD PTR[rdi], rsi : rsi에 들어있는 값을 rdi가 가리키는 주소에 8바이트로 대입
    - lea dst, src : src의 유효 주소(Effective Address, EA)를 dst에 저장함
      - lea rsi, [rbx+8*rcx] : rbx+8*rcx를 rsi에 대입

#### 2. 산술 연산
- <u>덧셈, 뺄셈, 곱셈, 나눗셈 연산을 지시함</u>
- ex.
  - add dst, src : dst에 src의 값을 더함
    - add ax, WORD PTR[rdi] : ax += *(WORD *)rdi => *은 역참조로 rdi가 가리키는 __메모리 주소에서 2바이트 크기의 값을 가져옴
  - sub dst, src: dst에서 src의 값을 뺌
    - sub ax, WORD PTR[rdi] : ax -= *(WORD *)rdi
  - inc op: op의 값을 1 증가시킴
    - inc eax : eax += 1
  - dec op: op의 값을 1 감소 시킴
    - inc eax : eax -= 1

#### 3. 논리 연산
- <u>and, or, xor, neg, not 등의 비트 연산을 지시함</u>
  - and dst, src: dst와 src의 비트가 모두 1이면 1, 아니면 0
~~~cpp
[Register]
eax = 0xffff0000
ebx = 0xcafebabe

[Code]
and eax, ebx

[Result]
eax = 0xcafe0000
~~~
  -  or dst, src: dst와 src의 비트 중 하나라도 1이면 1, 아니면 0
~~~cpp
[Register]
eax = 0xffff0000
ebx = 0xcafebabe

[Code]
or eax, ebx

[Result]
eax = 0xffffbabe
~~~
  - xor dst, src: dst와 src의 비트가 서로 다르면 1, 같으면 0
~~~cpp
[Register]
eax = 0xffffffff
ebx = 0xcafebabe

[Code]
xor eax, ebx

[Result]
eax = 0x35014541
~~~
  - not op: op의 비트 전부 반전
~~~cpp
[Register]
eax = 0xffffffff

[Code]
not eax

[Result]
eax = 0x00000000
~~~

#### 4. 비교
- <u>두 피연산자의 값을 비교하고, 플래그를 설정함</u>
  - ex.
  - cmp op1, op2: op1과 op2를 비교
    - @ 값은 op1에 대입하지 않음
    - 1. mov rax, 0xA
    - 2. mov rbx, 0xA
    - 3. cmp rax, rbx ; ZF=1
  - test op1, op2: op1과 op2를 비교
    - @ test는 두 피연산자 op1, op2에 and연산을 취함. 마찬가지로 값은 op1에 대입하지 않음
    - xor rax, rax
    - test rax, rax ; ZF=1

#### 5. 분기
- <u>분기 명령어는 __rip__ 을 이동시켜 실행 흐름을 바꿈</u>
  - ex.
  - jmp addr:addr을 rip으로 이동시킴
    - 1. xor rax, rax
    - 2. jmp 1 ; jump to 1
  - je addr : 직전 비교한 두 피연산자가 같으면 점프
    - 1. mov rax, 0xcafebabe
    - 2. mov rbx, 0xcafebabe
    - 3. cmp rax, rbx ; rax = rbx
    - 4. je 1 ; jump to 1
  - jg addr : 직전 비교한 두 연산자 중 전자가 더 크면 점프
    - 1. mov rax, 0x31337
    - 2. mov rbx, 0x13337
    - 3. cmp rax, rbx ; rax>rbx
    - 4. jg 1 ; jump to 1

### Opcode: 스택
- ex.
  - push val : val을 스택 최상단에 쌓음
    - 연산
    - rsp -= 8
    - [rsp] = val
  - 예제
~~~cpp
[Register]
rsp = 0x7fffffffc400

[Stack]
0x7fffffffc400 | 0x0  <- rsp
0x7fffffffc408 | 0x0

[Code]
push 0x31337
~~~
  - 결과
~~~cpp
[Register]
rsp = 0x7fffffffc3f8

[Stack]
0x7fffffffc3f8 | 0x31337 <- rsp 
0x7fffffffc400 | 0x0
0x7fffffffc408 | 0x0
~~~
  - pop reg : 스택 최상단에 저장된 값을 꺼내서 레지스터인 reg에 대입
    - 내부적으로 수행되는 연산
    - reg = [rsp]
      rsp += 8

  - 함수 호출 및 반환 관련 명령어
    - ex. call, ret, leave
      - call addr : 주소 값 addr에 위치한 프로시져 호출
      - leave : 스택프레임을 정리함
      - ret : 함수에서 반환하여 원래 실행하던 코드로 돌아오는 명령어

### 시스템 콜
- 커널모드 : 운영체제가 시스템을 제어하기 위해 시스템 소프트웨어에 부여하는 권한
- 사용자모드: 운영체제가 사용자에게 부여하는 권한
- 시스템 콜: 유저 모드에서 커널 모드의 시스템 소프트웨어에게 어떤 동작을 요청하기 위해 사용됨
![Alt text](/syscall.png)
