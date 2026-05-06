# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A PlantUML macro library providing sprites, macros, and stereotypes for Azure API Management components. The single distributable file is `dist/v1/ApiManagement.puml`, which embeds all SVG sprites inline and exposes macros for use in `.puml` diagrams.

## Validating Changes

There is no build system. To verify that `.puml` files render correctly, use the PlantUML CLI or the [PlantUML VS Code extension](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml):

```bash
# Render a single diagram to PNG
java -jar plantuml.jar samples/hello-world.puml

# Render all samples
java -jar plantuml.jar samples/*.puml
```

The rendered PNGs in `samples/` should be updated when the corresponding `.puml` files change.

## Architecture

`dist/v1/ApiManagement.puml` has four sections:

1. **Styling** — global `skinparam` settings and CSS for table rendering in notes
2. **Generic Procedures** — `$setApimEntitySkin` and `$createApimBaseEntity` are the base helpers used by all entity macros
3. **Icons** — one block per resource type (User, Subscription, Product, API, Backend). Each block defines an inline SVG sprite, calls `$setApimEntitySkin`, then exposes a `$Apim<Type>` macro. Sprites that are small at default scale pass an explicit `$scale=3` to `$createApimBaseEntity`.
4. **Operations / Legend** — `$ApimOperations` renders a note-attached table; `$ApimSymbolLegend` renders a conditional legend using PlantUML's `!if` blocks.

SVG source files live in `dist/v1/sprites/*.svg` and are embedded verbatim into `ApiManagement.puml`.

## Macro Reference

| Name | Signature |
|------|-----------|
| `$ApimUser` | `$ApimUser($alias, $label)` |
| `$ApimSubscription` | `$ApimSubscription($alias, $label)` |
| `$ApimProduct` | `$ApimProduct($alias, $label)` |
| `$ApimAPI` | `$ApimAPI($alias, $label)` |
| `$ApimBackend` | `$ApimBackend($alias, $label)` |
| `$ApimOperations` | `$ApimOperations($api, $operations, $alignment="bottom")` |
| `$ApimSymbolLegend` | `$ApimSymbolLegend($includeUser, $includeSubscription, $includeProduct, $includeApi, $includeBackend, $alignment="bottom")` |

Raw sprites (`$ApimSprite`, `$ApimSubscriptionSprite`, etc.) are also available for use with `rectangle` syntax.

## CLI Tool

`src/` contains a .NET 10.0 global tool (`azure-apim-plantuml`) that will generate `.puml` diagrams from a live Azure API Management instance. The tool authenticates via `DefaultAzureCredential`, calls the Azure APIM REST API, and emits `.puml` files that `!include` `dist/v1/ApiManagement.puml`.

The tool is at an early skeleton stage — `Program.cs` contains only a placeholder. Feature work is driven by specs in `docs/specs/` (e.g., `FEAT-001-authenticate-to-azure.md`). The impact map and planned diagram types are in `docs/impact-map.md`.

### Development Commands

```bash
# Build
dotnet build src/AzureApimPlantuml.slnx

# Run tests
dotnet test src/AzureApimPlantuml.slnx

# Run a single test class
dotnet test src/AzureApimPlantuml.Tests --filter "ClassName=MyTestClass"

# Run locally
dotnet run --project src/AzureApimPlantuml

# Pack as a global tool
dotnet pack src/AzureApimPlantuml
```

### Project Structure

- `src/AzureApimPlantuml/` — main executable, packaged as a dotnet global tool
- `src/AzureApimPlantuml.Tests/` — MSTest 4 test project; main project exposes internals via `InternalsVisibleTo`
- `docs/specs/` — feature specs that drive implementation; read the relevant spec before implementing a feature
- `docs/architecture/` — arc42 architecture documentation (mostly scaffolding, fill in as decisions are made)
