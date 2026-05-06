---
name: APIM CLI Tool Impact Map
description: Impact map for the .NET CLI tool that generates PlantUML diagrams from live Azure APIM instances
type: project
---

Impact mapping session completed 2026-05-06. The CLI tool will retrieve APIM config and generate PlantUML diagrams using the existing library at dist/v1/ApiManagement.puml.

**Key decisions:**
- Built as a .NET global tool (dotnet tool install -g)
- Four diagram types: product overview, product details, API details, backend details
- Each diagram has default elements (includable/excludable via CLI params) and optional elements (operations, admin subscriptions)
- Legend auto-reflects included entities (if products excluded, legend omits product entry)
- Generated .puml files use URL-based !include to the library by default; CLI param to override path/URL
- Export command saves APIM config to local file; diagram commands default to Azure but accept local file parameter
- MVP: all four diagram types + auth + data retrieval + config export + library include path + CLI interface + file output (12 features total)
- 1.0 adds: CI/CD support (non-interactive auth, exit codes)
- Cut features: PNG/SVG rendering (PlantUML handles this), diff between runs (git diff suffices), interactive review mode, web UI

**Why:** Teams have no diagrams or outdated ones. Generating from live APIM eliminates manual drift.

**How to apply:** All future architecture and implementation decisions should align with this scope. Resist scope creep beyond the defined MVP and 1.0 boundaries.

Impact map saved to: docs/impact-map.md
