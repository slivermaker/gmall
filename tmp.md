### 数据仓库分层与汇总

- **DWS（Data Warehouse Summary）层**  
  - 进行轻度汇总，处理小窗口级别的数据汇总。  
  - 处理海量资源，例如“666jz”。  

- **数据存储**  
  - 存储介质应支持SQL查询，常用技术包括：  
    - MySQL：适合结构化数据存储。  
    - Hive：适合大数据处理。  
    - HBase + Phoenix：适合实时查询。  
    - ClickHouse：适合实时分析。  
    - Doris：适合大规模数据分析。  

- **实时查询**  
  - 需要支持大流量数据的实时查询，推荐使用ClickHouse和Doris。  

- **ADS（Application Data Store）层**  
  - 根据具体时间范围对数据进行进一步汇总。  
  - 使用SQL语句进行数据汇总和查询，例如：  
    ```sql
    SELECT ..., SUM(...) FROM t WHERE time >= ... AND time <= ...
    ```

### 关键点总结  
- 分层处理：DWS层轻度汇总，ADS层进一步汇总。  
- 存储选择：支持SQL的数据库系统，如MySQL、Hive、HBase+Phoenix、ClickHouse、Doris。  
- 实时查询：确保支持大流量实时查询，推荐ClickHouse和Doris。  
- SQL应用：在ADS层使用SQL进行数据汇总和查询。  

这些内容可以帮助理解数据仓库的分层结构、存储选择及实时查询的实现方式。