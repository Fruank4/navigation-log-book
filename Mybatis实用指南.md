## Mybatis实用指南

**MyBatis-Spring 会帮助将 MyBatis 无缝地整合到至Spring 中，极大程度地简化在Spring中操作数据的过程：**

- 允许Mybatis参与到Spring的事务管理中

- 使我们直接通过java代码操作数据库，无需手动创建Dao、Mapper以及SqlSession。

- 将 Mybatis 的异常转换为 Spring 的 `DataAccessException`

**对于Java8，最新版本的mybatis-spring是2.1.0**

```java
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```



**1、创建一个SqlSessionFactory，它负责创建到数据库的连接会话**

```java
@Bean(name = "sqlSessionFactory")
public SqlSessionFactory sqlSessionFactory(DataSource dataSource,
@Value("classpath:mybatis/mybatis-config.xml") Resource configLocation) throws Exception {
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource);
    sqlSessionFactoryBean.setConfigLocation(configLocation);
    return sqlSessionFactoryBean.getObject();
}
```

在基础的 MyBatis 用法中，是通过 `SqlSessionFactoryBuilder` 来创建 `SqlSessionFactory` 的。而在mybatis-spring中，一般使用SqlSessionFactoryBean来创建SqlSessionFactory，同时还可以把数据源DataSource和配置文件mybatis-config.xml与其绑定。

**2、创建数据源DataSource，这里通常会进行数据源的访问配置**

```java
@Configuration
public class DataSourceConfig {
    
  	@Bean(name = "dataSource", initMethod = "init", destroyMethod = "destroy")
    public DataSource dataSource() {
        return TDataSourceBuilder
                .create()
                .appName("IDLE_MESSAGE_CONFIG_APP")
                .dynamicRule(true)
                .sharding(false)
                .build();
    }
}
```

**3、创建mybatis-config.xml，用于控制Java类与数据库之间如何交互**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- mybatis的配置文件 -->
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
		<!--类型别名用于为Java类型设置一个缩写名称，旨在mapper.xml文件中，降低冗余的全限定类名书写-->
    <typeAliases>
	    <typeAlias alias="Author" type="com.laone.mybatis.mysql.domain.Author"/>
	    <typeAlias alias="Scene" type="com.laone.mybatis.mysql.domain.Scene"/>
      <package name="com.laone.mybatis.domain"/>
    </typeAliases>

  	<!--类型处理器用于完成Java类型T与数据库存储之间的转换，通过extend BaseTypeHandler<T>-->
    <typeHandlers>
        <typeHandler handler="com.laone.mybatis.mysql.handler.StringListTypeHandler"/>
				<typeHandler handler="com.laone.mybatis.mysql.handler.StringMapTypeHandler"/>
      	<package name="com.laone.mybatis.handler"/>
    </typeHandlers>

  	<!--映射文件用于定义各种SQL语句，以完成Java类型到数据库表的映射-->
    <mappers>
        <mapper resource="com/laone/mybatis/mapper/author-mapper.xml"/>
        <mapper resource="com/laone/mybatis/mapper/Scene-mapper.xml"/>
        <package name="com.laone.mybatis.mapper"/>
    </mappers>
</configuration>
```

**4、创建一个SqlSessiontemplate，用于完成具体的数据库连接**

```java
@Bean(name = "sqlSessionTemplate")
public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
    return new SqlSessionTemplate(sqlSessionFactory);
}
```

SqlSessionTemplate 是 MyBatis-Spring 的核心，替代默认的DefaultSqlSession，因为：

- 线程安全，并且可以被多个映射器共享实用
- 可以保证SqlSession与当前Spring的事务相关

**5、注册映射器Mapper**

通过Mybatis-Spring对映射器的管理，你将无需手动创建Mapper，更无需再操作SqlSession（创建，打开，关闭）。Mybatis-Spring为我们创建线程安全的映射器，这样我们就可以直接注入到其他Bean中。

Mybatis-Spring最强大的功能就是自动扫描映射器，将指定路径中的映射器统统注册到Spring容器当中。

而所有映射器都将使用这里的sqlSessionFactory以及sqlSessionTemplate，即共享相同的数据源和配置。

```java
@Configuration
@MapperScan(basePackages = "com.alibaba.idlefish.heracles.ext.tddl.mapper",
        sqlSessionFactoryRef = "sqlSessionFactory",
        sqlSessionTemplateRef = "sqlSessionTemplate")
public class MybatisConfig {
}
```

如果Mapper没有使用@Component注解显式指定名称，将会使用映射器的首字母小写非全限定类名作为名称。

**6、定义Mapper**

```java
@Mapper
public interface Author {
	Author queryById(String id);
  
  List<Author> pageQueryByColumn(@Param("offset") int offset, @Param("limit") int limit, 	 @Param("column") String column, @Param("value") String value);
  
  int insert(Author author);
  
  int update(Author author);
  
  int delete(String id);
 
}
```

**7、定义Mapper.xml文件**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.laone.mybatis.mysql.domain.Author">
  <!--定义Java Class 与 数据库 查询结果 的映射，供整个Mapper文件重复利用-->
	<resultMap id="authorResultMap", type="Author">
	  <id property="id", column="id"/>
	  <result property="authorId", column="author_id"/>
	  <result property="writeType", column="wirte_type"/>
	  <result property="config", column="config"/>
	  <result property="status", column="status"/>
	  <result property="attributes", column="attributes"/>
	  <result property="countryId", column="country_id"/>
	  <result property="gmtCreate", column="gmt_create"/>
	    <result property="gmtModified", column="gmt_modified"/>
	</resultMap>
	
  <!--重复的字段基本永远无法避免，Sql模板带你简化带你飞-->
	<sql id="baseColumn">
	    id,author_id, wirte_type, config, status, attributes, country_id, 
		  gmt_create, gmt_modified
	</sql>


	<select id="queryById", parameterType="String", resultMap="authorResultMap">
	  SELECT
	  <include refid="baseColumn">
    FROM author_table
    WHERE author_id = #{authorId}
	</select>
    
  <select id="pageQueryByStatus" resultMap="authorResultMap">
      SELECT
      <include refid="baseColumn"/>
      FROM author_table
    <!--#{}会在编译后set成对应类型的变量，${}则原封不动直接替换成变量，#方式🉑️防止Sql注入。-->
    	WHERE ${column}=#{value}
      ORDER BY id DESC
      LIMIT #{offset}, #{limit}
  </select>
    
  <!--无需传递值，自动为主键列生成编码-->  
	<insert id="insertAuthor" paramterType="Author" useGeneratedKeys="true" keyProperty="id">
  	INSERT INTO author_table 
    (<include refid="baseColumn">) 
    VALUES 
    (null, #{authorId}, #{writeType}, #{config}, #{status}, #{attributes}, 
    #{countryId},NOW(), NOW())
	</insert>

    
	<update id="updateAuthor" paramterType="Author">
	  UPDATE author_table 
	  SET
	    config = #{config},
	    status = #{status},
	    country_id = #{countryId}
	  WHERE id = #{id}
	</update>
	
    
	<delete id="deleteAuthor" parameterType="String">
	  DELETE 
	  FROM	author_table
	  WHERE id = #{id}
	</delete>
```

**参考**

mybatis文档

https://mybatis.org/mybatis-3/zh/getting-started.html

mybatis-spring文档

https://mybatis.org/spring/zh/getting-started.html



















