# Sources and pins

| Source | Role | Licence |
| --- | --- | --- |
| `UniXNU/AdrenoMetal` | Metal frontend, `metal2vulkan` bridge, plugin discovery, NativeMali-shaped ABI | MIT (frontend), see repo |
| `Ken-u/MaliMetal` | Mali-G610 (PanVK v10) Metal stack, verified on EVB10; iOS Mesa build recipe, Darwin PanVK patches | see repo |
| `Dracen-W/MaliMetal` | Mali-G52 (Bifrost v7) port plan, file-by-file map, `NativeMali` seam | GPL-2.0-only |
| `Dracen-W/Mali-G52` | NativeMali JM kernel for Bifrost v7 (closest kernel base) | GPL-2.0-only |
| `Ken-u/Mali` | NativeMali CSF kernel for G610 (ABI reference, not JM) | see repo |
| `Dracen-W/UniXNU-merlinx` | G52 device integration, GPU addon, scanout | see repo |
| Mesa | PanVK, Midgard compiler, genxml v4/v5, Panfrost GL | MIT |
| Linux `drivers/gpu/drm/panfrost` | T830 JM/MMU/job register model | GPL-2.0 |
| Linux Exynos 7870 clock/PMU/regulator | Exynos power/clock sources | GPL-2.0 |
| UniXNU `UniXNU` | kernel/payload/ADT, GPU addon mount, XNU patches | see repo |
| `UniXNU/UniXNU-gtaxllte` | device port, kernel entry milestone | see repo |

## Reconnaissance revisions (2026-11-15)

- Mesa `main` (shallow sparse): `src/panfrost/vulkan/meson.build` shows
  `bifrost_archs = [6, 7]`, `jm_archs = [6, 7]`; `genxml/v5.xml` and
  `compiler/midgard` are present; no arch 5 Vulkan backend.
- `Ken-u/MaliMetal` README: verified on Mali-G610/RK3588 EVB10, iPadOS
  17.7.11 UI at about 50 fps.
- `Dracen-W/MaliMetal` README: Mali-G52, preparatory, kernel driver is the
  critical path.
- `Dracen-W/Mali-G52` README: Bifrost v7 JM, M0-M7 state, host-tested, no
  on-device kext build yet.

## Forks created for the port

- `ahmedraafat-official/Mali-G52` (from `Dracen-W/Mali-G52`) — T830 kernel work
  branch.
- `ahmedraafat-official/MaliMetal` (from `Dracen-W/MaliMetal`) — T830 PanVK
  backend/plugin work branch.

The Mesa v5 backend is developed as a patch series in this repository
(`mesa-patches/` once M1 starts) against the pinned Mesa revision above.
