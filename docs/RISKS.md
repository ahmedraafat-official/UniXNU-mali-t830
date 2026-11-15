# Risks

## 1. No upstream PanVK Midgard v5 backend

This is the primary risk. Mesa's Vulkan driver has no arch 5 backend; the
v6/v7 (Bifrost JM) code is a template, but Midgard's descriptor, tiler, blend
and job formats differ. The Panfrost GL v5 code proves the hardware can be
driven, not that the Vulkan backend is a small diff.

Mitigation: build the v5 backend bottom-up with host tests from `genxml/v5.xml`;
keep the first target to the QuartzCore subset, not Vulkan conformance.

## 2. Midgard compiler integration

The Midgard compiler exists for GL; PanVK's shader metadata/ABI was written for
Bifrost/Valhall. Expect interface and lowering fixes.

Mitigation: compile the MaliMetal fixture shaders first; compare against the
Panfrost GL v5 compiler output.

## 3. Exynos 7870 GPU power/clock

No NativeMali driver covers Exynos. The G3D domain, CMU_G3D, regulators and
reset are board/SoC-specific and must be read from mainline/Panfrost/vendor
sources. A wrong power sequence can hang or damage the board.

Mitigation: read-only probe first; fixed voltages; reverse teardown; no DVFS.

## 4. Midgard JM job format

The G52 job packer is Bifrost v7. The v5 header/slot format differs; the
Panfrost `v5.xml` and `panfrost_job.c` are the authority. A wrong job can fault
the GPU or the bus.

Mitigation: one `WRITE_VALUE` job first, with a fault handler and a reset path.

## 5. Display and scanout

DECON/DSI drivers are a separate large port. Without them the GPU can render
off-screen but the UI cannot be shown.

Mitigation: static test pattern and boot log first; the GPU path can proceed in
parallel.

## 6. Memory (2 GB)

iPadOS 17 runs on 2 GB iPads, but the port also carries the GPU driver, the
Metal frontend, QuartzCore and the scanout buffers. Memory pressure is likely.

Mitigation: 4 KB pages, no unnecessary copies, the `metal2vulkan` single-sample
policy, measure early.

## 7. Time and uncertainty

The G52 MaliMetal port is itself preparatory; the G610 is the only verified Mali
Metal. A T830 port adds a new Vulkan backend and a new SoC power layer. This is
a multi-month research project with no guarantee of a usable UI.

Mitigation: milestone gates, documented evidence, stop/rollback points, keep the
tablet recoverable (stock BOOT backup, TWRP).
