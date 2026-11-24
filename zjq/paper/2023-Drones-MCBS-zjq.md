# Paper 1 MCBS 多机冲突协调与实验记录

Xiaoxiong Liu、Yuzhan Su、Yan Wu、Yicong Guo 于 2023 年发表 *Multi-Conflict-Based Optimal Algorithm for Multi-UAV Cooperative Path Planning*，刊于 *Drones* 7(3), 217。本项目主要借鉴**单棵约束树协调多机、发生冲突时重算受约束飞机**的做法。这一思路贯穿阶段 B 的离线规划和阶段 C 的起飞前规划。 [本地论文](2023-Drones-MCBS-zjq.pdf) · [出版原文](https://doi.org/10.3390/drones7030217)

## 论文解决的问题和方法

论文把三维环境中的单机飞行约束和机队协同约束交给不同层处理。低层使用稀疏 A* 为某一架飞机生成满足约束的路径，在展开航路点时排除违背飞行限制的候选。高层在同一棵约束树中维护整队路径组合，每个节点保存当前约束和相应路径。 [论文 §3.2，PDF 第 12–17 页](2023-Drones-MCBS-zjq.pdf#page=12)

- **空间冲突**指同一时刻的点或航段上的飞机距离不足。高层分别给冲突双方设约束，生成两个分支。每个分支只要求低层重算其中一架，其余路径继续参与检查。 [论文 §3.2.1，图 9–12，PDF 第 13–16 页](2023-Drones-MCBS-zjq.pdf#page=13)
- **时间协同冲突**来自任务对到达时间差或到达次序的要求。作者先消解空间冲突，再用路径延长等办法处理时间协同，把空间冲突数和时间协同违约加入高层启发信息。论文第 4 节用仿真比较 MCBS、传统 CBS 与稀疏 A*，用路径成本、协同时间和计算时间等指标解释结果。 [论文 §3.2.1、§4，PDF 第 14–24 页](2023-Drones-MCBS-zjq.pdf#page=14)

## 项目采用与调整的内容

| 借鉴点 | 项目实现与调整 |
|---|---|
| 单棵约束树维护全队解 | 根节点先生成十条轨迹。检测到冲突时分别约束双方，子节点继承其余轨迹。项目优先选择剩余冲突较少的节点，再比较总航程及最长单机航程。 |
| 让低层搜索服从约束 | 论文把路径长度与环境代价组合。项目用三维稀疏 FOCAL A* 在满足地形、限高及冲突约束的范围内生成候选，随后构造七次 Hermite 连续轨迹，再检查机型运动学和整队间距。 |
| 消解时域内的相互接近 | 论文的点和边冲突启发了连续时间的安全体检查。项目对齐两机轨迹的分段时间，对相对位置多项式求连续距离极值和保守界，再独立作 ECEF 复核。 |
| 约束表达与任务目标 | 项目把 CBS 冲突位置转成受约束飞机的保守空间球约束，便于对整个候选曲线作连续证明。十机按原起飞和终点规则执行，按各自到达时刻考核。论文的同时到达和时间顺序约束属于其协同任务设计。 |

空间间距仍然具有时间语义，即比较同一时刻的两条轨迹。它与“要求十机同时到达”是两个不同的验收问题。空间球限制是项目采用的保守工程取法，高层排序也综合了 Paper 13 的总航程和最大单机航程思路。 CBS、机对证明及局部修复实现（工作区 `stage_b_sitl/planning/planner.py`） · 阶段 B 连续规划说明（工作区 `stage_b_cbs/README_阶段B.md`）

## 实验中的改进与证据

1. **阶段 B 的两类证据要分别看。** 最早的十机离线自然任务在约束树根节点即可行。后来验收的共同地面起飞、限高版出现自然冲突，完整轨迹报告记录了约束树生成 5 个节点、展开 3 个，根节点有 2 处冲突。该版完成十机 SITL 全程与 45 对实测间距验收。 离线阶段说明（工作区 `stage_b_cbs/README_阶段B.md`） · 限高版规划报告（工作区 `stage_b_sitl/verified_ceiling_6900/planning/report.json`） · 阶段 B 实测验收（工作区 `stage_b_sitl/GROUND_LAUNCH_VERIFICATION.md`）
2. **阶段 C 在同一棵树内消除重复证明。** 当子节点只改动一架飞机时，包含该飞机的 9 对重新核算。其余机对的分段、时标和机型参数一致时，同轮复用已有证书。JPS 优先优化版记录了 81 次重新计算和 189 次缓存命中，放飞前的独立程序仍全量核验 45 对。 证明缓存（工作区 `stage_c/plan_initial.py#L35`） · 历史优化汇总（工作区 `stage_c/acceptance_archive/astar_initial_release/historical_metrics/planning_optimization/validation_summary.json`）
3. **正式初始规划统一采用稀疏 FOCAL A*。** 同场景三策略各 6 次离线对照，JPS 优先、根节点 A*、全 A* 的规划中位数分别为 18.90、15.75、10.58 秒，18 次输出轨迹一致。正式全 A* 版完成初始规划、独立校验、十机飞行和一次 `quad_01` 实际接管。 三策略试验汇总（工作区 `stage_c/acceptance_archive/astar_initial_release/historical_metrics/astar_dispatch_comparison/summary.json`） · 正式版复测（工作区 `stage_c/acceptance_archive/astar_initial_release/validation_summary.json`）

阶段 C 的在线事务沿用了“只变更一机，整队复查”的思路。在线候选由 `solve.py` 在限定区间内修正原参考，再验证其余九机组合。当前实际接管证据覆盖单架四旋翼，论文中的任务时间协同和更一般的动态场景属于各自另外的实验范围。 在线候选求解（工作区 `stage_c/solve.py`） · [当前实验进度](../progress/README.md)

本地副本保留作者、出版方及许可信息。原文首页版权栏标明 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)。 [本地原件版权栏](2023-Drones-MCBS-zjq.pdf#page=2)
