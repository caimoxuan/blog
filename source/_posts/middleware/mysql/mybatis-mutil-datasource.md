---
title: mybatis多数据源配置，动态切换和注意事项
date: 2019-03-03 00:40:25
thumbnail: 
categories:
    - 中间件
tags:
    - mysql
---


## springboot + mybatis 多数据源配置

&emsp;&emsp;在项目中操作数据的时候，可能的情况是需要再同一个项目中同时操作多个数据库，一般情况下的项目都是连接一个mysql就可以了，但是有的时候是需要操作多个然后汇总数据的；

<!-- more -->

### 方案一 使用sharding-sphere

&emsp;&emsp;这是一个开源的分库分表插件，我们可以使用它的自带分库功能来完成我们的数据源配置的功能，他的原理是使用配置文件配置数据源 + 表名 + 分库条件， 如果多个数据源中的表不相同的话，目前来说默认回抛出一个检验异常，我们可以关闭它，详见官方文档：[https://shardingsphere.apache.org/document/current/cn/overview/](sharding-sphere官网)
有分库分表需求的时候个人认为这个比mycat好用不要太多，并且性能也可扩展，但是今天介绍的是一种spring-jdbc自带的方式来手动切换数据源；

### mybatis多数据源 + AbstractRoutingDataSource 动态切换

首先我们来配置多数据源

``` java

@Configuration
// 这里声明dao扫描路径
@MapperScan(basePackages="com.xxx.xxx.dao", sqlSessionFactoryRef="sessionFactory")
public class MybatisConfig {

		// 下面是2个数据源的数据库的相关配置
		/**
		* datasource:
			driver-class-name: com.mysql.cj.jdbc.Driver
			data:
				url: xxxx
				username: xx
				password: xx
			user:
				url: xxx
				username: xxx
				password: xxx
		*/
    @Autowired
    private DataSourceProperties dataSourceProperties;

    @Autowired
    private UserSourceProperties userSourceProperties;

		// 数据库驱动 也是上面的配置
    @Value("${datasource.driver-class-name}")
    private String driverClassName;


    /**
     * 请求的数据数据源1 这里使用的是hikar数据库连接池 可切换Druid等其他
		 * 数据源一定要起好名字，不然不好区分
     * @return
     */
    @Bean(name="data")
    public DataSource dataSourceData() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setJdbcUrl(dataSourceProperties.getUrl());
        dataSource.setUsername(dataSourceProperties.getUsername());
        dataSource.setPassword(dataSourceProperties.getPassword());
        return dataSource;
    }

    /**
     * 用户的数据源2
     * @return
     */
    @Bean(name="user")
    public DataSource dataSourceUser() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setJdbcUrl(userSourceProperties.getUrl());
        dataSource.setUsername(userSourceProperties.getUsername());
        dataSource.setPassword(userSourceProperties.getPassword());
        return dataSource;
    }


		/**
		*  这里配置的是总的动态数据源， 将多个数据源汇总起来形成一个动态数据源
		*/
    @Bean(name="dynamicDataSource")
    @Primary
    public DynamicDataSource DataSource(@Qualifier("data") DataSource dataSourceData,
                                        @Qualifier("user") DataSource dataSourceUser){
        Map<Object, Object> targetDataSource = new HashMap<>(2);
        targetDataSource.put(DataBaseType.DATA, dataSourceData);
        targetDataSource.put(DataBaseType.USER, dataSourceUser);
        DynamicDataSource dataSource = new DynamicDataSource();
        dataSource.setTargetDataSources(targetDataSource);
				// 设置默认的数据源
        dataSource.setDefaultTargetDataSource(dataSourceData);

        return dataSource;
    }


		/**
		 * 最后是sql会话配置
		 */
    @Bean(name="sessionFactory")
    public SqlSessionFactory sessionFactory(@Qualifier("dynamicDataSource")DynamicDataSource dataSource) throws Exception{
        SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
				// sql的xml文件的扫描路径
        sessionFactoryBean.setMapperLocations(resolver.getResources("classpath*:mapper/*.xml"));
        // 别名设置，这里配置了 打了包之后可能回不生效 可以添加 VFS.addImplClass(SpringBootVFS.class) 解决，原因是扫描包的方式有改变；在springboot的环境下，Mybatis的DefaultVFS这个扫包会出现问题，所以只能修改VFS；本来这些都可以在springboot的配置文件中配置
				sessionFactoryBean.setTypeAliasesPackage("com.xxx.xxx.model");

				//mybatis配置文件路径 可以不要
        sessionFactoryBean.setConfigLocation(new ClassPathResource("mybatis-config.xml"));
        return sessionFactoryBean.getObject();
    }
```

接下来我们需要使用spring-jdbc中的AbstractRoutingDataSource，这个类中有获取当前数据源的抽象方法，是由我们来实现的；
``` java

public class DynamicDataSource extends AbstractRoutingDataSource {

		// 原理就是在执行sql之前会来调用这个方法来获取当前的数据源类型去配置中获取相应的数据源来执行sql
    @Override
    protected Object determineCurrentLookupKey() {
        return DatabaseContextHolder.getDatabaseType();
    }
}

```

上面有一个DatabaseContextHolder类，这个类就是一个ThreadLocal，保存当前线程的数据源类型；
``` java
public class DatabaseContextHolder {

    private static final ThreadLocal<DataBaseType> contextHolder = new ThreadLocal<>();

    public static void setDatabaseType(DataBaseType type){
        contextHolder.set(type);
    }

    public static DataBaseType getDatabaseType(){
        return contextHolder.get();
    }
		// 最好有remove， 但是没有问题也不大；
}
```

最后DataBaseType就是一个枚举数据源的类型, 标记各个数据源;
``` java
@Getter
public enum DataBaseType {
    DATA("DATA", "1"),
    USER("USER", "2");

    DataBaseType(String name, String value){
        this.name = name;
        this.value = value;
    }

    private String name;

    private String value;
}
```

完成上述的步骤之后，我们就可以开始使用了；需要访问那个数据源，dao层是肯定知道的，懒的话，可以在dao上自定义注解然后使用切面的方式设置数据源，这里提供简单使用的方法：
``` java
//在进行数据库操作之前，就手动设置：
DatabaseContextHolder.setDatabaseType(DataBaseType.USER);
//这个操作每个sql调用的时候都需要 ，比如一个service中的一个方法使用了多个dao，每个dao调用的数据源不同，需要在每个dao的方法之前设置数据源；
```
