---
layout: post
title:  "Hadoop: The Definitive Guide (3)"
categories: Hadoop
tags:  Hadoop BigData MapReduce
---

* content
{:toc}


## MapReduce



map: (K1, V1) → list(K2, V2)
combiner: (K2, list(V2)) → list(K2, V2)
reduce: (K2, list(V2)) → list(K3, V3)
partition: (K2, V2) → integer

Classes


```java
public class Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT> {
  public class Context extends MapContext<KEYIN, VALUEIN, KEYOUT, VALUEOUT> {
    // ...
  }
  protected void map(KEYIN key, VALUEIN value, 
      Context context) throws IOException, InterruptedException {
    // ...
    context.write(KEYOUT key, VALUEOUT value);
  }
}

public class Reducer<KEYIN, VALUEIN, KEYOUT, VALUEOUT> {
  public class Context extends ReducerContext<KEYIN, VALUEIN, KEYOUT, VALUEOUT> {
    // ...
  }
  protected void reduce(KEYIN key, Iterable<VALUEIN> values,
      Context context) throws IOException, InterruptedException {
    // ...
    context.write(KEYOUT key, VALUEOUT value);
  }
}


public abstract class Partitioner<KEY, VALUE> {
  public abstract int getPartition(KEY key, VALUE value, int numPartitions);
}


```

Minimal MR program
```java
public class MinimalMapReduceWithDefaults extends Configured implements Tool {
  
  @Override
  public int run(String[] args) throws Exception {
     if (args.length != 2) {
      System.err.printf("Usage: %s [genericOptions] %s\n\n",
        this.getClass().getSimpleName(), "<input> <output>");
      GenericOptionsParser.printGenericCommandUsage(System.err);
      return null;
    }
    Job job = new Job(conf);
    job.setJarByClass(tool.getClass());
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));

    if (job == null) {
      return -1;
    }
    
    job.setInputFormatClass(TextInputFormat.class);
    
    job.setMapperClass(Mapper.class);
    
    job.setMapOutputKeyClass(LongWritable.class);
    job.setMapOutputValueClass(Text.class);
    
    job.setPartitionerClass(HashPartitioner.class);
    
    job.setNumReduceTasks(1); // the number of map tasks is equal to the number of splits that the input is turned into
    job.setReducerClass(Reducer.class);

    job.setOutputKeyClass(LongWritable.class);
    job.setOutputValueClass(Text.class);

    job.setOutputFormatClass(TextOutputFormat.class);
    
    return job.waitForCompletion(true) ? 0 : 1;
  }
  
  public static void main(String[] args) throws Exception {
    int exitCode = ToolRunner.run(new MinimalMapReduceWithDefaults(), args);
    System.exit(exitCode);
  }
}
```
