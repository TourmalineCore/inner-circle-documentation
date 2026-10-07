## API Documentation for Swagger / OpenAPI

We use built-in ASP.NET Core features to add metadata to OpenAPI (Swagger).
This allows us to describe the API directly in code, without XML comments or third-party libraries.

[Official documentation](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/include-metadata?view=aspnetcore-10.0&tabs=minimal-apis)

### Controller-level attributes

![Controller Metadata](./images/controller-metadata.png)

`[Tags("Tag 1")]`  
Sets the default OpenAPI tag for all endpoints in the controller.
Used for grouping endpoints in Swagger UI.

### Endpoint-level attributes

![Endpoint Metadata](./images/endpoint-metadata.png)

`[EndpointName("GetAllItems")]`  
Sets the operationId in the OpenAPI specification.

`[EndpointSummary("Get all items")]`  
Provides a short summary displayed in Swagger UI.

`[EndpointDescription("This is a description.")]`  
Provides a detailed description of the endpoint.

`[Tags("Tag 2")]`  
Overrides or adds OpenAPI tags for this specific endpoint.

### How this metadata appears in Swagger UI:

![Swagger UI](./images/swagger-ui-example.png)
![Endpoint Metadata in Swagger UI](./images/endpoint-metadata-ui.png)

### Parameter description

`[Description("ID of the item to delete")]`  
Adds a human-readable description to an endpoint parameter.

![Parameter description attribute](./images/parameter-description.png)

### How this metadata appears in Swagger UI:

![Endpoint Metadata in Swagger UI](./images/parameter-description-ui.png)

### DTO field metadata

![DTO field metadata](./images/DTO-field-metadata.png)

Data annotations on DTO properties are used both for runtime validation and for enriching the OpenAPI (Swagger) schema.

`[Description(...)]`  
Adds a descriptive text for the field.  

`[DefaultValue("Walther")]`, `[DefaultValue(2L)]`  
Defines an example/default value for the field in the OpenAPI schema.  
Useful for Swagger UI examples and documentation (does not enforce runtime defaults).

`[MaxLength(128)]`  
Adds a maximum length constraint.

`[RegularExpression(...)]`  
Documents and enforces a validation rule for the field.  
Appears as a regex pattern.

`[Range(1, 100)]`  
Restricts allowed numeric values and is exposed as `minimum` / `maximum`.

`[Required]`  
Ensures a property has a value at runtime and is marked required in Swagger. Unlike C# required, [Required] does not enforce compile-time initialization - it only works during runtime validation.

### How this metadata appears in Swagger UI:

![DTO field metadata in Swagger](./images/DTO-field-metadata-ui.png)
---
to be continued..