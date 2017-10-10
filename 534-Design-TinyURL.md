# TinyUrl   设计方案

## Questions
1. 所有的短地址数`(26+26+10)^6=56800235584` 个
2. 标识符可以是自增的，将标识符可以设计成一个62进制数
 `0-9` 对应 `0-9`
  `10-35` 对应 `a-z`
  `36-61` 对应 `A-Z`




## 标识符的计算

使用hash 方式难以避免冲突

使用标识符自增方式来生成标识符，能够避免冲突，并且六位的 62 进制数可以容纳 500 多亿条url


## 存储

使用 mysql 存储
表设计如下：

字段名   | 类型  
------  | ----  
id      | bigint
url     | varchar(255)


Sql:

```sql
CREATE TABLE urls  (
  `id` bigint(0) UNSIGNED NOT NULL AUTO_INCREMENT,
  `url` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
  INDEX `long2short`(`url`)
);
```

`long2short`索引用于创建新短地址时是判断此源地址是否已经存在。


## 缓存使用 

key 设计:	`u:{id}`
value ：`{url}`
过期时间：`7天`

由于一般新生成的短连接访问较高，几天后访问会回落，所以过期时间设置为 7 天


## 相关算法

源地址插入到mysql 中后能够得到行记录`id`, 通过将`id`转换为 62 进制数来获取到 url 的端链接标识符

Golang 代码：

```go
func insert(s []rune, index int, char rune) []rune {
    rear := append([]rune{}, s[index:]...)
    s = append(s[0:index], char)
    s = append(s, rear... )
    return s
}

func base10ToBase62(n int) string {
    var elements []rune = []rune("0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ")
    var s = make([]rune, 0, 6)
    for ; n!=0 ;  {
        s = insert(s, 0, elements[n%62])
        n /= 62
    }

    for ;len(s)!=6 ;  {
        s = insert(s, 0, '0')
    }

    return string(s)
}

func main(){
    fmt.Println(base10ToBase62(12))
} 
```
 


## 如何做水平扩展

可以根据通过对10进制标识符取模将短链接散列到不同的机器上例如

	id%62
各个机器上不共享 id，url 设计为

	/id%62/62进制标识符


## 可用性
