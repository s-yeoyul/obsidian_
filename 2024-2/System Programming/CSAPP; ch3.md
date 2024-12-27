`linux > gcc -0g -o p p1.c p2.c
-0x는 optimization level
O1, O2는 높은 최적화가 수행되지만 이해하기 어렵다. 성능은 좋다
우리는 공부하는 거니까 Og로 하면 된다. 
![[Pasted image 20240918000125.png]]
suffix table!
Intel의 초기 processor는 16비트인데 이것땜시롱 word=16비트=2바이트.
즉 w stands for word = 2byte.

# 3.4 Accessing Informations
CPU는 16개의 레지스터로 이루어져 있고 integer data를 저장한다.
![[Pasted image 20240918001045.png]]
레지스터는 각각의 기능이 있다. 자유롭게 사용할 수 있지만 convention이 있다. 다룰 예정.
가장 unique한 기능은 `%rsp`로, run-time stack의 end position을 가리키는 역할을 한다.
![[Pasted image 20240919191510.png]]
`%rax`는 function의 return value 역할을 한다.
그리고 line 2는 mem -> reg, line 3은 reg -> mem. 

1. pointer는 사실 그냥 address를 의미하는 것이고, dereferencing a pointer는 pointer를 register로 카피해서 이 레지스터를 메모리 참조로 쓰는것이다.
2. 지역변수 x는 메모리보단 레지스터에 바로 저장될때도 있다. 그게 훨씬 빠름.

C에서 \*는 dereferencing operator로써 장소에 저장된 값에 접근하게 함.
반면 &는 pointer를 create함.
![[Pasted image 20240919195300.png]]
#### `pushq %rax`
rax에 저장된 값이 스택에 push되며 stack이 아래로 자란다.
스택은 0x8만큼 자란다.. (subq $8, %rsp)
실제로 pushq는 1바이트로 인코딩 되어있지만 subq + movq로 바꾸면 8바이트가 된다.

#### `popq %rdx`
스택의 top 값이 rdx에 저장되며 pop되고 stack이 위로 준다.
당연히 8만큼 줄고, 적혀있던 값은 사라지지는 않는다(위 그림 3열 참조), 그러나 다른 값이 들어오면 Overwrite된다.

스택의 top은 언제나 %rsp가 관리한다....!!


## ALU
![[Pasted image 20240921162436.png]]
AT&T convention의 비직관적인 operand 순서에 주의합시다..

`lea(load effective address)` 함수는 S의 effective address(=&S)를 카피해서 D에 load 한다.
mov랑 비슷하지만 메모리를 참조하지 않는다. 즉 reg에 저장된 값 그 자체로 연산을 한다, 저장된 값을 주소취급하여 dereference하지 않는다는 소리..!
lea가 사용되는 이유는 simple arithmetic을 이용하고 싶기 때문이다.
#### Problem 3.10

![[Pasted image 20240921202540.png]]
나는 답을 이렇게 생각했다.
```c
short arith3(short x, short y, short z) {
	short p1 = y | z;
	short p2 = p1 >> 9;
	short p3 = ~p2;
	short p4 = p3 - y;
	return p4;
}
```
그런데 답지에서는 `short p4 = y - p3;` 이라고 ,...
왜그러지 싶어서 저걸 그대로 asm to C converter에 넣어보았다.
![[Pasted image 20240921202718.png]]
마지막 줄에 `rbx -= rsi;` 라 나와있는 걸 보니 `p4 = p3 - y` 가 맞다. 내말이 맞다 ㅎ

**근데.. bax저거 오타겠지..? 나중에 조교님에게든 교수님에게든 물어보아야겠다.**
![[Pasted image 20240921203828.png]]
그리고 reg를 0으로 initialize하는데 사용되는 xor 연산은, 0으로 초기화하는 방법 중 가장 필요한 바이트 수가 적어서 가장 효율적인 방법이다.
논리회로 수준에서 생각해봐도 mov에 사용되는 sequential logic보단 xor로 combinational하게 해결하는 방법이 더욱 효율적임을 알 수 있다. 

# 3.6 Control

CPU는 AL 계산을 묘사하는 성질을 들고잇다. condition code에.
![[Pasted image 20240921205819.png]]
위 연산은 저장된 값을 건들지 않고, 즉 update 하지 않고 갖고 놀기만 한다.
자동으로 바뀌는 Condition Code를 Explicit하게 설정하는 방법이다.

![[Pasted image 20240921212804.png]]
direct jump는 Label로 바로 뛴다.
`jmp .L1` 같은거.
반면 indirect jump는 점프 지점을 레지스터나 메모리로부터 읽어온다.
`jmp *%rax`, `jmp *(%rax)` 같은거.

jump instruction이 binary level에서 어떻게 이루어지는지 한 번 확인해보자기.
![[Pasted image 20240921214247.png]]
jmp instruction은 line 2에 위치한다. 
binary를 보면 
	4004d3: eb 03
이라고 되어 있다. eb는 jmp, 03은 얼마나 점프를 뛰는지를 의미한다. (Offset)
다음줄의 주소는 4004d5이다. 여기에 offset을 더해주면 4004d8이 된다. 
4004d8이 바로 jmp의 destination이다.
![[Pasted image 20240921214436.png]]
비교하면 명확해진다.
![[Pasted image 20240921220834.png]]
이를 gcc로 컴파일할 때
![[Pasted image 20240921220437.png]]
컴파일러가 if, else statement를 만들 때 C code를 저런 식으로 해석하게 된다.
**즉, 컴파일러는 then-statement와 else-statement를 공간적으로, block 단위로 분리하게 된다!!**
# 3.7 Procedures
procedure = functions = method = ...

P 가 Q를 call -> Q가 execute -> P로 return 되는 과정에서 다음과 같은 메커니즘이 필요하다.

1. passing control: Q의 시작 주소 및 P로 return할 주소 컨트롤
2. Passing data: P는 여러개의 argument를 Q로 전달할 수 있어야 하고 Q는 P에게 value를 return해야 한다
3. Allocating and deallocating memory: Q가 시작될 때 지역 변수를 위한 공간을 할당해야 하고 return하기 전에 free해야한다.

함수가 call되고 return되는 과정에서 원래 실행되고 있던 procedure는 suspended된다. 
이는 Last in First Out 되는 자료구조인 stack의 pipelining을 이용하여 관리가 가능하다.

![](https://i.imgur.com/egZgXdN.png)

현재 실행중인 procedure는 항상 스택의 탑에 있다. 
P가 Q를 call하면, 스택의 P의 return address가 push 된다.
Q에게는 스택 바운더리를 연장하면서 지역 변수를 저장, argument를 세팅할 공간이 스택 프레임에서 할당된다.

## .2 Control transfer

call을 하면 address를 스택에 push하고 PC를 Q의 시작점으로 세팅한다. 
push한 주소는 return address라고 불리고, call instruction의 바로 다음 지점을 가리킨다.
반대로 ret을 하면 return address를 스택에서 pop하고 PC를 A로 세팅한다. 

	callq, retq 는 IA32가 아니고 x86-64임을 강조하기 위해 붙은 것이다.

![](https://i.imgur.com/O4RI0lX.png)
`%rip는 명령어의 주소를 가리킴`
![](https://i.imgur.com/JS3FwiD.png)

여기서 `%rsp`의 주소가 왜  0x7fffffffe820 에서 0x7fffffffe818로 바뀌는지 모르겠다.
call을 하면 `-8(%rsp)`을 해서 0x7fffffffe810으로 되는건 알겠는데...

>[!note] 
>0x20 - 0x8 = 0x18이다 ㅎㅎ
>16진수 연산은 항상 조심하자! 특히 0이 있을 때 10으로 생각하기 쉬우니...

## .3 Data Transfer
Procedure call에는 passing data as argument 작업이 필요하고
returning from a procedure 에도 역시 returning value 작업이 필요하다.
x86-64에서 이 작업은 register를 통해 일어난다.
![](https://i.imgur.com/AuMf3Jx.png)

인자가 6개를 넘어가면 앞서 보았듯이 7번째 인자부터는 스택의 top인 Argument build area에서 관리한다.

## .4 Local Storage on stack
![](https://i.imgur.com/zpUcBCU.png)
이 예제를 이해해보는 것이 procedure을 전체적으로 이해하는데 도움이 되는 것 같다. 
rdi, rsi는 특정 함수의 argument로 쓰이는 것이 아니라 그냥 모든 함수가 rdi, rsi, ... 에 있는 녀석들을 인자로 받아서 쓴다. 그렇게 정해져 있다. 그게 ABI 이고...!!

## .5 Local Storage in Register
caller 함수가 callee 함수를 call 할 때, callee 함수는 caller가 찜콩해둔 reg에 overwrite 해서는 안된다.
이를 위해 x86-64에는 레지스터 사용 convention이 존재한다...!

convention에 의해 %rbx, %rbp, %r12 ~ %r15는 callee-saved reg이다.
즉 P가 Q를 call했을때 Q는 지역 변수를 위에 나열한 곳에 저장해야 한다는 소리.
**Q가 아무리 우당탕탕 지랄을 해도 저기에 저장해놓은 값은 무적이다. 절대로 corrupt되지 않는다.**

상기한 reg 외에 다른 모든 레지스터(%rsp 제외)는 caller-saved reg이다.
즉 얘네는 어떠한 함수가 와도 쓸 수 있는 자유로운 레지스터라는 소리.

## .6 Recursive Procedures
이게 나온 과제니까 집중해서 읽고 과제를 바로 해보자...!!

# 3.8 Array Allocation and Access
## .1 Basic Principles
`T A[N]` 에서 시작 위치를 x라 하자.
위 코드를 Declare하는 순간 2가지 일이 발생한다.

>[!important] 
>1. T의 크기 x N 만큼의 contiguous memory allocation
>2. A라는 identifier를 array의 시작점으로 사용할 수 있게 introducing, 그 값은 x

![](https://i.imgur.com/Eym3XfO.png)

만약 `int E[n]` 이 선언 되었고 i-th index의 값을 `%eax`에 참조하고 싶다면
`movl (%rdx, %rcx, 4), %eax // E in %rdx, i in %rcx`
라고 작성하면 된다.


# 3.9 Heterogeneous DS
![](https://i.imgur.com/xIir6Y9.png)

## .1 structure

>structure: groups objects of possibly different types into a single object.

structure의 구현은 배열과 비슷하다, 구조체의 구성 요소가 contiguous하게 메모리에 배치된다는 점에서!
![](https://i.imgur.com/mctoAWh.png)
와 같은 structure는,
![](https://i.imgur.com/oqKmTPv.png)
의 형식으로 저장된다.
int는 4바이트, 포인터는 8바이트.
int 배열은 4바이트 x size.
**using these offsets as displacements in memory referencing instructions!!!**


