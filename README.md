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

To create an instance of the class, you need to provide a [project ID](https://developer.kenticocloud.com/docs/using-delivery-api#section-getting-project-id) and a valid [Content Management API Key](https://developer.kenticocloud.com/v1/docs/importing-to-kentico-cloud#importing-content-items).

```csharp
var OPTIONS = new ContentManagementOptions() { ProjectId = "bb6882a0-3088-405c-a6ac-4a0da46810b0", ApiKey = "ew0...1eo" }; 
// Initialize an instance of the ContentManagementClient client
var client = new ContentManagementClient(OPTIONS);
```

Once you create a `ContentManagementClient`, you can start managing content in your project by calling methods on the client instance. See [Basic content item import](#basic-content-item-import) for details.

### Importing content items

Importing content items is a 2 step process, using 2 separate methods:

1. Creating an empty content item which serves as a wrapper for your content.
2. Adding content inside a language variant of the content item.

Each content item can consist of several localized variants. **The content itself is always part of a specific language variant, even if your project only uses one language**. See our [Importing to Kentico Cloud](https://developer.kenticocloud.com/v1/docs/importing-to-kentico-cloud#section-importing-your-content) tutorial for a more detailed explanation. 

#### 1. Creating a content item

```csharp
// Create an instance of the Content Management client
var client = new ContentManagementClient(OPTIONS);

// Define a content type of the imported item by its codename
var contentType = new ManageApiReference() { CodeName = "cafe" };
// Define the imported content item
var item = new ContentItemPostModel() { Name = "Brno", Type = contentType };

// Add your content item to your project in Kentico Cloud
var responseItem = await client.AddContentItemAsync(item);
);
```

Kentico Cloud will generate an internal ID and codename for the (new and empty) content item and include it in the response. In the next step, we will add the actual (localized) content.


#### 2. Adding language variants

To add localized content, you have to specify: 

* The content item you are importing into.
* The language variant of the content item.
* The content elements of the language variant you want to insert or update. Omitted elements will remain unchanged. 

```csharp
var client = new ContentManagementClient(OPTIONS);

private static Dictionary<string, object> ELEMENTS = new Dictionary<string, object> {
    { "street", "Nove Sady 25" },
    { "city", "Brno" },
    { "country", "Czech Republic" },
    { "state", "Jihomoravsky kraj" },
    { "zip_code", "60200" },
    { "phone", "+420 444 444 444" },
    { "email", "brnocafe@kentico.com" }
};
var contentItemVariantUpdateModel = new ContentItemVariantUpdateModel() { Elements = ELEMENTS };

// Specify the content item and the language varaint 
var itemIdentifier = ContentItemIdentifier.ByCodename("brno");
var variantIdentifier = ContentVariantIdentifier.ByLanguageCodename("en-US");
var identifier = new ContentItemVariantIdentifier(itemIdentifier, variantIdentifier);

// Upsert a language variant of your content item
var responseVariant = await client.UpsertVariantAsync(identifier, contentItemVariantUpdateModel);
);
```

TO-DO: Example of importing other types of elements besides text (Assets, Modular content, Taxonomy, Numbers...)

### Importing assets

Importing assets using Content Management SDK is a 3-step process:

1. Upload a file to Kentico Cloud.
2. Create a new asset using the given file reference.
3. Link to the asset from a language variant of a content item. 

#### 1. Upload a file 

```csharp
var client = new ContentManagementClient(OPTIONS);

var stream = new MemoryStream(Encoding.UTF8.GetBytes("Hello world from CM API .NET SDK tests!"));
var fileName = "Hello.txt";
var contentType = "text/plain";

var fileResult = await client.UploadFile(stream, fileName, contentType);
```
Kentico Cloud will generateand internal id that serves as pointer to your file. You will use it in the next step to create the actual asset. 

#### 2. Create an asset 
```csharp
var asset = new AssetUpsertModel
    {
        FileReference = fileResult,
        Descriptions = new List<AssetDescriptionsModel>()
    };
var externalId = "Hello";

var assetResult = await client.UpsertAssetByExternalId(externalId, asset);
```

TO-DO: How to import asset descriptions
TBD: Will there be no method for just creating an asset? 

#### 3. Use the asset in a language variant 

TO-DO

### Importing modular and linked content

TO-DO





