---
title: "Elastisch, a minimalistic Clojure client for ElasticSearch: Getting Started with ElasticSearch and Clojure"
layout: article
---

## About this guide

This guide combines an overview of Elastisch with a quick tutorial that helps you to get started with it.
It should take about 15 minutes to read and study the provided code examples. This guide covers:

 * Features of Elastisch
 * Clojure and ElasticSearch version requirements
 * How to add Elastisch dependency to your project
 * A very brief introduction to Elastisch capabilities
 * Examples of the most common operations with Elastisch

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/elastisch.docs).


## What version of Elastisch does this guide cover?

This guide covers Elastisch 1.0 preview releases.



## Elastisch Overview

Elastisch is an idiomatic Clojure client for [ElasticSearch](http://elasticsearch.org/), a modern, scalable, powerful full text search server. It is simple and easy to use,
has good baseline performance and strives to support every ElasticSearch feature. Elastisch is minimalistic: it mimics ElasticSearch operations in its API but does not add
any abstractions or  ODM-like layers beyond that.


## Supported Clojure versions

Elastisch is built from the ground up for Clojure 1.3 and later.


## Supported ElasticSearch versions

Elastisch is developed and tested with recent stable ElasticSearch releases (0.19.x or later). It is very likely that earlier versions also work fine but we do not run
[continuous integration](http://travis-ci.org/#!/clojurewerkz/elastisch) against them.


## Adding Elastisch Dependency To Your Project

Monger artifacts are [released to Clojars](https://clojars.org/clojurewerkz/elastisch).

### With Leiningen

    [clojurewerkz/elastisch "1.0.0-beta1"]

### With Maven

Add Clojars repository definition to your `pom.xml`:

{% gist 65642c4b53d26539e5f6 %}

And then the dependency:

    <dependency>
      <groupId>clojurewerkz</groupId>
      <artifactId>elastisch</artifactId>
      <version>1.0.0-beta1</version>
    </dependency>

It is recommended to stay up-to-date with new versions. New releases and important changes are announced [@ClojureWerkz](http://twitter.com/ClojureWerkz).



## Connecting

Before you can index and search with Elastisch, it is necessary to tell Elastisch what ElasticSearch node to use. To do so, you use the `clojurewerkz.elastisch.rest/connect!`
function that takes an endpoint as its sole argument:

{% gist 2fd18ab64260be7098c9 %}

By default Elastisch will use the endpoint at `http://localhost:9200`.


## Indexing

Before data can be searched over, it needs to be indexed. Indexing is the process of scanning the text and building a list of search terms and data structures
called a **search index**. Search index allows search engines such as ElasticSearch to retrieve relevant documents for a query efficiently.

The process of indexing involves a few steps:

 * Create an index
 * [Optionally] Defining **mappings** (how documents should be indexed)
 * Submitting documents for indexing via HTTP or other APIs

Next we will take a look at each stage in more detail.


### Creating an Index

ElasticSearch provides support for multiple indexes. Indexes can be thought of as databases in a DBMS. 

To create an index, use the `clojurewerkz.elastisch.rest.index/create` function:

{% gist 396d878ae0ff624190a9 %}

Most commonly used index settings are `"number_of_shards"` and `"number_of_replicates"`. We won't go into details about them in this guide,
please refer to the [Indexing guide](/articles/indexing.html) for more details.



### Creating a Mapping

Mappings define which fields in documents are indexed, if/how they are tokenized, analyzed and so on. Each index in ElasticSearch may have one or more
**mapping types**. Mapping types can be thought of as tables in a database (although this analygy does not always stand).

Mapping types are specified when an index is created using the `:mapping` option:

{% gist 917d482b6b017e626302 %}

Please refer to the [Indexing guide](/articles/indexing.html) for more information about mapping types, analyzers and so on.



### Indexing documents

To add a document to an index, use the `clojurewerkz.elastisch.rest.document/create` function. This will cause document id to be generated
automatically:

{% gist 09098979301600a233a7 %}

`clojurewerkz.elastisch.rest.document/put` will add a document to the index but expects document id to be provided:

{% gist 17e77615e93abad1b109 %}

TBD


### Checking responses

`clojurewerkz.elastisch.rest.document/create`, `clojurewerkz.elastisch.rest.document/put`, `clojurewerkz.elastisch.rest.index/create` and other functions
return ElasticSearch responses as Clojure maps. To check if they are successful, use functions in the `clojurewerkz.elastisch.rest.response` namespace,
for example, `clojurewerkz.elastisch.rest.response/ok?` or `clojurewerkz.elastisch.rest.response/conflict?`:

{% gist 3696c16de1665fe6f055 %}



## Querying

### Overview

ElasticSearch supports [multiple kinds of queries](http://www.elasticsearch.org/guide/reference/query-dsl/): from simple like term and prefix query to compound like the bool query. Some queries can also have
filters associated with them.


### Performing queries

To perform a query with Elastisch, use the `clojurewerkz.elastisch.rest.document/search` function. It takes index name, mapping name and query
(as a Clojure map):

{% gist 1dfaf5fa28a46712c41a %}

Search requests with Elastisch have exactly the same structure as JSON documents in the [ElasticSearch Query API guide](http://www.elasticsearch.org/guide/reference/query-dsl/)
but passed as Clojure maps. `:query`, `:sort`, `:facets` and other keys that [ElasticSearch Search API documentation](http://www.elasticsearch.org/guide/reference/api/search/)
mentions are passed as maps.

Because every search request contains query information (the `:query` key), you can either pass an entire query as a map or use one or more convenience functions
from the `clojurewerkz.elastisch.query` namespace (more on them later in this guide).

The example from above can also be written like so:

{% gist a0b3f8e653769688e1dd %}


### Searching Against Multiple Indexes or Mappings

To search against multiple indexes or mappings, pass them as vectors to their respective function arguments.

TBD: example


### Checking results

Results returned by search functions have the same structure as ElasticSearch JSON responses:

{% gist 5e0892e1fd56a33f3083 %}

Several functions in the `clojurewerkz.elastisch.rest.response` namespace can be used to access more specific piece of information from a response,
such as total number of hits.

The most commonly used are:

 * `clojurewerkz.elastisch.rest.response/total-hits`: returns number of hits (documents found)
 * `clojurewerkz.elastisch.rest.response/hits-from`: returns a collection of hits (stored document plus id, score and index information)
 * `clojurewerkz.elastisch.rest.response/any-hits?`: returns true if there is at least one document found
 * `clojurewerkz.elastisch.rest.response/no-hits?` is a [complement function](http://clojuredocs.org/clojure_core/clojure.core/complement) to `any-hits?`
 * `clojurewerkz.elastisch.rest.response/ids-from`: returns a collection of document ids collected from hits

All of them take a response map as the only argument.


### Different kinds of queries

ElasticSearch is a feature rich search engine and it supports many types of queries. Even though
all queries can be passed as Clojure maps, it is common to use convenient functions from the
`clojurewerkz.elastisch.query` to construct queries.

We won't cover all the functions in this guide but will take a closer look at a few of them:

#### Term query

[Term query]() matches documents that have fields that contain a term (exactly as given, not analyzed).

Short example:

{% gist fc01bb14221852a4175d %}

Raw query:

{% gist e47f48e9f2879e9285f8 %}

TBD: full example

#### Query string query

[Query string query](http://www.elasticsearch.org/guide/reference/query-dsl/query-string-query.html) parses provided query string using a query parser. It supports Lucene query language
expressions such as "this AND that OR thus":

Short example:

{% gist e9613b3f2c5b9141ff23 %}

Raw query:

{% gist f469349de876d1122883 %}

TBD: full example

Query string query accepts many parameters that can be found on the [ElasticSearch documentation site](http://www.elasticsearch.org/guide/reference/query-dsl/query-string-query.html).

#### Range query

TBD

Short example:

{% gist  %}

Raw query:

{% gist  %}

TBD: full example

#### Boolean (bool) query

TBD

Short example:

{% gist  %}

Raw query:

{% gist  %}

TBD: full example

#### IDs query

TBD

Short example:

{% gist  %}

Raw query:

{% gist  %}

TBD: full example

#### Field query

TBD

Short example:

{% gist  %}

Raw query:

{% gist  %}

TBD: full example

#### More Like This query

TBD

Short example:

{% gist  %}

Raw query:

{% gist  %}

TBD: full example


### Sorting Results

ElasticSearch supports flexible sorting of search results. To specify the desired sorting,
use the `:sort` option `clojurewerkz.elastisch.rest.document/search` accepts:

{% gist 304487b510685f04384d %}

More examples can be found in this [ElasticSearch documentation section on sorting](http://www.elasticsearch.org/guide/reference/api/search/sort.html).
`:sort` values that Elastisch accepts are structured exactly the same as JSON documents in that section.


### Using Offset, Limit

To limit returned number of results, use `:from` and `:size` options `clojurewerkz.elastisch.rest.document/search` accepts:

{% gist 2b80925039b047a8df9b %}

Default value of `:from` is 0, of `:size` is 10.


### Using Filters

TBD


### Highlighting

Having search matches highlighted in the UI is very useful in many cases. ElasticSearch can highlight matched in search results. To enable highlighting,
use the `:highlight` option `clojurewerkz.elastisch.rest.document/search` accepts. In the example above, search matches in the `biography` field will
be highlighted (wrapped in `em` tags) and search hits will include one extra "virtual" field called `:highlight` that includes the highlighted fields and the highlighted fragments
that can be used by your application.

{% gist ae91b88ca45fa83bf417 %}

More examples can be found in this [ElasticSearch documentation section on retrieving subsets of fields](http://www.elasticsearch.org/guide/reference/api/search/highlighting.html).
`:highlight` values that Elastisch accepts are structured exactly the same as JSON documents in that section.

Highlighting will be covered in more detail in the [Querying](/articles/querying.html) guide.


### Retrieving a Subset of Fields

To limit returned number of results, use the `:fields` option `clojurewerkz.elastisch.rest.document/search` accepts:

{% gist 2ff96ae345b858cae055 %}

More examples can be found in this [ElasticSearch documentation section on retrieving subsets of fields](http://www.elasticsearch.org/guide/reference/api/search/fields.html).
`:fields` values that Elastisch accepts are structured exactly the same as JSON documents in that section.


## Updating documents

TDB


## Deleting documents

TBD



## Wrapping up

Congratulations, you now can use Elastisch to work with ElasticSearch. Now you know enough to start building a real application. ElasticSearch is very feature rich and
this guide does not cover many topics (or does not cover them in detail):

 * Index operations
 * Advanced mapping features (e.g. settings or nested mappings)
 * Search across multiple indexes and/or mapping types
 * Filtering
 * Analyzers
 * Facets
 * "More like this" queries
 * Multi Search and Bulk Operations
 * Percolation
 * Index templates
 * Routing and distribution features

and more. They are covered in the rest of the guides, with links to respective ElasticSearch documentation sections.

We hope you find Elastisch reliable, consistent and easy to use. In case you need help, please ask on the [mailing list](https://groups.google.com/forum/#!forum/clojure-elasticsearch)
and follow us on Twitter [@ClojureWerkz](http://twitter.com/ClojureWerkz).


## What to Read Next

The documentation is organized as [a number of guides](/articles/guides.html), covering different topics in depth:

 * [Indexing](/articles/indexing.html)
 * [Querying](/articles/querying.html)
 * [Facets](/articles/facets.html)
 * [Percolation](/articles/percolation.html)
 * [Routing and Distribution](/articles/distribution.html)

They also cover topics from this guide in more detail.


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Elastisch mailing list](https://groups.google.com/forum/#!forum/clojure-elasticsearch)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the
documentation better.
