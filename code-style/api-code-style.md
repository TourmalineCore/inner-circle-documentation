# API Code Style

This document outlines the code style guidelines for API services.

## 1. Moving Requests Folder

"Requests" folder have been moved from the **Application** layer to the **Api** layer.

This change improves the separation of concerns by grouping all request-related code within the Api context.


## 2. Layers Srtucture

What and where we agreed to put?

### Background

Previously, we had 5 layers, but now we've decided to keep only 3.

We moved the **DataAccess** layer into **Application**, as it no longer makes sense to have a separate layer.

Additionally, we removed the **Tests** layer and distributed the tests throughout the code. Now, our unit-tests are located next to the classes they test. For example, in the **Application** layer, commands are placed alongside their corresponding tests, as shown in the image below:

![Tests Location Example](./images/tests-location-example.png)

### Conclusion

Now we are using 3 layers:

1. **Api**

- This layer contains controllers, requests, and responses.

2. **Application**

- Here lies the business logic, including commands, queries, and mapping with DbContext.

3. **Core**

- This layer contains classes that describe entities. These classes are used in DbContext to create tables. We do not return Core classes or perform any operations with them. 
   
   Our **Application** layer accepts DTOs or models, transforms these DTOs or models into Core classes in commands, and saves them in the database. Similarly, queries extract Core classes from the database but immediately convert them into models for passing up (for example, to responses).


## 3. Naming Convention of Methods That Return Tasks

All methods that return tasks must end with `async`.

```csharp
// example
public async Task DoSomethingAsync()
```


## 4. Controller Responses Naming

Everything that comes back from the controller must end with `response`.

```csharp
// example
public async Task<SomeResponse> DoSomething()
```


## 5. Explicit Response Mapping

In the controller, explicit mapping of data should always occur before returning the response.

```csharp
// example
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

---
to be continued..
