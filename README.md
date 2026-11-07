# bevy_render 0.20.1, patched for ExtractMeMaybe

The `bevy_render` 0.20.1 package as published on crates.io (upstream
[bevyengine/bevy](https://github.com/bevyengine/bevy) commit
`a0577b5c6a7b208719dd761c5c6a7a539e472a97`, path `crates/bevy_render`; see `.cargo_vcs_info.json`),
with this README in place of Bevy's, as the first commit, so that a `[patch.crates-io]` entry takes
its dependencies from crates.io as the published crate does. The commits after it are the changes,
each marked `emm:` in the source, and one fix from upstream `main`, as upstream wrote it and
unmarked. Licensed as upstream: MIT or Apache-2.0 (`LICENSE-MIT`,
`LICENSE-APACHE`).

## Changes

- `prepare_indirect_parameters_build_jobs` makes one indirect-parameters build job per run of
  contiguous batch ranges of a view, phase and mesh class (`join_contiguous`), not one per batch
  set. Each job is a `set_bind_group` and a dispatch in each of bevy_pbr's early, late and main
  `*_build_indirect_parameters` passes for a camera, and in its early and late shadow passes
  (`per_view_shadow_pass`) for a shadow view. wgpu encodes each at `finish` and puts a barrier
  before it, since the passes write `read_write` storage. A sorted phase has a batch set every few draws, so a
  big scene's camera had 4,000-12,000 jobs, each run three times, and they kept the GPU fully busy.
  The build shader reads a job only as `[first_batch_index, last_batch_index)` and each batch's own
  batch set, so a joined job builds the same batches.
  [ExtractMeMaybe#937](https://github.com/smashedbanan/ExtractMeMaybe/issues/937) has the
  measurements.
- `.cargo-ok` is not tracked. The first commit carried it from the unpacked package, but it is not
  part of the published crate: it is cargo's mark of a finished unpack, which cargo writes into its
  own checkout of this repo.

## Backports from upstream main

A commit from bevyengine/bevy `main` after the 0.20 branch, applied whole to this crate with
`git format-patch --relative=crates/bevy_render` and `git am`, its upstream author kept. It needed
no change to apply or build on 0.20.1.

- bevyengine/bevy#25816 (`7c82c4c2`): `DirtySpecializations` finds last frame's pending
  specialisations and queueings by main entity among this frame's visible entities. 0.20.1 kept
  the render entity a mesh had when it went pending, so a mesh whose render entity changed while
  it stayed visible, as adding a component that requires `SyncToRenderWorld` does, lost its
  pending specialisation. Queueing now drops a pending mesh that is no longer visible, as
  specialisation already did. smashedbanan/bevy_pbr carries the related bevyengine/bevy#25815.
