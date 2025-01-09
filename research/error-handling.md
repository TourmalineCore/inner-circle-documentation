# Error handling
## General rules

1. Use custom exception types:
```cs
public class CustomException : Exception
{
   public CustomException(string message) : base(message) { }
}

```
2. Include all the important information in the exception message: the error code, a clear and detailed description of the problem, and a possible solution (optional)
3. Response body should be:
```json
{
  "title": "List of books is empty",
  "status": 500,
  "detail": "at Api.Controllers.BooksController.GetAllBooksAsync() in ..."
}

```
4. Use [error handling middleware](https://akashjwork.medium.com/best-practices-for-creating-an-effective-error-handler-in-asp-net-core-7-fc8d7c83f585)
5. IExceptionHandler: 
- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-9.0#iexceptionhandler
- https://www.milanjovanovic.tech/blog/global-error-handling-in-aspnetcore-8

## How to set up IExceptionHandler?
1. Add ExceptionHandler to builder:
Program.cs:
```cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
app.UseExceptionHandler();
```
2. Create your ExceptionHandler:
GlobalExceptionHandler.cs
```cs
internal sealed class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        // Log error to console
        _logger.LogError(
            exception, "Exception occurred: {Message}", exception.Message);
        // Add ProblemDetails with statuscode, title and details of exception
        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "Server error",
            Detail = exception.Message
        };

        httpContext.Response.StatusCode = problemDetails.Status.Value;
        // Return exception as json
        await httpContext.Response
            .WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }
}

```
## Interesting links
1. [Result types of error in C#](https://dev.to/ephilips/better-error-handling-in-c-with-result-types-4aan) - it's interesting, but it doesn't intercept the exceptions, as I understand it;
2. [General rules](https://medium.com/@jepozdemir/effective-error-handling-in-c-best-practices-ed1052717714) about errors handling.
