# API Code Style

This document outlines the code style guidelines for API services.


## 1. Layers Srtucture

Now we agreed to use 3 layers:

1. **Api**

- This layer contains controllers, requests, and responses. This helps to group all request-related code within the Api context.

2. **Application**

- Here lies the business logic, including commands, queries, and mapping with DbContext.

3. **Core**

- This layer contains classes that describe entities. These classes are used in DbContext to create tables. We do not return Core classes to user or perform any operations with them. 
   
   Our **Application** layer accepts models, transforms them into Core classes in commands, and saves them in the database. Similarly, queries extract Core classes from the database but immediately convert them into models for passing up (for example, to responses).


## 2. Test location

We place unit-tests next to the classes they test on all layers of the application. You can read [our article](https://www.tourmalinecore.com/articles/dotnet-unit-testing) about it. For example, in the **Application** layer, commands are placed alongside their corresponding tests, as shown in the image below:

![Tests Location Example](./images/tests-location-example.png)


## 3. Naming Convention of Methods That Return Tasks

All methods that return tasks must end with `Async`.

```csharp
public async Task DoSomethingAsync()
```


## 4. Controller Responses Naming

Everything that comes back from the controller must end with `Response`.

```csharp
public async Task<SomeElseResponse> DoSomethingElseAsync()
```


## 5. Explicit Response Mapping

In the controller, explicit mapping of data should always occur before returning the response.

```csharp
public async Task<CreateResponse> CreateAsync(CreateRequest createRequest)
{
    var createCommandParams = new CreateCommandParams
    {
        FirstParam = createRequest.FirstParam,
        SecondParam = createRequest.SecondParam,
    };

    var newId = await _createCommand.CreateAsync(createCommandParams);

    return new CreateResponse()
    {
        NewId = newId
    };
}
```


## 6. RORO pattern

We use RORO pattern (Request Object Response Object). You can read 
[our article](https://www.tourmalinecore.com/articles/React) about it.

---
to be continued..
