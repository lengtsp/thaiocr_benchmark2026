# BOT Credit Bureau Law Golden OCR Dataset - 2026-08-29

## Dataset identity

- Dataset ID: `bot_credit_bureau_21p_golden_20260829`
- Document: `พระราชบัญญัติการประกอบธุรกิจข้อมูลเครดิต พ.ศ. ๒๕๔๕`
- Language: Thai (`th`)
- Scope: 21 rendered source pages.
- Primary ground truth: `inputs/bot_credit_bureau_2559/pages_2200_gray220/page-*.png`.
- Corroborating visual source: `inputs/bot_credit_bureau_2559/pages_2200/page-*.png`.

## Schema

The JSON has a top-level `pages` list. Each page has `page` (integer), `text` (UTF-8 transcription), `review_status`, and `source_image`. It also records the source PNG SHA-256, review provenance, notes, an explicit correction list, and `uncertain_glyphs`. Top-level fields record document identity, source policy, transcription policy, and review summary.

`text` is a presentation-aware ground-truth transcription intended for later character-, word-, and document-level OCR scoring. Line breaks preserve useful visible structure but should be normalized by a scorer only when the evaluation protocol explicitly says to do so.

## Source and Review Procedure

Every page from 01 through 21 was visually inspected on 2026-08-29 against the 2200-pixel gray-level source PNG. The 2200 original-render PNG was available as a visual corroboration source. The PDF text layer was not used as ground truth because it is known to be corrupted. The existing Typhoon OCR result was used only as a comparison draft and was corrected where the page image showed a discrepancy.

Review covered body text, section and chapter headings, Thai page folios, repeated `สำนักงานคณะกรรมการกฤษฎีกา` headers, visible footnotes and amendment notes, and the names/dates on page 21. The 21 page images have SHA-256 values in the JSON for source binding.

## Scope and Exclusions

Included: all visible legal text, repeated office headers, page folios, headings, Thai digits, footnote text, amendment references, and page-21 attribution names.

Excluded: non-text decorative horizontal rules. Superscript position is represented as ordinary Thai digits in the surrounding transcription; the digit identity is retained, while typography is deliberately not modeled. No hidden PDF text, inferred missing material, or editorial modernization was introduced.

## Correction Ledger

- All pages: removed draft-only structural markup where present; visually confirmed text against source images.
- Pages 1-17: converted draft Latin superscript footnote symbols to the Thai digits visibly printed in the source where applicable.
- Page 1: removed the draft Markdown heading marker from the title.
- Page 18: restored the visible `๑๙` reference following `มาตรา ๖๔`.
- Page 19: restored visible amendment references `๒๐` and `๒๑` after the amendment titles.
- Page 20: attached visible amendment references `๒๒` and `๒๓` to their respective titles instead of retaining draft line-break artifacts.
- Page 21: visually confirmed `อังศุมาลี`, `อุดมลักษณ์`, `วิชพงษ์`, `ชนิกา`, and `ปริญสินีย์`, with their dates and roles.

## Limitations

This is a single-reviewer image-controlled reference, not a multi-annotator adjudicated legal edition. It is a transcription benchmark, not legal advice or a replacement for the official gazette. Layout semantics beyond line breaks, such as exact x/y coordinates, font styling, and superscript placement, are outside this schema. The review found no unresolved glyphs in the 21 rendered pages; `uncertain_glyphs` is empty for every page.
