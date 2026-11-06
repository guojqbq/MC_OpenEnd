# MC_OpenEnd — PMSM 三拓扑统一仿真研究

本仓库用于在 **MATLAB/Simulink R2026a** 中建立可复现的电机控制仿真，统一比较：

1. 星接三相 PMSM + 单逆变器 + FOC；
2. 开绕组三相 PMSM + 双逆变器 + FOC，含共模/零序电压与零序电流抑制对照；
3. 双三相（六相）PMSM + 两组三相逆变器 + FOC。

**工程任务全文：** [`docs/Claude_Agent_任务指令_R2026a.md`](docs/Claude_Agent_%E4%BB%BB%E5%8A%A1%E6%8C%87%E4%BB%A4_R2026a.md)

**Claude Code 项目入口：** [`CLAUDE.md`](CLAUDE.md)

## 本地准备

建议本仓库克隆到 `C:\workspace\MC_OpenEnd`；参数读取自 `C:\workspace\MC_RM108_Softswitch\parameters\motor`（仅只读，不能修改原仓库）。

在仓库根目录启动 `claude`，让 Claude Code 阅读 `CLAUDE.md` 与完整任务指令。先检查 MATLAB 版本、许可、库模块以及参数清单，之后按 P0～P4 分阶段实现和实测验证。**在未运行 MATLAB 之前，任何指标与 PPT 中的仿真结果都不得作为已验证结果。**

## 安全边界

仅允许仿真和研究脚本工作；不得改动原有电机控制固件、烧写硬件、控制电机台架，或覆盖参数来源目录。模型应保存为独立、可复现的 `.slx` 工程。