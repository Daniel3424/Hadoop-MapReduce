# MapReduce
***
맵리듀스는 간단한 단위작업을 반복하여 처리할 때 사용하는 프로그래밍 모델입니다. 간단한 단위작업을 처리하는 맵(Map) 작업과 맵 작업의 결과물을 모아서 집계하는 리듀스(Reduce) 단계로 구성됩니다.

하둡에서 분산처리를 담당하는 맵리듀스 작업은 맵과 리듀스로 나누어져 처리됩니다. 맵, 리듀스작업은 병렬로 처리가 가능한 작업으로, 여러 컴퓨터에서 동시에 작업을 처리하여 속도를 높일 수 있습니다.

![mp](https://www.supinfo.com/articles/resources/207908/2807/3.png)

# MapReduce 작업 단위
***
하둡 v1의 작업 단위는 잡(job)이고, 하둡 v2의 작업 단위는 애플리케이션(application) 입니다. YARN 아키텍처가 도입되면서 이름은 변경되었지만 동일하게 관리됩니다. 잡은 맵 태스크와 리듀스 태스크로 나누어 집니다. 태스크는 어템프트(attempt) 단위로 실행됩니다.

하둡 잡이 생성되면 아이디가 job_xxx_xxx 로 생성됩니다. 이 아이디로 잡의 상태, 로그를 확인할 수 있습니다. YARN에서는 application_xxx_xxx 로 확인할 수 있습니다. 접두어는 다르지만 같은 작업입니다.

잡에서 생성되는 맵태스크와 리듀스태스크는 아이디가 attempt_xxx_xxx_m_000000_0으로 생성됩니다. 맵태스크는 중간아이디가 m으로 생성되고, 리듀스 태스크는 r로 생성됩니다.

```
잡아이디: job_1520227878653_30484
애플리케이션 아이디: application_1520227878653_30484
어템프트 아이디: attempt_1520227878653_30484_m_000000_0
```
# MapReduce Failover(작업극복)
***
맵리듀스는 실행 중 오류가 발생하면 설정1된 횟수만큼 자동으로 반복됩니다. 반복후에도 오류가 발생하면 작업을 종료합니다.

다음은 작업을 실행하고 오류가 발생한 로그입니를 보면 잡(job_1520227878653_30484)이 생성되고 실행되는 중 오류가 발생하여 맵잡의 어템프트(attempt_1520227878653_30484_m_000000_0)가 반복 되고 있습니다. 잡 아이디 마지막의 숫자가 증가하면서 반복 횟수를 알려줍니다.

```
$ hadoop jar cctv.jar com.sec.cctv.CctvMain  /user/cctv/ /user/cctv_output/ 
18/10/19 08:22:42 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
18/10/19 08:22:42 INFO input.FileInputFormat: Total input paths to process : 1
18/10/19 08:22:42 INFO lzo.GPLNativeCodeLoader: Loaded native gpl library
18/10/19 08:22:42 INFO lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 418fa8c602f2a4b153c1a89806305f6b5a27a524]
18/10/19 08:22:42 INFO mapreduce.JobSubmitter: number of splits:1
18/10/19 08:22:42 INFO Configuration.deprecation: mapred.job.queue.name is deprecated. Instead, use mapreduce.job.queuename
18/10/19 08:22:42 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1520227878653_30484
18/10/19 08:22:43 INFO impl.YarnClientImpl: Submitted application application_1520227878653_30484
/application_1520227878653_30484/
18/10/19 08:22:43 INFO mapreduce.Job: Running job: job_1520227878653_30484
18/10/19 08:22:48 INFO mapreduce.Job: Job job_1520227878653_30484 running in uber mode : false
18/10/19 08:22:48 INFO mapreduce.Job:  map 0% reduce 0%
18/10/19 08:22:51 INFO mapreduce.Job: Task Id : attempt_1520227878653_30484_m_000000_0, Status : FAILED
Error: java.io.IOException: Type mismatch in value from map: expected org.apache.hadoop.io.Text, received org.apache.hadoop.io.IntWritable
    at org.apache.hadoop.mapred.MapTask$MapOutputBuffer.collect(MapTask.java:1095)
    at org.apache.hadoop.mapred.MapTask$NewOutputCollector.write(MapTask.java:724)
    at org.apache.hadoop.mapreduce.task.TaskInputOutputContextImpl.write(TaskInputOutputContextImpl.java:89)
    at org.apache.hadoop.mapreduce.lib.map.WrappedMapper$Context.write(WrappedMapper.java:112)
    at com.sec.cctv.CctvMapper.map(CctvMapper.java:17)
    at com.sec.cctv.CctvMapper.map(CctvMapper.java:9)
    at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:146)
    at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:796)
    at org.apache.hadoop.mapred.MapTask.run(MapTask.java:342)
    at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:164)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:422)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1698)
    at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:158)

18/10/19 08:22:54 INFO mapreduce.Job: Task Id : attempt_1520227878653_30484_m_000000_1, Status : FAILED
Error: java.io.IOException: Type mismatch in value from map: expected org.apache.hadoop.io.Text, received org.apache.hadoop.io.IntWritable
    at org.apache.hadoop.mapred.MapTask$MapOutputBuffer.collect(MapTask.java:1095)
    at org.apache.hadoop.mapred.MapTask$NewOutputCollector.write(MapTask.java:724)
    at org.apache.hadoop.mapreduce.task.TaskInputOutputContextImpl.write(TaskInputOutputContextImpl.java:89)
    at org.apache.hadoop.mapreduce.lib.map.WrappedMapper$Context.write(WrappedMapper.java:112)
    at com.sec.cctv.CctvMapper.map(CctvMapper.java:17)
    at com.sec.cctv.CctvMapper.map(CctvMapper.java:9)
    at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:146)
    at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:796)
    at org.apache.hadoop.mapred.MapTask.run(MapTask.java:342)
    at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:164)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:422)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1698)
    at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:158)

18/10/19 08:22:57 INFO mapreduce.Job: Task Id : attempt_1520227878653_30484_m_000000_2, Status : FAILED
Error: java.io.IOException: Type mismatch in value from map: expected org.apache.hadoop.io.Text, received org.apache.hadoop.io.IntWritable
    at org.apache.hadoop.mapred.MapTask$MapOutputBuffer.collect(MapTask.java:1095)
    at org.apache.hadoop.mapred.MapTask$NewOutputCollector.write(MapTask.java:724)
    at org.apache.hadoop.mapreduce.task.TaskInputOutputContextImpl.write(TaskInputOutputContextImpl.java:89)
    at org.apache.hadoop.mapreduce.lib.map.WrappedMapper$Context.write(WrappedMapper.java:112)
    at com.sec.cctv.CctvMapper.map(CctvMapper.java:17)
    at com.sec.cctv.CctvMapper.map(CctvMapper.java:9)
    at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:146)
    at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:796)
    at org.apache.hadoop.mapred.MapTask.run(MapTask.java:342)
    at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:164)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:422)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1698)
    at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:158)

18/10/19 08:23:02 INFO mapreduce.Job:  map 100% reduce 100%
18/10/19 08:23:02 INFO mapreduce.Job: Job job_1520227878653_30484 failed with state FAILED due to: Task failed task_1520227878653_30484_m_000000
Job failed as tasks failed. failedMaps:1 failedReduces:0

18/10/19 08:23:02 INFO mapreduce.Job: Counters: 13
    Job Counters 
        Failed map tasks=4
        Killed reduce tasks=7
        Launched map tasks=4
        Other local map tasks=3
        Data-local map tasks=1
        Total time spent by all maps in occupied slots (ms)=327375
        Total time spent by all reduces in occupied slots (ms)=0
        Total time spent by all map tasks (ms)=7275
        Total time spent by all reduce tasks (ms)=0
        Total vcore-milliseconds taken by all map tasks=7275
        Total vcore-milliseconds taken by all reduce tasks=0
        Total megabyte-milliseconds taken by all map tasks=10476000
        Total megabyte-milliseconds taken by all reduce tasks=0
```

# Map InputSplit
***
맵의 입력은 스플릿(InputSplit)단위로 분할됩니다. 맵작업은 큰 데이터를 하나의 노드에서 처리하지 않고, 분할하여 동시에 병렬 처리하여 작업 시간을 단축합니다.

스플릿이 작으면 작업 부하가 분산되어 성능을 높을 수 있습니다. 하지만 스플릿의 크기가 너무 작으면 맵 작업의 개수가 증가하고 맵 작업 생성을 위한 오버헤드가 증가하여 작업이 느려질 수 있습니다. 따라서 작업에 따라 적절한 개수의 맵 작업을 생성해야 합니다. 일반적으로 맵 작업의 적절한 스플릿 크기는 데이터 지역성의 이점을 얻을 수 있는 HDFS 블록의 기본 크기(128MB)입니다.

# Map 작업 데이터 지역성
***
맵 작업은 HDFS에 입력 데이터가 있는 노드에서 실행할 때 가장 빠르게 동작합니다. 클러스터의 네트워크 대역을 사용하지 않고 처리할 수 있기 때문입니다. 데이터가 있는 노드에서 작업을 처리할 수 없다면 동일한 랙의 노드, 다른 랙의 노드 순서로 맵 작업이 실행가능한 노드를 찾습니다.

맵 작업의 적절한 스플릿 크기가 HDFS 블록의 기본크기인 이유는 단일 노드에 해당 블록이 모두 저장된다고 확신할 수 있는 입력 크기이기 때문입니다. 스플릿 크기가 블록의 기본 크기일때 맵 작업이 로컬 디스크의 데이터만 이용하여, 다른 노드에서 데이터를 전송받아 처리할 때 보다 빠르게 작업을 처리할 수 있습니다.

맵 작업의 결과는 로컬 디스크에 임시 저장 됩니다. 맵 작업의 결과는 리듀스 작업의 입력으로 쓰이는 임시 결과물이기 때문입니다. 리듀스 작업은 맵 작업의 결과를 입력으로 받기 때문에 지역성의 장점이 없습니다. 리듀스 작업의 결과는 HDFS에 저장됩니다.

리듀스 작업의 개수는 입력 크기와 상관없이 지정할 수 있습니다. 리듀스가 여러개이면 리듀스의 개수 만큼 파티션을 생성하고 맵의 결과를 각 파티션에 분배합니다. 파티션별로 키가 존재하고 동일한 키는 같은 파티션에 전달됩니다.

![mpq](https://lovingtocode.files.wordpress.com/2015/09/asdf1.jpg)

# MapReduce 작업의 종류
***
맵리듀스는 리듀서 작업이 있는 경우와 없는 경우가 있습니다. 파일을 읽어서 바로 쓰는 작업의 경우 리듀서가 필요 없어서 매퍼만 있는 작업(Mapper Only)이 됩니다. 집계를 진행해야 해서 리듀서가 필요한 경우 정렬이 필요한 경우는 리듀서가 하나만 생성됩니다. 나머지의 경우 리듀서가 여러개로 생성됩니다. 각 작업의 최종 매퍼, 리듀서의 수만큼 파일이 생성됩니다.

# Reducer가 하나인 경우 
***
모든 데이터의 정렬작업 같은 경우입니다. 리듀서 하나로 모든 작업을 처리하기 때문에 시간이 오래걸립니다.

![reducer](https://autofei.files.wordpress.com/2010/06/2-2.png?w=300&h=165)

# Reducer가 여러개인 경우 
***
일반적인 집계 작업의 경우 리듀서가 여러개 생성됩니다. 리듀서의 수만큼 파일이 생성됩니다. HDFS의 부하를 방지하기 위해서 추가적인 파일 머지 작업이 필요할 수 있습니다.

![reducer](https://autofei.files.wordpress.com/2010/06/2-3.png?w=300)

# Reducer가 없는 경우(Mapper Only 작업) 
***
원천 데이터를 읽어서 가공을하고 바로 쓰는 경우입니다. 리듀서 작업이 없기 때문에 빠릅니다. 매퍼의 수만큼 파일이 생성되기 때문에 추가적인 파일 머지 작업이 필요할 수 있습니다.

![reducer](https://autofei.files.wordpress.com/2010/06/2-4.png?w=300&h=254)

