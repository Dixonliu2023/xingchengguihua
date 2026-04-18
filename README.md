# travel-product-creator

Claude/Codex travel product design skill for end-to-end itinerary planning, product packaging, and route validation.

## Contents

- `SKILL.md`: primary skill definition
- `integrations/gaode/`: Gaode (Amap) MCP integration notes
- `integrations/`: other integration references
- `references/`, `templates/`, `workflows/`, `assets/`, `examples/`, `config/`: supporting materials

## Publish Target

This repository packages the local `travel-product-creator` skill so it can be versioned and shared through GitHub.

## Notes

- The skill references Gaode MCP via the files under `integrations/gaode/`.
- Runtime credentials such as API keys should be provided through environment variables and must not be committed.

## License

MIT
