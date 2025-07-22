---
title: "MapReduce - Xử lý dữ liệu phân tán"
layout: post
date: 2022-10-29 21:12:50 +0700
image: /assets/images/blog/bigdata/2022-10-01/hadoop-logo.jpeg
headerImage: false
tag:
- bigdata
category: blog
author: Long Nguyen
description:  Hadoop MapReduce là một framework tính toán cho phép viết các ứng dụng có khả năng xử lý được lượng dữ liệu cực lớn (nhiều terabyte) trên nhiều máy tính đồng thời. Trong bài viết này mình sẽ giới thiệu về MapReduce thông qua một ví dụ đơn giản là bài toán WordCount.
---

## Nội dung
1. [Giới thiệu tổng quan](#introduction)
2. [Bài toán WordCount](#wordcount)
3. [Sơ lược hoạt động trong chương trình MapReduce](#mapreduce)
4. [Kết luận](#conclusion)

## Giới thiệu tổng quan <a name="introduction"></a>

Trong bài viết trước mình đã giới thiệu về HDFS - Hệ thống File phân tán (bạn có thể xem lại [tại đây](/hdfs-he-thong-file-phan-tan/)). Với HDFS chúng ta có được một hệ thống có khả năng lưu trữ dữ liệu vô hạn (không bị phụ thuộc vào phần cứng). Để xử lý được lượng dữ liệu cực lớn, lưu trữ phân tán trên các node của HDFS, chúng ta cần một phương pháp tính toán gọi là MapReduce.

Dữ liệu thay vì được tập trung lại một node để xử lý (tốn nhiều chi phí, bất khả thi) thì chương trình MapReduce sẽ được gửi đến các node đang có dữ liệu và sử dụng tài nguyên của chính node đó cho việc tính toán và lưu trữ kết quả. Toàn bộ quá trình này được thực hiện tự động, người dùng (lập trình viên) chỉ cần định nghĩa 2 hàm Map và Reduce:

- *map*: là hàm biến đổi, nhận đầu vào là một cặp <Key, Value> và cần trả về 1 hoặc nhiều cặp <Key, Value> mới.
- *reduce*: là hàm tổng hợp, nhận đầu vào là cặp <Key, Value[]> trong đó list values đầu vào chứa tất cả các value có cùng key và cần trả về 1 cặp <Key, Value> kết quả.

## Bài toán WordCount <a name="wordcount"></a>

*Đề bài:* Cho một file dữ liệu (log) được lưu trữ trên HDFS, đếm số lần 1 từ xuất hiện trong file và ghi kết quả trên HDFS. Mỗi từ cách nhau bởi dấu cách.

*Mã giả*

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

Trong chương trình này, hàm *map* sẽ chia văn bản đầu vào thành các từ, với mỗi từ tách ra được sẽ trả về một cặp từ cùng số đếm 1. Hàm *reduce* nhận đầu vào là từ cùng danh sách tất cả số đếm của từ đó, nó sẽ thực hiện tính tổng số đếm để được số lần xuất hiện của từ đó.

### Cài đặt và chạy trên Hadoop cluster

Mình sẽ sử dụng các docker container đã build từ bài viết [này](/huong-dan-cai-hadoop-cluster/) để cài đặt và thử nghiệm chương trình WordCount.

Start HDFS

```sh
root@node01:~# $HADOOP_HOME/sbin/start-dfs.sh
```

Tạo 1 file trên HDFS làm input cho bài toán Wordcount

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

> Lưu file với tên file `WordCount.java` trong thư mục wordcount

Bổ sung biến môi trường

```sh
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
```

Compile code

```sh
root@node01:~/wordcount# hadoop com.sun.tools.javac.Main WordCount.java
root@node01:~/wordcount# jar cf wc.jar WordCount*.class
```

Chạy và kiểm tra kết quả

```sh
root@node01:~/wordcount# hadoop jar wc.jar WordCount /input_wordcount.log /ouput_wordcount 
root@node01:~/wordcount# hdfs dfs -cat /ouput_wordcount/part-r-00000
"script".	1
#1	10
#110	1
#129	1
...
```

## Sơ lược hoạt động trong chương trình MapReduce <a name="mapreduce"></a>

Quá trình thực thi chương trình MapReduce có thể tóm tắt lại trong các giai đoạn sau:

- *Init:* Một `ApplicationMaster`(AM) được khởi tạo và duy trì cho đến khi chương trình kết thúc, nhiệm vụ của nó là quản lý điều phối các task thực thi trên các node. Chúng ta sẽ tìm hiểu kỹ hơn về AM trong bài viết sau.
- *Map:* AM xác định các node có dữ liệu đầu vào trên HDFS và yêu cầu chúng thực hiện việc biến đổi dữ liệu đầu vào theo chỉ dẫn được người dùng viết trong hàm `map`
- *Shuffle and Sort:* Các cặp <Key, Value> kết quả từ hàm `map` sẽ được trộn giữa các node và sắp xếp theo key. Các cặp <Key, Value> có cùng key sẽ được gom nhóm lại với nhau.
- *Reduce:* Trong giai đoạn này, các cặp <Key, Value> cùng key sẽ được xử lý theo chỉ dẫn được viết trong hàm `reduce` để ra được kết quả cuối cùng.

Khi xử lý những dữ liệu có khối lượng lớn, AM sẽ tạo ra nhiều task (mỗi task xử lý một block) và thực thi chúng trên nhiều node khác nhau để tăng hiệu năng. Nếu trong quá trình chạy có 1 node bị hỏng, AM sẽ chuyển các task của node đó sang thực thi trên node khác mà không làm ảnh hưởng đến các task khác và toàn bộ chương trình.

## Kết luận <a name="conclusion"></a>

Qua bài viết này mình đã giới thiệu những vấn đề cơ bản nhất về mô hình tính toán MapReduce thông qua ví dụ với bài toán Wordcount. Bạn có thể tìm hiểu thêm về MapReduce trong [bài báo này][google-mapreduce] của Google. Hẹn gặp lại trong các bài viết sau!

[google-mapreduce]: https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf
