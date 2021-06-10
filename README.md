# MySQL ASP.NET 5.0

Convert an [ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-5.0) Web Application project to use [MySQL](https://www.mysql.com/) with [Entity Framework](https://docs.microsoft.com/en-us/ef/), enabling development on macOS, linux, or Windows targets using IDEs such as [VS Code](https://code.visualstudio.com/), [Visual Studio](https://visualstudio.microsoft.com/), or [JetBrains Rider](https://www.jetbrains.com/rider/).

This project uses [.NET 5.0](https://dotnet.microsoft.com/download/dotnet/5.0) target framework, ASP.NET Core Web Application (Model-View-Controller) project scaffold from Visual Studio 2019 (version 16.10.1) to connect to MySQL 8.0.

![vscode](https://user-images.githubusercontent.com/1213591/106405974-812cba80-63fd-11eb-9c22-3f8eeff9136f.png)

For previous versions of .NET Core 3.x, 2.x, 1.x, see the [releases](https://github.com/jasonsturges/mysql-dotnet-core/releases) for past implementations in this repository.


## Quick Start

To immediately use this solution, make sure your [environment setup](#environment-setup) is complete; then, jump to [running the solution](#running-the-solution).


## Environment Setup

Make sure you have the [.NET 5.0 SDK](https://dotnet.microsoft.com/download) installed on your system.

If you're using Visual Studio Code, you will need to generate ASP.NET Core developer certificates by issuing the following commands from a terminal:

    dotnet dev-certs https --clean
    dotnet dev-certs https

For command line `database ef` commands, you will need to install Entity Framework Core tools .NET CLI:

    dotnet tool install --global dotnet-ef
    
Make sure you have [MySQL 8.0 Server](https://dev.mysql.com/downloads/) installed on your system; or, use a [Docker image](https://hub.docker.com/_/mysql) instead of installing MySQL Server.  In a terminal, execute the following to spin up a Docker image of MySQL:

    docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=mypassword -d mysql


## Running the solution

Before the solution can be executed, Entity Framework migrations must be run to setup the database.

Configure connection string in project's appsettings.json, replacing the `username`, `password`, and `database` appropriately:

```cs
"ConnectionStrings": {
  "DefaultConnection":"server=localhost;userid=myusername;password=mypassword;database=mydatabase;"
},
```

Execute the migration using either Visual Studio Package Manager Console (from menu: Tools -> NuGet Package Manager -> Package Manager Console):

    >> Update-Database

Or, from the command line via DotNet CLI, execute the following command inside the project directory, **where the .csproj file is located**:

    $ dotnet ef database update

After running the migration, the database is created and web application is ready to be run.

Run the solution via your IDE; or, execute the following command line

    dotnet run

Then, load via browser to either https or http endpoints:

- https://localhost:5001
- http://localhost:5000


## Project Setup

Project setup has already been completed in this repository, ready for use as a template for your next project.

Otherwise, adapt the steps below to incorporate MySQL into your solution.

### Install NuGet packages

Install the `MySql.EntityFrameworkCore` NuGet package in the ASP.NET web application.

To do this, you can use the `dotnet` command line by executing:

    dotnet add package MySql.EntityFrameworkCore --version 5.0.3.1

Or, edit the project's .csproj file and add the following line in the `PackageReference` item group:

    <PackageReference Include="MySql.EntityFrameworkCore" Version="5.0.3.1" />

### Modify Startup.cs

In `Startup.cs` under `ConfigureServices()` method, replace the `UseSqlServer` / `UseSqlite` option with MySQL:

```cs
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseMySQL(Configuration.GetConnectionString("DefaultConnection")));
```

### Migration Issues with DbContext

Upon upgrading MySQL Oracle Connector, Entity Framework migrations may fail with the errors:

> MySql.Data.MySqlClient.MySqlException (0x80004005): Specified key was too long; max key length is 3072 bytes
> 
> MySql.Data.MySqlClient.MySqlException (0x80004005): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'max) NULL, PRIMARY KEY (`Id`))
> 
> Failed executing DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
> ```sql
> CREATE TABLE `AspNetRoles` (
>     `Id` TEXT NOT NULL,
>     `Name` TEXT NULL,
>     `NormalizedName` TEXT NULL,
>     `ConcurrencyStamp` TEXT NULL,
>     PRIMARY KEY (`Id`)
> );
> ```
> MySql.Data.MySqlClient.MySqlException (0x80004005): BLOB/TEXT column 'Id' used in key specification without a key length
> ```sql
> CREATE TABLE `AspNetRoles` (
>     `Id` nvarchar(450) NOT NULL,
>     `Name` nvarchar(256) NULL,
>     `NormalizedName` nvarchar(256) NULL,
>     `ConcurrencyStamp` nvarchar(max) NULL,
>     PRIMARY KEY (`Id`)
> );
> ```
> MySql.Data.MySqlClient.MySqlException (0x80004005): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'max) NULL,


To resolve this, add the following code within the ApplicationDbContext.cs `OnModelCreating()`.

```cs
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : IdentityDbContext
{

    // ...

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        builder.Entity<IdentityRole>(entity => entity.Property(m => m.Id).HasMaxLength(450));
        builder.Entity<IdentityRole>(entity => entity.Property(m => m.ConcurrencyStamp).HasColumnType("varchar(256)"));

        builder.Entity<IdentityUserLogin<string>>(entity =>
        {
            entity.Property(m => m.LoginProvider).HasMaxLength(127);
            entity.Property(m => m.ProviderKey).HasMaxLength(127);
        });

        builder.Entity<IdentityUserRole<string>>(entity =>
        {
            entity.Property(m => m.UserId).HasMaxLength(127);
            entity.Property(m => m.RoleId).HasMaxLength(127);
        });

        builder.Entity<IdentityUserToken<string>>(entity =>
        {
            entity.Property(m => m.UserId).HasMaxLength(127);
            entity.Property(m => m.LoginProvider).HasMaxLength(127);
            entity.Property(m => m.Name).HasMaxLength(127);
        });
    }
```

Then, generate a new migration using Visual Studio Package Manager Console (from menu: Tools -> NuGet Package Manager -> Package Manager Console):

    >> Add-Migration

Or, from the command line via DotNet CLI:

    $ dotnet ef migrations add CreateIdentitySchema
    

## Troubleshooting

### Create Entity Framework Migration Table in MySQL

If running `dotnet ef` fails initially, the `__efmigrationshistory` table may not exist.  Past versions of Entity Framework migration tools failed to create this table.  

Assure you're running the lastest tools:

    dotnet tool update --global dotnet-ef

Otherwise, manually create the migrations history table in the MySQL database by executing the following SQL script.

```sql
use mydatabase;

CREATE TABLE `mydatabase`.`__EFMigrationsHistory` (
  `MigrationId` text NOT NULL,
  `ProductVersion` text NOT NULL,
  PRIMARY KEY (`MigrationId`(255)));
```

### Deprecated MySQL NuGet Packages

Note that `MySql.Data.EntityFrameworkCore` NuGet package is deprecated, and is now: `MySql.EntityFrameworkCore`.
