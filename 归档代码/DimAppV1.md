```java
package com.yyz.realtime.app.dim;


import com.yyz.realtime.util.FlinkSourceUtil;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.runtime.state.hashmap.HashMapStateBackend;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
/**
 * 今天是2025-01-14
 * 有请 yyz
 * 分享一下今天的心情吧：
 *  无论走到哪里，都应该记住，过去都是假的，回忆是一条没有尽头的路，一切以往的春天都不复存在
 */
public class DimApp {
    public static void main(String[] args) {



        System.setProperty("HADOOP_USER_NAME","atguigu");
        Configuration conf = new Configuration();
        conf.setInteger("rest.port", 2000);
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment(conf);
        env.setParallelism(2);
        // 启用检查点机制，设置检查点间隔为3000毫秒
        env.enableCheckpointing(3000);
        env.setStateBackend(new HashMapStateBackend());
        env.getCheckpointConfig().setCheckpointStorage("hdfs://hadoop162:8020/gmall/DimApp");
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        env.getCheckpointConfig().setCheckpointTimeout(20 * 1000);
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);
        env.getCheckpointConfig().setExternalizedCheckpointCleanup(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);


        DataStreamSource<String>stream=env.addSource(FlinkSourceUtil.getKafkaSource("DimApp","ods_db"));
        stream.print();

        try {
            env.execute();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}




/*

### 1. **检查点机制（Checkpointing）**
   - **作用**：Flink 的检查点机制用于实现容错。它会定期保存应用程序的状态，以便在发生故障时能够从最近的一个检查点恢复，确保数据处理的精确一次（Exactly-Once）语义。
   - **关键配置**：
     - `env.enableCheckpointing(interval)`：启用检查点并设置检查点间隔（单位为毫秒）。
     - `setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE)`：设置检查点模式为精确一次（Exactly-Once）或至少一次（At-Least-Once）。
     - `setCheckpointTimeout(timeout)`：设置检查点的超时时间，如果检查点未在指定时间内完成，则会被丢弃。
     - `setMaxConcurrentCheckpoints(n)`：设置最大并发检查点数量。
     - `setMinPauseBetweenCheckpoints(pause)`：设置两个检查点之间的最小暂停时间，避免检查点过于频繁。

---

### 2. **状态后端（State Backend）**
   - **作用**：状态后端决定了 Flink 如何存储和管理应用程序的状态（例如算子状态、键控状态等）。
   - **常见状态后端**：
     - **HashMapStateBackend**：将状态存储在内存中，适合状态较小的作业。
     - **RocksDBStateBackend**：将状态存储在磁盘上（通过 RocksDB），适合状态较大的作业。
   - **配置方法**：
     ```java
     env.setStateBackend(new HashMapStateBackend());
     ```

---

### 3. **检查点存储（Checkpoint Storage）**
   - **作用**：检查点存储定义了检查点的持久化位置。Flink 支持将检查点存储在文件系统（如 HDFS、S3）或内存中。
   - **配置方法**：
     ```java
     env.getCheckpointConfig().setCheckpointStorage("hdfs://hadoop162:8020/gma11/DimApp");
     ```
   - **常见存储类型**：
     - **HDFS**：适合大规模分布式环境。
     - **本地文件系统**：适合测试或小规模环境。
     - **S3**：适合云环境。

---

### 4. **外部化检查点（Externalized Checkpoint）**
   - **作用**：外部化检查点允许在作业取消时保留检查点，以便后续可以从这些检查点恢复作业。
   - **配置方法**：
     ```java
     env.getCheckpointConfig().setExternalizedCheckpointCleanup(
         CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION
     );
     ```
   - **选项**：
     - `RETAIN_ON_CANCELLATION`：作业取消时保留检查点。
     - `DELETE_ON_CANCELLATION`：作业取消时删除检查点。

---

### 5. **并行度（Parallelism）**
   - **作用**：并行度决定了 Flink 作业中每个算子的并行任务数。合理的并行度可以提高作业的性能。
   - **配置方法**：
     ```java
     env.setParallelism(1); // 设置全局并行度为1
     ```
   - **注意**：
     - 并行度可以根据算子的需求单独设置。
     - 并行度过高可能导致资源竞争，过低可能导致资源利用率不足。

---

### 6. **Flink 的容错机制**
   - **核心思想**：通过检查点机制实现状态的一致性。
   - **恢复流程**：
     1. 从最近的检查点恢复状态。
     2. 重新处理从检查点到故障点的数据。
   - **精确一次语义（Exactly-Once）**：确保每条数据只被处理一次，即使在发生故障时也是如此。

---

### 7. **Flink 的运行时架构**
   - **JobManager**：负责作业调度和检查点协调。
   - **TaskManager**：负责执行具体的任务。
   - **Slot**：TaskManager 中的资源单元，每个 Slot 可以运行一个任务。

---

### 8. **Flink 的应用场景**
   - **实时数据处理**：如实时监控、实时报警。
   - **流批一体**：Flink 支持流处理和批处理，可以用同一套 API 处理两种数据。
   - **事件驱动应用**：如实时推荐、实时风控。

---

### 9. **Flink 的编程模型**
   - **DataStream API**：用于流处理。
   - **DataSet API**：用于批处理（已逐渐被 Table API 和 SQL 取代）。
   - **Table API & SQL**：用于声明式数据处理。

---

### 10. **Flink 的生态系统**
   - **连接器（Connectors）**：支持 Kafka、HDFS、JDBC 等数据源和目的地。
   - **状态管理**：支持键控状态（Keyed State）和算子状态（Operator State）。
   - **窗口操作**：支持滚动窗口、滑动窗口、会话窗口等。

---

### 11. **常见问题与调试**
   - **检查点失败**：可能是由于状态过大、网络延迟或资源不足。
   - **背压（Backpressure）**：下游处理速度跟不上上游生产速度，可以通过增加并行度或优化代码解决。
   - **状态一致性**：确保状态后端和检查点配置正确。

---


 */

```