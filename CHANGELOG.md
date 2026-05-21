# Changelog

All notable updates to `travel-product-creator` should be recorded in this
file.

## Update Rules

- Add a new dated section for every meaningful skill update.
- Record added, changed, fixed, or removed items in concise bullets.
- When templates, integrations, workflows, or schema contracts change, log them
  here.
- If a change affects how the skill should be used, mention the user-facing
  impact explicitly.

## 2026-05-21

### Added

- Added [integrations/canonical-schema.md](C:/Users/Administrator/.claude/skills/travel-product-creator/integrations/canonical-schema.md) to define a unified data contract across Excel, Feishu Base, and mixed sources.
- Added [integrations/excel-adapter-spec.md](C:/Users/Administrator/.claude/skills/travel-product-creator/integrations/excel-adapter-spec.md) to specify CSV and Excel ingestion behavior.
- Added [integrations/feishu-base-adapter-spec.md](C:/Users/Administrator/.claude/skills/travel-product-creator/integrations/feishu-base-adapter-spec.md) to replace vague Feishu MCP assumptions with a concrete Base adapter model via `lark-cli`.
- Added [integrations/field-mapping-examples.md](C:/Users/Administrator/.claude/skills/travel-product-creator/integrations/field-mapping-examples.md) with real mapping examples from client CSV files.
- Added [skill-refactor-outline.md](C:/Users/Administrator/.claude/skills/travel-product-creator/skill-refactor-outline.md) to document the proposed system refactor path.
- Added [templates/product-doc.md](C:/Users/Administrator/.claude/skills/travel-product-creator/templates/product-doc.md) to restore the missing deep-flow product document output.

### Changed

- Reserved a future pricing extension in the canonical architecture through the `rate_cards` object and pricing-layer references.
- Rebuilt [templates/simple-itinerary.md](C:/Users/Administrator/.claude/skills/travel-product-creator/templates/simple-itinerary.md) into a client-facing finished itinerary structure based on a full product handout format, including opening copy, product narrative, quick-look schedule, cuisine, accommodation, detailed day-by-day storytelling, service standards, exclusions, and travel notes.
- Reworked [templates/sales-pitch.md](C:/Users/Administrator/.claude/skills/travel-product-creator/templates/sales-pitch.md) into a shorter sales companion that stays aligned with the new client-facing itinerary structure.
- Clarified the user-facing output split: `simple-itinerary.md` is now the main customer delivery template, while `sales-pitch.md` serves as the supporting sales summary.
- Rewrote [SKILL.md](C:/Users/Administrator/.claude/skills/travel-product-creator/SKILL.md) around source adapters, canonical schema, Gaode validation boundaries, and the new customer-facing vs internal output contract.

### Fixed

- Fixed a broken output dependency by adding the previously missing `product-doc.md` template referenced by the skill design.

## Future Requirement

For all future updates to this skill, append a new dated section here before
considering the update complete.
