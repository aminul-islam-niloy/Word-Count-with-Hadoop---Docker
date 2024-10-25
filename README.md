
# Word Count with Hadoop - Docker

This guide demonstrates how to set up a Hadoop pseudo-distributed environment in Docker and run a Word Count MapReduce job. The process includes running a Hadoop Docker container, creating input data, writing a Java MapReduce program, compiling it, and retrieving the output.

## Prerequisites

Ensure Docker is installed on your system. or install https://docs.docker.com/desktop/install/windows-install/

## Docker Setup and Initial Commands

1. **Run the Docker Container**  
   Execute the following command to start the Hadoop container with the appropriate ports and volume mapping:
   ```bash
   docker run -p 9870:9870 -p 8088:8088 -v D:/hedoop:/home/hadoop/data -it --name=hadoop macio232/hadoop-pseudo-distributed-mode
   ```

2. **Start Hadoop Services**  
   To start Hadoop services inside the Docker container:
   ```bash
   docker start hadoop
   docker exec -it hadoop /bin/bash
   start-all.sh
   ```

3. **Verify Hadoop Web UI**  
   - Access the Hadoop NameNode UI at: [http://localhost:9870/explorer.html#/](http://localhost:9870/explorer.html#/)
   - Resource Manager can be accessed at [http://localhost:8088](http://localhost:8088)

## Preparing Input Data

1. **Create Input Text File**  
   Navigate to the shared data folder:
   ```bash
   cd /home/hadoop/data
   echo "Hello Hadoop Hello Docker This is a MapReduce Program" > data119.txt
   ```

2. **Upload File to HDFS**  
   Create a directory and upload the input file:
   ```bash
   hdfs dfs -mkdir -p /user/hadoop/folder119
   hdfs dfs -put /home/hadoop/data/data119.txt /user/hadoop/folder119/
   ```

## Writing the WordCount MapReduce Program

Create a `WordCount.java` or `vi WordCount.java` and select `i for insert` mode, esc for exit and `:wq` for save and go to bashe . In WordCount file, follow this code:
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import java.io.IOException;
import java.util.StringTokenizer;

public class WordCount {
    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
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
```

## Compiling and Running the MapReduce Job

1. **Compile the Code**  
   ```bash
   javac -classpath `hadoop classpath` -d . WordCount.java
   jar cf wordcount119.jar WordCount*.class
   ```

2. **Run the Job**  
   ```bash
   hadoop jar wordcount119.jar WordCount /user/hadoop/folder119 /user/hadoop/output119
   ```

3. **View Output**  
   ```bash
   hdfs dfs -cat /user/hadoop/output119/part-r-00000
   ```

## Retrieving Output to Local Machine

1. **Get Output File from HDFS**  
   ```bash
   hdfs dfs -get /user/hadoop/output119/part-r-00000 /home/hadoop/data/
   ```

## HDFS Commands Summary

- **List Files in HDFS**  
  ```bash
  hdfs dfs -ls /user/hadoop/output119
  ```

- **Remove Output Directory**  
  ```bash
  hdfs dfs -rm -r /user/hadoop/output119
  ```

- **Remove All Files in HDFS User Directory**  
  ```bash
  hdfs dfs -rm -r -SkipTrash /user/*
  ```

## Notes

- **Start/Stop Hadoop**  
  To start Hadoop services, use `start-all.sh`, and for shutdown, use `stop-all.sh`.
- **Text Editor**  
  Use `vi` to create or edit files within the Docker container:
  - `i`: Enter insert mode.
  - `Esc`: Exit insert mode.
  - `:wq`: Save and quit.
  - `:q!`: Quit without saving.

---

This README provides the steps necessary to set up and run a basic Hadoop Word Count MapReduce job in a Docker container. Modify paths and commands as needed for your setup.