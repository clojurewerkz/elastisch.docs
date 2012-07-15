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

This guide covers Elastisch 1.0.0-alpha4 and later 1.0.x preview releases.



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

    [clojurewerkz/elastisch "1.0.0-alpha4"]

### With Maven

Add Clojars repository definition to your `pom.xml`:

{% gist 65642c4b53d26539e5f6 %}

And then the dependency:

    <dependency>
      <groupId>clojurewerkz</groupId>
      <artifactId>elastisch</artifactId>
      <version>1.0.0-alpha4</version>
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

TBD



## Querying

### Overview

TBD


### Performing queries

TBD


### Different kinds of queries

TBD



## Wrapping up

Congratulations, you now can use Elastisch to work with ElasticSearch. Now you know enough to start building a real application.

We hope you find Elastisch reliable, consistent and easy to use. In case you need help, please ask on the [mailing list](https://groups.google.com/forum/#!forum/clojure-elasticsearch)
and follow us on Twitter [@ClojureWerkz](http://twitter.com/ClojureWerkz).


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Elastisch mailing list](https://groups.google.com/forum/#!forum/clojure-elasticsearch)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the
documentation better.
