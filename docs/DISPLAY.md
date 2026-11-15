# Display and scanout

The GPU path is only useful once the panel is owned by XNU and the
MaliMetal/Metal present path has a scanout to write into.

## Hardware

- Exynos 7870 **DECON** (display controller) and **DSI**.
- Panel: 1200x1920 BOE2, 32 bpp, stride 4800 bytes.
- S-Boot/UEFI scanout buffer at `0x67000000`, reserved
  `0x67000000-0x68194000` (two buffers).
- Touch: STM FTS over the Exynos I2C.

## Work

1. **DECON driver** in XNU: window setup, timing, buffer address, vsync/tear
   interrupt. Mainline reference:
   `drivers/gpu/drm/exynos/exynos7_drm_decon.c`.
2. **DSI driver**: PHY, PLL, command/video mode, panel init sequence. Mainline
   reference: `drivers/gpu/drm/exynos/exynos_dsi.c` plus the panel's
   `panel-samsung-*.c` or the stock sequence from the vendor kernel.
3. **Framebuffer handover**: keep the S-Boot buffer as the first scanout so the
   boot log and the first UI frame have a valid surface before the DRM/KMS-like
   driver reconfigures the panel.
4. **MaliMetal scanout**: fork `GpuScanoutClient.h` / `GpuScanoutPresenter.h`
   from AdrenoMetal; replace the AGX display tunnel with the DECON buffer
   handle/fence. `backboardd` publishes the surface; the presenter waits on the
   GPU fence and flips the DECON address.
5. **Input**: the touch/keypad drivers are separate; the UI cannot be used
   without them.

## Constraints

- The DECON buffer is outside the XNU owned window; the driver must map it
  explicitly (device/write-combined) and keep it out of the general allocator.
- No IOMMU is required for the display; the DECON reads physical addresses.
- The first milestone is a static test pattern and the boot log, not
  QuartzCore.
