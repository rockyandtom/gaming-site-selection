# Game Selection & Listing

A single-file operating-phase methodology for an iframe game / content site: where to find games worth adding, how to prove an embed source actually works, how to catch new keyword opportunities, and what the listing and SEO maintenance rhythm looks like.

This repository contains one methodology document and this English guide. `SKILL.md` is kept in its original Chinese.

## Background

Once an iframe game site is live, the largest early risk is not a lack of traffic — it is publishing games that cannot be played. A visitor clicks Play, gets a blank screen, leaves immediately, and the site accumulates the bounce behaviour that search engines read as low quality. Adding more games makes this worse, not better, if the additions are unverified.

That single failure mode determines the shape of the methodology: verification is a hard gate placed **after** finding a candidate and **before** listing it. Finding sources is cheap; proving that an embed renders a playable game is the part people skip. The document therefore spends less effort on "where to look" than on the accept/reject criteria for an embed source.

The channel ranking is not theoretical. It came from two practitioner write-ups on expanding a game library and finding promising titles, then was tested on a real project in 2026-09. The results include the unglamorous details that matter in practice: which aggregator list pages are server-rendered and can be scraped with a simple pattern, which return 403 on their detail pages, which are JavaScript-rendered and expose only a template string in the HTML, and which were unreachable from the test machine. Those observations are dated and will change; the ranking and the verification gate are the durable parts.

This Skill is the companion to `游戏站开发与部署` (Game Site Development & Deployment). That one covers building and launching the site; this one covers every day after launch.

## What it does

1. Ranks channels for finding games by effort-to-yield: aggregator new / trending lists, search-engine keyword mining (`iframe games`, `embed games`, `game distribution`, `html5 games embed`), GitHub code search for `<iframe language:html path:games/`, image-based search, developer-mode fallback, platforms that offer embeds, and source / emulator options.
2. Records the 2026-09 availability of each aggregator source, including which were JavaScript-rendered, which returned 403 on detail pages and which were blocked locally, so a new source can be evaluated against the same criteria.
3. Defines the mandatory embed-source verification gate with hard accept/reject criteria — the one step that must not be skipped before a game goes live.
4. Covers how to catch games attached to new keywords, the listing workflow, and the ongoing SEO maintenance cadence.

## When to use it

Use it after an iframe game site is live and you are expanding or refreshing its library. Apply the channel ranking when building a candidate pool, and apply the verification gate to every single candidate before publishing it.

Do not use it to choose a stack, deploy a site or bind a domain — that is the development-and-deployment Skill.

## Repository contents

- `SKILL.md` — the full methodology (Chinese): scope, channel ranking with measured availability, embed-source verification criteria, new-keyword discovery, listing flow and maintenance rhythm.
- `README.md` — this English guide.

## How to use

1. Read `SKILL.md` and start from the highest-yield channel: pull the new / trending lists from aggregators that render server-side, then subtract titles already in your library.
2. Extend the pool with keyword mining, GitHub code search for embed URLs in other people's repositories, and image-based discovery. Record every new site you find — the pool grows with the number of sources.
3. Before any listing, verify the embed source against the hard criteria in `SKILL.md`. A candidate that renders a blank frame fails, regardless of how good the title looks.
4. Follow the listing flow and the SEO maintenance rhythm for newly added pages, and re-check sources periodically because third-party availability changes.

There are no runnable scripts in this package. It is prose guidance; URL patterns quoted in the document come from the tested project and are examples, not bundled tooling.

## Requirements

- Pure methodology text. No scripts, credentials, private buckets or build artifacts.
- All named channels are public sources. The document bundles no account, token or service dependency.

## Scope and limitations

- `SKILL.md` is in Chinese. This README is the English entry point, not a translation.
- The availability notes are a 2026-09 snapshot. Re-test any source before relying on it.
- This Skill covers the operating phase only; build and deployment are in the companion Skill.

## Origin

Extracted from a personal WorkBuddy skill library and cleaned for public release: machine-specific absolute paths were replaced with portable ones, backup and build artifacts were excluded, and no credentials are included.
