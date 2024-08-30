# Section 02: Controllers, Routing, and Attributes

This markdown file provides a comprehensive overview of controllers, routing, attributes, and data annotations in ASP.NET Core Web API.

## Controllers in ASP.NET Core

### Create a Controller

- Controllers are typically located in `Controllers` folder.
- They must derive from `ControllerBase` or `Controller` class and have the `[ApiController]` attribute for Web Apis.
- Have a suffix of `Controller` in its name (e.g., HomeController).

```csharp
[ApiController]
[Route("api/[controller]")]
public class HomeController : ControllerBase
{
   // Actions go here
}
```

### Action Methods

- An action method in a controller is a public method that handles an HTTP request.

```csharp
[HttpGet]
public IActionResult GetAll()
{
    return Ok(new { message = "Hello World" });
}
```

#### Return Types in Action Methods

**1. _`IActionResult`_**

The most flexible return type, allowing you to return various HTTP status codes along with content (e.g., JSON, HTML).

```csharp
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    if (id <= 0)
    {
        return BadRequest("Invalid product ID.");
    }

    var product = _productService.GetProductById(id);

    if (product == null)
    {
        return NotFound();
    }

    return Ok(product);
}

```

**2. _`ActionResult<T>`_**

A generic version of `IActionResult` that allows returning a specific type along with HTTP status codes. Useful for APIs that need to return strongly-typed data.

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    if (id <= 0)
    {
        return BadRequest("Invalid product ID.");
    }

    var product = _productService.GetProductById(id);

    if (product == null)
    {
        return NotFound();
    }

    return product;
}
```

**3. _Returning a Custom Object_**

You can directly return a custom object (e.g., a model class) from an action method. ASP.NET Core will automatically serialize it to JSON.

```csharp
[HttpGet("{id}")]
public Product GetProduct(int id)
{
    return _productService.GetProductById(id);
}
```

#### Action Method Attributes

- `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`, `[HttpPatch]`: Specify the HTTP method.

- `[FromBody]`: Binds a parameter from the request body.
- `[FromQuery]`: Binds a parameter from the query string.
- `[FromRoute]`: Binds a parameter from the route data.
- `[FromForm]`: Binds a parameter from form data in the request.
- `[FromHeader]`: Binds a parameter from a specific HTTP header.

---

Examples

```csharp
public class TemperatureRequest
{
    public int Min { get; set; }
    public int Max { get; set; }
}

[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing",
        "Bracing",
        "Chilly",
        "Cool",
        "Mild",
        "Warm",
        "Balmy",
        "Hot",
        "Sweltering",
        "Scorching"
    };

    // /weatherForecast/generate
    [HttpPost]
    [Route("generate")]
    public IActionResult Generate([FromQuery] int count, [FromBody] TemperatureRequest request)
    {
        return Ok(
            Enumerable
                .Range(1, count)
                .Select(index => new WeatherForecast
                {
                    Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
                    TemperatureC = Random.Shared.Next(request.Min, request.Max),
                    Summary = Summaries[Random.Shared.Next(Summaries.Length)]
                })
                .ToArray()
        );
    }
}
```

Testing with Rest Client

```http
POST http://localhost:5177/weatherforecast/generate?count=2
Content-Type: application/json

{
    "min":"-23",
    "max":"25"
}
###
```

## [API Controller] Attribute

The [ApiController] attribute can be applied to a controller class to enable the following opinionated, API-specific behaviors:

- Attribute routing requirement
- Automatic HTTP 400 responses
- Binding source parameter inference
- Multipart/form-data request inference
- Problem details for error status codes

### Attribute routing requirement

Attribute routing will be required if you use [ApiController], e.g.:

```csharp
[ApiController]
[Route("[controller]")]
public class DataTablesController: ControllerBase
```

Actions are inaccessible via conventional routes defined by `UseEndpoints`, `UseMvc`, or `UseMvcWithDefaultRoute` in `Startup.Configure`.

### Automatic HTTP 400 responses

The `[ApiController]` attribute makes model validation errors automatically trigger an HTTP 400 response. Consequently, the following code is unnecessary in an action method:

```csharp
if (!ModelState.IsValid)
{
    return BadRequest(ModelState);
}
```

#### Default BadRequest response

```JSON
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "|7fb5e16a-4c8f23bbfc974667.",
  "errors": {
    "": [
      "A non-empty request body is required."
    ]
  }
}
```

To make automatic and custom responses consistent, call the `ValidationProblem` method instead of `BadRequest`. `ValidationProblem` returns a `ValidationProblemDetails` object as well as the automatic response.

### Binding source parameter inference

A binding source attribute defines the location at which an action parameter's value is found. The following binding source attributes exist:

| Attribute      | Binding source                                      |
| -------------- | --------------------------------------------------- |
| [FromBody]     | Request body                                        |
| [FromForm]     | Form data in the request body                       |
| [FromHeader]   | Request header                                      |
| [FromQuery]    | Request query string parameter                      |
| [FromRoute]    | Route data from the current request                 |
| [FromServices] | The request service injected as an action parameter |

The binding source inference rules behave as follows:

- `[FromServices]` is inferred for complex type parameters registered in the DI Container.

- `[FromBody]` is inferred for complex type parameters. An exception to the `[FromBody]` inference rule is any complex, built-in type with a special meaning, such as `IFormCollection` and `CancellationToken`. The binding source inference code ignores those special types.

- `[FromForm]` is inferred for action parameters of type `IFormFile` and `IFormFileCollection`. It's not inferred for any simple or user-defined types.

- `[FromRoute]` is inferred for any action parameter name matching a parameter in the route template. When more than one route matches an action parameter, any route value is considered `[FromRoute]`.

- `[FromQuery]` is inferred for any other action parameters.

#### FromBody inference notes

When an action has more than one parameter bound from the request body, an exception is thrown. For example, all of the following action method signatures cause an exception:

```csharp
// [FromBody] inferred on both because they're complex types.
[HttpPost]
public IActionResult Action1(Product product, Order order)


// [FromBody] attribute on one, inferred on the other because it's a complex type.
[HttpPost]
public IActionResult Action2(Product product, [FromBody] Order order)


// [FromBody] attribute on both.
[HttpPost]
public IActionResult Action3([FromBody] Product product, [FromBody] Order order)

```

For more information about `api-controller` attibute please visit the [Create web APIs with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-8.0).

### Define supported request content types with the [Consumes] attribute

The [Consumes] attribute allows an action to limit the supported request content types. Apply the [Consumes] attribute to an action or controller, specifying one or more content types:

```csharp
[HttpPost]
[Consumes("application/xml")]
public IActionResult CreateProduct(Product product)
```

Requests that don't specify a Content-Type header of application/xml result in a 415 Unsupported Media Type response.
