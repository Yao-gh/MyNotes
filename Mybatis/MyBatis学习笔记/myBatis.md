# 一、MyBatis

​	MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Ordinary Java Object,普通的 Java对象)映射成数据库中的记录。 

```java
/**
	 * 1、根据xml配置文件（全局配置文件）创建一个SqlSessionFactory对象 有数据源一些运行环境信息
	 * 2、sql映射文件；配置了每一个sql，以及sql的封装规则等。 
	 * 3、将sql映射文件注册在全局配置文件中
	 * 4、写代码：
	 * 		1）、根据全局配置文件得到SqlSessionFactory；
	 * 		2）、使用sqlSession工厂，获取到sqlSession对象使用他来执行增删改查
	 * 			一个sqlSession就是代表和数据库的一次会话，用完关闭
	 * 		3）、使用sql的唯一标志来告诉MyBatis执行哪个sql。sql都是保存在sql映射文件中的。
	 * 
	 * @throws IOException
	 */

String resource = "mybatis-config.xml";//全局配置文件
InputStream inputStream = Resources.getResourceAsStream(resource);
// 1、获取sqlSessionFactory对象
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
// 2、获取sqlSession对象
SqlSession openSession = sqlSessionFactory.openSession();
// 3、获取接口的实现类对象
//会为接口自动的创建一个代理对象，代理对象去执行增删改查方法
EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
```



# 二、全局配置文件

####1.根标签

```java
<configuration></<configuration>
```

#### 2.子标签properties

```java
<!--
	mybatis可以使用properties来引入外部properties配置文件的内容；
	   resource：引入类路径下的资源
	   url：引入网络路径或者磁盘路径下的资源
-->
	<properties resource="druid.properties"></properties>
```

#### 3.子标签setting

这是 MyBatis 中极为重要的调整设置，它们会改变MyBatis 的运行时行为。

```java
<!-- 
	 settings包含很多重要的设置项
	    setting:用来设置每一个设置项
		name：设置项名
		value：设置项取值
-->
	<settings>
	//MyBatis 插入空值时，需要指定JdbcType 
	//mybatis insert空值报空值异常，但是在pl/sql不会提示错误，主要原因是mybatis无法进行转换，
		<setting name="jdbcTypeForNull" value="NULL"/>   
	//数据库列一般是以单词命名，单词间以下划线分隔，而java属性一般采用驼峰命名法，这样设置就可以自动进行映射表中的列和类中的属性。默认是false
		<setting name="mapUnderscoreToCamelCase" value="true"/>
	//延迟加载的全局开关。开启时，所有关联的对象都会延迟加载（设置两个）。特定关系中可通过设置fetchType属性来覆盖该项的状态，默认false  
		<setting name="lazyLoadingEnabled" value="true"/>
		<setting name="aggressiveLazyLoading" value="false"/>
	</settings>
```

#### 4.子标签typeAliases：别名处理器

```java
<!-- 
    typeAliases：别名处理器：可以为我们的java类型起别名 别名不区分大小写
-->
	<typeAliases>
		<!-- 
    		1、typeAlias:为某个java类型起别名
			type:指定要起别名的类型全类名;默认别名就是类名小写；employee
			alias:指定新的别名
		 -->
		<typeAlias type="com.atguigu.mybatis.bean.Employee" alias="emp"/>
		
		<!-- 
             2、package:为某个包下的所有类批量起别名 
			name：指定包名（为当前包以及下面所有的后代包的每一个类都起一个默认别名（类名小写））
		-->
		<package name="com.atguigu.mybatis.bean"/>
		
		<!-- 3、批量起别名的情况下，使用@Alias注解为某个类型指定新的别名 -->
	</typeAliases>
	
 虽然有这么多的别名可以使用：但是建议大家还是使用全类名，看SQL语句是怎么被封装为JAVA 对象的时候简单！
```

#### 5.typeHandlers 类型处理器 

​	类型处理器：负责如何将数据库的类型和java对象类型之间转换的工具类

​	无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。

1. 自定义类型

   我们可以重写类型处理器或创建自己的类型处理器来处理不支持的或非标准的类型。

   - 步骤
     - 实现org.apache.ibatis.type.TypeHandler接口或者继承org.apache.ibatis.type.BaseTypeHandler
     - 指定其映射某个JDBC类型（可选操作）
     - 在mybatis全局配置文件中注册

#### 6.objectFactory 对象工厂 

#### 7.plugins 插件 

#### 8.子标签environments

````java
<!-- 
	environments：环境们，mybatis可以配置多种环境 ,default指定使用某种环境。可以达到快速切换环境。
	   	 environment：配置一个具体的环境信息；必须有两个标签；id代表当前环境的唯一标识
	  	 transactionManager：事务管理器；
	   	   type：事务管理器的类型;JDBC(JdbcTransactionFactory)|MANAGED(ManagedTransactionFactory)
				自定义事务管理器：实现TransactionFactory接口.type指定为全类名
		 dataSource：数据源;
					type:数据源类型;UNPOOLED(UnpooledDataSourceFactory)
								|POOLED(PooledDataSourceFactory)
								|JNDI(JndiDataSourceFactory)
					自定义数据源：实现DataSourceFactory接口，type是全类名
		 -->	 
	<environments default="dev_mysql"> //default用来指定使用某种环境，来达到快速切换环境
		<environment id="dev_mysql">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
		
		<environment id="dev_oracle">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${orcl.driver}" />
				<property name="url" value="${orcl.url}" />
				<property name="username" value="${orcl.username}" />
				<property name="password" value="${orcl.password}" />
			</dataSource>
		</environment>
	</environments>
````

#### 9.子标签databaseIdProvider

````java
<!-- 
   databaseIdProvider：支持多数据库厂商的；
	    type="DB_VENDOR"：VendorDatabaseIdProvider
		作用就是得到数据库厂商的标识(驱动getDatabaseProductName())，mybatis就能根据数据库厂商标识来执行不同的sql;
		 	MySQL，Oracle，SQL Server,xxxx
   在sql映射文件中的sql语句中添加 databaseId="mysql"
-->
	<databaseIdProvider type="DB_VENDOR">
		<!-- 为不同的数据库厂商起别名 -->
		<property name="MySQL" value="mysql"/>
		<property name="Oracle" value="oracle"/>
		<property name="SQL Server" value="sqlserver"/>
	</databaseIdProvider>
````

```java
 MyBatis匹配规则如下：
    – 1、如果没有配置databaseIdProvider标签，那么databaseId=null
    – 2、如果配置了databaseIdProvider标签，使用标签配置的name去匹配数据库信息，匹配上设置databaseId=配置指定的值，否则依旧为null
    – 3、如果databaseId不为null，他只会找到配置 databaseId的sql 语句
    – 4、MyBatis 会加载不带 databaseId 属性和带有匹配当前数据库 databaseId 属性的所有语句。如果同时找到带有 databaseId 和不带 databaseId 的相同语句，则后者会被舍弃。
```



#### 10.子标签mappers

````java
<!-- 
    将我们写好的sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中
-->
	<mappers>
		<!-- 
			mapper:注册一个sql映射 
				注册配置文件
				resource：引用类路径下的sql映射文件 mybatis/mapper/EmployeeMapper.xml
				url：引用网路路径或者磁盘路径下的sql映射文件 file:///var/mappers/AuthorMapper.xml
					
				注册接口
				class：引用（注册）接口，
					1、有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下；
					2、没有sql映射文件，所有的sql都是利用注解写在接口上;
					推荐：
						比较重要的，复杂的Dao接口我们来写sql映射文件
						不重要，简单的Dao接口为了开发快速可以使用注解；
		-->
		<mapper resource="mybatis/mapper/EmployeeMapper.xml"/>
		<mapper class="com.atguigu.mybatis.dao.EmployeeMapperAnnotation"/>
		
		<!-- 批量注册： -->
		<package name="com.atguigu.mybatis.dao"/>
	</mappers>
````



# 三、SQL映射文件

####1.标签mapper

````java
<mapper namespace="">
<!-- 
        namespace:名称空间;指定为接口的全类名
        id：唯一标识
        resultType：返回值类型
        #{id}：从传递过来的参数中取出id值
 -->
</mapper>
````

####2.增删改

````java
1.  /**
	 * 1、mybatis允许增删改直接定义以下类型返回值
	 * 		Integer、Long、Boolean、void
	 * 2、我们需要手动提交数据
	 * 		sqlSessionFactory.openSession();===》手动提交
	 * 		sqlSessionFactory.openSession(true);===》自动提交
	 * @throws IOException 
	 */
2. <!--resultType：如果返回的是一个集合，要写集合中元素的类型  -->
3. <!-- parameterType：参数类型，可以省略， 
	获取自增主键的值：
		mysql支持自增主键，自增主键值的获取，mybatis也是利用statement.getGenreatedKeys()；
		useGeneratedKeys="true"；使用自增主键获取主键值策略
		keyProperty；指定对应的主键属性，也就是mybatis获取到主键值以后，将这个值封装给javaBean的哪个属性
	-->
````

#### 3.参数处理

````java
1.单个参数：mybatis不会做特殊处理，
	#{参数名/任意名}：取出参数值。
2.多个参数：mybatis会做特殊处理。
	多个参数会被封装成 一个map，
		key：param1...paramN,或者参数的索引也可以
		value：传入的参数值
	#{}就是从map中获取指定的key的值；
	//两个参数，按单个参数那样取值
	异常：
	org.apache.ibatis.binding.BindingException: 
	Parameter 'id' not found. 
	Available parameters are [1, 0, param1, param2]
	操作：
		方法：public Employee getEmpByIdAndLastName(Integer id,String lastName);
		取值：#{id},#{lastName}

【命名参数】：明确指定封装参数时map的key；@Param("id")
	多个参数会被封装成 一个map，
		key：使用@Param注解指定的值
		value：参数值
	#{指定的key}取出对应的参数值
POJO：（Javabena）
如果多个参数正好是我们业务逻辑的数据模型，我们就可以直接传入pojo；
	#{属性名}：取出传入的pojo的属性值	
Map：
如果多个参数不是业务模型中的数据，没有对应的pojo，不经常使用，为了方便，我们也可以传入map
	#{key}：取出map中对应的值
TO：
如果多个参数不是业务模型中的数据，但是经常要使用，推荐来编写一个TO（Transfer Object）数据传输对象
Page{
	int index;
	int size;
}

========================思考================================	
public Employee getEmp(@Param("id")Integer id,String lastName);
	取值：id==>#{id/param1}   lastName==>#{param2}

public Employee getEmp(Integer id,@Param("e")Employee emp);
	取值：id==>#{param1}    lastName===>#{param2.lastName/e.lastName}

##特别注意：如果是Collection（List、Set）类型或者是数组，也会特殊处理。
		  也是把传入的list或者数组封装在map中。
		  key：Collection（collection）,
			   如果是List还可以使用这个key(list)
		  	   数组(array)
public Employee getEmpById(List<Integer> ids);
	取值：取出第一个id的值：   #{list[0]}
	
========================结合源码，mybatis怎么处理参数==========================
总结：参数多时会封装map，为了不混乱，我们可以使用@Param来指定封装时使用的key；
#{key}就可以取出map中的值；

(@Param("id")Integer id,@Param("lastName")String lastName);
ParamNameResolver解析参数封装map的；
//1、names：{0=id, 1=lastName}；构造器的时候就确定好了
	确定流程：
	1.获取每个标了param注解的参数的@Param的值：id，lastName；  赋值给name;
	2.每次解析一个参数给map中保存信息：（key：参数索引，value：name的值）
		name的值：
			标注了param注解：注解的值
			没有标注：
				1.全局配置：useActualParamName（jdk1.8）：name=参数名
				2.name=map.size()；相当于当前元素的索引
	{0=id, 1=lastName,2=2}
args【1，"Tom",'hello'】:
public Object getNamedParams(Object[] args) {
    final int paramCount = names.size();
    //1、参数为null直接返回
    if (args == null || paramCount == 0) {
      return null;
    //2、如果只有一个元素，并且没有Param注解；args[0]：单个参数直接返回
    } else if (!hasParamAnnotation && paramCount == 1) {
      return args[names.firstKey()];     
    //3、多个元素或者有Param标注
    } else {
      final Map<String, Object> param = new ParamMap<Object>();
      int i = 0;
      //4、遍历names集合；{0=id, 1=lastName,2=2}
      for (Map.Entry<Integer, String> entry : names.entrySet()) {  
      	//names集合的value作为key;  names集合的key又作为取值的参考args[0]:args【1，"Tom"】:
      	//eg:{id=args[0]:1,lastName=args[1]:Tom,2=args[2]}
        param.put(entry.getValue(), args[entry.getKey()]); 
        // add generic param names (param1, param2, ...)param
        //额外的将每一个参数也保存到map中，使用新的key：param1...paramN
        //效果：有Param注解可以#{指定的key}，或者#{param1}
        final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
        // ensure not to overwrite parameter named with @Param
        if (!names.containsValue(genericParamName)) {
          param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
      }
      return param;
    }
  }
}
===========================参数值的获取======================================
#{}：可以获取map中的值或者pojo对象属性的值；
${}：可以获取map中的值或者pojo对象属性的值；

select * from tbl_employee where id=${id} and last_name=#{lastName}
Preparing: select * from tbl_employee where id=2 and last_name=?
	区别：
		#{}:是以预编译的形式，将参数设置到sql语句中；PreparedStatement；防止sql注入
		${}:取出的值直接拼装在sql语句中；会有安全问题；
		大多情况下，我们去参数的值都应该去使用#{}；
		原生jdbc不支持占位符的地方我们就可以使用${}进行取值
		比如分表、排序。。。；按照年份分表拆分
			select * from ${year}_salary where xxx;
			select * from tbl_employee order by ${f_name} ${order}
#{}:更丰富的用法：
	规定参数的一些规则：
	javaType、 jdbcType、 mode（存储过程）、 numericScale、
	resultMap、 typeHandler、 jdbcTypeName、 expression（未来准备支持的功能）；

	jdbcType通常需要在某种特定的条件下被设置：
		在我们数据为null的时候，有些数据库可能不能识别mybatis对null的默认处理。比如Oracle（报错）；
		JdbcType OTHER：无效的类型；因为mybatis对所有的null都映射的是原生Jdbc的OTHER类型，oracle不能正确处理;
		
		由于全局配置中：jdbcTypeForNull=OTHER；oracle不支持；两种办法
		1、#{email,jdbcType=OTHER};
		2、jdbcTypeForNull=NULL
			<setting name="jdbcTypeForNull" value="NULL"/>
````

#### 4.select返回值

#####1.resultType

```java
1.返回javaBean
2.返回list //resultType：如果返回的是一个集合，要写集合中元素的类型
3.返回map 
	//1).返回一条记录的map；key就是列名，值就是对应的值  resultType="map"
	//2)多条记录封装一个map：Map<Integer,Employee>:键是这条记录的主键，值是记录封装后的javaBean  
	//resultType="com.atguigu.mybatis.bean.Employee"
	//@MapKey:告诉mybatis封装这个map的时候使用哪个属性作为map的key
```

##### 2.resultMap:自定义结果集映射规则

````java
1.自定义某个Javabean的封装规则
<!--自定义某个javaBean的封装规则
	type：自定义规则的Java类型
	id:唯一id方便引用
-->
	<resultMap type="com.atguigu.mybatis.bean.Employee" id="MySimpleEmp">
		<!--指定主键列的封装规则
            id定义主键，底层有优化；
                column：指定哪一列
                property：指定对应的javaBean属性
		-->
		<id column="id" property="id"/>
		<!-- 定义普通列封装规则 -->
		<result column="last_name" property="lastName"/>
		<!-- 其他不指定的列会自动封装：我们只要写resultMap就把全部的映射规则都写上。 -->
		<result column="email" property="email"/>
		<result column="gender" property="gender"/>
	</resultMap>
	<select id="getEmpById"  resultMap="MySimpleEmp">
		select * from tbl_employee where id=#{id}
	</select>
````

```java
2.联合查询：级联属性封装结果集(一对一)
    (1)
        <resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp">
            <id column="id" property="id"/>
            <result column="last_name" property="lastName"/>
            <result column="gender" property="gender"/>
            //另一张表得数据封装为对象
            <result column="did" property="dept.id"/>
            <result column="dept_name" property="dept.departmentName"/>
        </resultMap>
	(2)使用association定义关联的单个对象的封装规则
        <resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp2">
            <id column="id" property="id"/>
            <result column="last_name" property="lastName"/>
            <result column="gender" property="gender"/>
            <!--  
    			association可以指定联合的javaBean对象
                    property="dept"：指定哪个属性是联合的对象
                    javaType:指定这个属性对象的类型[不能省略]
            -->
            <association property="dept" javaType="com.atguigu.mybatis.bean.Department">
                <id column="did" property="id"/>
                <result column="dept_name" property="departmentName"/>
            </association>
        </resultMap>
        <select id="getEmpAndDept" resultMap="MyDifEmp">
		select e.id id,e.last_name last_name,e.gender gender,e.d_id d_id,d.id did,d.dept_name dept_name FROM tbl_employee e,tbl_dept d WHERE e.d_id=d.id AND e.id=#{id}
		</select>
	 (3)使用association进行分步查询：
	 	1、先按照员工id查询员工信息
		2、根据查询员工信息中的d_id值去部门表查出部门信息
		3、部门设置到员工中；
         <resultMap type="com.atguigu.mybatis.bean.Employee" id="MyEmpByStep">
            <id column="id" property="id"/>
            <result column="last_name" property="lastName"/>
            <result column="email" property="email"/>
            <result column="gender" property="gender"/>
            <!-- 
            	association定义关联对象的封装规则
                    select:表明当前属性是调用select指定的方法查出的结果
                    column:指定将哪一列的值传给这个方法
                流程：使用select指定的方法（传入column指定的这列参数的值）查出对象，并封装给property指定的属性
             -->
            <association property="dept" select="com.atguigu.mybatis.dao.DepartmentMapper.getDeptById" column="d_id">
            </association>
         </resultMap>
         <select id="getEmpByIdStep" resultMap="MyEmpByStep">
            select * from tbl_employee where id=#{id}
            <if test="_parameter!=null">
                and 1=1
            </if>
         </select>
         
<!-- 可以使用延迟加载（懒加载）；(按需加载)
	 	Employee==>Dept：
	 		我们每次查询Employee对象的时候，都将一起查询出来。
	 		部门信息在我们使用的时候再去查询；
	 		分段查询的基础之上加上两个配置：
 -->
```

```java
3.联合查询（一对多）
	(1)
    <!--嵌套结果集的方式，使用collection标签定义关联的集合类型的属性封装规则  -->
        <resultMap type="com.atguigu.mybatis.bean.Department" id="MyDept">
            <id column="did" property="id"/>
            <result column="dept_name" property="departmentName"/>
            <!-- 
                collection定义关联集合类型的属性的封装规则 
                ofType:指定集合里面元素的类型
            -->
            <collection property="emps" ofType="com.atguigu.mybatis.bean.Employee">
                <!-- 定义这个集合中元素的封装规则 -->
                <id column="eid" property="id"/>
                <result column="last_name" property="lastName"/>
                <result column="email" property="email"/>
                <result column="gender" property="gender"/>
            </collection>
        </resultMap>
        <!-- public Department getDeptByIdPlus(Integer id); -->
        <select id="getDeptByIdPlus" resultMap="MyDept">
            SELECT d.id did,d.dept_name dept_name,
                    e.id eid,e.last_name last_name,e.email email,e.gender gender
            FROM tbl_dept d
            LEFT JOIN tbl_employee e
            ON d.id=e.d_id
            WHERE d.id=#{id}
        </select>
	(2)collection：分段查询
        <resultMap type="com.atguigu.mybatis.bean.Department" id="MyDeptStep">
            <id column="id" property="id"/>
            <id column="dept_name" property="departmentName"/>
            <collection property="emps" 
                select="com.atguigu.mybatis.dao.EmployeeMapperPlus.getEmpsByDeptId"
                column="{deptId=id}" fetchType="lazy">
            </collection>
        </resultMap>
        <!-- public Department getDeptByIdStep(Integer id); -->
        <select id="getDeptByIdStep" resultMap="MyDeptStep">
            select id,dept_name from tbl_dept where id=#{id}
        </select>
        
	<!-- 扩展：多列的值传递过去：
			将多列的值封装map传递；
			column="{key1=column1,key2=column2}"
			fetchType="lazy"：表示使用延迟加载；
				- lazy：延迟
				- eager：立即
	 -->
	<!-- <discriminator javaType=""></discriminator>
		鉴别器：mybatis可以使用discriminator判断某列的值，然后根据某列的值改变封装行为
		封装Employee：
			如果查出的是女生：就把部门信息查询出来，否则不查询；
			如果是男生，把last_name这一列的值赋值给email;
	 -->
     <resultMap type="com.atguigu.mybatis.bean.Employee" id="MyEmpDis">
	 	<id column="id" property="id"/>
	 	<result column="last_name" property="lastName"/>
	 	<result column="email" property="email"/>
	 	<result column="gender" property="gender"/>
	 	<!--column：指定判定的列名 javaType：列值对应的java类型  -->
	 	<discriminator javaType="string" column="gender">
	 		<!--女生  resultType:指定封装的结果类型；不能缺少。/resultMap-->
	 		<case value="0" resultType="com.atguigu.mybatis.bean.Employee">
	 			<association property="dept" 
			 		select="com.atguigu.mybatis.dao.DepartmentMapper.getDeptById"
			 		column="d_id">
		 		</association>
	 		</case>
	 		<!--男生 ;如果是男生，把last_name这一列的值赋值给email; -->
	 		<case value="1" resultType="com.atguigu.mybatis.bean.Employee">
		 		<id column="id" property="id"/>
			 	<result column="last_name" property="lastName"/>
			 	<result column="last_name" property="email"/>
			 	<result column="gender" property="gender"/>
	 		</case>
	 	</discriminator>
	 </resultMap>
```

#### 5.动态sql

````html
	<!-- 
        • where 
        • if:判断
        • choose (when-otherwise):分支选择；带了break的swtich-case
        	如果带了id就用id查，如果带了lastName就用lastName查;只会进入其中一个
        • trim 字符串截取(where(封装查询条件), set(封装修改条件))
        • foreach 遍历集合
        • set:与if联用，用于update
	 -->
````

```java
1.<!-- 自定义字符串的截取规则 -->
     <select id="getEmpsByConditionTrim" resultType="com.atguigu.mybatis.bean.Employee">
	 	select * from tbl_employee
	 <!-- 后面多出的and或者or where标签不能解决 
	 	prefix="":前缀：trim标签体中是整个字符串拼串 后的结果。 prefix给拼串后的整个字符串加一个前缀 
	 	prefixOverrides="":前缀覆盖： 去掉整个字符串前面多余的字符
	 	suffix="":后缀 suffix给拼串后的整个字符串加一个后缀 
	 	suffixOverrides="" 后缀覆盖：去掉整个字符串后面多余的字符		
	 -->
	 	<!-- 自定义字符串的截取规则 -->
	 	<trim prefix="where" suffixOverrides="and">
	 		<if test="id!=null">
		 		id=#{id} and
		 	</if>
		 	<if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
		 		last_name like #{lastName} and
		 	</if>
		 	<if test="email!=null and email.trim()!=&quot;&quot;">
		 		email=#{email} and
		 	</if> 
		 	<!-- ognl会进行字符串与数字的转换判断  "0"==0 -->
		 	<if test="gender==0 or gender==1">
		 	 	gender=#{gender}
		 	</if>
		 </trim>
	 </select>
```

````java
2.choose (when-otherwise)
<select id="getEmpsByConditionChoose" resultType="com.atguigu.mybatis.bean.Employee">
	 	select * from tbl_employee 
	 	<where>
	 		<!-- 如果带了id就用id查，如果带了lastName就用lastName查;只会进入其中一个 -->
	 		<choose>
	 			<when test="id!=null">
	 				id=#{id}
	 			</when>
	 			<when test="lastName!=null">
	 				last_name like #{lastName}
	 			</when>
	 			<when test="email!=null">
	 				email = #{email}
	 			</when>
	 			<otherwise>
	 				gender = 0
	 			</otherwise>
	 		</choose>
	 	</where>
	 </select>
````

```java
3.set
	<update id="updateEmp">
	 	update tbl_employee 
		<set>
			<if test="lastName!=null">
				last_name=#{lastName},
			</if>
			<if test="email!=null">
				email=#{email},
			</if>
			<if test="gender!=null">
				gender=#{gender}
			</if>
		</set>
		where id=#{id} 
	</update>
```

```jav
4.foreach
	(a)
	<select id="getEmpsByConditionForeach" resultType="com.atguigu.mybatis.bean.Employee">
	 	select * from tbl_employee
	 	<!--
	 		collection：指定要遍历的集合：
	 			list类型的参数会特殊处理封装在map中，map的key就叫list
	 		item：将当前遍历出的元素赋值给指定的变量
	 		separator:每个元素之间的分隔符
	 		open：遍历出所有结果拼接一个开始的字符
	 		close:遍历出所有结果拼接一个结束的字符
	 		index:索引。遍历list的时候是index就是索引，item就是当前值
	 				      遍历map的时候index表示的就是map的key，item就是map的值
	 		
	 		#{变量名}就能取出变量的值也就是当前遍历出的元素
	 	  -->
	 	<foreach collection="ids" item="item_id" separator="," open="where id in(" close=")">
	 		#{item_id}
	 	</foreach>
	 </select>
	 
	 （b）批量保存
	 <!--MySQL下批量保存：可以foreach遍历   mysql支持values(),(),()语法-->
	 <!--public void addEmps(@Param("emps")List<Employee> emps);  -->
        <insert id="addEmps">
            insert into tbl_employee(字段...) 
            values
            <foreach collection="emps" item="emp" separator=",">
                (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
            </foreach>
         </insert>
         
        <!-- 这种方式需要数据库连接url属性allowMultiQueries=true；
        	 这种分号分隔多个sql可以用于其他的批量操作（删除，修改） 
        -->
	 <insert id="addEmps">
	 	<foreach collection="emps" item="emp" separator=";">
	 		insert into tbl_employee(last_name,email,gender,d_id)
	 		values(#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
	 	</foreach>
	 </insert>
```

```java
5.两个内置参数
		<!-- 两个内置参数：
			不只是方法传递过来的参数可以被用来判断，取值。。。
			mybatis默认还有两个内置参数：
			_parameter:代表整个参数
				单个参数：_parameter就是这个参数
				多个参数：参数会被封装为一个map；_parameter就是代表这个map
			
			_databaseId:如果配置了databaseIdProvider标签。
			_databaseId就是代表当前数据库的别名oracle
		  -->
```

```java
6.<!-- bind：可以将OGNL表达式的值绑定到一个变量中，方便后来引用这个变量的值 -->
    <select id="getEmpsTestInnerParameter" resultType="com.atguigu.mybatis.bean.Employee">
	  		<!-- bind：可以将OGNL表达式的值绑定到一个变量中，方便后来引用这个变量的值 -->
	  		<bind name="_lastName" value="'%'+lastName+'%'"/>
	  		<if test="_databaseId=='mysql'">
	  			select * from tbl_employee
	  			<if test="_parameter!=null">
	  				where last_name like #{lastName}
	  			</if>
	  		</if>
	  		<if test="_databaseId=='oracle'">
	  			select * from employees
	  			<if test="_parameter!=null">
	  				where last_name like #{_parameter.lastName}
	  			</if>
	  		</if>
	  </select>
```

```java
7.抽去重用代码
<!-- 
	  	抽取可重用的sql片段。方便后面引用 
	  	1、sql抽取：经常将要查询的列名，或者插入用的列名抽取出来方便引用
	  	2、include来引用已经抽取的sql：
	  	3、include还可以自定义一些property，sql标签内部就能使用自定义的属性
	  			include-property：取值的正确方式${prop},
	  			#{不能使用这种方式}
	  -->
	  <sql id="insertColumn">
	  		<if test="_databaseId=='oracle'">
	  			employee_id,last_name,email
	  		</if>
	  		<if test="_databaseId=='mysql'">
	  			last_name,email,gender,d_id
	  		</if>
	  </sql>
	  
       使用 <include refid="insertColumn"></include>
```

#### 6.OGNL

````java
	OGNL(Object Graph Navigation Language)对象图导航 语言， 这是一种强大的表达式语言 ，通过它可以非常方便的来操作对象属性。 类似于我们的 EL等
	访问对象属性： person.name
    调用方法： 	  person.getName()
    调用静态属性/方法： @java.lang.Math@PI
      				 @java.util.UUID@randomUUID()
    调用构造方法： new com.atguigu.bean.Person(‘admin’).name
    运算符： +,-*,/,%
    逻辑运算符： in,not in,>,>=,<,<=,==,!= //注意：xml中特殊符号如”,>,<等这些都需要使用转义字符
````



# 四、MyBatis逆向工程

````java
MyBatis Generator ：
	简称MBG，是一个专门为MyBatis框架使用者定制的代码生成器，可以快速的根据表生成对应的映射文件，接口，以及bean类。支持基本的增删改查，以及QBC风格的条件查询。但是表连接、存储过程等这些复杂sql的定义需要我们手工编写
	官方文档地址 http://www.mybatis.org/generator/
	官方工程地址 https://github.com/mybatis/generator/releases
````

````java
• 使用步骤：
– 1）编写MBG的配置文件（重要几处配置）
    1）jdbcConnection配置数据库连接信息
    2）javaModelGenerator配置javaBean的生成策略
    3）sqlMapGenerator 配置sql映射文件生成策略
    4）javaClientGenerator配置Mapper接口的生成策略
    5）table 配置要逆向解析的数据表
        tableName：表名
        domainObjectName：对应的javaBean名
– 2）运行代码生成器生成代码
• 注意：
    Context标签
        targetRuntime =“MyBatis3“可以生成带条件的增删改查
        targetRuntime =“MyBatis3Simple“可以生成基本的增删改查
如果再次生成，建议将之前生成的数据删除，避免xml向后追加内容出现的问题
````

# 五、MyBatis运行原理

````java
1.获取sqlSessionFactory对象
	解析文件的每一个信息保存在Configuration中，返回包含Configuration的DefaultSQLSessionFactory
	注意：MapperedStatment代表一个增删改查的详细信息
2.获取sqlSession对象
	返回一个DefaultSQLSession对象，包含Configuration和Executor
	这一步创建Executor对象
3.获取接口的实现类对象（MapperProxy）
	getMapper使用MapperProxyFactory创建一个MapperProxy代理对象
	代理对象里面包含了DefaultSqlSession(Exector)
4.执行增删改查方法
````

1. 获取sqlSessionFactory对象

   ![sqlSessionFactory初始化](.\imgs\sqlSessionFactory初始化.png)

2. 获取sqlSession对象

![sqlSession](.\imgs\sqlSession.png)

3. 获取接口的实现类对象（MapperProxy）

   ![MapperProxy](.\imgs\MapperProxy.png)

4. 执行增删改查方法



#六、MyBatis的插件