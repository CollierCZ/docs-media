[![Forums](https://img.shields.io/badge/chat-on%20forums-orange.svg)](https://forums.kenticocloud.com)

# Kentico Cloud Delivery JavaRx/AndroidRx SDK

| Platform        | Maven Central Package  | jCenter Package | 
| ------------- |:-------------:| :-------------:|
| Android      | [![Android](https://img.shields.io/maven-central/v/com.kenticocloud/delivery-android.svg)](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22delivery-android%22) | [![Android](https://api.bintray.com/packages/kentico/KenticoCloudDeliveryJavaRxSDK/delivery-android/images/download.svg)](https://bintray.com/kentico/KenticoCloudDeliveryJavaRxSDK/delivery-android) |
| JavaRx      | [![JavaRx](https://img.shields.io/maven-central/v/com.kenticocloud/delivery-rx.svg)](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22delivery-rx%22) | [![JavaRx](https://api.bintray.com/packages/kentico/KenticoCloudDeliveryJavaRxSDK/delivery-rx/images/download.svg)](https://bintray.com/kentico/KenticoCloudDeliveryJavaRxSDK/delivery-rx) |

A client library for retrieving content from [Kentico Cloud](https://kenticocloud.com/) written in Java 7 for both `Java` & `Android` projects.

Even though the SDK is built with [ReactiveX programming](http://reactivex.io/) and supports [RxJava2](https://github.com/ReactiveX/RxJava) and [RxAndroid](https://github.com/ReactiveX/RxAndroid) querying, it also integrates with [OkHttp](http://square.github.io/okhttp/) for those developers who do not want to use Rx or are not familiar with it.

## Prerequisites

Java 7+

Android 2.3+ (minSdk 21)

## Getting started

The first step is to include SDK into your project, for example, as a Gradle compile dependency. Based on the project type choose of the following:

#### Java

```
compile 'com.kenticocloud:delivery-rx:2.0.1'
```

#### Android

```
compile 'com.kenticocloud:delivery-android:2.0.1'
```

Note: The only difference between these two dependencies is the 'Observable' they present for ReactiveX to subscribe to. Android will present a standard Rx2AndroidNetworking request while Java will present a generic http request as an observable. Most of your imports will be from the shared com.kenticocloud.delivery_core which is automatically included with both packages.

### Configuration

```java
// Kentico Cloud project Id
String projectId = "683771be-aa26-4887-b1b6-482f56418ffd";

// Type resolvers are required as they convert the content items to models in runtime based on 'system.type' 
// property of your items.
List<TypeResolver<?>> typeResolvers = new ArrayList<>();

// First you need to create models representing your items. 
// Following is an example of the sample Cafe type.
// Note that type resolvers are optional and you are not required to use them. However, the best
// practise is to use safe types instead of relying on dynamic objects and values.
public final class Cafe extends ContentItem {

    // This is the codename of your content type in Kentico Cloud
    public static final String TYPE = "cafe";

    @ElementMapping("country")
    public TextElement country;

    @ElementMapping("email")
    public TextElement email;

    public String getCountry() {
        return country.getValue();
    }

    public String getEmail() {
        return email.getValue();
    }
}

// After you define models, we add type resolvers that will eventually convert items from JSON to your model in runtime.
// Please note that currently you need to have models for all content types you intend to work with. We plan on releasing
// an update that will be able to return generic ContentItem if strongly typed model is not found.
typeResolvers.add(new TypeResolver<>(Cafe.TYPE, new Function<Void, Cafe>() {
    @Override
    public Cafe apply(Void input) {
        return new Cafe();
        }
    }
));

// Prepare configuration object (note there are other parameters for e.g. preview API key)
DeliveryConfig config = DeliveryConfig.newConfig(projectId)
    .withTypeResolvers(typeResolvers);
```

### Initialization

#### Java

Imports
```java
import com.kenticocloud.delivery_rx.DeliveryService;
import com.kenticocloud.delivery_core.IDeliveryService;
```

Service
```java
IDeliveryService deliveryService = new DeliveryService(config);
```

#### Android

Imports
```java
import com.kenticocloud.delivery_android.DeliveryAndroidService;
import com.kenticocloud.delivery_core.IDeliveryService;
```

Service
```java
IDeliveryService androidDeliveryService = new DeliveryAndroidService(config);
```

### Get data with Rx (Observables pattern)

```java
deliveryService.<Cafe>items()
    .type(Cafe.TYPE)
    .getObservable()
    .subscribe(new Observer<DeliveryItemListingResponse<Cafe>>() {
        @Override
        public void onSubscribe(Disposable disposable) {
        }

        @Override
        public void onNext(DeliveryItemListingResponse<Cafe> response) {
            // Access cafe items
            List<Cafe> cafes = response.getItems();

            // Use methods from your strongly typed model
            String country = cafes.get(0).getCountry();
        }

        @Override
        public void onError(Throwable throwable) {
        }

        @Override
        public void onComplete() {
        }
    });
```

### Get data using HttpAdapter (OkHttp client)

```java
DeliveryItemListingResponse<Cafe> response = this.deliveryService.<Cafe>items()
    .type(Cafe.TYPE)
    .get();
        
List<Cafe> cafes = response.getItems();
```

## API Reference

### Property binding 

First, make sure that your model extends `ContentItem` class, then create public fields with `ElementMapping` decorator which will make sure that the value from your field is mapped to the property. Based on what type of field you have, choose the proper element type. Supported element types include:

`AssetsElement`, `ContentElement`, `DateTimeElement`, `ModularContentElement`, `MultipleChoiceElement`, `NumberElement`, `RichTextElement`, `TaxonomyElement`, `TextElement` and `UrlSlugElement`

Example below shows a typical class with different elements:

```java
public final class Coffee extends ContentItem {

    public static final String TYPE = "coffee";

    @ElementMapping("product_name")
    public TextElement productName;

    @ElementMapping("price")
    public NumberElement price;

    @ElementMapping("image")
    public AssetsElement image;

    @ElementMapping("short_description")
    public RichTextElement shortDescription;
}
```

### Filtering, sorting

SDK contains all available [filters](https://developer.kenticocloud.com/v1/reference#content-filtering) and other parameters ([sort](https://developer.kenticocloud.com/v1/reference#content-ordering), [projection](https://developer.kenticocloud.com/v1/reference#projection), [paging](https://developer.kenticocloud.com/v1/reference#listing-response-paging)) as predefined methods for each query type (e.g. different options are available for items and taxonomies query). All of these methods are written in builder pattern which helps developers efficiently creating their queries. 

Examples:

```java
MultipleItemQuery<Cafe> query = deliveryService.<Cafe>items()
    .equalsFilter("elements.title", "London")
    .limitParameter(5)
    .depthParameter(2)
    .orderParameter("elements.street", OrderType.Desc);
```

If you need to add other query parameters to URL directly, you can use `addParameter` method:

```java
MultipleItemQuery<Cafe> query = deliveryService.<Cafe>items()
    .addParameter(new Filters.EqualsFilter("elements.title", "London"));
```

### Querying data

Each type of data (item, taxonomy, elements etc.) can be obtained using the available methods in `IDeliveryClient`. Following are basic examples of different queries:

```java
// items
SingleItemQuery<Cafe> cafeQuery = deliveryService.<Cafe>item("boston");
MultipleItemQuery<Cafe> cafesQuery = deliveryService.<Cafe>items();

// types
SingleTypeQuery typeQuery = deliveryService.type("Cafe");
MultipleTypeQuery typesQuery = deliveryService.types();

// taxonomies
SingleTaxonomyQuery taxonomyQuery = deliveryService.taxonomy("personas");
MultipleTaxonomyQuery taxonomiesQuery = deliveryService.taxonomies();

// elements
SingleContentTypeElementQuery elementQuery = deliveryService.contenTypeElement("cafe", "country");
```

To execute query choose either `get` or `getObservable` method whether you want to work with [ReactiveX](http://reactivex.io) API or not.

```java
// Get examples
Cafe cafe = cafeQuery.get().getItem();
List<Cafe> cafes = cafesQuery.get().getItems();

 // Observable examples
cafesQuery.getObservable()
    .subscribe(new Observer<DeliveryItemListingResponse<Cafe>>() {
        @Override
        public void onSubscribe(Disposable disposable) {
        }

        @Override
        public void onNext(DeliveryItemListingResponse<Cafe> response) {
            // Access cafe items
            List<Cafe> cafes = response.getItems();

            // Use methods from your strongly typed model
            String country = cafes.get(0).getCountry();
        }

        @Override
        public void onError(Throwable throwable) {
        }

        @Override
        public void onComplete() {
        }
    });

```

### Custom query parameters

It is possible to create custom query parameters in case you need to have some additional information in URL. This can be useful if you use proxy and need to log some additional information.

To create custom parameter implement `IQueryParameter` and use it in combination with `addParameter` query method.

```java
public static class CustomFilter implements IQueryParameter {

    private String data;

    public CustomFilter(String data){
        this.data = data;
    }

    @Override
    public String getParam() {
        return "customData";
    }

    @Override
    public String getParamValue() {
        return this.data;
    }
}

MultipleItemQuery<Cafe> query = deliveryService.<Cafe>items()
    .addParameter(new CustomFilter("result"));

```

### Preview mode

To enable preview mode, pass your API Preview key to the configuration object:

```java
new DeliveryConfig(projectId, typeResolvers, "yourPreviewAPIKey");
```

To make calls against preview API globally, use default `QueryConfig` during inicialization. This can be overridden when executing particular queries. 

```java

// Configure default global query config that will enable preview mode by default.
QueryConfig defaultQueryConfig = new QueryConfig();
defaultQueryConfig.setUsePreviewMode(true);

DeliveryConfig config = new DeliveryConfig(projectId, typeResolvers, defaultQueryConfig);


// Enable preview mode for a specific call. This overrides global configuration.
MultipleItemQuery<Cafe> query = deliveryService.<Cafe>items()
    .type(Cafe.TYPE)
    .setUsePreviewMode(true);
```

### Getting URL of query

You can get URL of the query without executing it by calling `getQueryUrl` method on any `IQuery` object.

```java
deliveryService.items()
    .equalsFilter("elements.title", "Warrior")
    .limitParameter(5)
    .depthParameter(2)
    .skipParameter(1)
    .getQueryUrl();
``` 

Outputs

```
https://deliver.kenticocloud.com/683771be-aa26-4887-b1b6-482f56418ffd/items?elements.title=Warrior&limit=5&depth=2&skip=1
```

### Advanced configuration

During initialization of `DeliveryConfig` you can configure following options:

| Method        | Use
| ------------- |:-------------:
| withTypeResolvers | Sets type resolvers responsible for mapping response to strongly typed object
| withPreviewApiKey      | Sets preview API key
| withSecuredApiKey | Sets secured API key
| withDeliveryApiUrl | Sets custom URL of Kentico Cloud Endpoint
| withDeliveryPreviewApiUrl | Sets custom URL of Kentico Cloud preview Endpoint
| withThrowExceptionForUnknownTypes | If enabled, SDK will throw Exception if it cannot find strongly typed model (type resolver) for certain item in response
| withDefaultQueryConfig | Sets default query config for all queries made within SDK. This is useful when you want to set default behavior and then override it on per query level.

Example:

```java
IDeliveryConfig config = DeliveryConfig.newConfig("projectId")
    .withPreviewApiKey("previewApiKey")
    .withThrowExceptionForUnknownTypes(true)
    .withDeliveryApiUrl("customDeliveryEndpointUrl");
```

### Handling errors

SDK will automatically map [Kentico Cloud error responses](https://developer.kenticocloud.com/v1/reference) to `KenticoCloudResponseException` runtime Exception that you can handle.

```java
try {
    DeliveryItemListingResponse<IContentItem> response = deliveryService.items().get();
} catch (KenticoCloudResponseException ex) {
    String cloudErrorMessage = ex.getMessage(); // i.e. missing item
} catch (Exception ex){
    // other error
}
```

## Sample applications
| Android | JavaRx |
| ------ |-----|
| [![Android](https://vignette.wikia.nocookie.net/scribblenauts/images/2/24/Android_Logo.png/revision/latest?cb=20130410160638)](https://github.com/Kentico/KenticoCloudDeliveryJavaRxSDK/tree/master/sample-android-app) | [![JavaRx](http://reactivex.io/assets/Rx_Logo_S.png)](https://github.com/Kentico/KenticoCloudDeliveryJavaRxSDK/tree/master/sample-java-app) |

## Feedback & Contributing

Check out the [contributing](https://github.com/Kentico/KenticoCloudDeliveryJavaRxSDK/blob/master/CONTRIBUTING.md) page to see the best places to file issues, start discussions, and begin contributing.
