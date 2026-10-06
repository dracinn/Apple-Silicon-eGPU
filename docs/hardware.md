# Hardware Test Matrix

## Initial platform

| Platform | SoC | Initial status |
|---|---|---|
| MacBook Air M1 | T8103 / J313 | Primary target |

## eGPU hardware

The initial AMD test card should favor mature upstream Linux support rather than maximum performance. Record:

- GPU model
- PCIe generation
- enclosure/chassis
- Thunderbolt controller
- firmware versions
- power supply
- cable
- kernel version
- Asahi kernel commit
- GPU driver version
- Mesa version

Intel testing will be added after the AMD transport path is stable.

## Required logs

For every test capture, where applicable:

- uname -a
- kernel boot log
- Thunderbolt device information
- PCI device enumeration
- IOMMU/DART messages
- GPU driver binding state
- Mesa/Vulkan information

Do not attach sensitive system information unnecessarily.
