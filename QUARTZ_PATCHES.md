# Quartz WebKit customization ledger

This file records the engine behavior Quartz carries beyond upstream WebKit.
Update the relevant entry in every customization or upstream-sync pull request.
The [Quartz WebKit maintenance manual](https://github.com/QuartzBrowser/Quartz/blob/main/docs/WEBKIT_MAINTENANCE.md)
is the canonical guide to development branches, upstream merges, candidate
validation, promotion, and deliberate publication.

## Inventory baseline — 2026-09-13

| Item | Verified revision / evidence |
| --- | --- |
| Fork `main` inspected | [`4a523b0b3d1ddf66abbf9ec9b6351248e57db73c`](https://github.com/QuartzBrowser/WebKit/commit/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c), committed 2026-09-13 14:23:24 UTC |
| Original customization | [`7f1d29889cbd13cb4627b31f4db73651bbcf12e0`](https://github.com/QuartzBrowser/WebKit/commit/7f1d29889cbd13cb4627b31f4db73651bbcf12e0), committed 2026-09-13 02:07:18 UTC |
| Customization's upstream parent | [`73d6001e409c45f30c9a254ebcfbba30a67f57c4`](https://github.com/WebKit/WebKit/commit/73d6001e409c45f30c9a254ebcfbba30a67f57c4), [`320990@main`](https://commits.webkit.org/320990@main), committed 2026-09-12 |
| Latest incorporated upstream | [`4f56cc248e8a2e447f77d54d7612d2a6128daccb`](https://github.com/WebKit/WebKit/commit/4f56cc248e8a2e447f77d54d7612d2a6128daccb), [`321006@main`](https://commits.webkit.org/321006@main), committed 2026-09-13 10:22:34 UTC |
| Upstream merge and fork review | [`066adfd8e6a8bfbea7ea95f6b18ca087f03960ef`](https://github.com/QuartzBrowser/WebKit/commit/066adfd8e6a8bfbea7ea95f6b18ca087f03960ef), merged by [fork PR #1](https://github.com/QuartzBrowser/WebKit/pull/1) into the inspected `main` revision |

The [comparison against incorporated upstream](https://github.com/QuartzBrowser/WebKit/compare/4f56cc248e8a2e447f77d54d7612d2a6128daccb...4a523b0b3d1ddf66abbf9ec9b6351248e57db73c)
reports three fork commits ahead, zero behind, and exactly four changed files:
three implementation files and one Writing Tools API-test file. The three commits
are the original customization and two merge commits. The entries below account
for all four files in that comparison. They describe the actual retained source
at this baseline; future ledger-only commits will also appear in later diffs.

**Evidence boundary:** this initial inventory inspected GitHub commit metadata,
the upstream comparison, and the four retained source files. It did not build an
engine, run TestWebKitAPI, exercise Quartz UI, or independently revalidate the
historical test claims in PR #1. The regression commands and manual checks below
are instructions for the next validation, not new passing-test claims. Record
the actual engine SHA, Quartz SHA, macOS, architecture, Xcode, and result links
when they are run.

## QWK-001 — Respect the Writing Tools opt-out on every patched entry path

- **Status / owner:** active; Quartz maintainers.
- **Origin:** [customization commit `7f1d2988`](https://github.com/QuartzBrowser/WebKit/commit/7f1d29889cbd13cb4627b31f4db73651bbcf12e0).
- **User-visible reason:** a web view configured with Writing Tools behavior
  `None` must keep Writing Tools disabled, including when the whole web view is
  editable. Disabled UI paths should return before probing the Writing Tools UI
  framework. This supports Quartz's bundled-engine compatibility policy.
- **Upstream tracking:** no upstream issue or proposal link is recorded in the
  inspected customization commit. Add a link here when one is established.

### Retained implementation and behavior

| Source at the audited engine SHA | Retained behavior |
| --- | --- |
| [`WebPageProxyCocoa.mm`, `writingToolsBehavior()`](https://github.com/QuartzBrowser/WebKit/blob/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c/Source/WebKit/UIProcess/Cocoa/WebPageProxyCocoa.mm#L1484-L1502) | Checks configured `None` before the editable-web-view shortcut can return `Complete`. |
| [`WebViewImpl.mm`, complete-tools eligibility](https://github.com/QuartzBrowser/WebKit/blob/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c/Source/WebKit/UIProcess/mac/WebViewImpl.mm#L3255-L3259) | Requires behavior other than `None` before entering the complete-tools path. |
| [`WebViewImpl.mm`, menu/action validation](https://github.com/QuartzBrowser/WebKit/blob/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c/Source/WebKit/UIProcess/mac/WebViewImpl.mm#L3516-L3519) | Disables the `showWritingTools:` action when configured behavior is `None`. |
| [`WebViewImpl.mm`, direct invocation and affordance](https://github.com/QuartzBrowser/WebKit/blob/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c/Source/WebKit/UIProcess/mac/WebViewImpl.mm#L5558-L5577) | Direct invocation returns immediately for `None`; affordance eligibility also requires behavior other than `None`. |
| [`WebViewImpl.mm`, context-menu capability](https://github.com/QuartzBrowser/WebKit/blob/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c/Source/WebKit/UIProcess/mac/WebViewImpl.mm#L7814-L7817) | Checks disabled behavior before framework availability or class lookup. |

With behavior other than `None`, the existing selection, editability, single-line
input, availability, and requested-tool conditions still apply. The patch does
not force Writing Tools on. Without these guards, the previous implementation
could report `Complete` for an editable web view despite the explicit opt-out,
and the patched menu/direct-entry paths lacked the same configuration guard.

### Regression evidence to collect

The [retained API tests](https://github.com/QuartzBrowser/WebKit/blob/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c/Tools/TestWebKitAPI/Tests/WebKit/WKWebView/WritingTools.mm#L2805-L2880)
include `WritingTools.APIWithBehaviorNone` and
`WritingTools.APIWithBehaviorNoneAndEditableWebView`. Their shared helper checks
the opt-out with `_setEditable` false and true. On macOS it checks affordance
eligibility, action validation, direct selectors, and context-menu entries,
including top-level summarize/proofread/rewrite items when that feature is built.

From a WebKit checkout with compatible API-test binaries available, run:

```sh
Tools/Scripts/run-api-tests \
  WritingTools.APIWithBehaviorNone \
  WritingTools.APIWithBehaviorNoneAndEditableWebView \
  WritingTools.APIWithBehaviorDefault
```

Use the build configuration and `--root` appropriate to those binaries; the
[runner's options and test-name syntax](https://github.com/QuartzBrowser/WebKit/blob/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c/Tools/Scripts/run-api-tests#L158-L277)
are the source of truth. Record skips and missing framework/test support as
unexercised cases. A zero exit code does not substitute for confirming that the
named tests actually ran. The existing default-behavior test provides a useful
check that enabled behavior has not been accidentally suppressed.

For Quartz integration, select text in ordinary page content and a
`contenteditable` region with the opt-out configured; confirm no Writing Tools
affordance or context-menu action appears. Exercise the direct selector and
whole-web-view editable cases in the API-test harness. Inspect loaded-image
provenance during the packaged-app checks described in the maintenance manual;
these source guards alone do not prove that every system-framework path avoids
loading another engine image.

- **Conflict / migration points:** configured versus effective behavior,
  editable-view shortcuts, responder selectors, context-menu implementations,
  affordance scheduling, and Writing Tools SPI/availability guards.
- **Removal condition:** upstream provides equivalent opt-out behavior across
  these entry points, and targeted tests plus relevant Quartz runtime checks
  pass after removing the fork delta.
- **Latest sync review:** source retained at the 2026-09-13 baseline above.
- **Latest fresh runtime/test evidence from this inventory:** none collected.

## QWK-002 — Avoid unused Quick Look framework probing during view destruction

- **Status / owner:** active; Quartz maintainers.
- **Origin:** [customization commit `7f1d2988`](https://github.com/QuartzBrowser/WebKit/commit/7f1d29889cbd13cb4627b31f4db73651bbcf12e0).
- **Source:** [`WKImmediateActionController.mm`, `willDestroyView:`](https://github.com/QuartzBrowser/WebKit/blob/4a523b0b3d1ddf66abbf9ec9b6351248e57db73c/Source/WebKit/UIProcess/mac/WKImmediateActionController.mm#L79-L96).
- **Purpose:** avoid touching Quick Look availability/class lookup on this view
  teardown path when there is no animation controller needing cleanup.
- **Upstream tracking:** no upstream issue or proposal link is recorded in the
  inspected customization commit.

The added `animationController &&` check short-circuits before
`PAL::isQuickLookUIFrameworkAvailable()` and the preview-menu-item class lookup.
When a controller is present, the existing availability/type checks still run;
a Quick Look preview menu item's delegate is still cleared. The previous code
probed framework availability even when the animation controller was absent.
This is one specific lifecycle guard, not a claim that Quick Look is disabled
or that all potential mixed-engine loading paths are covered.

### Regression evidence to collect

No dedicated Quick Look regression test was added by the inspected four-file
customization commit. Add focused coverage when changing this subsystem. It
should distinguish all three teardown cases:

1. No animation controller: no Quick Look availability/class lookup is needed.
2. A non-preview animation controller: destruction completes normally without
   treating it as a preview menu item.
3. A Quick Look preview menu item: its delegate is cleared during destruction.

For packaged Quartz, repeatedly open and close ordinary tabs/windows without
invoking immediate actions, and inspect loaded-image provenance before and after
teardown. Separately exercise an available Quick Look/immediate-action preview,
then dismiss it and close the view; verify cleanup without a crash or stale
delegate. Record cases that cannot be exercised on the chosen macOS/hardware.
These manual checks are proposed validation and were not run for this inventory.

- **Conflict / migration points:** `willDestroyView:`, recognizer ownership,
  Quick Look soft linking, preview controller types, and delegate lifetime.
- **Removal condition:** upstream short-circuits or eliminates the unused
  framework probe while preserving preview cleanup; demonstrate the three
  teardown cases and Quartz provenance checks before removing this guard.
- **Latest sync review:** source retained at the 2026-09-13 baseline above.
- **Latest fresh runtime/test evidence from this inventory:** none collected.

## Update this ledger with the next change

Keep stable QWK identifiers. Record the exact customization and incorporated
upstream SHAs, source paths, changed conditions, conflict resolutions, tested
engine/Quartz pair, environment, and links to actual results. Mark entries
upstreamed or retired only after the replacement behavior is verified. Keep
application-side packaging/provenance obligations linked to the canonical
maintenance manual instead of inventing additional engine patches for them.
