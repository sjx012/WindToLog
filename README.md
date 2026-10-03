# WindToLog

Reads the BoM GPWT low level chart (SA) from image and turns the selected grid boxes into wind values ready for the NavLog_v12 sheet.

Built by SJX12.

## Overview

The tool is a single static HTML page with no server and no external API. The chart image is processed entirely in the browser using a purpose built OCR engine designed for this one chart format. Values are read box by box, checked for confidence, and combined into a route wind using vector averaging.

Both chart sources are supported:

| Source | Image size | Notes |
|---|---|---|
| NAIPS | 730 × 867 | 256 colour palette image |
| BoM website | 730 × 878 | Full colour, slightly different box geometry |

Retina screenshots at 2x or 3x are scaled back to chart resolution. Other zoom levels are rejected.

## How the OCR works

General purpose OCR is built for any font, size and layout. The GPWT chart is the opposite: one font, one size, fixed digit cells and a fixed coastline. The engine exploits that to stay small and fast while being highly accurate.

**1. Layout and grid**
Grid lines are found as long, unbroken dark runs, which keeps stacked digit strokes from being mistaken for lines. Box edges are then completed from the regular spacing, so a line hidden by the legend or the coastline is still recovered. The image size identifies the layout (NAIPS or BoM).

**2. Line and field alignment**
Each box holds six levels. The top of every text line is located from the ink profile. Within each box, the direction, speed and temperature fields are aligned separately by fitting the expected digit cells to the column ink profile (shift range of ±3 px). This absorbs the per box offsets that differ between NAIPS and BoM rendering.

**3. Digit matching**
Each digit cell is compared with a library of labelled glyph exemplars using Gaussian blurred template matching with a small positional search. Temperature sign is taken from text colour (red positive, blue negative). Coastline pixels are excluded from the comparison rather than guessed.

**4. Seeing through the coastline**
Where the coastline is drawn semi transparently, the ink underneath is reconstructed from how much each pixel darkens relative to its learned "over white paper" value. That reference is learned per layout from training charts, and a pixel is only trusted when its reference colour was observed over different digits, so repeated ink can never be mistaken for paper. This is used for black digits only, since palette quantisation distorts red and blue under green.

**5. Context check**
Uncertain digits are re-ranked using the neighbouring levels of the same box. Direction is solved across the whole column (Viterbi style) with a capped angular cost, weighted down in light winds where direction genuinely varies. Speed and temperature receive a smaller nudge from confident neighbours. Confident digits are never changed.

**6. Confidence and flagging**
Every digit carries a match error, a margin to the next best digit, and the fraction hidden by the coastline. A digit is flagged for the user when the margin is below a threshold that scales with coastline coverage. A position memory of how each digit looked at each coastline spot on verified charts can confirm, but never overrule, a borderline reading. Flagged values are highlighted and shown beside a zoomed crop of the chart for a quick check.

## Wind combining

Selected boxes are combined per level by vector (u/v) averaging, not by averaging directions as numbers. When the boxes disagree enough for their winds to partly cancel, a steadiness warning is shown with a diagram of the individual and combined vectors, and the user is advised to use separate values per leg.

## Training and validation

Ground truth comes from the text layer of the BoM GPWT PDF, which is paired with the PNG of the same issue time. No manual labelling is needed.

Accuracy is measured with leave one chart out testing: each chart is read using only what was learned from the other charts, with the matching NAIPS and BoM pair of the same time excluded together.

As of October 2026 (35 charts, 29,155 digits in the cadet block):

| Measure | Result |
|---|---|
| Digits read incorrectly | 16 |
| Incorrect digits not flagged | 0 |
| Digits flagged for review | about 2 % (NAIPS), 3 % (BoM) |

A Python reference implementation is used for training and testing, and the browser version is verified to produce identical readings and flags on every test chart.

## Privacy

Everything runs locally in the browser. Chart images and readings are never uploaded or stored.

## Disclaimer

This is a planning aid only. It is not an official source of meteorological information. Always check values against the current BoM chart before flight.

## Credits

Chart data: Bureau of Meteorology. Interface font: Fira Code via Google Fonts.
