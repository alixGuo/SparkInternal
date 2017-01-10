## SparkInternal
### [Executor资源是怎么申请来的](https://github.com/alixGuo/SparkInternal/blob/master/Executor%E8%B5%84%E6%BA%90%E6%98%AF%E6%80%8E%E4%B9%88%E7%94%B3%E8%AF%B7%E6%9D%A5%E7%9A%84.md)
- 普通资源申请分析
- 动态资源分配（DynamicResourceAllocation）  

### [Spark 2.0 Optimization Point Analysis](https://github.com/alixGuo/SparkInternal/blob/master/spark%202.0%20Optimization%20Point%20Analysis%20.md)
- 分析Spark2.0版本涉及的一些优化点  

### [Spark On Yarn 任务提交流程分析](https://github.com/alixGuo/SparkInternal/blob/master/Spark%20On%20Yarn%20%E4%BB%BB%E5%8A%A1%E6%8F%90%E4%BA%A4%E6%B5%81%E7%A8%8B%E5%88%86%E6%9E%90.pdf)  
- 整体分析了下Spark on Yarn的任务提交流程，对yarn-client和yarn-cluster提交方式做了对比  

### [spark-issue1根据任务并发数调整上传HDFS文件副本](https://github.com/alixGuo/SparkInternal/blob/master/%5Bspark-issue1%5D%E6%A0%B9%E6%8D%AE%E4%BB%BB%E5%8A%A1%E5%B9%B6%E5%8F%91%E6%95%B0%E8%B0%83%E6%95%B4%E4%B8%8A%E4%BC%A0HDFS%E6%96%87%E4%BB%B6%E5%89%AF%E6%9C%AC%E6%95%B0.md)  
- 通过动态调整spark .stage临时文件的副本数，降低spark任务拉文件对hdfs的压力。  
