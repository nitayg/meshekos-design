repo: mondaycom/vibe
branch: master
path: packages/style/src

## Last sync
date: 2026-08-19T13:58:00Z

### Updated in this project
- `ds/vibe-tokens.css` holds a verbatim subset of Vibe's light-theme colors, type scale, spacing, radius and motion tokens.
- `MeshekOS Board v3.dc.html` — board view: grouped collapsible rows, filled status cells, checkbox column, summary battery, add-item row.
- `MeshekOS Review Split.dc.html` — source-versus-draft review with conflict notice and the required-reason gate.
- Button geometry from `Button.module.scss` (sizeSmall 32px / sizeXs 24px, padding 4px 8px, radius 4px, text2 normal, secondary bordered in its role color, disabled via `--disabled-background-color` + `--disabled-text-color`).
- Input geometry and states from `BaseInput.module.scss` (small = 32px, padding-block 8px, padding-inline 8px/4px, hover border `--primary-text-color`, focus `--primary-color`, `aria-invalid` → `--negative-color`, 100ms ease-in transition).
- Chip, label, toast and field-caption geometry from `Label.module.scss` (radius 4px / padding 2px 8px; small = radius 2px / padding 0 4px), `Chips.module.scss` (small = 20px tall, padding 0 4px), `Toast.module.scss` (radius 4px, padding 8px, min-width 200px, top-centred, `--inverted-color-background`) and `FieldLabel.module.scss` (text2 normal on `--primary-text-color`).
- Tokens and style values only. No Vibe React components were copied (npm/bundler-only); primitives are hand-built from the real values.

## Screen map
| Screen | Built from |
| --- | --- |
| ds/vibe-tokens.css | packages/style/src/themes/light-theme.scss, typography.scss, spacing.scss, border-radius.scss, motion.scss |
| MeshekOS Board v3.dc.html | the token files + Button/BaseInput/Label/Chips/FieldLabel module.scss |
| MeshekOS Review Split.dc.html | the token files + Button/BaseInput/Label/Chips/Toast/FieldLabel module.scss |
| MeshekOS Shell v2.dc.html | the token files + Button.module.scss |
