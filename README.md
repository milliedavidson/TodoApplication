# Creating a Web API in .NET

Following [this guide](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-9.0&tabs=visual-studio) to build a Web API in .NET.

There are controller-based APIs and minimal APIs.

## Outline of the API

- ```GET /api/todoitems``` - get all to do items
- ```GET /api/todoitems/{id}``` - get to do items by ID
- ```POST /api/todoitems``` - add a new to do item
- ```PUT /api/todoitems{id}``` - update an existing to do item by id
- ```DELETE /api/todoitems{id}``` - delete a to do item by id

client -> HTTP request -> MVC app controller -> database
model -> serialise JSON body -> HTTP response -> client

## Visual Studio

- Create a new project
- Choose ASP.NET Core Web API 
- And check the 'use controllers' box for a controller-based API (you'd uncheck for a minimal API)
- Install the Entity Framework Core package: ```Microsoft.EntityFrameworkCore.InMemory```
- Run the project to check it works. The API is hosted at https://localhost:portnumber (randomly chosen port number, set at the project creation)
- If the port is incorrect, you will get a HTTP 404 not found
- Append **/weatherforecast** to the URL to test the WeatherForecast API and see the JSON response

## Add a model

A model is a set of classes which represent data that the application manages. For example:

```csharp
public class TodoItem
{
    public long Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
```

The ```Id``` property is acting as the **unique key** (in a relational database).

## Add a database context

The database context is the main class used with Entity Framework for a data model. Add this ```TodoContext``` class under the models folder.

## Register the DB context

In ASP.NET Core, services like the DB context must be registered within the **dependency injection** container (the ```program.cs``` file). The container provides the service to controllers. Here, we are using an in-memory database (it runs locally using the RAM):

```csharp
builder.Services.AddDbContext<TodoContext>(opt =>
    opt.UseInMemoryDatabase("TodoList"));
```

## Scaffold a controller

Right-click folder > add > new scaffolded item > API Controller with actions, using Entity Framework

TodoItem > Model class
TodoContext > DBContext class

This adds a controller for ```TodoItems``` with CRUD actions and adds the ```Microsoft.VisualStudio.Web.CodeGeneration.Design``` and ```Microsoft.EntityFrameworkCore.Tools``` NuGet packages (which are required for scaffolding).

Scaffolding is a code generation framework for ASP.NET, used in web applications. It uses T4 templates to **generate basic controllers and views for models**. It also generates instances for the domain model and code for all the CRUD operations.

## PostTodoItem method

Note, the HTTP endpoint name comes from the routing configuration in ASP.NET Core, *not* the method name. For example:

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<TodoItem>> GetTodoItem(long id)
{
}
```

The endpoint is ```GET api/TodoItems/{id}```, *not* ```GET api/GetTodoItem/{id}```. This is because the ```[Route]``` attribute on the controller class defines the *base URL* for all the endpoints in the controller:

```csharp
[Route("api/[controller]")]
[ApiController]
public class TodoItemsController : ControllerBase
{
}
```

The ```[controller]``` placeholder is replaced by the controller class name, minus the 'Controller' suffix. So, in this case, the controller class name is ```TodoItemsController```, which is why the placeholder is replaced by ```TodoItems```. So we have ```api/TodoItems/```.

Any extra route information is added to the end of this base URL. Such as ```[HttpPut("{id}")]```. No we have ```PUT api/TodoItems/{id}```. The method gets the value of the TodoItem from the body of the HTTP request.

The ```CreatedAtAction``` POST method:

```csharp
[HttpPost]
public async Task<ActionResult<TodoItem>> PostTodoItem(TodoItem todoItem)
{
    _context.TodoItems.Add(todoItem);
    await _context.SaveChangesAsync()
    return CreatedAtAction(nameof(GetTodoItem), new { id = todoItem.Id }, todoItem);
}
```

- **Returns a 201 status code** as standard (if successful). This means the resource (the to do item) has been created.

- **Adds a location header** to the response. A location header indicates the URL to redirect a page to and only provides meaning when served with a 3xx re-direction response or 201 created status response. In our case, it specifies the URI of the created to do item.

- **References the ```GetTodoItem``` action** (i.e. the method) to create the location header's URI. The ```nameof``` keyword is used to avoid hard-coding the action name in the CreatedAtAction call. **An action is a method on a controller that handles HTTP requests**. Actions are responsible for executing the logic required to fulfill the request - such as interacting with the database and processing data.
