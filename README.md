# MC_OpenEnd — PMSM 三拓扑统一仿真研究

本仓库用于在 **MATLAB/Simulink R2026a** 中建立可复现的电机控制仿真，统一比较：

1. 星接三相 PMSM + 单逆变器 + FOC；
2. 开绕组三相 PMSM + 双逆变器 + FOC，含共模/零序电压与零序电流抑制对照；
3. 双三相（六相）PMSM + 两组三相逆变器 + FOC。

**最新 Claude Agent 完整主指令：** [docs/AGENT_FULL_TASK.md](docs/AGENT_FULL_TASK.md)

**早期工程细化指令（补充参考）：** [`docs/Claude_Agent_任务指令_R2026a.md`](docs/Claude_Agent_%E4%BB%BB%E5%8A%A1%E6%8C%87%E4%BB%A4_R2026a.md)

**Claude Code 项目入口：** [`CLAUDE.md`](CLAUDE.md)

## 本地准备

建议本仓库克隆到 `C:\workspace\MC_OpenEnd`。

**电机参数权威来源：** [guojqbq/MC_RM108_Softswitch / main / parameters/motor](https://github.com/guojqbq/MC_RM108_Softswitch/tree/main/parameters/motor)。本项目使用**两个仓库**：`MC_OpenEnd` 存放新仿真模型，`MC_RM108_Softswitch` 仅提供源参数。若本机已有 `C:\workspace\MC_RM108_Softswitch\parameters\motor` 且与远端一致，可只读引用；否则由 Agent 自行读取或独立克隆源仓库，不得擅自更改源仓库文件。运行时记录参数来源 commit SHA；源文件包含脚本、常量参数、Ld/Lq 查表与星/角外特性，后者不等于开绕组/六相已验证外特性。

在仓库根目录启动 `claude`，让 Claude Code 阅读 `CLAUDE.md` 与完整任务指令。先检查 MATLAB 版本、许可、库模块以及参数清单，之后按 P0～P4 分阶段实现和实测验证。**在未运行 MATLAB 之前，任何指标与 PPT 中的仿真结果都不得作为已验证结果。**

## 安全边界

仅允许仿真和研究脚本工作；不得改动原有电机控制固件、烧写硬件、控制电机台架，或覆盖参数来源目录。模型应保存为独立、可复现的 `.slx` 工程。