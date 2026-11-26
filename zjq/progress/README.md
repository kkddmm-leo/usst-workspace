# 十机自主规划实验进度

当前实验范围是西藏地形的十机 AFSIM／ArduPilot SITL 仿真。阶段 A、B 及阶段 C 全 A* 初始规划与单四旋翼在线接管已验收。**阶段 D 已完成历史雷达驱动的十机单机及双机在线改线、两项 Warlock 关窗验收，并于 2026-11-26 补齐第 2 步三项天气故障回放。** 阶段 D 验收材料保存在工作区，场景包交付暂缓。 阶段 C 正式验收（工作区 `stage_c/acceptance_archive/astar_initial_release/validation_summary.json`） · 阶段 D 本地验收（工作区 `stage_d/acceptance_archive/weather_sitl_and_stop_20261125/验收清单.md`） · 天气故障验收（工作区 `stage_d/acceptance_archive/weather_faults_20261126/验收清单.md`）

## 阶段总览

| 阶段 | 状态 | 已核实的完成范围 |
|---|---|---|
| A 场景与编队 | 已验收 | 西藏场景的 4 架四旋翼、3 架 VTOL、3 架常规固定翼，共十架静态部署。核对位置、姿态、高度和静止状态。 |
| B 离线 CBS | 已验收 | 同一西藏自然任务完成十机离线规划、连续轨迹、45 对复核和 Warlock 路线加载。该次任务从约束树根节点取得可行解。 |
| B SITL 基线 | 已验收 | 十机共同地面起飞、全程跟踪、原终点处理与关窗停止，形成 6900 m 限高和 150 m 净空基线。 |
| C-0 快照与台架 | 已验收 | 十机源遥测回放，对齐时钟和状态，验证异常输入拒绝。三种机型分别完成单机台架，翼式机验证预装航段切换。 |
| C-1 十机运行中的在线事务 | 已验收并交付 | 十机共同参与安全校验，实际改线并接管 `quad_01`，完成超时回退和准备期关窗停止。正式初始规划已切换为全程稀疏 FOCAL A*。 |
| C-2 翼式机在线换轨 | 台架已通过，仍待编队验证 | VTOL 和固定翼已各做单机预装航段切换。十机任务中已经验证的在线改线对象是 `quad_01`。 |
| C-3 不停航有期限事务 | 固定触发单机用例已实证 | 飞行继续时异步求解未来切点，逾期回退。历史天气触发及双机协调提交已在阶段 D 验证。 |
| D 历史天气与动态重规划 | 本地验收通过 | 完成十机历史回放、单机及双机在线改线、求解中与候选准备期两项关窗，以及缺帧、过期帧和准备期新天气三项故障回放。实测求解采用完整重规划。 |

以前的 `c2_sitl_experiment` 是飞控保真度与接口分项验证，含三种机型跟踪、时钟和三机同飞实验。表中的 **C-2** 专指阶段 C 的翼式机在线换轨编队子阶段。 早期 C2 控制验证（工作区 `c2_sitl_experiment/c2_delivery_stage_20261112/c2_sitl/results/C2_验证报告.md`） · 阶段 C 子阶段定义（工作区 `HANDOFF_阶段C_AFSIM规划闭环.md`）

## 阶段 A 与阶段 B 的验收基线

### 阶段 A 静态场景

阶段 A 的十个起始槽位沿 45° 排列，相邻 **600 m**，统一 **6000 m MSL**，机头航向 **315°**。西藏自动检查记录十架均静止，位置漂移为 **0 m**，十条起终点直连均会受到地形阻挡。贡嘎、瑞士、科罗拉多在这轮保留纯地理场景。静态起始槽位用于场景部署和规划输入，它与后续 SITL 共同地面起飞的 **200 m** 相邻起飞位是两套配置。 阶段 A 操作手册（工作区 `stage_a_uav_experiment/README_阶段A实验.md`） · 自动检查记录（工作区 `stage_a_uav_experiment/runtime/check-report.json`）

### 阶段 B 离线规划

原离线入口 `START_STAGE_B_CBS.cmd` 生成十条轨迹，完成独立校验后由 Warlock 自动绘制。正式场景包中 `stage_b/阶段B_交付验收记录.md` 记录该次自然任务的规划及内部复核耗时 **51.482 秒**，最小地形净空保守下界 **205.675 m**，独立 ECEF 连续间距保守下界 **101.537 m**。该次 45 对检查通过，约束树生成及展开各 **1** 个节点，根节点冲突数为 **0**。这个阶段提供规划和显示基线，飞行执行由后续 SITL 入口负责。 阶段 B 离线实现和边界（工作区 `stage_b_cbs/README_阶段B.md`）

### 阶段 B 共同地面起飞 SITL

专用入口 `START_STAGE_B_CBS_sitl.cmd` 依次执行新规划、独立核查、路线装载和十机地面起飞。四旋翼与 VTOL 名义巡航 **12 m/s**，普通固定翼 **20 m/s**。验收采用 **6850 m MSL 参考限高、6900 m MSL 实测限高、规定阶段 150 m 保守净空**，并保持各机对各自的间距门限、原有起飞配置和终点规则。 阶段 B SITL 流程（工作区 `stage_b_sitl/README_十机SITL.md`）

限高版以 **2 倍**完成 **3474.96 仿真秒**。规划耗时 **226.96 秒**，最高海拔保守上界 **6806.70 m**，最小保守净空 **178.96 m**，45 对实测间距和十个原终点均通过。实际关窗测试约 **55 ms** 停止本轮十架 SITL，排空日志并返回 `3`。阶段 B 归档统一位于工作区 `stage_b_sitl`。 限高版验收说明（工作区 `stage_b_sitl/GROUND_LAUNCH_VERIFICATION.md`） · 阶段 B 关窗验收（工作区 `stage_b_sitl/verified_ceiling_6900_stop_test/stop_test_verification.json`）

## 阶段 C 现行正式流程

正式入口为场景包根目录的 `START_STAGE_C.cmd`，工作区保留同名入口。每次启动先重新规划。规划用 Warlock 确认绘制十条路线后，实时 Warlock 接替并确认装载同一批轨迹，十架 SITL 随后启动。起飞前的根节点、CBS 分支和适用局部窗口统一使用稀疏 FOCAL A*，独立校验继续执行原物理安全检查。 工作区入口（工作区 `START_STAGE_C.cmd`） · 阶段 C 使用说明（工作区 `stage_c/README.md`）

### 初始规划已经完成的优化

1. 原阶段 C 完整验收规划耗时 **254.03 秒**，其中四次高净空档 A* 试探合计 **210.41 秒**。限制试探预算并改进 FOCAL 队列、地形查询和机对证明缓存后，JPS 优先版完整入口规划耗时降至 **30.62 秒**。
2. 同场景三策略离线对照各 **6 次**，共 **18 次**均通过独立检查，轨迹逐条一致。JPS 优先、仅根节点 A*、全 A* 的规划中位数分别为 **18.90、15.75、10.58 秒**。
3. 正式全 A* 入口的两次实际复测如下。60 秒门槛应用于十机规划指标，Warlock 装载时间另计。

| 正式复测 | 十机规划墙钟时间 | 从启动脚本到实时 Warlock 确认装载 |
|---|---:|---:|
| 完整十机 SITL 与在线接管 | **16.37 秒** | **41.95 秒** |
| 准备期实际关窗 | **17.46 秒** | **44.37 秒** |

第一轮启动优化、18 次离线对照和正式接入各有独立证据，离线中位数与启动界面的端到端耗时分别报告。 耗时优化说明（工作区 `阶段C_十机路径规划时长优化说明.md`） · 离线对照记录（工作区 `stage_c/acceptance_archive/astar_initial_release/historical_metrics/astar_dispatch_comparison/summary.json`） · 正式验收汇总（工作区 `stage_c/acceptance_archive/astar_initial_release/validation_summary.json`）

### 在线接管及实际飞行验收

- **状态来源和触发。** 从 SITL 原始遥测截取并对齐十机状态。当前用例由固定仿真时刻触发，约 1120 仿真秒注入一次超时，约 1140 仿真秒启动接管求解。候选在期限内通过整队复检后提交，Warlock 绘制确认和实际遥测接管分别记录。C-0 的四个十机回放时刻中，最大时钟对齐不确定度约 **0.114 仿真秒**。 C-0 回放报告（工作区 `stage_c/runtime/c0_replay/report.json`） · 在线调度（工作区 `stage_c/transaction.py`）
- **接管结果。** 完整轮次中超时回退通过，`quad_01` 之后实际切换局部后缀，实测横移 **42.34 m**，改线段相对有效参考的最大误差 **0.59 m**。另九架继续沿初始参考执行，并纳入候选检查。 正式版闭环报告（工作区 `stage_c/acceptance_archive/astar_initial_release/full_workflow/flight/stage_c_report.json`）
- **全程结果。** 以 **1.5 倍**完成 **3474.96 仿真秒**，45 对机间距和十个原终点均通过。最小保守净空 **178.74 m**，最大海拔保守上界 **6806.69 m MSL**，全队最小保守间距 **48.00 m**。VTOL 名义 **12 m/s**，三架巡航误差峰值分别为 **9.95、8.38、4.01 m**，均通过 50 m 门限。 正式版全程约束报告（工作区 `stage_c/acceptance_archive/astar_initial_release/full_workflow/flight/execution_report.json`）
- **不通过轮次及重跑。** 全 A* 接入后的首轮完整飞行在约 484 仿真秒发生时钟对齐扰动，8 对保守间距下界未获认证。保留该轮诊断，沿用同一轨迹和原判据串行重跑，取得上述通过结果。 首轮诊断（工作区 `stage_c/acceptance_archive/astar_initial_release/full_clock_findings/flight/execution_report.json`）
- **自动停止。** 现行全 A* 入口在候选准备好且待提交时实际关闭 Warlock，本轮十架 SITL 在退出后约 **47 ms** 内停止，任务取消，日志排空，退出码 `3`。求解中关闭窗口也在前期版本中做过实测，旧原始资料收纳于历史压缩档。 正式版关窗记录（工作区 `stage_c/acceptance_archive/astar_initial_release/stop_prepared/stop_report.json`） · 旧验收清理与保全记录（工作区 `stage_c/acceptance_archive/astar_initial_release/cleanup_report.json`）

## 阶段 D 历史雷达实验

### 现行工作区入口

双击根目录的 `START_STAGE_D.cmd`，默认启动十机双机绕飞。入口自动核对两份已通过的回放证据，每次重新规划十条初始航线，再启动 SITL。从根目录运行 `.\START_STAGE_D.cmd --case single` 可切换到十机单机绕飞。默认双机和命令行单机均通过了新版入口的启动检查。 工作区飞行入口（工作区 `START_STAGE_D.cmd`） · 阶段 D 使用说明（工作区 `stage_d/README.md`）

离线选例使用 `.\stage_d\START_STAGE_D_PREFLIGHT.cmd`。独立预检将十条基线航迹与完整历史雷达序列对齐，找出原航迹会进入禁入区的片段，以及无冲突的对照。入口验证筛出 **1 个无冲突、12 个单机、12 个多机候选**，结果写入新运行目录的 `preflight.json`。 独立预检入口（工作区 `stage_d/START_STAGE_D_PREFLIGHT.cmd`） · 预检结果（工作区 `stage_d/runtime/preflight_launcher_check_20261126/preflight.json`）

### 当前的天气检测和改线

实验使用 60 帧已校验的爱荷华州 NOAA MRMS 历史雷达数据。离线选例和事后验收使用完整观测序列。起飞前先沿用阶段 C 的地形、限高和机间安全等约束规划十条初始航迹，天气约束目前在飞行中接入。 离线筛选（工作区 `stage_d/preflight.py#L69`） · 初始规划入口（工作区 `stage_c/workflow_stage_c.py#L55`）

正常回放约每 **120 仿真秒**发布一帧。每接收一个新的有效天气版本，系统按最新已发布帧提取强回波、外扩禁入区，并暂时按禁入区在未来 **600 仿真秒**保持原状来检查十机有效航迹。这种持续性预测用于估计航迹何时进入禁入区。查到预计冲突后，程序为具备在线接管能力的受影响飞机重新求解剩余航迹，候选通过天气、地形和全队 45 对间距校验后才可换轨。 天气模型（工作区 `stage_d/weather.py#L165`） · 在线事务（工作区 `stage_d/flight_live.py#L162`）

双机轮次在 **2163.01 仿真秒**收到第 36 帧，对 `quad_01` 和 `quad_04` 报出预计侵入风险。**2172.82 秒**生成通过校验的候选，**2191.02 秒**两架在同一次事务中换轨。验收采用事后观测序列与原始 SITL 遥测复核实际飞行。 双机换轨事件（工作区 `stage_d/runtime/flight_weather_multi_r2/flight/events.jsonl#L33`） · 原始遥测审计（工作区 `stage_d/runtime/flight_weather_multi_r2/flight/stage_d_report.json`）

### 本地验收

- **历史回放与在线改线。** 无冲突、单机和双机回放通过。使用 `START_STAGE_D.cmd` 实测 `quad_01` 单机改线和 `quad_01`、`quad_04` 同时协调改线，两轮均完成 **3474.96 仿真秒**、十个原终点和 45 对间距复核。 阶段 D 本地验收（工作区 `stage_d/acceptance_archive/weather_sitl_and_stop_20261125/验收清单.md`）
- **实际关窗。** 求解中和候选准备期均通过真实 Warlock 关窗验收。以 Warlock 退出为起点，两轮十架 SITL 最迟分别 **47 ms** 和 **63 ms** 退出，工作进程退出、日志排空、零迟到提交。 本地验收清单（工作区 `stage_d/acceptance_archive/weather_sitl_and_stop_20261125/验收清单.md`）
- **第 2 步故障回放。** 单帧缺失后可随下一帧恢复，持续缺帧在天气认证到期前 **15 仿真秒**停止。过期帧被拒收，达到接收截止的候选零提交，原航迹在预计侵入前停止。新帧在旧候选切换时刻到达时，旧候选撤销，随后只有新版天气绑定且通过整队校验的候选生效。三项使用十机基线、历史雷达和实际工作进程进行离线事务验证。 故障回放验收（工作区 `stage_d/acceptance_archive/weather_faults_20261126/验收清单.md`）
- **已验证范围。** 阶段 D 实际在线改线对象为四旋翼，VTOL 和固定翼参加编队安全复检。完整重规划已通过十机场景本地验收，局部修复及 IJPS 按实测收益决定是否开展。场景包交付暂缓。 阶段 D 终版方案（工作区 `阶段D_历史雷达回放与动态重规划实施方案.md`） · 基线场景与高度说明（工作区 `stage_b_sitl/README_十机SITL.md`）

### 论文方法与待验证范围

阶段 D 借鉴绕飞论文的带缓冲时变禁入区和滚动航迹检查。正式验收采用从雷达栅格提取禁入区、持续性预测和现有多机规划器。论文用 RainNet 从多帧已观测雷达图递推未来回波，这一步尚未接入阶段 D。 [绕飞论文方法](https://www.mdpi.com/2226-4310/13/6/545) · [RainNet 原论文](https://gmd.copernicus.org/articles/13/2631/2020/)

当前结果验证了历史天气驱动的十机改线流程。回放程序按发布时间使用雷达回波，仍利用档案中下一帧的发布时间确定部分航迹的保护时限。起飞前按当时可用天气规划、下一帧到达时间未知时的保护策略，以及在未参与选例和调参的历史天气上比较持续性预测、平移外推和 RainNet，均属于待验证内容。 历史帧发布（工作区 `stage_d/weather.py#L102`） · 保护时限计算（工作区 `stage_d/replay_run.py#L28`）

## 资料索引

- 文献及实现对照集中在 [Paper 1 MCBS](../paper/2023-Drones-MCBS-zjq.md)、[Paper 12 Integrated ECBS](../paper/2025-RAL-IntegratedECBS-zjq.md)、[Paper 13 3D-JPS／IJPS](../paper/2025-Drones-3DJPS-IJPS-zjq.md)和 [Real-Time 雷暴改线](../paper/2026-Aerospace_RealTimeRerouting_zjq.md)，每份说明旁有对应 PDF。
- 阶段 D 验收汇总在 `stage_d/acceptance_archive`，正式运行的原始遥测仍保存在 `stage_d/runtime`。阶段 C 正式原始材料在 `stage_c/acceptance_archive/astar_initial_release`，较早的完整文件归入 `superseded_stage_c_acceptance.7z`，常用指标在现行归档的 `historical_metrics`。 阶段 D 验收汇总（工作区 `stage_d/acceptance_archive/weather_sitl_and_stop_20261125/acceptance.json`） · 阶段 C 交付校验（工作区 `stage_c/acceptance_archive/astar_initial_release/delivery_verification.json`） · 目录归档约定（工作区 `AGENTS.md`）
- 本轮在本地整理供审阅。下轮若发布到 GitHub，随附版本的授权说明见各论文记录，尤其是 Paper 12 的 IEEE 出版版。 [Paper 12 版本说明](../paper/2025-RAL-IntegratedECBS-zjq.md)

文中的工作区路径用于定位实验源码和原始验收证据。仓库中已发布的论文与说明可以直接从 [论文目录](../paper/README.md) 查阅。
