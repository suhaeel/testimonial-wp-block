# Review: PR #19 — Replace deprecated `DOMNodeInserted` listener in `eb-animation-load.js`

- PR: https://github.com/EssentialBlocks/testimonial-wp-block/pull/19
- Branch reviewed: `fix/animation-mutation-event` (copy: `review/testimonial-pr-19`), base `dev`
- Reviewer: Abdullah Aas Suhaeel
- Verdict: **Approve**, with a suggestion to rebase so the PR diff shows only the real change

## What the PR does
`assets/js/eb-animation-load.js` attached a `DOMNodeInserted` listener for the admin panel. The PR replaces it with one capture-phase `change` listener on `document` that checks `event.target.id === 'eb-animation-style'`.

## "20 files, +2035 lines" is a display artifact
The PR was cut from `fix/audit-followups-1.3.1`, which was later squash-merged into `dev` as #18. GitHub therefore still lists #18's commits. The real effect on `dev`, checked with a 3-way merge (`git merge-tree origin/dev origin/fix/animation-mutation-event`):

- merges with **no conflicts**
- changes **1 file**: `assets/js/eb-animation-load.js`, +7 / -11

Suggestion: rebase the branch onto `dev` (`git rebase --onto origin/dev 2bb8560`) so the PR shows only that one commit and reviewers are not confused.

## Does it work? (tested in a real browser)
Headless Chrome 153, WordPress 7.1.3, Local site, plugin active.

1. Number of `DOMNodeInserted` listeners registered while the editor loads: **1 before, 0 after**.
2. Functional test of the admin handler: add `<div class="eb___animated eb___fadeIn">` in view, add `<select id="eb-animation-style">` after page load (like the inspector does), reset the div classes, fire `change` on the select, wait 900 ms.

| JS | Div classes after `change` |
|---|---|
| old (`dev`) | `eb___animated eb___fadeIn` (nothing happened) |
| PR #19 | `eb__animated eb__fadeIn` (handler ran) |

So this is not only a console-warning cleanup. Chrome removed mutation events (`DOMNodeInserted`) in Chrome 127, so on current Chrome the old admin handler **never ran at all**. The PR fixes a real functional bug.

## Findings
No blocking bugs.
1. (suggestion) Rebase onto `dev` as described above.
2. (info) The script is registered under the shared handle `essential-blocks-eb-animation`; the first standalone EB plugin to register it wins. The warning/bug only goes away when every standalone plugin ships this change (the PR notes this; image-gallery-block 1.4.2 already has it).
3. (info, pre-existing, not introduced here) `replaceAnimationClasses` queries only the main document, so elements inside the block-editor iframe canvas are not reached. Unchanged behaviour, out of scope.
4. (info) `setTimeout(..., 500)` is kept as before.
