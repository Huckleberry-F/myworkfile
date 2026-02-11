# 案例背景说明与运行指令（扩展版）

本文档给出仓库中代表性算例的工程背景、验证目标与可直接复制的运行命令。

## 1. 快速使用约定

- 编译：

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j2
```

- 通用执行模板：

```bash
./build/fem_solver <input.inp> <output.vtk> 1.0 <dense|pcg>
```

## 2. 经典入门案例

### 2.1 `examples/classic_bar_tension.inp`
- **背景**：一维拉伸杆，验证基本线弹性、边界约束和集中载荷。
- **看点**：端部位移应与解析公式 \(u=FL/EA\) 同数量级。
- **命令**：

```bash
./build/fem_solver examples/classic_bar_tension.inp output/classic_bar.vtk 1.0 dense
```

### 2.2 `examples/classic_beam_tip_load.inp`
- **背景**：悬臂梁端部横向载荷，验证 B31 梁单元弯曲刚度。
- **看点**：端部挠度与 \(FL^3/(3EI)\) 量级一致。
- **命令**：

```bash
./build/fem_solver examples/classic_beam_tip_load.inp output/classic_beam.vtk 1.0 dense
```

### 2.3 `examples/classic_solid_patch.inp`
- **背景**：实体 patch 压缩，用于验证 C3D8 基本稳定性。
- **命令**：

```bash
./build/fem_solver examples/classic_solid_patch.inp output/classic_solid.vtk 1.0 dense
```

## 3. 新增扩展示例（本次补充）

### 3.1 `examples/shell_plate_s4_256e.inp`
- **背景**：薄板弯曲（S4），网格 16×16，共 256 单元。
- **目的**：验证壳单元路径、`*Shell Section`、边缘固支约束。
- **建议检查**：
  - 右边中点下挠是否明显；
  - 反力主要集中在固支边。
- **命令**：

```bash
./build/fem_solver --check-inp examples/shell_plate_s4_256e.inp
./build/fem_solver examples/shell_plate_s4_256e.inp output/shell_plate_256e.vtk 1.0 dense
```

### 3.2 `examples/beam_chain_nonlinear_120e.inp`
- **背景**：长细梁链（120 个 B31）几何非线性加载。
- **目的**：验证 `*Step, nlgeom=YES`、`*Static` 增量参数、`*Controls` 与迭代日志输出。
- **建议检查**：
  - 终端应有每增量/每迭代的残量输出；
  - 随荷载增大，位移呈非线性增长。
- **命令**：

```bash
./build/fem_solver --check-inp examples/beam_chain_nonlinear_120e.inp
./build/fem_solver examples/beam_chain_nonlinear_120e.inp output/beam_chain_120e.vtk 1.0 pcg
```

### 3.3 `examples/contact_press_blocks_512e.inp`
- **背景**：上下双块压缩接触（C3D8），总计 512 单元，含摩擦。
- **目的**：验证接触主从面、法向罚函数、摩擦切向处理与非线性步进。
- **建议检查**：
  - 迭代初期残量较大，收敛后下降；
  - 启用 FIELD 输出频率时会产生多帧 VTK。
- **命令**：

```bash
./build/fem_solver --check-inp examples/contact_press_blocks_512e.inp
./build/fem_solver examples/contact_press_blocks_512e.inp output/contact_512e.vtk 1.0 pcg
```

## 4. 大规模案例建议

- `examples/cantilever_solid_2000e.inp`：2000 单元悬臂梁。
- 建议优先使用 `pcg`，并先执行 `--check-inp` 再算。

```bash
./build/fem_solver --check-inp examples/cantilever_solid_2000e.inp
./build/fem_solver examples/cantilever_solid_2000e.inp output/cantilever_2000e.vtk 1.0 pcg
```

## 5. 回归测试建议顺序

1. `classic_bar_tension`（最小线性）
2. `classic_beam_tip_load`（梁）
3. `shell_plate_s4_256e`（壳）
4. `beam_chain_nonlinear_120e`（非线性）
5. `contact_press_blocks_512e`（接触）
6. `cantilever_solid_2000e`（大规模）

这样可以逐级定位问题：从单元级、到非线性、到接触、到规模化性能。
