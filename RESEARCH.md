# Research Sources

This project uses the Apple Silicon Linux repositories already being watched for the broader M-series Linux development work as upstream research sources. They are reference material for architecture, device bring-up, kernel changes, boot/runtime behavior, and downstream integration.

## Current upstream picture

As of October 2026, Apple M1 PCIe and DART support are established Linux platform capabilities, while Apple USB4/Thunderbolt support remains active development work. Asahi's M1 support matrix lists PCIe from Linux 5.16 and DART from 5.15, while Thunderbolt and DP Alt Mode remain WIP. Aurora's current M1 notes likewise describe PCIe host support as configured while USB4/Thunderbolt remains driver/development work.

A September 2026 kernel series adds initial USB4/Thunderbolt support for Apple M1/M2/M3 SoCs. It describes Apple ACIO as containing the USB4 router, NHI and DART, with those functions dynamically populated around the ACIO co-processor. This is a major dependency for an Apple Silicon eGPU path because the external PCIe transport cannot be treated as a conventional PC Thunderbolt root complex.

### Upstream watch — week of 2026-10-26 to 2026-11-02

- **First working PCIe-over-USB4 on Apple Silicon outside M1 (open, unmerged).** Aurora posted a four-PR stack for the M4 MacBook Air (T8132, J713), tested together with a CalDigit TS5 dock on the back port: 2 x 20 Gb/s USB4 link at boot and on hotplug in both orientations, the dock's Intel i226 Ethernet working over the PCIe tunnel (2.3 Gbit/s), s2idle with and without the dock. This is a generic (non-GPU) PCIe endpoint behind an Apple USB4 tunnel — the Phase 1 gate shape — on T8132, not M1/T8103.
  - [aurora-silicon/linux#843](https://github.com/aurora-silicon/linux/pull/843) — ATC Type-C PHY CIO (USB4/TBT) mode; only atcphy1 (back port) mapped; calibration baked into board dtsi from the ADT.
  - [aurora-silicon/linux#844](https://github.com/aurora-silicon/linux/pull/844) — ACIO/`thunderbolt_apple` T8132 host routers (T6030 layout; no RUN write, LSTX after RTKit); DROM/calibration in dtsi; DP tunnelling not enabled; LSTX pin hand-back on router stop is an open follow-up.
  - [aurora-silicon/linux#845](https://github.com/aurora-silicon/linux/pull/845) — tunnel DART fixes: MSIs arrive as interrupt-controller writes on reserved DART streams that must be set to bypass (`apple,bypass-sids`) after every reset; fault handling/exception scan fixed (v2.2, all streams); **devices behind USB4 tunnel DARTs now get DMA domains instead of identity/bypass**, which is the correct model for untrusted external devices.
  - [aurora-silicon/linux#846](https://github.com/aurora-silicon/linux/pull/846) — T8132 PCIe-C root ports: tunnel reset block moved (0x1088/0x108c, with posted-request counters), own reset table, per-bridge 32-vector MSI slices of a 512-vector window shared with the internal port.
  - eGPU relevance: MSI routing through DART bypass streams, DMA-domain policy for tunnelled devices and per-SoC tunnel reset offsets are exactly the Phase 1/2 transport details this project must verify on T8103; none of this is M1 evidence and none of it exercises a GPU, large BARs or surprise removal.
- **USB4 router late-bind fix (merged).** [aurora-silicon/linux#855](https://github.com/aurora-silicon/linux/pull/855) (integration/aurora) and [#858](https://github.com/aurora-silicon/linux/pull/858) (aurora-wip, which distribution kernels pin), commit [6c3055be2e](https://github.com/aurora-silicon/linux/commit/6c3055be2e025363b7ffbd0b8c6902ab3f193493): CD321x/tipd ports now take a USB4/Thunderbolt router that binds after the deferred-probe timeout. Root cause was Fedora's `dracut-asahi` initramfs carrying `tps6598x` but not `thunderbolt_apple`; with a LUKS prompt the port gave up on the router, ACIO never started, no NHI appeared and docks ran USB 2.0 only. **Fedora/Atomic packaging note:** missing `thunderbolt_apple` in the initramfs silently disables the whole USB4/PCIe tunnel path; record initramfs contents and boot timing in every test report. Selftest follow-up: [#861](https://github.com/aurora-silicon/linux/pull/861).
- **DART release/re-add freeze (open).** [aurora-silicon/linux#841](https://github.com/aurora-silicon/linux/pull/841) — `apple-dart-apif` (T6050/M5 Pro/Max) left a removed PCI device's SIDs enabled on a freed domain, so remove + rescan hard-froze the machine at first DMA; adds a blocking `release_domain`. Different DART variant from M1, but the same failure class as eGPU unplug/replug and reset/rebind: Phase 2 hotplug tests must include remove → rescan → DMA, and check that the M1 DART driver provides a `release_domain` or equivalent.
- **PCI function reset handling (merged).** [aurora-silicon/linux 5bc9ff6401](https://github.com/aurora-silicon/linux/commit/5bc9ff640132538e6c211c0d152bd3df40230335) — `hci_bcm4377` gains `reset_prepare`/`reset_done`; a reminder that PCI-core/sysfs function resets on Apple platforms need driver-aware quiesce/rebind (relevant to the reset/rebind tests).
- **m1n1 PCIe port init fix (merged).** [AsahiLinux/m1n1 980fbbf55d](https://github.com/AsahiLinux/m1n1/commit/980fbbf55dcda85634955dc1d6eaaae146297cb7) ([#757](https://github.com/AsahiLinux/m1n1/pull/757), closes [#756](https://github.com/AsahiLinux/m1n1/issues/756)) — on machines with unused internal PCIe ports (M1/M2/M2 Pro Mac mini, two-port M1 iMac) m1n1 did not advance `config_base` past absent `pci-bridge[0-3]` nodes, breaking link-speed configuration. Affects internal ports, not J313 (MacBook Air), but m1n1 version belongs in the test matrix for any Mac mini/iMac test host.
- **Related Arm USB4 precedent (not Apple).** MediaTek posted "add support platform nhi driver" ([patchwork series 1196950](https://patchwork.kernel.org/series/1196950/), 2026-10-30): a platform (non-PCI) NHI driver, dedicated wakeup IRQ, USB3 adapter link-up timeout and a fix for PCIe tunnel activation/teardown warnings. Useful as a second non-PCI-NHI consumer of the generic Thunderbolt stack alongside Apple ACIO.
- No new revision of the September 2026 Apple M1/M2/M3 USB4/Thunderbolt kernel series was found on patchwork this week (lore.kernel.org search was not reachable from the automation; patchwork only). NVIDIA/open-gpu-kernel-modules#972 had no activity.

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
