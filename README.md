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
var item = new ContentItemCreateModel() { Name = "Brno", Type = contentType };

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
var languageIdentifier = LanguageIdentifier.ByLanguageCodename("en-US");
var identifier = new ContentItemVariantIdentifier(itemIdentifier, languageIdentifier);

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

var stream = new MemoryStream(Encoding.UTF8.GetBytes("Hello world from CM API .NET SDK"));
var fileName = "Hello.txt";
var contentType = "text/plain";

var fileResult = await client.UploadFileAsync(stream, fileName, contentType);
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

var assetResult = await client.AddAssetAsync(asset);
```

TO-DO: How to import asset descriptions
TBD: Do we use the basic AddAsset method in the introductiom, or do we push external IDs right away?

#### 3. Use the asset in a language variant 

TO-DO

### Importing modular and linked content

TO-DO

### Viewing a content item

```csharp
// You can also specify the content item by its codename or external ID
// var identifier = ContentItemIdentifier.ByCodename(EXISTING_ITEM_CODENAME);
// var identifier = ContentItemIdentifier.ByExternalId(EXTERNAL_ID);
var identifier = ContentItemIdentifier.ById(EXISTING_ITEM_ID);

var contentItemReponse = await _client.GetContentItemAsync(identifier);
```

### Deleting a content item

```csharp
var itemToDelete = await PrepareItemToDelete();

// You can also specify the content item by its codename or external ID
// var identifier = ContentItemIdentifier.ByCodename(itemToDelete.CodeName);
// var identifier = ContentItemIdentifier.ByExternalId(itemToDelete.ExternalId);
var identifier = ContentItemIdentifier.ById(itemToDelete.Id);

client.DeleteContentItemAsync(identifier);
```

### Updating a content item

```csharp
// You can also specify the content item by its codename
// var identifier = ContentItemIdentifier.ByCodename(EXISTING_ITEM_CODENAME);
var identifier = ContentItemIdentifier.ById(EXISTING_ITEM_ID);
var newSitemapLocations = new List<ManageApiReference>();

var item = new ContentItemUpdateModel() { Name = "New name", SitemapLocations = newSitemapLocations };

var contentItemReponse = await _client.UpdateContentItemAsync(identifier, item);
```

### Upserting a content item by external ID

```csharp
var sitemapLocations = new List<ManageApiReference>();
var type = new ManageApiReference() { CodeName = "cafe" };
var item = new ContentItemUpsertModel() { Name = "New or updated name", SitemapLocations = sitemapLocations, Type = type };

var contentItemResponse = await _client.UpsertContentItemByExternalIdAsync("EXTERNAL_ID", item);
```

### Listing content items

All at once:
```csharp
 var response = await client.ListContentItemsAsync();
 ```

With continuation:
```csharp
var response = await client.ListContentItemsAsync();
while (true)
{
    foreach (var item in response)
    {
        // use your content item
    }

    if (!response.HasNextPage())
    {
        break;
    }
    response = await response.GetNextPage();
}
 ```
 
 
### Upserting language variants

```csharp
 var contentItemVariantUpdateModel = new ContentItemVariantUpdateModel() { Elements = {
    { "street", "Nove Sady 25" },
    { "city", "Brno" },
    { "country", "Czech Republic" }
} };

var itemIdentifier = ContentItemIdentifier.ByCodename(EXISTING_ITEM_CODENAME);
// var itemIdentifier = ContentItemIdentifier.ById(EXISTING_ITEM_ID);
// var itemIdentifier = ContentItemIdentifier.ByExternalId(EXTERNAL_ID);


var languageIdentifier = LanguageIdentifier.ByCodename(EXISTING_LANGUAGE_CODENAME);
// var languageIdentifier = LanguageIdentifier.ById(EXISTING_LANGUAGE_ID);

var identifier = new ContentItemVariantIdentifier(itemIdentifier, languageIdentifier);

var responseVariant = await client.UpsertVariantAsync(identifier, contentItemVariantUpdateModel);

```

### Viewing a language variant

```chsarp

var itemIdentifier = ContentItemIdentifier.ByCodename(EXISTING_ITEM_CODENAME);
// var itemIdentifier = ContentItemIdentifier.ById(EXISTING_ITEM_ID);
// var itemIdentifier = ContentItemIdentifier.ByExternalId(EXTERNAL_ID);


var languageIdentifier = LanguageIdentifier.ByCodename(EXISTING_LANGUAGE_CODENAME);
// var languageIdentifier = LanguageIdentifier.ById(EXISTING_LANGUAGE_ID);

var identifier = new ContentItemVariantIdentifier(itemIdentifier, languageIdentifier);

var response = await _client.GetContentItemVariantAsync(identifier);
```

### Listing language variants

```csharp

var identifier = ContentItemIdentifier.ByCodename(EXISTING_ITEM_CODENAME);
// var identifier = ContentItemIdentifier.ById(EXISTING_ITEM_ID);
// var identifier = ContentItemIdentifier.ByExternalId(EXTERNAL_ID);

var responseVariants = await _client.ListContentItemVariantsAsync(identifier);
```

### Deleting language variants

```csharp
var itemIdentifier = ContentItemIdentifier.ById(itemResponse.Id);
var languageIdentifier = LanguageIdentifier.ByCodename(EXISTING_LANGUAGE_CODENAME);
var identifier = new ContentItemVariantIdentifier(itemIdentifier, languageIdentifier);

await _client.DeleteContentItemVariantAsync(identifier);
```




