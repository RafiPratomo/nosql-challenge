# NoSQL Database Setup and Analysis

This repository contains the setup and analysis of a NoSQL database using MongoDB and Python. The tasks include importing data, performing CRUD operations, and executing exploratory data analysis on a dataset of food establishments in the UK.

## Part 1: Database and Jupyter Notebook Set Up

1. **Import the Data**
   - Import the `establishments.json` file into MongoDB using the following command:
     ```sh
     mongoimport --db uk_food --collection establishments --file establishments.json --jsonArray
     ```

2. **Set Up the Notebook**
   - Import necessary libraries:
     ```python
     from pymongo import MongoClient
     from pprint import pprint
     ```
   - Create a MongoDB client instance and connect to the database:
     ```python
     mongo = MongoClient(port=27017)
     db = mongo['uk_food']
     ```
   - Verify database and collection creation:
     ```python
     print(mongo.list_database_names())
     print(db.list_collection_names())
     pprint(db.establishments.find_one())
     ```
   - Assign the collection to a variable for further use:
     ```python
     establishments = db['establishments']
     ```

## Part 2: Update the Database

1. **Add a New Restaurant**
   - Create and insert a new restaurant document:
     ```python
     new_restaurant = {
         "BusinessName": "Penang Flavours",
         "BusinessType": "Restaurant/Cafe/Canteen",
         "geocode": {"longitude": "0.0060", "latitude": "51.4860"}
     }
     establishments.insert_one(new_restaurant)
     ```

2. **Update Restaurant Information**
   - Find the `BusinessTypeID` for "Restaurant/Cafe/Canteen":
     ```python
     result = establishments.find_one({"BusinessType": "Restaurant/Cafe/Canteen"}, {"BusinessTypeID": 1, "BusinessType": 1})
     pprint(result)
     ```
   - Update the new restaurant with the correct `BusinessTypeID`:
     ```python
     business_type_id = result['BusinessTypeID']
     establishments.update_one(
         {"BusinessName": "Penang Flavours"},
         {"$set": {"BusinessTypeID": business_type_id}}
     )
     ```

3. **Remove Establishments in Dover**
   - Count and delete establishments in Dover:
     ```python
     print(establishments.count_documents({"LocalAuthorityName": "Dover"}))
     establishments.delete_many({"LocalAuthorityName": "Dover"})
     print(establishments.count_documents({"LocalAuthorityName": "Dover"}))
     ```

4. **Convert Data Types**
   - Update `latitude` and `longitude` to decimal numbers:
     ```python
     establishments.update_many({}, [
         {"$set": {"geocode.latitude": {"$toDouble": "$geocode.latitude"}}},
         {"$set": {"geocode.longitude": {"$toDouble": "$geocode.longitude"}}}
     ])
     ```
   - Update `RatingValue` to integers:
     ```python
     non_ratings = ["AwaitingInspection", "Awaiting Inspection", "AwaitingPublication", "Pass", "Exempt"]
     establishments.update_many({"RatingValue": {"$in": non_ratings}}, [{"$set": {"RatingValue": None}}])
     establishments.update_many({}, [{"$set": {"RatingValue": {"$toInt": "$RatingValue"}}}])
     ```

## Part 3: Exploratory Analysis

1. **Establishments with Hygiene Score of 20**
   ```python
   query = {"scores.Hygiene": 20}
   results = establishments.find(query)
   df = pd.DataFrame(results)
   print(len(df))
   print(df.head(10))
   ```
2. **Establishments in London with RatingValue >= 4**
   ```python
   query = {"LocalAuthorityName": "London", "RatingValue": {"$gte": 4}}
   results = establishments.find(query)
   df = pd.DataFrame(results)
   print(len(df))
   print(df.head(10))
   ```
3. **Top 5 Establishments with RatingValue 5 near Penang Flavours**
   ```python
   degree_search = 0.01
    latitude = 51.4860
    longitude = 0.0060
    
    query = {
        "RatingValue": 5,
        "$expr": {
            "$and": [
                {"$gte": [{"$toDouble": "$geocode.latitude"}, latitude - degree_search]},
                {"$lte": [{"$toDouble": "$geocode.latitude"}, latitude + degree_search]},
                {"$gte": [{"$toDouble": "$geocode.longitude"}, longitude - degree_search]},
                {"$lte": [{"$toDouble": "$geocode.longitude"}, longitude + degree_search]}
            ]
        }
    }
    sort = [("scores.Hygiene", 1)]
    results = establishments.find(query).sort(sort).limit(5)
    df = pd.DataFrame(results)
    print(df.head(10))
   ```
4. **Count of Establishments with Hygiene Score of 0 by Local Authority**
   ```python
   pipeline = [
    {"$match": {"scores.Hygiene": 0}},
    {"$group": {"_id": "$LocalAuthorityName", "count": {"$sum": 1}}},
    {"$sort": {"count": -1}}
    ]
    results = list(establishments.aggregate(pipeline))
    df = pd.DataFrame(results)
    print(len(df))
    print(df.head(10))
    ```
   
**Requirements**
  - MongoDB
  - Python
  - PyMongo
  - Pandas
