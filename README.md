# 设计和分析高效算法

三种技术：动态规划、贪婪算法和摊还分析。

## 动态规划

分治法将问题划分为不相交的子问题，递归地解决子问题，然后组合它们的解决方案来解决原始问题。而动态规划适用于当子问题重叠的情况（即子问题共享子子问题时）。

动态规划通常适用于优化问题。这类问题可能有许多可能的解决方案。每个解都有一个值，而你希望找到一个具有最优（最小或最大）值的解。将该解决方案称为一个最优解决方案，而不是仅此一个最优（可能有多个方案达到最优值）。

设计动态规划算法，分为四步：

1. 描述最优解的结构
2. 递归地定义最优解的值
3. 计算最优解的值，通常是自下而上的形式
4. 根据计算结果构建最优解

### 切杆问题

将一个杆切成几段，不同段长的杆价格不一样，要解决一个如何进行切杆才能使得利益最大化的问题。

当 $i=1, 2,...,$ 时，长度为 i 英寸的竿的价格为 $p\_i$，每根杆的长度都是整数。问题为：给定一根长度为 n 英寸的木棒和 $i = 1,2,...,n$ 的价格 $p\_i$ 表，求出将木棒切割成块出售可获得的最大收入 $r\_n$。当杆长为 n 的价格 $p\_n$ 足够大时，就不存在需要切割的最佳方案了。

有 $2^{n-1}$ 种不同的方法切割长度为 n 的木棒，因为可以在离左端 i 英寸处选择切割或不切割，其中$i = 1，2，...，n - 1$。（如果需要按分割部分递增地切，则考虑的情况会少一些，即切成0段，1段...n段，一共 n+1 种情况）如果最优解将杆切为 k 段($1\leq k\leq n$)，则对于将杆切为 $i\_1,i\_2,...,i\_k$ 的最优切法 $n=i\_1+i\_2+\cdots+i\_k$ 有相应的最大收入 $r\_n=p\_{i\_1}+p\_{i\_2}+\cdots+p\_{i\_k}$。

更一般地，可以用稍短的杆的最优收益来表示 $n\geq 1$ 的 $r\_n$:

$$r_n=\max\{p_n,r_1+r_{n-1},r_2+r_{n-2},...,r_{n-1}+r_1\}\tag{14.1}$$

第一个参数 $p\_n$ 对应没有切割，另外 $n - 1$ 个参数对应的是，在每个 $i = 1,2,...,n - 1$ 的情况下，将杆初始切割成大小为 $i$ 和 $n - i$ 两段，然后进一步优化切割，从这两段中获得收益 $r\_i$ 和 $r\_{n-i}$。由于无法提前知道哪个 $i$ 值能使收益最大化，因此必须考虑 $i$ 的所有可能值，并选择能使收益最大化的值。

要解决大小为 $n$ 的原始问题，需要解决相同类型的更小问题。一旦进行了第一次切割，由此产生的两个部分就构成了杆切割问题的独立实例。整体最优解包含了由此产生的两个子问题的最优解，使这两个子问题的收益最大化。则切杆问题表现出最优子结构：一个问题的最优解包含了相关子问题的最优解，这些子问题可以独立解决。

用一种稍简单的方法来安排该问题的递归，还是将分解看作两段组成，不同的是这里只对第二部分进行进一步分解，然后可以得到下式：

$$r_n=\max\{p_i+r_{n-i}:1\leq i\leq n\}\tag{14.2}$$

在这种表述中，最优解只包含一个相关子问题的解，而不是两个。

#### 自上而下递归解法

下面的 CUT-ROD 程序用直接的自上而下的递归实现了上面的式子。它的输入为价格 $p\[1:n]$ 和一个整数 n，返回长度为 n 的杆子的最大可能收益。对于长度 $n=0$，不需要递归，直接返回 0。紧接着初始化最大收益 $q=-\infty$，for 循环中计算 $q=\max{p\_i+\mathrm{CUT-ROD}(p,n-i):1\leq i\leq n}$，然后返回这个值。利用上式对 n 进行归纳可以证明这个答案等于所求 $r\_n$。

```
CUT-ROD(p,n)
if n == 0
	return 0
q = -infty
for i = 1 to n
	q = max{q, p[i] + CUT-ROD(p, n-i)}
return q
```

当输入大小稍大时，运行时间将会特别长，可见该算法的效率很低。问题在于，CUT-ROD 会以相同的参数值反复调用自身，这意味着它会重复求解相同的子问题。

现在分析该算法的运行时间，用 $T(n)$ 表示对于长度 n 时调用 CUT-ROD(p, n) 的总次数，也就等于递归树中根标号为 n 的子树上的节点数，包含根节点的初始调用。因此 $T(n)=1$，以及：

$$T(n)=1 + \sum_{j=0}^{n-1}T(j)\tag{14.3}$$

可以解得 $T(n)=2^n$，因此运行时间为 n 的指数级。再分析一下，由于算法考虑了切割的所有可能方法，因此对于长度 n 的杆有 n-1 个切割位置，也就是会产生 $2^{n-1}$ 种切割方法，而递归树中的叶节点对应了这些切割方法。递归树有 $2^{n-1}$ 个叶节点。从树根到树叶的简单路径上的标签给出了每次切割前右边剩余部分的大小。也就是说，这些标签给出了从杆的右端开始测量的相应的切点。

#### 使用动态规划找最优切割

可以使用动态规划使上面的递归方法变为高效的算法。动态规划的工作原理如下，不要像递归法那样反复求解相同的子问题，而是让每个子问题只求解一次。实际上有一个显然的方法：在第一次求解一个子问题时，保存它的解。如果以后需要再次参考这个子问题的解法，只需查找即可，而不必重新计算。

保存子问题解决方案需要付出代价：即需要额外的内存。因此，动态规划是**时间与内存权衡**的一个例子。节省时间的效果可能非常显著，例如，我们将使用动态规划，从指数时间的切杆算法缩短到 $O(n^2)$ 时间的算法。当涉及的不同子问题的数量是输入大小的多项式时，动态规划的运行时间就是多项式时间，而且你可以在多项式时间内解决每个子问题。

通常有两种等效的方法来实现动态规划。切杆问题的解决方案就说明了这两种方法。

第一种方法为带备忘录 (memoization) 的自上而下，以自然的方式编写递归，但将每个子问题的结果保存下来（通常保存在数组或哈希表中）。现在，算法首先检查它是否已经解决了这个子问题。如果是，它将返回保存的值，从而节省在这一层的进一步计算。如果没有，它会以通常的方式计算这个值，但也会保存它。这里说递归被记忆化了：即它"记住"了之前计算的结果。

第二种方法为自底向上法，它通常依赖于对子问题“大小”的自然概念，这样解决任何特定子问题只依赖于解决“更小”的子问题。该方法的步骤是按照大小顺序解决子问题，先解决最小的子问题，然后在解决每个子问题时，将其解决方案存储起来。这样，当解决特定的子问题时，已经保存了所有更小子问题的解决方案。这种方法的优点是每个子问题只需要解决一次，并且当首次遇到它时，已经解决了其所有先决子问题。

两种方法有相似的渐进运行时间。除非在特殊情况下，自上而下的方法实际上并不递归检查所有可能的子问题。自下而上的方法通常具有更好的常数因子，因为它的过程调用开销更低。

MEMOIZED-CUT-ROD 和 MEMOIZED-CUT-ROD-AUX 演示了如何将自上而下的 CUT-ROD 程序备忘录化。主程序 MEMOIZED-CUT-ROD 初始化一个新的辅助数组 $r\[0:n]$，其值均为 $−\infty$(表示未知)。然后调用其辅助程序 MEMOIZED-CUT-ROD-AUX，这个过程实际上是 CUT-ROD 的记忆化版本。首先，在第 1 行检查所需值是否已知，如果已知，则第 2 行返回它。否则，第 3 到第 7 行按照通常的方式计算所需值 q，第 8 行将其保存在 $r\[n]$ 中，第 9 行返回它。

使用自底向上的方法 BOTTOM-UP-CUT-ROD 利用了子问题的自然顺序：如果 $i < j$，则大小为 i 的子问题比大小为 j 的子问题“更小”。因此，该过程按顺序解决大小为 $j = 0, 1, …, n$ 的子问题。

```
MEMOIZED-CUT-ROD(p, n)
let r[0 : n] be a new array  // 存储解数组 r
for i = 0 to n
	r[i] = −∞
return MEMOIZED-CUT-ROD-AUX(p, n, r)
```

```
MEMOIZED-CUT-ROD-AUX(p, n, r)
if r[n] ≥ 0  // 对于长度 n是否已有解
	return r[n]
if n == 0
	q = 0
else q = −∞
for i = 1 to n  
	q = max{q, p[i] + MEMOIZED-CUT-ROD-AUX(p, n − i, r)}
r[n] = q 
return q
```

```
BOTTOM-UP-CUT-ROD(p, n)
let r[0 : n] be a new array  // 存储解数组 r
r[0] = 0
for j = 1 to n  // 自底向上
	q = −∞
	for i = 1 to j  
		q = max{q, p[i] + r[j − i]}
	r[j] = q 
return r[n]
```

#### 子问题图

当思考一个动态规划问题时，你需要了解所涉及的一系列子问题，以及子问题之间的相互依赖关系。而子问题图恰恰体现了这些信息。

可以把子问题图看作是自上而下递归法的递归树的"缩小"或"折叠"版本，同一子问题的所有节点都凝聚成一个顶点，所有边都是从父节点指向子节点。

自底向上方法通过将子问题图的顶点按照一种特定顺序进行处理，确保在解决某个子问题之前，所有它依赖的子问题都已经解决。换句话说，这种顺序是子问题图的“逆拓扑排序”，或者说是子问题图的转置的拓扑排序。这保证了不会在某个子问题的解决之前尝试解决它所依赖的子问题。而自顶向下方法可以被视为对子问题图的一种“深度优先搜索”。

在动态规划中，每个子问题只需要解决一次，因此算法的运行时间取决于解决每个子问题所需的时间总和。通常情况下，计算子问题的解所需的时间与子问题图中相应顶点的度数（出边的数量）成正比。子问题的数量等于子问题图中顶点的数量。在这种常见情况下，动态规划的运行时间与顶点和边的数量呈线性关系。

#### 重建解决方案

上面两种方案都返回了切杆问题最优解的值，但是没有解本身：即切段的长度。

下面的程序 EXTENDED-BOTTOM-UP-CUT-ROD 不仅记录了杆长为 j 的最大收益 $r\_j$，还记录了切割第一部分的最优时的长度 $s\_j$。它与 BOTTOM-UP-CUT-ROD 几乎一样，只是在第 1 行创建了数组 s，并在第 8 行更新了 $s\[j]$，以保存在求解大小为 j 的子问题时切割的第一部分的最优长度 i。

程序 PRINT-CUT-ROD-SOLUTION 将价格数组 $p\[1:n]$ 和杆长 n 作为输入，调用 EXTENDED-BOTTOM-UP-CUT-ROD 计算出最优第一部分长度数组 $s\[1:n]$。然后打印出长度为 n 的杆的最佳切割中的完整长度列表。

```
EXTENDED-BOTTOM-UP-CUT-ROD(p, n)
let r[0:n] and s[1:n] be new arrays
r[0] = 0
for j = 1 to n
	q = -infty
	for i = 1 to j
		if q < p[i] + r[j-i]
			q = p[i] + r[j-i]
			s[j] = i
	r[j] = q
return r and s
```

```
PRINT-CUT-ROD-SOLUTION(p, n)
(r,s) = EXTENDED-BOTTOM-UP-CUT-ROD(p, n)
while n > 0
	print s[n]  // 杆的切割位置
	n = n - s[n]
```

### 矩阵链乘法

给定一个由 n 个矩阵组成的乘法序列 $〈A\_1, A\_2, \cdots, A\_n〉$，其中的矩阵不一定是方阵，目标是使用长方形矩阵相乘的标准算法（之前对于方阵相乘的算法不适用）计算 $A\_1A\_2\cdots A\_n$，且经量减少标量乘法的次数。

由于矩阵乘法满足结合律，因此相乘的次序（即如何加括号）将对分析乘法的代价有很大影响。首先考虑两个矩阵相乘的代价，下面给出了标准算法的步骤 RECTANGULAR-MATRIX-MULTIPLY，它对矩阵 $A=(a\_{ij}),B=(b\_{ij}),C=(c\_{ij})$ 计算 $C=C+A\cdot B$，其中 A 为 $p\times q$ 矩阵，B 是 $q\times r$ 矩阵，C 为 $p\times r$ 矩阵。

```
RECTANGULAR-MATRIX-MULTIPLY(A, B, C, p, q, r)
for i = 1 to p
	for j = 1 to r
		for k = 1 to q
			c_ij = c_ij + a_ik * b_kj
```

运行时间取决于标量乘法的次数，即 $pqr$，因此，这里认为矩阵乘法的代价就是标量乘法的次数。

为了说明矩阵乘法使用不同括号所产生的不同代价，考虑三个矩阵的链 $〈A\_1, A\_2, A\_3〉$ 问题。假设它们的维度分别为 $10\times 100,100\times 5,5\times 50$，按照 $((A\_1A\_2)A\_3)$ 相乘，标量乘法的次数为两次矩阵相乘的次数再相加，即 $10\cdot 100\cdot 5 + 10\cdot 5\cdot 50=7500$。如果按照 $(A\_1(A\_2A\_3))$ 相乘，就会变成 $100\cdot 5\cdot 50 + 10\cdot 100\cdot 50=75000$。可见第一种相乘的速度要比后者快十倍。

矩阵链乘法的问题描述如下：给定一个包含 n 个矩阵的矩阵链 $〈A\_1, A\_2, \cdots, A\_n〉$，其中第 i 个矩阵的维度为 $p\_{i-1}\times p\_i$ (其中 $i=1,,2,\ ...,\ n$)，需要找到一种最优的括号化方式，使得在计算矩阵链相乘时所需的标量乘法次数最少。输入为维度序列 $〈p\_0,p\_1,p\_2,...,p\_n〉$。

矩阵链乘法问题并不涉及矩阵的实际相乘，我们的目标只是确定一个代价最低的矩阵乘法顺序。通常情况下，虽然确定最佳相乘顺序的过程需要一定的计算时间，但这个时间投入往往会在之后实际执行矩阵相乘时得到回报。

#### 括号化方式的数量

要先认识到检查所有可能的括号化方式并不是高效的算法。用 $P(n)$ 表示长度 n 的矩阵链的可选括号化数，当 $n = 1$ 时只有一种括号化方式。当 $n\geq 2$ 时，一个完全括号化的矩阵乘积可以被看作是两个完全括号化的子矩阵乘积的乘积，而这两个子矩阵乘积的分割点可以在第 k 个和第 (k + 1) 个矩阵之间发生，其中 $k=1,2,...,n-1$。因此可以得到递归形式： $$P ( n )=\left\{\begin{array} {l l} {{1}} & {{\mathrm{i f} \: n=1 \;,}} \\ {{\sum_{k=1}^{n-1} P ( k ) P ( n-k )}} & {{\mathrm{i f} \: n \geq2 \;.}} \end{array} \right.\tag{14.6}$$

问题 12-4 中证明了具有相似递归的卡特兰数数列的解是 $\Omega(\frac{4^n}{n^{3/2\}})$，后面的一个练习会证明该递归的解为 $\Omega(2^n)$。即解的数量是 n 的指数级，穷举搜索的粗暴方法是一种糟糕的策略。

#### 使用动态规划

使用开头的四步顺序用动态规划确定如何对矩阵链进行最佳括号化：

1. Characterize the structure of an optimal solution.
2. Recursively define the value of an optimal solution.
3. Compute the value of an optimal solution.
4. Construct an optimal solution from computed information.

第一步：最佳括号化的结构

在第一步中，先找出最优子结构，然后利用它从子问题的最优解中构建出整体的最优解。用 $A\_{i:j}$ 表示 $A\_iA\_{i+1}\cdots A\_j$ 的结果矩阵(其中 $i\leq j$)，如果 $i\<j$，还要继续用括号分割，假设在 $A\_k$ 和 $A\_{k+1}$ 之间（其中 $i\leq k\<j$）。即对于某个 k，先计算 $A\_{i:k}$ 和 $A\_{k+1:j}$，然后相乘得到 $A\_{i:j}$。这时的代价为计算 $A\_{i:k}$ 和 $A\_{k+1:j}$ 的代价，再加上这两个相乘的代价。

该问题的最优子结构是这样的，假设对 $A\_iA\_{i+1}\cdots A\_j$ 最优括号化时，将乘积从 $A\_k$ 和 $A\_{k+1}$ 之间分割，那么，在这个最优的括号化方案中，对于子链的括号化方式 $A\_iA\_{i+1}\cdots A\_k$ 必须也是最优的。因为如果存在一种更低成本的方式来括号化子链 $A\_iA\_{i+1}\cdots A\_k$，那么我们可以在 $A\_iA\_{i+1}\cdots A\_j$ 最优括号化方案中用这种更低成本的括号化方式替代，从而产生另一种括号化方案，其成本低于最优方案，这与最优解的定义相矛盾。对子链 $A\_{k+1}A\_{k+2}\cdots A\_j$ 也是一样。

然后可以从子问题的最优解中构造出整体问题的最优解。对于矩阵链乘法问题的任何非平凡实例，解决方案都需要进行乘积的分割，并且任何最优解都包含了子问题实例的最优解。因此，为了构建矩阵链乘法问题实例的最优解，可以将问题分割成两个子问题（分别是 $A\_iA\_{i+1}\cdots A\_k$ 和 $A\_{k+1}A\_{k+2}\cdots A\_j$ 的最优括号化），找到两个子问题实例的最优解，然后将这些最优子问题解组合起来。为了确保考虑到了最优的分割点，必须考虑所有可能的分割点。

第二步：递归方案

子问题是在 $1\leq i\leq j\leq n$ 时，确定 $A\_iA\_{i+1}\cdots A\_j$ 的最小代价的括号化方式。记 $m\[i,j]$ 是计算 $A\_{i:j}$ 所需的最小标量乘法次数，对于总问题，计算 $A\_{1:n}$ 的最小代价则是 $m\[1,n]$。

然后按照下面方式递归定义 $m\[i,j]$。当 $i=j$ 时，不需要计算乘积，于是 $m\[i,i]=0$。当 $i\<j$ 时，利用第一步中最优解的结构，假设 $A\_iA\_{i+1}\cdots A\_j$ 的最优括号化方案是在 $A\_k$ 和 $A\_{k+1}$ 处分割（$i\leq k\<j$），于是 $m\[i,j]$ 就等于计算子乘积 $A\_{i:k}$ 的代价 $m\[i,k]$ 加上计算子乘积 $A\_{k+1:j}$ 的代价 $m\[k+1,j]$，再加上这两项乘积的代价 $p\_{i-1}p\_kp\_j$。于是，得到：

$$m [ i, j ]=m [ i, k ]+m [ k+1, j ]+p_{i-1}p_{k}p_{j}$$

由于不知道 k 的值，所以需要尝试所有可能的 k 的取值，即 $k=i,i+1,...,j-1$，一共 $j-i$ 个。因此括号化乘积 $A\_iA\_{i+1}\cdots A\_j$ 的最小代价递推定义为：

$$m[i,j]=\begin{cases} 0 & if\ i=j\\ \min\{m[i,k]+m[k+1,j]+p_{i-1}p_{k}p_{j}\} & if\ i<j \end{cases}\tag{14.7}$$

$m\[i,j]$ 给了子问题最优解的代价，但是并未给出如何构建最优解，所以还需要定义 $s\[i,j]$ 为乘积 $A\_iA\_{i+1}\cdots A\_j$ 最优括号化的分割点 k。

第三步：计算最优方案代价

如果按照式 14.7 写出递归算法，并不比检查乘积的每一种括号化的粗暴方法好多少，仍然是指数级运行时间。

幸运的是，并没有那么多不同的子问题：对于满足 $1\leq i ≤ j ≤ n$ 的 i 和 j 的每种选择，只有一个子问题，即一共有 $\binom{n}{2}+n=\Theta(n^2)$ 子问题。递归算法可能会在递归树的不同分支中多次遇到每个子问题。**子问题重叠**的特性是动态规划适用的第二个标志（第一个标志是**最优子结构**）。

如下程序 MATRIX-CHAIN-ORDER，是采用自下而上的表格法计算最优代价的方案。输入是一个矩阵维度序列 $p = 〈p\_0，p\_1，...，p\_n〉$ 以及 n，即对于 $i = 1，2，...，n$，矩阵 $A\_i$ 的维度为 $p\_{i-1}\times p\_i$。程序使用一个辅助表 $m\[1 : n, 1 : n]$ 来存储 $m\[i, j]$ 的成本，另一个辅助表 $s\[1 : n - 1, 2 : n]$ 则记录了哪个索引 k 在计算 $m\[i, j]$ 时达到了最优成本。表 s 将有助于构建最优解。

```
MATRIX-CHAIN-ORDER(p, n)
let m[1:n, 1:n] and s[1:n-1, 2:n] be new tables
for i = 1 to n  // 长度为 1 的链
	m[i,i]=0
for l = 2 to n   // l 为链长
	for i = 1 to n-l+1   // 链起始点 Ai
		j = i + l - 1    // 链结尾点 Aj
		m[i,j] = infty
		for k = i to j-1  // 尝试 A_1:k A_k+1:j
			q = m[i,k] + m[k+1, j] + p_i-1 p_k p_j
			if q < m[i,j]
				m[i,j] = q    // 存储代价
				s[i,j] = k    // 存储索引
return m and s
```

可以从式 14.7 中看出，要计算 $A\_{i:j}$ 需要先计算 $A\_{i:k}$ 和 $A\_{k+1:j}$，而这两个子链的长度明显要比原链长要小，因此算法的顺序应为在表 m 中从较短的矩阵链填充到较长的矩阵链。

可以从该程序的时间复杂度为 $\Omega(n^3)$，空间复杂度为 $\Theta(n^2)$。

第四步：构造最优解

表格 $s\[1:n-1, 2:n]$ 包含了如何括号化的信息，每个条目 $s\[i, j]$ 都记录了一个 k 值，即 $A\_iA\_{i+1}\cdots Aj$ 的最优括号化乘积在 $A\_k$ 和 $A\_{k+1}$ 之间分割。于是计算 $A\_{1:n}$ 的最优方法为 $A\_{1:s\[1:n]}A\_{s\[1,n]+1 : n}$，然后使用递归，$s\[1, s\[1, n]]$ 是计算 $A\_{1:s\[1,n]}$ 时的上一次矩阵乘法分割点，而 $s\[s\[1,n] + 1, n]$ 是计算 $A\_{s\[1,n]+1:n}$ 时的上一次矩阵乘法分割点。根据 MATRIX-CHAIN- ORDER 计算出的 s 表以及索引 i 和 j，递归过程 PRINT-OPTIMAL-PARENS 将打印出矩阵链乘积 $A\_iA\_{i+1}\cdots A\_j$ 的最优括号化方案。

```
PRINT-OPTIMAL-PARENS(s, i, j)
if i == j
	print "A"_i
else print "("
	PRINT-OPTIMAL-PARENS(s, i, s[i, j])
	PRINT-OPTIMAL-PARENS(s, s[i, j] + 1, j)
	print ")"
```

### 动态规划的元素

使用动态规划的两个要素：最优子结构和重叠子问题。

#### 最优子结构

用动态规划求解优化问题的第一步是确定最优解的结构特征。如果问题的最优解中包含了子问题的最优解，那么问题就具有最优子结构。

在发现最优子结构的过程中遵循如下步骤：

1. 提出问题：在解决一个问题时，我们需要做出某个选择。例如，对于一个切割棒的问题，可以选择在某个位置进行初始切割；对于矩阵链乘法问题，可以选择在哪个索引处分割矩阵链。做出这个选择后，会产生一个或多个子问题。
2. 假设给定最优选择：假设我们已经知道了能够导致最优解的选择。此时我们不关心如何找到这个选择，只是假设我们已经知道了它。
3. 确定子问题：给定这个选择之后，我们需要确定由此产生的子问题是什么，并且如何最好地描述这些子问题的空间。
4. 证明子问题的最优性：使用“剪贴法”证明子问题的解也必须是最优的。具体来说，通过假设每个子问题的解不是最优的，然后推导出一个矛盾。即，通过“剪掉”非最优的子问题解，并“贴上”最优的子问题解，证明可以得到一个更好的原问题解，从而与我们之前的假设（我们已经有一个最优解）相矛盾。如果一个最优解产生多个子问题，它们通常如此相似，以至于我们可以很容易地将一个子问题的“剪贴”论证修改应用到其他子问题上。

要描述子问题空间的特征，一个好的经验法则是尽量保持空间的简单性，然后在必要时进行扩展。

贪心算法与动态规划相似，适用的问题也有最优子结构。不同在于，贪心算法不是先找到子问题的最优解，然后再做出最佳的选择，而是先做出一个“贪心”的选择，即当时看起来最好的选择，然后解决由此产生的子问题，而不必费心解决所有可能相关的更小的子问题。

注意，一定要会识别问题是否具有最优子结构。考虑以下两个问题，其输入包括一个有向图 $G = (V, E)$ 和顶点 $u, v ∈ V$：

* **无权最短路径**：找出一条从 u 到 v 的最少边的路径，这条路径必须是简单路径。
* **无权最长简单路径**：找出一条从 u 到 v 的由最多条边组成的简单路径。（如果不要求路径必须简单，问题就无法解释，因为重复遍历循环会产生任意多边的路径）。

无权最短路径问题具有最优子结构。假设 $u\neq v$，那么从 u 到 v 的任何路径 p 都必须包含一个中间顶点，即 w（注意，w 可以是 u 或 v）。就可以将路径 $u \stackrel{p} {\rightarrow} v$ 分为两个子路径 $u \stackrel{p\_{1\}} {\rightarrow} w \stackrel{p\_{2\}} {\rightarrow} v$。路径 p 中的边数量等于 p1 边数加上 p2 的边数，则如果 p 是一条从 u 到 v 的最优（即最短）路径，那么 p1 一定是一条从 u 到 w 的最短路径。可以用剪贴法证明：如果 u 到 v 存在一条比 p1 更短的路径 $p\_1'$，那么可以剪掉 p1，将 $p\_1'$ 粘贴进去，产生新的比 p 还短的路径 $u \stackrel{p\_{1}'} {\rightarrow} w \stackrel{p\_{2\}} {\rightarrow} v$，与 p 路径的最优性矛盾。同理 P2 必须是 w 到 v 的最短路径。因此，要找出一条从 u 到 v 的最短路径，需要考虑所有中间顶点 w，找出一条从 u 到 w 的最短路径和一条从 w 到 v 的最短路径，并选择一个能产生最短路径的中间顶点 w。图算法中的 Floyd 算法利用对最优子结构这种观察的变体，在有向加权图中的每对顶点之间找出一条最短路径。

但是，对于无权最长简单路径问题却不存在这样的最优子结构，即如果将一条最长简单路径 $u \stackrel{p} {\rightarrow} v$ 分为两条子路径 $u \stackrel{p\_{1\}} {\rightarrow} w \stackrel{p\_{2\}} {\rightarrow} v$，则 p1 不一定是 u 到 w 的最长简单路径，p2 不一定是 w 到 v 的最长简单路径。

而且，对于最长简单路径问题，不仅没有最优子结构，而且还不一定能从子问题的解中组合出“合法”的解。事实上，寻找无权最长简单路径的问题似乎并不存在任何最优子结构。在这个问题上，还没有找到有效的动态规划算法。而且这个问题是 NP-complete 的，这意味着我们不可能在多项式时间内找到解决这个问题的方法。

虽然求解最长简单路径和最短路径的问题都要用到两个子问题，但是求解最长简单路径的子问题并不是**独立**的，而求解最短路径的子问题却是独立的。独立的子问题是指一个子问题的解决不会影响同一问题的另一个子问题的解决。在图 14.6 的例子中，从 q 到 t 寻找最长简单路径的问题有两个子问题：从 q 到 r 和从 r 到 t 寻找最长简单路径。对于第一个子问题，我们选择了路径 q → s → t → r，其中使用了顶点 s 和 t。如果顶点 t 不能出现在第二个问题的解中，那么就无法解决这个问题，因为 t 必须出现在解的路径上，而它并不是子问题解“拼接”在一起的顶点（这个顶点就是 r）。因为顶点 s 和 t 出现在一个子问题的解中，所以它们不能出现在另一个子问题的解中。但是，其中一个顶点必须出现在另一个子问题的解中，而最优解需要这两个顶点都出现。因此，我们说这些子问题不是独立的。从另一个角度看，在解决一个子问题时使用资源（这些资源就是顶点）会导致另一个子问题无法使用这些资源。

而最短路径的子问题不会共享资源。如果顶点 w 位于从 u 到 v 的最短路径 p 上，那么可以拼接最短路径 $u \stackrel{p\_1} {\rightarrow} w$ 和最短路径 $w \stackrel{p\_2} {\rightarrow} v$ 产生 u 到 v 的最短路径，则_除了点 w，没有其他顶点可以同时出现在路径 p1 和 p2 上_。证明：假设有顶点 $x\neq w$ 同时出现在路径 p1 和 p2 上，就可以将 p1 分为 $u \stackrel{p\_{ux\}} {\rightarrow} x \rightarrow w$，将 p2 分为 $w \rightarrow x \stackrel{p\_{xv\}} {\rightarrow} v$。根据问题的最优子结构，路径 p 的边数与 p1 和 p2 的边数和相同，假设 p 有 e 条边。现在可以构建一条从 u 到 v 的路径 $p'=u\stackrel{p\_{ux\}} {\rightarrow}x\stackrel{p\_{xv\}} {\rightarrow}v$，这里去掉了从 x 到 w 和从 w 到 x 的路径（每条路径至少有一条边），那么路径 p′ 最多包含 e-2 条边，这与 p 是最短路径的假设相矛盾。由此可得最短路径得子问题是独立的。

#### 重叠子问题

可以使用动态规划解决的优化问题的第二个特征是：子问题的空间必须“足够小”，即问题的递归算法可以反复解决相同的子问题，而不是总是产生新的子问题。通常情况下，不相同子问题的总数是输入大小的多项式数量级。当递归算法重复处理同一问题时，我们就说优化问题有**重叠的子问题**。相反，适合用分治法的问题通常会在递归的每一步产生新的问题。动态规划算法通常利用重叠子问题的特性，对每个子问题求解一次，然后将解法存储在表格中，需要时再进行查询，每次查询耗时是不变的。

回到之前的矩阵链乘法问题，观察到 MATRIX-CHAIN-ORDER 在解决较高行的子问题时，会重复查找较低行的子问题的解决方案。要了解其原理，请看下面的无穷递归过程 RECURSIVE-MATRIX-CHAIN，它通过计算矩阵链乘积 $A\_{i:j} = A\_iA\_{i+1} ⋯ A\_j$ 所需的最少标量乘法次数确定 $m\[i，j]$，这个程序是直接由式 14.7 得到的。可以通过绘制其生成的递归树看到许多数值对重复了好几次。

```
RECURSIVE-MATRIX-CHAIN(p, i, j)
if i == j
	return 0
m[i, j] = ∞
for k = i to j - 1
	q = RECURSIVE-MATRIX-CHAIN(p, i, k) 
		+ RECURSIVE-MATRIX-CHAIN(p, k + 1, j)
		+ p_{i-1}*p_k*p_j
	if q < m[i, j]
		m[i, j] = q
return m[i, j] 
```

用该递归过程计算 $m\[i, j]$ 所需的时间至少是 n 的指数级。使用 $T(n)$ 表示该过程计算 n 个矩阵链的最优括号化所需时间，可以得到递归式：

$$T ( n ) \geq\left\{\begin{array} {l l} {1} & {{\mathrm{if}} \, n=1} \\ {1+\sum_{k=1}^{n-1} ( T ( k )+T ( n-k )+1 )} & {{\mathrm{if}} \, n > 1} \end{array} \right.$$

注意到对于 $i = 1，2，...，n - 1$，每一项 $T(i)$ 作为 $T(k)$ 出现一次，作为 $T(n - k)$ 出现一次，并将求和中的 n-1 个 1 和前面的 1 合并，于是可以将递推关系重写为：

$$T ( n ) \geq 2\sum_{i=1}^{n-1} T ( i )+n \tag{14.8}$$

使用代换法证明 $T(n)=\Omega(2^n)$。猜测对于所有 $n\geq 1$ 都有 $T(n)\geq 2^{n-1}$，首先 $n=1$ 时，不用求和，得到 $T(1)\geq 1=2^0$，使用归纳法，对于 $n\geq 2$ 有：

$$\begin{aligned}T(n)&\geq 2\sum_{i=1}^{n-1}2^{i-1}+n \\ &=2\sum_{j=0}^{n-2}2^j+n \\ &=2(2^{n-1}-1)+n \\ &=2^n-2+n \\ &\geq 2^{n-1}\end{aligned}$$

因此，调用 `RECURSIVE-MATRIX-CHAIN(p, 1, n)` 所执行的总工作量至少是 n 的指数级。

将这种自上而下的递归算法（无记忆化）与自下而上的动态规划算法进行比较可以发现，后者更加高效，因为其利用了重叠子问题的特性。当一个问题的自然递归解的递归树中重复出现相同的子问题，而互不相同子问题的总数又很少时，动态规划就能提高效率。

#### 重建最优解

在实际操作中，通常还会在一个单独的表格中存储你在每个子问题中所做的选择，这样就不必再用 cost 表重建这些信息了。如 MATRIX-CHAIN-ORDER 过程中的表 $s\[i.j]$ 就是这样的用处。

#### 记忆化

带记忆的递归算法会在一个表中为每个子问题的解维护一个条目，表中的每个条目最初都包含一个特殊的值，这个值用来指示该条目还没有被填充。当递归算法展开并第一次遇到某个子问题时，算法会计算这个子问题的解，将这个解存储在表中的相应位置，当后续再次遇到这个子问题时，算法只需要查找表中已存储的值，并将其返回，而不需要再次计算。这种方法假定你已经知道所有可能的子问题参数集合，并且你已经建立了表中位置和子问题之间的关系。另一种更通用的方法是使用哈希将子问题参数作为键来进行记忆化。这种方式适用于参数范围广泛或无法预先确定的情况。

下面程序 MEMOIZED-MATRIX-CHAIN 是程序 RECURSIVE-MATRIX-CHAIN 的带记忆化版本。请注意它与杆切割问题的自上而下记忆化方法的相似之处。

```
MEMOIZED-MATRIX-CHAIN(p, n)
let m[1 : n, 1 : n] be a new table
for i = 1 to n
	for j = i to n
		m[i, j] = ∞
return LOOKUP-CHAIN(m, p, 1, n)

LOOKUP-CHAIN(m, p, i, j)
if m[i, j] < ∞
	return m[i, j]
if i == j
	m[i, j] = 0
else for k = i to j - 1
	q = LOOKUP-CHAIN(m, p, i, k)
		+ LOOKUP-CHAIN(m, p, k + 1, j) + p_{i−1}*p_k*p_j
	if q < m[i, j]
		m[i, j] = q
return m[i, j]
```

可以看到，该过程与自下而上的方法都是维护了一个表 $m\[i, j]$ 存储计算矩阵 $A\_{i:j}$ 所需的最小标量乘法次数。`LOOKUP-CHAIN(m, p, i, j)` 总是返回 $m\[i, j]$的值，但它只在 LOOKUP-CHAIN 第一次调用 i 和 j 的特定值时才会计算它。

与自下而上的程序 MATRIX-CHAIN-ORDER 一样，记忆化程序 MEMOIZED-MATRIX-CHAIN 运行时间为 $O(n^3)$。

在实际应用中，如果所有子问题至少都需要解决一次，通常情况下，自底向上的动态规划算法比对应的自顶向下记忆化算法性能更好，常数因子上更有优势，因为自底向上的算法没有递归的开销，并且维护表的开销较小。此外，对于某些问题，你可以利用动态规划算法中表访问的规律性，进一步减少时间或空间需求，如在自底向上的动态规划中，有些问题具有规律的表访问模式，例如从左到右、从上到下填充表。通过识别这些规律，可以优化算法，减少不必要的计算或内存使用。另一方面，在某些情况下，子问题空间中的某些子问题可能根本不需要解决，（并不是所有问题的所有子问题都需要解决，特别是在问题的某些条件下或输入特性下，部分子问题可能根本不会被使用）在这种情况下，带记忆化的解决方案有一个优势，就是只解决那些确实需要解决的子问题。

### 最长公共子序列

### 最优二叉搜索树

给定一个包含 n 个不同键值的序列 $K=\<k\_1, k\_2, ..., k\_n>$，其中 $k\_1\<k\_2<...\<k\_n$，建立一个包含这些值的二叉搜索树。对于每个键值 $k\_i$，其被搜索的概率是 $p\_i$。由于可能出现不在 K 中的搜索值，所以用 n+1 个“虚拟”键值 $d\_0,d\_1,d\_2,...,d\_n$ 表示。$d\_0$ 表示所有小于 $k\_1$ 的值，$d\_n$ 代表所有大于 $k\_n$ 的值，而对于 $i = 1，2，...，n - 1$，键值 $d\_i$ 代表所有介于 $k\_i$ 和 $k\_{i+1}$ 之间的值。对于每个键值 $d\_i$，有相应的搜索概率 $q\_i$，每个键值 $k\_i$ 都是一个内部节点，每个假键值 $d\_i$ 都是一个叶子节点。由于每次搜索要么成功（找到某个键值 $k\_i$），要么不成功（找到某个假的 $d\_i$），所以有:

$$\sum_{i=1}^{n} p_{i}+\sum_{i=0}^{n} q_{i}=1\tag{14.10}$$

在给定每个键值和虚拟键值的搜索概率时，就可以确定二叉搜索树 T 中搜索的期望成本。假设搜索的实际成本为检查的节点数量，即搜索找到的节点的深度加 1。则 T 中进行一次搜索的期望成本为：

$$\begin{aligned}E[search\ cost\ in\ T]&=\sum_{i=1}^{n} ( \mathrm{d e p t h}_{T} ( k_{i} )+1 ) \cdot p_{i}+\sum_{i=0}^{n} ( \mathrm{d e p t h}_{T} ( d_{i} )+1 ) \cdot q_{i}\\&=1+\sum_{i=1}^{n} \mathrm{d e p t h}_{T} ( k_{i} ) \cdot p_{i}+\sum_{i=0}^{n} \mathrm{d e p t h}_{T} ( d_{i} ) \cdot q_{i}\end{aligned}\tag{14.11}$$

其中 $depth\_{T}$ 表示树T中节点的深度。

对于一组给定概率，目标是构建一个使得期望搜索成本最低的二叉搜索树，称这样的树为**最优二叉搜索树**。最优二叉搜索树不一定是总高度最小的树，也不总是在根处的键值具有最大概率。

与矩阵链乘法类似，穷尽检查所有可能的解不是高效的算法。问题 12-4 证明了有 n 个节点的二叉树的数量是 $\Omega(4^n/n^{3/2})$，即需要检查指数级的二叉树。

#### Step 1: 最优二叉搜索树的结构

为了描述最优二叉搜索树的最优子结构，首先观察子树。考虑一个二叉搜索树的任意子树，它必须包含连续范围 $k\_i, ...,k\_j$ 中的键值，其中 $1≤i≤j≤n$。此外，包含键 $k\_i, ..., k\_j$ 的子树也必须有虚拟键 $d\_{i−1}, ..., d\_j$ 作为其叶节点。

于是最优子结构为：如果最优二叉搜索树 T 有一个包含键 $k\_i, ..., k\_j$ 的子树 T′，那么对于包含键 $k\_i, ..., k\_j$ 和虚拟键 $d\_{i-1}, ..., d\_j$ 的子问题来说，这个子树 T′ 也一定是最优的。这里可以使用剪贴法证明，如果有一棵期望成本低于 T′ 的子树 T″，那么从 T 中剪切出 T′，再粘贴进 T″，就会得到一棵期望成本低于 T 的二叉搜索树，从而与 T 的最优性相矛盾。

有了最优子结构，下面介绍如何从子问题的最优解中构建问题的最优解。给定键 $k\_i, ...,k\_j$，其中一个键如 $k\_r$(i ≤ r ≤ j)，是包含这些键的最优子树的根。根 $k\_r$ 的左侧子树包含键 $k\_i, ...,k\_{r-1}$（以及虚拟键 $d\_{i-1}, ...,d\_{r-1}$），右侧子树包含键 $k\_{r+1}, ...,k\_j$（以及虚拟键 $d\_r, ...,d\_j$）。只要检查所有候选根 $k\_r$(i ≤ r ≤ j)，并确定所有包含 $k\_i, ..., k\_{r-1}$ 和包含 $k\_{r+1}, ...,k\_j$ 的最优二叉搜索树，就能保证找到一棵最优二叉搜索树。

需要注意“空”子树的情况，假设一个包含键 $k\_i, ...,k\_j$ 的子树，你选择了 $k\_i$ 作为该子树的根。这时 $k\_i$ 的左子树应该包含键 $k\_i, ..., k\_{i-1}$，即一个键都没有。注意子树还有虚拟键，虽然这里的左子树没有实键，但有一个虚拟键 $k\_{i-1}$。对称地，如果选择 $k\_j$ 作为根，那么 $k\_j$ 的右侧子树就不包含实键，但包含虚拟键 $d\_j$。

#### Step 2: 递归方案

为了递归定义最优解的值，子问题域是找到一棵包含键 $k\_i, ...,k\_j$ 的最优二叉搜索树，其中 $i ≥ 1$, $j ≤ n$, $j ≥ i - 1$（当 $j = i - 1$ 时，只有虚拟键 $d\_{i-1}$，而没有实键）。目标是计算 $e\[1, n]$，即搜索所有实键和虚拟键的最优二叉搜索树的期望成本。

当 $j = i - 1$，子问题只有虚拟键 $d\_{i-1}$，期望搜索成本为 $e\[i,i-1]=q\_{i-1}$。

当 $j ≥ i$ 时，需要从 $k\_i, ...,k\_j$ 中选择一个根 $k\_r$，然后以键 $k\_i、...、k\_{r-1}$ 为左子树，以键 $k\_{r+1}, ...,k\_j$ 为右子树，构建出最优二叉搜索树。当一个子树成为一个节点的子树时，即子树所有节点深度加 1 时，根据公式(14.11)，它的期望成本会增加子树中所有节点的概率之和。对于键值为 $k\_i, ...,k\_j$ 的子树，将该概率总和表示为：

$$w ( i, j )=\sum_{l=i}^{j} p_{l}+\sum_{l=i-1}^{j} q_{l}\tag{14.12}$$

于是，如果 $k\_r$ 是包含键 $k\_i, ..., k\_j$ 的最优子树的根，就有：

$$e [ i, j ]=p_{r}+( e [ i, r-1 ]+w ( i, r-1 ) )+( e [ r+1, j ]+w ( r+1, j ) )$$

注意到：$w ( i, j )=w ( i, r-1 )+p\_{\it r}+w ( r+1, j )$，可简化为：

$$e [ i, j ]=e [ i, r-1 ]+e [ r+1, j ]+w ( i, j )\tag{14.13}$$

递归公式(14.13)假设你知道哪个节点 $k\_r$ 是根节点。当然，我们会选择期望搜索成本最低的节点作为根节点，这就是递推公式的最终形式：

$$e [ i, j ]=\left\{\begin{aligned} {} & {{} q_{i-1}} & {} & {{} \mathrm{i f ~ j=i-1 ~} \;,} \\ {} & {{} \operatorname* {m i n} \left\{e [ i, r-1 ]+e [ r+1, j ]+w ( i, j ) : i \leq r \leq j \right\}} & {} & {{} \mathrm{i f ~ i \leq~ j ~} \;.} \\ \end{aligned} \right.\tag{14.14}$$

$e\[i,j]$ 值给出了最优二叉搜索树中的期望搜索成本，为了记录该树的结构，定义 $root\[i, j]$，其中 $1 ≤ i ≤ j ≤ n$。

#### Step 3: 计算最优二叉搜索树的期望搜索成本

可以发现，对最优二叉搜索树和矩阵链乘法的描述有一些相似之处。对于这两个问题域，子问题都由连续的索引子范围组成。直接实现递归式(14.14)就和直接递归的矩阵链乘法一样低效。可以用一个表 $e\[1:n+1,0:n]$ 来存储 $e\[i,j]$，其中第一个表索引需要到 n+1，而不是 n，是因为要使子树只包含虚拟键 $d\_n$，就需要计算并存储 $e\[n+1，n]$。第二个索引需要从 0 开始，是因为要使子树只包含虚拟键 $d\_0$，就需要计算并存储 $e\[1，0]$。并且这个表中只有 $j\geq i-1$ 时才有值。表 $root\[i, j]$ 记录了包含键 $k\_i, ...,k\_j$ 的子树的根，并且只使用了 $1 ≤ i ≤ j ≤ n$ 的项。

每次计算 $e\[i, j]$ 时，都要重新计算 $w(i,j)$ 的值，这将进行 $\Theta(j - i)$ 次加法，不如将这些值存储在一个表 $w\[1 : n + 1, 0 : n]$ 中。对于简单情况，$w\[i,i-1]=q\_{i-1}$，其中 $1\leq i\leq n+1$。对于 $j\geq i$ 的情况：

$$w [ i, j ]=w [ i, j-1 ]+p_{j}+q_{j}\tag{14.15}$$

下面的 OPTIMAL-BST 程序将概率 $p\_1, ...,p\_n$ 和 $q\_0, ...,q\_n$ 以及大小 n 作为输入，并返回表 e 和 root。

```
OPTIMAL-BST(p, q, n)
let e[1 : n + 1, 0 : n], w[1 : n + 1, 0 : n], and root[1 : n, 1 : n] be new tables
for i = 1 to n+1            //base cases
	e[i, i-1] = q_{i-1}	  //equation(14.14)
	w[i, i-1] = q_{i-1}
for l = 1 to n
	for i = 1 to n+1
		j = i + l - 1
		e[i, j] = ∞
		w[i, j] = w[i, j-1] + p_j + q_j   //equation(14.15)
		for r = i to j                    //try all possible roots r
			t = e[i, r-1] + e[r+1, j] + w[i, j] //equation(14.14)
			if(t < e[i, j])               //new minimum?
				e[i, j] = t
				root[i, j] = r
return e and root
```

OPTIMAL-BST 算法的时间复杂度与 MATRIX-CHAIN-ORDER 算法的时间复杂度相似，都是$\Theta(n³)$，因为它们都有三重嵌套循环，每个循环变量的范围最多是 n，因此运行时间的上下界都是 $O(n³)$ 和 $Ω(n³)$。

对于所有 $1 ≤ i < j ≤ n$ 的最优子树，总是存在根，使得 $root\[i, j - 1] ≤ root\[i, j] ≤ root\[i + 1, j]$。利用这一事实可以修改 OPTIMAL-BST 程序，使其在 $Θ(n^2)$ 时间内运行

## 摊还分析

摊还分析是一种计算数据结构操作时间的方法。它通过对一系列数据结构操作的时间需求进行平均，考虑所有操作的总体影响。通过摊销分析，可以发现，如果对一系列操作进行平均，则即使序列中存在单个操作可能很昂贵，但是操作的平均成本很小。摊销分析和平均情况分析不同在于摊销分析不涉及概率，摊销分析保证了在最坏情况下每个操作的平均性能表现，它更关注最糟糕的情况，而非基于概率的平均情况。

### 聚合分析

在聚合分析中，最坏情况下所有的n个操作共需要$T(n)$ 的运行时间，因此每次操作的平均成本或摊销成本为$T(n)/n$。即使操作序列中存在不同类型的操作，也会用这种平均成本来估算每种操作的平均代价。而另外两种方法可能会针对不同类型的操作分配不同的摊销成本。

#### 栈操作

两个最基本的栈操作：`PUSH(S, x)`和`POP(S)`，两种操作都花费$O(1)$。

由于每个操作的运行时间为$O(1)$，因此我们将每个操作的成本视为 1。因此，n 个 PUSH 和 POP 操作序列的总成本为 n，n 个操作的实际运行时间为$θ(n)$。

定义新的栈操作`MULTIPOP(S, k)`(假设k大于0)，它会删除堆栈 S 顶部的 k 个对象，如果堆栈包含的对象少于 k 个，则弹出整个堆栈。

```
MULTIPOP(S, k)

while not STACK-EMPTY(S) and k > 0
	POP(k)
	k = k - 1
```

假设一个栈有s个元素，则该操作的总代价为$\min {s, k}$，实际运行时间是该代价的线性函数。

分析在一个初始为空的栈上进行一系列n个PUSH、POP和MULTIPOP操作。由于堆栈的大小最多为n，则最坏情况下MULTIPOP操作的成本为$O(n)$，任何操作的最坏情况情况时间复杂度都是$O(n)$，因此，对于包含最多 n 个花费$O(n)$ 的 MULTIPOP 操作的 n 次操作序列来说，其总时间复杂度为$O(n^2)$。尽管该分析是正确的，但是并非每个操作都在最坏情况下花费了最大的代价，这使得 O(n^2) 的结果有些过于悲观，实际情况可能会更好。

单个 MULTIPOP 可能很昂贵，但聚合分析表明，在最初为空的堆栈上执行 n 个 PUSH、POP 和 MULTIPOP 操作的任何序列的成本上限均为$O(n)$。对 n 个操作进行平均，得出每个操作的平均成本为$O(n)/n = O(1)$。聚合分析将每项操作的摊还成本指定为平均成本。因此，在此示例中，所有三个堆栈操作的摊还成本均为$O(1)$。

回顾一下：虽然堆栈操作的平均成本和运行时间是$O(1)$，但分析并不依赖于概率推理。相反，分析在 n 个操作序列上得出了$O(n)$ 的最坏情况界限。将此总成本除以 n 得出每次操作的平均成本（即摊还成本）为$O(1)$。

#### 二进制计数器

考虑一个从0开始向上计数的k位二进制计数器，使用数组 $A\[0:k-1]$ 表示该k位计数器。存储在计数器中的二进制数 x 的最低位在 $A\[0]$ 中，最高位在 $A\[k − 1]$ 中，所以 $x = \sum\_{i=0}^{k-1} A\[i] \cdot 2^i$。初始情况 $x = 0$，即数组A中元素都为0。要将计数器加1(模 $2^k$)，调用INCREMENT算法：

```
INCREMENT(A,k)

i = 0
while i < k and A[i] == 1
	A[i] = 0
	i = i + 1
if i < k
	A[i] = 1
```

该算法的while循环的每次迭代都会使i加1，如果A\[i] = 1，则算法加一操作使该位置翻转为0，产生进位，否则循环结束。此时如果 $i < k$，则A\[i]必为0，因此需将A\[i]翻转为1，如果此时 $i = k$，说明所有的k位都已被翻转为0，计数器溢出。每个 INCREMENT 操作的成本与翻转的位数成线性关系。

在最坏的情况下，单次执行INCREMENT需要 $\theta(k)$ 时间，因此一个宽松的界限是：初始为零的计数器上的一系列 n INCREMENT 操作共需要 O(nk) 时间。但是一般情况下，对于 $i=0, 1, ..., k-1$，A\[i]在初始为0的计数器上进行n个INCREMENT操作翻转 $\lfloor\frac{n}{2^i}\rfloor$ 次。所以总的翻转次数为： $$\sum_{i=0}^{k-1}\lfloor\frac{n}{2^i}\rfloor < n\sum_{i=0}^{\infty}\frac{1}{2^i} = 2n$$

因此，最坏情况下，初始为零的计数器进行n次INCREMENT操作需要 $O(n)$ 时间，每个操作的平均成本，以及每个操作的摊余成本，为 $O(n)/n = O(1)$。

### 记账法

为不同的操作分配不同的费用，分配的费用可能高于或低于实际成本，这些收取的操作费用就是**摊销成本(amortized cost)**，当操作的摊销成本超过其实际成本时，可以将差额作为\*\*信用(credit)\*\*分配给其他对象，信用可以帮助支付后续那些摊销成本低于实际成本的操作的费用。因此，可以将一项操作的摊销成本分为实际成本和存入或正好用完的信用。

该方法与聚合分析不同，聚合分析中所有操作都具有相同的摊销成本，而记账法中不同操作可能有不同的摊销成本。

应当仔细选择操作的摊销成本。如果要使用摊销成本来表明在最坏情况下每个操作的平均成本很小，则必须确保操作序列的总摊销成本提供该序列的总实际成本的上限，而且该上限要能够适用于所有操作序列。将第i次操作的实际成本表示为 $c\_i$，摊销成本表示为 $\hat{c}_i$，所以对n个操作的所有序列需要有：$\sum_{i=1}^{n}\hat{c}_i\geq\sum_{i=1}^{n}c\_i$。

存储在该数据结构中的所有信用为所有的摊销成本与所有实际成本之差，即 $\sum\_{i=1}^{n}\hat{c}_i - \sum_{i=1}^{n}c\_i$，
