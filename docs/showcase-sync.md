# Showcase sync

- **Source commit:** `709fde1` plus the uncommitted web redesign in the working tree (new brand mark, icons, solve animation, learning-path view)
- **Synced on:** 2026-10-14
- **Previous baseline:** `2fcc5c5` (2026-10-02)

## What changed

- Product: learning path (20 quests, 6 tracks), local progress without an account, spoiler-free quest comments with reports and back-office review, light and dark themes, five isolation anomalies, email confirmation before submitting.
- Architecture: Cloudflare Tunnel with one hostname and a Caddy admin gate, comment moderation, per-grade limits and one grade per user.
- Decision records moved to the private source repo (`docs/adr/`), which is now their source of truth; the showcase keeps only a curated "How I work" section with three examples.
- Security pass: the back office's URL path removed from public text; no IPs, ports, tokens, account IDs, credentials or personal emails in text or media.
- Operations: auto-deploy after CI over pinned SSH, pre-migration dumps, Sentry source maps and releases, optional New Relic logs and host metrics, assets kept across deploys.
- Stack: NestJS 12, TypeScript 6, Vite 7; web unit tests in CI; ~26k lines of TypeScript.
- Presentation: personal-project framing with an at-a-glance table, a monitoring and observability section (Sentry, New Relic Infrastructure and Logs via Fluent Bit, UptimeRobot, health checks, backups) and a full grouped tech stack, including hosting (Spaceship) and the edge (Cloudflare).
- Media: all four GIFs re-recorded on the new design; added a comments GIF and four screenshots.
