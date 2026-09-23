# ✦ RLForge — Q-learning 强化学习锻造炉

<p align="center">
  <a href="https://github.com/CJX0712/rl-forge/actions/workflows/ci.yml"><img src="https://github.com/CJX0712/rl-forge/actions/workflows/ci.yml/badge.svg" alt="ci"></a>
  <a href="https://github.com/CJX0712/rl-forge/releases"><img src="https://img.shields.io/github/v/release/CJX0712/rl-forge?sort=semver" alt="release"></a>
  <a href="https://github.com/CJX0712/rl-forge/blob/main/LICENSE"><img src="https://img.shields.io/github/license/CJX0712/rl-forge" alt="license"></a>
  <img src="https://img.shields.io/badge/author-%E6%99%A8%E6%98%9F-1f6feb" alt="author">
</p>

单文件离线强化学习实验台：模型无关的 **Q-learning** 撞上模型方法的 **Value Iteration**，两种独立解法在随机网格世界上交叉验证。零依赖，双击 `index.html` 即用。

## 功能

- **20×15 网格世界**：随机墙壁（15%）+ 随机坑（3%），目标可达性由 VI 预检自动保证
- **Q-learning**：ε-greedy（0.3→0.05 退火）+ 学习率退火（α 0.5→0.05，GLIE 式）+ **乐观初始化 Q=50**
- **Value Iteration 对照**：一键求出 Bellman 精确最优解，白线叠加最优路径
- **可视化**：V 值热区、贪心路径、智能体动画演示；训练后自动对照两者报酬

## 奖励结构

每步 −1 · 目标 +100（终态）· 坑 −10（终态）· γ=0.95 · 碰墙原地不动仍付步代价

## 内置自检（8 项不变量）

| # | 不变量 |
|---|--------|
| 1 | VI 收敛：Bellman 残差 < 1e-8（实测 9.95e-11） |
| 2 | 空网格最优：贪心路径步数 == 曼哈顿距离 |
| 3 | **交叉验证：8 个随机世界 QL 贪心报酬 == VI 最优报酬，路径 maxQ−V* gap 全 0.00** |
| 4 | 确定性：同 seed 两次训练 Q 表逐位一致 |
| 5 | 碰墙语义：原地不动、仍付步代价 |
| 6 | 终态语义：V[goal]=V[pit]=0，从邻格走进 goal 即 done(+100) |
| 7 | 避坑：直线走廊被坑封死，VI 与 QL 都绕行不踩坑 |
| 8 | 环路保护：全零 Q 贪心走步在 maxSteps 内终止 |

## 无头验证

```bash
node _smoke.js   # 8/8 ALL GREEN
node _probe.js   # ASCII 世界 + QL/VI 路径叠加 + Q 表对照
```

## 本轮踩的三个真坑（都是 RL 本身的坑）

1. **跳坑自杀最优**：步代价 −1 + γ=0.95 下，远距状态 V* < −10 → 跳坑（−10 终态）数学上最优。目标奖励 10→100 后消除。
2. **探索失败级联**：坑密度高的世界，随机游走必踩坑终止 → 目标价值永远传不回起点，Q[start] 全 ≈ −10。乐观初始化（Q=+50）驱使系统性扫全动作空间后解决。
3. **欠收敛假象**：QL 与 VI 找到不同最优路线时，对方路线状态访问稀疏、Q 滞后。学习率退火（GLIE）+ ε 下限 0.05 后路径 gap 收敛到 0.000。

## 数学

- Q 更新：`Q[s,a] += α·(r + γ·max_a' Q[s',a'] − Q[s,a])`（终态目标 = r）
- Value Iteration：`V(s) = max_a (r + γ·V(s'))` 迭代至 Δ < 1e-10
- 两条路线独立：QL 无模型在线试错，VI 全图 Bellman 回传——结果一致即互证

MIT License
