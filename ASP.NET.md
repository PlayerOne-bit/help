# ASP.NET Core Backend Tutorial (Beginner-Friendly)

A practical introduction to building a backend using **C# and ASP.NET
Core Web API**.

This tutorial builds a small **Task Manager API**. You will learn how a
backend receives requests, validates data, works with a database, and
returns responses to a frontend such as React, Flutter, or JavaScript.

> **Target:** Beginner who knows basic C#\
> **Tools:** .NET SDK, Visual Studio Code or Visual Studio, and SQLite\
> **Project type:** ASP.NET Core Web API\
> **Note:** Commands and APIs may vary slightly between .NET versions.
> Use the current supported .NET SDK installed on your computer.

------------------------------------------------------------------------

# 1. What Is a Backend?

A backend is the part of an application that runs on a server. It
handles the application's data and rules.

For example, in a task manager:

-   The frontend displays the task list.
-   The backend receives requests such as "show my tasks" or "create a
    task."
-   The database stores the tasks.
-   The backend sends the result back to the frontend.

## How the parts communicate

``` text
User
 |
 v
Frontend (React / Flutter / Mobile App)
 |
 | HTTP request (GET, POST, PUT, DELETE)
 v
ASP.NET Core Web API
 |
 | C# business logic
 v
Database (SQLite / SQL Server / PostgreSQL)
 |
 v
ASP.NET Core Web API
 |
 | JSON response
 v
Frontend displays the result
```

## Important terms

  Term       Meaning
  ---------- -------------------------------------------------
  API        A way for applications to communicate
  Backend    Server-side code and application logic
  Endpoint   A URL that performs a specific operation
  HTTP       The protocol used for web requests
  JSON       A common format for sending data
  Database   Where application data is stored
  CRUD       Create, Read, Update, and Delete
  ORM        A tool that maps C# objects to database records

------------------------------------------------------------------------

# 2. Install the Required Tools

Install these tools:

1.  **.NET SDK** --- provides the compiler, runtime, and `dotnet`
    command.
2.  **Visual Studio** or **Visual Studio Code** --- for writing C# code.
3.  **SQLite** --- a lightweight database. We will use it through Entity
    Framework Core.

Check whether .NET is installed:

``` bash
dotnet --version
```

See the installed SDKs:

``` bash
dotnet --list-sdks
```

If the command is not recognized, install the .NET SDK and reopen your
terminal.

------------------------------------------------------------------------

# 3. Create an ASP.NET Core Web API

Open a terminal in the folder where you keep your projects.

Create the project:

``` bash
dotnet new webapi -n TaskManagerApi
cd TaskManagerApi
```

Open the folder in your editor:

``` bash
code .
```

Run the application:

``` bash
dotnet run
```

The terminal displays the local address where the API is running. It
commonly looks like:

``` text
http://localhost:5000
https://localhost:7000
```

Your actual ports may be different.

Open the HTTPS or HTTP address in a browser. If the project includes
OpenAPI/Swagger, open the Swagger URL shown by the project or in its
launch settings.

## What does `dotnet new webapi` do?

It creates a starter ASP.NET Core API project with configuration and
sample code.

Common files:

  File or folder                     Purpose
  ---------------------------------- ---------------------------------------------------
  `Program.cs`                       Configures and starts the application
  `appsettings.json`                 Application settings
  `Properties/launchSettings.json`   Local development launch settings
  `Controllers/`                     API controllers, if using the controller template
  `Models/`                          Data classes you create
  `Data/`                            Database configuration and context you create

The exact starter files depend on the template and SDK version.

------------------------------------------------------------------------

# 4. Understand `Program.cs`

`Program.cs` is the application's starting point. It registers services
and configures the HTTP request pipeline.

A minimal API project may look similar to this:

``` csharp
var builder = WebApplication.CreateBuilder(args);

// Register services here.

var app = builder.Build();

// Configure middleware here.

app.MapGet("/", () => "Task Manager API is running!");

app.Run();
```

Important parts:

-   `WebApplication.CreateBuilder(args)` creates the application
    builder.
-   `builder.Services` is where you register services your app needs.
-   `builder.Build()` creates the application.
-   `app.MapGet(...)` creates a GET endpoint.
-   `app.Run()` starts the web server.

## What is a service?

A service is an object that provides functionality to other parts of
your application.

Examples:

-   Database context
-   Email sender
-   Authentication service
-   Business logic service

ASP.NET Core uses **dependency injection (DI)** to create and provide
registered services.

------------------------------------------------------------------------

# 5. Understand HTTP Methods

An API commonly uses these HTTP methods:

  Method   Purpose                  Example
  -------- ------------------------ ----------------------
  GET      Retrieve data            Get all tasks
  POST     Create data              Add a task
  PUT      Replace or update data   Update a task
  PATCH    Partially update data    Mark a task complete
  DELETE   Remove data              Delete a task

Example endpoint URLs:

``` text
GET     /api/tasks
GET     /api/tasks/5
POST    /api/tasks
PUT     /api/tasks/5
DELETE  /api/tasks/5
```

The number `5` is a route parameter. It identifies a specific task.

------------------------------------------------------------------------

# 6. Build Your First API Endpoint

For a simple first test, add this endpoint to `Program.cs` before
`app.Run()`:

``` csharp
app.MapGet("/api/hello", () =>
{
    return Results.Ok(new
    {
        message = "Hello from ASP.NET Core!"
    });
});
```

Run the application and visit:

``` text
http://localhost:5000/api/hello
```

Use the actual port printed in your terminal.

You should receive JSON similar to:

``` json
{
  "message": "Hello from ASP.NET Core!"
}
```

`Results.Ok(...)` sends an HTTP `200 OK` response with the supplied
data.

------------------------------------------------------------------------

# 7. Create a Task Model

A **model** describes the data your application works with.

Create a folder named `Models`.

Inside it, create `TaskItem.cs`:

``` csharp
namespace TaskManagerApi.Models;

public class TaskItem
{
    public int Id { get; set; }

    public string Title { get; set; } = string.Empty;

    public string? Description { get; set; }

    public bool IsCompleted { get; set; }
}
```

Explanation:

-   `Id` uniquely identifies a task.
-   `Title` stores the task name.
-   `Description` is optional because it uses `string?`.
-   `IsCompleted` stores whether the task is finished.

The model is a C# representation of a task. Later, Entity Framework Core
will map it to a database table.

------------------------------------------------------------------------

# 8. Learn CRUD with an In-Memory List First

Before connecting a database, understand CRUD using a list.

Add this to `Program.cs` before `app.Run()`:

``` csharp
using TaskManagerApi.Models;

var tasks = new List<TaskItem>();
var nextId = 1;

app.MapGet("/api/tasks", () =>
{
    return Results.Ok(tasks);
});

app.MapGet("/api/tasks/{id:int}", (int id) =>
{
    var task = tasks.FirstOrDefault(t => t.Id == id);

    return task is null
        ? Results.NotFound()
        : Results.Ok(task);
});

app.MapPost("/api/tasks", (TaskItem newTask) =>
{
    newTask.Id = nextId++;
    tasks.Add(newTask);

    return Results.Created($"/api/tasks/{newTask.Id}", newTask);
});

app.MapPut("/api/tasks/{id:int}", (int id, TaskItem updatedTask) =>
{
    var task = tasks.FirstOrDefault(t => t.Id == id);

    if (task is null)
        return Results.NotFound();

    task.Title = updatedTask.Title;
    task.Description = updatedTask.Description;
    task.IsCompleted = updatedTask.IsCompleted;

    return Results.Ok(task);
});

app.MapDelete("/api/tasks/{id:int}", (int id) =>
{
    var task = tasks.FirstOrDefault(t => t.Id == id);

    if (task is null)
        return Results.NotFound();

    tasks.Remove(task);

    return Results.NoContent();
});
```

> If you use this example, remove or rename any existing endpoint with
> the same route. Keep the required `using` statements at the top of the
> file.

## Test the endpoints

Use Swagger (if available), Postman, or another API client.

### Create a task

Send:

``` http
POST /api/tasks
Content-Type: application/json
```

Body:

``` json
{
  "title": "Learn ASP.NET Core",
  "description": "Build my first backend API",
  "isCompleted": false
}
```

### Get all tasks

``` http
GET /api/tasks
```

### Get one task

``` http
GET /api/tasks/1
```

### Update a task

``` http
PUT /api/tasks/1
Content-Type: application/json
```

Body:

``` json
{
  "title": "Learn ASP.NET Core Web API",
  "description": "Practice CRUD operations",
  "isCompleted": true
}
```

### Delete a task

``` http
DELETE /api/tasks/1
```

## Important limitation

The list exists only in application memory. When you stop and restart
the API, the tasks disappear. It is useful for learning routes and HTTP
methods, but real applications need persistent storage.

------------------------------------------------------------------------

# 9. Connect a Database with Entity Framework Core

**Entity Framework Core (EF Core)** is an Object-Relational Mapper
(ORM). It lets you work with database records using C# objects and LINQ.

Install the SQLite provider and EF Core design package:

``` bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Install the EF Core command-line tool if you do not already have it:

``` bash
dotnet tool install --global dotnet-ef
```

If it is already installed, update it when necessary:

``` bash
dotnet tool update --global dotnet-ef
```

## Create the database context

Create a folder named `Data`.

Inside it, create `AppDbContext.cs`:

``` csharp
using Microsoft.EntityFrameworkCore;
using TaskManagerApi.Models;

namespace TaskManagerApi.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<TaskItem> Tasks => Set<TaskItem>();
}
```

A `DbContext` represents a session with the database.

`DbSet<TaskItem>` represents the collection of task records you can
query and modify.

## Register the database in `Program.cs`

Add these namespaces at the top:

``` csharp
using Microsoft.EntityFrameworkCore;
using TaskManagerApi.Data;
```

Before `builder.Build()`, register the context:

``` csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite(
        builder.Configuration.GetConnectionString("DefaultConnection")));
```

## Configure the connection string

In `appsettings.json`, add or update the `ConnectionStrings` section:

``` json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=taskmanager.db"
  }
}
```

Keep any other existing settings in the file. JSON does not allow
comments or trailing commas.

## Create the database using migrations

A migration records changes to your database structure.

Run:

``` bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

EF Core creates the SQLite database file and the required table.

If the command cannot find your `DbContext`, check that:

-   The context class is public.
-   The context is registered in `Program.cs`.
-   The project builds successfully.
-   You are running the command in the project folder.

------------------------------------------------------------------------

# 10. Use the Database in Your Endpoints

Replace the in-memory list endpoints with database-backed endpoints.

For example, in `Program.cs`, after building the app:

``` csharp
app.MapGet("/api/tasks", async (AppDbContext db) =>
{
    var tasks = await db.Tasks.ToListAsync();

    return Results.Ok(tasks);
});

app.MapGet("/api/tasks/{id:int}", async (int id, AppDbContext db) =>
{
    var task = await db.Tasks.FindAsync(id);

    return task is null
        ? Results.NotFound()
        : Results.Ok(task);
});

app.MapPost("/api/tasks", async (TaskItem newTask, AppDbContext db) =>
{
    db.Tasks.Add(newTask);
    await db.SaveChangesAsync();

    return Results.Created($"/api/tasks/{newTask.Id}", newTask);
});

app.MapPut("/api/tasks/{id:int}", async (
    int id,
    TaskItem updatedTask,
    AppDbContext db) =>
{
    var task = await db.Tasks.FindAsync(id);

    if (task is null)
        return Results.NotFound();

    task.Title = updatedTask.Title;
    task.Description = updatedTask.Description;
    task.IsCompleted = updatedTask.IsCompleted;

    await db.SaveChangesAsync();

    return Results.Ok(task);
});

app.MapDelete("/api/tasks/{id:int}", async (int id, AppDbContext db) =>
{
    var task = await db.Tasks.FindAsync(id);

    if (task is null)
        return Results.NotFound();

    db.Tasks.Remove(task);
    await db.SaveChangesAsync();

    return Results.NoContent();
});
```

The database context is injected into each endpoint through dependency
injection.

`SaveChangesAsync()` commits the changes to the database.

> Do not keep both the in-memory and database versions of the same
> routes. Use one implementation.

------------------------------------------------------------------------

# 11. Understand the `async` and `await` Keywords

Database operations can take time. ASP.NET Core supports asynchronous
operations so a request does not need to block a thread while waiting
for I/O.

Example:

``` csharp
var tasks = await db.Tasks.ToListAsync();
```

-   `async` allows a method or endpoint handler to use `await`.
-   `await` waits for an asynchronous operation to finish without
    blocking the thread in the usual synchronous way.
-   `ToListAsync()` asynchronously retrieves the records.

Use asynchronous EF Core methods for database operations in web APIs.

------------------------------------------------------------------------

# 12. Use DTOs Instead of Exposing Database Models

A **DTO (Data Transfer Object)** defines the data your API accepts or
returns.

Why use DTOs?

-   Prevent clients from setting fields they should not control.
-   Avoid exposing internal database properties.
-   Control the shape of API responses.
-   Keep request and response contracts separate from database design.

Create a folder named `Dtos`.

Create `CreateTaskRequest.cs`:

``` csharp
namespace TaskManagerApi.Dtos;

public record CreateTaskRequest(
    string Title,
    string? Description
);
```

Create `UpdateTaskRequest.cs`:

``` csharp
namespace TaskManagerApi.Dtos;

public record UpdateTaskRequest(
    string Title,
    string? Description,
    bool IsCompleted
);
```

A POST endpoint can accept the DTO:

``` csharp
app.MapPost("/api/tasks", async (
    CreateTaskRequest request,
    AppDbContext db) =>
{
    var task = new TaskItem
    {
        Title = request.Title,
        Description = request.Description,
        IsCompleted = false
    };

    db.Tasks.Add(task);
    await db.SaveChangesAsync();

    return Results.Created($"/api/tasks/{task.Id}", task);
});
```

The client sends only the fields it is allowed to provide. The server
decides the ID and initial completion state.

------------------------------------------------------------------------

# 13. Add Basic Validation

Never assume that data sent by a frontend is valid. A user can send
requests directly to your API.

For a simple endpoint, validate the input:

``` csharp
app.MapPost("/api/tasks", async (
    CreateTaskRequest request,
    AppDbContext db) =>
{
    if (string.IsNullOrWhiteSpace(request.Title))
    {
        return Results.ValidationProblem(
            new Dictionary<string, string[]>
            {
                ["title"] = ["Title is required."]
            });
    }

    if (request.Title.Length > 100)
    {
        return Results.ValidationProblem(
            new Dictionary<string, string[]>
            {
                ["title"] = ["Title cannot exceed 100 characters."]
            });
    }

    var task = new TaskItem
    {
        Title = request.Title.Trim(),
        Description = request.Description?.Trim()
    };

    db.Tasks.Add(task);
    await db.SaveChangesAsync();

    return Results.Created($"/api/tasks/{task.Id}", task);
});
```

For larger projects, use validation attributes or a dedicated validation
library and keep validation rules organized.

------------------------------------------------------------------------

# 14. Understand HTTP Status Codes

Your API should return meaningful status codes.

  -----------------------------------------------------------------------
  Status                  Meaning                 When to use
  ----------------------- ----------------------- -----------------------
  200 OK                  Request succeeded       Successful GET or
                                                  update

  201 Created             Resource created        Successful POST

  204 No Content          Succeeded with no       Successful DELETE
                          response body           

  400 Bad Request         Invalid request         Invalid input

  401 Unauthorized        Authentication is       User is not
                          missing or invalid      authenticated

  403 Forbidden           User is authenticated   Insufficient permission
                          but not allowed         

  404 Not Found           Resource does not exist Invalid task ID

  409 Conflict            Request conflicts with  Duplicate or
                          current state           conflicting data

  500 Internal Server     Unexpected server error Unhandled server
  Error                                           problem
  -----------------------------------------------------------------------

Do not return `200 OK` for every situation. The status code helps the
frontend understand what happened.

------------------------------------------------------------------------

# 15. Organize a Larger ASP.NET Core Project

For a small learning API, Minimal APIs are convenient. For a larger
application, you may prefer Controllers and separate services.

A common structure is:

``` text
TaskManagerApi/
|
|-- Controllers/
|   |-- TasksController.cs
|
|-- Data/
|   |-- AppDbContext.cs
|
|-- Dtos/
|   |-- CreateTaskRequest.cs
|   |-- UpdateTaskRequest.cs
|
|-- Models/
|   |-- TaskItem.cs
|
|-- Services/
|   |-- TaskService.cs
|
|-- Repositories/
|   |-- TaskRepository.cs
|
|-- Properties/
|-- appsettings.json
|-- Program.cs
```

### What does each layer do?

  -----------------------------------------------------------------------
  Layer                               Responsibility
  ----------------------------------- -----------------------------------
  Controller                          Receives HTTP requests and returns
                                      HTTP responses

  Service                             Contains business rules and
                                      application operations

  Repository                          Handles data access, if your
                                      architecture uses this pattern

  DbContext                           Communicates with the database
                                      through EF Core

  Model / Entity                      Represents stored application data

  DTO                                 Defines data sent to or received
                                      from clients
  -----------------------------------------------------------------------

Not every project needs a repository layer. EF Core's `DbContext`
already provides data-access and unit-of-work behavior. Avoid adding
layers without a clear reason.

------------------------------------------------------------------------

# 16. What Is a Controller?

A controller groups related API endpoints.

Create `Controllers/TasksController.cs`:

``` csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TaskManagerApi.Data;
using TaskManagerApi.Dtos;
using TaskManagerApi.Models;

namespace TaskManagerApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class TasksController : ControllerBase
{
    private readonly AppDbContext _db;

    public TasksController(AppDbContext db)
    {
        _db = db;
    }

    [HttpGet]
    public async Task<ActionResult<List<TaskItem>>> GetAll()
    {
        var tasks = await _db.Tasks.ToListAsync();

        return Ok(tasks);
    }

    [HttpGet("{id:int}")]
    public async Task<ActionResult<TaskItem>> GetById(int id)
    {
        var task = await _db.Tasks.FindAsync(id);

        if (task is null)
            return NotFound();

        return Ok(task);
    }

    [HttpPost]
    public async Task<ActionResult<TaskItem>> Create(
        CreateTaskRequest request)
    {
        if (string.IsNullOrWhiteSpace(request.Title))
            return BadRequest("Title is required.");

        var task = new TaskItem
        {
            Title = request.Title.Trim(),
            Description = request.Description?.Trim()
        };

        _db.Tasks.Add(task);
        await _db.SaveChangesAsync();

        return CreatedAtAction(
            nameof(GetById),
            new { id = task.Id },
            task);
    }
}
```

Register controllers in `Program.cs`:

``` csharp
builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();

app.Run();
```

Also register the database context as shown earlier. Do not add a second
`builder.Build()` or `app.Run()` if they already exist.

The controller route `[Route("api/[controller]")]` uses the controller
name without the `Controller` suffix. `TasksController` therefore uses
`/api/tasks`.

------------------------------------------------------------------------

# 17. Connect a Frontend to Your API

A frontend sends HTTP requests to your backend.

Example using JavaScript `fetch`:

``` javascript
async function getTasks() {
    const response = await fetch("https://localhost:7000/api/tasks");

    if (!response.ok) {
        throw new Error("Failed to load tasks.");
    }

    return await response.json();
}
```

Example for creating a task:

``` javascript
async function createTask() {
    const response = await fetch("https://localhost:7000/api/tasks", {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            title: "Study ASP.NET Core",
            description: "Practice API development"
        })
    });

    if (!response.ok) {
        throw new Error("Failed to create task.");
    }

    return await response.json();
}
```

Replace the sample URL with the actual address and port where your API
is running.

## What is CORS?

**CORS (Cross-Origin Resource Sharing)** is a browser security
mechanism. It controls whether a frontend hosted on one origin can call
an API hosted on another origin.

For example:

``` text
Frontend: http://localhost:5173
Backend:  https://localhost:7000
```

These are different origins.

Configure a development CORS policy in `Program.cs`:

``` csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("FrontendPolicy", policy =>
    {
        policy
            .WithOrigins("http://localhost:5173")
            .AllowAnyHeader()
            .AllowAnyMethod();
    });
});
```

After building the app, but before mapping endpoints, add:

``` csharp
app.UseCors("FrontendPolicy");
```

Use the exact frontend origin, including its scheme and port. Do not use
unrestricted CORS in production unless there is a specific, justified
requirement.

------------------------------------------------------------------------

# 18. Authentication: What Comes Next?

Authentication answers: **Who are you?**

Authorization answers: **What are you allowed to do?**

A common API authentication approach is **JWT bearer authentication**.

Typical flow:

``` text
1. User enters username and password.
2. Frontend sends credentials to POST /api/auth/login.
3. Backend verifies the password against a stored password hash.
4. Backend creates a signed access token.
5. Frontend stores the token using an appropriate secure strategy.
6. Frontend sends the token with protected API requests.
7. Backend validates the token and identifies the user.
```

A protected request commonly includes:

``` http
Authorization: Bearer YOUR_ACCESS_TOKEN
```

Important security rules:

-   Never store plain-text passwords.
-   Hash passwords with a suitable password-hashing algorithm or
    established identity framework.
-   Do not hardcode production signing keys or database passwords.
-   Keep secrets in development secrets or a secure production secret
    store.
-   Use HTTPS.
-   Validate authorization on the backend. Hiding a button in the
    frontend is not security.
-   Give each user access only to the records they are allowed to
    access.

JWT authentication is a separate topic and should be implemented
carefully. Do not copy a random token-generation example into a
production system.

------------------------------------------------------------------------

# 19. Useful `dotnet` Commands

  Command                            Purpose
  ---------------------------------- ------------------------------------
  `dotnet new webapi -n MyApi`       Create a Web API project
  `dotnet new console -n MyApp`      Create a console project
  `dotnet run`                       Build and run the project
  `dotnet build`                     Compile the project
  `dotnet watch run`                 Run with development file watching
  `dotnet add package PackageName`   Install a NuGet package
  `dotnet list package`              List installed packages
  `dotnet restore`                   Restore dependencies
  `dotnet clean`                     Clean build outputs
  `dotnet ef migrations add Name`    Create a database migration
  `dotnet ef database update`        Apply migrations to the database

------------------------------------------------------------------------

# 20. Common Beginner Problems

## The API does not start

Try:

``` bash
dotnet build
dotnet run
```

Read the first meaningful error in the terminal. Check whether another
process is already using the configured port.

## The frontend cannot call the API

Check:

1.  Is the backend running?
2.  Is the frontend using the correct API URL and port?
3.  Is the request using the correct HTTP method?
4.  Is the CORS origin configured correctly?
5.  Is the browser reporting a CORS error, a network error, or an HTTP
    error?

These are different problems and require different fixes.

## The database is not updating

Try:

``` bash
dotnet ef migrations list
dotnet ef database update
```

Make sure you are using the intended connection string and database
file.

## The API returns 404

Check the route, HTTP method, controller registration, and endpoint
mapping.

For example, `/api/tasks` is not the same route as `/tasks`.

## The API returns 400

Inspect the response body. It may contain validation errors or indicate
that the JSON request does not match the expected DTO.

------------------------------------------------------------------------

# 21. Beginner Practice Project: Task Manager API

Build this project without copying every line immediately. Try each
requirement yourself.

## Required features

-   [ ] Create a task
-   [ ] Display all tasks
-   [ ] Display one task by ID
-   [ ] Update a task
-   [ ] Delete a task
-   [ ] Store tasks in SQLite
-   [ ] Validate that a task title is not empty
-   [ ] Return appropriate HTTP status codes
-   [ ] Test all endpoints using Swagger or Postman

## Optional improvements

-   [ ] Add a due date
-   [ ] Add task priority: Low, Medium, High
-   [ ] Search tasks by title
-   [ ] Filter completed and incomplete tasks
-   [ ] Add pagination
-   [ ] Add user registration and login
-   [ ] Ensure users can access only their own tasks
-   [ ] Connect a React or Flutter frontend

## Suggested endpoint design

  Method   Endpoint            Purpose
  -------- ------------------- ---------------
  GET      `/api/tasks`        Get all tasks
  GET      `/api/tasks/{id}`   Get one task
  POST     `/api/tasks`        Create a task
  PUT      `/api/tasks/{id}`   Update a task
  DELETE   `/api/tasks/{id}`   Delete a task

------------------------------------------------------------------------

# 22. Recommended Learning Order

Learn these topics in order:

1.  C# classes, objects, properties, methods, and interfaces
2.  HTTP methods, status codes, JSON, and REST API concepts
3.  ASP.NET Core routing and endpoints
4.  Models and DTOs
5.  Dependency injection
6.  Entity Framework Core and SQL
7.  CRUD operations
8.  Validation and error handling
9.  Controllers and service organization
10. CORS and frontend integration
11. Authentication with JWT or ASP.NET Core Identity
12. Authorization, security, and deployment

Do not rush into authentication or deployment before you understand
basic CRUD and database operations.

------------------------------------------------------------------------

# 23. Quick Knowledge Check

Try answering these questions without looking back.

1.  What is the difference between a frontend and a backend?
2.  What is an API endpoint?
3.  When should you use GET instead of POST?
4.  What does CRUD stand for?
5.  What is the purpose of a model?
6.  What is a DTO, and why is it useful?
7.  What does `DbContext` do?
8.  Why do we call `SaveChangesAsync()`?
9.  What is the difference between 401 and 403?
10. What problem does CORS solve?
11. What is the difference between authentication and authorization?
12. Why should a backend validate data even if the frontend already
    validates it?

## Final reminder

A backend is more than a collection of endpoints. A good backend also
has clear data models, validation, secure access rules, reliable
database operations, meaningful error responses, and maintainable code
organization.

**Your first goal:** Build and test the Task Manager CRUD API with
SQLite. Once it works, connect it to a frontend.
