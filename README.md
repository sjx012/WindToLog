# WindToLog

Reads the BoM GPWT low level chart (SA) from image and turns the selected grid boxes into wind values ready for the NavLog_v12 sheet.

Built by SJX12.

## Overview

The tool is a single static HTML page with no server and no external API. The chart image is processed entirely in the browser using a purpose built OCR engine designed for this one chart format. Values are read box by box, checked for confidence, and combined into a route wind using vector averaging.

**Accuracy.** In held out testing on 27 charts (22,491 digits), each chart read using only what was learned from the others:

| Measure | Result |
|---|---|
| Digits read correctly | 22,491 of 22,491 (100 %) |
| Wrong digits shown as confident | 0 |
| Digits flagged for a quick check | 0.04 % (NAIPS), 0.01 % (BoM) |

That is about one flagged digit every five charts across the whole 20 box area. A route only uses a few boxes, so most flights see none.

Both chart sources are supported:

| Source | Image size | Notes |
|---|---|---|
| NAIPS | 730 × 867 | 256 colour palette image |
| BoM website | 730 × 878 | Full colour, slightly different box geometry |

Retina screenshots at 2x or 3x are scaled back to chart resolution. Other zoom levels are rejected.

## Training and validation

### Ground truth

Every label comes from the text layer embedded in the official BoM GPWT PDF. Each PDF is paired with the PNG charts of the same valid time, and its text is mapped onto the 9 × 9 grid by position. No label is entered by hand.

| Ground truth audit | Value |
|---|---|
| PDF issues | 14 consecutive 3 hourly valid times, 03 Oct 09Z to 05 Oct 00Z 2026 |
| Boxes parsed | 1,134 |
| Forecast lines | 6,804 (6,636 with data, 168 blank on the chart) |
| Labelled digits | 46,452 |
| Format check failures | 0 |
| Value range covered | Wind 0 to 54 kt, temperature −12 to +32 °C |

### Dataset

27 chart images with PDF labels: 13 NAIPS and 14 BoM. Accuracy is measured on the 20 box planning area (columns 5 to 9, rows 6 to 9), which gives 3,213 forecast lines and 22,491 digits. Of these, 2,715 digits (12.1 %) are touched by the coastline and 1,411 are more than 20 % covered by it.

### Learned components

Every component is rebuilt from scratch at each retrain, separately for each layout.

| Component | Content | Size in page |
|---|---|---|
| Glyph bank | 59 digit exemplars across black, red and blue text, selected for maximum diversity | 14 KB |
| See through map | Paper colour under the coastline for 1,522 pixels | 10 KB |
| Position memory | 1,226 real digit appearances at 261 coastline positions | 227 KB |
| Virtual memory | Coastline cover and green level for 11,088 pixels, plus 154 clean digit appearances | 70 KB |

### Validation protocol

Leave one valid time out cross validation, 27 folds. For each chart, every learned component is rebuilt using only the other charts. The NAIPS and BoM charts of the same valid time carry identical values, so they are always held out together to prevent leakage.

Three measures are reported:

| Measure | Definition |
|---|---|
| Digit accuracy | Final value equals the PDF label |
| Flag rate | Share of digits highlighted for a manual check |
| Silent error | A wrong digit that is not flagged. This is the safety critical measure, and the release target is zero |

### Results

| Source | Charts | Digits | Correct | Flagged | Silent errors |
|---|---|---|---|---|---|
| NAIPS | 13 | 10,829 | 10,829 | 4 (0.04 %) | 0 |
| BoM | 14 | 11,662 | 11,662 | 1 (0.01 %) | 0 |
| Total | 27 | 22,491 | 22,491 (100 %) | 5 (0.02 %) | 0 |

23 of the 27 charts produced no flag at all, and all 2,715 coastline affected digits were read correctly. With zero errors in 22,491 digits, the true error rate is below 0.014 % (about 1 in 7,500) at 95 % confidence.

### Contribution of each stage

Same protocol and data, adding one stage at a time:

| Stage | Flagged | Wrong digits | Silent errors |
|---|---|---|---|
| Template matching, fixed confidence rule | 6.41 % | 14 | 0 |
| + confidence scaled by coastline coverage | 4.78 % | 14 | 0 |
| + position memory | 0.92 % | 14 | 0 |
| + virtual memory | 0.02 % | 0 | 0 |

The virtual memory removes 97.6 % of the remaining flags (206 to 5) and corrects all 14 wrong first readings, because it can recognise a digit under the coastline even at a position where that digit was never seen.

### Prospective testing

Before new charts are added to training, the released build is scored on them as unseen data, with all thresholds already fixed. Latest test (04 Oct 21Z and 05 Oct 00Z, 4 charts): 3,332 of 3,332 digits correct, 4 flagged, 0 silent errors. Across all three prospective tests so far (21 newly issued charts), no wrong digit has been shown unflagged.

### Release checks

A Python reference implementation is used for training and testing. Every release passes the same checks:

1. Prospective test of the current build on the newly issued charts.
2. Cross validation on the full dataset with zero silent errors.
3. Browser parity: the page must reproduce the reference readings and flags exactly (latest: 3,240 of 3,240 lines identical).

## Privacy

Everything runs locally in the browser. Chart images and readings are never uploaded or stored.

## Disclaimer

This is a planning aid only. It is not an official source of meteorological information. Always check values against the current BoM chart before flight.

## Credits

Chart data: Bureau of Meteorology. Interface font: Fira Code via Google Fonts.
