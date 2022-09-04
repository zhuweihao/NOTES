



spark使用jdk11时会出现如下警告

```
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.apache.spark.unsafe.Platform (file:/C:/Users/ZWH/.m2/repository/org/apache/spark/spark-unsafe_2.13/3.3.0/spark-unsafe_2.13-3.3.0.jar) to constructor java.nio.DirectByteBuffer(long,int)
WARNING: Please consider reporting this to the maintainers of org.apache.spark.unsafe.Platform
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
```







```xml


22/07/10 23:03:28 INFO CoarseGrainedExecutorBackend: Started daemon with process name: 7770@master
22/07/10 23:03:28 INFO SignalUtils: Registering signal handler for TERM
22/07/10 23:03:28 INFO SignalUtils: Registering signal handler for HUP
22/07/10 23:03:28 INFO SignalUtils: Registering signal handler for INT
22/07/10 23:03:29 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
22/07/10 23:03:29 INFO SecurityManager: Changing view acls to: zhuweihao
22/07/10 23:03:29 INFO SecurityManager: Changing modify acls to: zhuweihao
22/07/10 23:03:29 INFO SecurityManager: Changing view acls groups to: 
22/07/10 23:03:29 INFO SecurityManager: Changing modify acls groups to: 
22/07/10 23:03:29 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(zhuweihao); groups with view permissions: Set(); users  with modify permissions: Set(zhuweihao); groups with modify permissions: Set()
22/07/10 23:03:30 INFO TransportClientFactory: Successfully created connection to master/192.168.0.107:45781 after 120 ms (0 ms spent in bootstraps)
22/07/10 23:03:30 INFO SecurityManager: Changing view acls to: zhuweihao
22/07/10 23:03:30 INFO SecurityManager: Changing modify acls to: zhuweihao
22/07/10 23:03:30 INFO SecurityManager: Changing view acls groups to: 
22/07/10 23:03:30 INFO SecurityManager: Changing modify acls groups to: 
22/07/10 23:03:30 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(zhuweihao); groups with view permissions: Set(); users  with modify permissions: Set(zhuweihao); groups with modify permissions: Set()
22/07/10 23:03:30 INFO TransportClientFactory: Successfully created connection to master/192.168.0.107:45781 after 21 ms (0 ms spent in bootstraps)
22/07/10 23:03:31 INFO DiskBlockManager: Created local directory at /tmp/spark-4fecd97d-3956-4a79-b39d-6d9a15e577ad/executor-ec4d9371-c65d-47df-a942-2dc8a54c3369/blockmgr-dc95256e-1b82-43a3-a698-1d7d7e6f53d9
22/07/10 23:03:31 INFO MemoryStore: MemoryStore started with capacity 366.3 MiB
22/07/10 23:03:31 INFO WorkerWatcher: Connecting to worker spark://Worker@192.168.0.107:44129
22/07/10 23:03:31 INFO CoarseGrainedExecutorBackend: Connecting to driver: spark://CoarseGrainedScheduler@master:45781
22/07/10 23:03:31 INFO ResourceUtils: ==============================================================
22/07/10 23:03:31 INFO ResourceUtils: No custom resources configured for spark.executor.
22/07/10 23:03:31 INFO ResourceUtils: ==============================================================
22/07/10 23:03:31 INFO TransportClientFactory: Successfully created connection to /192.168.0.107:44129 after 18 ms (0 ms spent in bootstraps)
22/07/10 23:03:31 INFO CoarseGrainedExecutorBackend: Successfully registered with driver
22/07/10 23:03:31 INFO Executor: Starting executor ID 0 on host 192.168.0.107
22/07/10 23:03:32 INFO Utils: Successfully started service 'org.apache.spark.network.netty.NettyBlockTransferService' on port 40497.
22/07/10 23:03:32 INFO NettyBlockTransferService: Server created on 192.168.0.107:40497
22/07/10 23:03:32 INFO BlockManager: Using org.apache.spark.storage.RandomBlockReplicationPolicy for block replication policy
22/07/10 23:03:32 INFO BlockManagerMaster: Registering BlockManager BlockManagerId(0, 192.168.0.107, 40497, None)
22/07/10 23:03:32 INFO BlockManagerMaster: Registered BlockManager BlockManagerId(0, 192.168.0.107, 40497, None)
22/07/10 23:03:32 INFO BlockManager: Initialized BlockManager: BlockManagerId(0, 192.168.0.107, 40497, None)
22/07/10 23:03:32 INFO Executor: Starting executor with user classpath (userClassPathFirst = false): ''
22/07/10 23:03:32 INFO CoarseGrainedExecutorBackend: Got assigned task 0
22/07/10 23:03:32 INFO CoarseGrainedExecutorBackend: Got assigned task 1
22/07/10 23:03:32 INFO Executor: Running task 0.0 in stage 0.0 (TID 0)
22/07/10 23:03:32 INFO Executor: Running task 1.0 in stage 0.0 (TID 1)
22/07/10 23:03:33 INFO TorrentBroadcast: Started reading broadcast variable 1 with 1 pieces (estimated total size 4.0 MiB)
22/07/10 23:03:33 INFO TransportClientFactory: Successfully created connection to master/192.168.0.107:40695 after 6 ms (0 ms spent in bootstraps)
22/07/10 23:03:33 INFO MemoryStore: Block broadcast_1_piece0 stored as bytes in memory (estimated size 3.5 KiB, free 366.3 MiB)
22/07/10 23:03:33 INFO TorrentBroadcast: Reading broadcast variable 1 took 419 ms
22/07/10 23:03:33 INFO MemoryStore: Block broadcast_1 stored as values in memory (estimated size 6.2 KiB, free 366.3 MiB)
22/07/10 23:03:42 ERROR Executor: Exception in task 0.0 in stage 0.0 (TID 0)
java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2302)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1432)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2460)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2454)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:508)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:466)
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:85)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:52)
	at org.apache.spark.scheduler.Task.run(Task.scala:136)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:548)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1504)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:551)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
22/07/10 23:03:42 ERROR Executor: Exception in task 1.0 in stage 0.0 (TID 1)
java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2302)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1432)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2460)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2454)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:508)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:466)
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:85)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:52)
	at org.apache.spark.scheduler.Task.run(Task.scala:136)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:548)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1504)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:551)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
22/07/10 23:03:55 INFO CoarseGrainedExecutorBackend: Got assigned task 2
22/07/10 23:03:55 INFO Executor: Running task 1.1 in stage 0.0 (TID 2)
22/07/10 23:03:55 INFO CoarseGrainedExecutorBackend: Got assigned task 3
22/07/10 23:03:55 INFO Executor: Running task 0.1 in stage 0.0 (TID 3)
22/07/10 23:03:55 ERROR Executor: Exception in task 1.1 in stage 0.0 (TID 2)
java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2302)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1432)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2460)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2454)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:508)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:466)
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:85)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:52)
	at org.apache.spark.scheduler.Task.run(Task.scala:136)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:548)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1504)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:551)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
22/07/10 23:03:55 ERROR Executor: Exception in task 0.1 in stage 0.0 (TID 3)
java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2302)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1432)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2460)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2454)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:508)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:466)
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:85)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:52)
	at org.apache.spark.scheduler.Task.run(Task.scala:136)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:548)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1504)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:551)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
22/07/10 23:03:56 INFO CoarseGrainedExecutorBackend: Got assigned task 4
22/07/10 23:03:56 INFO Executor: Running task 1.2 in stage 0.0 (TID 4)
22/07/10 23:03:56 INFO CoarseGrainedExecutorBackend: Got assigned task 5
22/07/10 23:03:56 INFO Executor: Running task 0.2 in stage 0.0 (TID 5)
22/07/10 23:03:56 ERROR Executor: Exception in task 0.2 in stage 0.0 (TID 5)
java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2302)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1432)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2460)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2454)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:508)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:466)
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:85)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:52)
	at org.apache.spark.scheduler.Task.run(Task.scala:136)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:548)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1504)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:551)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
22/07/10 23:03:56 ERROR Executor: Exception in task 1.2 in stage 0.0 (TID 4)
java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2302)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1432)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2460)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2454)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:508)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:466)
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:85)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:52)
	at org.apache.spark.scheduler.Task.run(Task.scala:136)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:548)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1504)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:551)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
22/07/10 23:03:56 INFO CoarseGrainedExecutorBackend: Got assigned task 6
22/07/10 23:03:56 INFO Executor: Running task 0.3 in stage 0.0 (TID 6)
22/07/10 23:03:56 ERROR Executor: Exception in task 0.3 in stage 0.0 (TID 6)
java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2302)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1432)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2460)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2454)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:508)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:466)
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:85)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:52)
	at org.apache.spark.scheduler.Task.run(Task.scala:136)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:548)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1504)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:551)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
22/07/10 23:03:56 INFO CoarseGrainedExecutorBackend: Got assigned task 7
22/07/10 23:03:56 INFO Executor: Running task 1.3 in stage 0.0 (TID 7)
22/07/10 23:03:56 ERROR Executor: Exception in task 1.3 in stage 0.0 (TID 7)
java.lang.ClassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.rdd.MapPartitionsRDD.f of type scala.Function3 in instance of org.apache.spark.rdd.MapPartitionsRDD
	at java.io.ObjectStreamClass$FieldReflector.setObjFieldValues(ObjectStreamClass.java:2302)
	at java.io.ObjectStreamClass.setObjFieldValues(ObjectStreamClass.java:1432)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2460)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2454)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2378)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2236)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1692)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:508)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:466)
	at org.apache.spark.serializer.JavaDeserializationStream.readObject(JavaSerializer.scala:87)
	at org.apache.spark.serializer.JavaSerializerInstance.deserialize(JavaSerializer.scala:129)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:85)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:52)
	at org.apache.spark.scheduler.Task.run(Task.scala:136)
	at org.apache.spark.executor.Executor$TaskRunner.$anonfun$run$3(Executor.scala:548)
	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1504)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:551)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
22/07/10 23:04:57 WARN Executor: Issue communicating with driver in heartbeater
org.apache.spark.rpc.RpcTimeoutException: Future timed out after [10000 milliseconds]. This timeout is controlled by spark.executor.heartbeatInterval
	at org.apache.spark.rpc.RpcTimeout.org$apache$spark$rpc$RpcTimeout$$createRpcTimeoutException(RpcTimeout.scala:47)
	at org.apache.spark.rpc.RpcTimeout$$anonfun$addMessageIfTimeout$1.applyOrElse(RpcTimeout.scala:62)
	at org.apache.spark.rpc.RpcTimeout$$anonfun$addMessageIfTimeout$1.applyOrElse(RpcTimeout.scala:58)
	at scala.runtime.AbstractPartialFunction.apply(AbstractPartialFunction.scala:35)
	at org.apache.spark.rpc.RpcTimeout.awaitResult(RpcTimeout.scala:76)
	at org.apache.spark.rpc.RpcEndpointRef.askSync(RpcEndpointRef.scala:103)
	at org.apache.spark.executor.Executor.reportHeartBeat(Executor.scala:1053)
	at org.apache.spark.executor.Executor.$anonfun$heartbeater$1(Executor.scala:238)
	at scala.runtime.java8.JFunction0$mcV$sp.apply(JFunction0$mcV$sp.scala:18)
	at org.apache.spark.util.Utils$.logUncaughtExceptions(Utils.scala:2066)
	at org.apache.spark.Heartbeater$$anon$1.run(Heartbeater.scala:46)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:750)
Caused by: java.util.concurrent.TimeoutException: Future timed out after [10000 milliseconds]
	at scala.concurrent.impl.Promise$DefaultPromise.tryAwait0(Promise.scala:248)
	at scala.concurrent.impl.Promise$DefaultPromise.result(Promise.scala:261)
	at org.apache.spark.util.ThreadUtils$.awaitResult(ThreadUtils.scala:293)
	at org.apache.spark.rpc.RpcTimeout.awaitResult(RpcTimeout.scala:75)
	... 13 more
22/07/10 23:04:58 WARN TransportResponseHandler: Ignoring response for RPC 4836522307174728748 from master/192.168.0.107:45781 (81 bytes) since it is not outstanding
22/07/10 23:05:35 ERROR CoarseGrainedExecutorBackend: Executor self-exiting due to : Driver master:45781 disassociated! Shutting down.
22/07/10 23:05:35 INFO CoarseGrainedExecutorBackend: Driver from master:45781 disconnected during shutdown
22/07/10 23:05:35 ERROR CoarseGrainedExecutorBackend: RECEIVED SIGNAL TERM
81 disconnected during shutdown
```

