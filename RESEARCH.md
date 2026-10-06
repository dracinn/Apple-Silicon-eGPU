# Research Sources

This project uses the Apple Silicon Linux repositories already being watched for the broader M-series Linux development work as upstream research sources. These repositories are reference material for architecture, device bring-up, kernel changes, boot/runtime behavior, and downstream integration.

## Critical eGPU precedent: NVIDIA RTX 3090 on non-standard Arm PCIe

Although **NVIDIA is not a target of this project**, the successful RTX 3090 work is an important architectural reference because it demonstrates that a discrete high-end PCIe GPU can operate on a non-standard Arm platform when the platform and GPU-driver DMA assumptions are handled correctly.

Two pieces are especially relevant:

- [NVIDIA/open-gpu-kernel-modules#972](https://github.com/NVIDIA/open-gpu-kernel-modules/pull/972) — **"Fixes for non-standard Arm SoC PCIe integrations"**
- [aurora-silicon/linux#8](https://github.com/aurora-silicon/linux/pull/8) — **"Thunderbolt: M1/M2 Pro display, M1 PCIe and resume recovery"**

### What #972 teaches us

PR #972 is specifically aimed at Arm systems where normal PCIe assumptions do not hold. Its changes include:

- explicit DMA synchronization for non-coherent Arm systems
- detection of DMA coherency rather than assuming it
- correct CPU/device cache maintenance around DMA
- removal of assumptions that all Arm PCIe implementations provide standard cache-coherent behavior
- generic chipset handling needed by the NVIDIA driver on unusual Arm SoCs

The important lesson for this project is **not to copy NVIDIA code**. It is to treat DMA coherency and cache maintenance as first-class validation points for Apple Silicon PCIe.

### What Aurora Linux #8 teaches us

Aurora Silicon PR #8 is directly relevant to the Apple side of the problem. Its current series combines M1 Thunderbolt/PCIe work with recovery and suspend handling. The PR describes:

- M1/T8103 Thunderbolt display tunnels
- M1 PCIe enumeration
- IOMMU teardown
- PCIe tunnel recovery
- keeping healthy tunneled PCIe links through suspend-to-idle
- PCI parent/child resume ordering for tunneled devices
- Thunderbolt router/link recovery
- Apple DART command quiescing and restoration
- Apple PCIe tunnel state handling

The PR explicitly lists the original M1 work as including **"t8103/M1 DP tunnels, PCIe enumeration, IOMMU teardown"** and states that M1 PCIe support needs no module parameter.

### Combined implication

The RTX 3090 result should change how this project approaches bring-up:

**Apple Silicon platform/transport correctness + PCIe resource handling + DART/IOMMU + Arm DMA coherency + an existing Linux GPU driver**

is a more realistic path than treating the problem as "write an eGPU driver."

For AMD/Intel, the preferred endpoint remains the upstream Linux GPU stack:

- AMD: amdgpu + Mesa RADV/RadeonSI
- Intel: xe/i915 + Mesa ANV/Iris

The NVIDIA work is therefore a **validation precedent and diagnostic reference**, not a project target.

## Priority sources

### Aurora Silicon

Aurora Silicon is a high-priority source because its work spans newer Apple Silicon platforms and maintains its own Apple-specific Linux and bootloader trees.

- https://github.com/aurora-silicon/linux
- https://github.com/aurora-silicon/m1n1
- https://github.com/aurora-silicon/u-boot

For this project, watch especially for:

- Apple PCIe and external-I/O changes
- Thunderbolt/USB4 work
- DART/IOMMU and DMA changes
- interrupt and device-tree changes
- platform initialization needed before PCIe devices can be used
- changes that improve M1/T8103/J313 support
- PCIe tunnel recovery and suspend/resume changes

### Asahi Linux

Asahi remains the primary upstream reference for Apple Silicon Linux hardware support.

- https://github.com/AsahiLinux/linux
- https://github.com/AsahiLinux/m1n1
- https://github.com/AsahiLinux/asahi-installer
- https://github.com/AsahiLinux/speakersafetyd
- https://github.com/AsahiLinux/docs

For eGPU work, the kernel and m1n1 repositories are the most important. The installer/docs repositories are useful for identifying supported hardware, boot requirements, and reproducible development environments.

### Omarchy Mac

Omarchy Mac is a useful downstream integration/reference target for a practical Apple Silicon Linux desktop.

- https://github.com/omacom/omarchy-mac
- https://github.com/omarchy-mac/omarchy-mac-iso
- https://github.com/omarchy-mac/omarchy-pkgs-aarch64

Use it primarily to validate that eGPU support can eventually work in a real Apple Silicon desktop distribution rather than only in a development kernel.

### Dakota / Bluefin Asahi

Dakota and the Bluefin Asahi work are relevant downstream references for Fedora/Atomic-style Apple Silicon Linux integration.

Track them for:

- kernel packaging
- Asahi hardware enablement
- immutable/Atomic integration
- Mesa and graphics stack packaging
- hardware enablement that could affect eGPU deployment

### Bazzite

Bazzite is a downstream end goal rather than the place to implement Apple-specific PCIe/eGPU support.

- https://github.com/ublue-os/bazzite

Monitor Bazzite Apple Silicon work when it directly enables M-series Macs or consumes the kernel/Asahi capabilities this project produces. Do not duplicate Apple-specific kernel work in Bazzite.

## How these sources affect this project

Research from these repositories should feed the project in this order:

1. **Apple platform bring-up:** Aurora Silicon + Asahi Linux
2. **PCIe / Thunderbolt / DART / DMA:** Apple-specific kernel and m1n1 work
3. **Arm DMA coherency:** validate Apple Silicon behavior rather than assuming standard PCIe coherency
4. **GPU binding:** upstream Linux amdgpu first, then Intel xe/i915
5. **Userspace graphics:** upstream Mesa
6. **Desktop integration:** Omarchy Mac, Dakota/Bluefin Asahi
7. **Gaming-oriented downstream integration:** Bazzite

The project should reuse upstream work wherever possible. A change belongs here only when it is specifically required to make an external PCIe GPU work on Apple Silicon and is not already available upstream.

## Monitoring rule

A watched repository becomes actionable for this project when it contains a meaningful change involving:

- Thunderbolt or USB4
- PCIe host/endpoint support
- DART/IOMMU
- DMA coherency or cache maintenance
- MSI/MSI-X or interrupt routing
- BAR/resource handling
- PCIe reset or hotplug
- PCIe tunnel recovery
- power management
- device-tree descriptions of the relevant hardware
- kernel changes that materially improve external PCIe devices
- AMD or Intel GPU enablement on Apple Silicon
- Fedora/Atomic packaging required to consume those changes

Routine desktop changes should not be treated as eGPU progress unless they affect one of the layers above.
