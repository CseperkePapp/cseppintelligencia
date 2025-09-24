# Image Generation Experiments

A logbook for comparing models, prompts, and outcomes.

## Findings (2025-09-24)
- Gemini: Better at clean, polished portraits; tends to produce smoother skin and controlled lighting.
- ChatGPT (DALL·E): Strong at textured, rugged looks; more character in skin, grit, and environment.
- Safety behavior: ChatGPT refused image generation when asked to mix celebrity features. Avoid that framing; use non-celebrity descriptions.

## Tips
- Keep comparisons controlled: same seed (if supported), same base prompt, vary one factor at a time.
- Log: model, version, seed, resolution, guidance, negative prompts, reference images used.
- Save prompts and outputs in subfolders per experiment.

## Structure
- /experiments/YYYY-MM-DD-topic
  - prompt.txt
  - notes.md
  - outputs/

