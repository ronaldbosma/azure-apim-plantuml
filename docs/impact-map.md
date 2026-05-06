# Impact Map: Azure APIM PlantUML CLI Tool

> Generated: 2026-05-06

---

## 1. Goal

| Goal | Why | Success Criteria |
|------|-----|------------------|
| Generate valid PlantUML diagrams from a live Azure APIM environment to keep documentation in sync with reality | Teams either have no APIM diagrams or maintain them manually, which means they drift out of date. Automating generation from the live environment eliminates that drift. | The CLI produces valid `.puml` files that render correctly using the existing PlantUML library in this repository (`dist/v1/ApiManagement.puml`). |

---

## 2. Actors

| Actor | Needs | Problems | Related Goal |
|-------|-------|----------|--------------|
| **Developer** | Understand which APIs, products, backends, and subscriptions exist in their APIM instance. Generate diagrams without manual drawing. | Has no diagrams or limited/outdated diagrams of their APIM setup. Manual diagram creation is tedious and falls behind reality. | Generate valid PlantUML diagrams from live APIM |
| **Architect** | Review the overall APIM architecture. Communicate structure to stakeholders and team members. | Has no diagrams or limited/outdated diagrams. Cannot quickly get a visual overview of the APIM topology. | Generate valid PlantUML diagrams from live APIM |

---

## 3. Impacts (Behavioral Changes)

| # | Impact | Actor(s) | Phase |
|---|--------|----------|-------|
| 1 | **START** generating up-to-date APIM diagrams instead of manually drawing them (or having none) | Developer, Architect | MVP |
| 2 | **STOP** relying on outdated or nonexistent APIM documentation | Developer, Architect | MVP |
| 3 | **START** including diagram generation in CI/CD pipelines | Developer, Architect | Post-MVP |

---

## 4. Deliverables

### 4.1 Diagram Types

The tool generates four diagram types. Each diagram type has a set of default elements and optional elements. All default elements can be excluded via CLI parameters.

| Diagram Type | Default Elements | Optional Elements |
|-------------|-----------------|-------------------|
| **Product overview** | All products, subscriptions, APIs, backends, legend | Operations, admin subscriptions |
| **Product details** | Specific product's subscriptions, APIs, backends, legend | Operations, admin subscriptions |
| **API details** | Specific API's subscriptions, products, backends, legend | Operations, admin subscriptions |
| **Backend details** | Specific backend's subscriptions, products, legend | Operations, admin subscriptions |

### 4.2 Feature Breakdown

#### Epic 1: APIM Data Retrieval

| Feature | Description | Traces to |
|---------|-------------|-----------|
| **E1.F1** Authenticate to Azure | Authenticate using Azure Identity (e.g. `DefaultAzureCredential`) so the tool works with local credentials (az login, managed identity, etc.) | Impact 1 |
| **E1.F2** Retrieve APIM configuration | Call the Azure APIM REST API (or SDK) to fetch products, APIs, backends, subscriptions, and operations for a given APIM instance. By default, diagram generation commands load configuration from Azure. If a local export file is specified via CLI parameter, use that instead. | Impact 1 |
| **E1.F3** Export APIM configuration to local file | Provide a standalone `export` command that retrieves the APIM configuration from Azure and saves it to a local file. This allows generating multiple diagrams without repeated Azure API calls. | Impact 1 |

#### Epic 2: PlantUML Generation

| Feature | Description | Traces to |
|---------|-------------|-----------|
| **E2.F1** Generate product overview diagram | Produce a `.puml` file showing all products with their default relationships, using the macros from `dist/v1/ApiManagement.puml` | Impact 1, 2 |
| **E2.F2** Generate product details diagram | Produce a `.puml` file for a specific product | Impact 1, 2 |
| **E2.F3** Generate API details diagram | Produce a `.puml` file for a specific API | Impact 1, 2 |
| **E2.F4** Generate backend details diagram | Produce a `.puml` file for a specific backend | Impact 1, 2 |
| **E2.F5** Include/exclude diagram elements | Support CLI parameters to include optional elements (operations, admin subscriptions) and exclude default elements. The generated legend must automatically reflect only the entity types that are present in the diagram (e.g. if products are excluded, the legend omits the product entry). | Impact 1 |
| **E2.F6** Library include path | Generated `.puml` files use a `!include` with a URL to the PlantUML library (`dist/v1/ApiManagement.puml`) by default. An optional CLI parameter allows overriding the include path/URL to point to a custom or local copy of the library. | Impact 1 |

#### Epic 3: CLI Interface

| Feature | Description | Traces to |
|---------|-------------|-----------|
| **E3.F1** .NET global tool packaging | Package as a `dotnet tool install -g` global tool | Impact 1 |
| **E3.F2** CLI commands and parameters | Provide commands for each diagram type with parameters for APIM instance name, resource group, subscription ID, output path, element include/exclude flags, optional local config file path (to use an export instead of loading from Azure), and optional library include path/URL override | Impact 1 |
| **E3.F3** Output to file | Write generated `.puml` content to a specified file path (default: stdout or current directory) | Impact 1, 2 |

#### Epic 4: CI/CD Support (Post-MVP)

| Feature | Description | Traces to |
|---------|-------------|-----------|
| **E4.F1** Non-interactive authentication | Ensure the tool works with service principal / managed identity credentials in pipeline environments | Impact 3 |
| **E4.F2** Exit codes and error output | Return meaningful exit codes and structured error output for pipeline integration | Impact 3 |

---

## 5. Roadmap

### MVP

The absolute minimum to cause Impacts 1 and 2: a developer or architect can run the tool locally and get valid PlantUML diagrams from their live APIM instance.

| Feature | Rationale |
|---------|-----------|
| **E1.F1** Authenticate to Azure | Without authentication, nothing works. |
| **E1.F2** Retrieve APIM configuration | The tool's core purpose is reading live APIM state. |
| **E1.F3** Export APIM configuration to local file | Avoids repeated Azure API calls when generating multiple diagrams. Enables offline diagram generation. |
| **E2.F1** Generate product overview diagram | The most valuable single diagram -- gives a full picture of the APIM instance. |
| **E2.F2** Generate product details diagram | Needed for focused analysis of a specific product. |
| **E2.F3** Generate API details diagram | Needed for focused analysis of a specific API. |
| **E2.F4** Generate backend details diagram | Needed for focused analysis of a specific backend. |
| **E2.F5** Include/exclude diagram elements | Without this, users cannot tailor diagrams to their needs. Required because the four diagram types share a parameterized include/exclude model. |
| **E2.F6** Library include path | Generated `.puml` files must reference the PlantUML library to render. URL-based include by default; override parameter needed for custom/local library paths. |
| **E3.F1** .NET global tool packaging | The agreed distribution mechanism. |
| **E3.F2** CLI commands and parameters | Users need a way to invoke the tool. |
| **E3.F3** Output to file | Diagrams must be saved somewhere useful. |

### 1.0

| Feature | Rationale |
|---------|-----------|
| **E4.F1** Non-interactive authentication | Enables CI/CD pipeline usage (Impact 3). |
| **E4.F2** Exit codes and error output | Makes pipeline integration reliable. |

### Beyond 1.0

No features are currently scoped beyond 1.0. Items that were considered and rejected:

| Considered | Decision | Reason |
|-----------|----------|--------|
| Render diagrams to PNG/SVG | **Cut** | PlantUML CLI or PlantUML server already does this. Not our job. |
| Diff diagrams between runs | **Cut** | Git diff on `.puml` text files is sufficient. |
| Interactive architecture review mode | **Cut** | Not a use case for this tool (confirmed during impact mapping). |
| Web UI | **Cut** | Contradicts the CLI-first design. No stated need. |

---

## 6. Traceability Matrix

Every MVP feature traces back through an impact to an actor and the goal.

```
Goal: Generate valid PlantUML diagrams from live APIM
  |
  +-- Actor: Developer / Architect
        |
        +-- Impact 1: START generating diagrams
        |     +-- E1.F1 Authenticate to Azure
        |     +-- E1.F2 Retrieve APIM configuration
        |     +-- E1.F3 Export APIM configuration to local file
        |     +-- E2.F1 Product overview diagram
        |     +-- E2.F2 Product details diagram
        |     +-- E2.F3 API details diagram
        |     +-- E2.F4 Backend details diagram
        |     +-- E2.F5 Include/exclude elements
        |     +-- E2.F6 Library include path
        |     +-- E3.F1 .NET global tool packaging
        |     +-- E3.F2 CLI commands and parameters
        |     +-- E3.F3 Output to file
        |
        +-- Impact 2: STOP relying on outdated documentation
        |     +-- E2.F1 Product overview diagram
        |     +-- E2.F2 Product details diagram
        |     +-- E2.F3 API details diagram
        |     +-- E2.F4 Backend details diagram
        |     +-- E3.F3 Output to file
        |
        +-- Impact 3: START using in CI/CD (post-MVP)
              +-- E4.F1 Non-interactive authentication
              +-- E4.F2 Exit codes and error output
```
