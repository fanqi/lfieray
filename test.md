# Liferay 6.2 中 Dynamic Query API 示例

## 介绍
Liferay提供了几种方法定义复杂的查询用来检索数据库中的数据。

通常情况下，在每个service Entity中，通过定义一些'finder'方法，可以便捷地满足基本的数据查询操作。

但是，有时候我们可能会遇到以下几种finder查询并不能满足的情况：

过于复杂的查询，例如子查询
需要实现一些聚合操作，像min、max、avg等
想得到复合对象或元组而不是映射的对象类型
查询优化
复杂的数据访问，像报表等
要实现这个目的，就需要通过Liferay提供的Hibernate的Dynamic Query API实现。

在本文中，我们将演示如何构建不同类型的Dynamic Query并执行它们。
