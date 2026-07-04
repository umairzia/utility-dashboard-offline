# ⚡ Provident Utility Bill Dashboard

A fully offline, single-file HTML dashboard that parses Provident (pemi.com) utility eBill PDFs entirely in your browser and turns them into an interactive usage and cost breakdown. No internet connection, no server, no dependencies — your bills never leave your machine.

## Why

Provident bills bundle electricity, thermal cooling, thermal heating, and hot water into one statement, which makes it hard to see *what's actually driving your costs* month to month. This dashboard extracts every figure from your PDFs and shows you the trends, the seasonal patterns, and where your money goes.

## Features

- **Drag-and-drop upload** — add as many bill PDFs as you want, from any year
- **"Ready to begin?" flow** — review your queued files, then process on demand
- **Live processing view** — per-file status with the extracted total shown as confirmation
- **Interactive dashboard**:
  - 8 summary metric cards (total spend, monthly average, peak/lowest bill, avg electricity, peak cooling, OER rebate, HST)
  - Monthly bill totals bar chart with average line, highest/lowest highlighted
  - Electricity usage line chart
  - Thermal cooling vs. heating bar chart
  - Full bill-by-bill data table (consumption + charges)
  - Auto-generated insights and recommendations that adapt to your data
- **100% offline** — all PDF parsing and rendering happens in-browser
- **Multi-year** — mix bills from different years; everything sorts chronologically by statement date

## How it works

The interesting part is the PDF parser. Provident's eBills use **CID-subset fonts**, meaning the text isn't stored as plain ASCII — each byte in a text string is a glyph code that maps to Unicode through an embedded `ToUnicode` CMap table. A naive text extractor returns gibberish.

This parser handles it properly, in ~130 lines of vanilla JavaScript:

1. Indexes all indirect objects in the PDF (`N N obj … endobj`)
2. Decompresses each stream using the browser's built-in `DecompressionStream` API (FlateDecode)
3. Parses `beginbfchar` / `beginbfrange` blocks from every CMap to build byte→Unicode maps
4. Maps font resource names (`/9`, `/a`, `/h`) to their CMaps via each font's `/ToUnicode` reference
5. Walks the content streams, tracks the active font (`Tf`), and decodes all `Tj` / `TJ` text through the correct CMap

No `pdf.js`, no CDN, no build step. Just open the file.

## Usage

1. Download `utility-dashboard_v2.html`
2. Open it in any modern browser (Chrome, Edge, Firefox, Safari)
3. Drag in your Provident eBill PDFs (or click to browse)
4. Click **Ready to begin?**
5. Explore your dashboard

That's it. You can double-click the file straight from your file system — no local server needed.

## Browser support

Requires a browser with the `DecompressionStream` API:

| Browser | Minimum version |
|---------|-----------------|
| Chrome / Edge | 80+ |
| Firefox | 113+ |
| Safari | 16.4+ |

All are from 2023 or earlier, so any current browser works.

## Compatibility notes

Built and tested against Provident / pemi.com residential eBills for a bulk-metered building (electricity, thermal cool, thermal heat, hot water). The parser keys off Provident's specific statement layout. Bills from other utilities, or a significantly redesigned Provident template, may not parse correctly without regex adjustments in the `parseBill()` function.

Verified against 13 real bills spanning June 2024 – June 2026; every extracted total, consumption reading, and charge matched the source PDF exactly.

## Privacy

Everything runs locally in your browser. No network requests are made. Your bills, account number, and address are never uploaded anywhere. You can confirm this by opening the file with your browser's dev tools Network tab open — it stays empty.

## File structure

```
/
├── utility-dashboard-README.md
└── utility-dashboard.html    # The entire app — open in a browser
```

## License

MIT — do whatever you want with it.

---

*Not affiliated with or endorsed by Provident Energy Management / pemi.com. This is a personal tool for reading your own bills.*
