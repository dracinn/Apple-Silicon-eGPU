# Apple Silicon eGPU

Linux eGPU enablement for Apple Silicon Macs, focused on AMD first and Intel second.

This project is a platform-enablement effort: make an external PCIe GPU look sufficiently normal to Linux that existing upstream GPU drivers can do the rest.

## Scope

- Apple Silicon Macs, initially M1/T8103 systems
- Linux / Asahi-based kernels
- AMD GPUs as the primary target
- Intel GPUs as a secondary target
- Existing upstream Linux GPU drivers wherever possible
- Apple USB-C/USB4/Thunderbolt PCIe tunneling
- PCIe enumeration, configuration space, BAR/resource allocation
- Apple DART/IOMMU and ARM64 DMA coherency/cache maintenance
- MSI/MSI-X, reset and PCIe AER/error recovery
- External/removable-device classification
- Hotplug, surprise removal, suspend/resume and runtime power management
- Mesa/Vulkan/OpenGL integration after kernel bring-up

## Explicitly out of scope

- NVIDIA support as a project target
- macOS eGPU support
- Reimplementing mature AMD or Intel GPU drivers unnecessarily
- Making ReBAR/large BAR a prerequisite for initial AMD bring-up

NVIDIA eGPU work may be referenced as an architectural and diagnostic precedent, especially where it exposes non-standard ARM64 PCIe, DMA, or external-link behavior.

## Architecture

Apple Silicon → USB-C / USB4 / Thunderbolt → Apple PCIe tunnel / external PCIe hierarchy → PCIe resource and BAR allocation → DART / IOMMU → ARM64 DMA coherency and cache maintenance → interrupts / AER / reset → external-device classification → AMD amdgpu or Intel xe/i915 → Mesa

The first milestone is not GPU rendering. It is proving that a generic PCIe endpoint can survive enumeration, resource assignment, DMA, interrupts and reset through the Apple external I/O path.

## Bring-up gates

1. Apple USB4/Thunderbolt infrastructure is initialized
2. PCIe tunnel is established and recoverable
3. Generic PCIe endpoint enumerates reliably
4. PCI bridge windows and endpoint BARs receive usable resources
5. DART/IOMMU mappings work for device DMA
6. ARM64 DMA coherency/cache maintenance is correct
7. MSI/MSI-X and PCIe AER/error recovery work
8. Device reset and hot-unplug behavior are understood
9. AMD external-device classification and amdgpu binding work
10. Mesa rendering/offload works
11. Runtime PM, suspend/resume and external display paths are stable

## Development milestones

1. Document Apple Silicon PCIe and Thunderbolt topology
2. Establish reproducible hardware/firmware/kernel/Mesa test matrix
3. Validate USB4/Thunderbolt discovery and PCIe tunneling
4. Validate generic PCIe enumeration and configuration-space access
5. Validate PCI bridge windows, BAR allocation and resource reassignment
6. Validate DART/IOMMU, DMA coherency and synchronization
7. Validate interrupts, reset and PCIe AER recovery
8. Bring up an AMD GPU with upstream amdgpu
9. Validate Mesa Vulkan/OpenGL and GPU offload
10. Add hotplug, surprise removal and power-management handling
11. Investigate Intel xe/i915 support
12. Validate external-display workflows
13. Upstream generally useful kernel changes where practical

## First hardware target

The initial development target is the Apple M1 platform (T8103), especially the MacBook Air/other M1 systems with a USB-C/Thunderbolt path that can be characterized reproducibly.

Other M-series Macs should be added as their USB4/Thunderbolt and PCIe implementations are characterized.

## Downstream integration

This repository is intentionally standalone. It should feed its upstreamable Apple-specific PCIe/USB4/DART work into Asahi/Aurora/Linux and then be consumed downstream by Fedora Asahi, Omarchy/Dakota/Bluefin Asahi, and eventually Bazzite where appropriate.

It should not become a Bazzite fork or duplicate Bazzite packaging work.

## Non-goals

This repository is not intended to replace Asahi Linux, Linux PCIe, Thunderbolt, amdgpu, xe, i915, or Mesa. Changes should be upstreamed to the appropriate project when generally useful.
