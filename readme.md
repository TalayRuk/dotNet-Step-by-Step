# Step-by-step Creating ASP.NET MVC Core 1.0

## "ToDoList"

#### link: https://www.microsoft.com/net/core#windowsvs2015


## CREATE A NEW PROJECT IN VISUAL STUDIO

##### 1. Create a new project in Visual Studio called ToDoList. Select Visual C# > .NET Core > ASP.NET Web Application and click OK.

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

            namespace ToDoListWithMigrations.Models
            {
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

          namespace ToDoListWithMigrations.Models
          {
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
                        options.UseSqlServer(Configuration["ConnectionStrings:DefaultConnection"]));
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
    - Right-click Project-name in Solution Explore > select Build
    - To check if we have any Errors, we need to correct it before generate database
    - Run Build again once all the errors are corrected

  9. in Command Line

    ```
      ToDoListWithMigrations/src/ToDo> dotnet ef migrations add Initial
     .../ToDo> dotnet ef database update
     //build the tables at the MSSQL

    ```
