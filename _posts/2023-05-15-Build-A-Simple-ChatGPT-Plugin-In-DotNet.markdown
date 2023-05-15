# Build A Simple ChatGPT Plugin In DotNet

## Introduction

In March of this year, OpenAI announced a plugin model with which developers can connect ChatGPT to external systems via APIs. These plugins enable ChatGPT to query external systems (for instance, live data), but also make it possible for ChatGPT to perform tasks on behalf of the user. This is potentially a big step in making ChatGPT an even more personal assistant.

Access to plugins was restricted, but last Friday OpenAI [made the feature available for all ChatGPT Plus users](https://help.openai.com/en/articles/6825453-chatgpt-release-notes). The feature can be found in the Beta features and can be enabled there. There are already quite a few plugins available in the Plugin Store, such as plugins for Expedia, Wolfram, and Zapier.

For plugins to be available to the public, they need to be accepted in the Plugin Store, but OpenAI also provides tooling for developers to run unverified plugins. This makes it possible to start developing your own plugins now!

Plugins are basically [REST APIs with an OpenAPI specification and a separate standardized manifest file](https://platform.openai.com/docs/plugins/introduction). The manifest contains some metadata so that ChatGPT knows what to expect from the API. ChatGPT also gets information from the OpenAPI specification. It is possible to run the plugins from localhost and use them in ChatGPT, which makes them really easy to develop and test.

## Building a Plugin

To start experimenting with plugins I created an application in ASP.NET Core, using the ASP.NET Core Web App template in Visual Studio. When configuring a project, make sure you have the checkmark “Enable OpenAPI support” switched on. This will add the Swagger Generation, enables SwaggerUI, etc. You can find [more information about Swagger / OpenAPI here](https://learn.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle). 

My test project is a really simple ToDo project (how original), without a data store and there is no logged in user. You can find the code [here on Github](https://github.com/camieleggermont/ChatGPT-Plugin-dotnet). The API has a few endpoints to create ToDo items, list them (all, based on a search term, or on due date), and mark the item as completed.

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled.png)

### Adding extra information to the API definition

ChatGPT gets its information about the plugin from two things: the manifest (we’ll talk about that later) and the OpenAPI specification. By default, the metadata in the OpenAPI specification is quite limited, but it can be extended. I’ve took the approach to add extra information for the api controllers and operations [via XML annotations](https://learn.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-7.0&tabs=visual-studio#xml-comments), but there are other options as well. I’ve also configured some meta info about the API itself, and added OperationIds to the OpenAPI specification file. These are not included by default, but ChatGPT needs them. 

To add the meta info, and use them in the API specifications:

1. Add the following to the project file of your web api

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

1. Update the configuration for SwaggerGen in program.cs: 

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.CustomOperationIds(apiDesc =>
    {
        return apiDesc.TryGetMethodInfo(out MethodInfo methodInfo) ? methodInfo.Name : null; // Add operationId
    });

    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Version = "v1",
        Title = "ToDo API",
        Description = "An ASP.NET Core Web API for managing ToDo items",
    });

    // using System.Reflection;
    var xmlFilename = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, xmlFilename));
});
```

1. In your controllers, add the XML annotations: 

```csharp
/// <summary>
        /// Search for ToDo items by title.
        /// </summary>
        /// <param name="term">The search term</param>
        /// <returns>The found todo items</returns>
        [HttpGet("search/{term}")]
        public async Task<IEnumerable<ToDo>> Search(string term)
        {
            return await toDoRepository.SearchAsync(term);
        }
```

### Adding the manifest file

The other thing needed to run the plugin is the manifest file. This file needs to be served from “<host>/.well-known/ai-plugin.json”, and contains info about the plugin. Here’s mine: 

```json
{
  "schema_version": "v1",
  "name_for_human": "TODO Plugin",
  "name_for_model": "todo",
  "description_for_human": "Plugin for managing a TODO list. You can add, complete and list todos (all, search on term and search on due date)",
  "description_for_model": "Plugin for managing a TODO list. You can add and complete TODOs. You can also list todos (all, search on term and search on due date)",
  "auth": {
    "type": "none"
  },
  "api": {
    "type": "openapi",
    "url": "https://localhost:7026/swagger/v1/swagger.json",
    "is_user_authenticated": false
  },
  "logo_url": "https://localhost:7026/todo.png",
  "contact_email": "camiel.eggermont@gmail.com",
  "legal_info_url": "http://camiel.io/legal"
}
```

To add this file in your project:

1. Create a folder “wwwroot” in your project. This folder will be used to serve static files.
2. Enable serving of static files in program.cs: 

```csharp
app.UseStaticFiles();
```

1. In the wwwroot folder, create a subfolder “.well-known”
2. In the well-known folder, create the ai-plugins.json file.

Note that there are some limitations to the manifest file. For instance, the description_for_human property can only be 120 characters long. 

### Set up CORS

ChatGPT calls your plugin (the API, OpenAPI specification and manifest) from another origin. Unless you explicitly allow that it is denied. I added an “all allowed” CORS policy, which is probably not the best idea for a production environment, but will do for my test. In program.cs, do the following:

1. Add CORS 

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("Open", builder => builder.AllowAnyOrigin().AllowAnyHeader().AllowAnyMethod());
});
```

1. Use CORS 

```csharp
app.UseCors("Open");
```

## Configure the plugin in ChatGPT

Now that all the code is built you can run it on your own machine. My project runs on [https://localhost:7026/](https://localhost:7026/). You can check whether CORS works correctly, for instance by using [https://www.test-cors.org/](https://www.test-cors.org/) to test [https://localhost:7026/swagger/v1/swagger.json](https://localhost:7026/swagger/v1/swagger.json) (the OpenAPI specification), [https://localhost:7026/.well-known/ai-plugin.json](https://localhost:7026/.well-known/ai-plugin.json) (the manifest) and [https://localhost:7026/api/ToDo](https://localhost:7026/api/ToDo) (an API endpoint). When this all works, you can make the magic happen by adding your development plugin to ChatGPT. To do this, head over to [https://chat.openai.com/](https://chat.openai.com/) and start a new chat. Choose the GPT-4 model, and select “plugins” from the dropdown menu.

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%201.png)

This will show the plugin menu. If you already experimented with some plugins, they will show here. Go to the plugin store from here:

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%202.png)

In the plugin store you can find all kinds of external plugins. We’re going to select “Develop your own plugin”:

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%203.png)

In the dialog that shows, we’ll put the URL where the plugin runs, and click “Find manifest file”. ChatGPT will try to find the manifest file and will parse it for all information.

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%204.png)

If there are errors in the manifest or in the OpenAPI specification, a dialog will be shown with some info:

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%205.png)

And if everything is ok, we can install the plugin:

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%206.png)

Once this is done, the plugin is installed, but ofcourse only for our own use. 

## Using the plugin

Now we can use the plugin! ChatGPT decides when it should use the plugin for it’s work, and this is totally integrated in the chat:

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%207.png)

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%208.png)

![Untitled]({{ site.baseurl }}/images/chatgpt_plugins/Untitled%209.png)

## Wrap up

The model for writing these plugins is fairly simple, but provide big opportunities. 

In this example we haven’t looked at things like authorization, but I think it does demonstrate how things work. You can find the code for [my test project on Github](https://github.com/camieleggermont/ChatGPT-Plugin-dotnet).