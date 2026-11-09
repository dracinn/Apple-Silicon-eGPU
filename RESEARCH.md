# Research Sources

This project uses the Apple Silicon Linux repositories already being watched for the broader M-series Linux development work as upstream research sources. They are reference material for architecture, device bring-up, kernel changes, boot/runtime behavior, and downstream integration.

## Current upstream picture

As of October 2026, Apple M1 PCIe and DART support are established Linux platform capabilities, while Apple USB4/Thunderbolt support remains active development work. Asahi's M1 support matrix lists PCIe from Linux 5.16 and DART from 5.15, while Thunderbolt and DP Alt Mode remain WIP. Aurora's current M1 notes likewise describe PCIe host support as configured while USB4/Thunderbolt remains driver/development work.

A September 2026 kernel series adds initial USB4/Thunderbolt support for Apple M1/M2/M3 SoCs. It describes Apple ACIO as containing the USB4 router, NHI and DART, with those functions dynamically populated around the ACIO co-processor. This is a major dependency for an Apple Silicon eGPU path because the external PCIe transport cannot be treated as a conventional PC Thunderbolt root complex.

### 2026-11-09 weekly watch (window 2026-11-02 to 2026-11-09)

Aurora Silicon Linux (`integration/aurora`):

- PCIe over USB4 now works on the M4 (T8132): [aurora-silicon/linux#846](https://github.com/aurora-silicon/linux/pull/846) (merged 2026-11-04) adds the T8132 PCIe-C tunnel controller ([a49f4a30a0](https://github.com/aurora-silicon/linux/commit/a49f4a30a0)) and tunnel DART support ([8b874b6f0c](https://github.com/aurora-silicon/linux/commit/8b874b6f0c)). An i226 Ethernet controller in a dock works behind it. DART details: a new `apple,bypass-sids` property bypasses only the reserved streams that carry endpoint MSIs to the AIC. Devices behind a USB4 port are treated as untrusted and get DMA (translated) domains, not identity domains. The commit notes that the i226 faulted on its first descriptor fetch under an identity domain. This is the first PCIe endpoint working through an Apple USB4 tunnel in a watched tree, and it is the closest reference for Phase 1 and Phase 2.
- Not yet offered on M1/M2: [aurora-silicon/linux#1015](https://github.com/aurora-silicon/linux/pull/1015) ([f1de4c0d47](https://github.com/aurora-silicon/linux/commit/f1de4c0d47)) says m1n1 does not bring up the PCIe-C tunnel controller on M1 and M2, and the kernel cold-initialises it only on the generations the PCI driver admits. As a result, "PCIe tunnelling is not offered on these ports". For T8103 this makes PCIe-C cold init (in m1n1 or the kernel) the specific missing piece.
- Suspend/resume ordering: [aurora-silicon/linux#1251](https://github.com/aurora-silicon/linux/pull/1251) ([e242c9bfc8](https://github.com/aurora-silicon/linux/commit/e242c9bfc8)) adds an `add_links` hook to `tb_nhi_ops`. The Apple NHI uses it to create a device link from PCIe-C to the NHI, so PCIe-C suspends before the NHI and resumes after the NHI rebuilds the tunnel. This matches the ordering ACPI links give on PCs. The link is stateless: it orders system sleep but does not tie probe/unbind together.
- Tunnel power management: [aurora-silicon/linux#956](https://github.com/aurora-silicon/linux/pull/956) ([abba42ce56](https://github.com/aurora-silicon/linux/commit/abba42ce56)) keeps the T8132 PCIe-C link up through s2idle by watching the genpd domain (`s2idle_hold_domain`). [aurora-silicon/linux#892](https://github.com/aurora-silicon/linux/pull/892) ([cacda34c69](https://github.com/aurora-silicon/linux/commit/cacda34c69)) adds an NHI quirk that keeps the host router's links out of CLx.
- Teardown/removal: [aurora-silicon/linux#1319](https://github.com/aurora-silicon/linux/pull/1319) (merged 2026-11-09, fixing issue [#1318](https://github.com/aurora-silicon/linux/issues/1318)) unregisters XDomains when the domain is removed and refuses XDomain DMA paths after unplug ([4a7c0cffef](https://github.com/aurora-silicon/linux/commit/4a7c0cffef)). This fixed a use-after-free Oops after an ACIO port restart. The fix is XDomain/thunderbolt-net, not PCIe, but the failure pattern is the same one the surprise-removal tests must cover. Follow-up [#1338](https://github.com/aurora-silicon/linux/pull/1338) is still open.
- Other USB4 bring-up: T6040 (M4 Pro) gets ACIO v2 router and Type 7 NHI support, plus a fix for swapped NHI ring interrupts ([aurora-silicon/linux#1220](https://github.com/aurora-silicon/linux/pull/1220), [3cb5397c7d](https://github.com/aurora-silicon/linux/commit/3cb5397c7d)); only XDomain/thunderbolt-net was tested. M5 Max (J716c) USB4/TB5 with a PCIe-C tunnel is open as [aurora-silicon/linux#1323](https://github.com/aurora-silicon/linux/pull/1323).
- PCIe link recovery and MSI (internal ports): [aurora-silicon/linux#1069](https://github.com/aurora-silicon/linux/pull/1069) ([d48ca539f4](https://github.com/aurora-silicon/linux/commit/d48ca539f4)) handles an endpoint that does not train. It re-asserts PERST# with first-bring-up timings and restarts the LTSSM, at most twice. [deaac66d82](https://github.com/aurora-silicon/linux/commit/deaac66d82) programs the MSI map of every listed root port so they share one allocator. Open [#1311](https://github.com/aurora-silicon/linux/pull/1311) attaches every internal-host power domain and keeps them on in s2idle.

Asahi:

- [AsahiLinux/m1n1@980fbbf55d](https://github.com/AsahiLinux/m1n1/commit/980fbbf55d) ([#757](https://github.com/AsahiLinux/m1n1/pull/757), fixes [#756](https://github.com/AsahiLinux/m1n1/issues/756)): m1n1 PCIe init did not advance `config_base` past non-present `pci-bridge` nodes, so link-speed setup on later ports was wrong (M1/M2 Mac mini, M1 iMac). This affects internal ports only, but it shows that m1n1's per-port PCIe setup is board-sensitive.
- AsahiLinux/linux: `bits/270-thunderbolt` was re-merged into `asahi-wip` on 2026-11-03 (a rebase; no new Thunderbolt commits identified). No other eGPU-layer changes.

Kernel patch series (patchwork):

- New: "PCI: apple: Integrate pwrctrl API" v2 ([linux-pci series 1201695](https://patchwork.kernel.org/series/1201695/), 2026-11-04). It moves Apple PCIe per-port power-up (Wi-Fi, SD reader, xHCI; t600x DT example) onto the pci-pwrctrl framework. It covers internal ports, but it changes how `pcie-apple` sequences port power.
- No new revision this week of "Initial USB4/Thunderbolt support for Apple M1/M2/M3 SoCs" ([v2, series 1159178](https://patchwork.kernel.org/series/1159178/), 2026-09-06) or "Apple SoC CIO (USB4/Thunderbolt) reset controller" ([v3, series 1178972](https://patchwork.kernel.org/series/1178972/), 2026-10-03).

NVIDIA (diagnostic only):

- [NVIDIA/open-gpu-kernel-modules#1482](https://github.com/NVIDIA/open-gpu-kernel-modules/pull/1482), [#1481](https://github.com/NVIDIA/open-gpu-kernel-modules/pull/1481) and [#1480](https://github.com/NVIDIA/open-gpu-kernel-modules/pull/1480) were opened 2026-10-23/24; the author followed up on 2026-11-04. They report an RTX 5060 Ti behind the Apple Thunderbolt/USB4 tunnel root port (`106b:100f`) on a 16-inch M1 Max running a 16K-page kernel. Findings:
  - The first init with write-combined (WC) GPU BAR mappings reset the machine. With WC disabled (`PDB_PROP_CL_DISABLE_IOMAP_WC`, BAR mapped as device memory), CUDA, a 1 GiB allocation and llama.cpp ran with no Xid.
  - 16K pages broke page-size selection (#1480).
  - DMA submaps must stay 2M-aligned on non-4K arm64 page sizes (#1481).
  - All results are n=1 on one machine. The PR text does not say which kernel tree supplied the M1 Max PCIe tunnel, and this is not reconciled with Aurora's statement that M1-generation ports do not offer PCIe tunnelling.
  - For the AMD path, this makes WC mapping of the VRAM BAR through the Apple tunnel a specific item to validate, along with 16K-page and IOVA-alignment assumptions.
- [#972](https://github.com/NVIDIA/open-gpu-kernel-modules/pull/972): no activity this week (last updated 2026-10-03).

No actionable changes this week in: aurora-silicon/m1n1, aurora-silicon/u-boot, AsahiLinux/docs, AsahiLinux/asahi-installer, ublue-os/bazzite (x86 kernel bumps and desktop fixes only).

## Critical eGPU precedent: non-standard ARM PCIe

Although NVIDIA is not a target of this project, the NVIDIA RTX 3090 work is an important architectural reference because it demonstrates that a discrete high-end PCIe GPU can operate on a non-standard Arm platform when platform and GPU-driver DMA assumptions are handled correctly.

- NVIDIA/open-gpu-kernel-modules#972 — Fixes for non-standard Arm SoC PCIe integrations
- aurora-silicon/linux#8 — M1 Thunderbolt/PCIe and recovery work

### Lessons from the NVIDIA precedent

The important lesson is not to copy NVIDIA code. It is to treat these as first-class validation points:

- DMA coherency must be detected rather than assumed.
- CPU/device cache maintenance must be correct.
- IOMMU translation must agree with GPU-visible addresses.
- ARM64 page-size/alignment assumptions can expose failures that are hidden on conventional systems.
- External PCIe links can have initialization and recovery behavior unlike internal PCIe.

## Apple Silicon transport

The Apple side is now the highest-priority dependency.

### Aurora Silicon

Aurora's Linux tree and m1n1 work are important because they cover Apple-specific PCIe, Thunderbolt/USB4, DART and recovery behavior. The organization continues active work on its Linux, m1n1 and U-Boot trees.

Track especially:

- Apple PCIe and external-I/O changes
- USB4/Thunderbolt NHI/router work
- DART/IOMMU and DMA changes
- PCIe tunnel recovery
- suspend/resume and parent/child ordering
- interrupt/device-tree changes
- M1/T8103/J313 support

### Asahi Linux

Asahi remains the primary upstream reference for Apple Silicon Linux hardware support.

Track:

- AsahiLinux/linux
- AsahiLinux/m1n1
- AsahiLinux/asahi-installer
- AsahiLinux/docs

For eGPU work, Linux and m1n1 are the highest-value sources. The docs/support matrix should be used to keep board-specific assumptions explicit.

### Apple USB4/Thunderbolt work

The September 2026 kernel series for M1/M2/M3 is especially relevant. It describes:

- ACIO as the parent block
- USB4 router hardware in ACIO
- NHI and DART within ACIO
- dynamic child-device population around the ACIO co-processor
- Apple-specific device-tree representation

This should be treated as a prerequisite branch of the project, not a detail to solve after the GPU is already attached.

## PCIe resource allocation and BARs

PCIe resource assignment is a distinct bring-up gate.

Linux has ongoing fixes for external PCIe GPUs where a bridge can receive an insufficient memory window for the GPU's BAR requirements. This means a GPU that enumerates correctly can still fail before amdgpu initialization because the PCI hierarchy cannot provide the required resources.

Project rule:

1. enumerate the hierarchy
2. inspect bridge windows
3. inspect endpoint BAR requirements
4. assign/reassign resources
5. only then debug amdgpu

Large BAR/ReBAR is not a prerequisite. First-light testing should prove that the GPU works with a resource layout the Apple/USB4 hierarchy can reliably provide.

## AMD external-GPU behavior

AMD is the first GPU target because upstream amdgpu already contains explicit external-device handling.

A recent amdgpu change disables runtime PM for externally attached dGPUs and notes that pci_is_thunderbolt_attached() does not cover all USB4 topologies. It instead accounts for removable-device state. This is directly relevant to Apple USB4 because Apple may not present the same bridge metadata as an Intel Thunderbolt hierarchy.

Track:

- external/removable classification
- runtime PM policy
- external VBIOS discovery
- switcheroo registration behavior
- GPU reset and PCIe error recovery
- large-BAR/resource behavior

The goal is to consume these behaviors, not replace them with an Apple-specific AMD driver.

## PCIe error handling and hotplug

External GPUs must be treated as removable PCIe devices.

Required tests include:

- link retraining
- transient PCIe completion failures
- AER reporting
- PCIe error recovery
- GPU reset/rebind
- surprise removal
- safe device teardown
- suspend/resume
- runtime PM

Linux Thunderbolt itself has explicit hotplug/tunnel handling, including queued hotplug work and external DP-resource management. Apple USB4 must eventually reach equivalent lifecycle behavior.

## Intel

No Apple Silicon + Intel Xe eGPU path is being treated as established yet.

Intel remains Phase 4 because the platform work should first prove that a generic external PCIe device and AMD GPU can survive the full lifecycle. Then compare xe/i915 requirements against the same transport, DART, DMA, reset and external-device layers.

## Downstream integration

### Omarchy Mac

Useful for practical Apple Silicon desktop integration and regression testing.

### Dakota / Bluefin Asahi

Useful for Fedora/Atomic kernel, Mesa and hardware-enable packaging.

### Bazzite

Bazzite is a downstream consumer, not the implementation location for Apple-specific PCIe/eGPU support. The standalone eGPU project should produce upstreamable kernel/platform work that can later be consumed by bazzite-asahi.

## NVIDIA references (diagnostic only)

NVIDIA remains explicitly out of scope. Its open driver and eGPU issue history is still useful for failure modes involving:

- non-coherent ARM64 DMA
- IOMMU/P2P address translation
- Thunderbolt/USB4 external-device detection
- ReBAR/BAR allocation
- surprise removal
- PCIe AER and completion timeouts
- external-GPU power management

Use these references to define tests and architecture, not as project dependencies.

## Monitoring rule

A watched repository becomes actionable for this project when it contains a meaningful change involving:

- USB4 or Thunderbolt
- PCIe host/endpoint support
- DART/IOMMU
- DMA coherency or cache maintenance
- MSI/MSI-X or interrupt routing
- BAR/resource handling
- PCIe reset or AER
- hotplug or surprise removal
- PCIe tunnel recovery
- power management
- device-tree descriptions of relevant hardware
- AMD or Intel GPU enablement on Apple Silicon
- Fedora/Atomic packaging required to consume those changes

Routine desktop changes are not eGPU progress unless they affect one of these layers.

## Primary references

- Aurora Silicon Linux: https://github.com/aurora-silicon/linux
- Aurora Silicon m1n1: https://github.com/aurora-silicon/m1n1
- Aurora Silicon U-Boot: https://github.com/aurora-silicon/u-boot
- Asahi Linux kernel: https://github.com/AsahiLinux/linux
- Asahi m1n1: https://github.com/AsahiLinux/m1n1
- Asahi docs: https://github.com/AsahiLinux/docs
- Bazzite: https://github.com/ublue-os/bazzite
