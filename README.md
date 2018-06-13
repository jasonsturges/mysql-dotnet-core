# MySQL ASP.NET Core 2.1

Convert an ASP.NET Core Web Application project to use MySQL with Entity Framework.

This enables development of ASP.NET Core projects using [VS Code](https://code.visualstudio.com/) on Mac OS X / macOS or linux targets.

![vscode](http://labs.jasonsturges.com/coreclr/mysql-dotnet-core.png)

This project uses .NET Core 2.1 target framework, ASP.NET Core Web Application project scaffold from Visual Studio 2017 (version 15.7.3).


## Project Setup

Project setup has already been completed in this repository.

Below, instructions are referenced to use MySQL in a ASP.NET Core project.


### Install NuGet packages

Install the `MySql.Data.EntityFrameworkCore` NuGet package in the ASP.NET web application.

To do this, you can use the `dotnet` command line by executing:

    $ dotnet add package MySql.Data.EntityFrameworkCore --version 8.0.11

Or, edit the project's .csproj file and add the following line in the `PackageReference` item group:

    <PackageReference Include="MySql.Data.EntityFrameworkCore" Version="8.0.11" />


### Update appsettings.json

Configure connection string in project's appsettings.json, replacing the `username`, `password`, and `database` appropriately:

    "ConnectionStrings":{
        "DefaultConnection":"server=localhost;userid=myusername;password=mypassword;database=mydatabase;"
    },


### Modify Startup.cs

Add using statements to `Startup.cs` source code:

    using MySql.Data.EntityFrameworkCore;
    using MySql.Data.EntityFrameworkCore.Extensions;

Then in the same file's `ConfigureServices()` method, replace the `UseSqlite` option with MySQL:

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        // Add framework services.
        services.AddDbContext<ApplicationDbContext>(options =>
                options.UseMySQL(Configuration.GetConnectionString("DefaultConnection")));


### Migration Issues with DbContext

Upon upgrading MySQL Oracle Connector, entity framework migrations were failing with the error:

> MySql.Data.MySqlClient.MySqlException (0x80004005): Specified key was too long; max key length is 3072 bytes

To resolve this, add the following code within the ApplicationDbContext.cs `OnModelCreating()`.

    using Microsoft.AspNetCore.Identity;

    // ...

        protected override void OnModelCreating(ModelBuilder builder)
        {
            // ...

            // Shorten key length for Identity
            builder.Entity<ApplicationUser>(entity => entity.Property(m => m.Id).HasMaxLength(127));
            builder.Entity<IdentityRole>(entity => entity.Property(m => m.Id).HasMaxLength(127));
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

Then, generate a new migration using Visual Studio Package Manager Console (from menu: Tools -> NuGet Package Manager -> Package Manager Console):

    >> Add-Migration

Or, from the command line via DotNet CLI:

    $ dotnet ef migrations add Initial


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

Execute the migration using either Visual Studio Package Manager Console (from menu: Tools -> NuGet Package Manager -> Package Manager Console):

    >> Update-Database

Or, from the command line via DotNet CLI, execute the following command inside the project directory, **where the .csproj file is located**:

    $ dotnet ef database update

After running the migration, the database is created and web application is ready to be run.
