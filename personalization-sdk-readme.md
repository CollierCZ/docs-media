# Kentico Cloud Personalization .NET SDK

[![Build status](https://ci.appveyor.com/api/projects/status/bbn6iy1yok766yux/branch/master?svg=true)](https://ci.appveyor.com/project/kentico/personalization-sdk-net/branch/master) [![Forums](https://img.shields.io/badge/chat-on%20forums-orange.svg)](https://forums.kenticocloud.com)


| Package | Version |
| :-- | :--: |
| KenticoCloud.Personalization | [![NuGet](https://img.shields.io/nuget/v/KenticoCloud.Personalization.svg)](https://www.nuget.org/packages/KenticoCloud.Personalization) |
| KenticoCloud.Personalization.AspNetCore | [![NuGet](https://img.shields.io/nuget/v/KenticoCloud.Personalization.AspNetCore.svg)](https://www.nuget.org/packages/KenticoCloud.Personalization.AspNetCore) |
| KenticoCloud.Personalization.MVC | [![NuGet](https://img.shields.io/nuget/v/KenticoCloud.Personalization.MVC.svg)](https://www.nuget.org/packages/KenticoCloud.Personalization.MVC) |


## Summary

The Kentico Cloud Personalization .NET SDK is a library used for retrieving personalization information (such as visitor's first visit, activity on a website, etc.) from [Kentico Cloud](https://kenticocloud.com/). You can use the SDK in the form of a [NuGet package](https://www.nuget.org/packages/KenticoCloud.Personalization).

## Prerequisities

To retrieve data from Kentico Cloud via the Personalization API, you need to have a Kentico Cloud subscription at <https://app.kenticocloud.com>. For more information see our [documentation](http://help.kenticocloud.com/).

## Basic scenarios

### Create PersonalizationClient instance

The **PersonalizationClient** class in the `KenticoCloud.Personalization` assembly is the main class of the SDK that enables you to query the API. To create an instance of the class, you need to provide your **personalization API key**. You can find the key in the Development section of Kentico Cloud.

### Getting UserID and SessionID

_User ID_ and _Session ID_ identify a specific visitor and his current session. These values are saved in a cookie and we provide methods to retrieve them in code.

* If you have an ASP.NET MVC application, use the [KenticoCloud.Personalization.MVC](https://www.nuget.org/packages/KenticoCloud.Personalization.MVC) NuGet package. This package provides extension methods for the `HttpRequestBase` object, which is generally available in ASP.NET MVC applications.
* If you have an ASP.NET Core application, use the [KenticoCloud.Personalization.AspNetCore](https://www.nuget.org/packages/KenticoCloud.Personalization.AspNetCore) NuGet package. This packages provides extension methods for the `HttpRequest` object, which is generally available in ASP.NET Core applications.

### Basic querying examples

Once you create the `PersonalizationClient` instance, you can start querying the Personalization API by calling methods on the class instance.

```C#
// Retrieves usual location of a visitor
var client = new PersonalizationClient("eyJh...5cCI");
var location = await client.GetVisitorUsualLocationAsync("0f2a1fa152b8e92d");
```

```C#
// Retrieves information about the first visit of a visitor
var client = new PersonalizationClient("eyJh...5cCI");
var visit = await client.GetFirstVisitAsync("0f2a1fa152b8e92d");
```

```C#
// Retrieves information about the current session of a visitor
var client = new PersonalizationClient("eyJh...5cCI");
var session = await client.GetCurrentSessionAsync("0f2a1fa152b8e92d", "8d532785326b0258");
```

### Example – use in ASP.NET MVC applications

The following example shows how you can use the Personalization API in an ASP.NET MVC application to find out where the current visitor came from (i.e., their origin) before browsing on your website.

```C#
using System.Threading.Tasks;
using System.Web.Mvc;
using KenticoCloud.Personalization;
using KenticoCloud.Personalization.MVC;

namespace DancingGoat.Controllers
{
    public class HomeController : Controller
    {
        // Initializes an instance of the PersonalizationClient class
        private static readonly PersonalizationClient PersonalizationClient = new PersonalizationClient("eyJh...5cCI");

        public async Task<ActionResult> Index()
        {
            // Retrieves User ID and Session ID of the current visitor
            var uid = this.Request.GetCurrentPersonalizationUid();
            var sid = this.Request.GetCurrentPersonalizationSid();

            // Retrieves the origin of the visitor's session
            var session = await PersonalizationClient.GetCurrentSessionAsync(uid, sid);
            ViewBag.PersonalizationSessionOrigin = session.Origin;

            return View();
        }
    }
}
```

See [Personalizing content](https://developer.kenticocloud.com/docs/personalizing-content) in our DevHub for a more detailed explanation of delivering different content to different visitors.

## Feedback & Contributing
Check out the [contributing](https://github.com/Kentico/personalization-sdk-net/blob/master/CONTRIBUTING.md) page to see the best places to file issues, start discussions and begin contributing.
