# Architecture

## Design principle

Do not write a new GPU driver unless upstream Linux lacks the required functionality.

Apple-specific work should primarily terminate at the Apple PCIe/USB4/DART boundary. Once an external GPU is presented as a conventional PCIe device, prefer the upstream AMD or Intel GPU stack.

## Layer model

### 1. Apple platform

Apple Silicon provides platform-specific controllers for USB-C/USB4/Thunderbolt, PCIe and DART/IOMMU.

The current Apple USB4 work is especially important because the NHI, DART and USB4 router are integrated into the Apple ACIO block rather than exposed like a conventional PC Thunderbolt controller.

### 2. External PCIe transport

The project must establish a usable PCIe path over the external I/O controller.

A generic PCIe endpoint is the first validation target. Do not start with an AMD GPU: transport and resource failures are much easier to isolate with a simple endpoint.

### 3. PCIe resources and BARs

External PCIe GPUs can require large memory windows. The PCI hierarchy must provide sufficient upstream bridge windows before GPU driver initialization.

Resource allocation is a separate problem from ReBAR. The initial target is a stable resource layout, not maximum BAR size.

### 4. DART / IOMMU

The GPU performs substantial DMA, so correct DART mappings are a prerequisite for stable operation.

Tests must cover:

- mapping/unmapping
- DMA synchronization
- invalidation/teardown
- page-size and alignment assumptions
- failures around large or boundary-crossing mappings
- suspend/resume restoration

### 5. ARM64 DMA coherency

Do not assume that a non-standard Apple PCIe path behaves like a conventional coherent x86 PCIe root complex.

DMA correctness must explicitly account for:

- CPU/device cache maintenance
- DMA map/unmap synchronization
- coherent versus non-coherent transactions
- buffer ownership transitions
- IOMMU translation

The NVIDIA non-standard-ARM PCIe work is useful precedent here, but no NVIDIA driver code is a project dependency.

### 6. Interrupts and PCIe error handling

Validate MSI/MSI-X and PCIe AER independently.

Transient external-link failures must not be treated as impossible. Error recovery, reset, link retraining and driver rebind behavior are part of the device lifecycle.

### 7. External/removable-device classification

USB4 topologies do not necessarily expose the same Thunderbolt metadata as an Intel Thunderbolt hierarchy.

Driver behavior that depends on Thunderbolt-attached state must therefore be checked against Apple/USB4 topology and Linux removable-device state. This is particularly important for AMD runtime PM, switcheroo and external VBIOS handling.

### 8. Linux GPU subsystem

Once the GPU appears as a normal PCIe device, prefer existing Linux drivers:

- AMD: amdgpu
- Intel: xe and, where appropriate, i915

The first AMD milestone should require as little Apple-specific amdgpu code as possible.

### 9. Mesa

Userspace rendering should use existing Mesa implementations:

- AMD: RADV / RadeonSI
- Intel: ANV / Iris

## Debugging strategy

Validate each layer independently:

1. USB4/Thunderbolt controller
2. PCIe tunnel
3. PCIe enumeration
4. PCIe configuration space
5. PCI bridge windows
6. Endpoint BARs
7. MSI/MSI-X
8. DART/IOMMU
9. DMA coherency/cache maintenance
10. GPU reset
11. PCIe AER recovery
12. External/removable classification
13. Kernel GPU driver binding
14. Command submission
15. Userspace rendering
16. Runtime PM
17. Hotplug/surprise removal
18. Suspend/resume
19. Display/offload

## Failure isolation rule

When a GPU fails to initialize, classify the failure before changing the GPU driver:

- no device → transport/enumeration
- BAR/resource failure → PCI hierarchy/resource allocation
- IOMMU fault → DART/DMA mapping
- DMA timeout/corruption → coherency/cache maintenance
- interrupt timeout → MSI/MSI-X/AIC/transport
- PCIe completion/AER failure → link/error recovery
- driver refuses external device → classification/power/VBIOS path
- rendering failure after successful bind → amdgpu/xe/Mesa
