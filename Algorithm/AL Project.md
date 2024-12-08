일단 제약 조건 때문에 무조건 Cell 1부터 순차적으로 배치해야 한다고 생각
Cell 1과 연결된 Net을 찾는다 ... O(m)
각 Cell에 대하여 ... O(n)
Given leftmost와 이전 cell의 p_i-1 + w_i-1 을 비교하여 더 큰 위치에 배치
이걸 계속 반복.
만약에 row의 끝을 넘어갈 경우
첫 번째 비교에서 작은 쪽 선택. 
이걸 최대 n번 반복할 수 있으므로, O(n)

따라서 최종적으로는 O(mn^2)가 나오는데

꺼림칙한건 O(m) 부분.. 그리고 optimality.

4번은 
a 누적합을 다 구해놓으며는 상수시간에 처리 가능.
누적합 구하는 과정이 O(K)

1번은
a는 BFS를 통해 깊이를 계산하고
깊이를 같게 만든 뒤
공통 부모를 찾게 하면 된다, O(N)
b는 트리가 링크드리스트가 된 경우가 최악
최선은 balanced tree가 된 경우
c는 

---
프로젝트 문제들에 대한 풀이 접근 방향을 설명해 드릴게요.

## 3. GPT driven DP
**Algorithm Proposal with Time Complexity :**
We are given:
• A row  with  cells  in a fixed order.
• Each cell  has a width .
• nets , each connected to one or more cells.
• Cells outside row  are fixed in position.

**Objective:** Find the placement  of cells in row  that minimizes the total Half-Perimeter Wire Length (HPWL) of all nets:

**Constraints:**
• Cells must be placed in the given order without overlap:
• Cells must be within the row boundaries.
• The placement must be valid (cells do not overlap and stay within the row).

**Algorithm Overview:**
We use a **dynamic programming** approach to find the optimal placement of the cells in row  that minimizes the total HPWL.

**Key Ideas:**

• **Discretize the possible positions** along the row to a finite set of grid points.
• **Dynamic Programming State**: Represents the minimal total HPWL for placing the first  cells ending at a specific position.
• **Transition**: Move from placing  cells to placing  cells by considering all valid positions for cell .
• **Cost Calculation**: For each placement, calculate the incremental HPWL contributed by the nets connected to cell .

**Algorithm Steps:**

1. **Discretize the Row:**

• Divide the row into a set of grid points where cells can start.

• Let the set of possible positions be .

• The number of grid points  is , based on cell widths and row length.

2. **Initialize Dynamic Programming Table:**

• Define  as the minimal total HPWL for placing the first  cells, ending with cell  starting at position .

• Initialize  for all .

3. **Dynamic Programming Iteration:**

• For  to :

• For each possible position  for :

• For each valid previous position  for :

• Ensure non-overlapping constraint:

  

• Calculate the incremental cost:

  

• is the change in total HPWL due to placing  at .

• Update .

4. **Calculate Incremental HPWL ():**

• For each net  connected to cell :

• If  is connected to other cells in row :

• Keep track of the minimum and maximum x-coordinates among connected cells.

• The incremental HPWL is updated based on how placing  at  affects .

5. **Traceback to Find Optimal Placement:**

• After filling the  table, find the minimal total HPWL in  for all .

• Trace back through the  table to determine the positions .


---
  

**1번 문제: Macro Placement (10점)**

  

**(a) Macro Block A 선택 알고리즘 (4점)**

  

• Find_center(U, V) 함수는 두 중요한 블록 U와 V를 기준으로 가장 가까운 공통 부모인 Macro Block A를 찾아 중심에 배치합니다.

• 이 알고리즘은 트리 탐색을 사용하며 BFS나 DFS로 탐색을 진행하여 두 노드의 가장 가까운 공통 부모를 찾을 수 있습니다.

• **Pseudo-Code**:

  
```
int Find_center(int u, int v) {
Queue<int> q;
    q.push(u); q.push(v);
    check[u] = check[v] = true;
    // BFS 또는 DFS로 공통 조상 탐색
    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (int adj : macro[node]) {
            if (!check[adj]) {
                q.push(adj);
                check[adj] = true;
                parent[adj] = node;
            }
        }
    }
    return 공통 조상 node;
}
```
  

**(b) 시간 복잡도 분석 (2점)**

  

• 최적 시간 복잡도: 트리의 높이가 낮을 경우 O(N)보다 작은 시간에 해결할 수 있습니다.

• 최악 시간 복잡도: 모든 노드를 탐색해야 할 때 O(N)의 시간 복잡도가 됩니다.

  

**(c) 개선된 알고리즘 (4점)**

  

• **Dynamic Programming** 적용하여 각 노드의 부모와 깊이를 사전에 계산하여 공통 조상을 빠르게 찾을 수 있습니다.

• 시간 복잡도는 O(log N)까지 줄일 수 있습니다.

  

**2번 문제: Kernighan-Lin 알고리즘을 통한 3D Placement (10점)**

  

**(a) Kernighan-Lin 알고리즘 단계별 시간 복잡도 (4점)**

  

1. **이득 계산**: 각 노드 간의 이동 이득 계산은 O(E)입니다.

2. **교환**: 가장 큰 이득을 가진 쌍을 교환하는데 필요한 시간은 O(V)입니다.

3. **락킹**: 고정된 셀에 대한 관리가 필요하며, 시간 복잡도는 O(V)입니다.

4. **반복**: 절단 비용이 최소화될 때까지 반복하며, 전체 시간 복잡도는 O(V^2)입니다.

  

**(b) 최적성 판단 (2점)**

  

• Kernighan-Lin 알고리즘은 절단 비용을 줄이는 데 최적화되어 있으나, 완벽한 최적해를 보장하지는 않습니다.

  

**(c) 근거 서술 (4점)**

  

• 이 알고리즘은 지역 최적해에 빠질 수 있으므로, 전역 최적해를 보장하는 방법이 아니지만, 일반적으로 좋은 절단 비용을 도출할 수 있습니다.

  

**3번 문제: Standard Cell Placement (10점)**

  

**(a) Polynomial Algorithm (5점)**

  

• 각 net의 HPWL을 최소화하기 위해 row 내에서 cell의 위치를 조정하며, 이를 반복하여 최적의 배치를 찾아냅니다.

• **Pseudo-Code**:

  

Algorithm Minimize_HPWL(row_r, cells_in_r, nets):

    Initialize HPWL_sum = 0

  

    for each net N_i in nets:

        fixed_left_x, fixed_right_x, fixed_top_y, fixed_bottom_y = get_fixed_positions(N_i)

        for each cell C_i in cells_in_r:

            for each possible position p_i:

                current_HPWL = calculate_HPWL(N_i, p_i, fixed_left_x, fixed_right_x, fixed_top_y, fixed_bottom_y)

                HPWL_sum = min(HPWL_sum, current_HPWL)

    return HPWL_sum

  

  

• **복잡도**: 

  

**(b) 시간 복잡도 분석 (2점)**

  

• 각 cell에 대해 모든 가능한 위치를 탐색하므로  복잡도를 가집니다.

  

**(c) Optimality 증명 (3점)**

  

• 주어진 조건에서 모든 위치를 탐색해 최소 HPWL을 찾으므로, 주어진 조건 내에서는 최적의 배치를 찾을 수 있습니다.

  

**4번 문제: Power & Area Optimization (10점)**

  

**(a) Multi-Bit Flip-Flop Banking을 위한 알고리즘 (6점)**

  

• grid 내에서 인접한 flip-flop들의 cost 합을 빠르게 구하는 방식으로, 누적 합을 이용해 효율적인 탐색이 가능하도록 합니다.

• **Pseudo-Code**:

  

Algorithm Calculate_Cost_Sum(grid, queries):

    Precompute cumulative sum for each row and column

    For each query:

        Calculate sum in the specified range using precomputed values

  

  

• **복잡도**:  (누적 합 기반으로 특정 구간의 합을 빠르게 계산 가능)

  

**(b) 구간 합과 특정 원소 수정 알고리즘 (4점)**

  

• 세그먼트 트리를 사용하여 구간 합과 값 업데이트를 O(log N) 시간 복잡도로 수행할 수 있습니다.

• **Pseudo-Code**:

  

int sum(int start, int end, int node, int left, int right) {

    if (범위 벗어남) return 0;

    if (완전 포함) return tree[node];

    int mid = (start + end) / 2;

    return sum(start, mid, node*2, left, right) + sum(mid+1, end, node*2+1, left, right);

}

  

void update(int start, int end, int node, int index, int diff) {

    if (범위 벗어남) return;

    tree[node] += diff;

    if (start != end) {

        int mid = (start + end) / 2;

        update(start, mid, node*2, index, diff);

        update(mid+1, end, node*2+1, index, diff);

    }

}

