### 1. mybatis简介
Hibernate采用全自动方式处理持久层与数据库的交互，而mybatis则提供一种半自动方式来解决持久层与数据库交互的问题，mybatis相比Hibernate让程序员拥有更大的自主权，这使得对于譬如针对高性能查询做SQL优化成为可能，然而，这也带来了更多的工作量。
>**Tips:**世间万事总是具有两面性的，鱼和熊掌往往不可兼得，关键看应用场景。


### 2. mybatis使用例子
#### 2.2 POJO
和Hibernate可以将注解直接写在POJO上，使POJO成为一个PO不同，而Hibernate的POJO就是个普通的POJO，如下
```java
public class Book {
	private Integer id;
	private String title;
	private String content;
}
```

#### 2.3 mapper / DAO
mybatis需要指定mapper文件，既可以使用xml来描述mapper项，也可以使用注解的interface。可以将注解的mapper interface看成DAO。例如
```java
public interface BookDao {
	@Select("select * from news_info where id = #{bookId}")
	public Book getBookById(int bookId);
}
```
mybatis会使用生成接口的子类方法来将注解的内容植入到`getBookById()`方法中。

#### 2.4 myBatis配置文件
以xml方式给出myBatis的配置项，如果在spring中使用MapperScannerConfigurer，则不需要在myBatis中给出mapper配置项。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<setting name="lazyLoadingEnabled" value="false" />
	</settings>
<!-- 	<mappers>
		<mapper class="dao.BookDao"></mapper>
	</mappers> -->
</configuration>
```

#### 2.5 MapperScannerConfigurer 和 Service
`MapperScannerConfigurer`可以很神奇地实现类似于spring的`<context:component-scan base-package="test" />`所实现的自动装配实例化过程。在spring中我们对于类似于2.3的DAO使用`@Repository`注解，并在使用DAO的地方添加`@Autowired`来实现自动装配实例化过程。然而对于2.3这样的mapper Interface我们并不能使用这样的方式来实现自动装配实例化，spring仅仅能针对POJO添加注解实现自动装配实例化。当然，这并不能阻止我们通过其他途径实现这样的巨大便捷性。`MapperScannerConfigurer`正是myBatis为我们提供的这样神奇工具。我们类似于如下方式在spring的配置xml进行申明
```xml
    <bean
        class="org.mybatis.spring.mapper.MapperScannerConfigurer"
        p:sqlSessionFactory-ref="sqlSessionFactory"
        p:basePackage="dao">
    </bean>
```
就可以完成自动装配实例化过程，对于扫描的dao package下的所有Interface，自动实现的动态代理子类实例就可以注入到对应的地方，譬如service层相应的类。譬如
```java
@Service
public class BookService {
	@Autowired
	BookDao bookDao;
	
	public Book getBook() {
		return bookDao.getBookById(1);
	}
	
	public void print() {
		System.out.println("Book title: " + title);
		System.out.println("Book content: " + content);
	}
}
```
这里的bookDao就会自动被注入spring容器中实例化的Mapper Interface DAO的动态代理子类。而中间的过程都由`MapperScannerConfigurer`帮助我们完成了。

#### 2.6 spring配置文件
为了配合myBatis使用，我们首先要配置一个数据源，这完全和JDBC的数据源配置相同。再申明一个`.SqlSessionFactoryBean`的Bean，这个Bean将引用我们的myBatis配置文件来配置myBatis数据库连接的一些项目。如果使用`MapperScannerConfigurer`，那么也需要做一些配置工作。一个配置的文件例子如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <context:component-scan base-package="test" />
    <context:component-scan base-package="service" />
    <context:property-placeholder location="classpath:jdbc.properties" />

    <bean
        id="dataSource"
        class="org.apache.commons.dbcp.BasicDataSource"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/sampledb"
        p:username="root"
        p:password="545765">
    </bean>

    <bean
        class="org.mybatis.spring.SqlSessionFactoryBean"
        id="sqlSessionFactory"
        p:dataSource-ref="dataSource"
        p:configLocation="classpath:myBatisConfig.xml">
    </bean>

    <bean
        class="org.mybatis.spring.mapper.MapperScannerConfigurer"
        p:sqlSessionFactory-ref="sqlSessionFactory"
        p:basePackage="dao">
    </bean>
</beans>
```
>**Tips:**我们也可以借助于`org.springframework.jdbc.datasource.DataSourceTransactionManager`非常简单地使用spring带给我的强大的事务管理功能。

#### 2.7 测试代码
使用Junit可以简单实现对myBatis测试的例子。
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="classpath:config.xml")
public class BookTest {
	
	@Autowired
	private BookService bookService;
	
	@Test
	public void getBookByIdTest() {
		bookService.getBook().print();
	}
}
```
运行结果如下：
```java
Book title: Gone with the wind
Book content: I will gone with the wind
```




