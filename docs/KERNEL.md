# NativeMali T830 kernel plan

Base to fork: `Dracen-W/Mali-G52` (NativeMali, Bifrost v7, Job Manager). It is
GPL-2.0-only and already has the right shape: read-only bring-up, ARM_MALI_LPAE,
BO state machine, job slots with software fences, a fixed IOUserClient ABI and
host tests.

## Reuse unchanged

- `NativeMaliABI.h`, `NativeMaliAbiValidate.*` — fixed-size wire ABI.
- `NativeMaliBoTable.*`, `NativeMaliHandleIndex.h`, `NativeMaliVm.*` —
  BO/address-space bookkeeping and ARM_MALI_LPAE page tables.
- `NativeMaliMmu.*` — AS slots, `panfrost_mmu_enable()`/disable, `FLUSH_PT`,
  fault decode (the T830 uses the same Mali MMU).
- `NativeMaliService.*`, `NativeMaliUserClient.*`, `NativeMaliFault.*`,
  `NativeMaliIrq.*` — device/client/fence logic; change only the interrupt
  source table.
- `NativeMaliMemory.*` — spans, map/unmap, cache policy.
- `NativeMaliJobSlot.*` — the `JS_HEAD_NEXT`/`JS_AFFINITY_NEXT`/`JS_CONFIG_NEXT`/
  `JS_FLUSH_ID_NEXT`/START handshake is the same JM model; the **job header**
  payload changes for v5.

## Replace

1. **Platform/power** (`NativeMaliBoard`, `NativeMaliPower`,
   `NativeMaliPwrap`, `NativeMaliMt6358`):
   - remove PWRAP/MT6358 and the MediaTek MFG domains;
   - read the Exynos 7870 GPU node from the device tree;
   - implement G3D power domain up/down, CMU_G3D mux/divider/gate, the
     S2MPU05/SM5703 rails with fixed voltage steps and the reverse teardown;
   - every write behind a milestone flag until the read-only probe is accepted.

2. **GPU core** (`NativeMaliGpuCore`, `NativeMaliGpu`):
   - GPU ID `0x830`, Midgard v5 model/revision;
   - Midgard feature probe and `GPU_LATEST_FLUSH_ID`;
   - `panfrost_gpu_soft_reset()` and L2/shader/tiler power sequences from the
     Panfrost v5 path;
   - no Bifrost-specific quirk writes.

3. **Job packing** (`NativeMaliJob`):
   - v5 JM job header and `WRITE_VALUE` payload from `genxml/v5.xml`;
   - 64-byte slot chain is a Bifrost v7 property; verify the Midgard slot size
     and alignment from the v5 XML and Panfrost `panfrost_job.c`.

4. **XNU glue** (`xnu/`):
   - the G52 repo's glue is paper code, never compiled;
   - build the IOKit subclass, `IOBufferMemoryDescriptor` page source, cache
     maintainer and interrupt registration against the real XNU 21H461 headers;
   - the device node/`Info.plist` must match the Exynos 7870 GPU node and the
     UniXNU GPU addon path.

## Exynos 7870 register sources

- `phase0/gtaxllte-stock.dts` — GPU node, interrupts, supplies, clocks.
- mainline Linux `arch/arm/boot/dts/exynos7870.dtsi` and
  `drivers/clk/samsung/clk-exynos7870.c` — CMU_G3D, G3D gates, parents.
- mainline `drivers/soc/samsung/exynos-pmu.c` / `exynos-pmu.h` — power domains.
- mainline `drivers/regulator/s2mpu05-regulator.c` — rails and voltage steps.
- stock Samsung kernel Mali Midgard driver — vendor power/clock sequence and
  the exact GPU revision (GPL; reference only, do not copy into MIT files).
- Panfrost `drivers/gpu/drm/panfrost/panfrost_regs.h`, `panfrost_gpu.c`,
  `panfrost_job.c`, `panfrost_mmu.c` — T830 JM/MMU register model.

## Bring-up order

M2: read-only probe of GPU ID, features, power/clock state, interrupts, with
writes compiled out. M3: soft reset, MMU enable, one BO mapped, one
`WRITE_VALUE` job with a software fence. M4: PanVK device creation and a real
job. Only then the Metal plugin.

## Fail-closed rules (from the G52 driver)

- read before write; writes behind `NATIVE_MALI_Mx_WRITES`;
- every register value cites `path:function` plus the pinned source hash;
- quiesce and roll back in reverse order on error;
- no DVFS; fixed voltages within the vendor OPP;
- never report ready after a partial bring-up.
