# Azure API Management PlantUML

[PlantUML](https://plantuml.com/) sprites, macros and stereotypes for creating PlantUML diagrams with [Azure API Management](https://azure.microsoft.com/products/api-management) components.

[Azure-PlantUML](https://github.com/plantuml-stdlib/Azure-PlantUML) includes symbols from all Azure services, but it doesn't include symbols for the resources inside Azure API Management. This project aims to fill that gap.

With Azure API Management PlantUML it is possible to create visually recognizable PlantUML diagrams for your API Management solutions.


## Getting Started

To use the API Management symbols in your PlantUML diagrams, you'll need include [ApiManagement.puml](./dist/v1/ApiManagement.puml) at the top of your `.puml` file.

If you want to use the latest version, you can include it in your PlantUML diagram directly from this repository:

```
!include https://raw.githubusercontent.com/ronaldbosma/azure-apim-plantuml/refs/heads/main/dist/v1/ApiManagement.puml
```

Or, you can download the file locally and include it like this:

```
!include path/to/ApiManagement.puml
```


## Available Symbols

The following symbols are available in the current version of Azure API Management PlantUML:

| Resource | Symbol | Macro | Example Usage |
|-|-|-|-|
| API | ![API](./dist/v1/sprites/API.svg) | `ApimAPI` | `$ApimAPI(alias, "label")` |
| Backend | ![Backend](./dist/v1/sprites/Backend.svg) | `ApimBackend` | `$ApimBackend(alias, "label")` |
| Product | ![Product](./dist/v1/sprites/Product.svg) | `ApimProduct` | `$ApimProduct(alias, "label")` |
| Subscription | ![Subscription](./dist/v1/sprites/Subscription.svg) | `ApimSubscription` | `$ApimSubscription(alias, "label")` |
| User | ![User](./dist/v1/sprites/User.svg) | `ApimUser` | `$ApimUser(alias, "label")` |