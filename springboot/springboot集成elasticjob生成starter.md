[TOC]

## spring-boot-starter-elasticjob简介

elasticjob是当当开源的一个分布式任务调度平台，通过和springboot2.2的整合成一个starter，无需xml繁琐的配置，即可快速启用elasticjob，相关代码已上传到github。

## 一、starter项目地址

[https://github.com/foolishboy66/spring-boot-starter-elasticjob.git](https://github.com/foolishboy66/spring-boot-starter-elasticjob.git)



## 二、使用步骤

### 1、引入maven依赖，如有需要，可以自行打包上传至私服

```java
<dependency>
    <groupId>com.github.foolishboy</groupId>
    <artifactId>spring-boot-starter-elasticjob</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

### 2、编写job代码

```java
@Slf4j
@ElasticJobScheduled(jobName = "${simple.job.demo.job.jobName}", cron = "${simple.job.demo.job.cron}", shardingTotalCount = "${simple.job.demo.job.shardingTotalCount}")
public class SimpleJobDemo implements SimpleJob {

    @Override
    public void execute(ShardingContext shardingContext) {

        log.info("当前分片项 shardingItem={}, jobName={}", shardingContext.getShardingItem(), shardingContext.getJobName());
        // do something...
    }
}
```

### 3、添加spring配置文件配置项，同时支持yaml和properties文件

- [x] application.properties

  ```properties
  # elasticjob注册中心地址
  spring.elastic-job.reg-center.server-lists=localhost:2181
  # elasticjob注册命名空间
  spring.elastic-job.reg-center.namespace=elastic-job-lite-springboot
  # 作业名称
  simple.job.demo.job.jobName=SimpleJobDemo
  # 任务执行的cron表达式
  simple.job.demo.job.cron=0/5 * * * * ?
  # 任务分片总数
  simple.job.demo.job.shardingTotalCount=128
  ```

- [x] application.yml

  ```yaml
  spring:
    elastic-job:
      reg-center:
        # elasticjob注册中心地址
        server-lists: localhost:2181
        # elasticjob注册命名空间
        namespace: elastic-job-lite-springboot
  simple:
    job:
      demo:
        job:
          # 作业名称
          jobName: SimpleJobDemoTest
          # 任务执行的cron表达式
          cron: 0/5 * * * * ?
          # 任务分片总数
          shardingTotalCount: 1
  ```



## 三、示例代码地址

[https://github.com/foolishboy66/spring-boot-starter-elasticjob-test.git](https://github.com/foolishboy66/spring-boot-starter-elasticjob-test.git)



