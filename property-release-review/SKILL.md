---
name: property-release-review
description: "Audits a commercial-photography catalog for entries that require property releases, trademark clearance, or copyright analysis on depicted subjects (buildings, interiors, permanent public artwork, branded venues, signage, vehicles). Four-bucket disposition (remove / off-Stripe-keep-gallery / SOFT-FLAG / OK / EXEMPT) leaning on Dutch FOP (Auteurswet Art. 18) for permanent public works. Mandatory 1024px-long-edge downscale before viewing any image. Sibling to /model-release-review for the persons-in-frame side. USE WHEN property release, image rights, can I sell this print, trademark in catalog, copyright in catalog, FOP question, sculpture in frame, venue in frame, branded building, catalog rights audit. NOT FOR persons / model releases (use /model-release-review)."
effort: medium
---

## 🚨 MANDATORY FIRST ACTION: Downscale Every Image to 1024px Long Edge

**Before reading or classifying ANY image, downscale to 1024px on the long edge. Process only the downscaled copy. If resize fails, HALT and prompt the user.**

```typescript
import sharp from "sharp";
await sharp(srcPath)
  .resize({ width: 1024, height: 1024, fit: "inside", withoutEnlargement: true })
  .jpeg({ quality: 82, mozjpeg: true })
  .toFile(outPath);
```

Output location: `MEMORY/WORK/{slug}/downscaled/`. Originals are never modified. See `[[downscale-images-before-processing]]` for the cross-cutting rule.

## The Question

For each non-exempt catalog entry: **is there anything in the depicted scene (building, interior, sculpture, mural, trademarked livery, signage, vehicle, named venue) that requires a property/trademark/copyright release before commercial sale?**

## Dutch Freedom of Panorama (Primary Defense)

**Auteurswet Article 18** grants broad commercial FOP for works *permanently installed in public space*:
- Permanent outdoor sculpture, mural, architecture — generally clear for commercial sale of photographs
- **Permanence is the key test** — temporary installations, traveling exhibitions, indoor museum pieces do NOT qualify
- Walls / facades visible from public street, public-square statues, public-park installations — typically OK
- Murals "installed deliberately" by the property owner count as permanent artwork — OK

## Trademark vs Copyright

| Concern | Type | Disposition |
|---------|------|-------------|
| Branded logos on buildings (Heineken sign, NS livery) | Trademark | Usually OFF-STRIPE (delist from sale, can keep as portfolio) |
| Distinctive livery / corporate identity | Trademark | OFF-STRIPE |
| Sculpture, painting, mural (permanent, public) | Copyright + FOP | OK under Dutch FOP |
| Sculpture, painting, mural (temporary or indoor) | Copyright | HARD-FLAG / REMOVE |
| Building exteriors (general architecture) | Generally clear | OK |
| Building interiors of paid-entry venues | Property right | HARD-FLAG / REMOVE |
| Installation art (esp. mannequins, mixed-media) | Copyright | HARD-FLAG / REMOVE |

## What Is and Isn't Sensitive

**Sensitive (paid-entry named venues — scrub metadata, treat as flag candidate):**
- Keukenhof, Artis, Bloemenmarkt, Hortus Botanicus, Vondelpark (borderline), Skalar, Kraftwerk Berlin
- Specific museum interiors, named gardens with admission
- Named festivals with ticketed admission and depicted installations

**NOT sensitive (keep as-is — these are general geography, not paid venues):**
- Cities (Amsterdam, Lisse, Haarlem)
- Neighborhoods (Grachtengordel Zuid, Jordaan)
- Districts (centrum)
- Towns
- GPS coordinates
- Artist, Copyright, DateTimeOriginal, Make, Model EXIF fields — these are YOUR rights / provenance; never strip them

## Buckets

| Bucket | Action | When |
|--------|--------|------|
| **Bucket 1** | Remove entirely (catalog + Stripe + file + original → archive) | Copyright HARD-FLAG with no FOP defense (e.g. installation art, indoor museum, temporary work) |
| **Bucket 2** | Off Stripe, keep in gallery | Trademark concern (branded livery, signage). Clear `sizes[]`, retain `stripeProductId`. |
| **SOFT-FLAG** | Judgment call | Surface to user; let them decide bucket |
| **OK** | Keep on sale | Cleared under FOP or no rights issue |
| **EXEMPT** | Skip entirely | Macro series per [[macro-exempt-from-release-review]] |

## Remediation Pipeline

**Bucket 1 (full removal):**
1. Edit `data/catalog.json` — remove entry
2. `bun scripts/catalog/05-generate.ts` — regenerate `lib/products.ts` + `lib/gallery.ts`
3. Move file from `public/images/{series}/{slug}.jpg` to `/root/photo-archive/property-release-removed/`
4. Move original from source archive to `/root/photo-archive/property-release-removed/`
5. `bun scripts/catalog/07-stripe-archive.ts --live --products=prod_X` — archive Stripe product + child prices

**Bucket 2 (off Stripe, keep in gallery):**
1. Edit `data/catalog.json` — clear `sizes: []` (KEEP `stripeProductId`)
2. `bun scripts/catalog/05-generate.ts` — regenerate; the generator filter (`e.stripeProductId && e.sizes && e.sizes.length > 0`) excludes from PRODUCTS but keeps in gallery
3. `bun scripts/catalog/07-stripe-archive.ts --live --products=prod_X` — archive Stripe product + child prices

**Macro metadata scrub (per `[[macro-exempt-from-release-review]]`):**
- Strip paid-venue identifiers from filename, IPTC Keywords/Subject, Caption, Location, Sublocation
- Keep cities, neighborhoods, districts, GPS, Artist, Copyright, dates, camera Make/Model

## Calibrated Debate, Not Capitulation

When the user challenges a flag:
1. Re-apply the 1024-gate (downscale + view)
2. State the strongest counter-argument honestly
3. Test against FOP / trademark / copyright frameworks
4. Concede when dispositive; name residual risk when not

Concession examples from the 2026-05-23 live review:
- **Portrait against painted brick** → conceded to OK after permanence-of-mural confirmed (FOP applies)
- **Clock-bicycle at Rijksmuseum** → conceded to OK after argument that the photo is a reinterpretation of a public-installation sculpture by an artist the photographer knows
- **Skalar by Bauder/Henke at Westerpark gasfabriek** → maintained HARD-FLAG (Bucket 1) — temporary light installation, no FOP defense
- **RAI logos** → reframed to Bucket 2 (without logos the image is fine; trademark is the only concern, so off-Stripe-keep-gallery)

## Distinction from `/model-release-review`

See sibling skill for the persons-in-frame side. When a photo trips both:
- Property side first — trademark/permanent-artwork concerns are usually dispositive of the sale decision
- A model-release OK doesn't restore a photo already off-sale for property reasons (and vice versa)

## Workflow

1. **Scope** — read the catalog, separate macro (EXEMPT) from non-macro
2. **Triage by description / IPTC** — flag obvious candidates (named venues, named installations, branded livery)
3. **Visual verification (1024-gate MANDATORY)** — downscale every candidate, view, classify
4. **Spot-check** — pick ≥2 OK-by-description entries, downscale + view, confirm
5. **Structured report** — bucket-by-bucket
6. **Surface decisions** — present buckets; let user assign final disposition
7. **Execute remediation** — per the pipeline above, with --dry-run before --live on Stripe

## Gotchas

- **Stripe key is LIVE** — `--dry-run` before any `--live` archive run, every time. The dry-run output should list every product + child price slated for archival.
- **Order matters** — archive Stripe AFTER catalog regeneration; otherwise the live site briefly shows a product with no Stripe price.
- **Don't delete originals** — move to `/root/photo-archive/property-release-removed/`. Restoration may be needed.
- **Don't enumerate macros** — they are EXEMPT en bloc.
- **EXIF Artist/Copyright/Make/Model are NEVER stripped** — those are the photographer's provenance, not sensitive metadata.
- **Generator filter is load-bearing** — `e.stripeProductId && e.sizes && e.sizes.length > 0` is what makes Bucket 2 semantics work; do not change without auditing both products.ts and gallery.ts consumers.

## When to invoke

- "review the catalog for property releases"
- "image rights audit"
- "can I sell this print"
- "is this image safe to sell"
- "copyright review on the catalog"
- "FOP question on [photo]"
- Adding new images to the catalog from venues / festivals with potential property/copyright concerns
