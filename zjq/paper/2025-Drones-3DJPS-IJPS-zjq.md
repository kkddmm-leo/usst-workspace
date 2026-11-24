# Paper 13 3D-JPS、IJPS 与实验记录

Yuan Lu、De Yan、Zhiqiang Wan、Chuanyan Feng 于 2025 年发表 *Conflict-Free 3D Path Planning for Multi-UAV Based on Jump Point Search and Incremental Update*，刊于 *Drones* 9(10), 688。本项目用这篇论文作三维跳点搜索的实现依据，也借鉴了**只重算受影响部分、保留其余路径、比较总航程和最长单机航程**的思路。 [本地论文](2025-Drones-3DJPS-IJPS-zjq.pdf) · [出版原文](https://doi.org/10.3390/drones9100688)

## 论文解决的问题和方法

论文面向大规模低空多机路径规划。低层把 JPS 扩展到三维 26 个方向，通过自然邻居、强制邻居和跳点规则减少逐格扩展，结合运动约束过滤方向，并在代价和启发量中引入风险权重。高层使用 DOCBS，在关注总路径代价的同时考虑最大单机路径代价，以减少相近解在约束树中的反复展开。 [论文 §3.2.1、图 2、§3.3，PDF 第 9–13 页](2025-Drones-3DJPS-IJPS-zjq.pdf#page=9)

论文的 IJPS 给路径附加时步，把冲突格作为带时间信息的临时障碍。节点的 `g` 表示当前代价，`rhs` 表示从前驱推得的最小一步预测代价。冲突引起两者不一致时，向受影响范围传播更新，再从新的起点重搜受影响的部分，与原路径中可保留的部分拼接。第 4 节表 4 记录 120 机规划总耗时 **235.3 秒**，其中高层冲突消解与重规划 **116.18 秒**。该时间为论文场景的批量规划计时。原文第 5 节把突发动态障碍、实时传感器接入、硬件在环和实飞列为后续工作。 [论文 §3.2.2、图 3、式 16，PDF 第 10–12 页](2025-Drones-3DJPS-IJPS-zjq.pdf#page=10) · [表 4 与结论](2025-Drones-3DJPS-IJPS-zjq.pdf#page=21)

## 项目采用与调整的内容

1. **阶段 B 落地了三维 JPS。** 在等距三维体素内使用 26 邻接、自然与强制邻居剪枝及递归跳点。用保守地形占用判断过滤体素，具体机型的动力学、净空及连续间距以平滑后的轨迹复核。旧版 JPS 使用 `500 × 500 × 500 m` 等距网格，备用稀疏 A* 使用 `500 × 500 × 100 m` 网格。两种网格分别使用各自的搜索规则，JPS 的对称剪枝限定在等距格网。 3D-JPS 实现（工作区 `stage_b_sitl/planning/jps3d.py`） · 阶段 B 实现说明（工作区 `stage_b_cbs/README_阶段B.md`）
2. **约束树增加最长单机航程作为后续排序信息。** 项目先比剩余冲突数，再比总航程，最后比最大单机航程。论文的 DOCBS 组合代价与项目排序都考虑平衡单机负担，具体代价形式分别由各自实现定义。 约束树实现（工作区 `stage_b_sitl/planning/planner.py#L329`）
3. **局部修复和证明复用分层实现。** CBS 中的局部修复选择原轨迹附近的有限窗口，保留未受影响的段，窗口跨度超过 **7 km** 或安全检查失败时回退到全局低层搜索。阶段 C 的起飞前规划还按轨迹及约束签名复用同轮的机对证明。当前复用对象是轨迹段及其安全证书，基于 `g/rhs` 的搜索状态一致性传播仍属于待实现内容。 局部修复（工作区 `stage_b_sitl/planning/planner.py#L373`） · 证明缓存（工作区 `stage_c/plan_initial.py#L35`）
4. **在线事务只改受影响飞机的一段参考。** 当前飞行验证修正 `quad_01` 的限定后缀，其余九架继续执行已确认的轨迹，并对新组合复核 45 对。这里采用七次平滑偏移与版本化提交。针对突发障碍的时变体素增量更新和自动触发由后续阶段 D 验证。 在线局部求解（工作区 `stage_c/solve.py`） · [当前实验进度](../progress/README.md)

## 现行初始规划的算法选择

早期阶段 B 的静态十机离线任务采用 3D-JPS，根节点已可行。换到共同地面起飞及 6850 m 参考限高场景后，原阶段 C 验收日志里有四次高净空档 A* 试探耗尽节点预算，合计 **210.41 秒**，整轮规划 **254.03 秒**。缩短试探预算、优化队列及缓存后，JPS 优先的完整入口将规划用时降到 **30.62 秒**。该轮最终十条底层路径都来自稀疏 A*。 第一轮优化记录（工作区 `stage_c/acceptance_archive/astar_initial_release/historical_metrics/planning_optimization/validation_summary.json`）

在同一十机场景做三种调度策略对照，每组 6 次，18 次均通过独立校验且十条轨迹一致。JPS 优先的规划中位数为 **18.90 秒**，根节点 A*、冲突分支 JPS 优先为 **15.75 秒**，全程稀疏 A* 为 **10.58 秒**。因此正式阶段 C 初始规划改用全程稀疏 FOCAL A*，研究对照仍保留 JPS 版本和 IJPS 的局部更新思想。正式入口已做完整十机 SITL、在线接管、超时回退和准备期关窗复测。结论对应当前固定场景和输入。 18 轮对照汇总（工作区 `stage_c/acceptance_archive/astar_initial_release/historical_metrics/astar_dispatch_comparison/summary.json`） · 正式复测（工作区 `stage_c/acceptance_archive/astar_initial_release/validation_summary.json`）

本地副本保留作者、出版方及许可信息。原文首页版权栏标明 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)。 [本地原件版权栏](2025-Drones-3DJPS-IJPS-zjq.pdf#page=2)
