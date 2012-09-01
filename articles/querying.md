---
title: "Elastisch, a minimalistic Clojure client for ElasticSearch: Querying"
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

More Like This (MLT) query find documents that are “like” provided text by running it against one or more fields.

With Elastisch, More Like This query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/mlt-query.html):

{% gist 2fef523a3ca1c69aae88 %}

Elastisch provides a helper function for constructing MLT queries, `clojurewerkz.elastisch.query/mlt`:

{% gist 4f5f1706cd9a551717b8 %}


### More Like This Field Query

More Like This Field is very similar to the More Like This query but operates on a single field.

With Elastisch, More Like This query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/mlt-field-query.html):

{% gist 72e15aee431b68c3f810 %}

Elastisch provides a helper function for constructing MLT field queries, `clojurewerkz.elastisch.query/mlt-field`:

{% gist f80dfba2a947b9a55f18 %}


### Fuzzy Query

A fuzzy based query that uses similarity based on Levenshtein (edit distance) algorithm. **Warning**: this query uses prefix length of 0 by default.
This will cause a full scan on all terms.

With Elastisch, fuzzy query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/fuzzy-query.html):

{% gist f32b5e41e76b6f7bbb32 %}

Elastisch provides a helper function for constructing fuzzy queries, `clojurewerkz.elastisch.query/fuzzy`:

{% gist b1e21f160ea61b437ad3 %}


### Fuzzy Like This Query

A cross between Fuzzy and More Like This queries.

With Elastisch, Fuzzy Like This (FLT) query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/fuzzy-like-this-query.html):

{% gist 5b8c3ef82a712170dd9f %}

Elastisch provides a helper function for constructing FLT queries, `clojurewerkz.elastisch.query/fuzzy-like-this`:

{% gist 14623996faad7ffb5181 %}


### Fuzzy Like This Field Query

Same as FTL query but works over a single field.

With Elastisch, Fuzzy Like This Field query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/fuzzy-like-this-field-query.html):

{% gist a87993ae1252816587a6 %}

Elastisch provides a helper function for constructing Fuzzy Like This Field queries, `clojurewerkz.elastisch.query/fuzzy-like-this-field`:

{% gist 4b98ff3ca587cc39ab15 %}


### Nested Query

Nested query allows to query nested objects/documents. The query is executed against the nested objects as if they were indexed as separate docs. The root parent document is returned.

With Elastisch, Nested query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/nested-query.html):

{% gist 25d3ab79e99a350da211 %}

Elastisch provides a helper function for constructing nested queries, `clojurewerkz.elastisch.query/nested`:

{% gist ca3e5be53b883e2877bb %}

`clojurewerkz.elastisch.query/nested` can be used in combination with other query helpers, such as `clojure.elastisch.query/term`, because they just return maps:

{% gist 769f9b5af752aa7e39e1 %}


### Span First Query

Matches spans near the beginning of a field.

With Elastisch, Span First query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/span-first-query.html):

{% gist 4c357f27e77065a81a57 %}

Elastisch provides a helper function for constructing Span First queries, `clojurewerkz.elastisch.query/span-first`:

{% gist a20ce931cd077c9a6498 %}


### Span Near Query

Matches spans which are near one another. One can specify slop, the maximum number of intervening unmatched positions, as well as whether matches are required to be in-order.

With Elastisch, Span Near query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/span-near-query.html):

{% gist 2efd07d4f12ea16328e6 %}

Elastisch provides a helper function for constructing Span Near queries, `clojurewerkz.elastisch.query/span-near`:

{% gist e7800727165abadbf3ff %}


### Span Not Query

Removes matches which overlap with another span query.

With Elastisch, Span Not query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/span-not-query.html):

{% gist 55c1a104d248ad9d4788 %}

Elastisch provides a helper function for constructing Span Not queries, `clojurewerkz.elastisch.query/span-not`:

{% gist 69251788c08d40fcff59 %}


### Span Or Query

Matches the union of its span clauses.

With Elastisch, Span Or query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/span-or-query.html):

{% gist f88c79c93cfde4f2787f %}

Elastisch provides a helper function for constructing Span Or queries, `clojurewerkz.elastisch.query/span-or`:

{% gist b11b07ececa627e1df5c %}


### Span Term Query

Matches spans containing a term.

With Elastisch, Span Term query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/span-term-query.html):

{% gist 383847bde046f755b6d2 %}

Elastisch provides a helper function for constructing Span Term queries, `clojurewerkz.elastisch.query/span-term`:

{% gist 451cc9f58f01e4edfefd %}


### Indices Query

Indices Query executes different queries against different indexes and combine the results.

With Elastisch, indices query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/indices-query.html):

{% gist 576f5e363f8d2be42dc3 %}

Elastisch provides a helper function for constructing fuzzy queries, `clojurewerkz.elastisch.query/indices`:

{% gist 736c8f60b263b63a063f %}


### Top Children Query

Top Children Query performs a query against child documents and aggregates scores of hits into the parent document.

With Elastisch, Top Children query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/top-children-query.html):

{% gist 6d476c53c33d4730a369 %}

Elastisch provides a helper function for constructing Top Children queries, `clojurewerkz.elastisch.query/top-children`:

{% gist ba42a803ce583f5fa6fc %}


### Has Child Query

Has Child Query returns parent documents that have child documents of the given type.

With Elastisch, Has Child query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/has-child-query.html):

{% gist 813f9485295d94c327a8 %}

Elastisch provides a helper function for constructing Has Child queries, `clojurewerkz.elastisch.query/has-child`:

{% gist a299331a201bb0a52e23 %}


### Constant Score Query

A query that wraps a filter or another query and simply returns a constant score equal to the query boost for every document in the filter.
The filter object can hold only filter elements, not queries. Filters can be much faster compared to queries since they don’t perform any scoring,
especially when they are cached.

With Elastisch, Constant Score query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/constant-score-query.html):

{% gist 1a4f25dc39d51dc0c8b7 %}

Elastisch provides a helper function for constructing Constant Score queries, `clojurewerkz.elastisch.query/constant-score`:

{% gist b843b4789c03d96a7f50 %}


### Custom Score Query

Custom Score query allows to wrap another query and customize the scoring of it by providing a script expression.

With Elastisch, Custom Score query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/custom-score-query.html):

{% gist 7f9e08572220f5aacd33 %}

Elastisch provides a helper function for constructing Custom Score queries, `clojurewerkz.elastisch.query/custom-score`:

{% gist 00ad20065bd8989e59f0 %}


### Custom Filters Score Query

A Custom Filters Score query allows to execute a query, and if the hit matches a provided filter (ordered), use either a boost or a script associated with it to compute the score.
This kind of query allows for very efficient parametrized scoring because filters do not perform any scoring and their results can be cached.

With Elastisch, Custom Filters Score query structure is the same as described in the [ElasticSearch query DSL documentation](http://www.elasticsearch.org/guide/reference/query-dsl/custom-filters-score-query.html):

{% gist b2bef712ad02f3d5f35a %}

Elastisch provides a helper function for constructing Custom Filters Score queries, `clojurewerkz.elastisch.query/custom-filter-score`:

{% gist 523f57fbd3da9729deaa %}




## Filters

Often search results need to be filtered (scoped): for example, to make sure results only contain documents that belong to a particular user account
or organization. Such filtering conditions do not play any role in the relevance ranking and just used as a way of excluding certain documents
from search results.

*Filters* is an ElasticSearch feature that lets you decide what documents should be included or excluded from a results of a query. Filters are similar
to the way term queries work but because filters do not participate in document ranking, they are significantly more efficient. Furthermore,
filters can be cached, improving efficiency even more.

To specify a filter, pass the `:filter` option to `clojurewerkz.elastisch.rest.document/search`:

{% gist e9c68d7d524f9d8457d5 %}

ElasticSearch provides many filters out of the box.

### Term and Terms Filter

Term filter is very similar to the Term query covered above but like all filters, does not contribute to relevance scoring and is more
efficient. Terms filter works the same way but for multiple terms.

With Elastisch, term filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/term-filter.html):

{% gist 52f799181e03bb8cda0c %}

### Range Filter

Range filter filters documents out on a range of values, similarly to the Range query. Supports numerical values, dates and strings.

With Elastisch, range filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/range-filter.html):

{% gist c69991390c1dff760cdb %}

### Exists Filter

Exists filter filters documents that have a specific field set. This filter always uses caching.

With Elastisch, Exists filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/exists-filter.html):

{% gist 67e6d4dddd934250a9ae %}

### Missing Filter

Exists filter filters documents that do not have a specific field set, that is, the opposite of the Exists filter.

With Elastisch, Missing filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/missing-filter.html):

{% gist 4a678dc47e4b64206f01 %}

### And Filter

The And filter matches documents using `AND` boolean operator on multiple subqueries. This filter does not use caching by default.

With Elastisch, And filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/and-filter.html):

{% gist a17a7078612e747f1920 %}

### Or Filter

The Or filter is similar to the And filter but matches documents using `OR` boolean operator on multiple subqueries. It does not use caching by default.

With Elastisch, Or filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/or-filter.html):

{% gist 7423a83e6cbeeb0e86b7 %}

### Not Filter

The Not filter filters out document that match its subquery. This filter does not use caching by default.

With Elastisch, Not filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/not-filter.html):

{% gist 35b76f5b9d6ef3bae843 %}

### Bool Filter

The Bool filter matches documents using boolean combinations of its subqueries. Similar in concept to the Boolean query, except that the clauses are other filters.

With Elastisch, Bool filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/bool-filter.html):

{% gist 4726c7239c8ea0e78c28 %}

### Limit Filter

The Limit filter limits the number of documents (per shard) that are taken for ranking.

With Elastisch, Limit filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/limit-filter.html):

{% gist 551b47a33ae02d377c9e %}

### Type Filter

This filter filters out documents based on their `_type` field value. It can work even if the `_type` field is not indexed.

With Elastisch, Type filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/type-filter.html):

{% gist a2e743b1c2d52c2317bb %}

### Prefix Filter

This filter matches documents with fields that have terms starting with the given prefix (**not analyzed**).

With Elastisch, Prefix filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/prefix-filter.html):

{% gist 9302d37b20d4731feba1 %}

### Query Filter

TBD

With Elastisch, And filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/query-filter.html):

{% gist  %}

### Geo Distance Filter

TBD

With Elastisch, And filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/geo-distance-filter.html):

{% gist  %}

### Geo Distance Range Filter

TBD

With Elastisch, And filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/geo-distance-range-filter.html):

{% gist  %}


### Geo Polygon Filter

TBD

With Elastisch, And filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/geo-distance-range-filter.html):

{% gist  %}


### Geo Bounding Box Filter

TBD

With Elastisch, And filter structure is the same as described in the [ElasticSearch Filter documentation](http://www.elasticsearch.org/guide/reference/query-dsl/geo-distance-range-filter.html):

{% gist  %}


### Filter Caching

Filters are often good candidates for caching that improves their performance further. Some filter types use caching by default:

 * term/terms
 * prefix
 * range
 * exists

Others are not cached by default:

 * numeric_range
 * script
 * various geo filters
 * compound filters (and, or, not)

It is possible to use `_cache` and `_cache_key` parameters to control caching behavior: disable caching or use custom cache key.

For more information, see [Filters and Caching](http://www.elasticsearch.org/guide/reference/query-dsl/)in ElasticSearch documentation.



## Highlighting

Having search matches highlighted in the UI is very useful in many cases. ElasticSearch can highlight matched in search results. To enable highlighting,
use the `:highlight` option `clojurewerkz.elastisch.rest.document/search` accepts. In the example above, search matches in the `biography` field will
be highlighted (wrapped in `em` tags) and search hits will include one extra "virtual" field called `:highlight` that includes the highlighted fields and the highlighted fragments
that can be used by your application.

{% gist ae91b88ca45fa83bf417 %}

More examples can be found in this [ElasticSearch documentation section on retrieving subsets of fields](http://www.elasticsearch.org/guide/reference/api/search/highlighting.html).
`:highlight` values that Elastisch accepts are structured exactly the same as JSON documents in that section.

TBD: examples, cover more features





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
