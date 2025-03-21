---
layout: global
title: Examples
type: "page singular"
navigation:
  weight: 4
  show: true
---
<h2>Apache Spark<span class="tm">&trade;</span> examples</h2>

These examples give a quick overview of the Spark API.
Spark is built on the concept of <em>distributed datasets</em>, which contain arbitrary Java or
Python objects. You create a dataset from external data, then apply parallel operations
to it. The building block of the Spark API is its [RDD API](https://spark.apache.org/docs/latest/rdd-programming-guide.html#resilient-distributed-datasets-rdds).
In the RDD API,
there are two types of operations: <em>transformations</em>, which define a new dataset based on previous ones,
and <em>actions</em>, which kick off a job to execute on a cluster.
On top of Spark’s RDD API, high level APIs are provided, e.g.
[DataFrame API](https://spark.apache.org/docs/latest/sql-programming-guide.html#datasets-and-dataframes) and
[Machine Learning API](https://spark.apache.org/docs/latest/mllib-guide.html).
These high level APIs provide a concise way to conduct certain data operations.
In this page, we will show examples using RDD API as well as examples using high level APIs.

<h2>RDD API examples</h2>

<h3>Word count</h3>
<p>In this example, we use a few transformations to build a dataset of (String, Int) pairs called <code>counts</code> and then save it to a file.</p>


<div class="tab-content">
<div class="tab-pane tab-pane-python active">
<div class="code code-tab">
{% highlight python %}
text_file = sc.textFile("hdfs://...")
counts = text_file.flatMap(lambda line: line.split(" ")) \
             .map(lambda word: (word, 1)) \
             .reduceByKey(lambda a, b: a + b)
counts.saveAsTextFile("hdfs://...")
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-scala">
<div class="code code-tab">
{% highlight scala %}
val textFile = sc.textFile("hdfs://...")
val counts = textFile.flatMap(line => line.split(" "))
                 .map(word => (word, 1))
                 .reduceByKey(_ + _)
counts.saveAsTextFile("hdfs://...")
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-java">
<div class="code code-tab">
{% highlight java %}
JavaRDD<String> textFile = sc.textFile("hdfs://...");
JavaPairRDD<String, Integer> counts = textFile
    .flatMap(s -> Arrays.asList(s.split(" ")).iterator())
    .mapToPair(word -> new Tuple2<>(word, 1))
    .reduceByKey((a, b) -> a + b);
counts.saveAsTextFile("hdfs://...");
{% endhighlight %}
</div>
</div>
</div>

<h3>Pi estimation</h3>
<p>Spark can also be used for compute-intensive tasks. This code estimates <span style="font-family: serif; font-size: 120%;">π</span> by "throwing darts" at a circle. We pick random points in the unit square ((0, 0) to (1,1)) and see how many fall in the unit circle. The fraction should be <span style="font-family: serif; font-size: 120%;">π / 4</span>, so we use this to get our estimate.</p>

<ul class="nav nav-tabs">
  <li class="lang-tab lang-tab-python active"><a href="#">Python</a></li>
  <li class="lang-tab lang-tab-scala"><a href="#">Scala</a></li>
  <li class="lang-tab lang-tab-java"><a href="#">Java</a></li>
</ul>

<div class="tab-content">
<div class="tab-pane tab-pane-python active">
<div class="code code-tab">
{% highlight python %}
def inside(p):
    x, y = random.random(), random.random()
    return x*x + y*y < 1

count = sc.parallelize(range(0, NUM_SAMPLES)) \
             .filter(inside).count()
print("Pi is roughly %f" % (4.0 * count / NUM_SAMPLES))
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-scala">
<div class="code code-tab">
{% highlight scala %}
val count = sc.parallelize(1 to NUM_SAMPLES).filter { _ =>
  val x = math.random
  val y = math.random
  x*x + y*y < 1
}.count()
println(s"Pi is roughly ${4.0 * count / NUM_SAMPLES}")
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-java">
<div class="code code-tab">
{% highlight java %}
List<Integer> l = new ArrayList<>(NUM_SAMPLES);
for (int i = 0; i < NUM_SAMPLES; i++) {
  l.add(i);
}

long count = sc.parallelize(l).filter(i -> {
  double x = Math.random();
  double y = Math.random();
  return x*x + y*y < 1;
}).count();
System.out.println("Pi is roughly " + 4.0 * count / NUM_SAMPLES);
{% endhighlight %}
</div>
</div>
</div>

<h2>DataFrame API examples</h2>
<p>
In Spark, a <a href="https://spark.apache.org/docs/latest/sql-programming-guide.html#dataframes">DataFrame</a>
is a distributed collection of data organized into named columns.
Users can use DataFrame API to perform various relational operations on both external
data sources and Spark’s built-in distributed collections without providing specific procedures for processing data.
Also, programs based on DataFrame API will be automatically optimized by Spark’s built-in optimizer, Catalyst.
</p>

<h3>Text search</h3>
<p>In this example, we search through the error messages in a log file.</p>

<ul class="nav nav-tabs">
  <li class="lang-tab lang-tab-python active"><a href="#">Python</a></li>
  <li class="lang-tab lang-tab-scala"><a href="#">Scala</a></li>
  <li class="lang-tab lang-tab-java"><a href="#">Java</a></li>
</ul>

<div class="tab-content">
<div class="tab-pane tab-pane-python active">
<div class="code code-tab">
{% highlight python %}
textFile = sc.textFile("hdfs://...")

# Creates a DataFrame having a single column named "line"
df = textFile.map(lambda r: Row(r)).toDF(["line"])
errors = df.filter(col("line").like("%ERROR%"))
# Counts all the errors
errors.count()
# Counts errors mentioning MySQL
errors.filter(col("line").like("%MySQL%")).count()
# Fetches the MySQL errors as an array of strings
errors.filter(col("line").like("%MySQL%")).collect()
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-scala">
<div class="code code-tab">
{% highlight scala %}
val textFile = sc.textFile("hdfs://...")

// Creates a DataFrame having a single column named "line"
val df = textFile.toDF("line")
val errors = df.filter(col("line").like("%ERROR%"))
// Counts all the errors
errors.count()
// Counts errors mentioning MySQL
errors.filter(col("line").like("%MySQL%")).count()
// Fetches the MySQL errors as an array of strings
errors.filter(col("line").like("%MySQL%")).collect()
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-java">
<div class="code code-tab">
{% highlight java %}
// Creates a DataFrame having a single column named "line"
JavaRDD<String> textFile = sc.textFile("hdfs://...");
JavaRDD<Row> rowRDD = textFile.map(RowFactory::create);
List<StructField> fields = Arrays.asList(
  DataTypes.createStructField("line", DataTypes.StringType, true));
StructType schema = DataTypes.createStructType(fields);
DataFrame df = sqlContext.createDataFrame(rowRDD, schema);

DataFrame errors = df.filter(col("line").like("%ERROR%"));
// Counts all the errors
errors.count();
// Counts errors mentioning MySQL
errors.filter(col("line").like("%MySQL%")).count();
// Fetches the MySQL errors as an array of strings
errors.filter(col("line").like("%MySQL%")).collect();
{% endhighlight %}
</div>
</div>
</div>

<h3>Simple data operations</h3>
<p>
In this example, we read a table stored in a database and calculate the number of people for every age.
Finally, we save the calculated result to S3 in the format of JSON.
A simple MySQL table "people" is used in the example and this table has two columns,
"name" and "age".
</p>

<ul class="nav nav-tabs">
  <li class="lang-tab lang-tab-python active"><a href="#">Python</a></li>
  <li class="lang-tab lang-tab-scala"><a href="#">Scala</a></li>
  <li class="lang-tab lang-tab-java"><a href="#">Java</a></li>
</ul>

<div class="tab-content">
<div class="tab-pane tab-pane-python active">
<div class="code code-tab">
{% highlight python %}
# Creates a DataFrame based on a table named "people"
# stored in a MySQL database.
url = \
  "jdbc:mysql://yourIP:yourPort/test?user=yourUsername;password=yourPassword"
df = sqlContext \
  .read \
  .format("jdbc") \
  .option("url", url) \
  .option("dbtable", "people") \
  .load()

# Looks the schema of this DataFrame.
df.printSchema()

# Counts people by age
countsByAge = df.groupBy("age").count()
countsByAge.show()

# Saves countsByAge to S3 in the JSON format.
countsByAge.write.format("json").save("s3a://...")
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-scala">
<div class="code code-tab">
{% highlight scala %}
// Creates a DataFrame based on a table named "people"
// stored in a MySQL database.
val url =
  "jdbc:mysql://yourIP:yourPort/test?user=yourUsername;password=yourPassword"
val df = sqlContext
  .read
  .format("jdbc")
  .option("url", url)
  .option("dbtable", "people")
  .load()

// Looks the schema of this DataFrame.
df.printSchema()

// Counts people by age
val countsByAge = df.groupBy("age").count()
countsByAge.show()

// Saves countsByAge to S3 in the JSON format.
countsByAge.write.format("json").save("s3a://...")
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-java">
<div class="code code-tab">
{% highlight java %}
// Creates a DataFrame based on a table named "people"
// stored in a MySQL database.
String url =
  "jdbc:mysql://yourIP:yourPort/test?user=yourUsername;password=yourPassword";
DataFrame df = sqlContext
  .read()
  .format("jdbc")
  .option("url", url)
  .option("dbtable", "people")
  .load();

// Looks the schema of this DataFrame.
df.printSchema();

// Counts people by age
DataFrame countsByAge = df.groupBy("age").count();
countsByAge.show();

// Saves countsByAge to S3 in the JSON format.
countsByAge.write().format("json").save("s3a://...");
{% endhighlight %}
</div>
</div>
</div>

<h2>Machine learning example</h2>
<p>
<a href="https://spark.apache.org/docs/latest/mllib-guide.html">MLlib</a>, Spark’s Machine Learning (ML) library, provides many distributed ML algorithms.
These algorithms cover tasks such as feature extraction, classification, regression, clustering,
recommendation, and more. 
MLlib also provides tools such as ML Pipelines for building workflows, CrossValidator for tuning parameters,
and model persistence for saving and loading models.
</p>

<h3>Prediction with logistic regression</h3>
<p>
In this example, we take a dataset of labels and feature vectors.
We learn to predict the labels from feature vectors using the Logistic Regression algorithm.
</p>

<ul class="nav nav-tabs">
  <li class="lang-tab lang-tab-python active"><a href="#">Python</a></li>
  <li class="lang-tab lang-tab-scala"><a href="#">Scala</a></li>
  <li class="lang-tab lang-tab-java"><a href="#">Java</a></li>
</ul>

<div class="tab-content">
<div class="tab-pane tab-pane-python active">
<div class="code code-tab">
{% highlight python %}
# Every record of this DataFrame contains the label and
# features represented by a vector.
df = sqlContext.createDataFrame(data, ["label", "features"])

# Set parameters for the algorithm.
# Here, we limit the number of iterations to 10.
lr = LogisticRegression(maxIter=10)

# Fit the model to the data.
model = lr.fit(df)

# Given a dataset, predict each point's label, and show the results.
model.transform(df).show()
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-scala">
<div class="code code-tab">
{% highlight scala %}
// Every record of this DataFrame contains the label and
// features represented by a vector.
val df = sqlContext.createDataFrame(data).toDF("label", "features")

// Set parameters for the algorithm.
// Here, we limit the number of iterations to 10.
val lr = new LogisticRegression().setMaxIter(10)

// Fit the model to the data.
val model = lr.fit(df)

// Inspect the model: get the feature weights.
val weights = model.weights

// Given a dataset, predict each point's label, and show the results.
model.transform(df).show()
{% endhighlight %}
</div>
</div>

<div class="tab-pane tab-pane-java">
<div class="code code-tab">
{% highlight java %}
// Every record of this DataFrame contains the label and
// features represented by a vector.
StructType schema = new StructType(new StructField[]{
  new StructField("label", DataTypes.DoubleType, false, Metadata.empty()),
  new StructField("features", new VectorUDT(), false, Metadata.empty()),
});
DataFrame df = jsql.createDataFrame(data, schema);

// Set parameters for the algorithm.
// Here, we limit the number of iterations to 10.
LogisticRegression lr = new LogisticRegression().setMaxIter(10);

// Fit the model to the data.
LogisticRegressionModel model = lr.fit(df);

// Inspect the model: get the feature weights.
Vector weights = model.weights();

// Given a dataset, predict each point's label, and show the results.
model.transform(df).show();
{% endhighlight %}
</div>
</div>
</div>

<a name="additional"></a>
<h1>Additional examples</h1>

Many additional examples are distributed with Spark:

 * Basic Spark: [Scala examples](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples), [Java examples](https://github.com/apache/spark/tree/master/examples/src/main/java/org/apache/spark/examples), [Python examples](https://github.com/apache/spark/tree/master/examples/src/main/python)
 * Spark Streaming: [Scala examples](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples/streaming), [Java examples](https://github.com/apache/spark/tree/master/examples/src/main/java/org/apache/spark/examples/streaming)
