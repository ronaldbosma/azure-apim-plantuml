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

Next, you can use the available macros to add the API Management resources to your diagram. 
Here's an example of a PlantUML diagram that includes an API, two products and a backend:

```
@startuml Hello World

!include https://raw.githubusercontent.com/ronaldbosma/azure-apim-plantuml/refs/heads/main/dist/v1/ApiManagement.puml

$ApimProduct(starterProduct, "Starter")
$ApimProduct(unlimitedProduct, "Unlimited")
$ApimAPI(echoApi, "Echo API")
$ApimBackend(echoBackend, "Echo Backend")

starterProduct --> echoApi
unlimitedProduct --> echoApi
echoApi --> echoBackend

@enduml
```

The diagram above will render as follows:

![Hello World](./samples/hello-world.png)


## Components

### Available Symbols

The following symbols are available in the current version of Azure API Management PlantUML:

| Resource | Symbol | Macro | Example Usage |
|-|-|-|-|
| API | ![API](./dist/v1/sprites/API.svg) | `ApimAPI` | `$ApimAPI(alias, "label")` |
| Backend | ![Backend](./dist/v1/sprites/Backend.svg) | `ApimBackend` | `$ApimBackend(alias, "label")` |
| Product | ![Product](./dist/v1/sprites/Product.svg) | `ApimProduct` | `$ApimProduct(alias, "label")` |
| Subscription | ![Subscription](./dist/v1/sprites/Subscription.svg) | `ApimSubscription` | `$ApimSubscription(alias, "label")` |
| User | ![User](./dist/v1/sprites/User.svg) | `ApimUser` | `$ApimUser(alias, "label")` |

### API Operations

It is possible to add operations to an API by using the `ApimAPIOperation` macro. This macro will render the operations as a table inside a note attached to the API.

The macro takes the following parameters:

| Parameter | Description |
|-|-|
| api | The alias of the API. |
| operations | A JSON array of operations. Each operation should specify the `Method`, `Description` and `UrlTemplate`. |
| alignment | The position of the note in relation to the API (default is `bottom`). |

Here's an example of how to add operations to an API:

```
@startuml Operations

!include https://raw.githubusercontent.com/ronaldbosma/azure-apim-plantuml/refs/heads/main/dist/v1/ApiManagement.puml

left to right direction

$ApimAPI(echoApi, "Echo API")

!$operations = [
    { "Method": "GET",    "Description": "Retrieve a resource",         "UrlTemplate": "/resources" },
    { "Method": "POST",   "Description": "Create a resource",           "UrlTemplate": "/resources" },
    { "Method": "PUT",    "Description": "Update a resource",           "UrlTemplate": "/resources/{id}" },
    { "Method": "PATCH",  "Description": "Partially Update a resource", "UrlTemplate": "/resources/{id}" },
    { "Method": "DELETE", "Description": "Delete a resource",           "UrlTemplate": "/resources/{id}" }
]
$ApimOperations(echoApi, $operations, "right")

@enduml
```

The diagram above will render as follows:

![Operations](./samples/operations.png)
