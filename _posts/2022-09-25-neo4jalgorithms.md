---
title: Neo4j图形算法
date: 2022-09-25 10:34:00 +0800
categories: [科研]
tags: [知识图谱]
pin: true
author: 剑

toc: true
comments: true
typora-root-url: ../../HaoJGit.github.io
math: false
mermaid: true

---
# 图形算法 #

1. stream 以stream的形式返回算法的结果
2. stats  返回汇总统计消息的单个记录，但不写入Neo4j数据库
3. mutate 将算法的结果写入project，并返回汇总统计数据结果的单个记录。
4. write 讲算法的结果写入Neo4j数据库，并返回汇总统计数据的单个记录


## Centrality 中心性 ##

内存估计
``` 
CALL gds.eigenvector.write.estimate('myGraph', {
  writeProperty: 'centrality',
  maxIterations: 20
})
YIELD nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory  
```




###  Page Rank ##
PageRank 算法根据传入关系的数量和相应源节点的重要性来衡量图中每个节点的重要性。

``` 
stream/stats/mutate/ Write 

CALL gds.pageRank.stream(
  graphName: String,
  configuration: Map
)
YIELD
  nodeId: Integer,
  score: Float

CALL gds.pageRank.stats(
  graphName: String,
  configuration: Map
)
YIELD
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  centralityDistribution: Map,
  configuration: Map

CALL gds.pageRank.mutate(
  graphName: String,
  configuration: Map
)
YIELD
  nodePropertiesWritten: Integer,
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  mutateMillis: Integer,
  centralityDistribution: Map,
  configuration: Map

CALL gds.pageRank.write(
  graphName: String,
  configuration: Map
)
YIELD
  nodePropertiesWritten: Integer,
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  writeMillis: Integer,
  centralityDistribution: Map,
  configuration: Map

 ```

### AritceleRank ##
ArticleRank 是page Rank算法的一个变种，用了衡量节点之间的传递影响。

``` 
stream/stats/mutate/ Write

CALL gds.articleRank.stream(
  graphName: String,
  configuration: Map
)
YIELD
  nodeId: Integer,
  score: Float


CALL gds.articleRank.stats(
  graphName: String,
  configuration: Map
)
YIELD
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  centralityDistribution: Map,
  configuration: Map


CALL gds.articleRank.mutate(
  graphName: String,
  configuration: Map
)
YIELD
  nodePropertiesWritten: Integer,
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  mutateMillis: Integer,
  centralityDistribution: Map,
  configuration: Map

CALL gds.articleRank.write(
  graphName: String,
  configuration: Map
)
YIELD
  nodePropertiesWritten: Integer,
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  writeMillis: Integer,
  centralityDistribution: Map,
  configuration: Map

  ```
### Eigenvector  特征向量 ###

Eigenvector是一种测量节点传递影响的算法，score越高意味着连接到许多高score的节点。Page Rank算法是Eigenvector（t特征向量中心性）的变体，具有额外的跳转频率。



``` 
CALL gds.eigenvector.stream(
  graphName: String,
  configuration: Map
)
YIELD
  nodeId: Integer,
  score: Float

CALL gds.eigenvector.stats(
  graphName: String,
  configuration: Map
)
YIELD
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  centralityDistribution: Map,
  configuration: Map

CALL gds.eigenvector.mutate(
  graphName: String,
  configuration: Map
)
YIELD
  nodePropertiesWritten: Integer,
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  mutateMillis: Integer,
  centralityDistribution: Map,
  configuration: Map

CALL gds.eigenvector.write(
  graphName: String,
  configuration: Map
)
YIELD
  nodePropertiesWritten: Integer,
  ranIterations: Integer,
  didConverge: Boolean,
  preProcessingMillis: Integer,
  computeMillis: Integer,
  postProcessingMillis: Integer,
  writeMillis: Integer,
  centralityDistribution: Map,
  configuration: Map

```
### Betweenness  中间性 ###

Betweenness是一种检测节点对图形中信息流的影响程度的方法。通常用于查找作为从图形的一部分到另一部分的桥梁的节点。
该算法计算图形中所有节点对之间的未加权最短路径。每个节点都会根据通过该节点的最短路径数收到一个分数。更频繁地位于其他节点之间的最短路径上的节点将具有更高的中间性中心性分数。


### degree  ###
度中心性算法可用于查找图形中的常用节点。度中心性测量来自节点的传入或传出（或两者）关系的数量，具体取决于关系投影的方向。

### Closeness  ###

接近性中心性是一种检测能够通过图形非常有效地传播信息的节点的方法。
节点的接近度中心性测量其与所有其他节点的平均距离（反距离）。具有高接近度分数的节点到所有其他节点的距离最短。

对于每个节点 u，接近中心性算法根据计算所有节点对之间的最短路径来计算其到所有其他节点的距离之和。然后将得到的总和倒置，以确定该节点的接近度中心性分数。

节点 u 的原始接近度中心性使用以下公式计算：

raw closeness centrality(u) = 1 / sum(distance from u to all other nodes)

### Harmonic ###
谐波中心性（也称为值中心性）是closeness 中心性的一种变体，其发明是为了解决原始公式在处理未连接图形时存在的问题。与许多中心性算法一样，它起源于社交网络分析领域。
谐波中心性被提出作为closeness中心性的替代方案，因此具有类似的用例。

例如，如果我们试图确定在城市中放置新公共服务的位置，以便居民可以轻松访问它，我们可能会使用它。
如果我们试图在社交媒体上传播信息，我们可以使用算法来找到可以帮助我们实现目标的关键影响者。


### The Hyperlink-Induced Topic Search (HITS) ###
超链接诱导的主题搜索 （HITS） 是一种链接分析算法，
它根据两个数（一个中心数和一个authority 权数）对节点进行评级。The authority score estimates the importance of the node within the network. The hub score estimates the value of its relationships to other nodes. 

## Community detection 聚类检测##
社区检测算法用于评估节点组的聚类或分区方式，以及它们加强或分离的趋势。