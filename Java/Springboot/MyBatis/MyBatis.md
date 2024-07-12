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

# 实例

第一个：

```java
@Data
public class Comment {
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    @TableField(value = "create_time")
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;

    @TableField("content")
    private String content;

    @TableField("user_id")
    private Long userId;

    @TableField("blog_id")
    private Long blogId;

    @TableField("isDelete")
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY) // 禁止 序列化
    private int isDelete;

    @TableField("parent_id")
    private Long parentId;

    @TableField(exist = false)
    private List<Comment> childComment;
}
```

```xml
<resultMap id="testMap" type="com.hoshino.momochi.model.domain.Comment">
    <id column="id" property="id"/>
    <result column="create_time" property="createTime"/>
    <result column="content" property="content"/>
    <result column="user_id" property="userId"/>
    <result column="blog_id" property="blogId"/>
    <result column="isDelete" property="isDelete"/>
    <result column="parent_id" property="parentId"/>
    <collection property="childComment" column="id" ofType="com.hoshino.momochi.model.domain.Comment"
                select="nextTreeNode">
        <id column="id" property="id"/>
        <result column="create_time" property="createTime"/>
        <result column="content" property="content"/>
        <result column="user_id" property="userId"/>
        <result column="blog_id" property="blogId"/>
        <result column="isDelete" property="isDelete"/>
        <result column="parent_id" property="parentId"/>
    </collection>
</resultMap>
```

第二个：收藏夹有收藏的文章列表，也有子收藏夹列表，同样子收藏夹也有一样的结构。

```java
@Data
public class CollectionBlog {
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    @TableField(value = "create_time")
    @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;

    @TableField("collection_name")
    private String collectionName;

    @TableField("user_id")
    private Long userId;

    @TableField("parent_id")
    private Long parentId;

    @TableField("isDelete")
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY) // 禁止 序列化
    private int isDelete;

    @TableField(exist = false)
    List<Blog> blogList;

    @TableField(exist = false)
    List<CollectionBlog> childList;
}
```

```xml
<resultMap id="TestAll" type="com.hoshino.momochi.model.domain.CollectionBlog">
    <id column="id" property="id"/>
    <result column="create_time" property="createTime"/>
    <result column="user_id" property="userId"/>
    <result column="collection_name" property="collectionName"/>
    <result column="parent_id" property="parentId"/>
    <result column="isDelete" property="isDelete"/>
    <collection property="blogList" ofType="com.hoshino.momochi.model.domain.Blog">
        <id column="bid" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="create_time" property="createTime"/>
        <result column="last_update_time" property="lastUpdateTime"/>
        <result column="blog_name" property="blogName"/>
        <result column="brief" property="brief"/>
        <result column="isDelete" property="isDelete"/>
    </collection>
    <collection property="childList" column="id" ofType="com.hoshino.momochi.model.domain.CollectionBlog"
                select="nextTreeNode">
        <id column="id" property="id"/>
        <result column="create_time" property="createTime"/>
        <result column="user_id" property="userId"/>
        <result column="collection_name" property="collectionName"/>
        <result column="parent_id" property="parentId"/>
        <result column="isDelete" property="isDelete"/>
    </collection>
</resultMap>
<select id="nextTreeNode" resultMap="TestAll">
    SELECT *, b.id AS bid
    FROM collection_blog c,
        blog b
    WHERE c.parent_id = #{id}
    AND b.id IN (SELECT blog_id
                    FROM collection_blog c,
                        blog_star b
                    WHERE c.id = b.collection_id
                    AND c.parent_id = #{id})
    AND c.isDelete = 0
</select>

<select id="findTreeNode" resultMap="TestAll">
    SELECT *, b.id AS bid
    FROM collection_blog c,
        blog b
    WHERE c.id = #{id}
      AND b.id IN (SELECT blog_id
                   FROM collection_blog c,
                        blog_star b
                   WHERE c.id = b.collection_id
                     AND c.id = #{id})
      AND c.isDelete = 0
</select>
```

下面这个版本则是不用连接，而单独用两个子查询达到目的。

```xml
<resultMap id="HOSHINO" type="com.hoshino.momochi.model.domain.CollectionBlog">
    <id column="id" property="id"/>
    <result column="create_time" property="createTime"/>
    <result column="user_id" property="userId"/>
    <result column="collection_name" property="collectionName"/>
    <result column="parent_id" property="parentId"/>
    <result column="isDelete" property="isDelete"/>
    <collection property="blogList" column="id" ofType="com.hoshino.momochi.model.domain.Blog"
                select="takana">
        <id column="bid" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="create_time" property="createTime"/>
        <result column="last_update_time" property="lastUpdateTime"/>
        <result column="blog_name" property="blogName"/>
        <result column="brief" property="brief"/>
        <result column="isDelete" property="isDelete"/>
    </collection>
    <collection property="childList" column="id" ofType="com.hoshino.momochi.model.domain.CollectionBlog"
                select="nakasha">
        <id column="id" property="id"/>
        <result column="create_time" property="createTime"/>
        <result column="user_id" property="userId"/>
        <result column="collection_name" property="collectionName"/>
        <result column="parent_id" property="parentId"/>
        <result column="isDelete" property="isDelete"/>
    </collection>
</resultMap>
<resultMap id="blogMap" type="com.hoshino.momochi.model.domain.Blog">
    <id column="bid" property="id"/>
    <result column="user_id" property="userId"/>
    <result column="create_time" property="createTime"/>
    <result column="last_update_time" property="lastUpdateTime"/>
    <result column="blog_name" property="blogName"/>
    <result column="brief" property="brief"/>
    <result column="isDelete" property="isDelete"/>
</resultMap>
<!--  List  -->
<select id="nakasha" resultMap="HOSHINO">
    SELECT *
    FROM collection_blog
    WHERE parent_id = #{id}
      AND isDelete = 0
</select>

<select id="takana" resultMap="blogMap">
    SELECT *, b.id AS bid
    FROM blog_star bs,
         blog b
    WHERE bs.collection_id = #{id}
      AND bs.blog_id = b.id
      AND bs.isDelete = 0
      AND b.isDelete = 0
</select>

<select id="findAllByUserId" resultMap="HOSHINO">
    SELECT *
    FROM collection_blog
    WHERE user_id = #{userId}
      AND parent_id = 0
      AND isDelete = 0
</select>
```
