# NyaTerm fork notes

This branch carries [NyaTerm](https://github.com/nyakang/nyaterm)'s local changes
to `gpui-component` on top of an unmodified upstream base.

- Fork: <https://github.com/nyakang/gpui-component>
- Upstream: <https://github.com/longbridge/gpui-component>
- Base revision: `0bfcb64023` (upstream `main`, `gpui-component` 0.5.2)
- Branch: `nyaterm`

NyaTerm uses this crate through the stable `nyaterm-ui` facade, so these are the
changes that could not be made on the NyaTerm side.

## Patches

1. `fix(tab_bar): let segmented tabs fill the bar` — for
   `TabVariant::Segmented` the inner container takes the full width and each
   wrapper becomes `flex_1` with `min_w_0`, so segments divide the bar evenly
   instead of hugging their labels. Other variants are unchanged.
2. `fix(dialog): make the backdrop event wrapper cover the viewport` — the
   wrapper was laid out in flow, so a press on the dimmed area outside its box
   neither dismissed the dialog nor blocked the press from reaching content
   underneath.
3. `feat(scrollbar): reveal a hover scrollbar from anywhere in the viewport` —
   `ScrollbarMode::Hover` revealed the bar only from inside the track bounds, so
   a hidden bar had to be aimed at blind. Adds
   `ScrollbarStateInner::hovered_viewport`, fed by `HitboxId::is_hovered` so an
   overlay does not reveal the bar behind it, and a `reveal_hover` predicate.
   Thumb styling still keys off track hover alone; `Scrolling` and `Always` are
   unchanged.

## Not carried here

- `fix(theme): re-project scrollbar theme on every palette change` — dropped when
  this series was rebased from `b1e78a51` onto `0bfcb640`. Upstream fixed the same
  bug independently in `222cf964` ("theme: Follow the global radius setting
  everywhere"), and its `Theme::sync_base` is a superset: where the NyaTerm patch
  rebuilt only `gpui_base::Theme::scrollbar`, `sync_base` replaces the whole Base
  projection, and upstream added a regression test for it
  (`base_projection_carries_a_square_radius_to_the_scrollbar`). Consumers that
  called `Theme::sync_scrollbar_theme(cx)` call `Theme::sync_base(cx)` instead.

The NyaTerm snapshot also rewires where dependencies come from, which is a
vendor-layout concern rather than a change to this library:

- `gpui`, `gpui_platform`, `gpui_web`, `gpui_macros` and `reqwest_client` are
  repointed from upstream's git dependencies to sibling paths under
  `vendor/zed`, so one `gpui` is shared with NyaTerm's Zed snapshot.
- `reqwest` is taken from the registry as `zed-reqwest` instead of upstream's
  git dependency, for the same reason.

This branch keeps upstream's dependency sources. A consumer that needs one
shared `gpui` should express that with `[patch]` entries in its own workspace.

Note upstream's `gpui` git dependency is not pinned to a revision, so building
this branch on its own resolves whatever `zed-industries/zed` currently has at
`main`, which may be far ahead of the snapshot these patches were written
against. That is a property of the upstream manifest, not of these patches.
`.github/workflows/nyaterm.yml` pins it to the revision NyaTerm uses, and that
pin has to move whenever NyaTerm's `Cargo.toml` does.

## Rebase notes

Only `crates/ui/src/tab/tab_bar.rs` conflicted, and only over layout: upstream
reworked that render body (`222cf964`, `cc86f8d4`, `9e069926`) and added
`.mx(-padding_x).px(padding_x)` to both the `tabs` and `tabs-inner` flexes. The
resolution keeps upstream's shape and re-inserts the one `w_full` line the patch
adds for `Segmented`. `crates/base/src/scrollbar.rs` and
`crates/base/src/dialog.rs` are still byte-identical upstream, so those two
patches applied untouched and are still needed.

## Validation

Validated against the matching Zed snapshot by pointing the `gpui*` dependencies
at a local checkout of <https://github.com/nyakang/zed> branch `nyaterm`
(`275f31ac`, upstream `4278ff36` plus NyaTerm's GPUI patches) through an
uncommitted `[patch]` in `.cargo/config.toml`, on Windows 11:

```sh
cargo test -p gpui-base          # 574 passed, incl. the two reveal_hover cases
cargo test -p gpui-base reveal   # 2 passed
cargo check -p gpui-component    # clean
```
