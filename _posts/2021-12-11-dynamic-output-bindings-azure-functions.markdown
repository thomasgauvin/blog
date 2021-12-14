---
layout: post
title:  "Dynamic Output Bindings with Azure Functions"
permalink: "dynamic-output-bindings-azure-functions"
date:   2021-12-11 18:02:06 -0400
---

![](../uploads/2021-12-12-12-55-39.png)

Bindings are a powerful Azure Functions feature which provide native integrations to other resources or services, such as databases or storage accounts. This makes it easier and faster to write functions that rely on other services. Two types of bindings exist: input (to import data) and output (to export data).

Bindings are specified *declaratively* in the `function.json` file. **While this declarative style is convenient, it does not allow *dynamic binding*.** In other words, the binding can not be computed in the code at runtime, which is necessary in scenarios where the output destination is decided by certain logic in the code.

***For all non-.NET languages (Python, Java, JavaScript, TypeScript, etc.)*, dynamic binding is not possible with native bindings.** Achieving this functionality would require implementing the output without using native output bindings, such as by using a client library to call the service directly [[see reference]](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp#binding-at-runtime).

***For C# and .NET languages*, native dynamic binding is supported.** This is done through the use of the [Binder](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Bindings/Runtime/Binder.cs) or [IBinder](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IBinder.cs) classes. This is how to do it:

<br><br />

In this example, we have a Blob object that needs to be output from an Azure Function in either `containerA` or `containerB` based on some logic.

### 1. In function.json, remove declarative output bindings

We'll be making use of the Binder class to bind dynamically. So, we need to remove the declarative binding from the function.json file.

``` diff
{
  "bindings": [
    {
      "authLevel": "function",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    }
-   {
-     "name": "outputBlob",
-     "direction": "out",
-     "type": "blob",
-     "path": "output/{rand-guid}",
-     "connection": "azurefunctionsoutput_STORAGE"
-   }
  ]
}
```

### 2. In function.cs/run.csx, create a dynamic binding attribute and output to the binding

A few steps are required in this file:

1. Add the `Binder binder` parameter to the method parameters
2. Dynamically compute the binding type attribute (in our case BlobAttribute)
3. Bind the output to the binding type attribute and output

<details>
  <summary>(imports)</summary>

  <pre>
  using System;
  using System.IO;
  using System.Threading.Tasks;
  using Microsoft.AspNetCore.Mvc;
  using Microsoft.Azure.WebJobs;
  using Microsoft.Azure.WebJobs.Extensions.Http;
  using Microsoft.AspNetCore.Http;
  using Microsoft.Extensions.Logging;
  using Newtonsoft.Json;
  using System.Net;
  </pre>
</details>

``` diff
namespace FunctionApp
{
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
+           Binder binder,
            ILogger log)
        {
            string fileName = req.Query["fileName"];

            BlobAttribute dynamicBlobAttribute;

+           //dynamic assignment of Blob attribute 
+           if (req.Query["container"] == "A")
+           {
+               dynamicBlobAttribute = new BlobAttribute($"container-a/{fileName}", FileAccess.Write);
+           }
+           else
+           {
+               dynamicBlobAttribute = new BlobAttribute($"container-b/{fileName}", FileAccess.Write);
+           }

            //open stream to image for example's sake
            WebClient myWebClient = new WebClient();
            Stream myStream = myWebClient.OpenRead("https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/Cat_poster_1.jpg/1200px-Cat_poster_1.jpg");

+           //Bind output with blob attribute and copy stream
+           using (var output = await binder.BindAsync<Stream>(dynamicBlobAttribute))
+           {
+               await myStream.CopyToAsync(output);
+           }
        }
    }
}
```

By adding this code to our method, we can imperatively/dynamically indicate the output we want to use for our Azure Function. This can be quite helpful in scenarios where Azure Cognitive Services are used to determine how to process an uploaded blob, among others. 

### Going further

What if we needed to output to multiple sources computed dynamically? This can be achieved by passing an array of attributes when binding to the binder, as discussed in the [docs](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp#multiple-attribute-example).

Taken from [Microsoft Docs](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp#multiple-attribute-example):
>``` c#
>using Microsoft.Azure.WebJobs;
>using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;
>
>public static async Task Run(string input, Binder binder)
>{
>    var attributes = new Attribute[]
>    {
>        new BlobAttribute("samples-output/path"),
>        new StorageAccountAttribute("MyStorageAccount")
>    };
>    using (var writer = await binder.BindAsync<TextWriter>(attributes))
>    {
>        writer.Write("Hello World!");
>    }
>}
>```

However, this is not supported when attempting to output to multiple sources of the same type. Attempting to do so results in the following runtime error: `System.Private.CoreLib: Exception while executing function: Function1. System.Private.CoreLib: Multiple custom attributes of the same type found.` The workaround that I have found is to bind to the outputs sequentially as such:

``` c#
//Bind output with blob attribute and copy stream
using (var output = await binder.BindAsync<Stream>(dynamicBlobAttribute))
{
    await myStream.CopyToAsync(output);
}

using (var output = await binder.BindAsync<Stream>(new BlobAttribute($"container-c/{fileName}", FileAccess.Write)))
{
    await myStream.CopyToAsync(output);

}
```

### Conclusion

We've seen how bindings with Azure Functions can facilitate data input and output, and how C# and .NET Functions offer advanced functionality with dynamic binding/binding at runtime. 

I hope you enjoyed the article and welcome any feedback! (If you find a typo or a better way of implementing the logic, definitely let me know)


### References and helpful articles: 

[Microsoft Docs](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp#binding-at-runtime), [https://www.rickvandenbosch.net/blog/dynamic-output-bindings-in-azure-functions/](https://www.rickvandenbosch.net/blog/dynamic-output-bindings-in-azure-functions/)
