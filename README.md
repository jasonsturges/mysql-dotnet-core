# MySQL ASP.NET Core 2.0

Convert an ASP.NET Core Web Application project to use MySQL with Entity Framework.

This enables development of ASP.NET Core projects using [VS Code](https://code.visualstudio.com/) on Mac OS X / macOS or linux targets.

![vscode](http://labs.jasonsturges.com/coreclr/mysql-dotnet-core.png)

This repository uses ASP.NET Core 2.0 Visual Studio 2017 ASP.NET Core Web Application project scaffold updated to use MySQL.

## Note about compatibility with .NET Core 2.0

There is currently an issue with Oracle's MySQL connector and .NET Core 2.0.  You may receive an error stating:

> System.TypeLoadException occurred HResult=0x80131522 Message=Method 'Clone' in type 

In the interim, I suggest using [Pomelo](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql/2.0.0-rtm-10062), which can be installed by executing the following command:

    $ dotnet add package Pomelo.EntityFrameworkCore.MySql --version 2.0.0-rtm-10062

Or, add the following line to your .csproj `ItemGroup`:

    <PackageReference Include="Pomelo.EntityFrameworkCore.MySql" Version="2.0.0-rtm-10062" />

In your Startup.cs, remove these using statements:

    // Remove these lines
    using MySQL.Data.EntityFrameworkCore;
    using MySQL.Data.EntityFrameworkCore.Extensions;

Then, change the casing of `UseMySQL` to `UseMySql`:

```
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddDbContext<ApplicationDbContext>(options =>
            options.UseMySql(Configuration.GetConnectionString("DefaultConnection")));
```

After that, you should have full MySQL Entity Framework functionality with .NET Core 2.


## Project Setup

Project setup has already been completed in this repository.

Below, instructions are referenced to use MySQL in a ASP.NET Core project.


### Install NuGet packages

Install the `MySql.Data.EntityFrameworkCore` NuGet package in the ASP.NET web application.

To do this, you can use the `dotnet` command line by executing:

    $ dotnet add package MySql.Data.EntityFrameworkCore --version 8.0.8-dmr

Or, edit the project's .csproj file and add the following line in the `PackageReference` item group:

    <PackageReference Include="MySql.Data.EntityFrameworkCore" Version="8.0.8-dmr" />


### Update appsettings.json

Configure connection string in project's appsettings.json, replacing the `username`, `password`, and `database` appropriately:

    "ConnectionStrings":{
        "DefaultConnection":"server=localhost;userid=myusername;password=mypassword;database=mydatabase;"
    },


### Modify Startup.cs

Add using statements to `Startup.cs` source code:

    using MySQL.Data.EntityFrameworkCore;
    using MySQL.Data.EntityFrameworkCore.Extensions;

Then in the same file's `ConfigureServices()` method, replace the `UseSqlite` option with MySQL:

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        // Add framework services.
        services.AddDbContext<ApplicationDbContext>(options =>
                options.UseMySQL(Configuration.GetConnectionString("DefaultConnection")));


## Running the solution

Before the solution can be executed, be sure to run entity framework migrations.


### Create Entity Framework Migration Table in MySQL

Running the `dotnet ef` fails initially as the `__efmigrationshistory` table doesn't exist.  Until this is resolved by the Entity Framework migration tools, manually create the migrations history table in the MySQL database by executing the following SQL script.

    use mydatabase;

    CREATE TABLE `mydatabase`.`__EFMigrationsHistory` (
      `MigrationId` text NOT NULL,
      `ProductVersion` text NOT NULL,
      PRIMARY KEY (`MigrationId`(255)));


### Run Entity Framework Migrations

Execute the following comment inside the project directory, where the `project.json` file is located:

    $ dotnet ef database update

After running the migration, the database is created and web application is ready to be run.
