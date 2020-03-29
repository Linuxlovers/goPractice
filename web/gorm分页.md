# golang: gorm不定条件查询和分页操作

gorm+gin不定条件查询
不定参数参数多用于table在后端的多条件筛选,这样的场景是无法预知用户需要使用那些筛选条件.只有当参数传递给后端时才清楚.所以后端需要根据传递的参数动态生成符合查询条件的sql语句或者orm操作.

在gorm可以分别任选上述两种中的任意一种:

1.orm操作;
2.拼接原生sql语句;

本文建议大家使用orm操作,以用户结构user为例,假设user的格式如下:

```go
type User struct {
    gorm.Model
    Birthday     time.Time
    Age          int
    Name         string  `gorm:"size:255"`
    Num          int     `gorm:"AUTO_INCREMENT"`
    Sex          string  `gorm:"size:"`
}
```

假设age,name和num是不定查询条件,前端的请求格式如下:

```
http://127.0.0.1:10090/user/?age=26&name=zhangchi1
```

后端逻辑处理如下.

```go
var db *gorm.DB   // 已经进行了db的初始化操作,db为全局变量

func getUsers(c *gin.Context)  {
    users := make([]User, 0)
    Db := db     

    if age, isExist := c.GetQuery("age"); isExist == true {
        ageInt, _ := strconv.Atoi(age)
        Db = Db.Where("age = ?", ageInt)
    }

    if num, isExist := c.GetQuery("num"); isExist == true {
        numInt, _ := strconv.Atoi(num)
        Db = Db.Where("num = ?", numInt)
    }

    if name, isExist := c.GetQuery("name"); isExist == true {
        Db = Db.Where("name = ?", name)
    }

    if err := Db.Find(&users).Error; err != nil {
        fmt.Println(err.Error())
    }

    c.JSON(http.StatusOK, gin.H{
        "data": users,
    })
}
```

请求url后,可以看到在调试模式下sql的执行语句是:

[2018-09-03 18:47:26]  [1.03ms]  SELECT * FROM `users`  WHERE `users`.`deleted_at` IS NULL AND ((age = '26') AND (name = 'zhangchi'))  
[13 rows affected or returned ] 
[GIN] 2018/09/03 - 18:47:26 | 200 |    1.263889ms |       127.0.0.1 | GET      /user/?age=26&name=zhangchi

这里需要注意一个细节,首先将全局的db变量赋值给了Db,如果用db直接进行操作,那一系列的赋值语句将会影响db的地址,影响后续的数据库操作.

Db := db
1
分页操作
分页操作是为了减少前端对后端请求的压力,对于一个系统,为了提高访问效率,不需要每次从后端请求全量的数据,采用分页的方式,获取指定页码的数据,页数(page)和每页的大小(page_size)都可以单独指定.

分页操作和不定条件查询可以同时存在,所以在上述的代码上继续进行累加.组合成一个获取指定条件user列表的接口:


```go
func getUsers(c *gin.Context)  {
    users := make([]User, 0)
    Db := db

    page, _ := strconv.Atoi(c.Query("page"))
    pageSize, _  := strconv.Atoi(c.Query("page_size"))

    if age, isExist := c.GetQuery("age"); isExist == true {
        ageInt, _ := strconv.Atoi(age)
        Db = Db.Where("age = ?", ageInt)
    }

    if num, isExist := c.GetQuery("num"); isExist == true {
        numInt, _ := strconv.Atoi(num)
        Db = Db.Where("num = ?", numInt)
    }

    if name, isExist := c.GetQuery("name"); isExist == true {
        Db = Db.Where("name = ?", name)
    }

    if page > 0 && pageSize > 0 {
        Db = Db.Limit(pageSize).Offset((page - 1) * pageSize)
    }

    if err := Db.Find(&users).Error; err != nil {
        fmt.Println(err.Error())
    }

    c.JSON(http.StatusOK, gin.H{
        "data": users,
    })
}
```

最核心的代码如下:

```go
if page > 0 && pageSize > 0 {
    Db = Db.Limit(pageSize).Offset((page - 1) * pageSize)
}
```



limit定位每页大小, Offset定位偏移的查询位置.并且先进行条件筛选,最后做分页操作.

小结
分页和不定条件查询主要是配合前端的table进行操作,用户可以根据所需的条件进行筛选.为了提高访问效率,可以指定table的每页大小.
