# Step-by-step Creating ASP.NET MVC Core 1.0

## "ToDoList"

#### link: https://www.microsoft.com/net/core#windowsvs2015


## CREATE A NEW PROJECT IN VISUAL STUDIO

##### 1. Create a new project (Ctrl+Shift+N) in Visual Studio called ToDoList. Select Visual C# > .NET Core > ASP.NET Web Application and click OK.

#### 2. select Empty underneath ASP.NET Core Templates. Deselect the option to "host in the cloud" if it's selected.

#### 3. Add "dependencies" to project.json & Save

```

project.json
----------------------------------------
...
  "dependencies": {
...
    "Microsoft.AspNetCore.Mvc": "1.0.0"
  },
...

Then -Save the files,
     -the message Restoring packages will pops up in the Solution Explorer
= the dependencies have been added
```

#### 4. Go to Startup.cs: add these

```
public void ConfigureServices(IServiceCollection services)
        {
            //1 add the MVC service to the project.
            //The ConfigureServices method is where we configure famework services to our application

            services.AddMvc();
        }

        public void Configure(IApplicationBuilder app)
        {
            //2 - UseMvc(...) - telling our app that it'll be using the MVC framework
            //Then the Configure method is where we tell ASP.NET what frameworks we want to use in our app
            app.UseMvc(routes =>
            {
              //this also set up default routing which tells the project to use the index action of the Home Controller as the default route,
              // and if we have any parameters will be passed as id
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });

            app.Run(async (context) =>
            {
                await context.Response.WriteAsync("Hello World!");

            });
        }
...
```

## To Use Migrations: skip steps 5-7 & jump right to 8-B)  

##### 5. Add Controller
  - Basic Controller:

```

- Right-click the ToDoList project in the Solution Explorer
- select Add > New Folder and name it Controllers.
- Then right-click the Controllers folder
- select Add > New Item.
- In the window that pops up,
- select MVC Controller Class and name it HomeController.cs (make sure you are in the ASP.NET tab).

dotNet Core will automatically generate code in HomeController.cs

NOTE:** HomeController inherits from the Controller class which has public methods, called "actions".
//This action return an instance of IActionResult (this can be many forms, but in this case we choose View
//View method is called w/no parameters = it'll look for the view w/the same name as the action (in this case Index() method will return the **Index view)*

```

#### 6. Add View (in order to see it in our Browser; need to have controller as well)

```
- Add a folder named Views to the project,
- then right-click the Views folder
- add another folder named Home inside of Views folder.
- Right-click the Home folder
- select Add > New Item. (Shift + Ctrl + A)
- Select MVC View Page & change name at bottom of popup window to Index.cshtml.

** By convention, .NET will look for views corresponding to the HomeController in the Home folder within the Views folder.
- now we can add some html to make a welcome page for our project

```

#### 7. We can run  to see the result in Browser

#### 8-A) We can create the database through MS SQL Server Manualy
  - Or We can use MIGRATION through VS Studio to automatically create the database

```
## 1. Open the Microsoft SQL Server Mng Studio

Connect:
Server type: Database Engine
Server name: (localdb)\MSSQLLocalDB
Authentication: Windows Authentication


## 2. Create a new database  and name it ToDoList


Go to Object Explorer, Expand (localdb)\MSSQLLocalDB
- Right-click the Databases folder
- Select New Database
- Type "ToDoList" for the Database name and click OK.


## 3. Create Table


  - Expand the ToDoList database
  - Right-click Tables > click table
  - In the Column Name: ItemId
  - Tab to Datat Type: int
  - Right-click the ItemId & Select Set Primary Key = Unique identifier for this table  
  - When the Primary Key is set the Allow Nulls is automatically deselected
  - Click on the Table file on the window not   at the Object Explorer  
  - Ctrl + Save the table
  - Change table name to the desire Table name When the window is poped up


## 4. In the Properties menu on the right of the table
  - Expand Table Designer
  - at Identity Column
  - Select the drop-down menu,
  - select  LocationId

## 5. Add next Columne Properties and Data Type  

```  
#### 8-B) Create Database Through MIGRATIONS
- You can do this from the begining of the project
- Start New Project from step 7

  1. Add dependencies to project.json
    ```
      "dependencies": {
        "Microsoft.EntityFrameworkCore.SqlServer": "1.0.0",
        "Microsoft.Extensions.Configuration.FileExtensions": "1.0.0",
        "Microsoft.Extensions.Configuration.Json": "1.0.0"
        ...
      },
    ```

  2. Add classes to Models for Category ( these classes will use to build the database )
      - For ToDoList case : Item & Category  
      - Right-click Project's name > Add > New Folder name it "Models"
      - Ctrl+Shift+A when Models is highlight Right-click  new item> Add> Class
      - Change name at the bottom of the pop up to Category.cs (Table_name.cs)
      - Delete using Microsoft.AspNetCore.Mvc;

        ```
          Models/Category.cs
          ******************************************
            using System.Collections.Generic;
            using System.ComponentModel.DataAnnotations;
            using System.ComponentModel.DataAnnotations.Schema;

            namespace ToDoListWithMigrations.Models
            {   [Table("Categories")]
              //Table Naming Convention need to be plural
                public class Category
                {
                    [Key]
                    public int CategoryId { get; set; }
                    public string Name { get; set; }
                    public virtual ICollection<Item> Items { get; set; }
                }
            }

        ```
  3. Add classes to Models for Item  ( these classes will use to build the database )
      - Right-click on the Item that has error red underline
      - Click on the Light bulb and Click Generate class Item in new files

        ```
          Models/Item.cs
          ******************************************
          using System.ComponentModel.DataAnnotations;
          using System.ComponentModel.DataAnnotations.Schema;


          namespace ToDoListWithMigrations.Models
          { [Table("Items")]
            public class Item
            {
                [Key]
                public int ItemId { get; set; }
                public string Description { get; set; }
                public int CategoryId { get; set; }
                public virtual Category Category { get; set; }
            }
          }

        ```

  4. Create ToDoDbContext.cs from Models folder
    - ToDoDbContext extends DbContext.
    - Each DbSet will become a table in the database.
    - Right-click Project's name > Add > New Folder name it "Models"
    - Ctrl+Shift+A when Models is highlight Right-click  new item> Add> Class
    - Change name at the bottom of the pop up to ToDoDbContext.cs **(Dont need to change it to have WithMigrations)**
    - More info about Entity Framework (ef) & DbContext http://patrickdesjardins.com/blog/how-to-register-model-builder-without-having-to-manually-add-them-one-by-one
    ```
      Models/ToDoDbContext.cs
      ----------------------------------
        using Microsoft.EntityFrameworkCore;

        namespace ToDoListWithMigrations.Models
        {
          public class ToDoDbContext : DbContext
          {
              public DbSet<Category> Categories { get; set; }

              public DbSet<Item> Items { get; set; }

              public ToDoDbContext(DbContextOptions<ToDoDbContext> options)
                  : base(options)
              {
              }

              protected override void OnModelCreating(ModelBuilder builder)
              {
                  base.OnModelCreating(builder);
              }
          }
        }
    ```

  5. Next Configure our Startup.cs for our app to use Entity Framework.

    ```
    Startup.cs
    -------------------------------------------------------
      using Microsoft.AspNetCore.Builder;
      using Microsoft.AspNetCore.Hosting;
      using Microsoft.AspNetCore.Http;
      using Microsoft.Extensions.DependencyInjection;
      using Microsoft.Extensions.Configuration;
      using Microsoft.EntityFrameworkCore;
      using Microsoft.EntityFrameworkCore.Infrastructure;
      using ToDoListWithMigrations.Models;

      namespace ToDoListWithMigrations
      {
        public class Startup
        {
            public IConfigurationRoot Configuration { get; set; }
            public Startup(IHostingEnvironment env)
            {
                var builder = new ConfigurationBuilder()
                    .SetBasePath(env.ContentRootPath)
                    .AddJsonFile("appsettings.json");
                Configuration = builder.Build();
            }
            public void ConfigureServices(IServiceCollection services)
            {
                services.AddEntityFramework()
                    .AddDbContext<ToDoDbContext>(options =>
                        options.UseSqlServer(Configuration.GetConnectionStrings("DefaultConnection")));
                    //The ASP.NET Core Configuration system reads the ConnectionString. For local development, it gets the connection string from the appsettings.json file:

            }

            public void Configure(IApplicationBuilder app)
            {

                app.Run(async (context) =>
                {
                    await context.Response.WriteAsync("Hello World!");
                });
            }
        }
      }
    ```  

  6. Go to appsettings.json : Connect database to our app using a connection string
    - Right-click the PRoject Name in Solution Explorer
    - Ctrl+shift+A >ASP.NET at the window select "ASP.NET Configuration File"
    - Add Database name in this case is ToDoListWithMigrations at _CHANGE_ME_

      ```
      {
        "ConnectionStrings": {
          "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=_CHANGE_ME;Trusted_Connection=True;MultipleActiveResultSets=true"
        }
      }
      ```

  7. Set up the app to use data migrations to create the database for us
    - Open up project.json
    - Add "Microsoft.EntityFrameworkCore.Tools" to the "dependencies" and "tools" sections as below.

    ```
    "dependencies": {
      ...
        "Microsoft.EntityFrameworkCore.Tools": {
          "version": "1.0.0-preview2-final",
          "type": "build"
        }
      },

      "tools": {
        ...
        "Microsoft.EntityFrameworkCore.Tools": {
          "version": "1.0.0-preview2-final",
          "imports": [
            "portable-net45+win8+dnxcore50",
            "portable-net45+win8"
          ]
        }
      },
    ```

  8. We need to compile the existing files before we can create the database
    - Right-click Project-name in Solution Explore > select Build (Ctrl+Shift+B)
    - To check if we have any Errors, we need to correct it before generate database
    - Run Build again once all the errors are corrected

  9. in Command Line
    - If these commands are run from an incorrect folder, then you will get an error stating Unable to resolve project.

      ```
        ToDoListWithMigrations/src/ToDo> dotnet ef migrations add Initial
        .../ToDo> dotnet ef database update
       //build the tables at the MSSQL

      ```
        
        - **dotnet ef migrations add Initial** = adds code to the project that Entity Framework will use to update the database.
        - After running cmd above, the project has a new folder named Migrations, containing this code with file ending *_Initial.cs*
        - **Note:** migration names should usually be something descriptive. For instance, if we were adding a column to the Item table to indicate whether or not it has been completed, we could name the migration something like like AddCompletedColumn.
        - **dotnet ef database update** = creates or updates the actual database.
        - You can see that after running this command, the Project contains an Items and Categories tables.
        - If you created the initial migration when the database already exists, the database creation code is generated but it doesn't have to run because the database already matches the data model. When you deploy the app to another environment where the database doesn't exist yet, this code will run to create your database, so it's a good idea to test it first. That's why you changed the name of the database in the connection string earlier -- so that migrations can create a new one from scratch.

  10. Connect to MSSQL Server to check if the database created
      - To save Database to project (we can do this once the ConnectionString property has been initialized.)
      - Right-click Database-Name: click Task > import data

      ```
        Server type: Database Engine
        Server name: (localdb)\MSSQLLocalDB
        Authentication: Windows Authentication
      ```

  11. If you need to update the database, for example forgot to make table name into plural:
      - correct all the table names & Build the project and then add the migration in the command prompt:  

      ```
        ToDoListWithMigrations/src/ToDo> dotnet ef migrations add MakeTableNamesPlural
      ```

  12. In order for Entity Framework to update the database with this change, we need to tell it which migration to use.
      - The migration name is the name of the migration file that is generated.
      -  When I ran the above migration, Entity Framework created the file *20160413182147_MakeTableNamesPlural.cs.*
      - To update the database, I'll use this command:     

      ```
        ToDoListWithMigrations/src/ToDo> dotnet ef database update 20160413182147_MakeTableNamesPlural

        **
        When we run this command, If we get the following error:

        > Cannot find the object "Items" because it does not exist or you do not have permissions.

      ```

  13. Let's look in the migration file and see what's going on.
      - In the _MakeTableNamesPlural class_, there's an Up method and a Down method, which is the code that runs when you add or remove a migration from the database, respectively.
      - Migrations calls the Up method to implement the data model changes for a migration. When you enter a command to roll back the update, Migrations calls the Down method.
      - The _Up method_ should look something like this:   

        ```
        Migrations/20160413182147_MakeTableNamesPlural.cs
        _______________________________________________________________________
        ...
        protected override void Up(MigrationBuilder migrationBuilder)
        {
          migrationBuilder.DropForeignKey(name: "FK_Item_Category_CategoryId", table: "Item");
          migrationBuilder.AddForeignKey(
              name: "FK_Item_Category_CategoryId",
              table: "Items",
              column: "CategoryId",
              principalTable: "Categories",
              principalColumn: "CategoryId",
              onDelete: ReferentialAction.Cascade);
          migrationBuilder.RenameTable(
              name: "Item",
              newName: "Items");
          migrationBuilder.RenameTable(
              name: "Category",
              newName: "Categories");
        }
        ...

        ```
      - **If we get the error Cannot find the object "Items"** : This mean we haven't change our table names in the models yet
      - We rewrite the code to rename the table to plural using **.RenameTable** like this:

        ```
        Migrations/20160413182147_MakeTableNamesPlural.cs
        _____________________________________________________
        ...
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropForeignKey(name: "FK_Item_Category_CategoryId", table: "Item");
            migrationBuilder.RenameTable(
                name: "Item",
                newName: "Items");
            migrationBuilder.RenameTable(
                name: "Category",
                newName: "Categories");
            migrationBuilder.AddForeignKey(
                name: "FK_Item_Category_CategoryId",
                table: "Items",
                column: "CategoryId",
                principalTable: "Categories",
                principalColumn: "CategoryId",
                onDelete: ReferentialAction.Cascade);
        }
        ...

        ```

      - Now we've renamed the table before adding the foreign key to it. Save the file, and then run the command to update the database:

        ```
          > dotnet ef database update 20160413182147_MakeTableNamesPlural
        ```

      - And the names of the tables in the database have been updated!

  14. In the migrations folder >ProjectDbContextModelSnapshot.cs
      - This also contain the relationship of the tables

  15. PROBLEM ** After created migration, When I try to import data from SQl to save to the file, Kept getting:
      - Cannot save data, there's no connection!

  16. Check out for more information about Entity Framework Code First Migrations

      **https://msdn.microsoft.com/en-us/library/jj591621(v=vs.113).aspx**  
  17. To remove Migration in the Powershell
      ```
         dotnet ef migrations remove command.
      ```   
    

#### 9. or Follow along this lesson for Migrations: https://docs.microsoft.com/en-us/ef/core/get-started/aspnetcore/new-db      
#### 10. To run Migration in Nuget Package Manager Console
  ```
  Here's the CMD for 1st  time using Migration in the Visual Studio 
  At Menu Bar  >  Tools ‣ Nuget Package Manager ‣ Package Manager Console       
  At the bottom the Package Manger Console Bar will pop open. Type in 
   PM> Install-Package EntityFramework

  for first time using the migration in each project  
   PM> Enable-Migrations

   PM> Add-Migration InitialCreate  or MigrationName 
    PM>Update-Database

   to Add more feature

   Add-Migration AddImage 
   Update-Database

   to Delete
   Remove-Migration (edited)
    
  ```
  
    
     - This command has added a Migrations folder to our project, this new folder contains two files:
        - The Configuration class. This class allows you to configure how Migrations behaves for your context. For this walkthrough we will just use the default configuration. Because there is just a single Code First context in your project, Enable-Migrations has automatically filled in the context type this configuration applies to.
      An InitialCreate migration. This migration was generated because we already had Code First create a database for us, before we enabled migrations. The code in this scaffolded migration represents the objects that have already been created in the database. In our case that is the Blog table with a BlogId and Name columns. The filename includes a timestamp to help with ordering. If the database had not already been created this InitialCreate migration would not have been added to the project. Instead, the first time we call Add-Migration the code to create these tables would be scaffolded to a new migration.
      Multiple Models Targeting the Same Database
      When using versions prior to EF6, only one Code First model could be used to generate/manage the schema of a database. This is the result of a single __MigrationsHistory table per database with no way to identify which entries belong to which model.
      Starting with EF6, the Configuration class includes a ContextKey property. This acts as a unique identifier for each Code First model. A corresponding column in the __MigrationsHistory table allows entries from multiple models to share the table. By default, this property is set to the fully qualified name of your context.
      Generating & Running Migrations
      Code First Migrations has two primary commands that you are going to become familiar with.
      Add-Migration will scaffold the next migration based on changes you have made to your model since the last migration was created
      Update-Database will apply any pending migrations to the database
      We need to scaffold a migration to take care of the new Url property we have added. The Add-Migration command allows us to give these migrations a name, let’s just call ours AddBlogUrl.
      Run the Add-Migration AddBlogUrl command in Package Manager Console
      In the Migrations folder we now have a new AddBlogUrl migration. The migration filename is pre-fixed with a timestamp to help with ordering
