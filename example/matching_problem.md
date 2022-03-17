# マッチング問題（割当て問題）

## 問題

$n$人の作業員に $m$個の仕事を割り当てるとき、最も費用が最小となる割り当て方を見つけたい。

## 解法アルゴリズム

- ハンガリー法
- 線形計画法（PuLP）

## 例題

各作業員の作業とその費用が下表に示されているとき、総費用が最小となる割り当て方を求めよ。

表：各作業員にかかる費用
| 作業員$i$ / 作業$j$ | a   | b   | c   | d   | e   |
| :---------- | :-- | :-- | :-- | :-- | :-- |
| A           | 9   | 5   | 1   | 9   | 7   |
| B           | 2   | 8   | 2   | 7   | 5   |
| C           | 1   | 3   | 9   | 5   | 9   |
| D           | 8   | 7   | 2   | 6   | 4   |
| E           | 2   | 3   | 6   | 2   | 8   |

## 解答(線形計画法)

### 最小化する目的関数

作業員$i$に作業$j$を割り当てるか否かを、変数$X_{ij}$で表すこととする。また、$X_{ij}$が$1$ならば作業を行うが、$0$ならば作業を行わないとする。作業員$i$が作業$j$を行う費用を$W_{ij}$とすると、最小化する目的関数$z$は次のようになる。

$$
z = \sum_i\sum_j{X_{ij} * W_{ij}}
$$

### 制約条件

各作業員には異なる作業が割り当てられ、各作業には異なる作業員が割り当てられる。そのため制約条件は、「作業員$i$が行う作業は1つ」と、「作業$j$に割り当てられる作業員は1人」となる。

$$
\sum_i{X_{ij}} = 1 (i=0,...,n-1) \\
\sum_j{X_{ij}} = 1 (j=0,...,n-1)
$$

### pythonコード

```python
import pulp

#      a  b  c  d  e
ps = [[9, 5, 1, 9, 7],  # A
      [2, 8, 2, 7, 5],  # B
      [1, 3, 9, 5, 9],  # C
      [8, 7, 2, 6, 4],  # D
      [2, 3, 6, 2, 8]]  # E

def solver(ws):
    prob = pulp.LpProblem('assignment')
    # 変数
    xs = []
    for i in range(len(ws)):
        xs1 = [pulp.LpVariable('x_{}_{}'.format(i, j), cat='Binary') for j in range(len(ws))]
        xs.append(xs1)

    # 目的関数
    prob += pulp.lpSum([pulp.lpDot(ws[i], xs[i]) for i in range(len(ws))])

    # 制約
    for i in range(len(ws)):
        prob += pulp.lpSum(xs[i]) == 1                               # 作業員iに、1つの仕事
        prob += pulp.lpSum([xs[j][i] for j in range(len(ws))]) == 1  # 作業jには、1人の作業員
    
    # 探索開始
    status = prob.solve(pulp.PULP_CBC_CMD(msg=0))
    print("Status", pulp.LpStatus[status])
    for xs1 in xs:
        print([x.value() for x in xs1])
    print(prob.objective.value())

# 実行
solver(ps)
```

次のように解が得られる。割当て結果は（作業員、作業）とすると、 (A, c), (B, a), (C, b), (D, e), (E, d) となり、総費用は$12$となる。

```shell
Status Optimal
[0.0, 0.0, 1.0, 0.0, 0.0]
[1.0, 0.0, 0.0, 0.0, 0.0]
[0.0, 1.0, 0.0, 0.0, 0.0]
[0.0, 0.0, 0.0, 0.0, 1.0]
[0.0, 0.0, 0.0, 1.0, 0.0]
12.0
```

## 参考文献

- [Python3 Programming: PuLP による数理最適化超入門: 割当て問題](http://www.nct9.ne.jp/m_hiroi/light/pulp05.html)
