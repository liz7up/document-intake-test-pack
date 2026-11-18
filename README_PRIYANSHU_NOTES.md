# Synthetic Police Report Verification Pack (v0 Loop)

## Provisional validation defaults (not yet reviewed)

The `docs/10` draft v0 was sent on 3 Nov 2026. No review notes had arrived by 8 Nov, so this pack runs on the defaults below (also machine-readable in `00_provisional_policy.json`). They are configuration, not hard-coded: any notes will land as v0.1 without changing pipeline behaviour.

| Open question | Default used |
|---|---|
| Fuzzy edit distance on noisy scans | `1` |
| Check B strategy | `reference_or_label` |
| Redaction darkness / overlap thresholds | `45` / `0.30` |
| First N docs of a new layout to review | `3` |
| Second-pass verifier scope | `pattern_fields_only` |

## Files Included
0. `00_provisional_policy.json` — The five open questions above, marked `provisional_unreviewed`.
1. `01_police_report_redacted_hardstop.pdf` — Synthetic report where `Protected Witness / SSN` has a solid black vector rectangle drawn over intact underlying text in the PDF text layer.
2. `01_hardstop_output.json` — Output for Document #1:
   - `status`: `"stopped"` (`final_state: "stopped"`).
   - `hard_stop_report`: Triggered by `vector_rect` + `rendered_dark_region` overlap check on page 1.
   - `data` & `fields`: `{}` (underlying text is stripped during PDF inspection and never appears in the JSON).
3. `02_police_report_ocr_exceptions.pdf` — Separate synthetic report testing OCR damage, field states, and queue routing (without black-box redaction).
4. `02_exceptions_queue_flags.json` — Queue flags raised before human review:
   - `report_number` (`"RPT-2026-1O48"`): Flagged by `CHECK_C_POSITIONAL_OCR_DAMAGE` (position 10 expected digit `'D'` under mask `AAA-DDDD-DDDD`, found `'O'`) and `CHECK_C_SECOND_PASS_DISAGREE` (first pass `'RPT-2026-1O48'` vs second pass `'RPT-2026-1048'`, routed to review without auto-fixing).
   - `damage_estimate_usd` (`"[ILLEGIBLE SMUDGE: $1,?50.??]"`): Extractor set `state: "illegible"`, flagged by `STATE_ILLEGIBLE` and routed to review.
   - `witness_phone` (blank on form): Extractor set `state: "empty"` and `value: null` (did not guess).
5. `02_exceptions_final_approved_webhook.json` — Final webhook emitted once after human review approval (`final_state: "approved"`), carrying `document_id`, `idempotency_key`, and per-field `state: "corrected"`, `human_changed: true`, and `old_value`.
