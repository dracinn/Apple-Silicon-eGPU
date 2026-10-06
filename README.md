# Apple Silicon eGPU

Linux eGPU support for Apple Silicon Macs, focused on AMD and Intel GPUs.

## Scope

- Apple Silicon Macs, initially M1/T8103 systems
- Linux / Asahi-based kernels
- AMD GPUs as the primary target
- Intel GPUs as a secondary target
- Existing upstream Linux GPU drivers wherever possible
- Thunderbolt/USB4 PCIe tunneling, PCIe enumeration, DART/IOMMU, DMA, interrupts, reset, hotplug and power management

## Explicitly out of scope

- NVIDIA support for the initial project
- macOS eGPU support
- Reimplementing mature AMD or Intel GPU drivers unnecessarily

## Architecture

Apple Silicon Mac → USB-C/Thunderbolt → PCIe tunneling → External GPU → Linux GPU driver → Mesa.

AMD uses amdgpu with RADV/RadeonSI. Intel uses xe/i915 with ANV/Iris.

The first milestone is PCIe device visibility through the Apple Silicon external I/O path; GPU rendering comes later.

## Development milestones

1. Document Apple Silicon PCIe and Thunderbolt topology
2. Establish reproducible hardware/firmware/kernel test matrix
3. Validate Thunderbolt device discovery
4. Validate PCIe enumeration through the external link
5. Validate DART/IOMMU mappings and DMA
6. Validate interrupts, BARs and device reset
7. Bring up an AMD GPU with upstream amdgpu
8. Validate Mesa/Vulkan/OpenGL rendering
9. Add hotplug and power-management handling
10. Investigate Intel GPU support
11. Validate external-display and offload workflows
12. Upstream generally useful kernel changes where practical

## First hardware target

The initial development target is the Apple M1 platform (T8103), with other M-series Macs added as their PCIe/Thunderbolt implementations are characterized.

## Non-goals

This repository is not intended to replace Asahi Linux, Linux PCIe, Thunderbolt, amdgpu, xe, i915, or Mesa. Changes should be upstreamed to the appropriate project when generally useful.
