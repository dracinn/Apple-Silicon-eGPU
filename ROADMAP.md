# Roadmap

## Phase 0 — Research

- [ ] Map M1/T8103 USB-C, Thunderbolt and PCIe topology
- [ ] Identify existing Asahi Thunderbolt and PCIe work
- [ ] Identify DART/IOMMU requirements
- [ ] Build a hardware test matrix
- [ ] Define kernel versions and required patches

## Phase 1 — PCIe transport

- [ ] Detect Thunderbolt/USB4 infrastructure
- [ ] Establish PCIe tunneling
- [ ] Enumerate a simple PCIe device
- [ ] Verify PCIe configuration-space access
- [ ] Verify BAR mapping
- [ ] Verify MSI/MSI-X interrupts

## Phase 2 — DMA and device lifecycle

- [ ] Validate DART mappings
- [ ] Validate DMA allocation and synchronization
- [ ] Test device reset
- [ ] Test Thunderbolt hotplug
- [ ] Test suspend/resume
- [ ] Document failure modes

## Phase 3 — AMD eGPU

- [ ] Enumerate a supported AMD GPU
- [ ] Bind upstream amdgpu
- [ ] Validate VRAM/GART
- [ ] Validate command submission
- [ ] Validate display engine
- [ ] Validate Mesa Vulkan
- [ ] Validate OpenGL
- [ ] Test GPU offload

## Phase 4 — Intel eGPU

- [ ] Determine supported Intel GPU generations
- [ ] Test xe where applicable
- [ ] Test i915 where required
- [ ] Validate Mesa ANV/Iris
- [ ] Test GPU offload

## Phase 5 — Integration

- [ ] Hotplug reliability
- [ ] Power management
- [ ] External displays
- [ ] Multi-GPU scheduling/offload
- [ ] Fedora Asahi compatibility
- [ ] Aurora/Bazzite compatibility
- [ ] Upstream applicable kernel changes

## Out of scope

- NVIDIA
- macOS
- A replacement GPU driver stack
