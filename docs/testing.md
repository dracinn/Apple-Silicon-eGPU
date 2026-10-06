# Testing

## Test progression

Never begin by assuming the GPU driver is the problem.

### A. Transport

Verify that the external Thunderbolt/USB4 link is established.

### B. PCIe

Verify that a known-good PCIe endpoint enumerates and that configuration space and BARs are accessible.

### C. DMA

Verify IOMMU/DART mappings with a simple PCIe device before introducing GPU workloads.

### D. GPU

Attach a supported AMD GPU and determine whether Linux can bind amdgpu without Apple-specific modifications.

### E. Userspace

Only after kernel bring-up succeeds:

- Vulkan
- OpenGL
- GPU offload
- external display
- suspend/resume

## Reproducibility

Every bug report should include hardware, firmware, kernel, Mesa and relevant logs. Avoid reporting a GPU as unsupported until the PCIe transport and IOMMU layers have been independently validated.
