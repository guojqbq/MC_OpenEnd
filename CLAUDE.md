# MC_OpenEnd — Claude Code 入口

你正在开发 MATLAB/Simulink R2026a 的三种 PMSM 驱动拓扑比较研究。

## 优先级最高的任务源

**先完整阅读 [docs/Claude_Agent_任务指令_R2026a.md](docs/Claude_Agent_任务指令_R2026a.md)，这是本项目的工程规格与验收清单。**
不要仅根据 README 生成简介、仿真框图或伪造结果。交付必须包含经过实际 MATLAB 验证的电机电气模型、FOC 和对比指标。

## 路径与安全约束

- 仓库工作根目录：C:\workspace\MC_OpenEnd（若实际仓库另有路径，以当前仓库根目录为准）。
- **权威参数仓库**：`https://github.com/guojqbq/MC_RM108_Softswitch`，分支 `main`，目录 `parameters/motor/`：https://github.com/guojqbq/MC_RM108_Softswitch/tree/main/parameters/motor 。该仓库为**唯一优先参数来源**，只读访问，记录实际使用的源 commit SHA。不要仅根据本机路径假设存在。
- 如本机已有 `C:\workspace\MC_RM108_Softswitch\parameters\motor`，可校验仓库地址、分支、commit 和文件完整性后作为本地读取路径；如不存在，允许从 GitHub 读取或克隆到独立的参考目录（不修改源库）。
- 重点检查 `motor_parameter.md`（基础常量）、`load_rm108_motor_parameters.m`（结构体/默认值）、`LdLq_lookup_from_screenshots.xlsx`（饱和查表）、`RM108_Motor_External_Characteristics_Specification.md` 和 `RM108_Star_Delta_Torque_Speed_Data.csv/.xlsx`（星角外特性）。不得把星角结果直接当成开绕组或六相仿真结果。
- 参数冲突需记录并给出建模选值依据，例如 20 kHz PWM 下 10 μs deadtime 的适用性、700 A trip 与 735.4 A peak limit 的关系；`L0=0.5*Ld` 只是当前脚本采用的假设，需要敏感性分析。
- 仅用于 Simulink 仿真；**不得写入固件、烧写电控、操作真实电机/逆变器、执行台架实验**。
- 若参数不足，标注假设，不得补造真实电机或器件参数；六相模型不可把三相参数当作真实六相测试数据。
- Git commit 分阶段进行，不 force push，不覆盖已有用户工作，不自动删除任何原始数据。

## 必须遵循的执行次序

P0 环境/参数核验 → P1 星接基线真实运行 → P2 开绕组双逆变器与 S0/S1/S2 共模/零序抑制 → P3 双三相/六相 FOC → P4 公平对比、量化指标、技术报告及 PPT。

- 同一顶层 .slx 内用 Variant 等结构组织三拓扑，但每一配置都要独立仿真，禁止把不同电机并在同一轴上。
- 先保证可运行、可复现，再扩展高级指标。每阶段提交代码与 VALIDATION.md、progress.md 的实际证据。
- 没有执行 MATLAB 不得宣称已成功仿真或完成；无法运行时保留明确的执行命令与阻塞原因。
- 模块路径/参数名称要在本机 R2026a 实测核验，不能臆造库路径。
- 仿真结果 PPT 只能引用已计算的真实结果；其余标记“待验证”。

## 本次启动后的第一步

1. 查看 git status、仓库文件和目标路径；检测 MATLAB 版本/许可与 Simscape 模块。
2. 从 GitHub `main` 的 `parameters/motor/` 列出并检查所有源文件，核对实际源 commit；建立参数溯源与假设表。
3. 产出 docs/design_review.md 及初始 VALIDATION.md、progress.md；随即按任务文件逐阶段推进。在没有严重阻塞时不必等待新指令，但每阶段必须真实运行和自测。

## 重要更新：星接 THD v0.3

优先审查 GitHub Release https://github.com/guojqbq/MC_RM108_Softswitch/releases/tag/rm108-star-thd-v0.3 。星接 P1 首选模型为附件 RM108_StarTHD_v0_3.zip 内的 PMSM6T_RM108_StarTHD_v0_3_dev.slx，PWM 7 kHz，MATLAB R2026a。先完整解压、校验附件 SHA-256 0801916ab9f7f21931ad8539b1d66170e7113a82ee967b2b14aafe3ae33aa31f，再阅读 README、九工况验证报告/复算脚本并实际运行；不可用 main 分支旧 v0.1 代替。Release 声称低频整数谐波 THD（≤5 kHz）0.307%–1.468%，相对原方案改善 59.8%–82.1%；≤50 kHz 全部非基波指标为 5.436%–21.909%。这些是来源声明，未经本项目独立复算时不得当作实测结果。三拓扑应首先统一 7 kHz PWM 频率公平比较，再单独开展频率扫描。保留原 Release 包只读，所有改动在 MC_OpenEnd 工作副本内完成。

## 固定拓扑精简要求（重要）

本项目研究**三种固定连接拓扑**，不是星—角软切换，不要将原 RM108 软切换装置原样移植进正式模型。复用 Release v0.3 和其他旧模型时，先在只读源模型上完成基线复算，再在 **MC_OpenEnd 内的新副本**清除不必要的星—角切换继电器/接触器（K2/K3/K5/K6 等，按实际作用识别）、TopologyFSM、T1～T6 过渡状态、切换时序/ZCS 及其专用控制信号、切换检测和切换专属测试逻辑。连接改成直接固定星点（Star）、绕组两端分别接双桥（OpenEnd）、两套固定三相绕组/星点（Dual Three-Phase），三者仅由顶层 Variant **仿真前选型**，不在运行中通过继电器切换。不能只将多余逻辑禁用但保留复杂冗余电气网络。注意**不能误删正常 FOC、开绕组共模/零序抑制及必要的真实零序通路**。记录移除模块清单和修改前后连接图、参数/算法差异，删除后重新执行 Update Diagram、Star v0.3 基线与九工况、三拓扑独立仿真和 THD 验收；源仓库和原 ZIP 不得修改。
