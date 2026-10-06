# Research Sources

This project uses the Apple Silicon Linux repositories already being watched for the broader M-series Linux development work as upstream research sources. These repositories are reference material for architecture, device bring-up, kernel changes, boot/runtime behavior, and downstream integration.

## Priority sources

### Aurora Silicon

Aurora Silicon is a high-priority source because its work spans newer Apple Silicon platforms and maintains its own Apple-specific Linux and bootloader trees.

- https://github.com/aurora-silicon/linux
- https://github.com/aurora-silicon/m1n1
- https://github.com/aurora-silicon/u-boot

For this project, watch especially for:
- Apple PCIe and external-I/O changes
- Thunderbolt/USB4 work
- DART/IOMMU and DMA changes
- interrupt and device-tree changes
- platform initialization needed before PCIe devices can be used
- changes that improve M1/T8103/J313 support

### Asahi Linux

Asahi remains the primary upstream reference for Apple Silicon Linux hardware support.

- https://github.com/AsahiLinux/linux
- https://github.com/AsahiLinux/m1n1
- https://github.com/AsahiLinux/asahi-installer
- https://github.com/AsahiLinux/speakersafetyd
- https://github.com/AsahiLinux/docs

For eGPU work, the kernel and m1n1 repositories are the most important. The installer/docs repositories are useful for identifying supported hardware, boot requirements, and reproducible development environments.

### Omarchy Mac

Omarchy Mac is a useful downstream integration/reference target for a practical Apple Silicon Linux desktop.

- https://github.com/omacom/omarchy-mac
- https://github.com/omarchy-mac/omarchy-mac-iso
- https://github.com/omarchy-mac/omarchy-pkgs-aarch64

Use it primarily to validate that eGPU support can eventually work in a real Apple Silicon desktop distribution rather than only in a development kernel.

### Dakota / Bluefin Asahi

Dakota and the Bluefin Asahi work are relevant downstream references for Fedora/Atomic-style Apple Silicon Linux integration.

Track them for:
- kernel packaging
- Asahi hardware enablement
- immutable/Atomic integration
- Mesa and graphics stack packaging
- hardware enablement that could affect eGPU deployment

### Bazzite

Bazzite is a downstream end goal rather than the place to implement Apple-specific PCIe/eGPU support.

- https://github.com/ublue-os/bazzite

Monitor Bazzite Apple Silicon work when it directly enables M-series Macs or consumes the kernel/Asahi capabilities this project produces. Do not duplicate Apple-specific kernel work in Bazzite.

## How these sources affect this project

Research from these repositories should feed the project in this order:

1. **Apple platform bring-up:** Aurora Silicon + Asahi Linux
2. **PCIe / Thunderbolt / DART / DMA:** Apple-specific kernel and m1n1 work
3. **GPU binding:** upstream Linux amdgpu first, then Intel xe/i915
4. **Userspace graphics:** upstream Mesa
5. **Desktop integration:** Omarchy Mac, Dakota/Bluefin Asahi
6. **Gaming-oriented downstream integration:** Bazzite

The project should reuse upstream work wherever possible. A change belongs here only when it is specifically required to make an external PCIe GPU work on Apple Silicon and is not already available upstream.

## Monitoring rule

A watched repository becomes actionable for this project when it contains a meaningful change involving:

- Thunderbolt or USB4
- PCIe host/endpoint support
- DART/IOMMU
- DMA
- MSI/MSI-X or interrupt routing
- BAR/resource handling
- PCIe reset or hotplug
- power management
- device-tree descriptions of the relevant hardware
- kernel changes that materially improve external PCIe devices
- AMD or Intel GPU enablement on Apple Silicon
- Fedora/Atomic packaging required to consume those changes

Routine desktop changes should not be treated as eGPU progress unless they affect one of the layers above.