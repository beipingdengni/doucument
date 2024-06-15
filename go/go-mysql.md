## GO MYSQL

### 安装

```bash
go get -u github.com/go-sql-driver/mysql
```

### 连接

```go
_ import "github.com/go-sql-driver/mysql"  // 执行mysql包下的所有init()函数
// 全局常量
var (
	dbConn *sql.DB
  err error
)
// init 方法是 go 语言中最先执行
// 初始化连接
func init(){
   dbConn,err = sql.Open("mysql","root:123456@tcp(127.0.0。1:3306)/demo?charset=utf8")
    if err != nil{
		fmt.Println(err)
	}
    // 设置数据库最大连接数
	dbConn.SetConnMaxLifetime(100)
    // 设置上数据库最大闲置连接数
	DB.SetMaxIdleConns(10)
    // 验证连接
	if err := DB.Ping(); err != nil {
		fmt.Println("open database fail")
		return nil
	}
}
解释
// sql.Open()：中的数据库连接串格式为："用户名:密码@tcp(IP:端口)/数据库?charset=utf8"。Open函数可能只是验证其参数格式是否正确，实际上并不创建与数据库的连接。如果要检查数据源的名称是否真实有效，应该调用Ping方法。返回的DB对象可以安全地被多个goroutine并发使用，并且维护其自己的空闲连接池。因此，Open函数应该仅被调用一次，很少需要关闭这个DB对象。
// SetMaxOpenConns：设置与数据库建立连接的最大数目。 如果n大于0且小于最大闲置连接数，会将最大闲置连接数减小到匹配最大开启连接数的限制。 如果n<=0，不会限制最大开启连接数，默认为0（无限制）。
// SetMaxIdleConns：设置连接池中的最大闲置连接数。 如果n大于最大开启连接数，则新的最大闲置连接数会减小到匹配最大开启连接数的限制。 如果n<=0，不会保留闲置连接。
//DB的类型为:*sql.DB，有了DB之后我们就可以执行CRUD操作。Go将数据库操作分为两类：Query与Exec。两者的区别在于前者会返回结果，而后者不会。
//	1. Query表示查询，它会从数据库获取查询结果（一系列行，可能为空）。
//	2. Exec表示执行语句，它不会返回行。
// 此外还有两种常见的数据库操作模式：
//	1. QueryRow表示只返回一行的查询，作为Query的一个常见特例。
//	2. Prepare表示准备一个需要多次使用的语句，供后续执行用。
// 使用完毕，记得释放数据库连接
// defer DB.Close()
```

### 插入、更新、删除

> func (db *DB) Exec(query string, args ...interface{}) (Result, error)

```go
// 添加
func AddComment(videoId string, authorId int, commentContent string) error {
	commentId := uuid.New().String()
    // 防止sql注入
	stmIns, err := dbConn.Prepare("INSERT INTO comments (id,video_id,author_id,content) VALUES (?,?,?,?)")
	if err != nil {
		return err
	}
	_, err = stmIns.Exec(commentId, videoId, authorId, commentContent)
	if err != nil {
		return err
	}
	defer stmIns.Close()
	return nil
}
// 更新、删除
func AddComment(videoId string,content string) error {
	commentId := uuid.New().String()
    // 防止sql注入
	stmIns, err := dbConn.Prepare("update comments set id=? where id=?")
	if err != nil {
		return err
	}
	_, err = stmIns.Exec(videoId,content)
	if err != nil {
		return err
	}
    n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return err
	}
	defer stmIns.Close()
	return nil
}
```

#### 事务相关方法处理

```go
// 开始事务
func (db *DB) Begin() (*Tx, error)
//提交事务
func (tx *Tx) Commit() error
//回滚事务
func (tx *Tx) Rollback() error

// 事务操作示例
func transactionDemo() {
	tx, err := DB.Begin() // 开启事务
	if err != nil {
		if tx != nil {
			tx.Rollback() // 回滚
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}
	sqlStr1 := "Update college set name='呜呜呜' where id=?"
	ret1, err := tx.Exec(sqlStr1, 2)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql1 failed, err:%v\n", err)
		return
	}
	affRow1, err := ret1.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}
	sqlStr2 := "Update college set name='哈哈哈' where id=?"
	ret2, err := tx.Exec(sqlStr2, 3)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql2 failed, err:%v\n", err)
		return
	}
	affRow2, err := ret2.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}
	fmt.Println(affRow1, affRow2) // 打印
	if affRow1 == 1 && affRow2 == 1 {
		fmt.Println("事务提交啦...")
		tx.Commit() // 提交事务
	} else {
		tx.Rollback()
		fmt.Println("事务回滚啦...")
	}
	fmt.Println("exec trans success!")
}
```



### 查询

> 单行结果：func (db *DB) QueryRow(query string, args ...interface{}) *Row
>
> 多行结果：func (db *DB) Query(query string, args ...interface{}) (*Rows, error)

#### 查询结果

```go
// 查询单行数据
func selectOneRow()(string,error){
    // 防止sql注入
    stmtins,err :=dbconn.Prepare("select * from user where user_name =?")
    if err != nil{
        return "", err
    }
  	// 自动映射结果
    var password string
    err = dbconn.QueryRow(user_name).Scan(&password)
    // 判断出错或者没有查到数据
    if err != nil && err != sql.ErrNoRows{
        return "",err
    }
    // 关闭数据库连接
    defer stmtins.Close()
    return password,nil
}
```

#### 多行结果

```go
type Comment struct {
	Id string
	VideoId string
	UserName string
	Content string
}
// 查询多行数据
func ListComments(videoId string)(*data.Comment,error){
    // 防止sql注入
    stmtout,err := dbConn.Prepare("select * from comments")
    var commentsArr []*data.comment
    if err != nil && err != sql.ErrNoRows{
        return comment,err
    }
    // 循环结构并赋值,然后append到数组中
    for rows.Next{
        var id,user_name,content string
        if err := rows.Scan(&id,&user_name,&content);err != nil{
            return commentsArr,err
        }
        comment := &data.comment{
            id,
            videoId,
            user_name,
            content
        }
        commentsArr = append(commentsArr,comment)
    }
    defer rows.Close()
    return commentsArr,nil
}
```

#### 查询返回-数组map

```go
// 使用以下定义函数查询
args := []interface{}{} //sql格式化参数
resultNames := []string{"nick_name", "c_time"} //查询结果字段名称  传入Database.Query时的顺序必须要和sql查询字段顺序一致
res, err := dbMap.Query(db, "select nick_name,c_time from sys_account", args, resultNames)

// 定义通用查询函数
func Query(db *DB, Sql string, args []interface{}, dest []string) ([]map[string]string, error) {
	/*
		数据库查询入口 只能返回map[string]string类型结果集
		@params
      Sql: 被执行的sql
      args: Sql的参数  没有参数传空interface{}数组 顺序和类型一定要和sql中一致
      dest: 返回结果集map中key的名称
		@result
			[]map[string]string: 查询结果map对象数组
			error: 错误信息,有错误返回错误信息   没有错误返回nil
	*/
	//定义返回的结果集
	var results []map[string]string
	//定义接收查询结果切片 interface{}类型
  // 不过来数据全部返回使用此方法
  // cols, _ := rows2.Columns()
  // destP := make([]interface{}, len(cols))
	destP := make([]interface{}, len(dest))
	//把每个结果初始化为*string
	for i := range dest {
		destP[i] = new(string)
	}
	//调用DB查询 得到游标rows
	rows, err := db.Query(Sql, args...)
	if err != nil {
		return results, err
	}
	//循环取出所有行
	for rows.Next() {
		//定义存储单行查询结果的map对象
		result := make(map[string]string)
		//取出当前行的查询结果到destP中
		err = rows.Scan(destP...)
		if err != nil {
			return results, err
		}
		//循环把destP中的结果按照dest的key存储到result中
		for i := range dest {
			//destP[i]中存储的是interface{}类型的string指针
			//destP[i].(*string)是string指针值
			//*destP[i].(*string)是string的值
			result[dest[i]] = *destP[i].(*string)
		}
		//把单行查询结果添加到results中
		results = append(results, result)
	}
	return results, err
}
```

#### 常用-查询返回-map(包含全部数据)

```go
func selects() {
    db, err := sql.Open("mysql", "root:123456@tcp(127.0.0.1:3306)/test?charset=utf8&parseTime=True&loc=Local")
    // 验证连接
    if err := DB.Ping(); err != nil {
      fmt.Println("open database fail")
      return nil
    }
    // 查询数据
    //查询数据，取所有字段
    rows2, _ := db.Query("SELECT * FROM people")
    //返回所有列
    cols, _ := rows2.Columns()
    //这里表示一行所有列的值，用[]byte表示
    vals := make([][]byte, len(cols))
    //这里表示一行填充数据
    scans := make([]interface{}, len(cols))
    //这里scans引用vals，把数据填充到[]byte里
    for k, _ := range vals {
        scans[k] = &vals[k]
    }
    i := 0
    result := make(map[int]map[string]string)
    for rows2.Next() {
        //填充数据
        rows2.Scan(scans...)
        //每行数据
        row := make(map[string]string)
        //把vals中的数据复制到row中
        for k, v := range vals {
            key := cols[k]
            fmt.Printf(string(v))
            //这里把[]byte数据转成string
            row[key] = string(v)
        }
        //放入结果集
        result[i] = row
        i++
    }
    //fmt.Println(result)
    for k, v := range result {
        fmt.Printf("第%d行", k)
        fmt.Println(v["id"] + "===>" + v["first_name"])
    }
    db.Close()
}

// 未尝试过，可以验证，这种方式更简单
//  var results []map[string]interface{}
//  for rows.Next() {
//        result = make(map[string]interface{})
//        err = rows.MapScan(result)
//        if err != nil {
//            log.Fatal(err)
//        }
//        results = append(results, result)
//    }
//    jsonResults, _ := json.Marshal(results)
//    fmt.Println(string(jsonResults))
```

