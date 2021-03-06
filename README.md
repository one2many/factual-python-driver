# About

This is the Factual-supported Python driver for [Factual's public API](http://developer.factual.com).

# Install

```bash
pip install factual-api
```

# Get Started

Include this driver in your project:
```python
from factual import Factual
factual = Factual('YOUR_KEY', 'YOUR_SECRET')
```
If you don't have a Factual API key yet, [it's free and easy to get one](https://www.factual.com/api-keys/request).

## Schema
Use the schema API call to determine which fields are available, the datatypes of those fields, and which operations (sorting, searching, writing, facetting) can be performed on each field.

Full documentation: http://developer.factual.com/api-docs/#Schema
```python
s = factual.table('places').schema()
print(s)
```

## Read
Use the read API call to query data in Factual tables with any combination of full-text search, parametric filtering, and geo-location filtering.

Full documentation: http://developer.factual.com/api-docs/#Read

Related place-specific documentation:
* Categories: http://developer.factual.com/working-with-categories/
* Placerank, Sorting: http://developer.factual.com/search-placerank-and-boost/

```python
places = factual.table('places')

# Full-text search:
places_with_count = places.search('century city mall').include_count(True)
data = places_with_count.data()
print('showing {}/{} rows: {}'.format(places_with_count.included_rows(), places_with_count.total_row_count(), data))

# Row filters:
#  search restaurants (http://developer.factual.com/working-with-categories/)
#  note that this will return all sub-categories of 347 as well.
data = places.filters({'category_ids':{'$includes':347}}).data()
print(data)

#  search restaurants or bars
data = places.filters({'category_ids':{'$includes_any':[312,347]}}).data()
print(data)

#  search entertainment venues but NOT adult entertainment
data = places.filters({'$and':[{'category_ids':{'$includes':317}},{'category_ids':{'$excludes':318}}]}).data()
print(data)

#  search for Starbucks in Los Angeles
data = places.search('starbucks').filters({'locality':'los angeles'}).data()
print(data)

#  search for starbucks in Los Angeles or Santa Monica 
data = places.search('starbucks').filters({'$or':[{'locality':{'$eq':'los angeles'}},{'locality':{'$eq':'santa monica'}}]}).data()
print(data)

# Paging:
#  search for starbucks in Los Angeles or Santa Monica (second page of results):
data = places.search('starbucks').filters({'$or':[{'locality':{'$eq':'los angeles'}},{'locality':{'$eq':'santa monica'}}]}).offset(20).limit(20).data()
print(data)

# Geo filter:
#  coffee near the Factual office
from factual.utils import circle
data = places.search('coffee').geo(circle(34.058583, -118.416582, 1000)).data()
print(data)

# Existence threshold:
#  prefer precision over recall:
data = places.threshold('confident').data()
print(data)

# Get a row by factual id:
data = factual.get_row('places', '03c26917-5d66-4de9-96bc-b13066173c65')
print(data)
```

## Facets
Use the facets call to get summarized counts, grouped by specified fields.

Full documentation: http://developer.factual.com/api-docs/#Facets
```python
# show top 5 cities that have more than 20 Starbucks in California
data = factual.facets('places').search('starbucks').filters({'region':'CA'}).select('locality').min_count(20).limit(5).data()
print(data)
```

## Resolve
Use resolve to generate a confidence-based match to an existing set of place attributes.

Full documentation: http://developer.factual.com/api-docs/#Resolve
```python
# resovle from name and address info
data = factual.resolve('places', {'name':'McDonalds','address':'10451 Santa Monica Blvd','region':'CA','postcode':'90025'}).data()
print(data)

# resolve from name and geo location
data = factual.resolve('places', {'name':'McDonalds','latitude':34.05671,'longitude':-118.42586}).data()
print(data)
```

## Match
Match is similar to resolve, but returns only the Factual ID and is intended for high volume mapping.

Full documentation: http://developer.factual.com/api-docs/#Match
```python
data = factual.match('places', {'name':'McDonalds','address':'10451 Santa Monica Blvd','region':'CA','postcode':'90025','country':'US'}).data()
print(data)
```

## Crosswalk
Crosswalk contains third party mappings between entities.

Full documentation: http://developer.factual.com/places-crosswalk/

```python
# Query with factual id, and only show entites from Yelp:
data = factual.crosswalk().filters({'factual_id':'3b9e2b46-4961-4a31-b90a-b5e0aed2a45e','namespace':'yelp'}).data()
print(data)
```

```python
# query with an entity from Foursquare:
data = factual.crosswalk().filters({'namespace':'foursquare','namespace_id':'4ae4df6df964a520019f21e3'}).data()
print(data)
```

## World Geographies
World Geographies contains administrative geographies (states, counties, countries), natural geographies (rivers, oceans, continents), and assorted geographic miscallaney.  This resource is intended to complement the Global Places and add utility to any geo-related content.

```python
# find California, USA
data = factual.table('world-geographies').filters({'$and':[{'name':{'$eq':'California'}},{'country':{'$eq':'US'}},{'placetype':{'$eq':'region'}}]}).select('contextname,factual_id').data()
print(data)
# returns 08649c86-8f76-11e1-848f-cfd5bf3ef515 as the Factual Id of "California, US"
```

```python
# find cities and town in California (first 20 rows)
data = factual.table('world-geographies').filters({'$and':[{'ancestors':{'$includes':'08649c86-8f76-11e1-848f-cfd5bf3ef515'}},{'country':{'$eq':'US'}},{'placetype':{'$eq':'locality'}}]}).select('contextname,factual_id').data()
print(data)
```

## Submit
Submit new data, or update existing data. Submit behaves as an "upsert", meaning that Factual will attempt to match the provided data against any existing places first. Note: you should ALWAYS store the *commit ID* returned from the response for any future support requests.

Full documentation: http://developer.factual.com/api-docs/#Submit

Place-specific Write API documentation: http://developer.factual.com/write-api/

```python
values = {
    'name': 'Factual',
    'address': '1999 Avenue of the Stars',
    'address_extended': '34th floor',
    'locality': 'Los Angeles',
    'region': 'CA',
    'postcode': '90067',
    'country': 'us',
    'latitude': 34.058743,
    'longitude': -118.41694,
    'category_ids': [209,213],
    'hours': 'Mon 11:30am-2pm Tue-Fri 11:30am-2pm, 5:30pm-9pm Sat-Sun closed',
}
resp = factual.submit('us-sandbox', values=values).user('a_user_id').write()
print(resp)
```

Edit an existing row:
```python
resp = factual.submit('us-sandbox', '4e4a14fe-988c-4f03-a8e7-0efc806d0a7f', {'address_extended':'35th floor'}).user('a_user_id').write()
print(resp)
```


## Flag
Use the flag API to flag problems in existing data.

Full documentation: http://developer.factual.com/api-docs/#Flag

Flag a place that is a duplicate of another. The *preferred* entity that should persist is passed as a GET parameter.
```python
resp = factual.flag('us-sandbox', '4e4a14fe-988c-4f03-a8e7-0efc806d0a7f').duplicate(preferred='9d676355-6c74-4cf6-8c4a-03fdaaa2d66a').user('a_user_id').write()
print(resp)
```

Flag a place that is closed.
```python
resp = factual.flag('us-sandbox', '4e4a14fe-988c-4f03-a8e7-0efc806d0a7f').problem('closed').comment('was shut down when I went there yesterday.').user('a_user_id').write()
print(resp)
```

Flag a place that has been relocated, so that it will redirect to the new location. The *preferred* entity (the current location) is passed as a GET parameter. The old location is identified in the URL.
```python
resp = factual.flag('us-sandbox', '4e4a14fe-988c-4f03-a8e7-0efc806d0a7f').relocated(preferred='9d676355-6c74-4cf6-8c4a-03fdaaa2d66a').user('a_user_id').write()
print(resp)
```

## Clear
The clear API is used to signal that an existing attribute's value should be reset.

Full documentation: http://developer.factual.com/api-docs/#Clear
```python
resp = factual.clear('us-sandbox', '4e4a14fe-988c-4f03-a8e7-0efc806d0a7f', fields='latitude,longitude').user('a_user_id').write()
print(resp)
```

## Boost
The boost API is used to signal rows that should appear higher in search results.

Full documentation: http://developer.factual.com/api-docs/#Boost
```python
resp = factual.boost('us-sandbox', '4e4a14fe-988c-4f03-a8e7-0efc806d0a7f', q='local business data').user('a_user_id').write()
print(resp)
```

## Multi
Make up to three simultaneous requests over a single HTTP connection. Note: while the requests are performed in parallel, the final response is not returned until all contained requests are complete. As such, you shouldn't use multi if you want non-blocking behavior. Also note that a contained response may include an API error message, if appropriate.

Full documentation: http://developer.factual.com/api-docs/#Multi

```python
# Query read and facets in one request:
import json
read_query = factual.table('places').search('starbucks').geo(circle(34.041195,-118.331518,1000))
facets_query = factual.facets('places').search('starbucks').filters({'region':'CA'}).select('locality').min_count(20).limit(5)
raw_resp = factual.multi({'read':read_query,'facets':facets_query})
query_results = json.loads(raw_resp)
print(query_results['read'])
print(query_results['facets'])
```


## Error Handling
The driver may throw a `factual.api.APIException` exception for invalid requests or a more generic exception for network errors or other problems.

## Debug Mode
To see debug information about the requests being sent to Factual, you can get the url created by a query:
```python
q = factual.table('places').search('starbucks').filters({'region':'CA'}).limit(10)
print(q.get_url())
```


## Custom timeouts
You can set the request timeout (in seconds):
```python
# set the timeout as 1 second
factual = Factual('YOUR_KEY', 'YOUR_SECRET', timeout=1.0)
```
`Timeout` exceptions are raised when the server does not issue a response within the specified time.


# Where to Get Help

If you think you've identified a specific bug in this driver, please file an issue in the github repo. Please be as specific as you can, including:

  * What you did to surface the bug
  * What you expected to happen
  * What actually happened
  * Detailed stack trace and/or line numbers

If you are having any other kind of issue, such as unexpected data or strange behaviour from Factual's API (or you're just not sure WHAT'S going on), please contact us through the [Factual support site](http://support.factual.com/factual).
