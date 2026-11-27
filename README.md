# rt_cinema.entpkg

Real-time hardware ray-tracing (`native.trace@2`), native `SHADER_F16`, HiDPI `scale()`-adaptive, and zero-asset 16× anisotropic GPU PBR rendering building blocks for [Entelechy](https://github.com/Fynn-3D/entelechy).

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
- **`graphs/gtao.esn`**: Capped half-resolution (`<= 960×600`) native `SHADER_F16` 12-tap Screen-Space GTAO + 5-step contact shadows + depth-preserving 3×3 bilateral denoise (`Grid<Surface>` → `Grid<Vec2 as f16>`).
- **`graphs/rt_shadows.esn`**: Capped half-resolution multi-mesh hardware RT soft sun shadows (`native.trace@2 { flags = 1 }`) with GPU indirect ray compaction, distance-adaptive sun disk penumbra, and smooth 2×2 bilinear readout (`rt_sample_shadow_hits`).
- **`graphs/cinema_finish.esn`**: Complete `SHADER_F16` filmic post-processing stack (`sanitize_hdr` + volumetric composite → zero-flicker Halton 8-sample TAA with unjittered reprojection, motion-gated disocclusion & cut-ID reset → capped `640×400` optical bokeh DOF & bloom → AgX filmic grade + distance-faded sharpening → FXAA).

### Kernels & Pure Functions (`kernels/`)
- **`kernels/taa.ent`**: `RtTaaState`, `rt_apply_clip_jitter`, `rt_cinema.cut_none@1`, `rt_cinema.taa_start@1`, `rt_cinema.taa_step@1`, `rt_cinema.taa_init@1`, `rt_cinema.taa_resolve@1`.
- **`kernels/gtao.ent`**: `rt_safe_norm`, `rt_cinema.sun_const@1`, `rt_cinema.screen_ao@1`, `rt_cinema.ao_blur@1`.
- **`kernels/rt_helpers.ent`**: `rt_invalid_ray`, `rt_logical_extent`, `rt_sample_shadow_hits`, `rt_cinema.same_lens@1`, `rt_cinema.rt_dims@1`, `rt_cinema.rt_probe_init@1`, `rt_cinema.rt_gaze_probes@1`, `rt_cinema.rt_shadow_rays@1`.
- **`kernels/pbr_maps.ent`**: Zero-asset `ONCE`-cached `1024×1024` seamless procedural PBR textures (`Grid<Vec4 as unorm8>` + `Grid<Vec2 as unorm8>`) with auto-generated GPU mipmap chains + 16× hardware anisotropic filtering when sampled via 4-arg `at_repeat(..., ddx, ddy)` (or 3-arg `lod`):
  - `kind = 0`: Chiseled River Granite
  - `kind = 1`: Ancient Woodland Oak Bark
  - `kind = 2`: Damp Woodland Floor, Moss & Leaf Litter
  - `kind = 3`: Board-Formed Brutalist Architectural Concrete
  - `kind = 4`: Riveted Battleship Steel Armor Plating
  - `kind = 5`: Honed Dark Basalt Ashlar Plaza Paving
  - `kind = 6`: Cold-Rolled Brushed Stainless Steel Aerospace Livery
  - Helpers: `rt_compute_footprint`, `rt_box_uv`, `rt_box_duv`, `rt_unpack_normal`, `rt_apply_tangent_normal`.
- **`kernels/weather_wave.ent`**: Smoothly-gated 3D spherical world wavefront (`RtWaveSample`, `rt_eval_world_wave`), 4-layer volumetric cumulus sky (`rt_eval_volumetric_sky`), sub-pixel pinpoint starfield (`rt_eval_stars`), forked lightning (`rt_eval_lightning`), wind-slanted rain streaks (`rt_eval_rain_glint`), and concentric raindrop ripples (`rt_eval_rain_ripples`).
- **`kernels/post.ent`**: `rt_cinema.vol_none@1`, `rt_cinema.sanitize_hdr@1`, `rt_cinema.bright_pass@1`, `rt_cinema.cinema_grade@1`.

## Verification

Run the standalone package validation and GPU verification replay:

```sh
entelechy package rt_cinema.entpkg
entelechy check rt_cinema.entpkg/tests/showcase.esn
entelechy replay rt_cinema.entpkg/tests/showcase.replay --backend gpu
```

## License

MIT — see [LICENSE](LICENSE).
