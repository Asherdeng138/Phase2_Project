# Phase 2 — Beamforming for mmWave Massive MIMO
# 第二阶段：毫米波大规模 MIMO 波束成形

MATLAB experiments on array patterns, spatial precoding, spectral efficiency, energy efficiency, and runtime in mmWave MIMO systems. Methods include digital/analog beamforming, hybrid sparse precoding, Dinkelbach optimization, and PCEM variants.

本项目使用 MATLAB 研究毫米波 MIMO 中的阵列方向图、空间预编码、频谱效率、能量效率及算法运行时间，包含数字/模拟波束成形、混合稀疏预编码、Dinkelbach 优化和不同版本的 PCEM 方法。

The original project notes associate this code with **“Enhancing 5G: Energy Efficient Solutions for mmWave Massive MIMO Systems.”** This guide describes the stored code without claiming independent reproduction of every result.

原项目说明将代码与 **“Enhancing 5G: Energy Efficient Solutions for mmWave Massive MIMO Systems”** 研究关联。本说明介绍仓库中的实际代码，不代表全部实验结果均已独立复现。

Background project / 基础项目：[Phase 1 — modulation and diversity](https://github.com/Asherdeng138/Phase1project).

## 1. Archive and entry-point map / 压缩包与实验入口

The source is stored in **eight independent ZIP/7z archives** under [`Phase_2_project-main/`](Phase_2_project-main/). Extract an archive before running its `.m` files. Repeated filenames in different archives may contain different implementations.

源代码存放在 [`Phase_2_project-main/`](Phase_2_project-main/) 下的 **8 个独立 ZIP/7z 压缩包**中，运行前需要解压。不同压缩包中的同名文件可能包含不同实现。

| Archive / 压缩包 | Extracted folder → entry / 解压目录 → 入口 | Experiment / 实验内容 |
| --- | --- | --- |
| `ArrayFactor_Plotting_UniformLinearArray.7z` | `ArrayFactor_Plotting_UniformLinearArray/fig218.m` | Uniform linear array patterns in different coordinates. / 均匀线阵在不同坐标下的方向图。 |
| `BeamPattern_Plot_For_Different_Steering_Angles.7z` | `BeamPattern_Plot_For_Different_Steering_Angles/eq28.m` | Beam patterns for five steering angles. / 五个转向角的方向图。 |
| `Spatially_Precoding.7z` | `Spatially_Precoding/fig3.m` | Unconstrained, hybrid sparse, and beam-steering precoding. / 无约束、混合稀疏与波束转向预编码比较。 |
| `Comparative Performance of Beamforming Methods.zip` | `PCEM_backup/fig6.m` | SE/EE comparison of digital, analog, Dinkelbach, brute-force, and single-user PCEM/brute-force methods. / 多种方法的频谱效率与能量效率比较。 |
| `Energy Efficiency vs. Iteration for Different SNR Levels.zip` | `Fig3/fig3.m` | Energy-efficiency histories at several SNR values. / 不同 SNR 下的能量效率迭代曲线。 |
| `Examining PCEM Performance Under 120° and 360° Coverage.zip` | `tri/fig6.m` | PCEM with 120°/360° coverage and random/fixed distances. / 不同覆盖角度、随机与固定距离的 PCEM 比较。 |
| `Impact of User Distance Variations on PCEM Performance.zip` | `fix&randow_dis/fig6.m` | Random versus fixed average user distances. / 随机距离与固定平均距离比较。 |
| `Runtime Analysis for Varying Array Size and SNR.zip` | `tic_toc/tic_toc.m` | Runtime versus transmit-array size and SNR. / 不同发射阵列规模和 SNR 下的运行时间。 |

The spelling `fix&randow_dis` matches the archive. Some archives also contain `.fig` figures, `.jpg` images, `.mat` workspace snapshots, and `.asv` editor backups. Use `.m` entry points to rerun experiments; existing figures are not evidence of a new successful run.

表中保留了原始目录名 `fix&randow_dis`。部分归档还有 `.fig` 图窗、`.jpg` 图片、`.mat` 工作区快照和 `.asv` 编辑器备份。重新运行实验应从 `.m` 入口开始；已有图片不能代替本次运行验证。

## 2. Dependencies / 运行依赖

| Component / 组件 | When needed / 使用场景 |
| --- | --- |
| MATLAB | Intended runtime. Original notes recommend R2021a or newer; complete cross-version compatibility has not been verified. / 主要运行环境；原说明建议 R2021a 或更新版本，但尚未验证完整跨版本兼容性。 |
| [CVX](https://cvxr.com/cvx/) | `DinkelbachMethod.m` uses `cvx_begin`/`cvx_end`. Install a MATLAB-compatible distribution and run `cvx_setup`. / Dinkelbach 方法依赖 CVX，需安装并执行 `cvx_setup`。 |
| `lambertw` | Required by several PCEM variants; check Symbolic Math Toolbox in MATLAB. The runtime archive's PCEM variant uses `fminbnd` instead. / 多个 PCEM 版本依赖 Lambert W 函数；MATLAB 中需检查 Symbolic Math Toolbox。运行时间实验的 PCEM 版本使用 `fminbnd`。 |
| ZIP/7z extractor / 解压工具 | MATLAB `unzip` supports ZIP; use a compatible extractor such as 7-Zip for 7z. / ZIP 可用 MATLAB `unzip`；7z 需使用兼容解压工具。 |
| GNU Octave | A subset is verified below; full MATLAB/CVX compatibility is not established. / 下文列出的部分已验证，不能保证替代完整 MATLAB/CVX 工作流。 |

Before large optimization runs, check `which cvx_begin` and `which lambertw`. Several entry scripts create figures or progress dialogs.

运行较大的优化实验前，请使用 `which cvx_begin`、`which lambertw` 检查依赖。多个入口脚本会创建图窗或进度条。

## 3. Quick start / 快速开始

### A. Clone and extract / 克隆并解压

```bash
git clone https://github.com/Asherdeng138/Phase2_Project.git
cd Phase2_Project

# Requires 7z; a desktop archive manager works too.
# 需要安装 7z，也可使用图形化解压工具。
7z x 'Phase_2_project-main/ArrayFactor_Plotting_UniformLinearArray.7z' -oexperiments/array-factor
7z x 'Phase_2_project-main/Spatially_Precoding.7z' -oexperiments/spatial
```

Use a separate destination for each archive; do not overwrite extracted directories containing your edits.

每个压缩包使用独立目标目录，不要覆盖已有修改的解压目录。

### B. Start with an array-pattern plot / 先运行阵列方向图

From the repository root in MATLAB / 在 MATLAB 中从仓库根目录开始：

```matlab
cd(fullfile('experiments', 'array-factor', ...
    'ArrayFactor_Plotting_UniformLinearArray'));
run('fig218.m');
print(gcf, 'array-factor.png', '-dpng', '-r150');
```

This produces four subplots without CVX or PCEM dependencies. For headless Octave, run from the same extracted directory:

该实验生成四个子图，不需要 CVX 或 PCEM 相关依赖。无界面的 Octave 可在同一解压目录中运行：

```bash
octave --no-gui --quiet --eval "set(0,'defaultfigurevisible','off'); graphics_toolkit('gnuplot'); run('fig218.m'); print('array-factor.png','-dpng');"
```

The command requires an installed Octave runtime, gnuplot, and fonts. / 此命令需要已安装 Octave、gnuplot 和可用字体。

### C. Run the precoding comparison / 运行预编码比较

Start a fresh MATLAB/Octave session from the repository root / 从仓库根目录启动新的 MATLAB/Octave 会话：

```matlab
cd(fullfile('experiments', 'spatial', 'Spatially_Precoding'));
rng(55408);
run('fig3.m');
print(gcf, 'spatial-precoding.png', '-dpng', '-r150');
save('spatial-precoding.mat', 'snrValuesDB', ...
    'spectralEffOptimal', 'spectralEffHybrid', 'spectralEffBeam');
```

Defaults: 64 transmit/16 receive antennas, one stream, four RF chains, `-40:5:0` dB, and 100 trials per SNR. This is **900 channel realizations**, each evaluated by three methods. Each result array is `100 × 9`; the figure shows trial averages.

默认配置：64 发射/16 接收天线、1 个数据流、4 条 RF 链、`-40:5:0` dB、每个 SNR 重复 100 次，共 **900 个信道样本**，每个样本计算三种方法。各结果矩阵为 `100 × 9`，图中展示重复试验后的均值。

### D. Run a ZIP-based experiment / 运行 ZIP 中的实验

From the repository root in MATLAB / 从 MATLAB 中的仓库根目录执行：

```matlab
unzip(fullfile('Phase_2_project-main', ...
    'Comparative Performance of Beamforming Methods.zip'), ...
    fullfile('experiments', 'comparison'));
cd(fullfile('experiments', 'comparison', 'PCEM_backup'));
which cvx_begin
which lambertw

% Run after the dependencies above are available.
% 确认上述依赖可用后再运行，默认包含较多重复试验。
rng(55408);
run('fig6.m');
```

This complete comparison requires CVX and PCEM dependencies. Successfully running an individual digital/analog/brute-force class does not validate the whole `fig6.m` experiment.

完整比较需要 CVX 和 PCEM 依赖。单独运行数字、模拟或暴力搜索类，不代表整个 `fig6.m` 已通过验证。

## 4. Code structure / 代码结构

The usual flow is **parameters → channel generation → algorithm object → metric collection → averaging and plotting**.

典型流程为：**参数 → 信道生成 → 算法对象 → 指标收集 → 平均与绘图**。

| File/class / 文件或类 | Responsibility / 职责 |
| --- | --- |
| `fig3.m`, `fig6.m`, `tic_toc.m` | Configure and run experiment loops. / 配置参数并执行实验循环。 |
| `ChannelGeneration.m` | Generate channel matrices, array responses, and path gains; implementations differ by archive. / 生成信道矩阵、阵列响应和路径增益，不同归档的实现不同。 |
| `OptimalUnconstraint.m` | SVD-based unconstrained precoding/combining reference. / 基于 SVD 的无约束预编码与合并参考。 |
| `HybridSparsePrecoding.m` | Hybrid precoding/combining using orthogonal matching pursuit. / 使用正交匹配追踪实现混合预编码与合并。 |
| `BeamSteering.m` | Select steering-vector pairs using effective channel gains. / 根据有效信道增益选择转向向量对。 |
| `DigitalBeamforming.m`, `AnalogBeamforming.m` | Digital and analog baselines. / 数字与模拟波束成形基线。 |
| `BruteForce.m` | In the comparison archive, scans RF-chain counts with generated analog precoders, not every possible beamformer. / 在比较实验中扫描 RF 链数量并生成模拟预编码器，并非枚举全部可能的波束方案。 |
| `DinkelbachMethod.m` | Iterative energy-efficiency optimization with CVX subproblems. / 通过 CVX 子问题迭代优化能量效率。 |
| `PCEM*.m` | Power/energy-efficiency optimization for each experiment's user/channel assumptions. / 对应不同用户和信道假设的功率及能量效率优化版本。 |

**Many constructors immediately run their algorithms.** Changing a property after construction does not automatically refresh computed metrics. For different hardware parameters, inspect the class lifecycle and explicitly recompute through the relevant method, usually `launch`, where appropriate.

**很多类在构造对象时就立即运行算法。** 构造后修改属性，不会自动更新已计算指标。调整硬件参数时，应检查类的调用流程，在适当情况下通过相应方法（通常为 `launch`）重新计算。

## 5. Parameters, metrics, and outputs / 参数、指标与输出

| Setting/result / 参数或结果 | Meaning / 含义 |
| --- | --- |
| `numberTransmitAntennas`, `numberReceiveAntennas` | Array sizes. The spatial archive uses the spelling `numberRecieveAntennas`; match its actual key. / 天线数；空间预编码版本使用 `numberRecieveAntennas` 拼写，需匹配实际键名。 |
| `numberDataStreams`, `numberRFChains` | Data-stream and RF-chain counts. / 数据流数与 RF 链数。 |
| `numberCluster`, `numberRayPerCluster`, `angularSpread` | Multipath channel configuration. / 多径信道参数。 |
| `snrValuesDB`, `SNR_dB_values`, `tryNumber` | SNR sweep and Monte Carlo repetitions. / SNR 范围与蒙特卡洛重复次数。 |
| `P_RF_chain`, `P_PS`, `P_CP`, `P_T`, `P_R`, `eta_PA` | Hardware power and amplifier-efficiency assumptions. / 硬件功耗及功率放大器效率假设。 |
| SE / spectral efficiency / 频谱效率 | Reported in bits/s/Hz. / 通常表示为 bits/s/Hz。 |
| EE / energy efficiency / 能量效率 | Rate divided by modeled power; inspect bandwidth/rate normalization before interpreting bits/Joule labels. / 相应速率除以模型功耗；解释 bits/Joule 标签前应核对带宽与速率归一化。 |

Distance, coverage, and comparison scripts generally use `0:5:25` dB and 1,000 trials. The runtime script uses transmit arrays `[16, 32, 64, 128, 256]`, SNR values `[-5, 0, 10, 20]`, and 100 trials. Settings are assigned inside entry scripts, so command-window variables do not override those assignments.

距离、覆盖范围和比较实验通常使用 `0:5:25` dB、1000 次重复；运行时间实验使用 `[16, 32, 64, 128, 256]` 发射天线、`[-5, 0, 10, 20]` dB 和 100 次重复。参数在入口脚本内赋值，命令窗口设置同名变量不能覆盖这些赋值。

Results mainly remain in workspace variables and figures; save/export them explicitly. For reproducible comparisons, record random seed, software/toolbox versions, parameters, and channel-generation variant. Inspect `ChannelGeneration` rather than assuming it consumes every parameter-map entry.

结果主要保留在变量和图窗中，需要显式保存或导出。重复比较时应记录随机种子、软件和工具箱版本、参数及信道生成版本。参数表中有某个键，不代表当前 `ChannelGeneration` 实际使用了它。

## 6. Path conflicts / 路径冲突

**Do not add all extracted folders with `addpath(genpath(...))`.** Archives contain different versions of classes with the same names. Use one experiment directory per fresh session. Run `which ChannelGeneration -all` to verify the loaded class comes from the intended archive.

**不要用 `addpath(genpath(...))` 一次加入全部解压目录。** 不同实验包含同名但实现不同的类。建议每个实验使用独立会话和工作目录，用 `which ChannelGeneration -all` 确认实际加载来源。

Several entry scripts clear the workspace and open progress dialogs; save unrelated work first. Headless plotting of small examples does not establish headless compatibility of all progress-dialog workflows.

多个入口会清空工作区或打开进度窗口，请先保存其他工作。小型示例的无界面绘图通过，不代表所有带进度窗口的完整实验已验证无界面兼容性。

## 7. Verified scope / 已验证范围

Checks below passed on **GNU Octave 10.3.0**, using the original extracted `.m` files.

以下检查在 **GNU Octave 10.3.0** 中使用原样解压的 `.m` 文件通过。

| Check / 检查 | Evidence / 验证内容 |
| --- | --- |
| `fig218.m` | Finite normalized array factor and PNG export. / 阵列因子有限性、归一化与 PNG 导出。 |
| `eq28.m` | Five steering-angle curves, final curve bounds, and PNG export. / 五条转向角曲线、末条曲线数值范围及 PNG 导出。 |
| Spatial `fig3.m` | Full 900-channel sweep for three methods; expected array sizes, finite nonnegative values, PNG and MAT export. / 三种方法完整 900 信道扫描，结果尺寸、有限性、非负性与 PNG/MAT 输出。 |
| Comparison archive's `DigitalBeamforming`, `AnalogBeamforming`, `BruteForce` | Nine evaluations: three methods at 0, 10, 25 dB, with positive finite SE/EE. / 三种方法在 0、10、25 dB 共九次计算，SE/EE 为正且有限。 |

Full Dinkelbach/CVX runs, PCEM experiments, and the runtime sweep were **not validated**. Some PCEM variants need `lambertw`, absent from the checked environment. Gnuplot emitted plotting warnings, including an extra-legend-label warning for spatial precoding: numerical checks and image export passed, but exact legend layout was not validated. These checks establish execution and basic numerical sanity, not scientific accuracy or reproduction of publication results. The repository does not include an automated test suite.

完整 Dinkelbach/CVX、PCEM 实验和运行时间扫描**尚未验证**。部分 PCEM 版本需要已检查环境中缺失的 `lambertw`。gnuplot 有绘图警告，包括空间预编码图的额外图例标签警告：数值检查和图像导出通过，但未验证精确图例排版。这些检查证明可以执行并满足基本数值检查，不代表证明科学正确性或复现论文结果。仓库未提供自动测试套件。

## 8. Contact / 联系方式

Contact / 联系邮箱：[djc1311249833@163.com](mailto:djc1311249833@163.com).
