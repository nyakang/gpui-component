# NyaTerm fork notes

This branch carries [NyaTerm](https://github.com/nyakang/nyaterm)'s local changes
to `gpui-component` on top of an unmodified upstream base.

- Fork: <https://github.com/nyakang/gpui-component>
- Upstream: <https://github.com/longbridge/gpui-component>
- Base revision: `884062aeb3` (upstream `main`, `gpui-component` 0.6.0)
- Branch: `nyaterm`

NyaTerm uses this crate through the stable `nyaterm-ui` facade, so these are the
changes that could not be made on the NyaTerm side.

## Patches

1. `fix(tab_bar): let segmented tabs fill the bar` — for
   `TabVariant::Segmented` the inner container takes the full width and each
   wrapper becomes `flex_1` with `min_w_0`, so segments divide the bar evenly
   instead of hugging their labels. Other variants are unchanged.
2. `feat(scrollbar): reveal a hover scrollbar from anywhere in the viewport` —
   `ScrollbarMode::Hover` revealed the bar only from inside the track bounds, so
   a hidden bar had to be aimed at blind. Adds
   `ScrollbarStateInner::hovered_viewport`, fed by `HitboxId::is_hovered` so an
   overlay does not reveal the bar behind it, and a `reveal_hover` predicate.
   Thumb styling still keys off track hover alone; `Scrolling` and `Always` are
   unchanged.
3. `feat(menu): support configurable popup appearance` — adds an opt-in
   `PopupMenuAppearance` for row, typography, icon-slot, spacing, separator,
   radius and disabled-state metrics. The default keeps the upstream rendering
   unchanged, while nested submenus inherit the nearest parent appearance
   unless they explicitly override it. NyaTerm uses this at its `nyaterm-ui`
   boundary to align ordinary component menus with its richer tab context menu.
4. `build(deps): use the NyaTerm GPUI fork` — upstream 0.6.0 uses the
   `gpui-pre 0.3.1` package set. This branch points every Zed-derived workspace
   dependency at one revision of `nyakang/zed`, rebased onto that package
   set's `801c087a` source snapshot, so NyaTerm keeps its dynamic-texture and
   hidden-cursor APIs without linking two incompatible GPUI copies.

## Not carried here

- `fix(theme): re-project scrollbar theme on every palette change` — dropped when
  this series was rebased from `b1e78a51` onto `0bfcb640`. Upstream fixed the same
  bug independently in `222cf964` ("theme: Follow the global radius setting
  everywhere"), and its `Theme::sync_base` is a superset: where the NyaTerm patch
  rebuilt only `gpui_base::Theme::scrollbar`, `sync_base` replaces the whole Base
  projection, and upstream added a regression test for it
  (`base_projection_carries_a_square_radius_to_the_scrollbar`). Consumers that
  called `Theme::sync_scrollbar_theme(cx)` call `Theme::sync_base(cx)` instead.
- `fix(dialog): make the backdrop event wrapper cover the viewport` — upstream
  `df1d07b2` fixed the same collapsed-wrapper bug with
  `.absolute().inset_0()` and added `the_backdrop_fills_the_host`. The merge
  keeps that implementation and drops NyaTerm's older `.size_full()` hunk.

Upstream 0.6.0 also renamed `gpui-component-assets` to `gpui-kit-assets`.
NyaTerm keeps its existing dependency key as a Cargo alias so application code
continues to import `gpui_component_assets`.

## Merge notes

The 0.6.0 merge moved `crates/ui` to `crates/component`; Git carried the
segmented-tab and popup-appearance changes across the rename automatically.
The scrollbar patch also merged without conflict. Only
`crates/base/src/dialog.rs` conflicted, because upstream had independently
fixed the same bug; the resolution takes the upstream implementation and test.

## Validation

Validated on Windows 11 against `nyakang/zed:nyaterm` revision
`3b3066c872`, whose patch stack is based on the same `801c087a` snapshot as
`gpui-pre 0.3.1`:

```sh
cargo test -p gpui-base
cargo test -p gpui-base reveal
cargo test -p gpui-component menu::popup_menu --lib
cargo check -p gpui-component
cargo clippy -p gpui-component --all-targets
```
