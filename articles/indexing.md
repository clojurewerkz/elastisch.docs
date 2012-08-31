---
title: "Elastisch, a minimalistic Clojure client for ElasticSearch: Indexing, Analysis, built-in Analyzers"
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


## Indexing Documents

There are two ways to index a document with ElasticSearch: submit it for indexing without the id or update a document with
a provided id, in which case if the document already exists, it will be updated (a new version will be created).

Document ids are internal to ES and don't have to match identifiers in other data stores you may be using side by side
with ElasticSearch.

While it is fine and common to use automatically created indexes early in development, manually creating indexes lets you configure
a lot about how ElasticSearch will index your data and, in turn, what kind of queries it will be possible to execute against it.

### Creating Documents

To submit a document and have its id generated, use the `clojurewerkz.elastisch.rest.document/create` function. It takes an index name,
a mapping type (more on mappings later in this guide) and the document you want to be indexed, as a Clojure map:

{% gist 0f22eeb2ff459e37ecd8 %}

It returns the response as a Clojure map that contains whether the response was successful, the generated document id,
and so on. `clojurewerkz.elastisch.rest.response/ok?` is a predicate function that should be used to verify that the
response is was successful.

If the index does not exist, it will be automatically created.

TBD: the `:op_type` option.


### Updating Documents

`clojurewerkz.elastisch.rest.document/put` works a lot like `clojurewerkz.elastisch.rest.document/create` but it requires document id
to be passed as the 3rd argument  and can be used to update existing documents (the common "put-if-absent" idiom):

{% gist 4903ddfd635e08b3b49a %}

ElasticSearch will version documents when they are updated by default. More on this later in this guide.


### Updating With Scripts

TBD


## Creating Indexes

ElasticSearch will create an index the first time it is used but it is also possible to precreate an index with
specific settings, mappings, and so on with the `clojurewerkz.elastisch.rest.index/create` function. In the simplest case,
it only takes index name (a string) as the only argument:

{% gist af984cb5bc86339bcaae %}

It is also possible to define settings and mapping types at the time an index is created. Just pass `:settings` and `:mappings` options:

{% gist 0e462a2bdcd2e744bb57 %}

{% gist 917d482b6b017e626302 %}


### Index Settings

Index settings let you control many aspects of how ElasticSearch will use an index, store it, update it and so on.
ElasticSearch documentation on index settings uses dot separated names for nested setting map attributes.

For example, to set `index.refresh_interval` to 10 seconds, pass the following map for the `:settings` key:

{% gist 1c524ff9ea227da6df34 %}

Index settings can be updated for an existing index using the `clojurewerkz.elastisch.rest.index/update-settings` function, as
described later in this guide.

For the reference list of index settings, see

 * [Update Index Settings Operation guide](http://www.elasticsearch.org/guide/reference/api/admin-indices-update-settings.html)
 * [Index modules guide](http://www.elasticsearch.org/guide/reference/index-modules/)



## Overview of Mapping Types

ElasticSearch has the concept of **mappings** that define which fields in documents are indexed, if/how they are analyzed and if they are stored. Each index in
ElasticSearch may have one or more **mapping types**. Mapping types can be thought of as tables in a database (although this analogy does not always stand).
Mapping types is the heart of indexing in ElasticSearch and provide access to a lot of ElasticSearch functionality.

For example, a blogging application may have types such as "article", "comment" and "person". Each has distinct **mapping settings** that define a set of fields
documents of the type have, how they are supposed to be indexed (and, in turn, what kind of queries will be possible over them), what language each field is in
and so on. Getting mapping types right for your application is the key to good search experience. It also takes time and experimentation.

Searches can be performed across multiple types or a single type across multiple indexes.


## Creating Mapping Types

Mapping types can specified when an index is created using the `:mappings` option:

{% gist 917d482b6b017e626302 %}

It is possible to update mapping types after they are created, as described later in this guide.


## Mapping Type Settings

### Overview

Mapping types define document fields and of what core types (e.g. string, integer or date/time) they are. Settings are provided to ElasticSearch as a JSON document
and this is how they are [documented on the ElasticSearch site](http://www.elasticsearch.org/guide/reference/mapping/), for example:

{% gist 714418e5b21a0d7b439c %}

With Elastisch, mapping settings are specified as Clojure maps. A very minimalistic example:

{% gist 3423279800b78b313fbf %}

Here is a brief and very incomplete list of things that you can define via mapping settings:

 * Document fields, their types, whether they are analyzed
 * Document time-to-live (TTL)
 * Whether document type is indexed
 * Special fields (`"_all"`, default field, etc)
 * [Document-level boosting](http://www.elasticsearch.org/guide/reference/mapping/boost-field.html)
 * [Timestamp field](http://www.elasticsearch.org/guide/reference/mapping/timestamp-field.html)

When an index is created using the `clojurewerkz.elastisch.rest.index/create` function, mapping settings are passed with the `:mappings` option:

{% gist 917d482b6b017e626302 %}

When it is necessary to update mapping for an indexing index with the `clojurewerkz.elastisch.rest.index/update-mapping` function, they are passed as a positional argument:

{% gist f8d296a40c496d793444 %}


### Defining Fields

Settings are passed as maps where keys are names (strings or keywords) and values are maps of the actual settings. In this example, the only setting is
`:properties` which defines a single field which is a string that is not analyzed:

{% gist 3423279800b78b313fbf %}

Next lets take a look at a more realistic example of the tweet type where we have both username and text, and text is analyzed:

{% gist 41c33566daaf7bb996a1 %}

The second field has the same core type (string) and specifies an analyzer we want ElasticSearch to use for this field. Different types of analyzers
are described later in this guide. Note that the default value of the `:analyzer` field is `"default"`, so in this example it could have been omitted.

In the example below the same tweet type is extended with one more field, `:timestamp`:

{% gist 0696da93ad048c545762 %}

Because `:timestamp` is a date and there are multiple date formats in use, we specify which particular format will be used by our application: `"basic_date_time_no_millis"`.
An example timestamp in this format looks like this: `"20120802T101232+0100"`, generalized version is `"yyyyDDD’T’HHmmssZ"`. [ElasticSearch supports multiple date/time formats](http://www.elasticsearch.org/guide/reference/mapping/date-format.html).

The `:include_in_all` setting instructs ElasticSearch to not include timestamps in the special `"_all"` field (described later in this document).

Another common type of field is integer:

{% gist 05c2593eb8bb3630e14a %}

Boolean fields are also very common and supported by ElasticSearch:

{% gist 197c42bad2a448f12be2 %}

Here we see one more setting in action, `:boost`. Boost is a multipler that is applied to field score during document scoring. It lets developer express
that matches in some fields (e.g. title) are more important than others (for example, metadata). In the previous example we also define default
boolean field value with the`:default` key.

ElasticSearch supports indexing and querying over nested documents (very much like document databases MongoDB and CouchDB):

{% gist db2a15ae779e94d68a98 %}

Location field in the example above is of type `"object"` and has its own set of `:properties`. It is possible to have one of those properties to
be of type `"object"` and have its own set of properties, and so on.


### Core ElasticSearch Field Types

So far we have demonstrated a few core field types:

 * string
 * date/time
 * integer
 * boolean
 * object

ElasticSearch documentation covers more [mapping/field types](http://www.elasticsearch.org/guide/reference/mapping/core-types.html).



## Getting Mappings

To retrieve information about an existing mapping type, use the `clojurewerkz.elastisch.rest.index/get-mapping` function:

{% gist a5b4dc1cea1d62fcd5aa %}

It is possible to fetch multiple mapping types in a single request by passing a collection (typically a vector) for the
second argument.

It is possible to specify a collection of indexes (typically a vector) or the special `"_all"` value to fetch mapping types
information for multiple (or all) indexes.


## Updating Mappings

It is possible to update an existing mapping type using the `clojurewerkz.elastisch.rest.index/update-mapping` function:

{% gist f8d296a40c496d793444 %}

It is possible to specify multiple indexes by passing a vector of names or the special value `"_all"` to update a mapping for all
existing indexes:

{% gist b9211ee31ed658056113 %}

### Mapping Conflicts

When a mapping already exists, defining it with different attributes may result in conflicts.
ElasticSearch will try to be reasonably smart and merge mapping definitions. However, in some cases (like if core type of a field has changed)
it is not possible to do. In that case, by default ElasticSearch will respond with an error (40x status code) and Elastisch will raise an exception.

It is possible to instruct ElasticSearch to ignore conflicts and simply use the most recent provided mapping by passing the `:ignore_conflicts` option
to `clojurewerkz.elastisch.rest.index.update-mapping`:

{% gist 9411e81621cde484b3bf %}

For more information, see [ElasticSearch guide on Put Mapping operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-put-mapping.html).



### Deleting Mappings

To delete an existing mapping type, use the `clojurewerkz.elastisch.rest.index/delete-mapping` function:

{% gist 7eab32561f3e61ffb24a %}

Deleting a mapping type **causes all its data/documents to be deleted**. Think of it as dropping a database table.


## Disabling Analysis for Fields

Analyzing fields during indexing makes many types of queries possible but also may result in some information being
split into tokens that don't have any real value on their own and won't be useful for the kind of queries performed
in an app. Some examples include usernames, URLs, filesystem paths, human names.

It is possible to disable analysis for a field. In that case, the field's value still will be searchable as a single
token (exact match, case sensitive).

To disable analysis for a field, set the `:index` setting for the field in index mapping:

{% gist d70854f334a656a8c868 %}

A more complete example:

{% gist 770dff256b6a0c34249d %}


## Stored Fields

In addition to being analyzed, field values can be stored in ElasticSearch. Stored fields then will be included in the results.
To instruct ElasticSearch to store values in a field, use the `:store` setting in the mapping:

{% gist b42785c708eafaa8fab4 %}

By default fields are not stored but the entire JSON document submitted for indexing is stored and can be (and often is) displayed
by applications in the UI.


## Built-in Analyzers

ElasticSearch supports multiple analyzers and it is possible to define custom ones, often without writing any code (just reusing
existing tokenizers and filters). This section briefly describes some of them. To experiment with
analysis and analyzers, you can use the [Analyze API operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-analyze.html) on
any existent index, for example, via tool like `curl`.

### Standard Analyzer

The most sophisticated built-in analyzer. Intelligent enough to handle (tokenize correctly) email addresses, most of organization names
and so on.

TBD: more details

### Whitespace Analyzer

Whitespace analyzer is very simplistic: it splits text into tokens on whitespace characters. Tokens are not lowercased. Can be useful for
lists of identifiers that are separated by spaces, if case sensitivity is not an issue.

### Simple Analyzer

Splits text into tokens on non-letter characters. Tokens are lowercased. *Discards numeric characters*.

### Stopword Analyzer

The same as simple analyzer but also removes [stop words](http://en.wikipedia.org/wiki/Stop_words).



### Defining Custom Analyzers

With ElasticSearch, it is possible to define custom analyzers without writing any code. Analyzers combine

 * A tokenizer, an entity that takes a string and produces **tokens**
 * Zero or more **token filters** that modify or remove tokens (similarly to how `clojure.core/filter`, `clojure.core/map` and `clojure.core/remove` functions
   work)
 * Char filters that modify inputs before tokenization (for example, replace `ß` with `ss` or strip HTML tags)

Analyzers are [configured](http://www.elasticsearch.org/guide/reference/index-modules/analysis/) using index settings:

{% gist 898c8a5813de6e280b30 %}

In the above example we define a custom analyzer that uses 3 predefined filters and a custom list of stop words. In addition, we instruct ElasticSearch to use this
custom analyzer for the `:text` field of the `:tweet` type in the mapping.

ElasticSearch has many [predefined analyzers, tokenizers and token filters to choose from](http://www.elasticsearch.org/guide/reference/index-modules/analysis/).


## Language-specific Analyzers

ElasticSearch offers a variety of [language-specific analyzers](http://www.elasticsearch.org/guide/reference/index-modules/analysis/lang-analyzer.html) that come with language-specific (e.g. Russian or Arabic or German) stop words:

 * arabic
 * armenian
 * basque
 * brazilian
 * bulgarian
 * catalan
 * chinese
 * cjk (Chinese, Japanese, Korean)
 * czech
 * danish
 * dutch
 * english
 * finnish
 * french
 * galician
 * german
 * greek
 * hindi
 * hungarian
 * indonesian
 * italian
 * norwegian
 * persian
 * portuguese
 * romanian
 * russian
 * spanish
 * swedish
 * turkish
 * thai

All analyzers support setting custom stopwords either via configuration, or by using an external stopwords file by setting `stopwords_path`.


## Choosing Analyzers For Common Use Cases

### Usernames, Ids, Zip Codes

For "keyword-like" fields like usernames, ZIP codes or other identifiers, it usually makes sense to make the field
non-analyzed. It will then be searchable by the exact case sensitive match. If case sensitivity is an issue, it is
possible to define a custom analyzer that uses the [lowercase token filter](http://www.elasticsearch.org/guide/reference/index-modules/analysis/lowercase-tokenfilter.html)
or your application can lowercase all values before submitting them to ElasticSearch for indexing.


### Email, URLs

Email and URLs can either be non-analyzed or use a custom analyzer that tokenizes emails and URLs as single tokens with the [UAX Email URL Tokenizer ](http://www.elasticsearch.org/guide/reference/index-modules/analysis/uaxurlemail-tokenizer.html).


### Human Names

If your application does not need sophisticated search capabilities like finding people named Shawn for the query "Sean", first and last name can be
good candidates for non-analyzed fields.

For phonetic name matching, there is a [phonetic analysis plugin for ElasticSearch](https://github.com/elasticsearch/elasticsearch-analysis-phonetic).


### Document Titles

Standard analyzer is usually a very good choice for document titles in most non-specialized applications. Some exceptions to that include technical or scientific
documents (for example, of biological or chemical nature). In that case, a custom analyzer may be necessary.


### Document Body

Standard analyzer is usually a very good choice for document titles in most non-specialized applications. Just like with title, technical or scientific
documents may need a custom analyzer.



## Document TTL (Time-to-Live)

ElasticSearch supports [Time-to-Live](http://www.elasticsearch.org/guide/reference/mapping/ttl-field.html) documents: it is possible to specify expiration period
on a document. It can either be done per mapping type via mapping definition, as demonstrated below:

{% gist 980374a17ba4e25632a5 %}

or on a per-document basis by including the `:_ttl` field in a document.

TBD: full examples


## Document Versioning

Each indexed document in ElasticSearch has a version number. It is accessible via the `:_version` key in the response to operations like `clojurewerkz.elastisch.rest.document/create`, `clojurewerkz.elastisch.rest.document/put`.

When you update a document, it is possible to specify the exact version being used.
This helps ensure that no data is lost due to concurrent updates of the same document in the read-modify-write update scenarios.

To do so, pass the `:version` option to `clojurewerkz.elastisch.rest.document/put`:

{% gist  %}

When reading to update, setting `:preference` to `"_primary"` helps ensure you won't get stale reads:

{% gist  %}


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


## Index Templates

ElasticSearch [index templates](http://www.elasticsearch.org/guide/reference/api/admin-indices-templates.html) let you specify settings and/or mappings
for indexes that match certain name patterns upfront. For example, in the case of one index per company in the app, it may be desirable to
use the same mapping types (schema) for all indexes. Because it is likely that the indexes are named the same way (e.g. `customer[identifier]`),
index templates are applied to indexes matching a certain naming pattern.

To create a template, use the `clojurewerkz.elastisch.rest.index/create-template` function:

{% gist 9f7fffcb78e719110c17 %}

To delete a template, there is `clojurewerkz.elastisch.rest.index/delete-template`:

{% gist 26770b5c2ffcfa4ea22a %}

Finally, `clojurewerkz.elastisch.rest.index/get-template` lets you retrieve information about an indexisting index template:

{% gist 3589cb1631f2782b3e45 %}




## Index Aliases

`clojurewerkz.elastisch.rest.index/update-aliases` and `clojurewerkz.elastisch.rest.index/get-aliases`
are new functions that implement support for [index aliases](http://www.elasticsearch.org/guide/reference/api/admin-indices-aliases.html):

{% gist f5ac6d24d74494f047a9 %}

It is possible to have multiple aliases for an index or one alias to refer to multiple indexes.


## Misc Topics

### How to Set Index Refresh Interval

Set the `index.refresh_interval` to a value like `"5s"`, `"30s"`, `"30m"` and so on when you create an index or update index settings:

{% gist 0a097f151376d37e0af4 %}

For example, to set `index.refresh_interval` to 10 seconds, pass the following map for the `:settings` key:

{% gist 1c524ff9ea227da6df34 %}



### The _all Field

[The `_all` field](http://www.elasticsearch.org/guide/reference/mapping/all-field.html) is a special document field that includes the content of one or more (possibly all) document fields combined.
It is helpful in cases when querying against documents with unknown document structure.

It is possible to disable the `_all` field for a mapping or exclude certain fields from being added to it. This is done
on the per mapping type basis, via mapping settings:

{% gist 81c050ecfbc288ea1b52 %}

When disabling the _all field, it is a good practice to set `index.query.default_field` to a different value
(for example, `:text` or `:body` in case of article-like documents).


### Default Query Field

To set default query field for a mapping, specify `index.query.default_field` in index settings:

{% gist 42eae254ac4cc5771923 %}


### Field Boosting

TBD


### Per-Document Boosting

TBD


### Testing How Text is Analyzed

It is possible to use the [ElasticSearch Analyze API operation](http://www.elasticsearch.org/guide/reference/api/admin-indices-analyze.html) to see how different
analyzers process (tokenize and filter) various pieces of text.


## Wrapping Up

The indexing process is a very important aspect of search. Get it right, and your application's search results will be useful. Get it wrong, it most likely they won't
be useful at all. ElasticSearch is very flexible and feature rich when it comes to indexing and how your data is analyzed. Support for custom and language-specific analyzers
will satisfy even the most demanding needs.

Elastisch follows ElasticSearch REST API structure and supports nearly all features related to indexing.


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
