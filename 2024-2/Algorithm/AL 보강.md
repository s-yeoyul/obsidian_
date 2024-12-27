# 1.2 Five Representative Problems
대충 들어봤는데 이거 그냥 첫시간에 했는데...? 
코로나때 녹강 다시 올려주는듯.

# 2.1 Computational Tractability
컴퓨터 계산적으로 접근 가능한가? 에 대한 고민
접근 가능 = 계산 시간이 빠름 = 쉬움.
근데 이것도 한거같음 이미.

# 2.2 Asymptotic Order of Growth
수학적인 증명을 하네? 재밌다!!!
과제1-a 풀 때![](https://i.imgur.com/m88Q602.png)

보면 저거 증명할 때 exist a natural number N s.t. n>N -> f(n) >= c1\*h(n)
의 양식으로 증명하고 있음.
과제 문제나 시험문제 풀 때도 저런 엄밀한 서술을 해야할 듯.

Q. d차 다항식은 Big-Theta(n^d)임을 보이시오.
1. f(n) = O(n^d)
    ![](https://i.imgur.com/Bsush2u.png)
2. f(n) = Big-Omega(n^d)
   ![](https://i.imgur.com/XZoQcO9.png)
![](https://i.imgur.com/tvrg7Bm.png)

![](https://i.imgur.com/QXrLtAH.png)
QED.

# 2.4 Survey of Common Running Times

## 1. Linear Time
예시 
**Computing the maximum**
```
max <- a1
for i = 2 to n {
	if (a_i > max)
		max <- a_i
}
```
**Merge**
Given two sorted lists; A and B, size of n
```
i = j = 1
while (both lists are non-empty){
	if(ai <= bj) append ai to output list and i++
	else append bj to output list and j++
}
append remainder of nonempty list to output list
```
worst case : 2n 
so merge function is O(n)
## 2. nlogn time
divide and conquer에서 자주 발생
**Mergesort**
![](https://i.imgur.com/MLCnv8D.png)

O(n)이 depth(=logn)개. 따라서 O(nlogn).
**Heapsort**
MakeHeap : O(n)
RemoveMin + MakeHeapAgain: O(logn)
=> O(nlogn)
**Largest empty interval**
Timestamp간의 길이가 가장 큰 것을 찾고싶음.
Sort하고 scan해야 함.
=> **O(nlogn)** + O(n)

## 3. Quadratic Time
가장 가까운 2차원 좌표 계산

## 4. Cubic Time
1~n 의 부분집합 S1~Sn의 disjointness를 파악하는 문제
```algorithm
foreach set Si{
	foreach other set Sj{
		foreach element p of Si {
			determine whether p also belongs to Sj
		}
		if (no elem of Si belongs to Sj)
			report that these two are disjoint
	}
}
```

## 5. O(n^k) Time
**Independent set size of k**
Grpah에서 간선으로 연결되지 않은 두개의 노드쌍이 몇 개 있는지 확인.
```
foreach subset of k nodes { // 총 개수 nCk <= n^k / k!
	check whether S in an indep set // O(k^2)
	if (S is an indep set) report S is indep
}
```
O(k^2 * n^k / k!) = O(n^k)

## 6. Exponential Time
**Indep set**
그래프에서 indep set의 크기의 최대
k = 2, 3, 4, ...
```
*S <- NULL
foreach subset S of nodes {
	check whether S in an indep set
	if (S is largest indep set seen so far) update *S <- S
}
```