---
name: ad-copy-pro
description: >
  Builds an AI-powered HTML ad copy generator for The Linux Foundation's paid media campaigns.
  Covers all channels: Google Ads (RSA, Display, PMax), Microsoft Ads, LinkedIn (Single Image,
  Text, Message, Carousel), Reddit, Meta (FB/IG), Brave Ads, Feathr, and X/Twitter. Enforces
  platform character limits at both prompt and post-processing layers. Delivers a self-contained
  interactive HTML tool with per-field character counts and one-click copy export.

  Trigger whenever someone asks to: create ad copy, write paid media copy, generate ads for any
  channel, build a campaign brief, or create copy for an LF event, certification, course, or
  awareness campaign. Also trigger for "build me an ad copy tool", "I need copy for [channel]",
  or any LF paid media workflow mention.
---

# Ad Copy Pro — Linux Foundation Paid Media Tool

This skill builds and delivers a self-contained, AI-powered HTML ad copy generator tailored to
The Linux Foundation's brand voice and paid media stack.

## What to build

Deliver a **single HTML file** saved to `/mnt/user-data/outputs/lf_ad_copy_generator.html` and
presented with `present_files`. The file must be fully self-contained (no external dependencies
beyond Google Fonts and the Anthropic API).

The tool lets the user:
1. Paste a landing page URL
2. Select a campaign goal
3. Describe their target audience
4. Optionally add a key value prop / offer
5. Check one or more ad channels
6. Click **Generate** — which calls the Anthropic API and streams back copy

## Reference implementation

A complete, working implementation is bundled at `assets/lf_ad_copy_generator.html`.

**Always start from this file** — copy it to `/home/claude/` and then modify as needed rather
than building from scratch. This avoids re-implementing the character limit enforcement logic,
the channel metadata, the brand voice prompt, and the UI.

```bash
cp /mnt/skills/user/ad-copy-pro/assets/lf_ad_copy_generator.html /home/claude/lf_ad_copy_generator.html
```

Then deliver it:
```bash
cp /home/claude/lf_ad_copy_generator.html /mnt/user-data/outputs/lf_ad_copy_generator.html
```

## Channels supported

| Channel ID | Platform | Ad Formats |
|---|---|---|
| `google` | Google Ads | RSA (Search), Performance Max Asset, Display |
| `microsoft` | Microsoft / Bing Ads | Responsive Search Ad, Audience Ad |
| `linkedin` | LinkedIn Ads | Single Image, Text Ad, Message Ad (InMail), Carousel |
| `reddit` | Reddit Ads | Promoted Post, Conversation Ad |
| `meta` | Meta (Facebook & Instagram) | Single Image, Story/Reel, Carousel |
| `brave` | Brave Ads | Notification Ad, In-Browser Display |
| `feathr` | Feathr | Display Retargeting, Email Banner |
| `twitter` | X / Twitter | Promoted Post, Website Card |

## Character limit enforcement (critical)

Every generated string is enforced at **two layers**:

**Layer 1 — Prompt-level**: The AI system prompt instructs the model to count every character,
rewrite any string that exceeds its limit, and run a final self-check before returning JSON.

**Layer 2 — Post-processing**: After JSON is parsed, `enforceCharLimits()` trims any remaining
violations using `resolveLimit(fieldKey, adType)` — a context-aware function that resolves the
correct limit based on both the field name AND the ad type (e.g., `headline` resolves to 30 for
Google RSA, 70 for LinkedIn Single Image, 27 for Meta, etc.).

See `assets/lf_ad_copy_generator.html` for the full `resolveLimit()` logic if you need to add
new channels or formats.

## Brand voice

- **Tone**: Authoritative but accessible. Open source credibility without jargon overload.
- **Emphasis**: Community impact, career advancement, industry recognition, trust, real-world adoption.
- **CTAs**: Always specific and action-oriented — "Earn Your CKA", "Register for KubeCon",
  "Start Your LFCA", "Join 1M+ Open Source Devs" — never generic filler like "Learn More".
- **Never**: Vague superlatives, padding, or corporate speak.

## Customization guidance

When a user wants to **add a new channel**, update three things in the HTML:
1. Add the channel to `CHANNEL_META` with name, color, badge, and adTypes
2. Add a checkbox to the channel grid in the HTML
3. Add the format spec to `buildPrompt()` character limits section
4. Add limit resolution cases to `resolveLimit()` for any ambiguous field names

When a user wants to **adjust brand voice**, edit the `BRAND VOICE:` paragraph in `buildPrompt()`.

When a user wants to **change the UI theme**, edit the CSS variables at the top of `<style>`:
`--bg`, `--accent`, `--surface` etc. are the primary levers.

## Copy export format

The **per-card copy button** serializes from `window._cardMap` (keyed by card DOM ID → ad data),
producing labeled plain text with numbered arrays. The **Copy All** button iterates
`window._lastAdData` in channel order, separated by `═══` headers and `───` dividers — suitable
for pasting directly into Google Docs, spreadsheets, or campaign briefs.

## Delivery checklist

- [ ] File is at `/mnt/user-data/outputs/lf_ad_copy_generator.html`
- [ ] `present_files` called with the output path
- [ ] No broken external dependencies (Google Fonts CDN is acceptable)
- [ ] Anthropic API called with `claude-sonnet-4-20250514`, `max_tokens: 8000`
- [ ] Character limits enforced in both prompt and post-processing
- [ ] All 8 channels available in the UI
