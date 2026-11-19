# Synthetic Police Report Verification Pack (v0.1 — Merged Review & Observations)

## Status of numeric limits

All numeric limits in `00_v0_1_policy.json` and `VALIDATION_RULES_v0_1.md` remain **unverified reasoning to test** (configurable in `ValidationPolicy`), not production-proven constants. Table-header column mapping (Q2) is documented in `VALIDATION_RULES_v0_1.md` and deferred in code until the first table layout.

## Summary of v0 → v0.1 Changes

| Source | Change in v0.1 |
|---|---|
| Q1 + `g` | Check A compares normalised text strictly **inside the cited `bbox`** (`±4pt` padding). Short name fields on scans allow length-scaled edit distance (`<=12` chars → `1`, `>12` → `2`), keeping the extractor's value and never swapping in OCR text. |
| Q2 | Table-header unique x-overlap rule written into `VALIDATION_RULES_v0_1.md`; implementation deferred (no tables in current schema). |
| Q3 + `a` | Retained vector box check and added **250 DPI colour-independent per-glyph rendered contrast check** (annotations included, lower threshold `8.0` on thin glyphs). `render_mode=3` does not trigger a stop by itself, while its glyph boxes still undergo rendered contrast checking. Added unapplied `/Redact` annotation and outside-`CropBox` stops. `detection_methods` is now a multi-valued list. |
| Q4 + `e` | `new_layout_review_count_n` updated `3 → 10` (`enforce_new_layout_warmup = true`). Layout graduation requires `approved_count >= 10` human-reviewed docs (never raw `seen_count`) within `layout_correction_rate_limit = 0.20`, reverting to full review if weekly sample corrections exceed the limit (or on any correction for zero-correction graduations). `layout_stats` persists per `workspace + sender_domain + layout_key`. |
| Q5 + `d` | Replaced JSON-seeing verifier with an **independent blind second-pass call per page** covering only `found` patterned fields (`complainant_name` and `incident_summary` carry `second_pass_value: null` and `check_c_second_pass: "not_applicable"`). Isolated `second_pass_timeout_ms` flags timed-out fields `SECOND_PASS_NOT_VERIFIED` → review (never counted as pass), alerting when share exceeds `not_verified_alert_share`. |
| `b` | Stop records and `stopped` webhook `source` include `extractor_called: false`; stop reason states pre-extraction halt (`halted before extraction`). |
| `c` | **Stop is held first (`stopped_held`)**: triggers owner alert with `extractor_called: false` and emits **no webhook** until a person either confirms real improper redaction (`status: "stopped"` → single `document.final(stopped)`) or clears a false stop (enters extraction → validation → human review → single `document.final(approved)`). Every `document_id` emits `document.final` at most once. |
| `f` | Human-reviewed approvals set `check_status: "human_approved"` and per-field `value_source: "extractor" | "human" | "none"`. |
| `g` + check-3 | Character positions unified to **1-based** (`position: 11` on `'RPT-2026-1O48'`; `null` on length mismatch). Every output and final webhook includes a full per-field `field_checks` ledger (`passed | failed | not_run | not_applicable`). |

## Files Included

0. `00_v0_1_policy.json` & `VALIDATION_RULES_v0_1.md` — Unverified v0.1 policy parameters and merged English rule specification.
1. `01_police_report_redacted_hardstop.pdf` & `01_hardstop_output.json` — Full black vector box over 42-char run: shows `held_state_before_confirmation` (`stopped_held`, `extractor_called: false`, `webhook_emitted: false`), `detection_methods: ["vector_rect", "glyph_contrast"]`, and confirmed `stopped` payload with empty `data`/`fields`.
2. `02_police_report_ocr_exceptions.pdf`, `02_exceptions_queue_flags.json`, `02_exceptions_final_approved_webhook.json` — `RPT-2026-1O48` flagged at **1-based position 11** + blind 2nd-pass disagreement; free-text `second_pass_value: null`; approved webhook carries `check_status: "human_approved"`, `value_source`, and full `field_checks` ledger.
3. `03_police_report_clean.pdf`, `03a_clean_warmup_review.json`, `03b_clean_graduated_auto_approved.json` — Both paths on Document #3: `03a` cold layout (`approved_count < 10` → held in review by `LAYOUT_NEW_WARMUP`) and `03b` graduated layout (`approved_count = 10` → `auto_approved` webhook with `check_status: "passed"` and all `field_checks` visible).
4. `04_police_report_partial_redaction.pdf` & `04_partial_redaction_output.json` — Partial cover over only the trailing 11-char SSN (`overlap_ratio ≈ 0.234 < 0.30`): missed by span-level `vector_rect`, caught by per-glyph `glyph_contrast` (`redacted_char_count: 11`), zero SSN leakage.
5. `05_police_report_scan_ocr.pdf` & `05_scan_ocr_output.json` — 200 DPI raster scan + `render_mode=3` invisible OCR layer (`'Elena Rostava'` in OCR layer, `'Platform 12'` in summary): verifies `render_mode=3` does not false-stop, 1-char name edit distance passes while keeping extractor's `'Elena Rostova'`, and 2-char difference routes to review.
