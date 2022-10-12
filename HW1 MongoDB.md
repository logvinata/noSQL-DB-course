# HW1 MongoDB
## 1. Mongo Installation
MongoDB 6.0 Community Edition was installed on Ubuntu LTS 18.04
## 2. Dataset
The dataset used here is part of [Uber Pickups in New York City](https://www.kaggle.com/datasets/fivethirtyeight/uber-pickups-in-new-york-city?resource=download&select=uber-raw-data-jul14.csv) for July 2014.

The data were added to the 'first_db' database using MongoDB Compass add function

## 3. Queries and performance before indexing
### Number of distinct lattitude values

```
db.Uber_july.distinct("Lat").length
>4982
```
Performance before indexing:
```
d = new Date; db.Uber_july.distinct("Lat").length; 
print(new Date - d + 'ms')
>'668ms'
```
After:
```
d = new Date; db.Uber_july.distinct("Lat").length; 
print(new Date - d + 'ms')
>'558ms'
```


### total number of records in July
```
db.Uber_july.countDocuments();
>796121
```

Performance before indexing:
```
d = new Date; db.Uber_july.countDocuments();
print(new Date - d + 'ms')
>'338ms'
```

d = new Date; db.Uber_july.countDocuments();
print(new Date - d + 'ms')
>'298ms'


### Change name of the field from 'Date/Time' to 'Time'  and backwards for all records
before indexing:
```
d = new Date; db.Uber_july.updateMany( {}, { $rename: { "Date/Time": "Time" } } ); 
print(new Date - d + 'ms')
>'8814ms'
```
After indexing:
```
d = new Date; db.Uber_july.updateMany( {}, { $rename: { "Date/Time": "Time" } } ); 
print(new Date - d + 'ms')
>'2140ms'
```
What's this? I don't have `Date/Time` field anymore, probably it just touched every document to make sure the field was not there.
Let's try to change the field name back (**this is the field that was used for indexing!**)
```
d = new Date; db.Uber_july.updateMany( {}, { $rename: { "Time": "Date" } } ); 
print(new Date - d + 'ms')
>'23841ms'
```
This operation was **extremely slow due to indexing**.

Let's check whether we still have text indexes:
```
db.Uber_july.getIndexes()

[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  {
    v: 2,
    key: { _fts: 'text', _ftsx: 1 },
    name: 'Time_text',
    weights: { Time: 1 },
    default_language: 'english',
    language_override: 'language',
    textIndexVersion: 3
  }
]
```
Yes, we do! 

Now, what happens if I change the name of the other field? will it be faster?
```
d = new Date; db.Uber_july.updateMany( {}, { $rename: { "Lon": "Longitude" } } ); 
print(new Date - d + 'ms')
> ''9385ms''
```
Yes the operation is about 2 times **faster** if we don't change the indexed field.


### Number of travels on the July, 14th:
```
db.Uber_july.countDocuments({Time : {$regex : "7/14/"}})
>27350 
```
(27350 is close to average)



Time before indexing:
```
d = new Date; db.Uber_july.countDocuments({Time : {$regex : "7/14/"}});
print(new Date - d + "ms")
>'452ms'
```
After:
```
d = new Date; db.Uber_july.countDocuments({Time : {$regex : "7/14/"}});
print(new Date - d + "ms")
>'303ms'
```


### Convert date and time format:
Original dates had this format: `Time :"7/1/2014 0:03:00"`
Conversion:
```
db.Uber_july.aggregate( [ {
   $project: {
      date: {
         $dateFromString: {
            dateString: '$Time',
            timezone: 'America/New_York'
         }
      }
   }
} ] )
```
Output:
```
{ _id: ObjectId("634445ebc41bf69ddee24ca7"),
  date: 2014-07-01T04:03:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24ca8"),
  date: 2014-07-01T04:05:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24ca9"),
  date: 2014-07-01T04:06:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24caa"),
  date: 2014-07-01T04:09:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cab"),
  date: 2014-07-01T04:20:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cac"),
  date: 2014-07-01T04:35:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cad"),
  date: 2014-07-01T04:57:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cae"),
  date: 2014-07-01T04:58:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24caf"),
  date: 2014-07-01T05:04:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb0"),
  date: 2014-07-01T05:08:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb1"),
  date: 2014-07-01T05:12:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb2"),
  date: 2014-07-01T05:23:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb3"),
  date: 2014-07-01T05:45:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb4"),
  date: 2014-07-01T06:07:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb5"),
  date: 2014-07-01T06:48:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb6"),
  date: 2014-07-01T07:11:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb7"),
  date: 2014-07-01T07:14:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb8"),
  date: 2014-07-01T07:20:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cb9"),
  date: 2014-07-01T07:28:00.000Z }
{ _id: ObjectId("634445ebc41bf69ddee24cba"),
  date: 2014-07-01T07:38:00.000Z }
Type "it" for more
```
Time before indexing from `explain("executionStats")': `` 
```
>'258ms'
```
After:
```
'246ms'
```
Using `.explain(executionStats)` we see `COLLSCAN`, this query does not use indexes!
The query that uses indexes:
```
db.Uber_july.find({$text : {$search : "7/14/"}})
```
and using `.explain(executionStats)` we get `stage: 'IXSCAN'` and  `executionTimeMillis: 0`

### Indexing
I desided to index Time Field and evaluate the effect on different operations.
This required text index:
`db.Uber_july.createIndex( { Time: "text" } )`

Afer indexing all the operations were repeated and performance times recorded.










## Notes
1.I also tried `.explain('executeStats')` to get execution time, but it didn't work with agregate functions
    2. I spent several hours trying to convert dates to standard format and append as new field, but failed. What is the correct way to do it?
    
    3. Indexing does speed up the search by the indexed key, but may slow down the operations that change the indexed key