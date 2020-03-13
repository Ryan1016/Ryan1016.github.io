---
title: WordCount
layout: post
categories: BigData
---

## һ��ʵ��Ŀ��
��ϤHDFS�������в�����Java����HDFS�ķ���

## ����ʵ��ƽ̨
- ����ϵͳ��Linux
- Hadoop�汾��2.7.1��
- Maven

## ����ʵ�鲽��

**1. ����Maven**

**2. ����Maven����Դ���½�Settings.xml�ļ����������������**

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository/>
  <interactiveMode/>
  <usePluginRegistry/>
  <offline/>
  <pluginGroups/>
  <servers/>
  <mirrors>
    <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>https://maven.aliyun.com/repository/public </url>
    </mirror>
  </mirrors>
  <proxies/>
  <profiles/>
  <activeProfiles/>
</settings>

```

��Eclipse�˵�������ѡ��Windows->Preferences->Maven->User Settings�У�ѡ����һ�����½���Settings.xml�ļ�
![set setting.xml](https://img-blog.csdnimg.cn/20200108151223141.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNDIyNDQ4,size_16,color_FFFFFF,t_70)

**3. ����Maven Project**

![����Maven Project](https://img-blog.csdnimg.cn/20200108151542978.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNDIyNDQ4,size_16,color_FFFFFF,t_70)ѡ�� Maven-archetype-quickstart ѡ��
![ѡ�� Maven-archetype-quickstart ѡ��](https://img-blog.csdnimg.cn/20200108151612875.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNDIyNDQ4,size_16,color_FFFFFF,t_70)

**4. ���Maven������Ҽ���Ŀ����ѡ��Maven��>Add Dependency������Hadoop-client����**

![���������ͼƬ����](https://img-blog.csdnimg.cn/20200108151714933.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNDIyNDQ4,size_16,color_FFFFFF,t_70)

**5. ��дWordCount����**

- ����һ��txt�ļ�������֮���ÿո�" "����������WordCount����ȡ���ļ�

- WordCountMapper

```java

import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;


public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        // ���ı�������ת����String
        String line = value.toString();
        
        // 2���ݿո���һ���зֳɵ���
        String[] words = line.split(" ");
        
        // ���������Ϊ<���ʣ�1>
        for(String word:words){
            // ��������Ϊkey��������1��Ϊvalue,�Ա��ں��������ݷַ������Ը��ݵ��ʷַ����Ա�����ͬ���ʻᵽ��ͬ��reducetask��
            context.write(new Text(word), new IntWritable(1));
        }
    }
}

```

- WordCountReducer

```java

import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordCountReduce extends Reducer<Text, IntWritable, Text, IntWritable> {

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {

        int count = 0;

        // ���ܸ���key�ĸ���
        for(IntWritable value:values){
            count +=value.get();
        }
        
        // ����ض�key���ܴ���
        context.write(key, new IntWritable(count));
    }
}

```

- WordCount

```java

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;


public class WordCount {
    public static void main(String[] args) throws Exception {
        //��ȡ������Ϣ������job����ʵ��
        Configuration configuration = new Configuration();
        //�����ύ��yarn������,windows��Linux������һ��
//        configuration.set("mapreduce.framework.name", "yarn");
//        configuration.set("yarn.resourcemanager.hostname", "node22");
        Job job = Job.getInstance(configuration);
        
        job.setJarByClass(WordCount.class);
        
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReduce.class);
        
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        
        // ָ��job������ԭʼ�ļ�����Ŀ¼
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        
        //  ��job�����õ���ز������Լ�job���õ�java�����ڵ�jar���� �ύ��yarnȥ����
//        job.submit();
        boolean result = job.waitForCompletion(true);
        System.exit(result?0:1);
    }
}

```

**6. �ύ���ֲ�ʽ ϵͳ��������**

- ������Hadoop

```powershell
cd /usr/local/hadoop
./sbin/start-dfs.sh
```

- �ٰ��½����ļ�wordcount.txt�ϴ���hdfs�ϣ�

```powershell
./bin/hadoop fs -put -f ./wordcount.txt /input
```

- ���WordCount���룬����һ��jar��,��ΪWordCount��ѡ������ΪWordCount��������/usr/local/hadoop�У�
- ����jar�ļ���

```powershell
hadoop jar /usr/local/hadoop/WordCount.jar /input /output
```

- �鿴�������н����

```powershell
hadoop fs -ls /output
```

- �鿴������

```powershell
 hadoop fs -cat /output/part-r-00000
```

![������](https://img-blog.csdnimg.cn/20200108153523882.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNDIyNDQ4,size_1,color_FFFFFF,t_70)




---
---
**��ӭ�鿴�ҵ�CSDN���ͣ�[Welcome To Ryan's Home](https://blog.csdn.net/qq_41422448)**

---
---