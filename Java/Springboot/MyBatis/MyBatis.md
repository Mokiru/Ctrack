# 依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus</artifactId>
    <version>3.4.3</version>
    <scope>compile</scope>
</dependency>
```

# xml模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="Mapper接口路径">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="实体类路径">
        <id column="id" property="id" />
        <result column="数据表中列名" property="实体类中对应字段名" />
        <!--....-->
    </resultMap>

</mapper>
```

Mapper接口应`extends BaseMapper<实体类>`

# association(一对一查询)

我们可以在结果映射中添加`<association />`标签。

例如：
```xml
<association property="实体属性名称" javaType="类类型">
<!-- -->
</association>
```

上面这种我们可以在`<resultMap>`的实体类中添加一个字段来接收这个参数，为一对一，也就是说这个一可以是任意自定义类型，无论是Admin，Student....都可以。或者更像SQL查询时使用LEFT OUTER JOIN或者说使用了什么COUNT()这种函数，新增了一列或者多列，但是是一对一的类型，那么就可以使用这个，来获取额外的列。

那如果我不想使用这些，但是我想根据这个表的某个字段去另一个表中查找然后合并到这里面呢？例如：我有两个表，分别是文章表，和点赞表，我需要拿着文章id去点赞表中查询这个文章id的点赞数量
```xml
<association property="likes" javaType="java.lang.Long" column="id" select="getCount" />
```

其中`likes`是点赞数，是文章实体类的一个字段，类型是`Long`类型，`column`表示用于匹配的列，`select`表示获取该数据的子查询的名称（也就是说获取点赞数量我得再写一个`<select id="getCount" resultType="java.lang.Long"> </select>`这样去获取点赞数量）。

# collection(一对多查询)

```xml
<collection property="实体字段" ofType="集合单个元素类型">
<!-- 结果映射 -->
</collection>
```

上面这种可以应对简单的连接查询例如直接使用外连接，如一个班级可以有多个学生，我如何查询班级信息(`class_info`)的同时能通过班级的`StudentList`字段拿到班级的学生(`student`)列表呢？显然SQL语句很好写`SELECT * FROM class_info ci, student s WHERE ci.id=s.class_id` 就可以获得对应的关系了，但是我们如何获取到这些数据呢？首先在`class_info`需要有`StudentList`这个字段，当然这个字段名称可以自定义，这里为了统一如此命名。
```xml
<collection property="StudentList" ofType=".....student学生的实体类类型">
<!-- 学生的结果集映射 -->
</collection>
```

如此我们可以通过返回的`class_info`或者班级信息列表中，可以查看`StudentList`的数据已经有了对应班级学生的所有信息了。

同样，如果有些地方不能用到连接查询，就得分成不同的子查询进行结果拼接呢？例如如果一个表的存储是一个树形结构，有`parent_id`字段标志当前数据的父节点的`id`是多少，我们该如何将其按照树形结构进行显示呢？这就需要用到类似`association`里面的方法了：
```xml
<collection property="" column="" ofType="" select="">

</collection>
```

显然我们可以知道这个和`association`的区别就是一对一和一对多了，这个可以返回一个集合，而`association`只能返回一个`Object`。

那么上述问题，我们假设只有`id`和`parent_id`.
```xml
<resultMap id="testMap" type="实体类名">
    <id column="id" property="id" />
    <result column="parent_id" property="parentId" />
    <collection property="child" column="id" ofType="实体类名（可以不同）" select="getTreeNode">
        <id column="id" property="id" />
        <result column="parent_id" property="parentId" />
    </collection>
</resultMap>

<select id="getTreeNode" resultType="实体类名">
    SELECT * FROM a WHERE parent_id=#{id}
</select>

<select id="get" resultMap="testMap">
    SELECT * FROM a
</select>
```
