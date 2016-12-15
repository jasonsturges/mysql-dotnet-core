Convert an ASP.NET Core project to use MySQL with Entity Framework.

### Install NuGet packages

Install the following NuGet package in the ASP.NET web application project:

    MySql.Data.EntityFrameworkCore


### Update appsettings.json

Configure connection string in project's `appsettings.json`:

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
