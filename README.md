# Data-Wrangling-Using-Python
The project involves choosing any area of the world in https://www.openstreetmap.org and use data munging techniques, such as assessing the quality of the data for validity, accuracy, completeness, consistency and uniformity, to clean the OpenStreetMap data for a part of the world. 

I used the data of Austin, TX and performed data wrangling using Python and MongoDB

# Problems Encountered in the Map
After initially downloading a small sample size of the Austin area and
running it against data.py, I observed the following problems:

## 1. Multiple Zip Codes
Zip codes were presented in the data under various permutations of tiger:zip_left,tiger:zip_right and postcode defined as semicolon delimited lists or colon delimited ranges. Given that zip codes are a common search criteria, I thought that it would be a good idea to collect and serialize all zipcodes from all sources into a single array, and populate this into the base of the node under zip. I cleaned the zip codes such that there are no extra characters(like colon) between the zip codes and also the length of the zip code should be 5.

The following query shows the top results of the cleaned zip codes.
db.austin.aggregate([{“$match":{"address.zip":{"$exists":1}}},
{"$group":{"_id":"$address.zip","count":{"$sum":1}}},{"$sort":
{"count":1}}])
{ "_id" : "78712", "count" : 8 }
{ "_id" : "78640", "count" : 13 }
{ "_id" : "78727", "count" : 28 }
{ "_id" : "78654", "count" : 34 }

## 2. Abbreviated Street Names
There were inconsistencies with street name abbreviations. For consistency, I translated all abbreviations into the long forms, e.g. EXPWY to Expressway, HWY to Highway, Dr to Drive.

The following query shows the top results of the query to show the abbreviated streets renamed to its full form.
db.austin.aggregate([{"$match":{"address.Street":{"$exists":1}}},
{"$group":{"_id":"$address.Street","count":{"$sum":1}}},{"$sort":
{"count":1}}])
{ "_id" : "Marshall Avenue", "count" : 2 }
{ "_id" : "Edison Cove", "count" : 4 }
{ "_id" : "Norton Avenue", "count" : 4 }
{ "_id" : "Concord Cove", "count" : 4 }
{ "_id" : "Whitney Cove", "count" : 4 }
{ "_id" : "Twain Cove", "count" : 4 }
 { "_id" : "Santa Domingo Lane", "count" : 4 }

## 3. Inconsistent Phone numbers
The phone numbers were listed under keys, phone and ‘contact:phone’. So, I consolidated the entire phone numbers under a single column, “phone” and cleaned them so that they follow a fixed pattern of +1-123-456-7890.

Second level tag values
I cleaned the data to ignore problematic second level tag values. In case of address, I considered only the Street name, house number and zip code. I removed other data such as the GNIS data.

# Data Overview
This section contains basic statistics about the dataset and the
MongoDB queries used to gather them.
## File sizes
austin.osm = 142MB
austin.osm.json = 644.7 MB
## Number of documents
> db.austin.find().count()
1630416
> db.austin.find({"type":"node"}).count()
666476
> db.austin.find({"type":"way"}).count()
963940
## Number of unique users
> db.austin.distinct("created.user").length
696
> db.austin.find({shop:{$exists:true}}).count()
254
> db.austin.find({amenity:"cafe"}).count()
 24
## Top 1 contributing user
> db.austin.aggregate([{'$group': { '_id' : '$created.user','count': {'$sum':1}}},{'$sort' : {'count': -1}},{'$limit' :1}])
{ "_id" : "patisilva_atxbuildings", "count" : 625308 }
## Number of users contributing only once:
> db.austin.aggregate([{'$group': { '_id': '$created.user','count': {'$sum': 1 }} }, {'$group': { '_id': '$count', 'num_users': {'$sum': 1}}}, {'$sort': {'_id': 1 }}, {'$limit': 1 }])
{ "_id" : 1, "num_users" : 116 }
# Additional Ideas
Some statistics about the data:
### Top user contribution percentage (“patisilva_atxbuildings”) 
    31.64% Combined top 2 users' contribution (“patisilva_atxbuildings” and “ccjjmartin_atxbuildings”) - 47.76%
A majority number of nodes do not include addresses and the large numberof nd reference tags, it would be better if much of these way tags and waypoint nodes are removed that will also compress the data and thus less storage size would be needed in the database.
It would be better if we use temperature and weather information in the maps. This information is not present in the Google maps as well. This could be a different new addition.
Also by pulling data from the Google maps, we can show traffic information in the map.

# Additional Data exploration:
## Total Zip codes:
db.austin.aggregate([{ '$match': {'zip': {'$exists': 1}}}, {'$unwind': '$zip'}, { '$group': {'_id': '$zip' }}, {'$group': {'_id': 'Zip Codes in Austin', 'count': { '$sum': 1 }}}])
 { "_id" : "Zip Codes in Austin", "count" : 81 }
## Top 10 appearing amenities:
> db.austin.aggregate([{"$match":{"amenity":{"$exists":1}}}, {"$group":{"_id":"$amenity","count":{"$sum":1}}}, {"$sort": {"count":-1}}, {“$limit":10}])
{ "_id" : "restaurant", "count" : 166 }
{ "_id" : "school", "count" : 133 }
{ "_id" : "fast_food", "count" : 127 }
{ "_id" : "pharmacy", "count" : 72 }
{ "_id" : "place_of_worship", "count" : 60 }
{ "_id" : "parking", "count" : 34 }
{ "_id" : "bank", "count" : 31 }
{ "_id" : "fuel", "count" : 28 }›
{ "_id" : "public_building", "count" : 25 }
{ "_id" : "cafe", "count" : 24 }
## Biggest religion
> db.austin.aggregate([{"$match":{"amenity":{"$exists": 1},"amenity":"place_of_worship"}},{"$group": {"_id":"$religion","count":{"$sum":1}}},{"$sort":{"count":-1}}, {"$limit":1}])
{ "_id" : "christian", "count" : 56 }
This is as expected because USA has majority of Christians.
## Popular cuisines:
> db.austin.aggregate([{"$match":{"amenity":{"$exists": 1},"amenity":"restaurant"}},{"$group":{"_id":"$cuisine","count": {"$sum":1}}},{"$sort":{"count":-1}},{"$limit":2}])
{ "_id" : null, "count" : 37 }
{ "_id" : "mexican", "count" : 13 }
Since, Texas is near to Mexico, it is no surprise that Mexican cuisine is the most popular in Austin.
# Conclusion
I found several problems due to the huge size of the data. There are a lot of missing address values in the data and the data is also a bit outdated and many places have been changed as of now. We can pull the updated information from Google API. After this review of the data it’s obvious that the Austin area is incomplete, though I believe it has been well cleaned for the purposes of this exercise.
