# Liferay 6.2 Dynamic Query API 示例

## 介绍

Liferay 提供了几种方法定义复杂的查询用来检索数据库中的数据。

通常情况下，在每个 service Entity 中，通过定义一些 `finder` 方法，可以便捷地满足基本的数据查询操作。

但是，有时候我们可能会遇到以下几种finder查询并不能满足的情况：

- 过于复杂的查询，例如子查询
- 需要实现一些聚合操作，像min、max、avg等
- 想得到复合对象或元组而不是映射的对象类型
- 查询优化
- 复杂的数据访问，像报表等

要实现这个目的，就需要通过Liferay提供的Hibernate的Dynamic Query API实现。

在本文中，我们将演示如何构建不同类型的 Dynamic Query 并执行它们。

# Dynamic Query基本语法

在Liferay中构建一个Dynamic Query基本语法的代码如下：
```
//构建动态查询，相当于select * from Entity_Name
DynamicQuery dynamicQuery = DynamicQueryFactoryUtil.forClass(Entity_Name.class);
//DynamicQueryFactoryUtil.forClass(Entity_Name.class,PortalClassLoaderUtil.getClassLoader());
//设置查询列
dynamicQuery.setProjection(Projection projection);
//设置查询条件
dynamicQuery.add(Criterion criterion);
//设置排序规则
dynamicQuery.addOrder(Order order);
//设置返回结果集的范围
dynamicQuery.setLimit(int start, int end);
//执行动态查询，得到结果集
Entity_NameLocalServiceUtil.dynamicQuery(dynamicQuery);
```

其中，

**Entity_Name**：实体名称，就是service.xml中制定的Entity名称。

**DynamicQuery** 也可以通过 DynamicQuery forClass(Class<?> clazz, ClassLoader classLoader) 来初始化。

## Dynamic Query应用示例
### 1、select * from organization_;
```
DynamicQuery dynamicQuery = DynamicQueryFactoryUtil.forClass(Organization.class);
List<Organization> Organizations = OrganizationLocalServiceUtil.dynamicQuery(dynamicQuery);
```
### 2、select * from organization_ where parentOrganizationId=0; 
```
DynamicQuery dynamicQuery = DynamicQueryFactoryUtil.forClass(Organization.class);
dynamicQuery.add(PropertyFactoryUtil.forName("parentOrganizationId").eq(0L));
List<Organization> Organizations = OrganizationLocalServiceUtil.dynamicQuery(dynamicQuery);
```
### 3、like、>、>=、<、<=、between ... and ...
```
// select * from organization_ where name like '组织机构%';
dynamicQuery.add(PropertyFactoryUtil.forName("parentOrganizationId").like("组织机构%"));
// select * from organization_ where organizationId >21212;
dynamicQuery.add(PropertyFactoryUtil.forName("organizationId").gt(21212L));
// select * from organization_ where organizationId >=21212;
dynamicQuery.add(PropertyFactoryUtil.forName("organizationId").ge(21212L));
// select * from organization_ where organizationId <21224;
dynamicQuery.add(PropertyFactoryUtil.forName("organizationId").lt(21224L));
// select * from organization_ where organizationId <=21224;
dynamicQuery.add(PropertyFactoryUtil.forName("organizationId").le(21224L));
// select * from organization_ where organizationId between 21212 and 21224;
dynamicQuery.add(PropertyFactoryUtil.forName("organizationId").between(21212L, 21224L));
```
### 4、and / or
```
// select * from organization_ where organizationId >= 21212 and organizationId <=21224;
// 第1种方法（不适用于or）
dynamicQuery.add(PropertyFactoryUtil.forName("organizationId").ge(21212L));
dynamicQuery.add(PropertyFactoryUtil.forName("organizationId").le(21224L));
// 第2种方法（适用于or，使用RestrictionsFactoryUtil.or）
Criterion criterion = null;
criterion = RestrictionsFactoryUtil.ge("organizationId", 21212L);
criterion = RestrictionsFactoryUtil.and(criterion, RestrictionsFactoryUtil.le("organizationId", 21224L));
dynamicQuery.add(criterion);
// 第3种方法（适用于or，使用RestrictionsFactoryUtil.disjunction()）
Junction junction = RestrictionsFactoryUtil.conjunction();
junction.add(PropertyFactoryUtil.forName("organizationId").ge(21212L));
junction.add(PropertyFactoryUtil.forName("organizationId").le(21224L));
dynamicQuery.add(junction);
```
### 5、order by
```
// select * from organization_ order by organizationId asc;
dynamicQuery.addOrder(OrderFactoryUtil.asc("organizationId"));
// select * from organization_ order by organizationId desc;
dynamicQuery.addOrder(OrderFactoryUtil.desc("organizationId"));
```
### 6、子查询
```
// select * from organization_ where parentOrganizationId=(select organizationId from organization_ where name='组织机构1');
DynamicQuery subDynamicQuery = DynamicQueryFactoryUtil.forClass(Organization.class);
subDynamicQuery.setProjection(ProjectionFactoryUtil.property("organizationId"));
subDynamicQuery.add(PropertyFactoryUtil.forName("name").eq("组织机构1"));
dynamicQuery.add(PropertyFactoryUtil.forName("parentOrganizationId").in(subDynamicQuery));
```
### 7、自定义列
```
// select name from organization_;
dynamicQuery.setProjection(ProjectionFactoryUtil.property("name"));
List<Object> names = OrganizationLocalServiceUtil.dynamicQuery(dynamicQuery);
for(Object name: names){
    System.out.println(name);
}
// select organizationId,name from organization_;
ProjectionList projectionList = ProjectionFactoryUtil.projectionList();
projectionList.add(ProjectionFactoryUtil.property("organizationId"));
projectionList.add(ProjectionFactoryUtil.property("name"));
dynamicQuery.setProjection(projectionList);
List<Object[]> organizations = OrganizationLocalServiceUtil.dynamicQuery(dynamicQuery);
for(Object[] organization: organizations){
    System.out.println(organization[0]+":"+organization[1]);
}
```
### 8、distinct
```
// select distinct name from organization_;
Projection projection = ProjectionFactoryUtil.distinct(ProjectionFactoryUtil.property("name"));
dynamicQuery.setProjection(projection);
```
### 9、group by
```
// select type_,count(type_) from organization_ group by type_;
ProjectionList projectionList = ProjectionFactoryUtil.projectionList();
projectionList.add(ProjectionFactoryUtil.property("type"));
projectionList.add(ProjectionFactoryUtil.count("name"));
projectionList.add(ProjectionFactoryUtil.groupProperty("type"));
dynamicQuery.setProjection(projectionList);
List<Object[]> organizations = OrganizationLocalServiceUtil.dynamicQuery(dynamicQuery);
for(Object[] organization: organizations){
    System.out.println(organization[0]+":"+organization[1]);
}
```
此外，max聚合函数调用方法如下：
```
max:ProjectionFactoryUtil.max(String propertyName)
```
其他聚合函数min、avg等可参考递推。

### 10、分页
```
// 取第1条到第10条记录
dynamicQuery.setLimit(0,10);
```
### 11、复合主键
如果实体是符合主键，我们要通过复合主键中的属性列进行查询的话，则需要在列名前面加上"primaryKey."，如下：
```
dynamicQuery.add(PropertyFactoryUtil.forName("primaryKey.organizationId").gt(21212L));
```
## 总结

以上只是一些基本的示例，能够解决我们在日常开发中遇到的大部分问题，此外 Dynamic Query API 也提供了一些更高级的扩展方法（eqAll、geAll 等），这些大家就一起探索吧，以后用到再更新。

通过以上示例，我们可以看到 Liferay 提供的 Dynamic Query API，其实就是通过一组 java 方法来组成 SQL 语句，执行并获得结果。可能有些朋友会觉得这种方法太过于繁琐，还不如直接写 SQL 来得方便直接。但是站在平台数据库兼容性的角度考虑，我们就会发现这种方式非常合适。因为 liferay 支持 mysql、oracle、db2 等多种数据库，如果直接写 SQL 的话，很可能碰到其他数据库的语法不支持的情况发生，像 oracle 中的递归查询 mysql 就不支持等。使用 Dynamic Query API 的话，我们就可以使用一套统一的语法来构建 SQL 语句，而不需要考虑底层数据库的差异，这样整个平台的移植性和兼容性就显著提高了很多。
