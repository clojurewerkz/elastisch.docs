---
title: "Elastisch, a minimalistic Clojure client for ElasticSearch: Getting Started with ElasticSearch and Clojure"
layout: article
---

## About this guide

This guide covers ElasticSearch indexing capabilities in depth, explains how Elastisch presents them in the API and
how some of the key features are commonly used. This guide covers:

 * What is indexing in the context of full text search
 * What kind of features ElasticSearch has w.r.t. indexing, how Elastisch exposes them in the API
 * Mapping types and how they define how the data is indexed by ElasticSearch
 * How to define mapping types with Elastisch
 * Lucene built-in analyzers, their characteristics, what different kind of analyzers are good for.
 * Other topics related to indexing and working with indexes

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/elastisch.docs).


## Overview

Before documents can be searched, they need to be **indexed**. Indexing is a process of taking a document with one or more fields, analyzing those
fields, producing data structures that can be efficiently searched over and storing them (in RAM, on disk, in a data store of some kind, etc).

**Analysis** is a process of several stages:

 * Tokenization: breaking field values into **tokens**
 * Filtering or modifying tokens
 * Combining them with field names to produce **terms**

How exactly a document was analyzed defines what search queries will match (find) it. ElasticSearch is based on [Apache Lucene](http://lucene.apache.org) and
offers several analyzers developers can use to achieve the kind of search quality and performance requirements they need. For example, different languages
require different analyzers: English, Mandarin Chinese, Arabic and Russian cannot be analyzed the same way.

It is possible to skip performing analysis for fields and specify if field values are stored in the index or not. Fields that are not stored
still can be searched over but will not be included into search results.

ElasticSearch allows users to define how exactly different kinds of documents are indexed, analyzed and stored.

### Multi-tenancy

ElasticSearch has excellent support for **multi-tenancy**: an ElasticSearch cluster can have a virtually unlimited number of indexes and mapping types.
For example, you can use a separate index per user account or organization in a SaaS (software as a service) product.


## Creating Indexes

ElasticSearch will create an index the first time it is used but it is also possible to precreate an index with
specific settings, mappings, and so on with the `clojurewerkz.elastisch.index/create` function. In the simplest case,
it only takes index name (a string) as the only argument:

{% gist af984cb5bc86339bcaae %}

It is also possible to define settings and mapping types at the time an index is created. Just pass `:settings` and `:mappings` options:

{% gist 0e462a2bdcd2e744bb57 %}

{% gist 917d482b6b017e626302 %}


### Index Settings

Index settings let you control many aspects of how ElasticSearch will use an index, store it, update it and so on.
ElasticSearch documentation on index settings uses dot separated names for nested setting map attributes.

For example, to set `index.refresh_interval` to 10, pass the following map for the `:settings` key:

{% gist 1c524ff9ea227da6df34 %}


For the reference list of index settings, see

 * [Update Index Settings Operation guide](http://www.elasticsearch.org/guide/reference/api/admin-indices-update-settings.html)
 * [Index modules guide](http://www.elasticsearch.org/guide/reference/index-modules/)



## Overview of Mapping Types

ElasticSearch has the concept of **mappings** that define which fields in documents are indexed, if/how they are analyzed and if they are stored. Each index in
ElasticSearch may have one or more **mapping types**. Mapping types can be thought of as tables in a database (although this analogy does not always stand).


## Creating Mapping Types

Mapping types can specified when an index is created using the `:mapping` option:

{% gist 917d482b6b017e626302 %}

It is possible to update mapping types after they are created, as described later in this guide.


### Mapping Type Settings

TBD


## Getting Mapping Types

To retrieve information about an existing mapping type, use the `clojurewerkz.elastisch.index/get-mapping` function:

{% gist a5b4dc1cea1d62fcd5aa %}

It is possible to fetch multiple mapping types in a single request by passing a collection (typically a vector) for the
second argument.

It is possible to specify a collection of indexes (typically a vector) or the special `"_all"` value to fetch mapping types
information for multiple (or all) indexes.


## Updating Mapping Types

It is possible to update an existing mapping type using the `clojurewerkz.elastisch.index/update-mapping` function:

{% gist f8d296a40c496d793444 %}

It is possible to specify multiple indexes by passing a vector of names or the special value `"_all"` to update a mapping for all
existing indexes:

{% gist b9211ee31ed658056113 %}

### Mapping Conflicts

When a mapping already exists, defining it with different attributes may result in conflicts.
ElasticSearch will try to be reasonably smart and merge mapping definitions. However, in some cases (like if core type of a field has changed)
it is not possible to do. In that case, by default ElasticSearch will respond with an error (40x status code) and Elastisch will raise an exception.

It is possible to instruct ElasticSearch to ignore conflicts and simply use the most recent provided mapping by passing the `:ignore_conflicts` option
to `clojurewerkz.elastisch.index.update-mapping`:

{% gist 9411e81621cde484b3bf %}

For more information, see [ElasticSearch guide on Put Mapping operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-put-mapping.html).



### Deleting Mapping Types

To delete an existing mapping type, use the `clojurewerkz.elastisch.index/delete-mapping` function:

{% gist 7eab32561f3e61ffb24a %}

Deleting a mapping type **causes all its data/documents to be deleted**. Think of it as dropping a database table.


## Disabling Analysis for Fields

It is possible to disable analysis for a field. In that case, the field's value still will be searchable as a single
token (exact match).

TBD


## Stored Fields

TBD


## Built-in Analyzers

ElasticSearch supports several kinds of analyzers. This section briefly describes some of them. To experiment with
analysis and analyzers, you can use the [Analyze API operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-analyze.html) on
any existent index, for example, via tool like `curl`.

### Standard Analyzer

The most sophisticated built-in analyzer. Intelligent enough to handle (tokenize correctly) email addresses, most of organization names
and so on.

TBD

### Whitespace Analyzer

Whitespace analyzer is very simplistic: it splits text into tokens on whitespace characters. Tokens are not lowercased.

TBD

### Simple Analyzer

Splits text into tokens on non-letter characters. Tokens are lowercased. *Discards numeric characters*.

TBD

### Stopword Analyzer

The same as simple analyzer but also removes [stop words](http://en.wikipedia.org/wiki/Stop_words).

TBD

### Language-specific Analyzers

TBD


## Common Use Cases for Built-in Analyzers

TBD


## Document TTL (Time-to-Live)

TBD


## Document Versioning

TBD


## Checking If an Index Exists

To check if an index exists, use the `clojurewerkz.elastisch.rest.index/exists?` function:

{% gist 776db68817a2febd847d %}


## Getting Index Settings

It is possible to fetch index settings using the `clojurewerkz.elastisch.rest.index/get-settings` function:

{% gist 51d27636d46c9941c9cf %}

It returns a Clojure map of settings.


## Updating Index Settings

To update index settings, use the `clojurewerkz.elastisch.rest.index/update-settings` function:

{% gist d57e34d65603b14566fc %}

See also ElasticSearch [Update Index Setting operation guide](http://www.elasticsearch.org/guide/reference/api/admin-indices-update-settings.html)


## Deleting an Index

To delete an index, use the `clojurewerkz.elastisch.rest.index/delete` function:

{% gist ece83eac980f48a748b2 %}


## Opening and Closing Indexes

ElasticSearch lets you "close" an index and later "reopen" it. Closed indexes only have their metadata in memory
so if an index is not used (for example, belongs to a suspended account), it is possible to save
some resources by closing it. When an index is reopened, it goes through the regular recovery process.

To open and close an index, use the `clojurewerkz.elastisch.rest-api.index/open` and `clojurewerkz.elastisch.rest-api.index/close`
functions, respectively. Both take index name as the only argument.


## Refreshing an Index

Refreshing an index makes all changes (added, modified and deleted documents) since the last refresh available for search. In other
words, index changes become "visible" to clients. ElasticSearch periodically refreshes indexes (configurable via index settings,
see earlier in this guide) but it is possible to refresh an index manually with the `clojurewerkz.elastisch.rest.index/refresh` function
that takes index name as the only argument:

{% gist 20f584500af4540a2f6a %}


## Optimizing an Index

Use `clojurewerkz.elastisch.rest.index/optimize` to optimize an index:

{% gist fae56a5583e204718004 %}

It takes the same options as documented in the [ElasticSearch guide on the Optimize Index operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-optimize.html)


## Flushing an Index

Use `clojurewerkz.elastisch.rest.index/flush` to flush an index:

{% gist 34fe2aa404f8585e1ff8 %}

It accepts the only option: `:refresh`. When passed as true, it will also refresh the index after flushing it.


## Clearing Index Cache

Use `clojurewerkz.elastisch.rest.index/clear-cache` to clear index cache:

{% gist 61957a1630e094318b66 %}

It takes the same options as documented in the [ElasticSearch guide on the Clear Cache Index operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-clearcache.html)


## Getting Index Stats

Use `clojurewerkz.elastisch.rest.index/stats` to get statistics about an index or multiple indexes:

{% gist e6eeb4925e952197afdf %}

It takes the same options as documented in the [ElasticSearch guide on the Index Stats operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-stats.html)


## Getting Index Status

Use `clojurewerkz.elastisch.rest.index/status` to retrieve status of one or more indexes:

{% gist 113594d20373065d3b2d %}

Accepted options are:

 * `:recovery`: should recovery status be returned?
 * `:snapshot`: should snapshotting status be returned?

As with many other functions in Elastisch, passing a collection for index name will perform
the operation on multiple indexes.

The `:indices` key in the returned map contains status for all requested indexes.


## Getting Segments Information for an Index

Use `clojurewerkz.elastisch.rest.index/segments` to retrieve information about index segments:

{% gist ca796b8a0c16402e5209 %}

It is possible to get info for multiple indexes at the same time: just pass a collection or the special value `"_all"`
for index name. This function accepts no options.


## Misc Topics

### How to Set Index Refresh Interval

TBD


### The _all Field

The `_all` field is a special document field that includes the content of one or more (possibly all) document fields combined.
It is helpful in cases when querying against documents with unknown document structure.

It is possible to disable the `_all` field for a mapping or exclude certain fields from being added to it.

TBD


### Default Query Field

TBD


### Testing How Text is Analyzed

It is possible to use the [ElasticSearch Analyze API operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-analyze.html) to see how different
analyzers process (tokenize and filter) various pieces of text.


## Wrapping Up

TBD


## What to Read Next

The documentation is organized as [a number of guides](/articles/guides.html), covering different topics in depth:

 * [Querying](/articles/querying.html)
 * [Facets](/articles/facets.html)
 * [Percolation](/articles/percolation.html)
 * [Routing and Distribution](/articles/distribution.html)



## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Elastisch mailing list](https://groups.google.com/forum/#!forum/clojure-elasticsearch)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the
documentation better.
