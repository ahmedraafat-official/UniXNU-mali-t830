# Mesa PanVK v5 porting map

Mesa revision: `main`, sparse checkout of `src/panfrost`, `src/vulkan`,
`src/gallium`, `src/util`, `src/compiler`, `src/mapi` (2026-11-15).

## What exists

```
src/panfrost/compiler/midgard/            Midgard compiler (NIR -> Midgard ISA)
src/panfrost/genxml/v4.xml, v5.xml        Midgard command/descriptor packs
src/panfrost/genxml/decode_jm.c           JM decoder
src/panfrost/vulkan/meson.build           biftost_archs = [6, 7]
                                          jm_archs = [6, 7]
src/panfrost/vulkan/jm/                   generic JM queue/command code
src/panfrost/vulkan/bifrost/              Bifrost-specific meta copy
src/panfrost/vulkan/csf/, fifthgen/,       Valhall/CSF backends
  valhall/
src/panfrost/model/pan_model.c            GPU model table (T830 = v5)
```

The Vulkan front end (`panvk_device.c`, `panvk_instance.c`,
`panvk_physical_device.c`, `panvk_image.c`, `panvk_buffer.c`,
`panvk_descriptor_set*`, `panvk_cmd_pool.c`, ...) is arch-neutral and already
compiled for each arch with `-DPAN_ARCH=N`.

## What a v5 backend needs

1. **Arch registration**
   - Add `5` to the supported arch list in `src/panfrost/vulkan/meson.build`
     and to `panvk_physical_device.c` model/feature selection.
   - `PAN_ARCH=5` must select `compiler/midgard`, `genxml/v5.xml`, and the
     `jm/` queue code.

2. **A `midgard/` Vulkan directory** (mirror of `bifrost/`):
   - `panvk_vX_meta_desc_copy.c` — Midgard descriptor copy/meta path, built from
     the v5 XML, not the Bifrost one.
   - v5 blend/attribute/tiler/descriptor packing (the `panvk_vX_*` files that
     Bifrost overrides; the rest come from `jm/`).
   - The v5 **job header** and `WRITE_VALUE` payload (`PAN_ARCH=5` packers);
     Bifrost v7's 64-byte header cannot be reused blindly.

3. **Compiler path**
   - Route `panvk_shader` through `compiler/midgard` for arch 5.
   - The Midgard compiler already produces the binary; the PanVK shader
     metadata (descriptor sizes, sysvals, workgroup layout) must be filled for
     v5.
   - Midgard has no Bifrost-style `MESA_SHADER` lowering assumptions; expect
     fixups in `panvk_nir*` and the compiler interface.

4. **Formats/tiling**
   - v5 format table and tiling/AFBC rules from the T830 model.
   - The Panfrost GL v5 code (`src/gallium/drivers/panfrost`,
     `panfrost_emit.c`, `panfrost_tiler.c`, `panfrost_blend.c`) is the register
     and format reference.

5. **Tests**
   - Host build of `libvulkan_panfrost` with `PAN_ARCH=5` (no device).
   - `panvk` unit tests for descriptor/format/job packing against the v5 XML.
   - A QEMU/unit harness is not a GPU test; the first device test is M4.

## What is deliberately not promised

- Vulkan 1.3 conformance on v5; the initial target is the subset QuartzCore and
  the MaliMetal frontend exercise (single-sample, no heaps/argument buffers/
  indirect commands, as documented by Dracen-W/MaliMetal).
- Geometry/tessellation features that Panfrost GL does not expose on Midgard.
- Performance tuning; correctness first.

## Reference points

- `Dracen-W/MaliMetal/docs/PORTING.md` — file-by-file AdrenoMetal reuse map.
- `Dracen-W/MaliMetal/docs/MESA_INTEGRATION.md` — the PanVK seam and
  `pan_kmod` boundary.
- `Dracen-W/MaliMetal/docs/SEAM.md` — the NativeMali ABI boundary.
- `Ken-u/MaliMetal/mesa-patches/` — the Darwin/iOS PanVK patches (host-pointer
  import, external buffers, modifiers, etc.) to rebase onto the v5 tree.
- `Ken-u/MaliMetal/mesa-build/` — the iOS cross-build recipe (LLVM 20,
  SPIRV-Tools, glslang, libdrm, Mesa host + iOS).
- Panfrost Linux `drivers/gpu/drm/panfrost` — T830 JM/MMU/job registers.
