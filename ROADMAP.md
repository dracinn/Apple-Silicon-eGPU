# Roadmap

## Phase 0 — Apple platform research

- [ ] Map M1/T8103 USB-C, USB4/Thunderbolt and PCIe topology
- [ ] Track Apple USB4/Thunderbolt host-controller work
- [ ] Identify existing Asahi/Aurora PCIe and DART/IOMMU support
- [ ] Build a hardware, enclosure, cable, firmware, kernel and Mesa test matrix
- [ ] Define minimum kernel/configuration requirements
- [ ] Reproduce the known Apple PCIe/tunnel recovery path before GPU testing

## Phase 1 — PCIe transport and resources

- [ ] Detect and initialize Apple USB4/Thunderbolt infrastructure
- [ ] Establish a recoverable PCIe tunnel
- [ ] Enumerate a simple non-GPU PCIe endpoint
- [ ] Verify PCIe configuration-space access
- [ ] Verify PCI bridge hierarchy and upstream windows
- [ ] Verify endpoint BAR assignment
- [ ] Verify resource reassignment when BARs do not fit initially
- [ ] Verify MSI/MSI-X interrupts
- [ ] Verify PCIe AER reporting and recovery

> **Upstream note (2026-11-02):** Aurora's open M4/T8132 stack ([#843](https://github.com/aurora-silicon/linux/pull/843)–[#846](https://github.com/aurora-silicon/linux/pull/846)) reports a generic endpoint (dock i226 Ethernet) working over an Apple USB4 PCIe tunnel, including MSI delivery via DART bypass streams. This is evidence the Phase 1 gate is reachable on Apple ACIO, not a pass on M1/T8103; nothing here is ticked. Fedora/Atomic images must ship `thunderbolt_apple` in the initramfs or carry the late-router fix ([#855](https://github.com/aurora-silicon/linux/pull/855)/[#858](https://github.com/aurora-silicon/linux/pull/858)), otherwise the tunnel path never starts.

**Gate:** a generic PCIe endpoint must remain usable through enumeration, resource assignment, interrupt delivery and link recovery before attaching a GPU.

## Phase 2 — DART, DMA and device lifecycle

- [ ] Validate Apple DART/IOMMU mappings
- [ ] Determine actual PCIe DMA coherency behavior on M1/T8103
- [ ] Validate CPU/device cache maintenance and DMA synchronization
- [ ] Test DMA across representative buffer sizes and alignments
- [ ] Test device reset
- [ ] Test PCIe link retraining/recovery
- [ ] Test Thunderbolt/USB4 hotplug
- [ ] Test surprise removal safely
- [ ] Test suspend/resume and parent/child ordering
- [ ] Document failure modes and required instrumentation

> **Upstream note (2026-11-02):** [aurora-silicon/linux#845](https://github.com/aurora-silicon/linux/pull/845) moves USB4-tunnelled devices onto DART DMA domains (not identity/bypass); [#841](https://github.com/aurora-silicon/linux/pull/841) shows remove → rescan → first DMA can hard-freeze when a DART driver lacks a `release_domain`. Hotplug and reset tests here should include that sequence. Hardware validation still required.

**Gate:** a generic endpoint must complete DMA and reset/recovery tests without silent corruption or IOMMU faults.

## Phase 3 — AMD eGPU

- [ ] Enumerate a supported AMD GPU
- [ ] Confirm PCI bridge windows and all required GPU BARs
- [ ] Bind upstream amdgpu without Apple-specific GPU-driver changes where possible
- [ ] Validate external/removable-device classification
- [ ] Validate VBIOS discovery path for an external GPU
- [ ] Validate VRAM/GART and DMA
- [ ] Validate command submission
- [ ] Validate GPU reset/error recovery
- [ ] Validate Mesa Vulkan
- [ ] Validate OpenGL
- [ ] Test GPU offload
- [ ] Test external display through the eGPU
- [ ] Test runtime PM policy for externally attached GPUs

**Important:** large BAR/ReBAR is an optimization/compatibility dimension, not a prerequisite for first light. First prove stable operation with the resource layout the platform can actually provide.

## Phase 4 — Intel eGPU

- [ ] Determine supported Intel GPU generations
- [ ] Test xe where applicable
- [ ] Test i915 where required
- [ ] Validate Intel external-device lifecycle behavior
- [ ] Validate Mesa ANV/Iris
- [ ] Test GPU offload
- [ ] Test external display
- [ ] Compare Intel-specific reset, DMA and power-management requirements with AMD

## Phase 5 — Integration and upstreaming

- [ ] Hotplug reliability
- [ ] Surprise-removal handling
- [ ] Runtime power management
- [ ] Suspend/resume
- [ ] External displays
- [ ] Multi-GPU scheduling/offload
- [ ] Fedora Asahi compatibility
- [ ] Aurora compatibility
- [ ] Omarchy / Dakota / Bluefin Asahi compatibility
- [ ] Bazzite compatibility
- [ ] Upstream applicable Apple kernel changes
- [ ] Document a reproducible developer/test setup

## Out of scope

- NVIDIA as a supported target
- macOS
- A replacement GPU driver stack
- Reimplementing standard Linux PCIe, amdgpu, xe/i915, or Mesa functionality
