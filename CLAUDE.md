# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Single-script Selenium scraper that logs into iFood (https://www.ifood.com.br/pedidos),
collects the user's recent orders, and writes a styled XLSX expense report
(`pedidos_ifood_30dias.xlsx`) with per-order rows plus total/average/count.
Code and UI text are in Brazilian Portuguese.

## Commands

```bash
pip install -r requirements.txt
python iFoodExpenseReport.py        # opens Edge, waits for manual login, then scrapes
```

There are no tests, linter, or build step. The `.pyproj`/`.slnx` files exist only
for opening the project in Visual Studio; the script runs standalone with `python`.

## Architecture

Everything lives in `iFoodExpenseReport.py`. The flow in `main()`:

1. **`build_driver()`** launches Edge with anti-detection flags and a **persistent
   profile** (`edge_profile/`). This is load-bearing: the iFood login session and
   Cloudflare clearance survive across runs, so the user usually only logs in once.
   Do not switch to a throwaway/headless profile without understanding this.
2. The script navigates to the orders page and **blocks on `input()`** for the user
   to log in manually in the opened browser — it is interactive by design, not
   headless/CI-friendly.
3. **Link collection** scrolls + clicks "Ver mais pedidos" repeatedly, counting
   `a[href*='/pedido/']` links; it stops after 4 stagnant rounds (no new links).
4. **Per-order extraction** visits each order URL and pulls three fields via the
   `extrair_*` functions. Each extractor tries a list of specific CSS selectors,
   then falls back to regex over the page body text — this layered fallback is
   deliberate because iFood's SPA markup changes. When fixing broken scraping,
   add/adjust selectors at the front of these lists rather than replacing the
   regex fallbacks.
5. **`cache_pedidos.json`** (keyed by order GUID) memoizes extracted
   `{restaurante, total, data}` and is saved after every newly-scraped order, so
   re-runs skip already-seen orders. Stale/wrong cached entries must be cleared
   from this file to force re-scraping.
6. Orders older than `DIAS` (default 30, top of file) are dropped, then `openpyxl`
   writes and (on Windows) auto-opens the XLSX.

## Locale parsing

Values are Brazilian-formatted: `parse_total` converts `"R$ 66,10"` → `66.10`
(strips `.` thousands sep, `,` → `.`); `parse_data` reads `dd/mm/aaaa`. The total
extractor specifically avoids matching "Subtotal" via `(?<![Ss]ub)\bTotal\b`.

## Gitignored artifacts

`edge_profile/`, `cache_pedidos.json`, and generated `.xlsx` are runtime output —
do not commit them.
