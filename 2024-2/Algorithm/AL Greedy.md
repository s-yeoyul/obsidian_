# 4.1 Interval Scheduling
## Definition

Job은 start time과 finish time이 존재한다. 두개의 job이 겹치지 않는 경우 **compatible**이라고 한다.
이 때 목표는 **최대 개수의 mutually compatible job들로 구성된 subset을 찾는 것이다.**
집합의 사이즈가 가장 큰 경우를 찾는 것이다!

## Greedy Algorithm

### 1. Earlist start time
![](https://i.imgur.com/1IHSlmc.png)
집합의 사이즈가 1이 되버리면서 fail.
### **2. Earlist finish time**

```pseudocode
INTERVAL_SCHEDULING(job, n)
	sort jobs by finish time in non-decreasing order
	A = empty set
	for i = 1 to n
		if job i is compatible with A
			A = A cup {j}
	return A
```

compatible을 확인하기 위해서는 마지막으로 추가된 job의 finish time을 기억하고 있다가 들어오는 막대의 start time을 비교해주면 됨. 즉 O(1)이고 loop을 도니까 O(n).
Sorting is dominant term, so O(nlogn)![rz0tiRh.png](https://i.imgur.com/rz0tiRh.png)

Optimality를 증명해보자.
![](https://i.imgur.com/rz0tiRh.png)
i_r+1이랑 j_r+1이랑 바꾸면 OPT가 조금 더 최적화됨(여유가 더 생기므로).
**Our Greedy algorithm is as good as optimal solution.**
### 3. Shortest interval
![](https://i.imgur.com/a6D4vMA.png)
음영이 진 부분의 길이가 가장 작다. 그러나 집합의 사이즈가 1이 되어버리면서 fail.
### 4. Fewest conflicts
![](https://i.imgur.com/OT7Y4PV.png)
conflict이 발생하는 막대의 개수가 가장 낮은 막대를 고른다. 저걸 고르면 최대 3개 인데 optimal은 누가 봐도 4개임.

---
# Interval Partitioning

## Definiton
Lecture(Job)의 start time과 finish time이 정의되어 있다.
목표는 **모든 강의를 겹치지 않게 수행하기 위한 최소 강의실 수를 찾는 것**이다!

>[!note] Definition
>Depth: Maximum number that contain any given time. 겹치는 강의 수의 최댓값.

Observation: 강의실의 수는 depth보다는 크거나 같아야 한다.
그러면 강의실이 depth로 해결된다면 그거는 optimal 임이 자명하다.
그러면 우리는 항상 depth 만큼의 partitioning을 할 수 있는가? **YES.**

```pseudocode
sort the intervals by their starting times
Let I1, ... , In be the intervals in this order
for i = 1 to n
	for each interval Ij precedes Ii in sorted order and overlaps it
		Exclude the classroom of Ij from consideration for Ii
	if there is any classroom in {1 ~ d} has not been excluded
		Assign a non-excluded classroom to Ii
	else 
		Leave Ii unlabeled
```

일단 정렬한다. 
정렬 순서대로 강의실에 넣어주면 된다.
빈 공간 있으면 붙이고 아니면 말고.

시간복잡도를 보자.
지금까지 넣은 강의 중 finish time이 가장 빠른거랑 비교해서 오케이면 거기에 넣으면 됨
강의실의 priority queue를 finish time key를 갖게 만들면 됨.
priority queue의 탐색 시간은 logn이므로 for문을 고려하면 nlogn
따라서 sort와 for문 모두 O(nlogn)의 time complexity를 가진다.

optimal함을 보이자.

먼저 어떠한 강의도 unlabel된채로 끝나지 않는다는걸 보이자.\
t + 1 <= d 일 것이다.
그러면 t <= d - 1 이므로 여분이 남아 있으므로 그 자원으로 label을 걸면 된다. 증명 완
다음으로 어떠한 overlapping interval 2개도 같은 Label에 걸리지 않는다. 첫 for문에 의해서 제외(exclude 어쩌고)