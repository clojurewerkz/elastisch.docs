---
title: "Elastisch, a minimalistic Clojure client for ElasticSearch: Getting Started with ElasticSearch and Clojure"
layout: article
---

## About this guide

This guide covers ElasticSearch search capabilities in depth, explains how Elastisch presents them in the API and how some of the key features are commonly used. This guide covers:

 * An overview of ElasticSearch search features
 * How to perform queries with Elastisch
 * How to work with responses
 * Different kinds of queries
 * How to use highlighting
 * Other topics related to querying

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a> (including images & stylesheets). The source is available [on Github](https://github.com/clojurewerkz/elastisch.docs).


## Overview

The whole point of a search server is to be able to run search queries against it and ElasticSearch has a lot to offer in this area.

ES supports multiple kinds of queries plus **filters**, which can be roughly thought as conditions in the `WHERE ...` clause in SQL.
Filters do not perform relevance scoring, only decide whether a particular document should be included or excluded from final
results. Because there is no relevance calculation involved, they are more efficient than compound queries with additional
conditions. A good use case for filters is filtering out results just for one customer (or organization, or any related group of documents).

"Full text" queries are analyzed just like documents are during indexing. ElasticSearch is distributed and lets users control request
routing on per-query basis.

Queries are submitted to ElasticSearch as JSON documents with a certain structure. With Elastisch, you can either use Clojure maps
that have exactly the same structure, or use a few helpful functions that make queries a little bit more concise. In all cases,
whenever a query requires nesting maps, Elastisch uses exactly the same structure as described in [ElasticSearch documentation on
query DSL](http://www.elasticsearch.org/guide/reference/query-dsl/).


## Performing queries

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

To search against multiple indexes or mappings, pass them as vectors to their respective function arguments:

{% gist 51b9bab4f00a6e533c9d %}


## Checking results

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


## Different kinds of queries

ElasticSearch is a feature rich search engine and it supports many types of queries. Even though
all queries can be passed as Clojure maps, it is common to use convenient functions from the
`clojurewerkz.elastisch.query` to construct queries.

### Term and Terms Queries

The Term query is the most basic query type. It matches documents that have a particular term.
A common use case for term queries is looking up documents with unanalyzed identifiers
such as usernames.

A close relative of the term query is the **terms query** which works the same way but takes multiple term values.

With Elastisch, term query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/term-query.html):

{% gist 2a66899e2b200cca499f %}

Elastisch provides a helper function for constructing term queries, `clojurewerkz.elastisch.query/term`:

{% gist 2d7b9e0b59f728a35751 %}

If provided values is a collection, Elastisch will construct a terms query under the hood.


### Text Query

The Text query accepts text, analyzes it and performs a boolean (the default), fuzzy or phrase query. Each query type takes additional options.
Depending on the exact "subtype", query structure will be slightly different so it is common to pass queries as Clojure maps instead of using the
helper function (described below).

The difference with field and query string query types is that text queries do not support field prefixes and do not attempt to Lucene Query syntax parsing.
As such, text queries can be seen as more limited and slighly more efficient kind of the query string query, suitable for many apps with non-technical
audience.

With Elastisch, text query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/text-query.html):

{% gist 6f7e1ee014c55feefbba %}

Elastisch provides a helper function for constructing boolean text queries, `clojurewerkz.elastisch.query/text`:

{% gist aa2ba7b948ace3662441 %}


### Query String Query

The Query String (QS) query accepts text, runs it through Lucene Query Language parser, analyzes it and performs query. It is the most advanced query
type with

The difference with field and query string query types is that text queries do not support field prefixes and do not attempt to Lucene Query syntax parsing.
As such, QS queries can be seen as more powerful and less efficient kind of text query, suitable for many apps with technical audience.

With Elastisch, QS query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/query-string-query.html):

{% gist f87ff598ff8dd84810a3 %}

Elastisch provides a helper function for constructing QS text queries, `clojurewerkz.elastisch.query/query-string`:

{% gist 43edc61a6bd0e31460c6 %}

For all the numerous options this query type accepts, see [ElasticSearch documentation on the subject](http://www.elasticsearch.org/guide/reference/query-dsl/query-string-query.html).


### Range Query

The Range query returns documents with fields that have numerical values, dates or terms within a specific range. One example of such query is retrieving
all documents where the `:date` field value is earlier than a particular moment in time, say, `"20120801T160000+0100"`.

Range queries work for numerical values and dates the way you would expect. For string and text fields, they match all documents with terms in the given range
(for example, `"cha"` to `"cze"`) in a particular field.

With Elastisch, range query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/range-query.html):

{% gist c0603c48788f3285001c %}

Elastisch provides a helper function for constructing range queries, `clojurewerkz.elastisch.query/range`:

{% gist 1ea880308ba360b36183 %}


### Boolean Query

A query that matches documents matching boolean combinations of other queries. It is built using one or more boolean clauses, each clause with a typed occurrence.
The occurrence types are documented on the [ElasticSearch page on boolean queries](http://www.elasticsearch.org/guide/reference/query-dsl/bool-query.html).

With Elastisch, boolean query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/bool-query.html):

{% gist 6973895e39dc8181f1f3 %}

Elastisch provides a helper function for constructing boolean queries, `clojurewerkz.elastisch.query/bool`:

{% gist 5dc9396bf16f0a5efca7 %}

`clojurewerkz.elastisch.query/bool` can be used in combination with other query helpers, such as `clojure.elastisch.query/term`, because they just return maps:

{% gist 50f7145c0c48c163151c %}


### Filtered Query

A query that applies a filter to the results of another query. Use it if you need to narrow down
results of an existing query efficiently but the condition you filter on does not affect relevance
ranking. One example of that is searching over accounts that are active.

With Elastisch, filtered query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/filtered-query.html):

{% gist 0467bff740c8219aabb1 %}

Elastisch provides a helper function for constructing filtered queries, `clojurewerkz.elastisch.query/filtered`:

{% gist f513a1533127b9848d11 %}

`clojurewerkz.elastisch.query/filtered` can be used in combination with other query helpers, such as `clojure.elastisch.query/term`, because they just return maps:

{% gist 08aecbf66de81a507537 %}


### Field Query

A query that executes a query string against a specific field. It is a simplified version of query_string query (it is equivalent to setting the `:default_field`
to the field this query executed against).

With Elastisch, field query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/field-query.html):

{% gist 57aa407c36bb7fbb0f29 %}

Elastisch provides a helper function for constructing field queries, `clojurewerkz.elastisch.query/field`:

{% gist 6e785d8bc947328ad18e %}


### Prefix Query

The Prefix query is similar to the term query but matches documents that have at least one term that begins with the given prefix.
One use case for prefix queries is providing text autocompletion results (works best for non-analyzed fields).

With Elastisch, prefix query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/prefix-query.html):

{% gist cac16c43a7357ba0fc8c %}

Elastisch provides a helper function for constructing prefix queries, `clojurewerkz.elastisch.query/prefix`:

{% gist b7c36240e9fa50cf813c %}


### Wildcard Query

The Wildcard query is a generalized version of Prefix query and usually is applicable in the same cases. Note that wildcard suffix queries such as `"*werkz"` have
very poor performance characteristics on large data sets.

With Elastisch, wildcard query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/wildcard-query.html):

{% gist 731724133c9ca66dc1a3 %}

Elastisch provides a helper function for constructing wildcard queries, `clojurewerkz.elastisch.query/wildcard`:

{% gist f613455403c88f045a72 %}


### IDs Query

The IDs query is searches for documents by their IDs (`:_id` field values). It is similar to `WHERE ... IN (...)` in SQL.

With Elastisch, IDs query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/ids-query.html):

{% gist a1da13c2614bd1a9c503 %}

Elastisch provides a helper function for constructing IDs queries, `clojurewerkz.elastisch.query/ids`:

{% gist dd15ca01686789328a73 %}


### Match All Query

The Match All query does what it sounds like: matches every single document in the index. Used almost exclusively during development or
in combination with other queries in compound queries (e.g. filtered).

With Elastisch, match-all query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/match-all-query.html):

{% gist bd0eb5a9b49f17a82592 %}

Elastisch provides a helper function for constructing match-all queries, `clojurewerkz.elastisch.query/match-all`:

{% gist b552bbb10de1753e8ba4 %}

An example of match-all query being used as part of a filtered query to find all people in a particular age bracket:

{% gist f99031c42bc8a21ad952 %}


### Dis-Max Query

Dis-Max or Disjunction Max query is a compound query where only the max score clause is used for ranking (as opposed to boolean queries, where scores are combined).
This is useful when searching for a word in multiple fields with different boost factors (so that the fields cannot be combined equivalently into a single search field).

Like other compound queries, Dis-Max returns the union of documents produced by subqueries.

With Elastisch, dis-max query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/dis-max-query.html):

{% gist 3f621675b0a98cf6b849 %}

Elastisch provides a helper function for constructing dis-max queries, `clojurewerkz.elastisch.query/dis-max`:

{% gist 77dac195ced1a821f302 %}

`clojurewerkz.elastisch.query/dis-max` can be used in combination with other query helpers, such as `clojure.elastisch.query/field`, because they just return maps:

{% gist 7e90251ffa0827975db8 %}


### Boosting Query

Boosting queries are used to demote results that match a particular query. For example, when searching for `"Berlin"`, most likely the intent is to find
information about Berlin in Germany, not one of the towns in North America. Boosting queries can be used to lower relevancy of some documents without
affecting scoring of the most.

With Elastisch, boosting query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/boosting-query.html):

{% gist d55b315dea4067315cba %}

Elastisch provides a helper function for constructing boosting queries, `clojurewerkz.elastisch.query/boosting`:

{% gist 5302e6f33b381bae8672 %}

`clojurewerkz.elastisch.query/boosting` can be used in combination with other query helpers, such as `clojure.elastisch.query/term`, because they just return maps:

{% gist b8ec660229692bf4b1ea %}


### More Like This Query

TBD


### More Like This Field Query

TBD


### Fuzzy Query

TBD


### Fuzzy Like This Query

TBD


### Fuzzy Like This Field Query

TBD


### Nested Query

TBD


### Span First Query

TBD


### Span Near Query

TBD


### Span Not Query

TBD


### Span Or Query

TBD


### Span Term Query

TBD


### Indices Query

Indices Query executes different queries against different indexes and combine the results.

TBD


### Top Children Query

TBD


### Has Child Query

TBD


### Constant Score Query

TBD


### Custom Score Query

TBD


### Custom Filter Score Query

TBD


## Query Examples

### Term Query

Given an index with the following mapping type:

{% gist 2832fa205ad1ea8044b2 %}

and indexed documents

{% gist bb5a6ca3179114ed40e8 %}

The following term query

{% gist 2d7b9e0b59f728a35751 %}

Will return the 1st document in hits and

{% gist 34ce76ceab1628c17370 %}

will also return the 1st document.


### Prefix Query

TBD



## Filters

TBD


## Highlighting

TBD


## Wrapping Up

ElasticSearch querying capabilities are just as rich as the indexing ones. With multiple kinds of queries, filtering, ability to query multiple indexes or
mapping types at once and features like ad-hoc boosting, you have plenty of tools and knobs for making search work exactly the way your domain model requires.

Elastisch follows ElasticSearch REST API structure (for example, the query DSL) and is strives to be as feature complete as possible when it comes to querying.


## What to Read Next

The documentation is organized as [a number of guides](/articles/guides.html), covering different topics in depth:

 * [Facets](/articles/facets.html)
 * [Percolation](/articles/percolation.html)
 * [Routing and Distribution](/articles/distribution.html)


## Tell Us What You Think!

Please take a moment to tell us what you think about this guide on Twitter or the [Elastisch mailing list](https://groups.google.com/forum/#!forum/clojure-elasticsearch)

Let us know what was unclear or what has not been covered. Maybe you do not like the guide style or grammar or discover spelling mistakes. Reader feedback is key to making the
documentation better.
