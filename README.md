# rt_cinema.entpkg

Real-time hardware ray-tracing (`native.trace@2`), native `SHADER_F16`, HiDPI `scale()`-adaptive & DRS-ready, and zero-asset 16× anisotropic GPU PBR rendering building blocks for [Entelechy](https://github.com/Fynn-3D/entelechy).

## Installation

Clone directly into your local Entelechy package shelf and point `ENTELECHY_PACKAGES` at the shelf directory:

```sh
git clone https://github.com/entpkg/rt_cinema.entpkg
export ENTELECHY_PACKAGES="$PWD"
```

Reference it by name from any `.esn` scene or `.ent` document:

```text
use "rt_cinema.entpkg";
```

## Dependencies

Depends only on Entelechy's built-in atom packages (`random.entpkg`, `geometry.entpkg`, `raster.entpkg`, `image.entpkg`).

## What It Provides

### Drop-in Sub-Graphs (`graphs/`)
- **`graphs/gtao.esn`**: Capped half-resolution (`<= 960×600`) native `SHADER_F16` 12-tap Screen-Space GTAO + 5-step contact shadows + depth-and-normal edge-preserving 3×3 bilateral denoise (`Grid<Surface>` → `Grid<Vec2 as f16>`).
- **`graphs/gtao_jitter.esn`**: Jitter-aligned variant taking `taa_state` so Halton-jittered raster pipelines keep sub-pixel-locked GTAO and contact shadows with zero edge bleed.
- **`graphs/rt_shadows.esn`**: Zero-stall capped half-resolution (`rt_dims` with `render_scale` for DRS) multi-mesh hardware RT soft sun shadows (`native.trace@2 { flags = 1 }`) with GPU indirect ray compaction, distance-adaptive sun disk penumbra, and smooth 2×2 bilinear readout (`rt_sample_shadow_hits`).
- **`graphs/cinema_finish.esn`**: Complete DRS-ready `SHADER_F16` filmic post-processing stack (`sanitize_hdr` + volumetric composite → zero-flicker Halton 8-sample TAA with `taa_init_like`, unjittered reprojection, motion-gated disocclusion & `cut_id` reset → capped `640×400` optical bokeh DOF & bloom → exposure-invariant distance-faded sharpening + AgX filmic grade + optional `encode_srgb` → FXAA).
- **`graphs/cinema_finish_dynamic.esn`**: Action-game / skinned-character variant taking `motion: Grid<Vec4 as f16>` (`xyz` world-space vertex displacement + `w` reactive translucency/VFX mask) with single-pass 3×3 silhouette velocity dilation & near-field foreground uncovering for zero-ghosting fast blades, capes, and hair.
- **`graphs/cinema_post.esn`**: Standalone Post-TAA filmic finish (`Image` linear HDR + depth-in-w → Optical Bokeh DOF → Wide-Halo Bloom → Exposure-Invariant Sharpening + AgX (`tone_mode = 0`) or UE5 ACES Chiaroscuro (`tone_mode = 1`) + optional `toe_power` & `encode_srgb` → FXAA) for scenes that composite fast translucent VFX/fire/rain after temporal resolve.

### Kernels & Pure Functions (`kernels/`)
- **`kernels/taa.ent`**: `RtTaaState`, `rt_apply_clip_jitter`, `rt_cinema.cut_none@1`, `rt_cinema.motion_none@1`, `rt_cinema.taa_start@1`, `rt_cinema.taa_step@1`, `rt_cinema.taa_init@1`, `rt_cinema.taa_init_like@1`, `rt_cinema.taa_resolve@1`, `rt_cinema.taa_resolve_dynamic@1`.
- **`kernels/gtao.ent`**: `rt_safe_norm`, `rt_cinema.sun_const@1`, `rt_cinema.screen_ao@1`, `rt_cinema.screen_ao_jitter@1`, `rt_cinema.ao_blur@1`, `rt_cinema.ao_blur_sharp@1`.
- **`kernels/rt_helpers.ent`**: `rt_invalid_ray`, `rt_logical_extent`, `rt_sample_shadow_hits`, `rt_cinema.same_lens@1`, `rt_cinema.rt_dims@1`, `rt_cinema.rt_dims_of@1`, `rt_cinema.rt_probe_init@1`, `rt_cinema.rt_gaze_probes@1`, `rt_cinema.rt_shadow_rays@1`.
- **`kernels/pbr_maps.ent`**: Zero-asset `ONCE`-cached `1024×1024` seamless procedural PBR textures (`Grid<Vec4 as unorm8>` + `Grid<Vec2 as unorm8>`) with auto-generated GPU mipmap chains + 16× hardware anisotropic filtering when sampled via 4-arg `at_repeat(..., ddx, ddy)` (or 3-arg `lod`):
  - `kind = 0`: Chiseled River Granite
  - `kind = 1`: Ancient Woodland Oak Bark
  - `kind = 2`: Damp Woodland Floor, Moss & Leaf Litter
  - `kind = 3`: Board-Formed Brutalist Architectural Concrete
  - `kind = 4`: Riveted Battleship Steel Armor Plating
  - `kind = 5`: Honed Dark Basalt Ashlar Plaza Paving
  - `kind = 6`: Cold-Rolled Brushed Stainless Steel Aerospace Livery
  - Helpers: `rt_compute_footprint`, `rt_box_uv`, `rt_box_duv` (crisp axis-aligned architecture), `rt_triplanar_albedo`, `rt_triplanar_normal` (seamless 3-axis blended curved cliffs/rocks/arches), `rt_unpack_normal`, `rt_apply_tangent_normal`.
- **`kernels/weather_wave.ent`**: Smoothly-gated 3D spherical world wavefront (`RtWaveSample`, `rt_eval_world_wave`), 4-layer volumetric cumulus sky (`rt_eval_volumetric_sky`), sub-pixel pinpoint starfield (`rt_eval_stars`), forked lightning (`rt_eval_lightning`), wind-slanted rain streaks (`rt_eval_rain_glint`), and concentric raindrop ripples (`rt_eval_rain_ripples`).
- **`kernels/post.ent`**: `rt_cinema.vol_none@1`, `rt_cinema.sanitize_hdr@1`, `rt_cinema.bright_pass@1`, `rt_cinema.cinema_grade@1` (supports `encode_srgb = 1` for calibrated linear-HDR -> AgX -> IEC sRGB output, or `0` for legacy v1.0 linear output).

## Verification

Run the standalone package validation and GPU verification replays:

```sh
entelechy package rt_cinema.entpkg
entelechy check rt_cinema.entpkg/tests/showcase.esn rt_cinema.entpkg/tests/dynamic_taa.esn
entelechy replay rt_cinema.entpkg/tests/showcase.replay rt_cinema.entpkg/tests/dynamic_taa.replay --backend gpu
```

## License

MIT — see [LICENSE](LICENSE).
