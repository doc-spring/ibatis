### 1. mybatis简介
Hibernate采用全自动方式处理持久层与数据库的交互，而mybatis则提供一种半自动方式来解决持久层与数据库交互的问题，mybatis相比Hibernate让程序员拥有更大的自主权，这使得对于譬如针对高性能查询做SQL优化成为可能，然而，这也带来了更多的工作量。
>**Tips:**世间万事总是具有两面性的，鱼和熊掌往往不可兼得，关键看应用场景。


### 2. mybatis使用例子
#### 2.2 POJO
和Hibernate可以将注解直接写在POJO上，使POJO成为一个PO不同，Hibernate的POJO就是个普通的POJO，以下为一个例子。
