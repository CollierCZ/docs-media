# Kentico Cloud Content Management .NET SDK

[![Build status](https://ci.appveyor.com/api/projects/status/3m3q2ads2y43bh9o/branch/master?svg=true)](https://ci.appveyor.com/project/kentico/content-management-sdk-net/branch/master)
[![NuGet](https://img.shields.io/nuget/v/KenticoCloud.Delivery.svg)](https://www.nuget.org/packages/KenticoCloud.ContentManagement)
[![NuGet](https://img.shields.io/nuget/dt/kenticocloud.delivery.svg)](https://www.nuget.org/packages/KenticoCloud.ContentManagement)
[![Forums](https://img.shields.io/badge/chat-on%20forums-orange.svg)](https://forums.kenticocloud.com)

## Summary

The Kentico Cloud Content Management .NET SDK is a client library used for managing content in Kentico Cloud. It provides read/write access to your Kentico Cloud projects.  
You can use the SDK in the form of a [NuGet package](https://www.nuget.org/packages/KenticoCloud.Delivery) to migrate your existing content into your Kentico Cloud project, or update content in your content items. 

The Content Management SDK does not provide any content filtering options and is not optimized for content delivery. If you need to deliver larger amounts of content we recommend using the [Delivery SDK](https://github.com/Kentico/delivery-sdk-net) instead.

## Prerequisites

To manage content in a Kentico Cloud project via the Content Management API, you first need to activate the API for the project. See our documentation on how you can [activate the Content Management API](https://developer.kenticocloud.com/v1/docs/importing-to-kentico-cloud#section-enabling-the-api-for-your-project).

## Using the ContentManagementClient

The `ContentManagementClient` class is the main class of the SDK. Using this class, you can import, update, view and delete content in your Kentico Cloud projects. 

To create an instance of the class, you need to provide a [project ID](https://developer.kenticocloud.com/docs/using-delivery-api#section-getting-project-id) and a valid [Content Management API Key](https://developer.kenticocloud.com/v1/docs/importing-to-kentico-cloud#section-enabling-the-api-for-your-project).

```csharp
private static ContentManagementOptions OPTIONS = new ContentManagementOptions() { ApiKey = "ew0...1eo", ProjectId = "bb6882a0-3088-405c-a6ac-4a0da46810b0" };
// Initializes an instance of the DeliveryClient client
var client = new ContentManagementClient(OPTIONS);
```

You can also provide the project ID and other parameters by passing the [`DeliveryOptions`](https://github.com/Kentico/delivery-sdk-net/blob/master/KenticoCloud.Delivery/Configuration/DeliveryOptions%20.cs) object to the class constructor. The `DeliveryOptions` object can be used to set the following parameters:

* `PreviewApiKey` – sets the Delivery Preview API key.
* `ProjectId` – sets the project identifier.
* `UsePreviewApi` – determines whether to use the Delivery Preview API.
* `WaitForLoadingNewContent` – makes the client instance wait while fetching updated content, useful when acting upon [webhook calls](https://developer.kenticocloud.com/docs/webhooks#section-requesting-new-content).

For advanced configuration options using Dependency Injection and ASP.NET Core Configuration API, see the SDK's [wiki](https://github.com/Kentico/delivery-sdk-net/wiki/Using-the-ASP.NET-Core-Configuration-API-and-DI-to-Instantiate-the-DeliveryClient).

Once you create a `DeliveryClient`, you can start querying your project repository by calling methods on the client instance. See [Basic querying](#basic-querying) for details.

### Filtering retrieved data

The SDK supports full scale of the API querying and filtering capabilities as described in the [API reference](https://developer.kenticocloud.com/reference#filtering-content-items).

```csharp
// Retrieves a list of the specified elements from the first 10 content items of
// the 'brewer' content type, ordered by the 'product_name' element value
DeliveryItemListingResponse response = await client.GetItemsAsync(
    new EqualsFilter("system.type", "brewer"),
    new ElementsParameter("image", "price", "product_status", "processing"),
    new LimitParameter(10),
    new OrderParameter("elements.product_name")
);
```

### Previewing unpublished content

To retrieve unpublished content, you need to create a `DeliveryClient` with both Project ID and Preview API key. Each Kentico Cloud project has its own Preview API key. 

```csharp
// Note: Within a single project, we recommend that you work with only
// either the production or preview Delivery API, not both.
DeliveryClient client = new DeliveryClient("YOUR_PROJECT_ID", "YOUR_PREVIEW_API_KEY");
```

For more details, see [Previewing unpublished content using the Delivery API](https://developer.kenticocloud.com/docs/preview-content-via-api).

