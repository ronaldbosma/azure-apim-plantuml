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

## Planned CLI Tool

`docs/impact-map.md` describes a planned .NET global tool (`dotnet tool install -g`) that generates `.puml` diagrams from a live Azure API Management instance. It has not been implemented yet. The tool will authenticate via `DefaultAzureCredential`, call the Azure APIM REST API, and emit `.puml` files that `!include` `dist/v1/ApiManagement.puml`.
