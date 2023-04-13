---
layout: default
title: Search relevance Stats API
nav_order: 65
parent: Search relevance
has_children: false
---

# Search Relevance Stats API
Introduced 2.7
{: .label .label-purple }

The Search Relevance Stats API provides information about the operations of the Search Relevance plugin. The Search Relevance plugin processes operations sent by the [Compare Search Results]({{site.url}}{{site.baseurl}}/search-plugins/search-relevance) Dashboards tool.

#### Example request

The following example request retrieves statistics for the Search Relevance plugin:

```json
GET /api/relevancy/stats
```
{% include copy-curl.html %}

#### Example response

The following is the response for the preceding request:

```json
{
  "data": {
    "search_relevance": {  
      "fetch_index": {
        "200": {
          "sum": 12.02572301030159,
          "count": 1
        }
      },
      "single_search": {
        "200": {
          "sum": 4.898337006568909,
          "count": 1
        }
      }
    }
  },
  "overall": {
    "response_time_avg": 8.46203000843525,
    "requests_per_second": 0.03333333333333333
  },
  "counts_by_component": {
    "search_relevance": 2
  },
  "counts_by_status_code": {
    "200": 2
  }
}
```

## Response fields

The following table lists all response fields.

| Field | Data type | Description |
| :--- | :--- | :--- | 
| [`data.search_relevance`](#the-datasearch_relevance-object) | Object | Statistics related to the Search Relevance operations. |
| `overall` | Object | The average statistics for all operations. |
| `overall.response_time_avg` | Double | The average response time for all operations, in milliseconds. |
| `overall.requests_per_second` | Double | The average number of requests per second for all operations. |
| `counts_by_component` | Object | The sum of all `count` values for all child objects of the `data` object. |
| `counts_by_component.search_relevance` | The total number of responses for all operations in the `search_relevance` object. |
| `counts_by_status_code` | Object | Contains a list of all response codes with their counts for all Search Relevance operations. |

### The `data.search_relevance` object

The `data.search_relevance` object contains the fields described in the following table.

| Field | Data type | Description |
| :--- | :--- | :--- |
| `comparison_search` | Object | Statistics related to the comparison search operation. A comparison search operation is a request to compare two queries when both Query 1 and Query 2 are filled out in the Compare Search Results tool. |
| `single_search` | Object | Statistics related to the single search operation. A single search operation is a request to run a single query when only one of Query 1 and Query 2 is filled out in the Compare Search Results tool. |
| `fetch_index` | Object | Statistics related to the operation of fetching the index or indexes for a comparison search or single search. |

Each of the `comparison_search`, `single_search`, and `fetch_index` objects contains a list of HTTP response codes. The following table lists the fields for each response code.

| Field | Data type | Description |
| :--- | :--- | :--- |
| `sum` | Double | The sum of the response times for the responses with this HTTP code, in milliseconds. |
| `count` | Integer | The total number of responses with this HTTP code.  |