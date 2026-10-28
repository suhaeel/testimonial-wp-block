# Review: PR #20 — Keep the block icon visible when the editor highlights it

- PR: https://github.com/EssentialBlocks/testimonial-wp-block/pull/20
- Branch reviewed: `fix/block-icon-selected` (copy: `review/testimonial-pr-20`), base `dev`
- Reviewer: Abdullah Aas Suhaeel
- Verdict: **Approve** (2 optional nits below)

## What the PR does
The Testimonial icon is filled `#6C3BFF`. On the blue highlighted row of the `/` inserter, a selected List View row and the drag chip, that purple almost disappears. The PR adds `assets/css/block-icon-selected.css` (recolours those fills to `currentColor`) and enqueues it on `enqueue_block_editor_assets`.

## Scope check
`git diff origin/dev...HEAD`: 2 files, +56 lines (the CSS file and `testimonial-wp-block.php`). No change to `dist/`, `controls`, `lib/` or saved block markup, so existing posts cannot be affected.

## How it was tested
Environment: Local site, WordPress 7.1.3, PHP 8.2.29, headless Chrome 153. The plugin folder was linked into the site and activated. The PR author states this plugin was not tested in a browser (only accordion-toggle and infobox), so it was tested here. "Before" = same session with the stylesheet request blocked.

| Where | Before | After |
|---|---|---|
| `/` inserter highlighted row (bg `rgb(56,88,233)`) | icon fill `rgb(108,59,255)` | `rgb(239,240,242)` |
| Selected List View row | `rgb(108,59,255)` | `rgb(255,255,255)` |

Screenshots: `review/screenshots/pr20-*.png`. Before, the icon is invisible on the blue row; after, it is clearly visible.

Other checks
- `php -l testimonial-wp-block.php`: no syntax errors (PHP 8.2.29).
- Icon SVG (`src/icon.js`, also in `dist/index.js`) has 3 `<path fill="#6C3BFF">` direct children of `<svg>`, so the CSS selectors match.
- The stylesheet loads once and is editor-only. No PHP notices or console errors.
- Not tested: the drag chip (same rule set as the two rows above; only the colour differs).

## Findings
No blocking bugs.
1. (nit) `'1.0.0'` is hard-coded as the style version. The handle is shared across the standalone plugins, so the first plugin to enqueue wins; if the CSS ever changes, browsers can keep the old file. Using a plugin-independent bump or `filemtime` avoids this.
2. (nit) The `#C5B1FF` and white-knockout rules are unused by this icon (it only has `#6C3BFF`). Harmless dead rules, acceptable because the file is shared across plugins.
3. (info) The PR says the rules match Essential Blocks' `block-icon-selected.css`. That file was not found in the local `essential-blocks-for-gutenberg` `main`, so this claim could not be verified.
