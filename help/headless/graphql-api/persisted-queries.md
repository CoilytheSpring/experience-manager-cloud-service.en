---
title: Persisted GraphQL queries
description: Learn how to to persist GraphQL queries in Adobe Experience Manager as a Cloud Service to optimize performance. Persisted queries can be requested by client applications using HTTP GET method and the response can be cached at the dispatcher and CDN layers, ultimately improving the performance of the client applications.
feature: Content Fragments,GraphQL API
exl-id: 080c0838-8504-47a9-a2a2-d12eadfea4c0
---
# Persisted GraphQL queries {#persisted-queries-caching}

Persisted queries are GraphQL queries that are created and stored on the Adobe Experience Manager (AEM) as a Cloud Service server. They can be requested with a GET request by client applications. The response of a GET request can be cached at the dispatcher and CDN layers, ultimately improving the performance of the requesting client application. This differs from standard GraphQL queries, which are executed using POST requests where the response cannot easily be cached.

>[!NOTE]
>
>Persisted Queries are recommended. See [GraphQL Query Best Practices (Dispatcher)](/help/headless/graphql-api/content-fragments.md#graphql-query-best-practices) for details, and the related Dispatcher configuration.

The [GraphiQL IDE](/help/headless/graphql-api/graphiql-ide.md) is available in AEM for you to develop, test, and persist your GraphQL queries, before [transferring to your production environment](#transfer-persisted-query-production). For cases that need customization (for example, when [customizing the cache](/help/headless/graphql-api/graphiql-ide.md#caching-persisted-queries)) you can use the API; see the curl example provided in [How to persist a GraphQL query](#how-to-persist-query).

## Persisted Queries and Endpoints {#persisted-queries-and-endpoints}

Persisted queries must always use the endpoint related to the [appropriate Sites configuration](graphql-endpoint.md); so they can use either, or both:

* The Global configuration and endpoint
  The query has access to all Content Fragment Models.
* Specific Sites configuration(s) and endpoint(s)
  Creating a persisted query for a specific Sites configuration requires a corresponding Sites-configuration-specific endpoint (to provide access to the related Content Fragment Models). 
  For example, to create a persisted query specifically for the WKND Sites configuration, a corresponding WKND-specific Sites configuration, and a WKND-specific endpoint must be created in advance.

>[!NOTE]
>
>See [Enable Content Fragment Functionality in Configuration Browser](/help/sites-cloud/administering/content-fragments/content-fragments-configuration-browser.md#enable-content-fragment-functionality-in-configuration-browser) for more details.
>
>The **GraphQL Persistent Queries** need to be enabled, for the appropriate Sites configuration. 

For example, if there is a particular query called `my-query`, which uses a model `my-model` from the Sites configuration `my-conf`:

* You can create a query using the `my-conf` specific endpoint, and then the query will be saved as following: 
`/conf/my-conf/settings/graphql/persistentQueries/my-query`
* You can create the same query using `global` endpoint, but then the query will be saved as following:
`/conf/global/settings/graphql/persistentQueries/my-query`

>[!NOTE]
>
>These are two different queries - saved under different paths. 
>
>They just happen to use the same model - but via different endpoints.

## How to persist a GraphQL query {#how-to-persist-query}

It is recommended to persist queries on an AEM author environment initially and then [transfer the query](#transfer-persisted-query-production) to your production AEM publish environment, for use by applications. 

There are various methods of persisting queries, including:

* GraphiQL IDE - see [Saving Persisted Queries](/help/headless/graphql-api/graphiql-ide.md#saving-persisted-queries) (preferred method)
* curl - see the following example
* Other tools, including [Postman](https://www.postman.com/)

The GraphiQL IDE is the **preferred** method for persisting queries. To persist a given query using the **curl** command line tool:

1. Prepare the query by PUTing it to the new endpoint URL `/graphql/persist.json/<config>/<persisted-label>`.

   For example, create a persisted query:

   ```shell
   $ curl -X PUT \
       -H 'authorization: Basic YWRtaW46YWRtaW4=' \
       -H "Content-Type: application/json" \
       "http://localhost:4502/graphql/persist.json/wknd/plain-article-query" \
       -d \
   '{
     articleList {
       items{
           _path
           author
           main {
               json
           }
       }
     }
   }'
   ```

1. At this point, check the response.

   For example, check for success:

     ```json
     {
       "action": "create",
       "configurationName": "wknd",
       "name": "plain-article-query",
       "shortPath": "/wknd/plain-article-query",
       "path": "/conf/wknd/settings/graphql/persistentQueries/plain-article-query"
     }
     ```

1. You can then request the persisted query by GETing the URL `/graphql/execute.json/<shortPath>`.

   For example, use the persisted query:

   ```shell
   $ curl -X GET \
       http://localhost:4502/graphql/execute.json/wknd/plain-article-query
   ```

1. Update a persisted query by POSTing to an already existing query path.

   For example, use the persisted query:

   ```shell
   $ curl -X POST \
       -H 'authorization: Basic YWRtaW46YWRtaW4=' \
       -H "Content-Type: application/json" \
       "http://localhost:4502/graphql/persist.json/wknd/plain-article-query" \
       -d \
   '{
     articleList {
       items{
           _path
           author
           main {
               json
           }
         referencearticle {
           _path
         }
       }
     }
   }'
   ```

1. Create a wrapped plain query.

   For example:

   ```shell
   $ curl -X PUT \
       -H 'authorization: Basic YWRtaW46YWRtaW4=' \
       -H "Content-Type: application/json" \
       "http://localhost:4502/graphql/persist.json/wknd/plain-article-query-wrapped" \
       -d \
   '{ "query": "{articleList { items { _path author main { json } referencearticle { _path } } } }"}'
   ```

1. Create a wrapped plain query with cache control.

   For example:

   ```shell
   $ curl -X PUT \
       -H 'authorization: Basic YWRtaW46YWRtaW4=' \
       -H "Content-Type: application/json" \
       "http://localhost:4502/graphql/persist.json/wknd/plain-article-query-max-age" \
       -d \
   '{ "query": "{articleList { items { _path author main { json } referencearticle { _path } } } }", "cache-control": { "max-age": 300 }}'
   ```

1. Create a persisted query with parameters:

   For example:

   ```shell
   $ curl -X PUT \
       -H 'authorization: Basic YWRtaW46YWRtaW4=' \
       -H "Content-Type: application/json" \
       "http://localhost:4502/graphql/persist.json/wknd/plain-article-query-parameters" \
       -d \
   'query GetAsGraphqlModelTestByPath($apath: String!, $withReference: Boolean = true) {
     articleByPath(_path: $apath) {
       item {
         _path
           author
           main {
           plaintext
           }
           referencearticle @include(if: $withReference) {
           _path
           }
         }
       }
     }'
   ```


## How to execute a Persisted query {#execute-persisted-query}

To execute a Persisted query, a client application makes a GET request using the following syntax:

```
GET <AEM_HOST>/graphql/execute.json/<PERSISTENT_PATH>
```

Where `PERSISTENT_PATH` is a shortened path to where the Persistent query is saved.

1. For example `wknd` is the configuration name and `plain-article-query` is the name of the Persistent query. To execute the query:

   ```shell
   $ curl -X GET \
       https://publish-p123-e456.adobeaemcloud.com/graphql/execute.json/wknd/plain-article-query
   ```

1. Executing a query with parameters.

    >[!NOTE]
    >
    > Query variables and values must be properly [encoded](#encoding-query-url) when executing a Persistent query.

   For example:

   ```xml
   $ curl -X GET \
       "https://publish-p123-e456.adobeaemcloud.com/graphql/execute.json/wknd/plain-article-query-parameters%3Bapath%3D%2Fcontent%2Fdam%2Fwknd%2Fen%2Fmagazine%2Falaska-adventure%2Falaskan-adventures%3BwithReference%3Dfalse
   ```

   See using [query variables](#query-variables) for more details.

## Using query variables {#query-variables}

Query variables can be used with Persisted Queries. The query variables are appended to the request prefixed with a semicolon (`;`) using the variable name and value. Multiple variables are separated by semicolons. 

The pattern looks like the following:

```
<AEM_HOST>/graphql/execute.json/<PERSISTENT_QUERY_PATH>;variable1=value1;variable2=value2
```

For example the following query contains a variable `activity` to filter a list based on an activity value:

```graphql
query getAdventuresByActivity($activity: String!) {
      adventureList (filter: {
        adventureActivity: {
          _expressions: [
            {
              value: $activity
            }
          ]
        }
      }){
        items {
          _path
        adventureTitle
        adventurePrice
        adventureTripLength
      }
    }
  }
```

This query can be persisted under a path `wknd/adventures-by-activity`. To call the Persisted query where `activity=Camping` the request would look like this:

```
<AEM_HOST>/graphql/execute.json/wknd/adventures-by-activity%3Bactivity%3DCamping
```

Note that `%3B` is the UTF-8 encoding for `;` and `%3D` is the encoding for `=`. The query variables and any special characters must be [encoded properly](#encoding-query-url) for the Persisted query to execute.

## Caching your persisted queries {#caching-persisted-queries}

Persisted queries are recommended as they can be cached at the dispatcher and CDN layers, ultimately improving the performance of the requesting client application.

By default AEM will invalidate the Content Delivery Network (CDN) cache based on a default Time To Live (TTL). 

This value is set to:

* 7200 seconds is the default TTL for the Dispatcher and CDN; also known as *shared caches*
  * default: s-maxage=7200
* 60 is the default TTL for the client (for example, a browser)
  * default: maxage=60

If you want to change the TTL for your GraphLQ query, then the query must be either:

* persisted after managing the [HTTP Cache headers - from the GraphQL IDE](#http-cache-headers)
* persisted using the [API method](#cache-api). 

### Managing HTTP Cache Headers in GraphQL  {#http-cache-headers-graphql}

The GraphiQL IDE - see [Saving Persisted Queries](/help/headless/graphql-api/graphiql-ide.md#managing-cache)

### Managing Cache from the API {#cache-api}

This involves posting the query to AEM using CURL in your command line interface. 

For an example:

```xml
curl -X PUT \
    -H 'authorization: Basic YWRtaW46YWRtaW4=' \
    -H "Content-Type: application/json" \
    "https://publish-p123-e456.adobeaemcloud.com/graphql/persist.json/wknd/plain-article-query-max-age" \
    -d \
'{ "query": "{articleList { items { _path author main { json } referencearticle { _path } } } }", "cache-control": { "max-age": 300 }}'
```

The `cache-control` can be set at the creation time (PUT) or later on (for example, via a POST request for instance). The cache-control is optional when creating the persisted query, as AEM can provide the default value. See [How to persist a GraphQL query](/help/headless/graphql-api/persisted-queries.md#how-to-persist-query), for an example of persisting a query using curl.

## Encoding the query URL for use by an app {#encoding-query-url}

For use by an application, any special characters used when constructing query variables (i.e semicolons (`;`), equal sign (`=`), slashes `/`) must be converted to use the corresponding UTF-8 encoding.

For example:

```xml
curl -X GET \ "https://publish-p123-e456.adobeaemcloud.com/graphql/execute.json/wknd/adventure-by-path%3BadventurePath%3D%2Fcontent%2Fdam%2Fwknd%2Fen%2Fadventures%2Fbali-surf-camp%2Fbali-surf-camp"
```

The URL can be broken down into the following parts:

| URL Part | Description |
|----------| -------------|
| `/graphql/execute.json` | Persistent query endpoint |
| `/wknd/adventure-by-path` | Persistent query path |
| `%3B`                | Encoding of `;` |
| `adventurePath`      | Query variable |
| `%3D`                | Encoding of `=` |
| `%2F`                | Encoding of `/` |
| `%2Fcontent%2Fdam...` | Encoded path to the Content fragment |

In plain text the request URI looks like the following:

```plaintext
/graphql/execute.json/wknd/adventure-by-path;adventurePath=/content/dam/wknd/en/adventures/bali-surf-camp/bali-surf-camp
```

To use a persisted query in a client app, the AEM headless client SDK should be used for [JavaScript](https://github.com/adobe/aem-headless-client-js), [Java](https://github.com/adobe/aem-headless-client-java), or [NodeJS](https://github.com/adobe/aem-headless-client-nodejs). The Headless Client SDK will automatically encode any query variables appropriately in the request.

## Transferring a persisted query to your Production environment  {#transfer-persisted-query-production}

Persisted queries should always be created on an AEM Author service and then published (replicated) to an AEM Publish service. Often, Persisted queries are created and tested on lower environments like local or Development environments. It is then necessary to promote Persisted queries to higher level environments, ultimately making them available on a production AEM Publish environment for client applications to consume.

### Package Persistent queries

Persistent queries can be built into [AEM Packages](/help/implementing/developing/tools/package-manager.md). AEM Packages can then be downloaded and installed on different environments. AEM Packages can also be replicated from an AEM Author environment to AEM Publish environments.

To create a Package:

1. Navigate to **Tools** > **Deployment** > **Packages**.
1. Create a new package by tapping **Create Package**. This will open a dialog to define the Package.
1. In the Package Definition Dialog, under **General** enter a **Name** like "wknd-persistent-queries".
1. Enter a version number like "1.0".
1. Under **Filters** add a new **Filter**. Use the Path Finder to select the `persistentQueries` folder beneath the configuration. For example for the `wknd` configuration the full path will be `/conf/wknd/settings/graphql/persistentQueries`.
1. Tap **Save** to save the new Package definition and close the dialog.
1. Tap the **Build** button in the newly created Package definition.

After the package has been built you can: 

* **Download** the package and re-upload on a different environment.
* **Replicate** the package by tapping **More** > **Replicate**. This will replicate the package to the connected AEM Publish environment.

<!--
1. Using replication/distribution tool:
   1. Go to the Distribution tool.
   1. Select tree activation for the configuration (for example, `/conf/wknd/settings/graphql/persistentQueries`).

* Using a workflow (via workflow launcher configuration):
  1. Define a workflow launcher rule for executing a workflow model that would replicate the configuration on different events (for example, create, modify, amongst others).
-->
