---
title: "Elastisch, a minimalistic Clojure client for ElasticSearch: Getting Started with ElasticSearch and Clojure"
layout: article
---

## About this guide

This guide combines an overview of Elastisch with a quick tutorial that helps you to get started with it.
It should take about 15 minutes to read and study the provided code examples. This guide covers:

 * Mapping types and how they define how the data is indexed by ElasticSearch
 * How to define mapping types with Elastisch
 * Lucene built-in analyzers, their characteristics, what different kind of analyzers are good for.
 * Other topics related to indexing

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


## Mapping Types

ElasticSearch has the concept of **mappings** that define which fields in documents are indexed, if/how they are analyzed and if they are stored. Each index in
ElasticSearch may have one or more **mapping types**. Mapping types can be thought of as tables in a database (although this analogy does not always stand).

### Creating and Updating Mapping Types

Mapping types can specified when an index is created using the `:mapping` option:

{% gist 917d482b6b017e626302 %}

Or using the `clojurewerkz.elastisch.index.update-mapping` function:

{% gist %}

It is possible to specify multiple indexes by passing a vector of names or the special value `"_all"` to update a mapping for all
existing indexes:

{% gist %}

TBD


### Mapping Conflicts

When a mapping already exists, defining it with different attributes may result in conflicts.
ElasticSearch will try to be reasonably smart and merge mapping definitions. However, in some cases (like if core type of a field has changed)
it is not possible to do. In that case, by default ElasticSearch will respond with an error (40x status code) and Elastisch will raise an exception.

It is possible to instruct ElasticSearch to ignore conflicts and simply use the most recent provided mapping by passing the `:ignore_conflicts` option
to `clojurewerkz.elastisch.index.update-mapping`:

{% gist %}

TBD

For more information, see [ElasticSearch guide on Put Mapping operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-put-mapping.html).


### Getting Mapping Types

To retrieve information about an existing mapping type, use the `clojurewerkz.elastisch.index.get-mapping` function:

{% gist %}

TBD


### Deleting Mapping Types

To delete an existing mapping type, use the `clojurewerkz.elastisch.index.delete-mapping` function:

{% gist %}

TBD


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


## Document Versioning

TBD


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
