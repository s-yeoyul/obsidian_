binary만 가지고 분석하는 방법!
디컴파일이라는 걸 씀. 

취약점
scanf로 %s를 하니까 overflow가 발생.
pw에서 stack overflow가 나기는 함.

pw를 길게 입력함. 32바이트보다 길게 입력
username이 pw에의해 덮인다,..!!
pw를 길게 줘서 username을 admin으로 덮어버리면 된다.

send, read가 잘 안되면 사이에 sleep 넣는것도 도움이 된다.
vis를 써가지고 heap 구조 보면서 진행.
vis 깨지면 tel

40~70까지 덮어야 함. 0x30 x 'Q'+ admin
새로운 비밀번호는 QQQ...QQadmin이 됨.

흠!

QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQadmin