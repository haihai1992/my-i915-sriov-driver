# my-i915-sriov-driver

Intel i915 SR-IOV kernel modules for **Unraid**, built from
[strongtz/i915-sriov-dkms](https://github.com/strongtz/i915-sriov-dkms) and
packaged as a Slackware `.txz` that installs directly on an Unraid server.

Used together with the [my-unraid-vgpu-manager](https://github.com/hellomrli/my-unraid-vgpu-manager)
plugin, which downloads this package, installs it and manages virtual
functions (VFs) for VM passthrough.

## What this produces

```
out/i915-sriov-<ver>-<kernel>-Unraid-<build>.txz
out/i915-sriov-<ver>-<kernel>-Unraid-<build>.txz.md5
```

The package contains four modules for the target Unraid kernel:

| module                | location in package                                                     |
|-----------------------|-------------------------------------------------------------------------|
| `i915.ko.xz`          | `lib/modules/<kernel>/kernel/drivers/gpu/drm/i915/`                     |
| `kvmgt.ko.xz`         | `lib/modules/<kernel>/kernel/drivers/gpu/drm/i915/`                     |
| `xe.ko.xz`            | `lib/modules/<kernel>/kernel/drivers/gpu/drm/xe/`                       |
| `intel_sriov_compat.ko` | `lib/modules/<kernel>/updates/compat/`                                |

A Unraid `slab` compatibility patch (see `patches/`) is applied to the
strongtz source before building on 6.x kernels.

## Releases

The GitHub Actions workflow (`build-i915-sriov.sh` + `.github/workflows/build.yml`)
builds the driver **in the cloud**:

- **manual**: run the *Build i915 SR-IOV driver* workflow, choose a strongtz
  tag (or `latest`), the Unraid kernel release and the package build number
- **daily**: a scheduled run picks up the newest strongtz release automatically

The resulting package is attached to a Release whose tag equals the kernel
release (e.g. `6.18.44-Unraid`), so the plugin can find it by kernel.

## Local build

```bash
# requires a Linux box with gcc/make/git/curl/tar/xz
TARGET_KERNEL_VERSION=6.18.44 KERNEL_RELEASE=6.18.44-Unraid \
I915_SRIOV_REF=2026.08.12.1 \
./scripts/build-i915-sriov.sh
```

The script downloads the ich777 Unraid kernel source tree
(`linux-6.18.44-Unraid.tar.xz`) which contains the prebuilt `.config` and
`Module.symvers`, builds the four modules against it, then packages them.

## Installation on Unraid

Install the package and reload the module stack:

```bash
upgradepkg --install-new --reinstall i915-sriov-<ver>-6.18.44-Unraid-1.txz
depmod -a
modprobe i915 enable_guc=3 max_vfs=7
echo 7 > /sys/devices/pci0000:00/0000:00:02.0/sriov_numvfs
```

Add to the syslinux append line for persistent SR-IOV at boot:

```
intel_iommu=on i915.enable_guc=3 i915.max_vfs=7 module_blacklist=xe
```

**Do not passthrough the PF (00:02.0) to a VM** - it would crash all other
VFs. Only passthrough VFs (00:02.1 - 00:02.7).
