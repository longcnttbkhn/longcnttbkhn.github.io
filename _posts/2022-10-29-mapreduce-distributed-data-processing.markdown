---
title: "MapReduce Distributed Data Processing"
layout: post
date: 2022-10-29 22:12:50 +0700
image: /assets/images/blog/bigdata/2022-10-01/hadoop-logo.jpeg
headerImage: false
tag:
- bigdata
category: english
author: Lake
description: Hadoop MapReduce is a computational framework that allows writing applications that can process extremely large amounts of data (multiple terabytes) on multiple computers simultaneously. In this article, I will introduce MapReduce through a simple example, the WordCount problem.
---

## Contents
1. [Overview](#introduction)
2. [WordCount problem](#wordcount)
3. [Overview of MapReduce program operations](#mapreduce)
4. [Conclusion](#conclusion)

## Overview <a name="introduction"></a>

In the previous article, I introduced HDFS - Distributed File System (you can review [here](/hdfs-distributed-file-system/)). With HDFS, we have a system that can store unlimited data (not dependent on hardware). To process extremely large amounts of data, distributed storage on HDFS nodes, we need a calculation method called MapReduce.

Instead of data being concentrated on one node for processing (costly, impossible), the MapReduce program will be sent to the nodes that have data and use the resources of that node to calculate and store the results. This whole process is done automatically, the user (programmer) only needs to define 2 functions Map and Reduce:

- *map*: is a transformation function, takes as input a pair of <Key, Value> and needs to return 1 or more new pairs of <Key, Value>.

- *reduce*: is a synthesis function, takes as input a pair of <Key, Value[]> in which the input values ​​list contains all values ​​with the same key and needs to return a pair of <Key, Value> as result.

## WordCount Problem <a name="wordcount"></a>

*Problem:* Given a data file (log) stored on HDFS, count the number of times a word appears in the file and write the result on HDFS. Each word is separated by a space.

*Pseudocode*

{% highlight ruby %}
map(String key, String value):
    // key:
    // value: document content
    for each word w in value:
        Emit(w, 1);

reduce(String key, Iterator values):
    // key: a word
    // values: a list of counts
    int sum = 0;
    for each v in values:
        sum += v;
    Emit(key, sum);
{% endhighlight %}

In this program, the *map* function will split the input text into words, with each separated word returning a pair of words with the same count of 1. The *reduce* function receives input as a word and a list of all the counts of that word, it will calculate the total count to get the number of occurrences of that word.

### Install and run on Hadoop cluster

I will use the docker containers built from [this](/how-to-install-hadoop-cluster-on-ubuntu/) article to install and test the WordCount program.

Start HDFS

```sh
root@node01:~# $HADOOP_HOME/sbin/start-dfs.sh
```

Create a file on HDFS as input for the Wordcount problem

```sh
root@node01:~# hdfs dfs -D dfs.replication=2 -appendToFile /lib/hadoop/logs/*.log /input_wordcount.log
```

Source code Java

{% highlight java %}
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
{% endhighlight %}

> Save the file as `WordCount.java` in the wordcount folder

Add environment variables

```sh
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
```

Compile code

```sh
root@node01:~/wordcount# hadoop com.sun.tools.javac.Main WordCount.java
root@node01:~/wordcount# jar cf wc.jar WordCount*.class
```

Run and check the results

```sh
root@node01:~/wordcount# hadoop jar wc.jar WordCount /input_wordcount.log /ouput_wordcount 
root@node01:~/wordcount# hdfs dfs -cat /ouput_wordcount/part-r-00000
"script".	1
#1	10
#110	1
#129	1
...
```
## Overview of MapReduce program operations <a name="mapreduce"></a>

The MapReduce program execution process can be summarized in the following stages:

- *Init:* An `ApplicationMaster` (AM) is initialized and maintained until the program ends, its task is to manage and coordinate the execution tasks on the nodes. We will learn more about AM in the following article.
- *Map:* AM identifies the nodes with input data on HDFS and requests them to perform the input data transformation according to the instructions written by the user in the `map` function
- *Shuffle and Sort:* The <Key, Value> pairs resulting from the `map` function will be shuffled between nodes and sorted by key. <Key, Value> pairs with the same key will be grouped together.
- *Reduce:* In this phase, the <Key, Value> pairs with the same key will be processed according to the instructions written in the `reduce` function to get the final result.

When processing large amounts of data, AM will create many tasks (each task processes a block) and execute them on many different nodes to increase performance. If during the running process, a node is broken, AM will transfer the tasks of that node to execute on another node without affecting the other tasks and the entire program.

## Conclusion <a name="conclusion"></a>

Through this article, I have introduced the most basic issues about the MapReduce computation model through an example with the Wordcount problem. You can learn more about MapReduce in [this article][google-mapreduce] of Google. See you in the next articles!

[google-mapreduce]: https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf
