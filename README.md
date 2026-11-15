# UniXNU Mali-T830 (Midgard v5) GPU port

Goal: Metal / QuartzCore / backboardd compositing for iPadOS 17.7.11 on the
Samsung Galaxy Tab A 10.1 (2016) LTE, **SM-T585 / gtaxllte**, Exynos 7870,
**Mali-T830 MP1** (Midgard v5, Job Manager), under UniXNU.

**Status: planning and reconnaissance. There is no buildable T830 Metal backend
yet.** The kernel-entry milestone (UniXNU -> DarwinBoot -> XNU) is tracked in the
device repository; this repository is the GPU component.

## Why this is a port and not a configuration

The proven UniXNU Mali Metal stack (`Ken-u/MaliMetal`, Mali-G610) runs through:

```
backboardd -> NativeMaliMetal -> metal2vulkan -> PanVK -> darwin-drm -> NativeMali.kext
```

PanVK has Vulkan backends for **Bifrost v6/v7 and Valhall v10+ only**. Mesa's
`src/panfrost/vulkan/meson.build` lists `bifrost_archs = [6, 7]` and
`jm_archs = [6, 7]`; there is no **arch 5 (Midgard v5)**. The T830 therefore has
no Vulkan driver, and `metal2vulkan` cannot run on it as-is.

What does exist for the T830:

- `src/panfrost/compiler/midgard` — the Midgard shader compiler.
- `src/panfrost/genxml/v4.xml` and `v5.xml` — the Midgard/Mali command and
  descriptor packers.
- the Panfrost Linux driver — T830 (`arm,mali-t830`) JM, MMU, job slots.
- `Dracen-W/Mali-G52` — a NativeMali JM kernel for Bifrost v7 (closest kernel
  code, but MediaTek power/PMIC and Bifrost v7 job headers).
- `Dracen-W/MaliMetal` — a preparatory G52 PanVK v7 port, and the file-by-file
  AdrenoMetal porting map.
- `UniXNU/AdrenoMetal` — the reusable Metal frontend, `metal2vulkan` bridge,
  `NativeMali`-shaped ABI and plugin discovery (MIT).

## Workstreams

1. **Mesa PanVK v5 backend** — add arch 5, a Midgard arch directory, v5
   descriptor/tiler/command code, the v5 job header, and the Midgard compiler
   path. Host tests without hardware. See [docs/PORTING.md](docs/PORTING.md).
2. **NativeMali T830 kernel** — fork the G52 JM driver, replace the MediaTek
   power/PMIC layer with Exynos 7870 CMU_G3D/PMU and regulators, add the v5 GPU
   ID and Midgard job header, keep the MMU/BO/service/ABI. See
   [docs/KERNEL.md](docs/KERNEL.md).
3. **Display/scanout** — Exynos DECON/DSI plus the MaliMetal scanout/backboardd
   plugin. See [docs/DISPLAY.md](docs/DISPLAY.md).
4. **Metal stack reuse** — `NativeBridgeMetalFrontend`, `metal2vulkan`,
   `NativeMaliMetalPlugin` with a T830 PanVK ICD path.

## Milestones

See [docs/MILESTONES.md](docs/MILESTONES.md). The short version:

| Milestone | Deliverable |
| --- | --- |
| M0 | reconnaissance and porting map (this repository) |
| M1 | PanVK arch 5 compiles and passes host tests (no GPU) |
| M2 | T830 power/clock/GPU-ID read-only probe on the tablet |
| M3 | MMU/BO/WRITE_VALUE job on T830 |
| M4 | PanVK device creation and one Vulkan job on T830 |
| M5 | Metal self-test through the plugin |
| M6 | QuartzCore/backboardd and scanout |
| M7 | iPadOS UI with GPU compositing |

## Risks

The T830 is the least-supported Mali generation in this stack: no PanVK
backend, Midgard ISA, 2 GB RAM, and an Exynos 7870 power/clock domain that no
existing NativeMali driver covers. [docs/RISKS.md](docs/RISKS.md) is the honest
list.

## Sources and licences

See [docs/SOURCES.md](docs/SOURCES.md). The kernel and Mesa work is
GPL-2.0-only where it derives from the G52/Mesa code; the AdrenoMetal-derived
frontend keeps its MIT headers. This repository's original text is
CC-BY-4.0.
