# README

## SETUP
1. Download [datasets](https://www.kaggle.com/dannielr/marvel-superheroes?select=superheroes_power_matrix.csv)
2. Install Spark. [See manual](https://www.freecodecamp.org/news/installing-scala-and-apache-spark-on-mac-os-837ae57d283f/).
3. Install MongoDB via Docker
```
docker pull mongo
docker run -d -p 27017:27017 --name "mongo" -v /<absolute path>/data:/data/db mongo
```

## Process Data in Spark-shell
1. Start spark shell with MongDB connector via cli
```
spark-shell --conf "spark.mongodb.input.uri=mongodb://localhost/test.heroes?readPreference=primaryPreferred" \
                  --conf "spark.mongodb.output.uri=mongodb://localhost/test.heroes" \
                  --packages org.mongodb.spark:mongo-spark-connector_2.12:2.4.2
```

2. import dependency in the shell
```
scala> import org.apache.spark.sql.functions.col
import com.mongodb.spark.MongoSpark
```
3. Run the join query to get the characters info with their power matrix
```
val char_info=spark.read.option("header",true).csv("Downloads/data/marvel_characters_info.csv").as("char_info")
val matrix = spark.read.option("header",true).csv("Downloads/data/superheroes_power_matrix.csv").as("matrix")
val char_matrix = char_info.join(matrix, col("char_info.Name")===col("matrix.Name"),"left")
val res = char_matrix.select(col("char_info.Name"), col("Gender"), col("Publisher"), col("Agility"), col("Durability"))

MongoSpark.save(res)
```
## Verify the output data in MongoDB
```
docker exec -it mongo mongo
```
Get all documents:
```
> db.heroes.find()
```
Some of data looks like:
```
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2403"), "Name" : "A-Bomb", "Gender" : "Male", "Publisher" : "Marvel Comics", "Agility" : "FALSE", "Durability" : "TRUE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2404"), "Name" : "Abe Sapien", "Gender" : "Male", "Publisher" : "Dark Horse Comics", "Agility" : "TRUE", "Durability" : "TRUE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2405"), "Name" : "Abin Sur", "Gender" : "Male", "Publisher" : "DC Comics", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2406"), "Name" : "Abomination", "Gender" : "Male", "Publisher" : "Marvel Comics", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2407"), "Name" : "Abraxas", "Gender" : "Male", "Publisher" : "Marvel Comics", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2408"), "Name" : "Absorbing Man", "Gender" : "Male", "Publisher" : "Marvel Comics", "Agility" : "FALSE", "Durability" : "TRUE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2409"), "Name" : "Adam Monroe", "Gender" : "Male", "Publisher" : "NBC - Heroes", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d240a"), "Name" : "Adam Strange", "Gender" : "Male", "Publisher" : "DC Comics", "Agility" : "FALSE", "Durability" : "TRUE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d240b"), "Name" : "Agent 13", "Gender" : "Female", "Publisher" : "Marvel Comics" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d240c"), "Name" : "Agent Bob", "Gender" : "Male", "Publisher" : "Marvel Comics", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d240d"), "Name" : "Agent Zero", "Gender" : "Male", "Publisher" : "Marvel Comics", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d240e"), "Name" : "Air-Walker", "Gender" : "Male", "Publisher" : "Marvel Comics", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d240f"), "Name" : "Ajax", "Gender" : "Male", "Publisher" : "Marvel Comics", "Agility" : "TRUE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2410"), "Name" : "Alan Scott", "Gender" : "Male", "Publisher" : "DC Comics", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2411"), "Name" : "Alex Mercer", "Gender" : "Male", "Publisher" : "Wildstorm", "Agility" : "TRUE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2412"), "Name" : "Alex Woolsly", "Gender" : "Male", "Publisher" : "NBC - Heroes", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2413"), "Name" : "Alfred Pennyworth", "Gender" : "Male", "Publisher" : "DC Comics" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2414"), "Name" : "Alien", "Gender" : "Male", "Publisher" : "Dark Horse Comics", "Agility" : "TRUE", "Durability" : "TRUE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2415"), "Name" : "Allan Quatermain", "Gender" : "Male", "Publisher" : "Wildstorm", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2416"), "Name" : "Amazo", "Gender" : "Male", "Publisher" : "DC Comics", "Agility" : "TRUE", "Durability" : "TRUE" }
```
Query by any specific field, say a hero named Adam Monroe:
```
> db.heroes.find({"Name" : "Adam Monroe"})
{ "_id" : ObjectId("60d08e100f6ef44cfe7d2409"), "Name" : "Adam Monroe", "Gender" : "Male", "Publisher" : "NBC - Heroes", "Agility" : "FALSE", "Durability" : "FALSE" }
{ "_id" : ObjectId("60d08e440f6ef44cfe7d26e8"), "Name" : "Adam Monroe", "Gender" : "Male", "Publisher" : "NBC - Heroes", "Agility" : "FALSE", "Durability" : "FALSE" }
```
