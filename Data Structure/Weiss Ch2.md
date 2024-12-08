## Algorithm Analysis

## 2.3 What to analyze
우리는 보통 average time이나 worst time case를 분석한다. 
best time도 때때로 분석하긴 하지만 typical behaviour을 설명해주지 못하기 때문에 흥미는 없다.

보통 우리가 요구하는 것은 worst case이다. 왜?
avg case는 계산하기도 어려울 뿐더러 avg는 bad input을 고려하지 않는다.
**-> bound 역할을 하는 worst case를 애용.**

## 2.4 Running Time Calculations
### 2.4.1 EX1
```cpp
int sum(int n) {
	int sum; // 1

	sum = 0; // 2
	for(int i = 1; i <= n; i++) { // 3
		sum += i * i * i; // 4
	}
	return sum; // 5
}
```
1. declarations, cost 0
2. assignment, cost 1
3. for loop: initialize 1 +  (N + 1) for all test + N for increment => cost 2N + 2
4. two multiplication, one addition, one assignment per loop => 4N
5. returningcost 1
총 6N + 4 = O(N)
### 2.4.2
```cpp
long fib(int n){
	if(n <= 1) return 1; // 1
	else return fib(n - 1) + fib(n - 2); // 2
}
```
`T(n) = T(n - 1) + T(n - 2) + 2`
2 = 1번째 줄에서 1 + 2번째 줄 addition에서 1
![](https://i.imgur.com/ObQBU27.png)
`T(n) >= fib(n)`이므로 T(n)은 exponentially grow.
![](https://i.imgur.com/ebvryvF.png)
이거 왜 이런지는 Ch7에서 밝혀질 것임.

### 2.4.4 Logarithms?
>[!important] 
>An algorithm is **O(log N)** if it takes constant time to **cut the problem size** by a fraction.

```cpp
long pow(long x, int n){
	if(n == 0) return 1;
	if(n == 1) return x;
	if(isEven(n)) return pow(x * x, n / 2);
	else return pow(x * x, n / 2) * x;
}
```


