## Assignment

The requirements of the application that you must implement are described below
in three tiers ("adequate", "good" and "excellent"). These tiers correspond to
one of the grading criteria (explained on the next page). To pass the lab, you
must at least achieve the "adequate" tier. "Good" and "excellent" tier
applications contain **additional** requirements (and may supersede previous
requirements where applicable) that the program needs to adhere to.

The goal of this lab is to make an application which efficiently determines how many people would need to relocate at different amount of sea level rise. 

### "Adequate" application requirements
1. The application takes an integer amount of sea level rise in meters from the command line as the first positional argument.
2. The application uses the ALOS Global Digital Surface Model dataset to determine which places are exactly at or below the new sea level, you can disregard any man-made or natural water barriers. So if the sea level rises 0 meters, everybody living in a place at elevation of 0 meters or lower has to relocate.
3.  The application uses the OpenStreetMap dataset to determine populations. Only cities, towns, villages and hamlets are used to determine if the population needs to relocate. Boroughs, suburbs, quarters etc. are parts of cities and should not be considered separately. The whole population of a city/town/village/hamlet can be assumed to live at the exact coordinates of the place in the OSM dataset.
4. When determining which place lies at which elevation, the application uses the [H3: Uber’s Hexagonal Hierarchical Spatial Index]
 to reduce the computational complexity of joins between the OSM and ALOS data sets. In the report (README.md), it is explicitly explained why using such a spatial index reduces the computational complexity for these types of joins.
5. The application uses the average elevation of an H3 tile to determine if a population needs to relocate.
6. The application uses an H3 resolution of 9.
7. The application outputs the sum of all evacuees on the command line.
8. The application outputs an `.orc` file with the following schema, where `place` is the name of a city/town/village/hamlet, and `num_evacuees` is the number of people in this place:
```scala
import org.apache.spark.sql.types._

val schema = StructType(
               Array(
                 StructField("place", StringType),
                 StructField("num_evacuees", LongType)
               )
             )
```

### "Good" application requirements

9. Produce a relocation plan which is a mapping from source place to the
    closest safe (one that is not underwater) city (not town or village or
    hamlet). For the distance calculation, consider the earth to be flat (like
    🥞), i.e. you don't have to take the curvature of the earth into
    consideration. In this case, the application outputs a modified `.orc` file
    with the following schema, where `place` is the name of a
    city/town/village/hamlet, `num_evacuees` is the number of people in this
    place, and `destination` is the name of the city to relocate to.

```scala
import org.apache.spark.sql.types._

val schema = StructType(
               Array(
                 StructField("place", StringType),
                 StructField("num_evacuees", LongType)
                 StructField("destination", StringType)
               )
             )
```

### "Excellent" application requirements

10. The application also calculates the distance of each place to the closest
    harbour.
11. If a harbour is closer than a safe city, 25% of the population of the place
    must relocate to the harbour to get a boat and live on the ocean. The other 
    75% will still relocate to the nearest safe city. The application outputs a 
    modified `.orc` file with the following schema, where `place` is the name of 
    a city/town/village/hamlet, `num_evacuees` is the number of people 
    evacuating to the `destination`, which is the name of the city to relocate
    to or `Waterworld` in the case of boat relocation:

```scala
import org.apache.spark.sql.types._

val schema = StructType(
               Array(
                 StructField("place", StringType),
                 StructField("num_evacuees", LongType)
                 StructField("destination", StringType)
               )
             )
```

12. The application produces a list of cities that are to be relocated to,
    including `Waterworld` as if it's a city. The list also provides the old
    population and the new population The application outputs this list as a
    secondary `.orc` file with the following schema, where `destination` is the
    name of the city (or `Waterworld`) that will receive evacuees,
    `old_population` is the population before relocation, and `new_population`
    is the population after relocation.

```scala
import org.apache.spark.sql.types._

val schema = StructType(
               Array(
                 StructField("destination", StringType),
                 StructField("old_population", LongType)
                 StructField("new_population", LongType)
               )
             )
```

### Input

For this lab, we will limit ourselves to answer the above questions for
the [Netherlands] subset of OpenStreetMap (provided by [Geofabrik]) and ALOS.
The `.osm.pbf` files can be converted to ORC files using the tool mentioned in
the introduction. The ALOS tool mentioned in the introduction can be used to
obtain Parquet files with elevation data.

**For the sea level rise, you should report your output for `0`, `10`, and `100`
meters,** but feel free to experiment with other levels.

### Hints

Try to keep the following in mind when building your application:

- Functionality: providing a correct answer to this query is not trivial,
  attempt to make your application as robust as possible.
- Scalable: your application should keep functioning correctly when the size of
  the input data set grows larger.
- Use H3 version 3.7.0, since higher versions may cause problems in the
  container setup.
- If you use all data while developing and debugging, testing a single code
  change may take very long, e.g. more than 10 minutes. Therefore, while
  developing, try to make your iterations as short as possible by limiting the
  input datasets. For example, you could first try to answer the question only
  for cities of a population higher than some large amount (so you end up
  answering the query for a few cities, rather than for thousands of places).
  Or, you could load only one or two ALOS tiles of which you know includes a few
  places, so you end up with less elevation data. Once you have a functional
  implementation, move towards calculating on the whole dataset to report the
  output and the run times.
- To get an idea of where a lot of computation takes place, make sure to check
  the Spark history server. This helps you think about how to improve expensive
  steps such as shuffles, by reordering or caching computations, or
  pre-processing data sets, or by thinking about when to include pieces of data
  in intermediate dataframes/sets.
- When writing ORC files, multiple files will be created. This is fine, because
  Spark saves each partition separately and for a typical deployment it does so 
  in a distributed manner. When a requirement says to write "an ORC file", it is
  OK if you end up with multiple files.
- For 0 meter sealevel rise, a whole lot of people should move. This is obviously not very realistic. If you want to see if you can get a more realistic answer, you can do so and put it in the report, might be worth some bonus points. But make sure you also show that you did the exercise as written.

[Netherlands]: http://download.geofabrik.de/europe/netherlands-latest.osm.pbf
[Geofabrik]: https://geofabrik.de/
[H3: Uber’s Hexagonal Hierarchical Spatial Index]: https://github.com/uber/h3
