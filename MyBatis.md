###各种持久层框架的比较
- Hibernate

它具有很好的面向对象，提供了相应的映射，但是不是万能的，例如索引、存储过程、函数等，对数据库查询的性能有很大的帮助，可以优化sql 语句；当表的结构很复杂的时候，Hibernate 自动生成的sql 语句，也相应的更复杂，在一些大数据量、高并发的场景下，并不适合。

- Spring JDBC

严格的说，它并不能算是持久层框架，它只是使用模板对JDBC 封装，在它里面没有映射文件、对象查询语言，缓存的概念。

- Mybatis

我们是直接在配置文件中编写原生的sql语句，这给我们优化sql语句，提供了便利。由于使用原生的sql语句，它的数据库的可移植性就会降低。

###The configuration XML

配置文件是MyBatis的核心，包含了数据库连接对象信息、事务管理等配置。
    
      <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">//配置标签文件
    <configuration>
        <typeAliases >
		<!-- //设置别名，使用user 就代表com.example.test.javabean.User 该类 -->
		<typeAlias type="com.example.test.javabean.User" alias="user"/>
		</typeAliases>
    	<environments default="development">
    		<environment id="development">
    			<transactionManager type="JDBC">//指定Mybatis的事务类型
    			</transactionManager>
    			<dataSource type="POOLED">//配置数据源
    				<property name="driver" value="com.mysql.jdbc.Driver"></property>
    				<property name="url" value="jdbc:mysql://localhost:3306/mybatistest"></property>
    				<property name="username" value="root"></property>
    				<property name="password" value="123456"></property>
    			</dataSource>
    		</environment>
    	</environments>
		<mappers>
	    <!-- 设置映射路径，因为是在类路径（也就是src下），如果是在其他包下，必须去设置文件目录如:com/example/xxx.xml -->
		<mapper resource="UserMapper.xml"></mapper>
		</mappers>
    </configuration>
这里需要注意configuration 下的子标签需要按照一定的顺序，不是随便放的，注意一下。

这里配置Mybatis配置文件，还可以通过Java代码去进行配置，这里还是推荐使用xml。

####细讲Mybatis的配置文件

#####property

该属性可以用在配置数据库属性

这里演示的是通过属性文件动态配置：

    			<dataSource type="POOLED">
    				<property name="driver" value="${jdbc_driverClassName}"></property>
    				<property name="url" value="${}jdbc_url"></property>
    				<property name="username" value="${jdbc_username}"></property>
    				<property name="password" value="${jdbc_password}"></property>
    			</dataSource>


上面的属性会被属性文件中定义的属性值去替换。

在创建SqlSessionFactory 的时候，需要通过Resources类将属性文件读取

    	String resource = "mybatis-configure.xml";
    	InputStream in = Resources.getResourceAsStream(resource);
    	Properties pros = Resources.getResourceAsProperties("jdbc.properties");//属性文件是在src 下
    	SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in,pros);

#####settings

该标签下主要用来修改Mybatis 的一些设置。

常用的说明：

- cacheEnabled：对于Mapper 是否使用缓存。
- lazyLoadingEnabled：是否使用懒加载，可以在某些关系中使用fetchType  属性去取代该设置。
- aggressiveLazyLoading：如果设置为true，那么它将一次加载所有的懒加载属性的对象，否则，按照需要进行加载。
- autoMappingBehavior：指定MyBatis应如何自动映射列到字段或属性。  NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集；FULL 会自动映射任意复杂的结果集（无论是否嵌套）。
- defaultExecutorType：配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。
- defaultFetchSize：果集获取数量（fetchSize）设置一个提示值。此参数只可以在查询设置中被覆盖。
- returnInstanceForEmptyRow：当返回行的所有列都是空时，MyBatis默认返回null。 当开启这个设置时，MyBatis会返回一个空实例。
- logImpl：指定 MyBatis 所用日志的具体实现，未指定时将自动查找。

#####typeAliases

别名是通过一个简短的名称去代表一个Java类型。

单个指定一个别名，我们在上面已经演示，但是这样一个一个的注册，多的时候，有没有简单的方法。

通常我们设置别名，都是设置JavaBean 类，那么我们可以直接去注册一个package，将该包下的所有JavaBean 注册一个别名


    <package name="com.example.test.javabean"/>

这是默认每一个JavaBean 的别名是类名的首字母小写。

但是我们可以在JavaBean 通过Mybatis 的注解进行配置。
    
    @Alias("user")
    public class User {
    	private String userId;
    	private String userName;

注意Mybatis 内置提供对一些常用的Java 类型的别名，特别是对原始类型，具体使用对的时候，请自行查找。

#####typeHandlers

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。Mybatis类型处理器（使用的时候，自行查询）

如何自定义类型处理器，实现TypeHandler接口，或继承一个BaseTypeHandler接口。具体可以参看Mybatis 的实现类。

自定义类型处理器，需要配置到Mybatis 的配置文件中：


      <typeHandlers>
      <typeHandler handler="具体的类型处理器的全限类名"/>
      </typeHandlers>


需要注意如果你自定义某种类型处理器，那么会覆盖已存在的。

通过类型处理器的泛型，MyBatis 可以得知该类型处理器处理的 Java 类型，不过这种行为可以通过两种方法改变：
1. 在类型处理器的配置元素（typeHandler element）上增加一个 javaType 属性（比如：javaType="String"）
2. 在类型处理器的类上（TypeHandler class）增加一个 @MappedTypes 注解来指定与其关联的 Java 类型列表。 如果在 javaType 属性中也同时指定，则注解方式将被忽略。

可以通过两种方式来指定被关联的 JDBC 类型：
1. 在类型处理器的配置元素上增加一个 jdbcType 属性（比如：jdbcType="VARCHAR"）
2. 在类型处理器的类上（TypeHandler class）增加一个 @MappedJdbcTypes 注解来指定与其关联的 JDBC 类型列表。 如果在 jdbcType 属性中也同时指定，则注解方式将被忽略。

    <typeHandlers >
    <typeHandler javaType="" jdbcType=""></typeHandler>
    </typeHandlers>

#####objectFactory
MyBatis 每次创建**结果对象的新实例**时，它都会使用一个对象工厂（ObjectFactory）实例来完成。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。 如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现。

    public class ExampleObjectFactory extends DefaultObjectFactory {
    	private static Logger log = Logger.getLogger(ExampleObjectFactory.class);
    
    	/**
    	 * 根据默认的构造函数创建结果对象实例
    	 */
    	@Override
    	public <T> T create(Class<T> type) {
    		// TODO Auto-generated method stub
    		return super.create(type);
    	}
    
    	/**
    	 * 根据构造函数的参数去创建结果对象实例
    	 */
    	@Override
    	public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    		// TODO Auto-generated method stub
    		return super.create(type, constructorArgTypes, constructorArgs);
    	}
    
    	// 设置属性
    	@Override
    	public void setProperties(Properties properties) {
    		// TODO Auto-generated method stub
    		Iterator<Object> it = properties.keySet().iterator();
    		while (it.hasNext()) {
    			log.debug(properties.getProperty(it.next().toString()));
    		}
    		super.setProperties(properties);
    	}
    
    	@Override
    	protected Class<?> resolveInterface(Class<?> type) {
    		// TODO Auto-generated method stub
    		return super.resolveInterface(type);
    	}
    
    	@Override
    	public <T> boolean isCollection(Class<T> type) {
    		// TODO Auto-generated method stub
    		return super.isCollection(type);
    	}
    
    }


我们在配置文件做如下添加：

    <objectFactory type="com.example.test.test1.ExampleObjectFactory">
    <property name="name" value="helloworld"/>
    </objectFactory>

这里的属性会传给setProperties方法

这样对象工厂每次在实例化结果实例的时候，都会包含一个key 为name的属性，至于其他的自定义操作，可以按照需要去设置。

##### plugins

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。

MyBatis 允许使用插件来拦截的方法调用包括：

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
- ParameterHandler (getParameterObject, setParameters)
- ResultSetHandler (handleResultSets, handleOutputParameters)
- StatementHandler (prepare, parameterize, batch, update, query)

    	@Intercepts({ @Signature(type = Executor.class, method = "update", args = { MappedStatement.class, Object.class }) })
    	public class ExamplePlugin implements Interceptor {
    
    	public Object intercept(Invocation invocation) throws Throwable {
    		// TODO Auto-generated method stub
    		return invocation.proceed();
    	}
    
    	public Object plugin(Object target) {
    		// TODO Auto-generated method stub
    		return Plugin.wrap(target, this);
    	}
    
    	public void setProperties(Properties properties) {
    		// TODO Auto-generated method stub
    
    	}
    
    }

上面的插件将会拦截在 Executor 实例中所有的 “update” 方法调用

配置在Mybatis 的配置文件中：

    	<plugins>
    	<plugin interceptor="com.example.test.test1.ExamplePlugin">
    	<property name="NAME" value="LAOQIANG"></property>
    	</plugin>
    	</plugins>

#####environments

MyBatis 可以配置成适应多种环境，例如，开发、测试和生产环境需要有不同的配置。共享相同 Schema 的多个生产数据库。

尽管可以配置多个环境，每个 SqlSessionFactory 实例只能对应一个环境、数据库。

关于环境在之前的Mybatis 的配置文件中，已经展示，这里不再演示。

我们在Mybatis 配置文件中可以配置多个环境，可以在加载

		`SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in,"development");`

在build 传入相应配置文件配置的environment 的id。如果这里没有指定，它会根据xml 中配置的default 加载默认的environment。

######transactionManager

Mybatis 支持两种事务管理器[JDBC|MANAGED]

- JDBC – 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。
- MANAGED – 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。

			<transactionManager type="MANAGED">
			<property name="closeConnection" value="false"/>
			</transactionManager>

**如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器， 因为 Spring 模块会使用自带的管理器来覆盖前面的配置。** 

######dataSource

Mybatis 提供三种数据源：[UNPOOLED|POOLED|JNDI]

1. UNPOOLED :这种数据源的实现，在每一次请求，都会开启一个connection，结束之后关闭。就如我们在上面的例子中配置的一样，简单方便使用。
2. UNPOOLED :这种数据源的实现利用"池"的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这是一种使得并发 Web 应用快速响应请求的流行处理方式。

				<dataSource type="UNPOOLED">
				<property name="driver" value="${jdbc_driverClassName}"></property>
				<property name="url" value="${jdbc_url}"></property>
				<property name="username" value="${jdbc_username}"></property>
				<property name="password" value="${jdbc_password}"></property>
				<!-- 设置最大存活连接数 -->
				<property name="poolMaximumActiveConnections" value="" />
				<!-- 设置最大空闲连接数 -->
				<property name="poolMaximumIdleConnections" value="" />
			</dataSource>

其余属性设置，用到自行百度

3.JNDI:这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。

如果你想使用第三方的数据源(如：c3p0)：

导入相应的依赖:

    public class C3P0DataSourceFactory extends UnpooledDataSourceFactory {
    
      public C3P0DataSourceFactory() {
    this.dataSource = new ComboPooledDataSource();
      }
    }

然后在Mybatis 配置文件中dataSource 标签下的type属性引入该类。

#####映射器（mappers）

我们现在就要定义 SQL 映射语句了。但是首先我们需要告诉 MyBatis 到哪里去找到这些语句。
    
    <!-- 使用相对于类路径的资源引用 -->
      <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
    <!-- 使用映射器接口实现类的完全限定类名 -->
      <mapper class="org.mybatis.builder.AuthorMapper"/>

    <!-- 将包内的映射器接口实现全部注册为映射器 -->
    <mappers>
      <package name="org.mybatis.builder"/>
    </mappers>


###加载MyBatis的配置文件

配置的路径下，可以通过通用类Resources，通过流去读取：

    String resource = "org/mybatis/example/mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);


###SqlSessionFactory 

每一个Mybatis 应用都必须要有一个是SqlSessionFactory实例，**SqlSessionFactory 最好是处于application作用域中，通过单例去获取。**它是从SqlSessionFactoryBuilder去创建，这个SqlSessionFactory是从xml 配置文件中加载的一个实例。

####如何获取SqlSessionFactory

    String resource = "mybatis-configure.xml";
    InputStream in = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);

###SqlSession
 
真正在MyBatis中起到增删改查的操作，还是通过SqlSession ，它是通过SqlSessionFactory 来获取的。

			SqlSession sqlSession= sqlSessionFactory.openSession();

####SqlSession 作用域

每一个线程都应该有自己的SqlSession，每个线程之间不共享SqlSession，它是线程不安全的。它的作用域：方法或者一次请求中。
每次使用完SqlSession，都需要在close
    		

### 探索Mybatis中的Mapper

#### 细讲xml 的Mapper 映射文件

Mybatis 的Mapper映射文件有几个配置元素（按照它们应该被定义的顺序）：
- cache – 给定命名空间的缓存配置。
- cache-ref – 其他命名空间缓存配置的引用。
- resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
- sql – 可被其他语句引用的可重用语句块。
- insert – 映射插入语句
- update – 映射更新语句
- delete – 映射删除语句
- select – 映射查询语句

#####select

    	<select id="selectUserById" resultType="user">
    		select * from user where userid = #{id}
    	</select>

这个定义了一个id为selectUserById的statement，返回一个User 类型的结果集。

 #{id} 告诉 MyBatis 创建一个预处理语句参数，通过 JDBC，这样的一个参数在 SQL 中会由一个“?”来标识。

select 常用的属性介绍：

- id：该命名空间位移标识该statement 的。
- parameterType：参数类型，可以不设置，Mybatis 通过TypeHandler去推断。
- resultType：设置返回类型，如果是集合的话，类型应该设置为包含的元素类型，而不是集合类型。
- resultMap：外部 resultMap 的命名引用。提供了javabean 类的属性和sql  字段的映射（注意：resultType 和resultMap 不能一起使用）
- flushCache：将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空。
- useCache：将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true。
- statementType：STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement。
- resultSets：这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的。


#####insert、delete、update

因为这三者类似，就放在一起去说明。

insert、delete、update 常用属性介绍，涉及到select 的，就不在说明：

- useGeneratedKeys：（仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键。
- keyProperty：（仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
- keyColumn(意思就是有多个主键属性，对应数据库就是联合主键)：（仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。

#####sql

这个元素可以被用来定义可重用的SQL代码段，可以包含在其他语句中。

	</select>
    <sql id = "table">
     ${table}
    </sql>
    <select id = "selectUserById1" resultType="user">
    select * from <include refid="table"><property name="table" value="user"/></include>
    </select>

这是只是起演示作用，没有实际意思。

可以看到的是我们通过include标签，可以去指定sql代码段的属性值，进行动态设置，动态的拼接，把不变的定义成sql代码块，把变化的动态去配置。

#####Parameters

对应映射文件中的参数设置。

	<select id="selectUserById" resultType="user" parameterType="int"><!--resultType 这里的结果类型返回的应该是一个类，这里使用了之前配置文件的别名，简单方便省事，好吧！ -->
		select * from user where userid = #{id}
	</select>

这是最简单的一种设置参数，它适用于java原始类型以及它的包装类（这里可以不显式指定，Mybatis 可以根据TypeHandler 进行推断）。如果要传入一个复杂的对象，包含多个属性，则需要进行如下设置

		<update id="updateUserById" parameterType="user">
		update user set username = #{userName} where userid = #{userId}
		</update>

这是它会根据传入的对象属性，去匹配数据库字段列名，一致就直接赋值到sql中，如果你的属性和数据库字段名不一致，那么需要借助resultMap 去映射对应关系。

#####ResultMap

简单的语句不需要明确的结果映射，而复杂一点的语句只需要描述它们的关系就行了。

	<select id="selectUserInformationById" resultType="map">
        select * from user
    </select>


这里使用的是简单的映射，将每次执行的结果封装成一个map，自动将对应的字段列名就是map的key。

		List<Object> list = sqlSession.selectList("UserMapper.selectUserInformationById");

这里查询的结果由于有多条，需要将map放在list 中，所以采用的是selectlist方式。


运行结果如下：


    [{userid=2, username=laoqiang1}, {userid=3, username=laok}, {userid=4, username=xiaok}, {userid=5, username=wangk}, {userid=6, username=lik}]

但我们知道map不是很好的领域模型，好的领域模型一般是采用javabean和pojo。

	<select id="selectUserInformationById" resultType="user">//这里的user 是User javabean 的别名
        select userid,username,usermessage from user
    </select>

这里就是将查询的结果映射到User javabean 中，它默认将javabean 的属性和数据库字段列对应，忽略大小写，如果不一样，将无法正确赋值。

javabean 的代码就不给出，请自行补充

	List<Object> list = sqlSession.selectList("UserMapper.selectUserInformationById");
		log.debug(list.toString());
		for(int i = 0;i<list.size();i++) {
			User u = (User) list.get(i);
			System.out.println(u.getUserName());
		}


如果你觉得上述数据库字段列和javabean 属性自动映射对应关系不是很清晰，你可以利用sql 特性，去使用别名来实现对应关系。

	<select id="selectUserInformationById" resultType="user">
        select userid as "userId",username as "userName",usermessage as "userMessage" from user
    </select>

通过数据库的as 关键字将列的别名设置成和javabean 属性一致，这样的映射关系更精确。

还可以使用外部的resultMap 去定义javabean 属性和数据库字段的对应关系

	<select id="selectUserInformationById" resultMap="user">
		select userid as "userId",username as "userName",usermessage as
		"userMessage" from user
	</select>

	<resultMap type="user" id="user">
		<result property="userId" column="userid" />
		<result property="userName" column="username" />
		<result property="userMessage" column="usermessage" />
	</resultMap>

比较推荐的就是最后这种映射配置。

高级ResultMap常用的子标签：

- constructor： 用于在实例化类时，默认是通过无参构造函数。如果是你自己提供了有参的构造函数，会覆盖无参的，这是必须用该标签去指定。
- idArg 是constructor的子标签：用在constructor中id 参数，标记出作为 ID 的结果可以帮助提高整体性能。
- arg 是constructor的子标签：用在constructor的普通参数。
- id ：一个 ID 结果;标记出作为 ID 的结果可以帮助提高整体性能。
- result：注入到字段或 JavaBean 属性的普通结果。
- association ：用来映射多个结果的关系。
- collection ：用来映射多个结果的集合。
- discriminator ：使用结果值来决定使用哪个 resultMap。
- case ： 基于某些值的结果映射。

ResultMap 常用的属性：

- id：当前命名空间中的一个唯一标识，用于标识一个ResultMap。
- type：类的完全限定名, 或者一个类型别名，指定映射的javabean。

基于一对一的关联配置

<!-- 注意resultMap 也是标签需要按照一定的顺序的 -->
		<resultMap type="user" id="user">
		<constructor>
		<!-- javabean 里面提供的构造函数包含主键，所以需要使用idArg，column 里面是构造函数中对应的属性的数据库字段名称， 
				我们通过name 去指定javabean 中的构造函数的参数和数据库的字段对应,此时你需要在你javabean 对应的构造函数上的参数，添加@param 
				注解，指定和name一样的值 -->
			<idArg column="userid" name="userId"></idArg>
			<arg column="username" name="userName"></arg>
		</constructor>
		<!-- 这里userid 是表的主键，你用id 标签标识出来 ,有助于性能 -->
		<id property="userId" column="userid" />
		<!-- result 用来对基本字段的映射的 -->
		<result property="userName" column="username" />
		<result property="userMessage" column="usermessage" />
		<!-- 用来描述一一对应的关系 association 里面可以直接引入一个外部的resultmap，可以在内部定义映射关系 -->
		<association property="address" javaType="address"><!-- 这里的javaType必须具备，否则会报错，找不到对应的javabean -->
			<id column="address_id" property="addressId"></id>
			<result column="address_name" property="addressName" />
		</association>
		<!-- columnPrefix 用于在同一个表，有两个一样类型（为了我避免重复的列名，这时mybatis 就会分不清，需要你手动设置），用来区分不同的字段是映射到哪个javabean类对象中，这里的值是根据sql 设置别名的字段前缀一致 -->
        //这个关联只是为了演示columnPrefix 用法
		<association property="address1" javaType="address" columnPrefix="co">
			<id column="address_id" property="addressId"></id>
			<result column="address_name" property="addressName" />
		</association>
	</resultMap>

关联查询语句：

	<select id="select1" resultMap="user">
		select * from user,address where
		user.address_id = address.address_id
	</select>

 javabean(set、get省略，请自行补充！)

	@Alias("user")
	public class User {
	private Integer userId;
	private String userName;
	private String userMessage;
	private Address address;
	private Address address1;

	public Address getAddress1() {
		return address1;
	}

	public void setAddress1(Address address1) {
		this.address1 = address1;
	}

	public Address getAddress() {
		return address;
	}

	public void setAddress(Address address) {
		this.address = address;
	}

	public User(@Param("userId") Integer userId, @Param("userName") String userName) {
		super();
		this.userId = userId;
		this.userName = userName;
	}

	}



######简单例子

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	<mapper namespace="UserMapper">
	<select id="selectUserById" resultType="user">
		select * from user where userid = #{id}
	</select>

    //如果你的数据库支持自动生成主键的字段,那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置到目标属性上就OK了,这是插入的时候，你就不需要在管userId，其实这个没必要，我个人觉得，如果你数据库设置了主键，没有该设置也会自动帮你生成。
	<insert id="insertUser" useGeneratedKeys="true" keyProperty="userId">
		insert into user(username) values (#{username})
	</insert>
    //进行批量insert
	<insert id="insertUserArray">
	 insert into user (username) values
	 <foreach collection="array"   separator="," item="item">
	 (#{item})
	 </foreach>
	</insert>
	
	foreach的主要用在构建insert条件中，它可以在SQL语句中进行迭代一个集合。foreach元素的属性主要有 item，index，collection，open，separator，close。item表示集合中每一个元素进行迭代时的别名，index指 定一个名字，用于表示在迭代过程中，每次迭代到的位置，open表示该语句以什么开始，separator表示在每次进行迭代之间以什么符号作为分隔符（一般都是逗号），close表示以什么结束。
	
	在使用foreach的时候最关键的也是最容易出错的就是collection属性，该属性是必须指定的，但是在不同情况下，该属性的值是不一样的，主要有一下3种情况：

 
	1.如果传入的是单参数且参数类型是一个List的时候，collection属性值为list

	2.如果传入的是单参数且参数类型是一个array数组的时候，collection的属性值为array

	3.如果传入的参数是多个的时候，我们就需要把它们封装成一个Map了，当然单参数也可以封装成map	
	
	<update id="updateUserById">
		<!-- 这里不同于只有一个参数，如果是传一个复合类型的参数，需要保证花括号的值需要和 resultMap中的result的property一致，这样才能保证参数会被正确的传入sql中 -->
		update user set username = #{userName} where userid = #{userId}
	</update>

	<delete id="deleteUserById">
		delete from user where userid=#{id}
	</delete>
	<resultMap type="user" id="user">
		<result property="userId" column="userid" />
		<result property="userName" column="username" />
	</resultMap>
	</mapper>


测试代码：

		sqlSession.insert("UserMapper.insertUser", "laoqiang1");
		User user = new User();
		user.setUserId(1);
		user.setUserName("laotie");
		sqlSession.update("UserMapper.updateUserById",user);//传入的参数是一个对象，它会根据result 将javebean对象的参数传给对应的sql字段
		sqlSession.delete("UserMapper.deleteUserById",1);


        //批量更新

		String[] name = new String[] {"laok","xiaok","wangk","lik"};
		sqlSession.insert("UserMapper.insertUserArray",name);






#### 采用xml 方式配置Mapper

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="UserMapper">
    	<select id="selectUserById" resultType="user"><!--resultType 这里的结果类型返回的应该是一个类，这里使用了之前配置文件的别名，简单方便省事，好吧！ -->
    		select * from user where userid = #{id}
    	</select>
    </mapper>

我们可以在Mapper文件中定义相应的sql语句，在sqlsession 中去全限名(namespace)+id 去调用

同时需要将该Mapper.xml注册到mybatis 的配置文件中。

    <mapper resource="UserMapper.xml"></mapper>

   
执行sql 操作：

     User user = sqlSession.selectOne("com.example.test.mapper.UserMapper.seleectUserById",1);
    
#### 通过代码去配置Mapper

    public interface UserMapper {
    	@Select("select * from user where userid=#{id}")
    	public User selectUserById(int id);
    }


这种方式实际上很类似于我们之前的dao层写法，里面通过注解将sql语句写入，执行该方法的时候，便会执行相应的sql，这里需要注意的一点就是sql 中占位符的参数，最好和方法中的参数的顺序、名称一致。

同时需要将该Mapper 类注册到mybatis 的配置文件中。

		`<mapper class="com.example.test.mapper.UserMapper"/>`

执行sql 操作：
    
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    User user = userMapper.selectUserById(1);
    log.debug("查询的结果是"+user.getUserName());

这两种方法，比较推荐的还是通过Mapper.class方法，采用Mapper.xml
这种方法涉及字符串，如果一旦写错，就会引出错误。



