# 滑动窗口

## 定长滑动窗口

总结成三步：入 - 更新 - 出
- `入`：下标为 `i` 的元素进入窗口，更新相关统计量
- `更新`：更新答案，`一般是更新最大值/最小值`
- `出`：下标为 `i−k+1` 的元素离开窗口，更新相关统计量

以上三步适用于所有定长滑窗题目

```python
class Solution:
    def maxVowels(self, s: str, k: int) -> int:
        res = count = 0
        for i, c in enumerate(s):
            # 1. 进入窗口
            if c in "aeiou":
                count += 1
            if i < k - 1:  # 窗口大小不足 k
                continue
            # 2. 更新答案
            res = max(res, count)
            # 3. 离开窗口
            if s[i - k + 1] in "aeiou":
                count -= 1
        return res
```

## 基于“滑动窗口 + 字符频率 + valid 计数”的模板

适用场景：
- 判断是否包含某些字符/异位词
- 找到所有异位词
- 最小窗口包含所有字符（如 LeetCode 76）
- 子串满足特定频率要求

通用模板代码（Python）：

```python
from collections import defaultdict

def sliding_window_template(s: str, t: str) -> str:
    need = defaultdict(int)
    window = defaultdict(int)
    for c in t:
        need[c] += 1

    left = right = 0
    valid = 0  # 记录当前窗口满足 need 条件的字符种类数

    # 根据题意定义结果变量，例如最小长度、起始位置、答案列表等
    res = []

    while right < len(s):
        # c 是将要加入窗口的字符
        c = s[right]
        right += 1

        # 更新窗口内数据
        if c in need:
            window[c] += 1
            if window[c] == need[c]:
                valid += 1

        # 判断左侧窗口是否需要收缩
        while right - left >= len(t):  # 对于固定长度匹配，如异位词
        # while valid == len(need):     # 对于所有 need 被满足，如最小覆盖子串
            # 如果满足条件，更新结果
            if valid == len(need):
                res.append(left)  # 对于找异位词，记录当前窗口起点
                # res = s[left:right]    # 对于最小覆盖子串，更新最小结果

            # d 是将要从窗口移除的字符
            d = s[left]
            left += 1

            # 更新窗口内数据
            if d in need:
                if window[d] == need[d]:
                    valid -= 1
                window[d] -= 1

    return res  # 或返回符合条件的子串、长度等
```

使用说明

| 场景 | 如何修改模板 |
| --- | --- |
| 找到所有异位词（如：LeetCode 438）| 保持 `right - left >= len(t)`，每次 `valid == len(need)` 就 `res.append(left)` |
| 判断是否包含异位词（如：LeetCode 567）| 一旦满足 `valid == len(need)` 就 `return True` |
| 最小覆盖子串（如：LeetCode 76）| 条件是 `valid == len(need)`，然后维护 `min_len` 和 `start`，不断缩小窗口 |
| 包含所有字符的最短子串 | 同上，但需在窗口满足后尽量缩小 `left` |

## 滑动窗口的动机从何而来？

| 触发因素 | 对应判断 |
| --- | --- |
| 目标包含连续区间 | 窗口思想可行 |
| 区间大小固定 | 固定长度窗口 |
| 需高效判断区间中是否所有元素满足某条件 | 滑窗 + 条件检查 |
| 可提前剪枝跳过无效区间 | 滑窗结构天然适合 |

## 从两端拿固定长度

这可以使用逆向思维，从两端变成从中间

# 双指针

## 排序 + 双指针

`排序 + 双指针`是解决三数问题的经典模式，它利用数组的有序性缩小搜索范围，避免冗余计算

逻辑高效清晰，适合在任意三数组合问题中应用

# 快慢指针移除元素

何时使用该模式：
- 从数组中就地移除特定元素
- 在空间受限的情况下过滤数组
- 类似于 283（移动零）、26（移除重复项）、80（最多两个重复项）

# 图论

## DFS / BFS

`DFS`：找连通块、判断是否有环等
`BFS`：求最短路

## 拓扑排序

把拓扑排序想象成一个黑盒，给它一堆杂乱的先修课约束，它会给你一个井井有条的课程学习安排

这一种在图上的「排序」，可以把杂乱的点排成一排
- 就是把一幅无环图「拉平」，且这个「拉平」的图里面所有箭头方向都是一致的：
  ![alt text](https://github.com/donnapersonal/picx-images-hosting/raw/master/image.6m4610s7ji.webp)
- 前提条件：图中无环，从而保证每条边都是从排在前面的点指向排在后面的点
- 即对于任意有向边 `x→y`，`x` 一定在 `y` 之前

拓扑排序可使用 `DFS` 算法，图的后序遍历结果进行反转就是拓扑排序结果

另外，也可用 `BFS` 算法借助每个节点的入度进行拓扑排序

拓扑排序正是用于处理：
- 有向图
- 依赖关系
- 按顺序处理所有节点，且没有环
- 换句话说，它非常适合解决`课程依赖`这种任务调度类型的问题。

选择解法（`DFS` VS. `BFS` 解法的对比）

| 方法 | 原理 | 特点 | 是否返回顺序 | 是否返回顺序 | 适用 |
| --- | --- | --- | --- | --- | --- |
| 拓扑排序（`BFS/Kahn` 算法）| 如果不能遍历所有节点 → 有环 | 递归思路清晰，适合链式依赖 | 可拓扑排序 | 可检测环（3 色状态）| 更倾向层级式任务调度 |
| `DFS + visited` 状态标记 | `DFS` 路径中再次遇到“正在访问”的节点 → 有环 | 更直观，基于入度 | 可拓扑排序 | 可检测环（剩余节点）| 更适合递归链式依赖 |