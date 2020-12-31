## Lecture 6 Hadoop Overview and History
### What is Hadoop
An open source software platform for distributed storage and distributed processing of very large data sets on computers clusters build from commodity hardware.

#### Why Hadoop
* Data is too damn big
* Vertical scaling does not work
* Horizontal scaling is linear
* Hadoop: it's not just for batch process anymore

## Lecture 7 Overview of the Hadoop Ecosystem
### Hadoop EchoSystem
![Hadoop EchoSystem](./images/hadoop_echosystem_2.PNG)

#### HDFS
Hadoop Distributed File System

#### YARN
Yet another resource negotiator 

#### MapReduce
Algorithm for hadoop
**Map:** Split the work into multiple parts
**Reduce:** Join the result from the multiple jobs

#### Flume
Used for ingesting unstructured or semi structured data

#### Sqoop
Used for ingesting structured data

#### Hive
Convert MapReduce code to SQL like language

#### Pig
High Level Scripting language which sits on top of MapReduce

#### Zookeeper 
Used for managing hadoop clusters

#### Ambari
Cluster Manager for Hadoop

#### Mesos
An alternative to yarn

#### Oozie 
Scheduler for hadoop jobs

#### Storm
* tool for processing real time streaming data

#### Kafka
* tool for ingesting data from other sources

#### HBase
* a very fast no sql database
* Exposes to data of your cluster to transactional platform

#### Spark
An open source cluster computing framework for real time processing.


#### Tez
* Similar to spark, produces more optimized query than MapReduce
* Can be use used with Hive

## Lecture 8 HDFS: What it is and how it works
Files are split into 128mb blocks. Blocks are replicated to multiple times.

**HDFS Architecture**
Single NameNode and one or more DataNodes. 

**Reading A File**
Client asks namenode for file, namenode will return the address of blocks. Client will go to that datanodes to retrive the blocks.

**Writing A File**
Client asks NameNode to store a file. NameNode returns a list of DataNodes to store the file. Client sends the data to first datanode, the first one stores the data and send it to the second one and so on. When the last one saved the data , it return ack to the previous one, and so on, finally client receives ack and informs Namenode that writing done.


**NameNode Failure Handling**
* Backup Metadata, namenode writes to local disk and nfs, in case of failure it read the data when boot.
* Secondary NameNode, maintains merged copy of edit log for backup
* HDFS Federation, each namenode manages a specific namespace volume
* HDFS H/A, Hot standby namenode using shared edit log, ZooKeeper tracks active namenode, uses extreme measure to ensure only one namenode is uses at a time.

**HDFS Usage:**
UI (Ambari), CLI, HTTP/HDFS Proxies, Java Interface, NFS Gateway

## Lecture 11. MapReduce: What it is and how it works
**Why MapReduce**
* Distributes the processing of data on your cluster
* Divides the data up into partitions that are MAPPED and REDUCED by mapper and reducer functions
* Resilient to failure - an application master monitors your mappers and reducers on each partition

**How MapReduce Works: Mapping**
* The mapper converts raw source data into key/value pairs
* The mapper automatically aggregates together all values for each unique key (Shuffle) and then sorts the keys(Sort).
* The reducer processes each keys values


## Lecture 12. How mapReduce distributes processing
Mapper splits the input into multiple slots and then separate datanodes process them separately.
Shuffle and sort aggregates and sort the mapper output.
Multiple reducers reduce different keys.

![MapReduce distributed processing](./images/udemy_hadoop_12_1.PNG)
![MapReduce distributed processing under the hood](./images/udemy_hadoop_12_2.PNG)

**How are mappers and reducers written?**
* native java
* streaming with other languages like python

**Handling Failure**
* Application master monitors worker tasks for error, restart as needed , preferably on a different node
* If application master goes down, YARN can try to restart it.
* If entire node goes down, resource manager will try to restart it.
* if resource manager goes down, HA using ZooKeeper have a standby.

## Chapter 19. Introduction to Ambari
Ambari is a UI for Hadoop Component
Reset ambari admin pw
```
# su root
# ambari-admin-password-reset
```
Ambari will restart. Login to Ambari with admin/new_pw

## Chapter 20. Introducing Pig
Pig uses Pig latin, a scripting language like sql to define map and reduce steps.
Pig stand on top of  MapReduce and Tez which is faster than MapReduce and uses them, so there is no performance penalty for using Pig. 
Tez uses directed acyclic graph looks into the inter relation of the steps and finds the most optimal path for executing steps , so it is faster than MapReduce.

## Chapter 22. Find old 5-start movies with Pig
```python
ratings = LOAD '/user/maria_dev/ml-100k/u.data' AS (userID:int, movieID:int, rating:int, ratingTime:int);
metadata = LOAD '/user/maria_dev/ml-100k/u.item' USING PigStorage('|')
	AS (movieID:int, movieTitle:chararray, releaseDate:chararray, videoRealese:chararray, imdblink:chararray);
   
nameLookup = FOREACH metadata GENERATE movieID, movieTitle,
	ToUnixTime(ToDate(releaseDate, 'dd-MMM-yyyy')) AS releaseTime;
   
ratingsByMovie = GROUP ratings BY movieID;
avgRatings = FOREACH ratingsByMovie GENERATE group as movieID, AVG(ratings.rating) as avgRating;
fiveStarMovies = FILTER avgRatings BY avgRating > 4.0;
fiveStarsWithData = JOIN fiveStarMovies BY movieID, nameLookup BY movieID;
oldestFiveStarMovies = ORDER fiveStarsWithData BY nameLookup::releaseTime;
DUMP oldestFiveStarMovies;
```

## Chapter 23. More Pig Latin
PIG QUERY commands
* LOAD STORE DUMP
* FILTER DISTINCT FOREACH/GENERATE MAPREDUCE STREAM SAMPLE
* JOIN COGROUP GROUP CROSS CUBE
* ORDER RANK LIMIT
* UNION SPLIT

Pig Diagnostic Commands
* DESCRIBE
* EXPLAIN
* ILLUSTRATE

PIG UDF Commands
* REGISTER
* DEFINE
* IMPORT

Others
* AVG CONCAT COUNT MAX MIN SIZE SUM
* PigStorage
* TextLoader
* JsonLoader
* AvroStorage
* ParquetLoader
* OrcStorage
* HBaseStorage

## Chapter 26. Why Spark?
 
![Spark Tasks](./images/udemy_hadoop_26_1.PNG)

**Driver Program** is a script which controls what's going to happen in your job
**Cluster manager** spark can use YARN or any other cluster manager like mesos or the built in cluster manager. Spark can run on Hadoop but it can work with other frameworks.

**Executor** - spark tries to retain as much information it can in memory

Spark is 100x faster than MapReduce in memory or 10x faster in disk.
Spark uses Directed Acyclic Graph to optimize workflows.
Spark is build around one main concept : the Resilient Distributed Dataset (RDD)

![Spark Components](./images/udemy_hadoop_26_2.PNG)
**Spark Streaming** - allows real time data ingestion and processing for Spark
**Spark SQL** - SQL interface for Spark
**MLLib** - machine learning and data mining library for Spark
**GraphX** - graph interface for Spark

## Chapter 27. THe Resilient Distributed Dataset (RDD)
It is a abstraction that hides the real data from programmer and presents the user with a Dataset interface.

**The SparkContext**
* created by driver program
* it creates and is responsible for RDD's
* spark shell creates a `sc` object for user

RDD's can be created from local file system, hdfs, s3, hive, JDBC, Cassandra, HBase, Elasticsearch, etc.

**Transforming RDD's**
* map
* flatmap
* filter
* distinct
* filter
* sample
* union, intersection, subtract, cartesian

**RDD Actions**
* collect
* count
* countByValue
* take
* top
* reduce
* and others

## Chapter 28. Find the movie with the lowest average rating -with RDD's
```python
from pyspark import SparkConf, SparkContext

# This function just creates a Python "dictionary" we can later
# use to convert movie ID's to movie names while printing out
# the final results.
def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames

# Take each line of u.data and convert it to (movieID, (rating, 1.0))
# This way we can then add up all the ratings for each movie, and
# the total number of ratings for each movie (which lets us compute the average)
def parseInput(line):
    fields = line.split()
    return (int(fields[1]), (float(fields[2]), 1.0))

if __name__ == "__main__":
    # The main script - create our SparkContext
    conf = SparkConf().setAppName("WorstMovies")
    sc = SparkContext(conf = conf)

    # Load up our movie ID -> movie name lookup table
    movieNames = loadMovieNames()

    # Load up the raw u.data file
    lines = sc.textFile("hdfs:///user/maria_dev/ml-100k/u.data")

    # Convert to (movieID, (rating, 1.0))
    movieRatings = lines.map(parseInput)

    # Reduce to (movieID, (sumOfRatings, totalRatings))
    ratingTotalsAndCount = movieRatings.reduceByKey(lambda movie1, movie2: ( movie1[0] + movie2[0], movie1[1] + movie2[1] ) )

    # Map to (movieID, averageRating)
    averageRatings = ratingTotalsAndCount.mapValues(lambda totalAndCount : totalAndCount[0] / totalAndCount[1])

    # Sort by average rating
    sortedMovies = averageRatings.sortBy(lambda x: x[1])

    # Take the top 10 results
    results = sortedMovies.take(10)

    # Print them out:
    for result in results:
        print(movieNames[result[0]], result[1])
```

## Chapter 29. Datasets and Spark 2.0
DataFrame extends RDD to a DataFrame object.
DataFrame contain Row objects and can run SQL queries

Use SparkSQL in Python
```python
from pyspark.sql import SQLContext, Row
hiveContext = HiveContext(sc)
inputData = spark.read.json(datafile)
inputData.createOrReplaceTempView(structuredView)
dataFrame = hiveContext.sql("sql query")
```

In Spark 2.0, a DataFrame in really a DataSet of Row objects.
Use can use user defined functions.

## Chapter 30. Find the movie with the lowest average rating -with DataFrames
```python
from pyspark.sql import SparkSession
from pyspark.sql import Row
from pyspark.sql import functions

def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames

def parseInput(line):
    fields = line.split()
    return Row(movieID = int(fields[1]), rating = float(fields[2]))

if __name__ == "__main__":
    # Create a SparkSession
    spark = SparkSession.builder.appName("PopularMovies").getOrCreate()

    # Load up our movie ID -> name dictionary
    movieNames = loadMovieNames()

    # Get the raw data
    lines = spark.sparkContext.textFile("hdfs:///user/maria_dev/ml-100k/u.data")
    # Convert it to a RDD of Row objects with (movieID, rating)
    movies = lines.map(parseInput)
    # Convert that to a DataFrame
    movieDataset = spark.createDataFrame(movies)

    # Compute average rating for each movieID
    averageRatings = movieDataset.groupBy("movieID").avg("rating")

    # Compute count of ratings for each movieID
    counts = movieDataset.groupBy("movieID").count()

    # Join the two together (We now have movieID, avg(rating), and count columns)
    averagesAndCounts = counts.join(averageRatings, "movieID")

    # Pull the top 10 results
    topTen = averagesAndCounts.orderBy("avg(rating)").take(10)

    # Print them out, converting movie ID's to names as we go.
    for movie in topTen:
        print (movieNames[movie[0]], movie[1], movie[2])

    # Stop the session
    spark.stop()
```


## Chapter 31. Movie recommendations with MLLib
```python
from pyspark.sql import SparkSession
from pyspark.ml.recommendation import ALS
from pyspark.sql import Row
from pyspark.sql.functions import lit

# Load up movie ID -> movie name dictionary
def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1].decode('ascii', 'ignore')
    return movieNames

# Convert u.data lines into (userID, movieID, rating) rows
def parseInput(line):
    fields = line.value.split()
    return Row(userID = int(fields[0]), movieID = int(fields[1]), rating = float(fields[2]))


if __name__ == "__main__":
    # Create a SparkSession
    spark = SparkSession.builder.appName("MovieRecs").getOrCreate()

    # Load up our movie ID -> name dictionary
    movieNames = loadMovieNames()

    # Get the raw data
    lines = spark.read.text("hdfs:///user/maria_dev/ml-100k/u.data").rdd

    # Convert it to a RDD of Row objects with (userID, movieID, rating)
    ratingsRDD = lines.map(parseInput)

    # Convert to a DataFrame and cache it
    ratings = spark.createDataFrame(ratingsRDD).cache()

    # Create an ALS collaborative filtering model from the complete data set
    als = ALS(maxIter=5, regParam=0.01, userCol="userID", itemCol="movieID", ratingCol="rating")
    model = als.fit(ratings)

    # Print out ratings from user 0:
    print("\nRatings for user ID 0:")
    userRatings = ratings.filter("userID = 0")
    for rating in userRatings.collect():
        print movieNames[rating['movieID']], rating['rating']

    print("\nTop 20 recommendations:")
    # Find movies rated more than 100 times
    ratingCounts = ratings.groupBy("movieID").count().filter("count > 100")
    # Construct a "test" dataframe for user 0 with every movie rated more than 100 times
    popularMovies = ratingCounts.select("movieID").withColumn('userID', lit(0))

    # Run our model on that list of popular movies for user ID 0
    recommendations = model.transform(popularMovies)

    # Get the top 20 movies with the highest predicted rating for this user
    topRecommendations = recommendations.sort(recommendations.prediction.desc()).take(20)

    for recommendation in topRecommendations:
        print (movieNames[recommendation['movieID']], recommendation['prediction'])

    spark.stop()

```
