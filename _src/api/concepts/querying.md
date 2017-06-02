---
title: Querying Collections
excerpt: Query collections using predefined or runtime filters, returning only the data you need
order: 9
---

* Querystring parameters
* Using a JSON query object
* Using an Aggregation Pipeline array

When querying a collection it is possible to override the default settings specified in the collection specification. Using the parameters below opens up the possibility of defining your business/domain logic within the API request itself.

Parameter       | Type        |  Description                                  | Default value        |  Example
:----------------|:------------|:----------------------------------------------|:---------------------|:--------------
count            | integer     | Maximum number of results to be returned   | 50                   | 10
page             | integer     | Page number                                   | 1                    | 2
skip             | integer     | The number of records to skip              | 0   | 3
sort             | string      | Field to sort on                          | `_id`                  | "title"
sortOrder       | string      | Sort direction                                | asc                  | desc
filter           | object        | MongoDB query object or Aggregation Pipeline array                            |                      | { fieldName: {"$in": ["a", "b"]}object}
fields           | object        | Specify the fields to return in the resultset.  |          | Include fields: {"field1":1,"field2":1} Exclude fields: {field2":0}
callback         | string      | Callback function to wrap the return result set in.  |               | thisIsMyCallback

## Querystring Parameters

#### count

Overrides the collection's `count` setting, specifying the maximum number of results to be returned.

#### page

Enables paging within the collection. Specifying a value for `page` along with `count` (or relying on the collection's default `count` setting) will utilise MongoDB's `skip()` method to skip the first (`page * count`) records in the collection.

#### skip

The `skip` value is normally calculated using the `count` and `page` values, so if `count = 10` and `page = 2` then `skip` becomes `10` (i.e. `(page-1) * count`). If `skip` is specified in the querystring, this value is added to the calculated value to avoid overlapping records on subsequent pages.

#### sort

Specifies the field to be used when sorting the collection. The default field is the collection's `_id` field.

#### sortOrder

Overrides the collection's `sortOrder` setting. Permitted values are `asc` for an ascending sort order and `desc` for a descending sort order.

#### fields

Specifies the fields to return. Extends the collection's `fieldLimiters` setting.

```
fields={"name":1,"email":1}
```

#### filter

Extends the collection's `defaultFilters` setting. There are two ways to use the `filter` parameter: passing a JSON query object for performing a standard query, and passing an array containing MongoDB Aggregation Pipeline stages.

##### JSON Query Object

```
 { fieldName: {"$in": ["a", "b"]} }
```

##### Aggregation Pipeline array

###### Examples with the following data set:

```
{
 make: "Ford",
 model: "Explorer",
 onRoadCost: 10000
},
{
 make: "Ford",
 model: "Escape",
 onRoadCost: 7000
},
{
 make: "Nissan",
 model: "Pathfinder",
 onRoadCost: 15000
},
{
 make: "Ford",
 model: "Ranger",
 onRoadCost: 27000
}
```

###### 1. Return only documents for `Ford` vehicles:

```
[
 { $match: { make: "Ford" } }
]
```

**Result:**

```
[
 {
   make: "Ford",
   model: "Explorer",
   onRoadCost: 10000
 },
 {
   make: "Ford",
   model: "Escape",
   onRoadCost: 7000
 },
 {
   make: "Ford",
   model: "Ranger",
   onRoadCost: 27000
 }
]
```

###### 2. Return the average `onRoadCost` for `Ford` vehicles:

```
 [
   {
     $match : { make: "Ford" }
   },
   {
     $group:
     {
       _id: "$make",
       onRoadCostAverage: { $avg : "$onRoadCost" }
     }
   }
 ]
```
**Result:**

```
[
 {
	"_id" : "Ford",
	"onRoadCostAverage" : 14666.666666666666
 }
]
```

See the MongoDB reference documentation for information about the [Aggregation Pipeline](http://docs.mongodb.org/manual/reference/operator/aggregation/#aggregation-pipeline-operator-reference).

###### Example usage

```
var query = [
   {
     $match : { make: "Ford" }
   },
   {
     $group:
     {
       _id: "$make",
       onRoadCostAverage: { $avg : "$onRoadCost" }
     }
   }
 ];

query = encodeURIComponent(JSON.stringify(query));
client.get('/versionName/databaseName/cars?filter=' + query);
```

#### callback

Overrides the collection's `callback` setting.

callback must be made up of letters only.


### Regex

Original | Converted
---------|----------
{ name: 'ford' } | { name: /^ford$/i }
{ name: { '$regex': 'ford' } } | { name: { '$regex': /ford/i } }
[ { '$group': { _id: '$name' } } ] | [ { '$group': { _id: '$name' } } ]
