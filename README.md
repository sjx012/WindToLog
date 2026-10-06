# WindToLog

Reads the BoM GPWT low level chart (SA) from image and turns the selected grid boxes into wind values ready for the NavLog_v12 sheet.

Open the tool: https://sjx012.github.io/WindToLog/

Built by SJX12 for Starlux cadets flying at FTA.

## Overview

The tool is a single static HTML page with no server and no external API. The chart image is processed entirely in the browser using a purpose built OCR engine designed for this one chart format. Values are read box by box, checked for confidence, and combined into a route wind using vector averaging.

**Accuracy.** In held out testing on 57 charts (47,481 digits), each chart read using only what was learned from the others:

| Measure | Result |
|---|---|
| Digits read correctly | 47,480 of 47,481 (99.998 %) |
| Wrong digits shown as confident | 0 |
| Digits flagged for a quick check | 0.02 % (NAIPS), 0.01 % (BoM) |

A route only uses a few boxes, so most flights see no flag at all.

Both chart sources are supported:

| Source | Image size | Notes |
|---|---|---|
| NAIPS | 730 × 867 | 256 colour palette image |
| BoM website | 730 × 878 | Full colour, slightly different box geometry |

Retina screenshots at 2x or 3x are scaled back to chart resolution. Other zoom levels are rejected.

## Training and validation

### Ground truth

Every label comes from the text layer embedded in the official BoM GPWT PDF, paired with the PNG charts of the same valid time. No label is entered by hand. The current dataset holds 57 chart images (28 NAIPS, 29 BoM) and 96,222 labelled digits.

### What is learned

The digits are drawn with identical pixels everywhere on the chart, and the coastline is a see through green line drawn on top. The engine learns both, separately for each layout:

| Component | Content |
|---|---|
| Glyph bank | Digit exemplars in black, red and blue text, selected for maximum diversity |
| Position memory | Real digit appearances at every coastline position |
| Virtual memory | How much the coastline covers each pixel, plus the clean look of every digit. Together they let the engine draw how any digit would look at any spot, including digits never seen there |

Red and blue digits are exact recolourings of the black ones, so a value never seen in one colour can still be read.

### Validation

Leave one valid time out cross validation: each chart is read using only what was learned from the other charts, and the NAIPS and BoM charts of the same time are always held out together.

The key safety measure is the silent error, a wrong digit that is not flagged. The release target is zero.

| Source | Charts | Digits | Correct | Flagged | Silent errors |
|---|---|---|---|---|---|
| NAIPS | 28 | 23,324 | 23,323 | 5 (0.02 %) | 0 |
| BoM | 29 | 24,157 | 24,157 | 3 (0.01 %) | 0 |
| Total | 57 | 47,481 | 47,480 (99.998 %) | 8 (0.02 %) | 0 |

The one wrong digit was flagged. With zero silent errors in 47,481 digits, the true silent error rate is below 0.01 % at 95 % confidence.

### Contribution of each stage

| Stage | Flagged | Wrong digits | Silent errors |
|---|---|---|---|
| Template matching, fixed confidence rule | 6.81 % | 39 | 14 |
| + stricter confidence near the coastline | 7.10 % | 39 | 0 |
| + position memory | 1.18 % | 39 | 0 |
| + virtual memory | 0.02 % | 1 | 0 |

### Generalisation

How the engine copes with values and positions it has never met, as a new season would bring:

| Test | Digits | Correct | Silent errors |
|---|---|---|---|
| Trained on older charts, tested on newer ones | 24,990 | 99.996 % | 0 |
| A value never seen at that coastline spot | 614 | 99.8 % | 0 |
| Boxes outside the trained area, with no coastline map | 141,645 | 99.88 % | 0 |
| A temperature digit never seen in red or blue | 1,747 | 100 % | 0 |

An unfamiliar situation produces more flags, never a confident wrong value.

### Release checks

Every update is tested on newly issued charts before they join training, cross validated with zero silent errors, and the browser page is verified to reproduce the Python reference readings and flags exactly on every box of every chart.

## Privacy

Everything runs locally in the browser. Chart images and readings are never uploaded or stored.

## Disclaimer

This is a planning aid only. It is not an official source of meteorological information. Always check values against the current BoM chart before flight.

## Credits

Chart data: Bureau of Meteorology. Interface font: Fira Code via Google Fonts.
