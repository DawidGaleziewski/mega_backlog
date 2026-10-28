# mega_backlog

Tickets split out of the support desk's ticket **#4471** ("dates parse mm/dd in eu locales", 412 reports, marked known), and the kb page about them. MEGA.dev Challenge S01E03 *what it shows*.

## Tickets

| # | Problem | Reports |
|---|---|---|
| [#1](https://github.com/DawidGaleziewski/mega_backlog/issues/1) | Date parser reads day-first dates month first: day and month swap | 379 |
| [#2](https://github.com/DawidGaleziewski/mega_backlog/issues/2) | booking-v3 form hint says MM/DD/YYYY but the form reads day first | 33 |
| [#3](https://github.com/DawidGaleziewski/mega_backlog/issues/3) | Dates move by exactly one day, not a day/month swap | 5 |

412 unique reports (the export has 427 rows; 15 ids appear twice). Counted the desk's way, by tenant: 379 + 33 = 412; the 5 one-day reports of #3 are among the 379, where the desk files them.

## The kb page

- Source (.svx): [`kb/src/routes/kb/incidents/4471-split/+page.svx`](kb/src/routes/kb/incidents/4471-split/+page.svx), signed `Dawid Galeziewski` (desk ref `DR-40PV-1MBD`).
- Its table component: [`kb/src/lib/kb/SplitTable.svelte`](kb/src/lib/kb/SplitTable.svelte). It computes the three counts from the kb's data with the kb's own `causes.ts` and the desk's `desk-notes.json`; no number is typed into the page.
- As the desk sees it: [`rendered/kb/incidents/4471-split.html`](rendered/kb/incidents/4471-split.html) (built site, CSS inlined) and a screenshot, [`rendered/4471-split.png`](rendered/4471-split.png).

## What is where

- `kb/`: the support kb export (`kb-export-267efc5e.zip`, exported 2026-10-04) as received, plus the two files above. Nothing else in it is changed.
- `rendered/`: the whole kb built with SvelteKit 3 + mdsvex (adapter-static), so every page, including the new one, is as the desk's site renders it.
