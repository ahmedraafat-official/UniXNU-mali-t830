# Milestones

Every milestone is gated: the next one starts only after the previous one has
evidence. "It compiles" is not evidence of hardware execution.

## M0 — reconnaissance (this repository)

- Map Mesa PanVK v6/v7/v10, the Midgard compiler/genxml and the Panfrost T830
  JM/MMU.
- Map the G52 NativeMali JM kernel and the G610 NativeMali CSF kernel.
- Map the MaliMetal frontend/plugin and the Exynos display work.
- Record the gaps and the licence boundaries.

Exit: this document set, source pins, forks.

## M1 — PanVK arch 5 host build

- Add arch 5 to `src/panfrost/vulkan/meson.build`, model and feature tables.
- Add the `midgard/` Vulkan arch directory and the v5 job/descriptor packing.
- Route shaders through `compiler/midgard`.
- Host tests for every packer/format/job against `genxml/v5.xml`.
- `libvulkan_panfrost` for `PAN_ARCH=5` builds (Linux host).

Exit: host tests pass; no GPU execution claimed.

## M2 — T830 read-only probe

- NativeMali T830 kext built into the UniXNU kernel, writes compiled out.
- Read GPU ID `0x830`, revision, features, power/clock state, interrupt lines.
- Confirm the G3D domain and CMU_G3D registers against the stock DTS and
  mainline sources.
- No BO, no MMU write, no job.

Exit: a signed probe record from the tablet (registers, clocks, supplies).

## M3 — MMU, BO, WRITE_VALUE

- Soft reset, L2/shader/tiler power, MMU enable.
- Allocate/map one BO, submit one `WRITE_VALUE` job, wait for the fence.
- Fault path tested by an intentionally bad address.

Exit: the fence signals and the write value is observed.

## M4 — PanVK device and one job

- PanVK opens the NativeMali client, creates a device, allocates a buffer and
  submits a Vulkan job through the T830 backend.
- The first shader is a trivial vertex/fragment pair through the Midgard
  compiler.
- Host-side trace compared with the Panfrost v5 reference.

Exit: a rendered triangle read back from a BO.

## M5 — Metal self-test

- `NativeMaliMetal.bundle` loads the T830 PanVK ICD.
- `metal2vulkan` creates a device and runs the compute/render self-test from
  `MaliMetal/usertest`.

Exit: Metal compute and render results read back.

## M6 — QuartzCore and scanout

- backboardd/QuartzCore use the Metal device for composition.
- DECON scanout presents the composed frame with a GPU fence.
- Touch/keypad input works.

Exit: the lock screen and home screen render on the panel.

## M7 — iPadOS UI

- SpringBoard, CoreAnimation and the system apps with GPU composition.
- Measure frame rate and memory on 2 GB; fix the hot paths.

Exit: a usable UI (not a benchmark).

## Parallel non-GPU prerequisites

The display, touch, eMMC, PMIC and Wi-Fi drivers are tracked in the gtaxllte
device repository. The GPU milestones can reach M5 without the panel, but M6/M7
need them.
