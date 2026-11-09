# Roadmap

## Phase 0 — Apple platform research

- [ ] Map M1/T8103 USB-C, USB4/Thunderbolt and PCIe topology
- [ ] Track Apple USB4/Thunderbolt host-controller work
- [ ] Identify existing Asahi/Aurora PCIe and DART/IOMMU support
- [ ] Build a hardware, enclosure, cable, firmware, kernel and Mesa test matrix
- [ ] Define minimum kernel/configuration requirements
- [ ] Reproduce the known Apple PCIe/tunnel recovery path before GPU testing

> Upstream note (2026-11-09): Aurora now tunnels PCIe over USB4 on the M4/T8132 ([aurora-silicon/linux#846](https://github.com/aurora-silicon/linux/pull/846)), but says PCIe tunnelling is not offered on M1/M2 ports: m1n1 does not bring up the PCIe-C controller and the kernel does not cold-init it there ([#1015](https://github.com/aurora-silicon/linux/pull/1015)). Treat T8103 PCIe-C cold init as an open Phase 0/1 dependency. Use the T8132 path as the reference implementation. See RESEARCH.md.

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

**Gate:** a generic PCIe endpoint must remain usable through enumeration, resource assignment, interrupt delivery and link recovery before attaching a GPU.

> Upstream note (2026-11-09): the T8132 tunnel DART bypasses only the MSI streams (`apple,bypass-sids`) and gives devices behind USB4 translated DMA domains ([#846](https://github.com/aurora-silicon/linux/pull/846)). MSI/MSI-X and DART validation should use that configuration, not identity mapping.

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

**Gate:** a generic endpoint must complete DMA and reset/recovery tests without silent corruption or IOMMU faults.

> Upstream note (2026-11-09): the PCIe-C to NHI sleep ordering now exists as a device link ([aurora-silicon/linux#1251](https://github.com/aurora-silicon/linux/pull/1251)). The suspend/resume ordering test should check that it is present on the target board.

## Phase 3 — AMD eGPU

- [ ] Enumerate a supported AMD GPU
- [ ] Confirm PCI bridge windows and all required GPU BARs
- [ ] Bind upstream amdgpu without Apple-specific GPU-driver changes where possible
- [ ] Validate external/removable-device classification
- [ ] Validate VBIOS discovery path for an external GPU
- [ ] Validate VRAM/GART and DMA
  - Upstream note (2026-11-09): an NVIDIA GPU behind the Apple tunnel root port (`106b:100f`, M1 Max) reset the machine with WC BAR mappings and worked with them disabled (n=1; [NVIDIA/open-gpu-kernel-modules#1482](https://github.com/NVIDIA/open-gpu-kernel-modules/pull/1482)). Check how amdgpu maps the VRAM BAR (WC vs. device) through the tunnel, and check 16K-page/2M IOVA alignment assumptions.
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
