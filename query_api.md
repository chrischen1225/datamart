# Datamart Query API

The query API provides an interface for searching complementary datasets that can be used
to augment the supplied dataset to improve prediction performance.

This API provides the three ways to search for complementary datasets: 
* Search by example
* Search by metadata query
* Search by content query

In _search by example_ the query engine finds other datasets that contain data similar to
the given the example data. For instance, give columns of a D3M Dataframe as example, the
query engine returns datasets with similar columns. In _search by metadata query_ the query
engine searches metadata descriptions of the datasets, and in _search by content_ the query
engine searches content of the datasets.

The API defines a domain specific language in JSON for specifying search queries. This 
language allows the three ways of searching to be mixed-and-matched and combined into one 
query. 

The JSON Schema of the query language is [here](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=594693729313479284833894&h2=Input-schema). Sample queries can be found [here](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=383295369662625070053841&h2=Input-sample-in-details) and [here](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=384363162480892852000682&h2=Input-samples-for-examples(Tax).

The query returns a ranked list of dataset matches in JSON. Each match contains a match score,
metadata of the matched dataset, and a description of how the dataset satisfies the given
query. The query results can used to further refined the search by TA3 users, or by smart
joiner to create augmented datasets.

The JSON Schema for result is [here](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=427212382360417372934863&h2=Output-Schema). Sample results can be found [here](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=079293874953179037454694&h2=Output-Sample).

### 1. Input

The API defines a query method that takes two inputs:
- data: example data, either D3M Dataset or D3M Dataframe
- query: a JSON object representing what the user what to query (See [2. Query Schema](#2.-Query-Schema)).

### 2. Query Schema

Domain-specific language to specify queries for searching datasets in Datamart. A `query`
contains three root-level properties: 
* `dataset`
* `required_variables`
* `desired_variables`

The `dataset` property is for searching at the dataset-level. This
includes searching the dataset metadata, and the content of the dataset. The
`required_variables` and `desired_variables` are for searching variables, i.e. columns, of
the dataset. Similarly, these properties include searching the column metadata and the content of
columns. For `required_variables` the matched dataset _must_ contain the specified columns,
and for `desired_variables` the matched dataset _should_ contain the specified columns. All
three properties are optional.



Below is a detailed description of the query schema:

- Descriptions:
  The `query` will be a JSON object, with three root properties: `dataset`, `required_variables` and `desired_variables`.
  - `dataset`: contains global specification of the desired datasets, like what the dataset is about, when the dataset is published etc., with the following fields:
    - `about`: a query __string__ that is matched with all information in a dataset, including dataset metadata, column metadata and all dataset values. A matching dataset should match at least one of the words in the query string. The matching algorithm gives preference to phrases when possible.
    - `name`: an __array of string__ of the names/titles of a dataset. (http://schema.org/name)
    - `description`: an __array of string__ of the descriptions of a dataset. (http://schema.org/description)
    - `keywords`: an __array of string__ of the keywords of a dataset. (http://schema.org/keywords)
    - `creator`: an __array of string__ of the creators of a dataset. (http://schema.org/creator)
    - `publisher`: an __array of string__ of the publishers of a dataset. (http://schema.org/publisher)
    - `date_created`: an __object__ specifying when a dataset is created, with fields `after` and `before`, each is a __string__ for a date. (http://schema.org/dateCreated) 
        _(inclusive for both `after` and `before`)_.
    - `date_published`: an __object__ specifying when a dataset is published, with fields `after` and `before`, each is a __string__ for a date. (http://schema.org/datePublished) 
        _(inclusive for both `after` and `before`)_.
    - `url`: an __array of string__ of the URLs of a dataset. (http://schema.org/url)
  - `required_variables` (optional): contains an __array of object__, each object will represent a [variable](#*variable) that is required in the matching datasets. All variables in the 'required_variables' set must be match by at least one column in a matching dataset. It is possible that an item is matched using a combination of columns. For example, a temporal item with day resolution can be matched by a dataset that represents dates using multiple columns, for year, month and date.  Typically, the 'required_variables' section is used to list columns to be used to perform a join. 
  - `desired_variables` (optional): contains an __array of object__, each object will represent a [variable](#*variable) that is desired in the matching datasets. The 'desired_variables' section describes the minimum set of columns that a matching dataset must have. A matching dataset must contain columns that match at least one of the 'desired_variables'. Typically, the 'desired_variables' are used to specify columns that will be used for augmentation.
  - *variable: an __object__, with a required key `type` whose value is one of [`temporal_entity`, `geospatial_entity`, `dataframe_columns`, `generic_entity`]:
    1. `temporal_entity`: describe columns containing temporal information. 
          - `type`: "temporal_entity"
          - `start`: a string for date(time), requested dates are equal or older than this date.
          - `end`: a string for date(time), requested dates are equal or more recent than this date.
          - `granularity`: enum, one of ["year", "month", "day", "hour", "minute", "second"], requested dates are well matched with the requested granularity. For example, if "day" is requested, the best match is a dataset with dates; however a dataset with hours is relevant too as hourly data can be aggregated into days.
    2. `geospatial_entity`: describe columns containing geospatial entities such as cities, countries, etc.
          - `type`: "geospatial_entity"
          - `circle`: object. Geospatial circle area identified using a radius and a center point on the surface of the earth.
            - `latitude`: number. The latitude of the center point
            - `longitude`: number. The longitude of the center point
            - `radius`: string. A string specify the radius of the area.
            - `granularity`: string(one of ["country", "state", "city", "county", "postalcode"]). The granularity of the entities contained in a bounding box.
          - `bounding_box`: object. Geospatial bounding box identified using two points on the surface of the earth.
            - `latitude1`: number. The latitude of the first point
            - `longitude1`: number. The longitude of the first point
            - `latitude2`: number. The latitude of the second point
            - `longitude2`: number. The longitude of the second point
            - `granularity`: string(one of ["country", "state", "city", "county", "postalcode"]). The granularity of the entities contained in a bounding box.
          - `named_entities`: object. A set of names of geospatial entities. This should be used when the requestor doesn't know what type of geospatial entities are provided, they could be cities, states, countries, etc. A matching dataset should have a column containing the requested entities.
            - `semantic_type`: string(one of ["http://schema.org/AdministrativeArea", "http://schema.org/Country", "http://schema.org/City", "http://schema.org/State"]).
            - `items`: array.
    3. `dataframe_columns`: describe columns that a matching dataset should have in terms of columns of a known dataframe. 
          - `type`: "dataframe_columns"
          - `index`: array. A set of indices that identifies a set of columns in the known dataset. When multiple indices are provides, the matching dataset should contain columns corresponding to each of the given columns.
          - `names`: array. A set of column headers that identifies a set of columns in the known dataset. When multiple headers are provides, the matching dataset should contain columns corresponding to each of the given columns.
          - `relationship`: string(one of ["contains", "similar", "correlated", "anti-correlated", "mutually-informative", "mutually-uninformative"]). The relationship between a column in the known dataset and a column in a matching dataset. The default is 'contains'.
    4. `generic_entity`: describe any entity that is not temporal or geospatial. Temporal and geospatial entities receive special treatment. Datamart can re-aggregate and disaggregate temporal and geo-spatial entities so that the granularity of the requested data and an existing dataset does not need to match exactly.
          - `type`: "generic_entity"
          - `about`: string. A query sting that is matched with all information contained in a column including metadata and values. A matching dataset should contain a column whose metadata or values matches at least one of the words in the query string. The matching algorithm gives preference to phrases when possible.
          - `variable_name`: array. A set of header names. A matching dataset should have a column that matches closely one of the provided names.
          - `variable_metadata`: array. A set of keywords to be matched with all the words appearing in the metadata of a column. A matching dataset should contain a column whose metadata matches at least one of the keywords.
          - `variable_description`: array. A set of keywords to be matched with all the words in the description of a column in a dataset. A matching dataset should contain a column whose description matches at least one of the keywords.
          - `variable_syntactic_type`: array. A set of syntactic types. A matching dataset should contain a column with any of the provided syntactic types. Comment: this should be defined using an enum.
          - `variable_semantic_type`: array. A set of semantic types. A matching dataset should contain a column whose semantic types have a non empty intersection with the provided semantic types.
          - `named_entities`: array. A set of entity names. A matching dataset should contain a column with the requested names.
          - `column_values`: object.
            - `items`: array. A set of arbitrary values of any type, string, number, date, etc. To be used with the caller doesn't know whether the values represent named entities. A matching dataset should contain a column with the requested values.
            - `relationship`: string(one of ["contains", "similar", "correlated", "anti-correlated", "mutually-informative", "mutually-uninformative"]). The relationship between the specified values and the values in a column in a matching dataset. The default is "contains".
- [Detailed sample query](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=383295369662625070053841&h2=Input-sample-in-details)
- [Real examples](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=384363162480892852000682&h2=Input-samples-for-examples(Tax)
- [JSON Schema](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=594693729313479284833894&h2=Input-schema)

### 3. Output
A list of query results, each represents a dataset by `metadata`,  `required_variables`, `desired_variables`, `other_variables` and a ranking `score`.
- `metadata`: an dataset metadata object (has the metadata for the matching dataset on global level, as well as the metadata for each variable.)
- `required_variables`: an array of array of variable names in the dataset. Each array of variable names is matched to an item in the `required_variables` query list.
- `desired_variables`: an array of array of variable names in the dataset.  Each array of variable names is matched to an item in the `desired_variables` query list.
- `other_variables`: an array of variable names that are in the matching dataset, but are not in `required_variables` nor `desired_variables` of the query.
- `score`: a number for how well the dataset is matched with the query
- [Result schema](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=427212382360417372934863&h2=Output-Schema)
- [Result sample](https://paper.dropbox.com/doc/Datamart-Query-API-Input-Output--ATnRjCJAaWFVnovGoA3~~zv_Ag-8Rc8HTkA2GAHS0yehJCfl#:uid=079293874953179037454694&h2=Output-Sample)




