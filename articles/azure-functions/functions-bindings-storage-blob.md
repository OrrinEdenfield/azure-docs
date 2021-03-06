---
title: Azure Blob storage bindings for Azure Functions
description: Understand how to use Azure Blob storage triggers and bindings in Azure Functions.
author: craigshoemaker

ms.topic: reference
ms.date: 11/15/2018
ms.author: cshoe
---

# Azure Blob storage bindings for Azure Functions

This article explains how to work with Azure Blob storage bindings in Azure Functions. Azure Functions supports trigger, input, and output bindings for blobs. The article includes a section for each binding:

* [Blob trigger](#trigger)
* [Blob input binding](#input)
* [Blob output binding](#output)

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!NOTE]
> Use the Event Grid trigger instead of the Blob storage trigger for blob-only storage accounts, for high scale, or to reduce latency. For more information, see the [Trigger](#trigger) section.

## Packages - Functions 1.x

The Blob storage bindings are provided in the [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) NuGet package, version 2.x. Source code for the package is in the [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.Storage/Blob) GitHub repository.

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## Packages - Functions 2.x and higher

The Blob storage bindings are provided in the [Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) NuGet package, version 3.x. Source code for the package is in the [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs) GitHub repository.

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## Trigger

The Blob storage trigger starts a function when a new or updated blob is detected. The blob contents are provided as input to the function.

The [Event Grid trigger](functions-bindings-event-grid.md) has built-in support for [blob events](../storage/blobs/storage-blob-event-overview.md) and can also be used to start a function when a new or updated blob is detected. For an example, see the [Image resize with Event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md) tutorial.

Use Event Grid instead of the Blob storage trigger for the following scenarios:

* Blob storage accounts
* High scale
* Minimizing latency

### Blob storage accounts

[Blob storage accounts](../storage/common/storage-account-overview.md#types-of-storage-accounts) are supported for blob input and output bindings but not for blob triggers. Blob storage triggers require a general-purpose storage account.

### High scale

High scale can be loosely defined as containers that have more than 100,000 blobs in them or storage accounts that have more than 100 blob updates per second.

### Latency issues

If your function app is on the Consumption plan, there can be up to a 10-minute delay in processing new blobs if a function app has gone idle. To avoid this latency, you can switch to an App Service plan with Always On enabled. You can also use an [Event Grid trigger](functions-bindings-event-grid.md) with your Blob storage account. For an example, see the [Event Grid tutorial](../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2Fazure%2Fazure-functions%2Ftoc.json). 

### Queue storage trigger

Besides Event Grid, another alternative for processing blobs is the Queue storage trigger, but it has no built-in support for blob events. You would have to create queue messages when creating or updating blobs. For an example that assumes you've done that, see the [blob input binding example later in this article](#input---example).

## Trigger - example

# [C#](#tab/csharp)

The following example shows a [C# function](functions-dotnet-class-library.md) that writes a log when a blob is added or updated in the `samples-workitems` container.

```csharp
[FunctionName("BlobTriggerCSharp")]        
public static void Run([BlobTrigger("samples-workitems/{name}")] Stream myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

The string `{name}` in the blob trigger path `samples-workitems/{name}` creates a [binding expression](./functions-bindings-expressions-patterns.md) that you can use in function code to access the file name of the triggering blob. For more information, see [Blob name patterns](#trigger---blob-name-patterns) later in this article.

For more information about the `BlobTrigger` attribute, see [Trigger - attributes](#trigger---attributes).

# [C# Script](#tab/csharp-script)

The following example shows a blob trigger binding in a *function.json* file and code that uses the binding. The function writes a log when a blob is added or updated in the `samples-workitems` [container](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources).

Here's the binding data in the *function.json* file:

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

The string `{name}` in the blob trigger path `samples-workitems/{name}` creates a [binding expression](./functions-bindings-expressions-patterns.md) that you can use in function code to access the file name of the triggering blob. For more information, see [Blob name patterns](#trigger---blob-name-patterns) later in this article.

For more information about *function.json* file properties, see the [Configuration](#trigger---configuration) section explains these properties.

Here's C# script code that binds to a `Stream`:

```cs
public static void Run(Stream myBlob, string name, ILogger log)
{
   log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

Here's C# script code that binds to a `CloudBlockBlob`:

```cs
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.WindowsAzure.Storage.Blob;

public static void Run(CloudBlockBlob myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name}\nURI:{myBlob.StorageUri}");
}
```

# [JavaScript](#tab/javascript)

The following example shows a blob trigger binding in a *function.json* file and [JavaScript code](functions-reference-node.md) that uses the binding. The function writes a log when a blob is added or updated in the `samples-workitems` container.

Here's the *function.json* file:

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

The string `{name}` in the blob trigger path `samples-workitems/{name}` creates a [binding expression](./functions-bindings-expressions-patterns.md) that you can use in function code to access the file name of the triggering blob. For more information, see [Blob name patterns](#trigger---blob-name-patterns) later in this article.

For more information about *function.json* file properties, see the [Configuration](#trigger---configuration) section explains these properties.

Here's the JavaScript code:

```javascript
module.exports = function(context) {
    context.log('Node.js Blob trigger function processed', context.bindings.myBlob);
    context.done();
};
```

# [Python](#tab/python)

The following example shows a blob trigger binding in a *function.json* file and [Python code](functions-reference-python.md) that uses the binding. The function writes a log when a blob is added or updated in the `samples-workitems` [container](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources).

Here's the *function.json* file:

```json
{
    "scriptFile": "__init__.py",
    "disabled": false,
    "bindings": [
        {
            "name": "myblob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

The string `{name}` in the blob trigger path `samples-workitems/{name}` creates a [binding expression](./functions-bindings-expressions-patterns.md) that you can use in function code to access the file name of the triggering blob. For more information, see [Blob name patterns](#trigger---blob-name-patterns) later in this article.

For more information about *function.json* file properties, see the [Configuration](#trigger---configuration) section explains these properties.

Here's the Python code:

```python
import logging
import azure.functions as func


def main(myblob: func.InputStream):
    logging.info('Python Blob trigger function processed %s', myblob.name)
```

# [Java](#tab/java)

The following example shows a blob trigger binding in a *function.json* file and [Java code](functions-reference-java.md) that uses the binding. The function writes a log when a blob is added or updated in the `myblob` container.

Here's the *function.json* file:

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "file",
            "type": "blobTrigger",
            "direction": "in",
            "path": "myblob/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

Here's the Java code:

```java
@FunctionName("blobprocessor")
public void run(
  @BlobTrigger(name = "file",
               dataType = "binary",
               path = "myblob/{name}",
               connection = "MyStorageAccountAppSetting") byte[] content,
  @BindingName("name") String filename,
  final ExecutionContext context
) {
  context.getLogger().info("Name: " + filename + " Size: " + content.length + " bytes");
}
```

---

## Trigger - attributes

# [C#](#tab/csharp)

In [C# class libraries](functions-dotnet-class-library.md), use the following attributes to configure a blob trigger:

* [BlobTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobTriggerAttribute.cs)

  The attribute's constructor takes a path string that indicates the container to watch and optionally a [blob name pattern](#trigger---blob-name-patterns). Here's an example:

  ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}")] Stream image,
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
  ```

  You can set the `Connection` property to specify the storage account to use, as shown in the following example:

   ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}", Connection = "StorageConnectionAppSetting")] Stream image,
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
   ```

  For a complete example, see [Trigger example](#trigger---example).

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  Provides another way to specify the storage account to use. The constructor takes the name of an app setting that contains a storage connection string. The attribute can be applied at the parameter, method, or class level. The following example shows class level and method level:

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("BlobTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ....
  }
  ```

The storage account to use is determined in the following order:

* The `BlobTrigger` attribute's `Connection` property.
* The `StorageAccount` attribute applied to the same parameter as the `BlobTrigger` attribute.
* The `StorageAccount` attribute applied to the function.
* The `StorageAccount` attribute applied to the class.
* The default storage account for the function app ("AzureWebJobsStorage" app setting).

# [C# Script](#tab/csharp-script)

Attributes are not supported by C# Script.

# [JavaScript](#tab/javascript)

Attributes are not supported by JavaScript.

# [Python](#tab/python)

Attributes are not supported by Python.

# [Java](#tab/java)

The `@BlobTrigger` attribute is used to give you access to the blob that triggered the function. Refer to the [trigger example](#trigger---example) for details.

---

## Trigger - configuration

The following table explains the binding configuration properties that you set in the *function.json* file and the `BlobTrigger` attribute.

|function.json property | Attribute property |Description|
|---------|---------|----------------------|
|**type** | n/a | Must be set to `blobTrigger`. This property is set automatically when you create the trigger in the Azure portal.|
|**direction** | n/a | Must be set to `in`. This property is set automatically when you create the trigger in the Azure portal. Exceptions are noted in the [usage](#trigger---usage) section. |
|**name** | n/a | The name of the variable that represents the blob in function code. |
|**path** | **BlobPath** |The [container](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources) to monitor.  May be a [blob name pattern](#trigger---blob-name-patterns). |
|**connection** | **Connection** | The name of an app setting that contains the Storage connection string to use for this binding. If the app setting name begins with "AzureWebJobs", you can specify only the remainder of the name here. For example, if you set `connection` to "MyStorage", the Functions runtime looks for an app setting that is named "MyStorage." If you leave `connection` empty, the Functions runtime uses the default Storage connection string in the app setting that is named `AzureWebJobsStorage`.<br><br>The connection string must be for a general-purpose storage account, not a [Blob storage account](../storage/common/storage-account-overview.md#types-of-storage-accounts).|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## Trigger - usage

# [C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-trigger.md)]

# [C# Script](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-trigger.md)]

# [JavaScript](#tab/javascript)

Access blob data using `context.bindings.<NAME>` where `<NAME>` matches the value defined in *function.json*.

# [Python](#tab/python)

Access blob data via the parameter typed as [InputStream](https://docs.microsoft.com/python/api/azure-functions/azure.functions.inputstream?view=azure-python). Refer to the [trigger example](#trigger---example) for details.

# [Java](#tab/java)

The `@BlobTrigger` attribute is used to give you access to the blob that triggered the function. Refer to the [trigger example](#trigger---example) for details.

---

## Trigger - blob name patterns

You can specify a blob name pattern in the `path` property in *function.json* or in the `BlobTrigger` attribute constructor. The name pattern can be a [filter or binding expression](./functions-bindings-expressions-patterns.md). The following sections provide examples.

### Get file name and extension

The following example shows how to bind to the blob file name and extension separately:

```json
"path": "input/{blobname}.{blobextension}",
```

If the blob is named *original-Blob1.txt*, the values of the `blobname` and `blobextension` variables in function code are *original-Blob1* and *txt*.

### Filter on blob name

The following example triggers only on blobs in the `input` container that start with the string "original-":

```json
"path": "input/original-{name}",
```

If the blob name is *original-Blob1.txt*, the value of the `name` variable in function code is `Blob1`.

### Filter on file type

The following example triggers only on *.png* files:

```json
"path": "samples/{name}.png",
```

### Filter on curly braces in file names

To look for curly braces in file names, escape the braces by using two braces. The following example filters for blobs that have curly braces in the name:

```json
"path": "images/{{20140101}}-{name}",
```

If the blob is named *{20140101}-soundfile.mp3*, the `name` variable value in the function code is *soundfile.mp3*.

## Trigger - metadata

# [C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-metadata.md)]

# [C# Script](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-metadata.md)]

# [JavaScript](#tab/javascript)

```javascript
module.exports = function (context, myBlob) {
    context.log("Full blob path:", context.bindingData.blobTrigger);
    context.done();
};
```

# [Python](#tab/python)

Metadata is not available in Python.

# [Java](#tab/java)

Metadata is not available in Java.

---

## Trigger - blob receipts

The Azure Functions runtime ensures that no blob trigger function gets called more than once for the same new or updated blob. To determine if a given blob version has been processed, it maintains *blob receipts*.

Azure Functions stores blob receipts in a container named *azure-webjobs-hosts* in the Azure storage account for your function app (defined by the app setting `AzureWebJobsStorage`). A blob receipt has the following information:

* The triggered function ("*&lt;function app name>*.Functions.*&lt;function name>*", for example: "MyFunctionApp.Functions.CopyBlob")
* The container name
* The blob type ("BlockBlob" or "PageBlob")
* The blob name
* The ETag (a blob version identifier, for example: "0x8D1DC6E70A277EF")

To force reprocessing of a blob, delete the blob receipt for that blob from the *azure-webjobs-hosts* container manually. While reprocessing might not occur immediately, it's guaranteed to occur at a later point in time.

## Trigger - poison blobs

When a blob trigger function fails for a given blob, Azure Functions retries that function a total of 5 times by default.

If all 5 tries fail, Azure Functions adds a message to a Storage queue named *webjobs-blobtrigger-poison*. The queue message for poison blobs is a JSON object that contains the following properties:

* FunctionId (in the format *&lt;function app name>*.Functions.*&lt;function name>*)
* BlobType ("BlockBlob" or "PageBlob")
* ContainerName
* BlobName
* ETag (a blob version identifier, for example: "0x8D1DC6E70A277EF")

## Trigger - concurrency and memory usage

The blob trigger uses a queue internally, so the maximum number of concurrent function invocations is controlled by the [queues configuration in host.json](functions-host-json.md#queues). The default settings limit concurrency to 24 invocations. This limit applies separately to each function that uses a blob trigger.

[The consumption plan](functions-scale.md#how-the-consumption-and-premium-plans-work) limits a function app on one virtual machine (VM) to 1.5 GB of memory. Memory is used by each concurrently executing function instance and by the Functions runtime itself. If a blob-triggered function loads the entire blob into memory, the maximum memory used by that function just for blobs is 24 * maximum blob size. For example, a function app with three blob-triggered functions and the default settings would have a maximum per-VM concurrency of 3*24 = 72 function invocations.

JavaScript and Java functions load the entire blob into memory, and C# functions do that if you bind to `string`, `Byte[]`, or POCO.

## Trigger - polling

Polling works as a hybrid between inspecting logs and running periodic container scans. Blobs are scanned in groups of 10,000 at a time with a continuation token used between intervals.

> [!WARNING]
> In addition, [storage logs are created on a "best effort"](/rest/api/storageservices/About-Storage-Analytics-Logging) basis. There's no guarantee that all events are captured. Under some conditions, logs may be missed.
> 
> If you require faster or more reliable blob processing, consider creating a [queue message](../storage/queues/storage-dotnet-how-to-use-queues.md) when you create the blob. Then use a [queue trigger](functions-bindings-storage-queue.md) instead of a blob trigger to process the blob. Another option is to use Event Grid; see the tutorial [Automate resizing uploaded images using Event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md).
>

## Input

Use a Blob storage input binding to read blobs.

## Input - example

# [C#](#tab/csharp)

The following example is a [C# function](functions-dotnet-class-library.md) that uses a queue trigger and an input blob binding. The queue message contains the name of the blob, and the function logs the size of the blob.

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

# [C# Script](#tab/csharp-script)

<!--Same example for input and output. -->

The following example shows blob input and output bindings in a *function.json* file and [C# script (.csx)](functions-reference-csharp.md) code that uses the bindings. The function makes a copy of a text blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

The [configuration](#input---configuration) section explains these properties.

Here's the C# script code:

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

# [JavaScript](#tab/javascript)

<!--Same example for input and output. -->

The following example shows blob input and output bindings in a *function.json* file and [JavaScript code](functions-reference-node.md) that uses the bindings. The function makes a copy of a blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

The [configuration](#input---configuration) section explains these properties.

Here's the JavaScript code:

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

# [Python](#tab/python)

<!--Same example for input and output. -->

The following example shows blob input and output bindings in a *function.json* file and [Python code](functions-reference-python.md) that uses the bindings. The function makes a copy of a blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "queuemsg",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "inputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "$return",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

The [configuration](#input---configuration) section explains these properties.

Here's the Python code:

```python
import logging
import azure.functions as func


def main(queuemsg: func.QueueMessage, inputblob: func.InputStream) -> func.InputStream:
    logging.info('Python Queue trigger function processed %s', inputblob.name)
    return inputblob
```

# [Java](#tab/java)

This section contains the following examples:

* [HTTP trigger, look up blob name from query string](#http-trigger-look-up-blob-name-from-query-string)
* [Queue trigger, receive blob name from queue message](#queue-trigger-receive-blob-name-from-queue-message)

#### HTTP trigger, look up blob name from query string

 The following example shows a Java function that uses the `HttpTrigger` annotation to receive a parameter containing the name of a file in a blob storage container. The `BlobInput` annotation then reads the file and passes its contents to the function as a `byte[]`.

```java
  @FunctionName("getBlobSizeHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage blobSize(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    final ExecutionContext context) {
      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### Queue trigger, receive blob name from queue message

 The following example shows a Java function that uses the `QueueTrigger` annotation to receive a message containing the name of a file in a blob storage container. The `BlobInput` annotation then reads the file and passes its contents to the function as a `byte[]`.

```java
  @FunctionName("getBlobSize")
  @StorageAccount("Storage_Account_Connection_String")
  public void blobSize(
    @QueueTrigger(
      name = "filename", 
      queueName = "myqueue-items-sample") 
    String filename,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{queueTrigger}") 
    byte[] content,
    final ExecutionContext context) {
      context.getLogger().info("The size of \"" + filename + "\" is: " + content.length + " bytes");
  }
```

In the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the `@BlobInput` annotation on parameters whose value would come from a blob.  This annotation can be used with native Java types, POJOs, or nullable values using `Optional<T>`.

---

## Input - attributes

# [C#](#tab/csharp)

In [C# class libraries](functions-dotnet-class-library.md), use the [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs).

The attribute's constructor takes the path to the blob and a `FileAccess` parameter indicating read or write, as shown in the following example:

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}

```

You can set the `Connection` property to specify the storage account to use, as shown in the following example:

```csharp
[FunctionName("BlobInput")]
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read, Connection = "StorageConnectionAppSetting")] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

You can use the `StorageAccount` attribute to specify the storage account at class, method, or parameter level. For more information, see [Trigger - attributes](#trigger---attributes).

# [C# Script](#tab/csharp-script)

Attributes are not supported by C# Script.

# [JavaScript](#tab/javascript)

Attributes are not supported by JavaScript.

# [Python](#tab/python)

Attributes are not supported by Python.

# [Java](#tab/java)

The `@BlobInput` attribute gives you access to the blob that triggered the function. If you use a byte array with the attribute, set `dataType` to `binary`. Refer to the [input example](#input---example) for details.

---

## Input - configuration

The following table explains the binding configuration properties that you set in the *function.json* file and the `Blob` attribute.

|function.json property | Attribute property |Description|
|---------|---------|----------------------|
|**type** | n/a | Must be set to `blob`. |
|**direction** | n/a | Must be set to `in`. Exceptions are noted in the [usage](#input---usage) section. |
|**name** | n/a | The name of the variable that represents the blob in function code.|
|**path** |**BlobPath** | The path to the blob. |
|**connection** |**Connection**| The name of an app setting that contains the [Storage connection string](../storage/common/storage-configure-connection-string.md) to use for this binding. If the app setting name begins with "AzureWebJobs", you can specify only the remainder of the name here. For example, if you set `connection` to "MyStorage", the Functions runtime looks for an app setting that is named "MyStorage." If you leave `connection` empty, the Functions runtime uses the default Storage connection string in the app setting that is named `AzureWebJobsStorage`.<br><br>The connection string must be for a general-purpose storage account, not a [blob-only storage account](../storage/common/storage-account-overview.md#types-of-storage-accounts).|
|n/a | **Access** | Indicates whether you will be reading or writing. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## Input - usage

# [C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-input-usage.md](../../includes/functions-bindings-blob-storage-input-usage.md)]

# [C# Script](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-input-usage.md](../../includes/functions-bindings-blob-storage-input-usage.md)]

# [JavaScript](#tab/javascript)

Access blob data using `context.bindings.<NAME>` where `<NAME>` matches the value defined in *function.json*.

# [Python](#tab/python)

Access blob data via the parameter typed as [InputStream](https://docs.microsoft.com/python/api/azure-functions/azure.functions.inputstream?view=azure-python). Refer to the [input example](#input---example) for details.

# [Java](#tab/java)

The `@BlobInput` attribute gives you access to the blob that triggered the function. If you use a byte array with the attribute, set `dataType` to `binary`. Refer to the [input example](#input---example) for details.

---

## Output

Use Blob storage output bindings to write blobs.

## Output - example

# [C#](#tab/csharp)

The following example is a [C# function](functions-dotnet-class-library.md) that uses a blob trigger and two output blob bindings. The function is triggered by the creation of an image blob in the *sample-images* container. It creates small and medium size copies of the image blob.

```csharp
using System.Collections.Generic;
using System.IO;
using Microsoft.Azure.WebJobs;
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Formats;
using SixLabors.ImageSharp.PixelFormats;
using SixLabors.ImageSharp.Processing;

public class ResizeImages
{
    [FunctionName("ResizeImage")]
    public static void Run([BlobTrigger("sample-images/{name}")] Stream image,
        [Blob("sample-images-sm/{name}", FileAccess.Write)] Stream imageSmall,
        [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageMedium)
    {
        IImageFormat format;

        using (Image<Rgba32> input = Image.Load<Rgba32>(image, out format))
        {
            ResizeImage(input, imageSmall, ImageSize.Small, format);
        }

        image.Position = 0;
        using (Image<Rgba32> input = Image.Load<Rgba32>(image, out format))
        {
            ResizeImage(input, imageMedium, ImageSize.Medium, format);
        }
    }

    public static void ResizeImage(Image<Rgba32> input, Stream output, ImageSize size, IImageFormat format)
    {
        var dimensions = imageDimensionsTable[size];

        input.Mutate(x => x.Resize(dimensions.Item1, dimensions.Item2));
        input.Save(output, format);
    }

    public enum ImageSize { ExtraSmall, Small, Medium }

    private static Dictionary<ImageSize, (int, int)> imageDimensionsTable = new Dictionary<ImageSize, (int, int)>() {
        { ImageSize.ExtraSmall, (320, 200) },
        { ImageSize.Small,      (640, 400) },
        { ImageSize.Medium,     (800, 600) }
    };

}
```

# [C# Script](#tab/csharp-script)

<!--Same example for input and output. -->

The following example shows blob input and output bindings in a *function.json* file and [C# script (.csx)](functions-reference-csharp.md) code that uses the bindings. The function makes a copy of a text blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

The [configuration](#output---configuration) section explains these properties.

Here's the C# script code:

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

# [JavaScript](#tab/javascript)

<!--Same example for input and output. -->

The following example shows blob input and output bindings in a *function.json* file and [JavaScript code](functions-reference-node.md) that uses the bindings. The function makes a copy of a blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

The [configuration](#output---configuration) section explains these properties.

Here's the JavaScript code:

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

# [Python](#tab/python)

<!--Same example for input and output. -->

The following example shows blob input and output bindings in a *function.json* file and [Python code](functions-reference-python.md) that uses the bindings. The function makes a copy of a blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "queuemsg",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "inputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "outputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

The [configuration](#output---configuration) section explains these properties.

Here's the Python code:

```python
import logging
import azure.functions as func


def main(queuemsg: func.QueueMessage, inputblob: func.InputStream,
         outputblob: func.Out[func.InputStream]):
    logging.info('Python Queue trigger function processed %s', inputblob.name)
    outputblob.set(inputblob)
```

# [Java](#tab/java)

This section contains the following examples:

* [HTTP trigger, using OutputBinding](#http-trigger-using-outputbinding-java)
* [Queue trigger, using function return value](#queue-trigger-using-function-return-value-java)

#### HTTP trigger, using OutputBinding (Java)

 The following example shows a Java function that uses the `HttpTrigger` annotation to receive a parameter containing the name of a file in a blob storage container. The `BlobInput` annotation then reads the file and passes its contents to the function as a `byte[]`. The `BlobOutput` annotation binds to `OutputBinding outputItem`, which is then used by the function to write the contents of the input blob to the configured storage container.

```java
  @FunctionName("copyBlobHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage copyBlobHttp(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    @BlobOutput(
      name = "target", 
      path = "myblob/{Query.file}-CopyViaHttp")
    OutputBinding<String> outputItem,
    final ExecutionContext context) {
      // Save blob to outputItem
      outputItem.setValue(new String(content, StandardCharsets.UTF_8));

      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### Queue trigger, using function return value (Java)

 The following example shows a Java function that uses the `QueueTrigger` annotation to receive a message containing the name of a file in a blob storage container. The `BlobInput` annotation then reads the file and passes its contents to the function as a `byte[]`. The `BlobOutput` annotation binds to the function return value, which is then used by the runtime to write the contents of the input blob to the configured storage container.

```java
  @FunctionName("copyBlobQueueTrigger")
  @StorageAccount("Storage_Account_Connection_String")
  @BlobOutput(
    name = "target", 
    path = "myblob/{queueTrigger}-Copy")
  public String copyBlobQueue(
    @QueueTrigger(
      name = "filename", 
      dataType = "string",
      queueName = "myqueue-items") 
    String filename,
    @BlobInput(
      name = "file", 
      path = "samples-workitems/{queueTrigger}") 
    String content,
    final ExecutionContext context) {
      context.getLogger().info("The content of \"" + filename + "\" is: " + content);
      return content;
  }
```

 In the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the `@BlobOutput` annotation on function parameters whose value would be written to an object in blob storage.  The parameter type should be `OutputBinding<T>`, where T is any native Java type or a POJO.

---

## Output - attributes

# [C#](#tab/csharp)

In [C# class libraries](functions-dotnet-class-library.md), use the [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs).

The attribute's constructor takes the path to the blob and a `FileAccess` parameter indicating read or write, as shown in the following example:

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
{
    ...
}
```

You can set the `Connection` property to specify the storage account to use, as shown in the following example:

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write, Connection = "StorageConnectionAppSetting")] Stream imageSmall)
{
    ...
}
```

# [C# Script](#tab/csharp-script)

Attributes are not supported by C# Script.

# [JavaScript](#tab/javascript)

Attributes are not supported by JavaScript.

# [Python](#tab/python)

Attributes are not supported by Python.

# [Java](#tab/java)

The `@BlobOutput` attribute gives you access to the blob that triggered the function. If you use a byte array with the attribute, set `dataType` to `binary`. Refer to the [output example](#output---example) for details.

---

For a complete example, see [Output example](#output---example).

You can use the `StorageAccount` attribute to specify the storage account at class, method, or parameter level. For more information, see [Trigger - attributes](#trigger---attributes).

## Output - configuration

The following table explains the binding configuration properties that you set in the *function.json* file and the `Blob` attribute.

|function.json property | Attribute property |Description|
|---------|---------|----------------------|
|**type** | n/a | Must be set to `blob`. |
|**direction** | n/a | Must be set to `out` for an output binding. Exceptions are noted in the [usage](#output---usage) section. |
|**name** | n/a | The name of the variable that represents the blob in function code.  Set to `$return` to reference the function return value.|
|**path** |**BlobPath** | The path to the blob container. |
|**connection** |**Connection**| The name of an app setting that contains the Storage connection string to use for this binding. If the app setting name begins with "AzureWebJobs", you can specify only the remainder of the name here. For example, if you set `connection` to "MyStorage", the Functions runtime looks for an app setting that is named "MyStorage." If you leave `connection` empty, the Functions runtime uses the default Storage connection string in the app setting that is named `AzureWebJobsStorage`.<br><br>The connection string must be for a general-purpose storage account, not a [blob-only storage account](../storage/common/storage-account-overview.md#types-of-storage-accounts).|
|n/a | **Access** | Indicates whether you will be reading or writing. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## Output - usage

# [C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-output-usage.md](../../includes/functions-bindings-blob-storage-output-usage.md)]

# [C# Script](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-output-usage.md](../../includes/functions-bindings-blob-storage-output-usage.md)]

# [JavaScript](#tab/javascript)

In JavaScript, access the blob data using `context.bindings.<name from function.json>`.

# [Python](#tab/python)

You can declare function parameters as the following types to write out to blob storage:

* Strings as `func.Out(str)`
* Streams as `func.Out(func.InputStream)`

Refer to the [output example](#output---example) for details.

# [Java](#tab/java)

The `@BlobOutput` attribute gives you access to the blob that triggered the function. If you use a byte array with the attribute, set `dataType` to `binary`. Refer to the [output example](#output---example) for details.

---

## Exceptions and return codes

| Binding |  Reference |
|---|---|
| Blob | [Blob Error Codes](https://docs.microsoft.com/rest/api/storageservices/fileservices/blob-service-error-codes) |
| Blob, Table, Queue |  [Storage Error Codes](https://docs.microsoft.com/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Blob, Table, Queue |  [Troubleshooting](https://docs.microsoft.com/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## Next steps

* [Learn more about Azure functions triggers and bindings](functions-triggers-bindings.md)

<!---
> [!div class="nextstepaction"]
> [Go to a quickstart that uses a Blob storage trigger](functions-create-storage-blob-triggered-function.md)
--->
