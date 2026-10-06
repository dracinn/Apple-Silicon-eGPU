# Architecture

## Design principle

Do not write a new GPU driver unless upstream Linux lacks the required functionality. Apple-specific work should primarily terminate at the PCIe/device-transport boundary.

## Layers

### 1. Apple platform

Apple Silicon contains platform-specific controllers for USB-C/Thunderbolt, PCIe and DART/IOMMU.

### 2. External PCIe transport

The project must establish a usable PCIe path over the external I/O controller. A generic PCIe endpoint should be the first validation target.

### 3. IOMMU / DART

The GPU performs substantial DMA. Correct DART mappings are therefore a prerequisite for stable operation.

### 4. Linux GPU subsystem

Once the GPU appears as a normal PCIe device, prefer existing Linux drivers:

- AMD: amdgpu
- Intel: xe and, where appropriate, i915

### 5. Mesa

Userspace rendering should use existing Mesa implementations:

- AMD: RADV / RadeonSI
- Intel: ANV / Iris

## Debugging strategy

Validate each layer independently:

1. Thunderbolt link
2. PCIe enumeration
3. PCIe configuration space
4. BARs
5. interrupts
6. IOMMU/DMA
7. GPU reset
8. kernel GPU driver binding
9. command submission
10. userspace rendering
11. display/offload
