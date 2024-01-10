# How to create a .NET8 CRUD WebAPI Azure CosmosDB for PostgreSQL MicroService

## 0. Prerequisites

- Install Visual Studio 2022 Community Edition

- Install .NET SDK 8.0.1

- Install Entity Framework Core tools reference - .NET Core CLI: https://learn.microsoft.com/en-us/ef/core/cli/dotnet

**dotnet ef** can be installed as either a global or local tool. Most developers prefer installing dotnet ef as a global tool using the following command:

```
dotnet tool install --global dotnet-ef
```

Update the tool using the following command:

```
dotnet tool update --global dotnet-ef
```

Before you can use the tools on a specific project, you'll need to add this package to your application

```
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Run the following commands to verify that EF Core CLI tools are correctly installed:

```
dotnet ef
```

## 1. Create .NET8 CRUD WebAPI for PostgreSQL

**Step 1**: Create a New .NET Web API Project. Open a command line interface (CLI).

Run the following command to create a new Web API project:

```
dotnet new webapi -n BookApiProject
```

Navigate into your project directory:

```
cd BookApiProject
```

**Step 2**: Add Required Packages. Add the necessary NuGet packages:

```
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Swashbuckle.AspNetCore
```

**Step 3**: Input the Program.cs source code:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.OpenApi.Models;
using Swashbuckle.AspNetCore.Swagger;

var builder = WebApplication.CreateBuilder(args);

// Configure your DbContext
var connectionString = builder.Configuration.GetConnectionString("MyPostgresDb");
builder.Services.AddDbContext<BookDbContext>(options =>
    options.UseNpgsql(connectionString));

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();

// Add Swagger support
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Book API", Version = "v1" });
});

var app = builder.Build();

// Initialize the database
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    var dbContext = services.GetRequiredService<BookDbContext>();
    dbContext.Database.Migrate(); // This applies pending migrations or creates the database if it doesn't exist
}

// Configure Swagger middleware
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "Book API v1"));
}

// Map CRUD operations for Books
app.MapGet("/books", async (BookDbContext db) => await db.Books.ToListAsync());
app.MapGet("/books/{id}", async (int id, BookDbContext db) => await db.Books.FindAsync(id) is Book book ? Results.Ok(book) : Results.NotFound());
app.MapPost("/books", async (Book book, BookDbContext db) =>
{
    db.Books.Add(book);
    await db.SaveChangesAsync();

    return Results.Created($"/books/{book.BookId}", book);
});
app.MapPut("/books/{id}", async (int id, Book inputBook, BookDbContext db) =>
{
    var book = await db.Books.FindAsync(id);

    if (book == null) return Results.NotFound();

    book.BookName = inputBook.BookName;

    await db.SaveChangesAsync();
    return Results.NoContent();
});
app.MapDelete("/books/{id}", async (int id, BookDbContext db) =>
{
    var book = await db.Books.FindAsync(id);

    if (book == null) return Results.NotFound();

    db.Books.Remove(book);
    await db.SaveChangesAsync();
    return Results.NoContent();
});

app.Run();

// Define your DbContext and Book entity
public class BookDbContext : DbContext
{
    public BookDbContext(DbContextOptions<BookDbContext> options)
        : base(options) { }

    public DbSet<Book> Books { get; set; }
}

public class Book
{
    public int BookId { get; set; }
    public string BookName { get; set; }
}
```

**Step 4**: Modify **appsettings.json** and set your database connection string

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "MyPostgresDb": "Server=c-mypostgresql.znj364bc3mdfyx.postgres.cosmos.azure.com;Database=citus;Port=5432;User Id=citus;Password=Luiscoco123456;Ssl Mode=Require;"
  }
}
```

**Step 5**: Run Migrations (if using Code First). Run the following command to add a migration:

```
dotnet ef migrations add InitialCreate
```

Apply the migration to your database:

```
dotnet ef database update
```

**Step 6**: Run Your Application

```
dotnet run
```

Access the Swagger UI by going to http://localhost:5000/swagger in your web browser.

## 2. Create Azure CosmosDB for PostgreSQL with Azure Portal

Sign in to Azure Portal:

Go to Azure Portal and sign in with your Azure account.

Create a New Resource:

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/7b95e62f-731b-44ad-880b-633c0f7ef1e2)

Click on "**Create a resource**" in the top left corner of the dashboard.

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQL/assets/32194879/5a5e735e-fd1b-4626-82ae-9b88b080ca8e)

Search for Azure Cosmos DB:

In the "New" window, search for "**Azure Cosmos DB**" and select it from the results.

Create Azure Cosmos DB Account:

Click the "**Create**" button to start configuring your Azure Cosmos DB account.

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQL/assets/32194879/ce384ba7-c577-4639-8505-63fcde737d58)

Select the appropriate subscription and resource group (or create a new resource group).

Enter an account name.

Choose the API as "**Azure Cosmos DB for PostgreSQL**".

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQL/assets/32194879/3b1483fa-6af8-47b0-a12c-54da5bedea96)

Input the new PostgreSQL values

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/f6b1b233-83e7-460f-8ef2-433fa3d33261)

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/05f2844a-a610-4c2f-936b-7f28b7ee3976)

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/bd82dfd6-6970-45d6-a9ae-4161ab407a82)

Review and create the account. 

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/75b71712-7e40-4cf6-bf56-d9d831cdd40e)

Once your account is created, go to it in the Azure portal.

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/eaeefe3f-a9d1-4fc5-8bb1-9335f13ec30c)

Under "**Quick start (preview)**" create a new database and a container within that database.

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/226bbe26-cfff-48eb-a08d-39239ffd844b)

## 3. Copy the Connection String in appsettings.json file

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/9a6b97c2-2ef5-49a6-ba6e-1d49fe0fdc80)

**IMPORTANT NOTE**: Do not forget to input your PASSWORD (Password=Luiscoco123456) in the connection string, by default is not set.

**appsettings.json**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "MyPostgresDb": "Server=c-mypostgresql.x7wzviwo42ae6e.postgres.cosmos.azure.com;Database=citus;Port=5432;User Id=citus;Password=Luiscoco123456;Ssl Mode=Require;"
  }
}
```

## 4. Add the FireWall rules

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/280da660-6ed8-4c7b-b963-a24ecbf9abb4)

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/dd6ca270-1f61-4c6c-af14-18581a6237e2)


## 5. Create Azure CosmosDB for PostgreSQL with Azure CLI

Ensure you have the Azure CLI installed and you're logged in. If not, you can download it from the Azure CLI website and log in using az login.

Create a Resource Group (if you don't already have one):

```
az group create --name YourResourceGroupName --location eastus
```

Create an Azure Cosmos DB Account:

```
az cosmosdb create --name YourCosmosDBAccountName --resource-group YourResourceGroupName --capabilities EnablePostgreSQL
```

Create a Database and Container:

Currently, creating databases and containers specifically for Azure Cosmos DB for PostgreSQL through the Azure CLI might not be supported directly. 

You might need to use the Azure Portal or Cosmos DB SDK for this step.

After setting up your Azure Cosmos DB for PostgreSQL, you'll need to retrieve the connection string to use in your .NET application. 

You can find this in the Azure Portal under your Cosmos DB account's "Connection String" section.

## 6. Migrate the database

Run this command for creating the initial migration

```
dotnet ef migrations add InitialCreate
```

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/c01166cc-9308-40e3-baad-b989b017c8cc)

Or this command for updating the migartion

```
dotnet ef database update InitialCreate
```

## 7. Verify your application

http://localhost:5270/swagger/index.html

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB-for-PostgreSQLv1/assets/32194879/b5cadcae-0a11-4326-80e0-860778170f6a)





