Validating your virtual machine setup
=====================================

After the `vagrant up` command has completed, you'll have a CentOS
virtual machine with the following installed:

* Hadoop HDFS
* Hadoop YARN
* Hive
* Spark

Let's take a look at each one and validate that it's installed and
setup as expected.

SSH into your virtual machine.

    vagrant ssh

Run an example MapReduce job.

    # create a directory for the input file
    hdfs dfs -mkdir wordcount-input

    # generate sample input and write it to HDFS
    echo "hello dear world hello" | hdfs dfs -put - wordcount-input/hello.txt

    # OR copy from local
    hdfs dfs -copyFromLocal /vagrant/sources/wordcount/datasets/input.txt wordcount-input

    hdfs dfs -ls wordcount-input

    # run the MapReduce word count example from local
    hadoop jar /vagrant/sources/wordcount/jar/wordcount-mr-0.0.1.jar com.example.wordcount_mr.WordCount wordcount-input/input.txt wordcount-output

    # run the MapReduce word count example
    hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop*example*.jar \
      wordcount wordcount-input wordcount-output

    # validate the output of the job - you should see the following in the output:
    #     dear   1
    #     hello  2
    #     world  1
    hdfs dfs -cat wordcount-output/part*

Launch the Hive shell.

    $ hive

Create a table and run a query over it.

    CREATE EXTERNAL TABLE wordcount (
        word STRING,
        count INT
    )
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/user/vagrant/wordcount-output';

    select * from wordcount order by count;

Next launch the interactive Spark shell.

    spark-shell --master yarn

Run word count in Spark.

// path: hdfs://10.211.55.101/user/root/wordcount-input

    // enter paste mode
    :paste
    sc.textFile("hdfs:///user/root/wordcount-input/input.txt")
       .flatMap(line => line.split(" "))
       .map(word => (word, 1))
       .reduceByKey(_ + _)
       .collect.foreach(println)

    <ctrl-D>

    sc.stop

# Test Spark
run clickstrem.sh

 spark-submit --class org.mahedi.clickstream.ClickStream --master spark://10.211.55.101:7077 /vagrant/sources/clickstream/jar/clickstream-spark-0.0.1-SNAPSHOT.jar hdfs://10.211.55.101/user/root/clickstream-input/msnbc990928.seq