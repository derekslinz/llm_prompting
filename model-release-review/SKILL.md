---
name: model-release-review
description: "Audits a commercial-photography catalog for entries that require a signed model release. Four-tier classification (HARD-FLAG / SOFT-FLAG / OK / EXEMPT) keyed to identifiability-to-the-general-public, not facial recognition alone. Stricter bar for children (parental release) and workers at workplaces. Mandatory 1024px-long-edge downscale before viewing any image — pixels falsify description-only assumptions every time. Pairs with /property-release-review (sibling skill) for the rights side of the same catalog. USE WHEN model release review, person release, can I sell this print, identifiable person audit, candid model audit, child in catalog, worker in catalog, portrait release audit. NOT FOR property/architecture/artwork rights (use /property-release-review)."
effort: medium
---

## 🚨 MANDATORY FIRST ACTION: Downscale Every Image to 1024px Long Edge

**Before reading or classifying ANY image, downscale to 1024px on the long edge. Process only the downscaled copy. If resize fails, HALT and prompt the user.**

This rule exists because description-only judgments (carry-over notes from prior sessions, captions, IPTC metadata) lie. Pixels don't.

```typescript
import sharp from "sharp";
await sharp(srcPath)
  .resize({ width: 1024, height: 1024, fit: "inside", withoutEnlargement: true })
  .jpeg({ quality: 82, mozjpeg: true })
  .toFile(outPath);
```

Output location: `MEMORY/WORK/{slug}/downscaled/` (or a comparable temp dir scoped to the active task). Originals are never modified. If any image fails to resize, stop and ask — do NOT skip and continue.

See `[[downscale-images-before-processing]]` in the project memory for the cross-cutting rule.

## The Question

For each non-exempt catalog entry: **does the image require a signed model release before commercial sale?**

Answer = HARD-FLAG (yes) / SOFT-FLAG (judgment call) / OK (no) / EXEMPT (macro, skipped).

## Identifiability Standard

**The legal bar is identifiable to the general public, NOT identifiable to close friends.** Privacy/personality rights protect against being recognized by the public from the image alone.

| Factor | Pushes toward HARD-FLAG | Pushes toward OK |
|--------|------------------------|------------------|
| Face | Visible, in focus, frontal or 3/4 | Back-turned, silhouette, deep shadow, glass distortion |
| Distance | Subject occupies significant frame | Small/distant figure, not the subject |
| Subject vs incidental | Person is what the photo is about | Person is compositional element only |
| Clothing | Branded, uniformed, singular | Generic, mass-retail |
| Body / posture | Distinctive (tattoos, gait, build the public knows) | Generic |
| Setting | Workplace (worker depicted at work) | Public space, anonymous context |

**Strict cases — always HARD-FLAG even with face partially obscured:**
- **Children** — parental release required regardless of identifiability nuance
- **Workers at workplace** — cook at food stall, driver in cab when face IS visible, performer on stage. Selling prints of someone doing their job is a stricter standard than editorial.

**Permissive cases — usually OK even though a person is in frame:**
- Pure back-turn + generic clothing + public space (archetypal/anonymous framing)
- Heavy glass distortion or reflection over the face
- Silhouettes against strong backlight
- Crowd shots where no individual is the subject
- Distance great enough that features cannot be discerned at print scale

## Tiers

| Tier | Action | Trigger |
|------|--------|---------|
| 🟥 **HARD-FLAG** | Release required before sale | Identifiable face/features, person is subject; OR child; OR worker at workplace |
| 🟧 **SOFT-FLAG** | Judgment call — surface to user | Framing/distance ambiguous; reasonable people could disagree |
| 🟩 **OK** | No release needed | Unidentifiable per the table above |
| 🚫 **EXEMPT** | Skip entirely | Macro series (`up-close/*`) per house rule [[macro-exempt-from-release-review]] |

## Portraits — Audit, Not New Ask

Posed/commissioned portraits are HARD-FLAG by series convention, but the user's standing position is **releases on file**. Treat the portraits pass as **audit-confirmation** (list them so the user can verify the release file exists for each subject), not as a flag for new acquisition.

## Calibrated Debate, Not Capitulation

When the user challenges a flag, **debate it honestly**:
1. Re-apply the 1024-gate (downscale + view the actual pixels)
2. State the strongest counter-argument you can find
3. Test it against the identifiability standard
4. Concede when the argument is dispositive; name residual risk when not
5. Carry over a name a residual risk if the user accepts a borderline case

The downscale gate exists because pixels routinely falsify carry-over notes — view first, opine second.

## Workflow

1. **Scope** — read the catalog, count entries, separate macro (EXEMPT) from non-macro
2. **Preliminary classification from descriptions/series** — portraits HARD-FLAG (audit), architecture-only OK by default, anything with humans → needs visual
3. **Visual verification (1024-gate MANDATORY)** — downscale every non-OK candidate, view it, classify
4. **Spot-check** — pick ≥2 OK-by-description entries, downscale + view, confirm assumption
5. **Structured report** — HARD-FLAG / SOFT-FLAG / OK / EXEMPT tiers; every HARD/SOFT has one-line reason; OK count tallied by exclusion
6. **Surface decision points** — dispositions per HARD-FLAG (release file lookup, off-Stripe, or remove); skill-improvement learnings

## Dispositions (mirror of `/property-release-review`)

| Bucket | Action | When |
|--------|--------|------|
| **Bucket 1** | Remove entirely (catalog + Stripe + file + originals → archive) | Release impossible AND image too risky to keep even unsold |
| **Bucket 2** | Off Stripe, keep in gallery (clear `sizes[]`, retain `stripeProductId`) | Release impossible but image acceptable as portfolio |
| **Bucket 3** | Keep on sale, release confirmed on file | Release exists, audit-confirmed |

The generator filter at `scripts/catalog/05-generate.ts` (`e.stripeProductId && e.sizes && e.sizes.length > 0`) handles Bucket 2 semantics automatically. The Stripe archive script at `scripts/catalog/07-stripe-archive.ts` handles the off-Stripe step.

Originals of removed photos go to `/root/photo-archive/property-release-removed/` (shared archive with property-release pass — rename only if collision).

## Distinction from `/property-release-review`

| Concern | This skill | Sibling |
|---------|-----------|---------|
| Faces / persons | ✓ | — |
| Buildings / interiors | — | ✓ |
| Permanent artwork (sculpture, murals) | — | ✓ |
| Branded venues, trademarks | — | ✓ |
| Child protection | ✓ | — |
| Worker-at-workplace | ✓ (model release) | partial (property release for branded uniform / workplace) |
| Dutch FOP (Auteurswet Art. 18) | — | ✓ |
| Macro exemption | ✓ | ✓ |
| 1024-gate | ✓ | ✓ |
| Debate-don't-capitulate | ✓ | ✓ |

When a photo trips both — escalate via the sibling skill first (property) since trademark/permanent-artwork concerns are usually dispositive of the sale decision regardless of model release.

## Lessons Encoded From the Live Review

- **Pixels falsify descriptions.** The 2026-05-24 model release review found two prior-session HARD-FLAG calls (solitary-figure-among-autumn-park-trees, yellow-intercity-train-at-haarlem-station) that the actual 1024px frame falsified — one was back-turned, the other had glass-distorted driver. The downscale gate exists because of these.
- **Glass distortion is dispositive.** Heavy slanted/reflective glass over a face = OK, not HARD-FLAG.
- **Back-turn + generic clothing + public space = OK.** Even with a four-attribute "fingerprint" (coat, bag, hat, hair), the public-identifiability bar is rarely met.
- **NS-branded train ≠ saving point.** A model-release OK doesn't restore a photo already pulled for property/trademark reasons.
- **Property and model concerns are separable.** Resolve property side first when both apply — trademark is usually dispositive.

## Gotchas

- Do not enumerate macros in the report — they are EXEMPT en bloc.
- Do not flag based on description alone — always downscale + view first.
- Do not auto-bucket HARD-FLAGs — present to the user and let them decide.
- Children require parental release; "child looks happy / parents nearby" is not a substitute.
- Worker-at-workplace HARD-FLAG persists even if the worker is part of a brand the photo glorifies — selling prints of someone doing their job is not editorial.

## When to invoke

- "review the catalog for model releases"
- "do I need a release for [photo]"
- "model release audit"
- "can I sell this print" (person in frame)
- "person release", "candid release"
- "review portraits for release on file"
- Adding catalog entries that contain people other than commissioned subjects
