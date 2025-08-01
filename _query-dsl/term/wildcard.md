---
layout: default
title: Wildcard
parent: Term-level queries
nav_order: 90
---

# Wildcard query

Use wildcard queries to search for terms that match a wildcard pattern. Wildcard queries support the following operators.

Operator | Description
:--- | :---
`*` | Matches zero or more characters.
`?` | Matches any single character.
`case_insensitive` | If `true`, the wildcard query is case insensitive. If `false`, the wildcard query is case sensitive. Default is `false` (case sensitive).

For a case-sensitive search for terms that start with `H` and end with `Y`, use the following request:

```json
GET shakespeare/_search
{
  "query": {
    "wildcard": {
      "speaker": {
        "value": "H*Y",
        "case_insensitive": false
      }
    }
  }
}
```
{% include copy-curl.html %}

If you change `*` to `?`, you get no matches because `?` refers to a single character.

Wildcard queries tend to be slow because they need to iterate over a lot of terms. Avoid placing wildcard characters at the beginning of a query because it could be a very expensive operation in terms of both resources and time.

The [wildcard field type]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/wildcard/) builds an index that is specially designed to be very efficient for wildcard and regular expression queries.

## Parameters

The query accepts the name of the field (`<field>`) as a top-level parameter:

```json
GET _search
{
  "query": {
    "wildcard": {
      "<field>": {
        "value": "patt*rn",
        ...
      }
    }
  }
}
```
{% include copy-curl.html %}

The `<field>` accepts the following parameters. All parameters except `value` are optional.

Parameter | Data type | Description
:--- | :--- | :---
`value` | String | The wildcard pattern used for matching terms in the field specified in `<field>`.
`boost` | Floating-point | A floating-point value that specifies the weight of this field toward the relevance score. Values above 1.0 increase the field’s relevance. Values between 0.0 and 1.0 decrease the field’s relevance. Default is 1.0.
`case_insensitive` | Boolean | If `true`, allows case-insensitive matching of the value with the indexed field values. Default is `false` (case sensitivity is determined by the field's mapping).
`rewrite` | String | Determines how OpenSearch rewrites and scores multi-term queries. Valid values are `constant_score`, `scoring_boolean`, `constant_score_boolean`, `top_terms_N`, `top_terms_boost_N`, and `top_terms_blended_freqs_N`. Default is `constant_score`.

If [`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/#expensive-queries) is set to `false`, then wildcard queries are not executed.
{: .important}
