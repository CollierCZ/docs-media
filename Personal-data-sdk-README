# Kentico Cloud Personal Data .NET SDK

[![Build status](https://ci.appveyor.com/api/projects/status/as11gvayj6vm087l?svg=true)](https://ci.appveyor.com/project/kentico/personal-data-sdk-net)

Nuget package will be released soon.

[![Forums](https://img.shields.io/badge/chat-on%20forums-orange.svg)](https://forums.kenticocloud.com)

## Summary

The Kentico Cloud Personal Data .NET SDK is a client library used for retrieving or deleting data about your tracked visitors. You can use the SDK in the form of a NuGet package (will be released soon).

## Prerequisites

To retrieve and delete data from Kentico Cloud via the Personal Data API, you need to have a Kentico Cloud subscription at <https://app.kenticocloud.com>. For more information see our [documentation](http://help.kenticocloud.com/).

#### Getting a User ID of a tracked visitor

Some methods of the SDK work with User ID, which identifies a specific visitor on your website. You can get this value either by [parsing the content of a cookie](https://github.com/Kentico/personalization-sdk-net#getting-userid-and-sessionid) stored by the tracking code in the browser or by calling a JavaScript method in the code of your website. For more details, see [Identifying visitors on websites](https://developer.kenticocloud.com/docs/retrieving-user-and-session-id).

## Using the PersonalDataClient

The `PersonalDataClient` class is the main class of the SDK. Using this class, you can retrieve and delete visitor data stored in your Kentico Cloud project.

To create an instance of the class, you need to provide a **Project ID** and your **Personal Data API Key**. You can find both inside the [Kentico Cloud app](https://app.kenticocloud.com/). From the app menu, choose *Project settings*. Then, under Development, choose *API keys*.

```csharp
var projectId = new Guid("<YOUR_PROJECT_ID");
var apiKey = "<YOUR_PERSONAL_DATA_API_KEY>";
client = new PersonalDataClient("https://personal-data-api.kenticocloud.com/", apiKey, projectId);
```

### Retrieving personal data of tracked visitors

#### Getting data by email address

```csharp
var response = await client.GetByEmailAsync("kroberts2y@godaddy.com");
```

#### Getting data by User ID:

```csharp
var response = await client.GetByUidAsync("aad50bb1223e4199");
```

### Deleting personal data of tracked visitors

#### Deleting data by email address:

```csharp
client.DeleteByEmailAsync("kroberts2y@godaddy.com");
```

#### Deleting data by User ID:

```csharp
client.DeleteByUidAsync("aad50bb1223e4199");
```

## Feedback & Contributing
Check out the [contributing](https://github.com/Kentico/personal-data-sdk-net/blob/master/CONTRIBUTING.md) page to see the best places to file issues, start discussions and begin contributing.
