# MC_OpenEnd — Claude Code 完整开发任务（当前主指令）

> 最新整合版：MATLAB/Simulink R2026a、RM108、Star THD v0.3、开绕组共模/零序抑制、双三相电机、三拓扑公平量化对比、PPT。
> **优先级**：本文件是当前主任务；`CLAUDE.md` 是自动入口；`docs/Claude_Agent_任务指令_R2026a.md` 作为更详细的技术验收补充。如出现不一致，以本文件的最新方案为准。
> 只允许本地仿真开发，不允许电机实物上电、烧写、台架操作。

## 1. 工作区与参数源

- 项目仓库：https://github.com/guojqbq/MC_OpenEnd
- 工作根目录：`C:\workspace\MC_OpenEnd`（以实际 git 根目录为准）
- 参考源仓库（**只读**）：https://github.com/guojqbq/MC_RM108_Softswitch
- RM108 电机参数来源：源仓库 `main/parameters/motor/`。本地参考位置为 `C:\workspace\MC_RM108_Softswitch\parameters\motor`；不存在时从 GitHub 获取并记录版本，不得误当作本工程仓库。
- 依次检查 `motor_parameter.md`、`load_rm108_motor_parameters.m`、`LdLq_lookup_from_screenshots.xlsx`、星角外特性规格和 CSV/XLSX，并追踪字段和源 commit。
- 核实 Rs、Ld/Lq、ψf、极对数、惯量、阻尼、Imax、母线、零序电感、饱和表、PWM、采样、死区及额定工况；严格区分相/线、峰值/RMS、假设/实测。原脚本的 L0=0.5Ld 属估算；20 kHz/10μs 死区以及 700A trip/735.4A 峰值限制需先核验。

## 2. 先复现最新星接 THD 基线，不要从零重写

首选 Release：https://github.com/guojqbq/MC_RM108_Softswitch/releases/tag/rm108-star-thd-v0.3

- ZIP：`RM108_StarTHD_v0_3.zip`，SHA-256 `0801916ab9f7f21931ad8539b1d66170e7113a82ee967b2b14aafe3ae33aa31f`。
- 模型：`PMSM6T_RM108_StarTHD_v0_3_dev.slx`，这里的 PMSM6T 是**六端子三相**，非六相。
- Release 声明 R2026a、固定星接、7 kHz、默认工况 2000 rpm / 10 N·m；九工况≤5 kHz低频整数谐波 THD 为 0.307%–1.468%，相对旧方案下降 59.8%–82.1%；≤50 kHz 全部非基波指标 5.436%–21.909%。**这些是发布者数据，未复算前不能宣称本项目已验证。**
- 通过授权下载，**完整解压**、验证 ZIP 哈希，阅读 README/参数/九工况脚本/报告/CSV，再在本机 R2026a 打开、Update Diagram、短仿真、复算九工况。记录谱分析窗、基波识别、整数谐波与全频段非基波定义；不可将两类指标混称。
- 审计原仓库 `model/reference/OEWPMSMVelocityControl.slx`、`RM108_softswitch.slx` 及 v0.3；建立模块 `REUSE/ADAPT/REBUILD` 清单。优先复用已有的 Simscape PMSM、电气变换器、FOC、PWM、检测及 THD 计算；不继承未验证的性能或参数。
- Release ZIP 和源仓库只读，所有修改在本仓库新副本中进行；获取失败明确写为阻塞，不以旧 v0.1 冒充 v0.3。

## 3. 清除旧星—角软切换内容（仅新副本）

本研究是**固定 Star / 固定 Open-End / 固定 Dual Three-Phase**，不是绕组动态星—角切换。先查电气连接与功能依赖，再从新副本删除仅用于旧软切换的 K2/K3/K5/K6 等继电器接触器、T1～T6、TopologyFSM、过渡状态、零电流切换 ZCS、切换检测/计时、冗余线束和专用测试逻辑；不要只禁用却保留复杂电气网络，也不能按名称盲删。**必须保留** FOC、PWM、必要的逆变器保护、传感器及开绕组共模/零序环流控制和真实物理零序回路。

生成 `docs/model_cleanup_audit.md`：列保留、修改、移除、原因、前后连接图和行为复核结果。删除后重新运行星接九工况并重新计算 THD，不能假设删除开关不改变结果。

## 4. 统一的可运行顶层 .slx

在同一个顶层 `PMSM_Topology_Comparison_R2026a.slx` 用 Variant/Model Reference **仿真前切换**三种拓扑，按配置独立运行（禁止并联同一机械轴共享负载）：

**A. Star：** RM108 三相电气 PMSM + 单三相两电平桥 + 固定浮动星点 + FOC；优先沿用成功复算的 v0.3 7 kHz THD 优化方案。

**B. Open-End：** 同一个三相绕组两端均接出 + 双三相逆变器 + 双桥协调调制 + dq/速度 FOC + 弱磁。重点建立共母线零序闭合通路；提供 S0 基础 PWM、S1 协调 PWM/零序电压抑制、S2 零序电流闭环三模式，报告实际效果和调制约束；隔离双电源另立工况。分别定义绕组零序电压 u0=(ua+ub+uc)/3、零序电流 i0=(ia+ib+ic)/3、相对机壳参考的共模电压和寄生漏电流；无寄生/轴承模型不得声称真实 EMI、轴承电流结论。

**C. Dual Three-Phase：** 两组三相电气绕组、两个星点、双三相桥、同一转子及正确的六相/双 dq 或 VSD FOC；包含绕组空间角和耦合。检查 R2026a 的 Six-Phase PMSM 实际块及其固定角度限制，不套用 R2026b 才有的命名/选项。六相参数不足时按明确的等效设计假设建模，不能把六端子三相模型冒充六相。

电机与逆变器必须具备真实物理电气端口。允许平均值模型用于早期闭环调通，但 THD、高频共模和开关损耗正式对比必须使用适用的开关级模型及足够时间分辨率。

## 5. 控制、仿真与公平性

- 三拓扑包含可靠的采样、电流 PI、速度 PI、Clarke/Park（六相使用正确变换）、解耦前馈、抗饱和、电流/电压限幅、弱磁和适用的 PWM 调制。
- 第一组**统一 7 kHz**、同工况、同量测窗口、同频谱定义；第二组单独扫描 PWM 频率（可含 20 kHz），不可用频率差异假装拓扑增益。
- **两种供电边界必须单独报告**：(1) 相同整车电池/总电源预算（主要）；(2) 相同单桥 DC-link 电压（辅助）。明确桥数量、额外电源/DC-DC、半导体电流/热应力与器件总量约束；不得无成本宣称电压增益。
- 对 Star/Open-End 尽可能保持同一物理电机参数与负载；六相采用明确的总铜量、损耗、空间角/磁耦合假设并做敏感性分析。
- 工况含低速稳态、速度/转矩阶跃、弱磁/高速边界、四象限与再生、母线跌落、PWM 死区与频率扫描；故障容错仅有真实故障模式、重构控制及仿真证据才可声称。

**必须自动量化**：≤5 kHz 整数谐波 THD、≤50 kHz 全部非基波指标（按原包明确算法）、相电流 RMS/峰值、dq 跟踪、动态超调/调节时间、转矩脉动、基速/转矩转速包络、调制裕量、u0/i0、共模电压峰值/RMS、零序环流/频谱、各桥直流功率、铜耗及可靠参数支持的器件损耗/效率、器件/传感器/接线数及成本复杂度。无器件实测参数不可伪称准确效率。

## 6. 交付与验收

实际完成而非只写脚本或画框图：

- 可打开、Update Diagram 通过且三拓扑可独立 `sim` 的顶层 .slx；
- 自动建模、初始化、批量测试、THD/共模/转矩/效率计算和导出 MATLAB .m；
- 参数溯源、假设记录、场景矩阵、运行日志、原始数据来源与结果 CSV/图；
- `docs/model_cleanup_audit.md`、`docs/star_thd_v03_reuse_audit.md`、`docs/technical_report.md`、`docs/conclusions.md`；
- 10–15 页带真实波形、指标表和对比结论的 `docs/Topology_Comparison_Result.pptx`；
- `README.md`、`VALIDATION.md`、`progress.md`，记录每阶段 PASS/FAIL/NOT-RUN、MATLAB 版本/许可/日志、文件及命令、Git commit 和遗留问题。

**绝不编造仿真数据或模型已经运行的证据。** 发布版数据需与项目复算数据严格分列；PPT 未完成部分标记“待验证”。

## 7. 连续执行顺序（可自主继续，不需要逐小步确认）

P0：安全检查 git 状态、更新仓库（不得覆盖未提交工作）、R2026a/许可/电气库探测、只读参数解析、Release ZIP 获取与复算、复用/移除审计。

P1：在新副本清理星—角切换组件，打通固定 Star，复跑 v0.3 九工况及 THD，保存证据。

P2：构建 Open-End 双桥固定电气拓扑，实现 S0/S1/S2 共模零序控制、验证真实零序物理回路及运行证据。

P3：构建双三相六相机电模型与独立 FOC，记录参数假设、实跑闭环。

P4：统一 7 kHz + 频率扫描 + 双供电边界批量对照，输出技术报告、带数值结论与 PPT。

分阶段提交 Git、小步自测并更新进度。若 MATLAB 不可用或参数不充分，记录阻塞和可复现的待运行步骤，继续不受阻部分，不得假称成功。不修改源仓库，不 force push，不碰固件、真实硬件、烧写或台架。
