# Date: September 24, 2025
# AI Incident Log

A running log of notable AI issues, failures, and lessons learned. Use concise entries with actionable takeaways.

## 2025-09-24
- Model: Gemini
- Task: Extract numbers from a spreadsheet column
- Outcome: Hallucinated ~50% of values (invented numbers not present in source)
- Likely cause: Overbroad instruction; insufficient grounding to source cells
- Safer pattern: Ask for a spreadsheet formula to extract numbers deterministically
- Example prompt: "Provide a Google Sheets formula to extract numeric values from cells in column B, preserving original order and leaving non-numeric cells blank."
- Action: Prefer formula-generation or programmatic parsing for deterministic tasks; reserve LLM extraction for fuzzy inputs with human review.

## Taxonomy
- Type: Data integrity / hallucination
- Severity: Medium (incorrect outputs, easy to miss)
- Mitigation tags: determinism, verification, formula-first
