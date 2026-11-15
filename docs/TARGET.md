# Target

| Item | Value |
| --- | --- |
| Tablet | Samsung Galaxy Tab A 10.1 (2016) LTE |
| Model | SM-T585 |
| UniXNU board | `gtaxllte` |
| SoC | Exynos 7870 (S5E7870), 8x Cortex-A53 |
| GPU | **Mali-T830 MP1**, Midgard v5, Job Manager (JM) |
| GPU ID | `0x830` (Panfrost model `T830`) |
| RAM | 2 GB (four banks, 28 MB secure hole at `0x7e400000`) |
| eMMC | 32 GB, BOOT 32 MiB, RECOVERY 38 MiB, USERDATA ~12.6 GB |
| Display | 1200x1920 BOE2, 32 bpp, stride 4800 bytes, framebuffer `0x67000000` |
| Touch | STM FTS |
| OS | iPadOS 17.7.11 21H461 XNU 10063.141.1, via UniXNU |

## GPU facts

- Midgard v5 is the last Midgard generation (`T820/T830/T860/T880`).
- Panfrost supports the T830 with **OpenGL ES 3.1 / OpenGL 3.1** on Linux.
- **PanVK has no Midgard v4/v5 backend**; Vulkan exists only for Bifrost v6/v7
  and Valhall v10+.
- The T830 has no firmware/GMU command processor; submission is the JM model
  (job slots, `JS_*` registers, software fences), like Bifrost v6/v7.
- The Midgard shader ISA is a VLIW/SIMD design, different from Bifrost; the
  compiler is `src/panfrost/compiler/midgard`.

## Exynos 7870 specifics to establish

These are not in any existing NativeMali driver and must be read from the
mainline Exynos 7870 clock/PMU sources and the stock Samsung kernel:

- G3D power domain and sub-domains, power-up/down sequence and status bits.
- CMU_G3D clock: mux, divider, gate and the 26 MHz/400 MHz parents.
- GPU regulators (S2MPU05/SM5703 rails), voltage steps and enable order.
- GIC SPI numbers for the Mali job/MMU/GPU interrupts.
- GPU reset (soft reset via `GPU_CMD`, and the SoC-level reset if needed).
- Bus/IOMMU: the Mali MMU is inside the GPU; the Exynos system MMU is not used
  for the GPU path in the stock driver.

The device tree carries the GPU node; `phase0/gtaxllte-stock.dts` is the
hardware authority for addresses, interrupts and supplies.
