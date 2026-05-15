## PageHelper 分页原理（简单版）

PageHelper 是一个**自动帮你分页**的工具，你不需要手写 SQL 的 `LIMIT`。

## 使用步骤（3步）

### 第1步：导入依赖
```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.6</version>
</dependency>
```

### 第2步：调用 `PageHelper.startPage()`
```java
// 告诉 PageHelper：我要第几页，每页几条
PageHelper.startPage(pageNum, pageSize);  
//                              ↑页码      ↑每页条数
//                           第1页       10条

// 然后正常查询（SQL 里不用写 LIMIT）
List<Article> as = articleMapper.list(userId, categoryId, state);
```

### 第3步：把查询结果转成 Page 对象
```java
// 强转为 Page 对象，里面就有分页信息了
Page<Article> p = (Page<Article>) as;

// 获取分页数据
p.getTotal();   // 总记录数（比如 100 条）
p.getResult();  // 当前页的数据列表
p.getPages();   // 总页数
p.getPageNum(); // 当前页码
```

## 图解原理

```
前端请求：pageNum=2, pageSize=10

PageHelper.startPage(2, 10)
    ↓
拦截你的 SQL：SELECT * FROM article WHERE user_id=1
    ↓
自动加上分页：SELECT * FROM article WHERE user_id=1 LIMIT 10, 10
                                  ↑ 自动添加 ↑ 第2页，跳过前10条
    ↓
同时执行 COUNT 查询：SELECT COUNT(*) FROM article WHERE user_id=1
    ↓
返回 Page 对象 {
    total: 100,       // 总共100条
    result: [...],    // 第11-20条数据
    pages: 10,        // 总共10页
    pageNum: 2        // 当前第2页
}
```

## 你的代码拆解

```java
@Override
public PageBean<Article> list(Integer pageNum, Integer pageSize, 
                               Integer categoryId, String state) {
    // 1. 创建返回对象
    PageBean<Article> pb = new PageBean<>();

    // 2. 开启分页（告诉 PageHelper：我要第 pageNum 页，每页 pageSize 条）
    PageHelper.startPage(pageNum, pageSize);
    //                  ↑ 比如：1     ↑ 比如：10

    // 3. 正常查询（不需要在 SQL 里写 LIMIT）
    Map<String, Object> map = ThreadLocalUtil.get();
    Integer userId = (Integer) map.get("id");
    List<Article> as = articleMapper.list(userId, categoryId, state);
    
    // 4. 把查询结果转成 Page 对象
    Page<Article> p = (Page<Article>) as;
    // Page 是 List 的子类，所以可以强转

    // 5. 取出分页信息，塞进 PageBean
    pb.setTotal(p.getTotal());    // 总记录数，比如 50
    pb.setItems(p.getResult());   // 当前页数据，10条

    return pb;  // 返回给前端
}
```

## 你的 Mapper 怎么写

```java
// ArticleMapper.java
// 注意：SQL 里不要写 LIMIT，PageHelper 会自动加
@Select("SELECT * FROM article WHERE create_user = #{userId}")
List<Article> list(Integer userId, Integer categoryId, String state);
```

或者在 XML 里：
```xml
<!-- ArticleMapper.xml -->
<select id="list" resultType="Article">
    SELECT * FROM article
    WHERE create_user = #{userId}
    <if test="categoryId != null">
        AND category_id = #{categoryId}
    </if>
    <if test="state != null and state != ''">
        AND state = #{state}
    </if>
    ORDER BY create_time DESC
    <!-- 不需要写 LIMIT #{pageNum}, #{pageSize} -->
</select>
```

## 前端对应关系

```typescript
// 前端传参
const params = {
    pageNum: 1,      // ← 对应 PageHelper.startPage(1, 10)
    pageSize: 10,    // ← 每页10条
    state: '已发布',
    categoryId: 1
}

// 后端返回
result.data = {
    total: 50,       // ← p.getTotal()
    items: [...]     // ← p.getResult()，10条数据
}
```

**核心就一句话：你只管调用 `PageHelper.startPage(页码, 每页条数)`，然后正常查询，它自动帮你分页。**