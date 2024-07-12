# 🚀 Build & Run .NET Web API with PostgreSQL in Docker 🐘 – Ultimate Guide for Modern Development 🌟

# Introduction
👋 Hey, Netcode-Hub family! 🚀 In our journey of mastering Web APIs, we've already explored connecting to SQLite and SQL Server databases within Docker. Today, we're diving into the world of PostgreSQL! 🐘✨

# Scenario:
Imagine you're working on a project that requires a robust, scalable database solution. PostgreSQL is known for its advanced features, strong performance, and flexibility. Now, imagine taking that power and containerizing it with your .NET Web API in Docker. This setup not only simplifies deployment but also ensures consistency across different environments—perfect for development, testing, and production. Sounds amazing, right? I'll show you exactly how to build, configure, and run your Web API with PostgreSQL in Docker. Let's get started and take your skills to the next level! 🎉

# Create Model
    public class Product
    {
        public int Id { get; set; }
        public string? Name { get; set; }
        public string? Description { get; set; }
        public int Quantity { get; set; }
    }

  # Create Connection String
    "ConnectionStrings": {
    "SQLiteConnection": "Data Source=YoutubeDb.db"
  }

# Create Db Context
    namespace DemoWebAPIWithPostgreSqlInDocker.Data
    {
        public class ProductDbContext(DbContextOptions<ProductDbContext> options) : DbContext(options)
        {
            public DbSet<Product> Products { get; set; }
        }
    }

  # Register Db Context
    builder.Services.AddDbContext<ProductDbContext>
      (o=>o.UseSqlite(builder.Configuration.GetConnectionString("SQLiteConnection")));

# Create Minimal Api Endpoints
    namespace DemoWebAPIWithPostgreSqlInDocker.Models
    {
        public static class ProductEndpoints
    {
    	public static void MapProductEndpoints (this IEndpointRouteBuilder routes)
        {
            var group = routes.MapGroup("/api/Product").WithTags(nameof(Product));
    
            group.MapGet("/", async (ProductDbContext db) =>
            {
                return await db.Products.ToListAsync();
            })
            .WithName("GetAllProducts")
            .WithOpenApi();
    
            group.MapGet("/{id}", async Task<Results<Ok<Product>, NotFound>> (int id, ProductDbContext db) =>
            {
                return await db.Products.AsNoTracking()
                    .FirstOrDefaultAsync(model => model.Id == id)
                    is Product model
                        ? TypedResults.Ok(model)
                        : TypedResults.NotFound();
            })
            .WithName("GetProductById")
            .WithOpenApi();
    
            group.MapPut("/{id}", async Task<Results<Ok, NotFound>> (int id, Product product, ProductDbContext db) =>
            {
                var affected = await db.Products
                    .Where(model => model.Id == id)
                    .ExecuteUpdateAsync(setters => setters
                      .SetProperty(m => m.Id, product.Id)
                      .SetProperty(m => m.Name, product.Name)
                      .SetProperty(m => m.Description, product.Description)
                      .SetProperty(m => m.Quantity, product.Quantity)
                      );
                return affected == 1 ? TypedResults.Ok() : TypedResults.NotFound();
            })
            .WithName("UpdateProduct")
            .WithOpenApi();
    
            group.MapPost("/", async (Product product, ProductDbContext db) =>
            {
                db.Products.Add(product);
                await db.SaveChangesAsync();
                return TypedResults.Created($"/api/Product/{product.Id}",product);
            })
            .WithName("CreateProduct")
            .WithOpenApi();
    
            group.MapDelete("/{id}", async Task<Results<Ok, NotFound>> (int id, ProductDbContext db) =>
            {
                var affected = await db.Products
                    .Where(model => model.Id == id)
                    .ExecuteDeleteAsync();
                return affected == 1 ? TypedResults.Ok() : TypedResults.NotFound();
            })
            .WithName("DeleteProduct")
            .WithOpenApi();
        }
    }}

# Map the Endpoint
    app.MapProductEndpoints();
    
# Create Docker File
    # See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.
    
    # This stage is used when running from VS in fast mode (Default for Debug configuration)
    FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
    USER app
    WORKDIR /app
    EXPOSE 80
    
    # This stage is used to build the service project
    FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
    ARG BUILD_CONFIGURATION=Release
    WORKDIR /src
    COPY ["DemoWebAPIWithPostgreSqlInDocker/DemoWebAPIWithPostgreSqlInDocker.csproj", "DemoWebAPIWithPostgreSqlInDocker/"]
    RUN dotnet restore "./DemoWebAPIWithPostgreSqlInDocker/DemoWebAPIWithPostgreSqlInDocker.csproj"
    COPY . .
    WORKDIR "/src/DemoWebAPIWithPostgreSqlInDocker"
    RUN dotnet build "./DemoWebAPIWithPostgreSqlInDocker.csproj" -c $BUILD_CONFIGURATION -o /app/build
    
    # This stage is used to publish the service project to be copied to the final stage
    FROM build AS publish
    ARG BUILD_CONFIGURATION=Release
    RUN dotnet publish "./DemoWebAPIWithPostgreSqlInDocker.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false
    
    # This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
    FROM base AS final
    WORKDIR /app
    COPY --from=publish /app/publish .
    ENTRYPOINT ["dotnet", "DemoWebAPIWithPostgreSqlInDocker.dll"]


# Create Docker-Compose File
    services:
        webapi:
          build:  
             context: .
             dockerfile: Dockerfile
          image: webapi_posgresql_v1
          ports:
             - "5003:80"
          environment:
             - ASPNETCORE_URLS=http://+:80
             - ASPNETCORE_ENVIRONMENT=Development
             - ConnectionsStrings__DefaultConnection=Host=postgreserver;username=postgres;password=NetcodeHub2024;database=MyDb; TrustServerCertificate=true;
          depends_on:
              - postgreserver
  
        postgreserver:
          image: postgres:latest
          environment:
            POSTGRES_DB: MyDb
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: NetcodeHub2024
          ports:
            - "5432:5432"
          volumes:
              - pgdata:/var/lib/postgresql/data
  networks:
    default:
      name: my_custom_network 
  
  volumes:
    pgdata:

        
        


  # Build and Run
    - docker-compose build
    - docker-compose up
    
![image](https://github.com/Netcode-Hub/DemoWebAPIWithSQLiteDbInDocker/assets/110794348/b6fe69b4-d5b9-479f-8c63-9258ebad569b) 
![image](https://github.com/user-attachments/assets/8a7b5f33-879a-4567-956b-25d211d2f342)


# Summary:
You've just learned how to build and run your .NET Web API connected to a PostgreSQL database as a Docker image. 🌟 Now you can leverage the power of PostgreSQL in your projects with ease, ensuring a consistent and scalable environment for your applications.

# Here's a follow-up section to encourage engagement and support for Netcode-Hub:
🌟 Get in touch with Netcode-Hub! 📫
1. GitHub: [Explore Repositories](https://github.com/Netcode-Hub/Netcode-Hub) 🌐
2. Twitter: [Stay Updated](https://twitter.com/NetcodeHub) 🐦
3. Facebook: [Connect Here](https://web.facebook.com/NetcodeHub) 📘
4. LinkedIn: [Professional Network](https://www.linkedin.com/in/netcode-hub-90b188258/) 🔗
5. Email: Email: [business.netcodehub@gmail.com](mailto:business.netcodehub@gmail.com) 📧
   
# ☕️ If you've found value in Netcode-Hub's work, consider supporting the channel with a coffee!
1. Buy Me a Coffee: [Support Netcode-Hub](https://www.buymeacoffee.com/NetcodeHub) ☕️
