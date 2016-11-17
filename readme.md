# Step-by-step Creating ASP.NET MVC Core 1.0

## "ToDoList"

#### link: https://www.microsoft.com/net/core#windowsvs2015


## CREATE A NEW PROJECT IN VISUAL STUDIO

1. Create a new project in Visual Studio called ToDoList. Select Visual C# > .NET Core > ASP.NET Web Application and click OK.

2. select Empty underneath ASP.NET Core Templates. Deselect the option to "host in the cloud" if it's selected.

3. Add "dependencies" to project.json & Save

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

4. Go to Startup.cs: add these

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

5. Add Controller
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

6. Add View (in order to see it in our Browser; need to have controller as well)

```
- Add a folder named Views to the project,
- then right-click the Views folder
- add another folder named Home inside of Views folder.
- Right-click the Home folder
- select Add > New Item. (Shift + Ctrl + A)
- Select MVC View Page & change name at bottom of popup window to Index.cshtml.

** By convention, .NET will look for views corresponding to the HomeController in the Home folder within the Views folder.

```
