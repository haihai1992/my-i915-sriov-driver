# my-i915-sriov-driver

为 **Unraid** 编译的 **Intel i915 SR-IOV 驱动**，基于 [strongtz/i915-sriov-dkms](https://github.com/strongtz/i915-sriov-dkms) 源码构建，打包为可直接安装到 Unraid 服务器的 Slackware `.txz` 格式。

配合 [my-unraid-vgpu-manager](https://github.com/hellomrli/my-unraid-vgpu-manager) 插件使用——插件负责下载本包、安装并管理 VF（虚拟功能）供虚拟机直通。

## 功能

- 在 Intel iGPU（支持 SR-IOV 的型号）上启用虚拟功能（VF）
- 每个 VF 可作为一个独立 GPU 直通给虚拟机
- 基于 strongtz 的 DKMS 模块，包含 Unraid 6.x 内核的 slab 兼容补丁

## 编译的模块

- `i915.ko` — 带 SR-IOV 支持的 i915 驱动
- `kvmgt.ko` — Intel GVT-g / mdev 支持
- `xe.ko` — 新 Xe 驱动（默认 blacklist，避免与 i915 冲突）
- `intel_sriov_compat.ko` — SR-IOV 兼容层

## 构建产物

```
out/i915-sriov-<版本>-<内核>-Unraid-<构建号>.txz   (+ .md5)
```

## 云编译（GitHub Actions）

`.github/workflows/build.yml` 执行 `scripts/build-i915-sriov.sh`：

- **手动触发**：运行 *Build i915 SR-IOV driver* 工作流，填写 strongtz 版本（如 `2026.08.12.1`）、Unraid 内核版本和构建号（留空则自动取最新）
- **每日自动检查**：每天 03:30（UTC）同时检测 [strongtz/i915-sriov-dkms](https://github.com/strongtz/i915-sriov-dkms) 和 [ich777/unraid_kernel](https://github.com/ich777/unraid_kernel) 两个仓库，**只有当任一仓库发布新版本时才编译**；两个都没更新则跳过，不再空跑
- 自动从 ich777 内核仓库下载对应内核源码树，应用 Unraid slab 补丁后编译（补丁仅对含 `include/linux/slab.h` 的 strongtz 源码生效，旧版源码自动跳过）

构建产物附加到 **tag 等于内核版本** 的 Release（如 `6.18.44-Unraid`、`6.18.43-Unraid`）。

### 内核 ↔ strongtz 版本对应

strongtz 上游每个版本限定了支持的内核范围（`dkms.conf` 的 `BUILD_EXCLUSIVE_KERNEL`），构建脚本会自动校验并提前报错：

| Unraid 内核 | 应选 strongtz 版本 | 备注 |
|---|---|---|
| 6.12.x（如 `6.12.98-Unraid`，Unraid 7.1.x） | `2026.03.05.6` | `2026.05.03` 起 upstream 放弃 6.12 |
| 6.17 – 7.1 | `2026.08.12.1`（最新） | |

## 本地构建

```bash
# 需要 Linux 环境，装有 gcc/make/curl/tar/xz
I915_SRIOV_REF=2026.08.12.1 KERNEL_RELEASE=6.18.44-Unraid \
./scripts/build-i915-sriov.sh
```

## Release

当前构建：**2026.08.12.1** for **6.18.43–6.18.47-Unraid**（tag 与内核版本一致）。
