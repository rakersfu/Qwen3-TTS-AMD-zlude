# AMD + ZLUDA（Windows）安装说明（中文整理版）

适用于：Stable Diffusion、ComfyUI、Z‑Image、Qwen3‑TTS、任意 PyTorch 程序

适配显卡：RDNA2（RX 6600 / 6700 / 6750 / 6800 / 6900 系列）

---

## 重要提示（先读）
- 本指南针对在 Windows 下使用 ZLUDA 让 PyTorch/ComfyUI 等项目跑在 AMD RDNA2 上的流程。
- 强制要求：HIP SDK 必须是 6.2.4；其他版本（如 HIP 7.x）会导致 ZLUDA 失效并触发杀毒误报。
- 推荐 Python 版本：3.11（例如 3.11.9）。Python 3.12+ 可能导致部分 wheel 无法安装。

---

## 目录
1. 前置条件
2. 下载与安装步骤（详细）
3. Windows 系统注意事项
4. 第一次启动与推荐启动参数
5. 最简安装步骤速览
6. 后续可选说明

---

## 1. 前置条件
- 已安装并启用符合要求的 AMD 显卡驱动（任意 2026 年之后的 Adrenalin 驱动即可）。
- 确认目标 GPU 属于 RDNA2（例如 RX 6600/6700/6750/6800/6900 系列）。
- 在 Windows 上有管理员权限以便安装 SDK/库与修改 Defender/SAC 设置。

---

## 2. 下载与安装步骤（详细）
以下按顺序执行：

### 2.1 安装 AMD 显卡驱动（Adrenalin）
- 使用任意 2026 年之后的 Adrenalin 驱动即可，无需 Pro 驱动或特殊版本。

### 2.2 安装 HIP SDK（必须是 6.2.4）
- 非常重要：只能安装 HIP SDK 6.2.4。
- HIP 7.x 会导致 ZLUDA 完全失效，且可能触发杀毒软件误报。
- 建议安装路径（示例）：

```text
C:\Program Files\AMD\ROCm\6.2\
```

### 2.3 安装 rocBLAS / Tensile 优化库（关键）
说明：HIP SDK 默认不包含针对 RDNA2 的 GEMM/Tensile 内核。如果缺少 Tensile 内核，会在矩阵乘法（matmul）处报错：

```text
CUBLAS_STATUS_NOT_SUPPORTED
```

必须安装与显卡架构匹配的 Tensile 包。

常见对应关系：
- RX 6800 / 6900 / 6750 GRE → gfx1030
- RX 6700 XT → gfx1031
- RX 6600 → gfx1032

步骤：
1. 从社区（例如 likelovewant 等）下载对应的 rocBLAS/Tensile 包，文件名类似：

```text
rocm.gfx1031.for.hip.sdk.6.2.4.*.7z
```

2. 解压到 HIP SDK 的 rocblas 库目录，例如：

```text
C:\Program Files\AMD\ROCm\6.2\bin\rocblas\library\
```

注意：文件必须匹配 GPU 架构与 HIP 6.2.4。

### 2.4 安装 Python 3.11
- 推荐：Python 3.11.9。
- 如果系统默认 Python 版本不是 3.11，安装 ComfyUI 时要手动指定 Python 路径。

### 2.5 安装 ComfyUI + ZLUDA（推荐 patientx 的 fork）
推荐使用 patientx/ComfyUI-Zluda fork，它能自动完成多项配置：
- 创建虚拟环境（venv）
- 安装 ZLUDA‑patched 的 PyTorch 2.x（cu118 相容包）
- 放置 ZLUDA 二进制文件到 `ComfyUI-Zluda\\zluda\\`
- 配置启动器 `comfyui-n.bat`
- 自动应用 cuDNN wrapper（用于解决 VAE decode / conv 错误）

使用方法（在仓库目录下）：

```text
install.bat
# 或
install-n.bat
```

备注：ZLUDA 推荐版本示例：3.9.5 nightly。
ZLUDA 的编译缓存位置（请不要删除，否则每次都要重新编译）：

```text
%LOCALAPPDATA%\\ZLUDA
```

---

## 3. Windows 系统注意事项（非常重要）

1) Smart App Control (SAC)
- SAC 会拦截并阻止 ZLUDA 的运行（例如会拦截 nccl.dll），即使添加 Defender 排除也可能无效。
- 解决办法：

  Windows 安全中心 → App & browser control → Smart App Control → Off

- 警告：关闭 SAC 后若重新开启可能需要重置 Windows，请谨慎操作。

2) Windows Defender 误报
- Defender 可能把 `nccl.dll` 等文件误报为 `Trojan:Win32/Pomal!rfn`。
- 解决办法：
  - 给 ComfyUI-Zluda 文件夹添加 Defender 排除；
  - 或者在误报项中选择“允许”。

---

## 4. 第一次启动 ComfyUI（ZLUDA 模式）与推荐参数

启动命令：

```text
comfyui-n.bat
```

注意事项：
- 第一次生成图像时会卡 10–15 分钟：这是 ZLUDA 为 GPU 编译内核，只发生一次，后续运行将显著加快。

- 推荐启动参数（对于 RDNA2 已内置，但列在此以便参考）：

```text
--use-quad-cross-attention
--reserve-vram 0.9
--disable-async-offload
--disable-pinned-memory
```

并设置环境变量：

```text
MIOPEN_FIND_MODE=2
```

这些参数可避免出现的常见错误：cuDNN 错误、conv_transpose1d 错误、VAE decode 错误、matmul 错误等。

- 额外说明：`comfyui-n.bat` 每次启动会自动执行 `git pull`（优点：自动更新；缺点：上游更新可能影响兼容性）。如果希望固定版本，可以直接用：

```text
zluda.exe -- python main.py
```

---

## 5. 最简安装步骤速览

1. 安装 AMD 显卡驱动
2. 安装 HIP SDK 6.2.4
3. 安装对应 GPU 的 rocBLAS/Tensile
4. 安装 Python 3.11
5. 安装 patientx/ComfyUI-Zluda
6. 关闭 SAC 并为 ComfyUI-Zluda 添加 Defender 排除
7. 使用 `comfyui-n.bat` 启动（首次编译需 10–15 分钟）

---

## 6. 后续（可选）
- 如果需要，我可以继续整理：
  - Qwen3‑TTS（语音克隆）在 ZLUDA 上的中文安装说明
  - 任意 PyTorch 程序在 ZLUDA 上运行的中文教程
  - AMD 6750 GRE 最佳配置指南

---

如果你希望我把 README 调整为更详细的英文版本、加入安装脚本示例、或分成多个文档（例如 docs/ 目录），告诉我我会继续完善.
