# WindToLog

Reads the BoM GPWT low level chart (SA) from image and turns the selected grid boxes into wind values ready for the NavLog_v12 sheet.

Built by SJX12.

## Overview

The tool is a single static HTML page with no server and no external API. The chart image is processed entirely in the browser using a purpose built OCR engine designed for this one chart format. Values are read box by box, checked for confidence, and combined into a route wind using vector averaging.

**Accuracy.** In blind testing on 27 charts (22,491 digits), each chart read using only what was learned from the others:

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

Ground truth comes only from the text layer of the BoM GPWT PDF, paired with the PNG of the same issue time. No manual labels are used.

Accuracy is measured with leave one chart out testing: each chart is read using only what was learned from the other charts, with the matching NAIPS and BoM pair of the same time excluded together. Results are shown in the Overview.

A Python reference implementation is used for training and testing, and the browser version is verified to produce identical readings and flags on every test chart.

## Privacy

Everything runs locally in the browser. Chart images and readings are never uploaded or stored.

## Disclaimer

This is a planning aid only. It is not an official source of meteorological information. Always check values against the current BoM chart before flight.

## Credits

Chart data: Bureau of Meteorology. Interface font: Fira Code via Google Fonts.
