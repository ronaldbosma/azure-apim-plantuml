# Azure API Management PlantUML

[PlantUML](https://plantuml.com/) sprites, macros and stereotypes for creating PlantUML diagrams with [Azure API Management](https://azure.microsoft.com/products/api-management) components.

[Azure-PlantUML](https://github.com/plantuml-stdlib/Azure-PlantUML) includes symbols from all Azure services, but it doesn't include symbols for the resources inside Azure API Management. This project aims to fill that gap.

With Azure API Management PlantUML it is possible to create visually recognizable PlantUML diagrams for your API Management solutions.


## Getting Started

To use the API Management symbols in your PlantUML diagrams, you'll need include [ApiManagement.puml](./dist/v1/ApiManagement.puml) at the top of your `.puml` file.

You can either download the file locally and include it in your PlantUML diagram like this:

```
!include path/to/ApiManagement.puml
```

Or if you want to use the latest version, you can include it directly from this repository:

```
!include https://raw.githubusercontent.com/ronaldbosma/azure-apim-plantuml/refs/heads/main/dist/v1/ApiManagement.puml
```

