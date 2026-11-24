# Paper 12 Integrated ECBS 连续轨迹与实验记录

Liya Li、Yucong Liu、Guoyuan Qi、Choon Ki Ahn 于 2025 年发表 *Integrated Multi-UAV Motion Planning Under Kinematic Constraints Based on Enhanced Conflict-Based Search*，刊于 *IEEE Robotics and Automation Letters* 10(12), 13050–13057。本项目借鉴了连续曲线冲突复核、相邻轨迹段的高阶状态传递和候选接受流程，并将这些思路用于放飞前校验和在线接管。 [本地论文](2025-RAL-IntegratedECBS-zjq.pdf) · [出版 DOI](https://doi.org/10.1109/LRA.2025.3627094) · [作者机构出版记录](https://pure.korea.ac.kr/en/publications/integrated-multi-uav-motion-planning-under-kinematic-constraints-/)

## 论文解决的问题和方法

在离散路径规划之后独立平滑轨迹，曲线可能偏离原格点折线，带来新的机间冲突。原文 ECBS 在高、低层均用 FOCAL 集合筛选候选。低层将短连续轨迹耦合进图搜索，格点节点记录时空信息，候选邻居参与短轨迹生成，候选的轨迹状态也用于评价搜索节点。高层结合连续轨迹与时空约束消解冲突。 [论文 §III–V，PDF 第 2–6 页](2025-RAL-IntegratedECBS-zjq.pdf#page=2)

论文采用**离线滚动时域优化**。对候选段多看一个动作时长 `τ`，求解到 `Tᵢⱼ + τ`，保留到 `Tᵢⱼ`，将位置、速度、加速度和 jerk 传给下一段。七次位置多项式给出可微的连续轨迹，目标包含平滑性和段长方面的代价。节点展开后先形成候选，只有被搜索选中的候选才纳入路径。原文的异步描述的是这一搜索内顺序，`τ` 由格宽和期望速度决定。仿真使用 NLopt 的 LN_COBYLA，并将在线飞行及动态障碍作为后续扩展。 [论文 §IV-A、图 1、式 7–11，PDF 第 3–4 页](2025-RAL-IntegratedECBS-zjq.pdf#page=3) · [论文 §VI–VII，PDF 第 6–8 页](2025-RAL-IntegratedECBS-zjq.pdf#page=6)

## 项目采用与调整的内容

| 借鉴点 | 项目实现与调整 |
|---|---|
| 搜索同时考虑可执行的连续轨迹 | 初始规划用稀疏 FOCAL A* 搜索地形可行的折线候选，简化后用分段七次 Hermite 多项式按端点和边界速度构造连续参考。候选构造后按机型速度、加速度、jerk、转弯和升降约束检查。论文使用数值优化求短段，项目使用 Hermite 边界求解和可行性筛选。 |
| 用连续轨迹消除平滑后的冲突 | CBS 的对象是各机已经构造好的连续轨迹。地形用 DEM 上界和 Bernstein 包络检查，机对用相对位置多项式及保守界证明，最后用独立 ECEF 审计复查全部 45 对。 |
| FOCAL 作为候选搜索机制 | 阶段 C 将 FOCAL 用在初始规划的低层 A*，开放表阈值取最小总代价的 `1.10` 倍，再按启发距离等规则选择候选。CBS 高层使用本项目的冲突与航程排序。论文对其框架的优化与性能结论适用于原文设置。 |
| 在未来切点保持高阶状态 | 在线求解器切分原参考，为 `quad_01` 的 64 仿真秒局部区间叠加七次平滑横移。偏移在接入和接回边界处为零，其速度、加速度和 jerk 也为零，由此继承原参考在这些切点的 C3 状态，保留原到达时标和终点。 |

初始构造、连续证明、FOCAL 与在线拼接分别保存在以下实现中。 轨迹与证明（工作区 `stage_b_sitl/planning/polynomial.py`） · 低层 FOCAL A*（工作区 `stage_c/focal_search.py`） · 在线拼接（工作区 `stage_c/solve.py#L20`）

## 从论文的离线思想到实际 SITL 接管

项目把候选接受的思想扩展为有版本的飞行事务。固定巡航事件从 SITL 源遥测取得对齐的十机快照，独立工作进程提前生成候选，对改线与其余九机组合重新执行检查。主流程核对基准版本、环境摘要、截止时刻以及实际轨迹捕获范围，之后在预定控制时刻切换。超时则保留当前参考。Warlock 的绘制和装载有单独记录，接管是否执行由源遥测核验。触发和超时设置来自本项目协议，论文的 `τ` 对应离线网格动作。 事务调度与提交（工作区 `stage_c/transaction.py`） · 源遥测验收（工作区 `stage_c/verify_live.py`）

正式全 A* 版通过验收的完整轮次以 **1.5 倍**运行。`quad_01` 实测横移 **42.34 m**，改线段相对生效参考的最大误差约 **0.59 m**。十机完成原终点规则，超时回退通过，准备期关闭 Warlock 后十架 SITL 在约 **47 ms** 内停止。该证据覆盖固定触发下的单架四旋翼接管。翼式飞机目前的接管证据来自各自的单机预装航段台架。 正式验收汇总（工作区 `stage_c/acceptance_archive/astar_initial_release/validation_summary.json`） · [当前实验进度](../progress/README.md)

本地文件是带 IEEE Xplore 授权水印的出版版。首页版权栏标明个人使用许可，并要求重新发布或分发取得 IEEE 许可。若向 GitHub 发布资料，PDF 的发布范围按该版本的授权确定，论文可通过 DOI 引用。 [本地原件版权栏](2025-RAL-IntegratedECBS-zjq.pdf#page=1)
