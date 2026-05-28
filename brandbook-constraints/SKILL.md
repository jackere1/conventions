---
name: brand-system-constraints
description: Lock a brand's visual and verbal identity into a small set of enforceable constraints, then reuse that lock as a playbook for humans and as a prompt scaffold for AI-generated content. Use whenever starting brand work for a product or project, defining a visual identity, preparing marketing or social media content (AI-generated or hand-made), auditing existing surfaces for consistency, or briefing someone to produce on-brand posts. Trigger even when the user only mentions one piece — "what colors should I use," "make this post on-brand," "our marketing looks inconsistent," "write me a social caption," "generate an ad image" — because every one of those needs the full constraint set behind it, not just the piece named.
---

# Brand System Constraints

This skill codifies brand identity as a small set of **enforceable constraints**, not a style guide to admire. The deliverable is always a filled-in `brand-lock.md` for a specific project. Once that file exists, it serves two jobs: a 90-second human playbook, and a copy-paste prompt prefix for any AI content generation.

## Core philosophy

1. **Constraints produce consistency; taste does not.** People composing by feel produce inconsistent work even when skilled. People filling locked templates produce consistent work even when unskilled. The job of this skill is to remove composition decisions, not to teach taste.

2. **Own one thing, not many.** Small brands get recognized by owning a *single* primary color and a *single* register. Mixing teal + purple + blue, or formal + chummy, is not "rich" — it's noise. When in doubt, subtract.

3. **The lock fits on one page.** If the brand spec can't be referenced in 90 seconds before someone posts, it won't be followed and it doesn't exist. No 40-page brandbooks. One `brand-lock.md`.

4. **Distinctiveness is measured in-feed, not in isolation.** A palette that looks fine on the website but vanishes against a Facebook feed full of dark gradients has failed the only test that matters for marketing. Pick for the half-second glance.

5. **AI-generated content needs MORE constraint, not less.** Generators default to slop and drift. The lock is what keeps AI output on-brand; without it, every post is a different brand.

## Workflow

**Phase 1 — Define the lock (once per project).**
Fill in `assets/brand-lock-template.md` → save as the project's `brand-lock.md`. Every field needs a concrete value. A field left as a placeholder is a hole the next person fills with guesswork. If the user hasn't decided a value, force the decision now or mark it `TODO` explicitly — never leave a silent blank.

**Phase 2 — Apply, in this order.**
1. Audit existing product surfaces (website, app UI) against the lock; fix mismatches first.
2. Apply to in-product UI: buttons, errors, empty states, price display.
3. Build the post templates (Phase-1 file references them; now make the actual Figma/Canva files).
4. Apply to sales/support DMs, onboarding emails.
5. Social posts last — they inherit the lock automatically once templates exist.

Doing social before the product lock means redoing the social when the product tightens. Don't.

**Phase 3 — Use the lock as a prompt scaffold.**
For any AI content generation (image, caption, ad), prepend the relevant lock fields to the prompt. See "Using the lock with AI" below.

## The eight constraint groups

Ordered by perception-impact per hour spent defining them.

1. **Voice & register (highest leverage, usually skipped).** For Mongolian: pick Та-formal *or* Чи-direct and never drift. Decide emoji policy (common rule: allowed in marketing, banned in product UI), English-vs-transliteration policy, and a hard cap on exclamation marks. A formal landing page paired with a chummy DM breaks trust faster than a wrong color.

2. **Logo system.** Two variants max: full lockup + symbol-only mark. Lock minimum size, clear space, and one color treatment per background type (light / dark / photo). Export SVG once, distribute, stop.

3. **Typography hierarchy.** One display, three heading sizes, one body, one caption. Tabular numerals for any currency/stat. That's ~6 named styles; content pulls from these only.

4. **Spacing rhythm.** One base unit (4 or 8px); only multiples. Invisible but it's the difference between calm and chaotic. Tailwind's spacing scale enforces this for free.

5. **Imagery rules.** Pick one lane: photography / illustration / AI-generated. If AI-generated, mandate a treatment layer (primary-color overlay 20–30% multiply, fixed aspect-crop per platform, no AI faces) or it reads as slop.

6. **Post template system.** ~5 locked templates (announcement, educational, testimonial, comparison, pure-brand). Each is a fixed file with locked image/headline/body/logo/CTA zones. Composers fill, never invent layouts.

7. **Copy patterns.** A headline formula, 2–3 fixed CTA verbs (never deviate), one price-presentation format, one number format (lock thousands/decimal separators).

8. **Naming & capitalization.** One spelling of the brand name. Capitalization rule for feature names. Decide whether the product noun is the English term or the localized term, and never mix.

## Using the lock with AI (Phase 3)

The `brand-lock.md` is a prompt prefix. Two modes:

**For AI image generation** — prepend the imagery, color, and logo fields:
> Generate a [template type] background image. Hard constraints: primary color [hex], accent [hex]. Aspect ratio [X]. No human faces. Abstract/object/scene only. Leave [top third / left half] clear for headline overlay. Style: [one descriptor from the lock, e.g. "flat, warm, daylight"].

Then apply the treatment layer in Canva/Figma, not in the generator — generators won't honor exact hexes.

**For AI copy generation** — prepend voice, copy-pattern, and naming fields:
> Write a [template type] caption in Mongolian. Register: [Та-formal / Чи-direct]. Max [N] sentences. CTA must be one of: [verb list]. Price format: [format]. Brand name spelled exactly: [name]. Emoji: [allowed / banned]. No more than one exclamation mark.

The point: the human never re-explains the brand to the model. The lock IS the brief.

## Auditing existing surfaces

When asked "is this on-brand" or "why does our stuff look inconsistent": read the surface against each of the eight groups, list mismatches as a flat checklist (surface → which field it violates → fix). Don't soften. The most common real findings are (a) more than one primary color in use, (b) register drift between channels, (c) composed-from-scratch posts with no template.

## Anti-pattern guard

This skill is itself a tempting place to hide. Building or polishing the *system* feels productive while the concrete work (picking the actual hexes, fixing the actual site, sending the actual DMs) stays undone. If the user is mid-sprint on something else (sales, shipping), defining the lock should take an hour and then get out of the way. Flag it directly if system-building is substituting for execution.
