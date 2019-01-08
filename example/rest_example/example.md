### Example of querying current rest service to augment a fake taxi example dataset

#### You can run your own flask
```commandline
conda activate datamart_env
python ../../datamart_web/webapp.py
```

#### There is a service running on dsbox02
#### Show a demo of query current REST API, input data is [fifa.csv](fifa.csv)

###### POST a query to default search for metadatas with the input dataset, along with a query string.

The following curl using `fifa` as query string, write to `result.json`

```commandline
curl -X POST 'http://dsbox02.isi.edu:9000/search/default_search?query_string=fifa' \
-F "file=@{your_path}/fifa.csv" > result.json
```

It should return many metadata hitted.

Suppose user select one, front end will return some request like [sample_query.json](./sample_query.json)

`selected_metadata` is the metadata user selected

`columns_mapping` is the mapping used for joiner, telling pairs of columns for join


###### Second request is for default join

We get the data from some soccer api

Then join two dataframes using default left outer join, write to `augmented.csv`
```commandline
value=$(<{your_path}/sample_query.json); \
curl -X POST  http://dsbox02:9000/augment/default_join \
-F "data=$value" > augmented.csv
```

It returns a csv `augmented.csv` now, which is [this one](../fifa_example/augmented.csv)
