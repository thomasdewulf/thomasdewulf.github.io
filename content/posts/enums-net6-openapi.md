---
title: User friendly enums in Swagger UI with .NET 6
---

When using enums in a .NET API that has Swagger UI (with **Swashbuckle**), they are shown as numbers by default. I found this not very user friendly as you'd need insight in what the number represents. After some googling, I found this solution:

## 1. Add a Enum Schema Filter

````c#
using Microsoft.OpenApi.Any;
using Microsoft.OpenApi.Models;
using Swashbuckle.AspNetCore.SwaggerGen;

namespace Your.Namespace;

public class EnumSchemaFilter : ISchemaFilter
{
    public void Apply(OpenApiSchema schema, SchemaFilterContext context)
    {
        if (context.Type.IsEnum)
        {
            schema.Enum.Clear();
            Enum.GetNames(context.Type)
                .ToList()
                .ForEach(name => schema.Enum.Add(new OpenApiString($"{name}")));
        }
    }
}
````

This piece of code:

- Clears the current Enum definitions
- Loops over the enum and re-adds the name to the Schema. In `new OpenApiString()`, you can do anything you want. You just have to make sure that model binding can work with it.

## 2. Adapt model binding

At this point, if you make an API request via the Swagger UI, it will send the string value of the Enum in the query or body (depending on how your endpoint works). But, the Json conversion expects an integer. To make strings work, you have to tell the JsonSerializet that it should expect a string. Luckily, this is already built in.

In your `Program.cs`, make following changes to the piece of code that registers the controllers:

```c#
using System.Text.Json.Serialization; // Add to the top of your file

builder.Services.AddControllers(/*Other config*/)).AddJsonOptions(options =>
{

    /*
    .. Other config
    */
    options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
});
```

## 3. That's it!