# #{ } 和 ${ } 取值的区别


```
#{} : 是以预编译的形式，将参数设置到sql语句中，防止sql注入；

${} : 取出的值会直接拼接在sql语句中，会有安全问题；
```


# ​`resultMap` 自定义结果


- `resultType` 和 `resultMap` 只能同时用一个



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.StudentMapper">

	<!--自定义某个javaBean的封装规则
	
	type：自定义规则的Java类型
	
	id:唯一id方便引用
	  -->
	<resultMap type="com.garcon.bean.Student" id="myStudent">
	
		<!--指定主键列的封装规则
		
		id定义主键会底层有优化
	 
		column：指定哪一列
		
		property：指定对应的javaBean属性
		  -->
		<id column="sid" property="sid"/>
		
		<!-- 定义普通列封装规则 -->
		<result column="last_name" property="lastName"/>
		
		<!-- 其他不指定的列会自动封装：我们只要写resultMap就把全部的映射规则都写上。 -->
		<result column="gender" property="gender"/>
		<result column="hobby" property="hobby"/>
	</resultMap>
	
	<!-- resultMap:自定义结果集映射规则；  -->
	<select id="getStudentById"  resultMap="myStudent">
		select * from student where sid=#{sid}
	</select>
	
</mapper>
```


# 关联查询


- 自定义 `关联查询` 结果集类型
- 查询 `学生信息` 的时候把对应的 `班级信息` 也查询出来
- `Student.java` 学生信息



```java
package com.garcon.bean;

public class Student {

	private Integer sid;
	private String lastName;
	private String gender;
	private String hobby;
	private Sclass sclass;//班级信息
	
	...
	}
```


- `Sclass.java` 班级信息



```
package com.garcon.bean;

public class Sclass {

	private Integer cid;//班级id
	private String cName;//班级名称
	
	...
	}
```


- `StudentMapper.java` 接口



```java
package com.garcon.dao;

public interface StudentMapper {

    //以sid查询学生信息
    public Student getStudentById(Integer sid);
}
```


- `StudentMapper.xml` 映射文件—— `resultMap` 级联属性



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.StudentMapper">

    <!--联合查询：级联属性封装结果集 -->
	<resultMap type="com.garcon.bean.Student" id="mystudent">
		<id column="sid" property="sid"/>
		<result column="last_name" property="lastName"/>
		<result column="gender" property="gender"/>
		<result column="hobby" property="hobby"/>
		<!--关联列用级联属性-->
		<result column="cid" property="sclass.cid"/>
		<result column="cname" property="sclass.name"/>
	</resultMap>
	
	<!--级联查询-->
	<select id="getStudentById"  resultMap="myStudent">
		SELECT
    		s.sid,
    		s.last_name,
    		s.gender,
    		s.hobby,
    		c.cid,
    		c.cname
		FROME
		    student s,
		    sclass c
		WHERE
		    s.sid=c.cid
		  AND
		    sid=#{sid}
	</select>
	
</mapper>
```


或 以下


- `StudentMapper.xml` 映射文件—— `resultMap` 的 `association` 属性



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.StudentMapper">

    <!--使用association定义关联的单个对象的封装规则-->
	<resultMap type="com.garcon.bean.Student" id="myStudent">
		<id column="sid" property="sid"/>
		<result column="last_name" property="lastName"/>
		<result column="gender" property="gender"/>
		<result column="hobby" property="hobby"/>
		
		<!-- 
		association:可以指定联合的javaBean对象
		    property="sclass"：指定哪个属性是联合的对象
		    javaType:指定这个属性对象的类型[不能省略]
		-->
		<association property="sclass" javaType="com.garcon.bean.Sclass">
			<id column="cid" property="cid"/>
			<result column="cname" property="cname"/>
		</association>
	</resultMap>
	
	<!--关联查询-->
	<select id="getStudentById"  resultMap="myStudent">
		SELECT
    		s.sid,
    		s.last_name,
    		s.gender,
    		s.hobby,
    		c.cid,
    		c.cname
		FROME
		    student s,
		    sclass c
		WHERE
		    s.sid=c.cid
		  AND
		    sid=#{sid}
	</select>
	
</mapper>
```


# 关联查询 collection


- #查询班级信息的同时查出班级所对应的学生信息
- `Student.java` 学生类



```java
package com.garcon.bean;

public class Student {

	private Integer sid;
	private String lastName;
	private String gender;
	private String hobby;
	private Sclass sclass;//班级信息
	
	...
	}
```


- `Sclass.java` 班级类



```java
package com.garcon.bean;

public class Sclass {

	private Integer cid;//班级id
	private String cName;//班级名称
	private List<Student> students;
	
	...
	}
```


- `SclassMapper.java` 班级接口



```java
package com.garcon.dao;

public interface SclassMapper {

    //以班级id查询班级
    public Sclass getSclassById(Integer cid);
}
```


- `sclassMapper.xml` 班级SQL映射文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.SclassMapper">
	
 <!--嵌套结果集的方式，使用collection标签定义关联的集合类型的属性封装规则  -->
	<resultMap type="com.garcon.bean.Sclass" id="Mysclass">
		<id column="cid" property="cid"/>
		<result column="cname" property="cName"/>
		<!-- 
			collection定义关联集合类型的属性的封装规则 
			ofType:指定集合里面元素的类型
		-->
		<collection property="students" ofType="com.garcon.bean.Student">
			<!-- 定义这个集合中元素的封装规则 -->
			<id column="sid" property="sid"/>
			<result column="last_name" property="lastName"/>
			<result column="gender" property="gender"/>
			<result column="hobby" property="hobby"/>
		</collection>
	</resultMap>
 
	<select id="getSclassById" resultMap="Mysclass">
		SELECT
	        s.sid,
	        s.last_name,
	        s.gender,
	        s.hobyy,
	        c.cid
	        c.cname
		FROM
		    student s
		LEFT JOIN
		    sclass c
		ON 
		    s.cid=c.cid
		WHERE
		    c.cid=#{cid}
	</select>
	
</mapper>
```


# 分步查询 association


- 先以学生的 `学号` 查询学生信息，再以 `学生信息中的班级编号` 查询班级信息
- `SclassMapper.java` 班级接口



```java
package com.garcon.dao;

public interface Sclass {

    //以班级id查询班级信息
    public Sclass getSclasstById(Integer cid);
}
```


- `StudentMapper.java` 学生接口



```java
package com.garcon.dao;

public interface StudentMapper {

    //以sid查询学生信息
    public Student getStudentById(Integer sid);
}
```


- `sclassMapper.xml` 班级SQL映射文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.SclassMapper">
	
	<!--以班级号查询班级信息-->
	<select id="getSclassById" resultType="com.garcon.bean.Sclass">
        SELECT * FROM sclass WHERE cid=#{cid}
	</select>
	
</mapper>
```


- `studentMapper.xml` 学生SQL映射文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.StudentMapper">
	
    <!-- 
        使用association进行分步查询：
		    1、先按照学生id查询学生工信息
	    	2、根据查询学生信息中的cid值去班级表查出班级信息
	    	3、班级设置到学生中；
	 -->
	 
	 <resultMap type="com.garcon.bean.Student" id="Mystudent">
	 	<id column="sid" property="sid"/>
	 	<result column="last_name" property="lastName"/>
	 	<result column="gender" property="gender"/>
	 	<result column="hobby" property="hobby"/>
	 	
	 	<!-- 
	 	    association:定义关联对象的封装规则
	 	    	select:表明当前属性是调用select指定的方法查出的结果
	 	    	column:指定将哪一列的值传给这个方法
	 		
	 	   	流程：使用select指定的方法（传入column指定的这列参数的值）查出对象，并封装给property指定的属性
	 	 -->
 		<association property="dept" 
	 		select="com.garcon.dao.SclassMapper.getSclassById"
	 		column="cid">
 		</association>
	 </resultMap>
	
	 <select id="getStudentById" resultMap="Mystudent">
	 	SELECT * FROM student WHERE sid=#{sid}
	 </select>
	
</mapper>
```


# 分步查询 collection


- 先以 `班级编号` 查询班级信息，再以 `班级编号` 查询班级的所有学生信息
- `SclassMapper.java` 班级接口



```java
package com.garcon.dao;

public interface Sclass {

    //以班级id查询班级信息
    public Sclass getSclasstById(Integer cid);
}
```


- `StudentMapper.java` 学生接口



```java
package com.garcon.dao;

public interface StudentMapper {

    //以班级编号查询学生信息
    public List<Student> getStudentByClassId(Integer sid);
}
```


- `sclassMapper.xml` 班级SQL映射文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.SclassMapper">
	
    <!-- collection：分段查询 -->
	<resultMap type="com.garcon.bean.Sclass" id="mySclass">
		<id column="cid" property="cid"/>
		<id column="cname" property="cName"/>
		<collection property="students" 
			select="com.garcon.dao.StudentMapper.getStudentByClassId"
			column="cid" fetchType="lazy"></collection>
	</resultMap>
 
	<select id="getSclasstById" resultMap="mySclass">
		select cid,cName from sclass where cid=#{cid}
	</select>
	
	<!--
	    扩展：
	    多列的值传递过去：
			将多列的值封装map传递；
			column="{key1=column1,key2=column2}"
			如：
			<collection 
			    property="" 
    			select=""
    			column="{key1=column1,key2=column2}"
    			fetchType="lazy">
    		</collection>
			
		    fetchType="lazy"：表示使用延迟加载；

				- lazy：延迟
				- eager：立即

	 -->
	
</mapper>
```


- `studentMapper.xml` 学生SQL映射文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.StudentMapper">
	
	<!--以班级编号查询所有学生-->
	 <select id="getStudentById" resultType="com.garcon.bean.Student">
	 	SELECT * FROM student WHERE class_id=#{class_id}
	 </select>
	
</mapper>
```


# 分步查询 延迟加载


#适用情况：假如查看学生信息的时候不加载班级信息，当需要查看学生的班级信息的时候再加载班级信息


- 在分步查询的基础上，在mybatis的全局配置文件的settings设置项中添加如下设置即可



```xml
<settings>
		<!--显示的指定每个我们需要更改的配置的值，即使他是默认的。防止版本更新带来的问题  -->
		<setting name="lazyLoadingEnabled" value="true"/>
		<setting name="aggressiveLazyLoading" value="false"/>
</settings>
```


# 鉴别器-discriminator


鉴别器：mybatis可以使用discriminator判断某列的值，然后根据某列的值改变封装行为


-  如果查出的是女生：就把班级信息查询出来，否则不查询；如果是男生，把last_name这一列的值赋值给 hobby; 
-  `studentMapper.xml` 



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.StudentMapper">
    <!-- 
		鉴别器：mybatis可以使用discriminator判断某列的值，然后根据某列的值改变封装行为
		封装Student：
			如果查出的是女生：就把班级信息查询出来，否则不查询；
			如果是男生，把last_name这一列的值赋值给 hobby;
	 -->
	 <resultMap type="com.garcon.bean.Student" id="mystudent">
	 	<id column="sid" property="sid"/>
	 	<result column="last_name" property="lastName"/>
	 	<result column="gender" property="gender"/>
	 	<result column="hobby" property="hobby"/>
	 	
	 	<!--
	 		column：指定判定的列名
	 		javaType：列值对应的java类型 
	 		-->
	 	<discriminator javaType="string" column="gender">
	 		<!--女生  resultType:指定封装的结果类型；不能缺少。/resultMap-->
	 		<case value="女" resultType="com.garcon.bean.Student">
	 			<association property="sclass" 
			 		select="com.garcon.dao.SclassMapper.getSclasstById"
			 		column="class_id">
		 		</association>
	 		</case>
	 		<!--男生 ;如果是男生，把last_name这一列的值赋值给hobby; -->
	 		<case value="男" resultType="com.garcon.bean.Student">
		 		<id column="id" property="id"/>
			 	<result column="last_name" property="lastName"/>
			 	<result column="last_name" property="hobby"/>
			 	<result column="gender" property="gender"/>
	 		</case>
	 	</discriminator>
	 </resultMap>
```


- `sclassMapper.xml`



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.garcon.dao.SclassMapper">
	
	<!--以班级号查询班级信息-->
	<select id="getSclassById" resultType="com.garcon.bean.Sclass">
        SELECT * FROM sclass WHERE cid=#{cid}
	</select>
	
</mapper>
```


# 动态SQL


## if


```xml
<!--test:判断表达式(OGNL)-->
<if test=""></if>

<!--
    应用场境：按条件查询学生，
        如果参数中有sid则查询条件带上sid
        如果参数中有last_name则查询条件带上last_name
        如果参数中有gender则查询条件带上gender
-->

<select id="getStudent" resultType="com.garcon.dao.StudentMapper">
	 	select * from student
	 	<!-- where -->
	 	<where>
		 	<if test="sid!=null">
		 		sid=#{sid}
		 	</if>
		 	<if test="lastName!=null AND lastName!=''">
		 		and last_name like #{lastName}
		 	</if>
		 	<if test="hobby!=null and hobby.trim()!=''">
		 		and hobby=#{hobby}
		 	</if> 
		 	<!-- ognl会进行字符串与数字的转换判断  "0"==0 -->
		 	<if test="gender==0 or gender==1">
		 	 	and gender=#{gender}
		 	</if>
	 	</where>
</select>
```


## where


- 去除查询条件中多余的第一个 `and` 或 `or`



```xml
<!--
    检查以下sql语句
-->
<select id="getStudent" resultType="com.garcon.dao.StudentMapper">
	 	select * from student
	 	where
		 	<if test="sid!=null">
		 		sid=#{sid}
		 	</if>
		 	<if test="lastName!=null AND lastName!=''">
		 		and last_name like #{lastName}
		 	</if>
</select>
<!--
    可以发现，当以上参数 sid 为空时，sql语句就变成了：
        select * from student where and last_name=?
        多出的 and 语法错误;
    解决以上情况可以加标签<where></where>:
    把所有的<if></if>条件都放在<where></where>标签中，可以去除第一个 and 和 or 如下：
-->        
<select id="getStudent" resultType="com.garcon.dao.StudentMapper">
	 	select * from student
	 	<where>
		 	<if test="sid!=null">
		 		sid=#{sid}
		 	</if>
		 	<if test="lastName!=null AND lastName!=''">
		 		and last_name like #{lastName}
		 	</if>
		 </where>
</select>
```


## trim


```xml
<!--
    检查以下sql语句
-->
<select id="getStudent" resultType="com.garcon.dao.StudentMapper">
	 	select * from student
	 	where
		 	<if test="sid!=null">
		 		sid=#{sid} and
		 	</if>
		 	<if test="lastName!=null AND lastName!=''">
		 		last_name like #{lastName}
		 	</if>
</select>

<!--
    可以发现，当以上参数 lastName 为空时，sql语句就变成了：
        select * from student where sid=? and
        结尾多出的 and 语法错误;
    解决以上情况可以加标签<trim></trim>:
    
        <trim prefix="" suffixOverrides="" suffix="" suffixOverrides=""></trim>
        
        后面多出的and或者or where标签不能解决 
    	 	prefix="":前缀：trim标签体中是整个字符串拼串 后的结果。
    	 			prefix给拼串后的整个字符串加一个前缀 
    	 	prefixOverrides="":
    	 			前缀覆盖： 去掉整个字符串前面多余的字符
    	 	suffix="":后缀
    	 			suffix给拼串后的整个字符串加一个后缀 
    	 	suffixOverrides=""
    	 			后缀覆盖：去掉整个字符串后面多余的字符
    如下：
-->  
<select id="getStudent" resultType="com.garcon.dao.StudentMapper">
	 	select * from student
	 	<trim prefix="where" suffixOverrides="and">
		 	<if test="sid!=null">
		 		sid=#{sid} and
		 	</if>
		 	<if test="lastName!=null AND lastName!=''">
		 		last_name like #{lastName}
		 	</if>
		 </trim>
</select>
```


## choose


```xml
<!--
    使用场景：
        如果参数中的id不为空就以id为条件查询，如果name不为空就已以name做查询条件
        如下：
-->
<select id="getStudent" resultType="com.garcon.dao.StudentMapper">
	 	select * from student 
    <where>
	 	<!-- 如果带了id就用id查，如果带了lastName就用lastName查;只会进入其中一个 -->
        <choose>
	 		<when test="id!=null">
	 				sid=#{sid}
	 		</when>
	 		<when test="lastName!=null">
	 				last_name like #{lastName}
	 		</when>
	 		<when test="hobby!=null">
	 				hobby = #{hobby}
	 		</when>
	 			<!--如果以上条件都不满足，使用如下条件-->
	 		<otherwise>
	 				gender = '女'
            </otherwise>
        </choose>
    </where>
</select>
```


## update-set


如果javaBean中哪个属性带了值就更新那个字段


```xml
<update id="updateStudent">
	 	<!-- Set标签的使用 -->
	 	update student 
		<set>
			<if test="lastName!=null">
				last_name=#{lastName},
			</if>
			<if test="hobby!=null">
				hobby=#{hobby},
			</if>
			<if test="gender!=null">
				gender=#{gender}
			</if>
		</set>
		where id=#{id} 
</update>
```


## update-trim


```xml
<update id="updateStudent">
		update student 
		<trim prefix="set" suffixOverrides=",">
			<if test="lastName!=null">
				last_name=#{lastName},
			</if>
			<if test="email!=null">
				email=#{email},
			</if>
			<if test="gender!=null">
				gender=#{gender}
			</if>
		</trim>
		where id=#{id}
	 </update>
```


## select-foreach


```xml
<select id="getStudent" resultType="com.garcon.bean.Student">
	 	select * from student
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
	 	<foreach collection="ids" item="item_id" separator=","
	 		open="where id in(" close=")">
	 		#{item_id}
	 	</foreach>
	 </select>
```


## insert-foreach


批量插入


```xml
<!--
	 MySQL下： 
	 第一种方法 
	 可以foreach遍历   mysql支持values(),(),()语法-->
	<insert id="addStudents">
	 	insert into student(last_name,gender,hobby,class_id) 
		values
		<foreach collection="students" item="student" separator=",">
			(#{student.lastName},#{student.gender},#{student.hobby},#{student.sclass.cid})
		</foreach>
	 </insert>
	 
<!--
    第二种方法
    这种方式需要数据库连接属性allowMultiQueries=true；
	 	这种分号分隔多个sql可以用于其他的批量操作（删除，修改） -->
    <insert id="addEmps">
	 	<foreach collection="emps" item="emp" separator=";">
	 		insert into student(last_name,gender,hobby,class_id)
	 		values((#{student.lastName},#{student.gender},#{student.hobby},#{student.sclass.cid})
	 	</foreach>
	 </insert> 
	 
	 
	 
<!-- 
    Oracle数据库批量保存： 
	 	Oracle不支持values(),(),()
	 	Oracle支持的批量方式
	 	1、多个insert放在begin - end里面
	 		begin
			    insert into employees(employee_id,last_name,email) 
			    values(employees_seq.nextval,'test_001','test_001@atguigu.com');
			    insert into employees(employee_id,last_name,email) 
			    values(employees_seq.nextval,'test_002','test_002@atguigu.com');
			end;
			
		2、利用中间表：
			insert into employees(employee_id,last_name,email)
		       select employees_seq.nextval,lastName,email from(
		              select 'test_a_01' lastName,'test_a_e01' email from dual
		              union
		              select 'test_a_02' lastName,'test_a_e02' email from dual
		              union
		              select 'test_a_03' lastName,'test_a_e03' email from dual
		       )
		       
		 oracle第一种批量方式
	 -->
	 <insert id="addEmps" databaseId="oracle">
	 	<foreach collection="emps" item="emp" open="begin" close="end;">
	 		insert into employees(employee_id,last_name,email) 
			    values(employees_seq.nextval,#{emp.lastName},#{emp.email});
	 	</foreach>
	 </insert>
	 	
	 	<!-- oracle第二种批量方式  -->
	 <insert id="addEmps" databaseId="oracle">
	 	insert into employees(employee_id,last_name,email)
            <foreach collection="emps" item="emp" separator="union"
	 			open="select employees_seq.nextval,lastName,email from("
	 			close=")">
	 				select #{emp.lastName} lastName,#{emp.email} email from dual
	 		</foreach>
	 </insert>
```


# 获取自增主键的值


方式一 :
使用insert标签的属性`useGeneratedKeys="true" keyProperty="id"`


```xml
<insert id="addUser" useGeneratedKeys="true" keyProperty="id">
        insert into user(username,password,email) values(#{username},#{password},#{email})
</insert>
```


方式二 : 插入时先查询将要插入数据的自增id值


```xml
 <insert id="addUser" useGeneratedKeys="true" keyProperty="id">
        <selectKey order="AFTER" keyProperty="id" resultType="int">
            select last_insert_id()
        </selectKey>
        insert into user(username,password,email) values(#{username},#{password},#{email})
    </insert>
```


# 参数处理


### 单个参数


可以接受基本类型，对象类型，集合类型的值。这种情况 MyBatis可直接使用这个参数，不需要经过任何处理。


### 多个参数


任意多个参数，都会被MyBatis重新包装成一个Map传入。Map的key是param1，param2，0，1…，值就是参数的值。


### 命名参数


为参数使用@Param起一个名字，MyBatis就会将这些参数封装进map中，key就是我们自己指定的名字


### POJO


当这些参数属于我们业务POJO时，我们直接传递POJO


### Map


我们也可以封装多个参数为map，直接传递


**注意 :** 当参数是一个Connection , List , Array数组时, 参数会封装成一个Map , key值为 : connection , list , array .


# 缓存机制


MyBatis 包含一个非常强大的查询缓存特性, 它可以非常方便地配置和定制。缓存可以极大的提升查询效率。


MyBatis系统中默认定义了两级缓存。


## 一级缓存 和 二级缓存。


1. 默认情况下，只有一级缓存（SqlSession级别的缓存, 也称为本地缓存）开启。
1. 二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
1. 为了提高扩展性。MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存



## 一级缓存


-  一级缓存(local cache), 即本地缓存, 作用域默认 为sqlSession。当 Session flush 或 close 后, 该Session 中的所有 Cache 将被清空。 
-  本地缓存不能被关闭, 但可以调用 clearCache()  来清空本地缓存, 或者改变缓存的作用域. 
-  在mybatis3.1之后, 可以配置本地缓存的作用域.  在 mybatis.xml 中配置 



## 一级缓存失效情况


同一次会话期间只要查询过的数据都会保存在当 前SqlSession的一个Map中


• key:hashCode+查询的SqlId+编写的sql查询语句+参数


一级缓存失效的四种情况


1. 不同的SqlSession对应不同的一级缓存
1. 同一个SqlSession但是查询条件不同
1. 同一个SqlSession两次查询期间执行了任何一次增 删改操作
1. 同一个SqlSession两次查询期间手动清空了缓存



## 二级缓存(second level cache)，全局作用域缓存


-  二级缓存默认不开启，需要手动配置 
-  MyBatis提供二级缓存的接口以及实现，缓存实现要求 POJO实现Serializable接口 
-  二级缓存在 SqlSession 关闭或提交之后才会生效 



**使用步骤 **


1. 全局配置文件中开启二级缓存



```xml
 <setting name="cacheEnabled" value="true"/>
```


2. 在需要使用二级缓存的映射文件处使用cache配置缓存



```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
	<!-- <cache eviction="FIFO" flushInterval="60000" readOnly="false" size="1024"></cache> -->
	<!--  
	eviction:缓存的回收策略：
		• LRU – 最近最少使用的：移除最长时间不被使用的对象。
		• FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
		• SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
		• WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
		• 默认的是 LRU。
	flushInterval：缓存刷新间隔
		缓存多长时间清空一次，默认不清空，设置一个毫秒值
	readOnly:是否只读：
		true：只读；mybatis认为所有从缓存中获取数据的操作都是只读操作，不会修改数据。
				 mybatis为了加快获取速度，直接就会将数据在缓存中的引用交给用户。不安全，速度快
		false：非只读：mybatis觉得获取的数据可能会被修改。
				mybatis会利用序列化&反序列的技术克隆一份新的数据给你。安全，速度慢
	size：缓存存放多少元素；
	type=""：指定自定义缓存的全类名；
			实现Cache接口即可；
	-->
```


3. 注意：POJO需要实现Serializable接口



## 缓存相关设置


1. 全局setting的cacheEnable：– 配置二级缓存的开关。一级缓存一直是打开的。
1. select标签的useCache属性： – 配置这个select是否使用二级缓存。一级缓存一直是使用的
1. sql标签的flushCache属性： – 增删改默认flushCache=true。sql执行以后，会同时清空一级和二级缓存。 查询默认flushCache=false。
1. sqlSession.clearCache()： – 只是用来清除一级缓存。
1. 当在某一个作用域 (一级缓存Session/二级缓存 Namespaces) 进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被clear。



## 第三方缓存整合


- EhCache 是一个纯Java的进程内缓存框架，具有快速、精 干等特点，是Hibernate中默认的CacheProvider。
- MyBatis定义了Cache接口方便我们进行自定义扩展。



步骤：


1.  导入ehcache包，以及整合包，日志包 
   1. `ehcache-core-2.6.8.jar`
   1. `mybatis-ehcache-1.0.3.jar`
   1. `slf4j-api-1.6.1.jar`
   1. `slf4j-log4j12-1.6.2.jar`
2.  编写ehcache.xml配置文件 
2.  配置cache标签 



```xml
<ehcache type="org.mybatis.caches.ehcache.EhcacheCache">
```


参照缓存：若想在命名空间中共享相同的缓存配置和实例。 可以使用 cache-ref 元素来引用另外一个缓存。
