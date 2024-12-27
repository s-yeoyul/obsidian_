### Introduction
ASMan 과제는 20개 라운드가 총 3번 반복됩니다. 각 라운드마다 출력되는 정수값을 받고, 적절한 instruction을 사용하여 명령어를 자동으로 보내는 프로그램을 pwntools를 이용해서 작성하면 됩니다. 
### Asman.py
`Asman.py` 코드를 보면, `get_full_asm`에서 `compute` 레이블이 우리가 입력해야 하는 부분임을 알 수 있습니다. 그리고 `main`부분에서, `compute`의 return value가 `%eax`에 저장되어 `printf`함수를 통해 전달되고 있음을 알 수 있습니다. 즉 compute에 들어갈 어셈블리어를 출력하는 코드를 작성하면 됩니다. 이제 각 라운드 별로 어떤 instruction을 사용해서 compute를 완성해야 하는지 알아보겠습니다.
### Round 1 ~ 20
첫 20개 라운드에서는 다음과 같은 instruction을 사용해서는 안됩니다 : `syscall, int, mov, inc, dec`
저는 `lea` instruction을 이용하여 레지스터에 정수 값을 전달하려고 합니다. 예를 들어 주어진 정수값이 n이면 `leal n(%eax), %eax` 를 이용하면 `%eax`의 값이 n 증가합니다. `int`자료형이기 때문에 suffix로 `l`을 사용하였습니다. `asman.py`에서 `main`함수를 보면 `call compute`로 넘어가기 전에 `movl $0, %eax`를 통해 0으로 초기화를 해주기 때문에 `%eax`에는 n이 성공적으로 저장될 것입니다.
### Round 21 ~ 40
다음 20개 라운드에서는 `mov` instruction을 사용해야 합니다. 만약 주어진 정수가 n이라면 `movl $n, %eax`를 이용하여 `%eax`에 n을 저장할 수 있습니다. 
### Round 41 ~ 60
마지막 20개 라운드에서는 `inc, shl` instruction을 사용해야 합니다. 주어진 정수가 n일 때, 제가 사용한 알고리즘은 다음과 같습니다:
1. n이 짝수면 n을 반으로 나누고, 배열에 `shl`를 추가.
   n이 홀수면 n을 decrement하고, 배열에 `inc`를 추가.
2. n이 0이 될 때까지 반복.
3. 배열을 뒤집어서 instruction 출력.
위와 같은 알고리즘을 사용하면 0으로부터 주어진 정수 n을 만들 수 있습니다.
### Pwntools
각각의 반복문마다 `recvuntil`함수를 이용하여 랜덤 정수 값을 받아옵니다. 이를 `int`자료형으로 변환해주고 `sendline, send`함수를 통해 적절한 instruction을 터미널에 보내주게 되면 성공적으로 flag를 얻을 수 있습니다.