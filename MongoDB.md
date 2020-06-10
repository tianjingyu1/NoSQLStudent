# 文档数据库-适用案例

事件记录
内容管理系统及博客平台
网站分析与实时分析
电子商务应用程序

# 文档数据库-不适用场合

包含多项操作的复杂事务

# MongoDB主要特点

面向集合存储：易存储对象类型的数据，包括文档内嵌对象及数组

模式自由：无需知道存储数据的任何结构定义，支持动态查询、完全索引，可轻易查询文档中内嵌的对象和数组

文档型：存储在集合中的文档，被存储为键-值对的形式。键用于唯一标识一个文档，为字符串类型；值则可是各种复杂的文件类型

高效的数据存储：支持二进制数据及大型对象

自动分片：以支持云级别的伸缩性，支持水平的数据库集群，可动态添加额外的服务器

多个存储引擎的支持：基于硬盘读写的存储引擎(WiredTiger)和基于内存的存储引擎(In-Memory）

# 数据对象被存储为集合中的文档

文档

文档表示单个实体的数据
文档可包含嵌入的子文档
表示文档的记录以BSON的方式存储

集合
MongoDB使用集合将数据编组。
集合是一组用途相同或类似的文档。
集合不受严格模式的管制。

MongoDB每个文档的key可以不一样

TRDB一个表中所有记录每个字段的数据格式在表定义初期就已固定

1. 字段名_id保留用于存储对象_id。
2. 字段名_id包含系统中独一无的ID。
3. ID由下列几部分组成：
从新纪元开始的秒数（4字节）；
3字节的机器标识符；
2字节的进程ID;
3字节的计数器（该计算器的起始值是随机的）。

# MongoDB数据类型

以二进制JSON对象的方式存储
JSON(JavaScript Object Notation)：JavaScript 对象表示
JSON是一种简单的数据表示方式，它易于理解、易于解析、易于记忆。
从另一方面来说，因为只有null、布尔、数字、字符串、数组和对象这几种数据类型，所以JSON有一定局限性。
例如，JSON没有日期类型，JSON只有一种数字类型，无法区分浮点数和整数、32位和64位数字。
JSON无法表示其他一些通用类型，如正则表达式或函数。

BSON是一种类JSON的二进制形式的存储格式，简称Binary JSON。
同JSON一样，支持内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date类型。
它支持下面数据类型。每个数据类型对应一个数字，在MongoDB中可以使用$type操作符查看相应的文档的BSON类型。

# 基本数据类型

null：用于表示空值或者不存在的字段，{“x”:null}
布尔型：布尔类型有两个值true和false，{“x”:true}
数值：shell默认使用64为浮点型数值。{“x”：3.14}或{“x”：3}。对于整型值，可以使用NumberInt（4字节符号整数）或NumberLong（8字节符号整数），{“x”:NumberInt(“3”)}{“x”:NumberLong(“3”)}
字符串：UTF-8字符串都可以表示为字符串类型的数据，{“x”：“呵呵”}
日期：日期被存储为自新纪元以来经过的毫秒数，不存储时区，{“x”:new Date()}
正则表达式：查询时，使用正则表达式作为限定条件，语法与JavaScript的正则表达式相同，{“x”:/[abc]/}
数组：数据列表或数据集可以表示为数组，{“x”： [“a“,“b”,”c”]}
内嵌文档：文档可以嵌套其他文档，被嵌套的文档作为值来处理，{“x”:{“y”:3 }}
对象Id：对象Id是一个12字节的字符串，是文档的唯一标识，{“x”: objectId() }
二进制数据：二进制数据是一个任意字节的字符串。它不能直接在shell中使用。
代码：查询文档中可以包括任何JavaScript代码，{“x”:function( ){/*…*/}}

# 规划数据模型

## 使用文档引用范式化数据

数据范式化
找出这样的对象属性，属性成为子对象，应作为一个独立的文档存储在对象文档中的不同集合中。
通过组织文档和集合以最大限度减少冗余和依赖。

### 优点      
减少数据库规模       独立的集合中存储子对象的拷贝，不是重复存储
修改操作   如果需要频繁修改子对象中的信息，将只需在一个地方修改，无需在包含它的每个对象中进行修改。
### 缺点
查找操作     查询对象时如果需要返回子对象，如果需要频繁访问这些对象，影响性能。

## 使用嵌入式文档对数据反范式化

指将子对象直接嵌入到主对象文档中的。

适用情形：主对象和子对象之间为一对一关系或者子对象很少且不会频繁更新。

### 优点：
只需一次查找即可获得整个对象。无需在其他集合中查找子对象。可极大地改善性能。

### 缺点：
一对多关系  存储多个拷贝。这将稍微降低插入速度，还将占用更多的磁盘空间。

## 使用固定集合

MongoDB能够创建大小固定的集合。
如果需要写入新文档，将删除最旧文档，在插入新文档。

优点：
按顺序排列文档
禁止执行导致文档增大的更新
自动删除集合中最旧文档

## 理解原子写入操作

写入操作在文档级是原子性的。不可能多个进程同时更新一个文档或集合。

反范式化文档的写入是原子性的。

写入范式化文档时，需要对其他集合中的子文档执行独立的写入操作，因此对范式化文档的写入可能不是原子性的。

## 考虑文档增大

更新文档时，必须考虑其在文档增大方面的影响。如果更新导致文档增大到超过了分配给它的磁盘空间，MongoDB就必须将文档移到磁盘其他位置，这将影响系统性能。频繁移动文档还可能会导致磁盘碎片问题。

缓解文档增大问题的方式之一：
对于频繁增长的属性，将其设计为范式化对象。
例如，不使用数组存储Cart对象中的商品，创建一个用于存储商品的CartItems集合。将加入到购物车的商品作为新对象存储到集合CartItems中，并在Cart对象中引用这些商品。

## 找出可使用索引、分片和复制的情形

索引    查询效率  自动创建基于_id字段的索引。

分片    拆分大型数据集合，放到集群的多个MongoDB服务器中。考虑数据量以及访问数据的请求数，以确定是否要对集合进行分片以及使用多少个分片。

复制   多个MongoDB实例，确保重要数据的备份拷贝。

## 使用大型集合还是大量集合

存在大量集合不会严重影响性能，但一个集合中包含大量数据会严重影响性能。

## 确定数据的生命周期

文档在集合在存储多久的问题
文档存活周期TTL （Time  to Live）

MongoDB中实现TTL机制两种方式
一种方法时在应用程序中对旧数据进行监视和清理的代码；
另一种方法是对集合设置TTL,指定多少秒后或到达指定时间后自动将文档删除。

## 考虑数据可用性和性能

数据可用性指的是数据库能够满足用户的功能需求。
     （网站能够被访问、数据的准确性）

数据性能要求指的是数据库必须以能够接受的速度提供数据。

## 开启或关闭MongoDB服务
用net stop MongoDB  关闭MongoDB服务：
用net start MongoDB  开启MongoDB服务：

### 代码 
启动MongoDB时指定了参数port和dbpath
mongod  -port 28008  -dbpath <mongo_install_location>/data/db

查看帮助：mongod –help

启动MongoDB服务之前需要必须创建数据库文件的存放文件夹，否则命令不会自动创建，而且不能启动成功。
在安装目录\data\下创建一个db目录

dbpath= F:\Program Files\MongoDB\Server\4.0\data\db
logpath=F:\Program Files\MongoDB\Server\4.0\data\log\mongo.log
logappend=true
日志采用追加模式，配置这个选项后，mongodb的日志会追加到现有的日志文件，而不是从新创建一个新文件。
journal=true    
启用日志文件，默认启用
port=27017
端口号     默认为27017

用管理员身份打开cmd，这时，启动MongoDB服务器用  -config，输入命令：
mongod -config "F:\Program Files\MongoDB\Server\4.0\mongo.config"

停止MongoDB
use   admin
db.shutdownServer( )

启动MongoDB shell命令： mongo

完整的登录命令
在命令行输入：
mongo 127.0.0.1:27017/admin

退出MongoDB  shell的命令
在命令行输入：exit

数据库实例操作
创建数据库 use 数据库名
若数据库存在，就连接它；若数据库不存在，就创建它
若只创建数据库，而不作任何操作，空数据库将被删除     
查看数据库 show dbs   
查看当前数据库中的集合列表  show  collections    
统计某数据库信息 db.stats()
删除某数据库 db.dropDatabase()
帮助指令：db.help()、db.Collection_name.help()、db.Collection_name.find().help()
查看当前数据库下集合名称 db.getCollectionNames()
查看数据库用户角色权限 show roles

MongoDB shell基于JavaScript，可创建脚本，脚本可运行多次。
脚本编程通过三种方式实现
使用命令行选项--eval执行JavaScript表达式
在MongoDB shell中使用方法load()来执行脚本
在命令mongo中指定要执行的JavaScript文件　

命令行选项--eval执行JavaScript表达式
参数--eval接受一个JavaScript字符串或JavaScript文件
mongo  DataBase_name  --eval “printjson(db.getCollectionNames())”

在命令行使用mongo执行JavaScript文件
执行脚本文件：mongo   shell_script.js

使用方法load(script_path)
如：MongoDB shell加载并执行脚本文件text.js：
load(“F:/chap04/generate_words.js”)

## 理解Admin数据库

MongoDB数据库默认是没有用户名及密码，不用安全验证的，只要连接上服务就可以进行CRUD操作。
在刚安装完毕的时候MongoDB都默认有一个admin数据库，而admin.system.users中将会保存比在其它数据库中设置的用户权限更大的用户信息。 

当admin.system.users中一个用户都没有时，即使mongod启动时添加了- -auth参数，如果没有在admin数据库中添加用户，此时不进行任何认证还是可以做任何操作，直到在admin.system.users中添加了一个用户。

system.user用户  每个数据库的用户账号都是以文档形式存储在system.users集合里面。

1、用户概念
MongoDB的用户是由 用户名+所属数据库名组成
例如:
     登录mongo  testdb1 ，创建用户testuser1

MongoDB的授权采用了角色授权的方法，每个角色包括一组权限。

MongoDB已经定义好了的角色叫内建角色，我们也可以自定义角色。

主要介绍内建角色，MongoDB内建角色包括下面几类：

 1）数据库用户角色：read、readWrite;
 2）数据库管理角色：dbAdmin、dbOwner、userAdmin；
 3）集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
 4）备份恢复角色：backup、restore；
 5） 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
 6）超级用户角色：root     // 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
 7） 内部角色：_system

read: 允许用户读取指定数据库 
readWrite: 允许用户读写指定数据库 
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile 
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户 
clusterAdmin：只在admin数据库中可用，具有所有分片和复制集相关函数的管理权限。 
readAnyDatabase：只在admin数据库中可用，具有所有数据库的读权限 
readWriteAnyDatabase：只在admin数据库中可用，具有所有数据库的读写权限 
userAdminAnyDatabase：只在admin数据库中可用，具有所有数据库的userAdmin权限 
dbAdminAnyDatabase：只在admin数据库中可用，具有所有数据库的dbAdmin权限。 
root：只在admin数据库中可用。超级账号，超级权限

### 搭建认证环境和认证登录

1、找到MongoDB配置文件,设置noauth=true
重启MongoDB后，登录admin账号，创建一个超级权限用户。
use admin
db.createUser(
    {user:'root',
     pwd:'root',
     roles:[{ "role" : "root", "db" : "admin" }]
     }
 );

2、关闭MongoDB

3、启用认证参数
     要保证权限认证生效，需要在MongoDB配置文件中加入auth=true，同时取消掉noauth=true

4、启动MongoDB

5、认证登录
```
> use adminswitched to db admin
> db.auth('root','root')
```
也可在启动MongoDB   shell时使用选项向数据库admin验证身份。
  mongo  admin –u  root  –p root

### 用户授权详解

1、创建用户并授权
   语法：
db.createUser(
     {   user:"UserName",
         pwd:"Password",
         roles:[{role:"RoleName",db:"Target_DBName"}]
    } 
 )
首先选择在哪个库创建用户，如test：use test； 创建用户有3项需要提供：用户名，密码，角色列表。

创建一个超级账号
use admin
db.createUser(
  {
    user:"root",
	 pwd:"root",
	 roles:[{role:"root",db:"admin"}]
)

注意：1.在哪个数据库创建的用户，在哪个数据库删除，且删除必须有权限。2.在哪个数据库创建的用户需在哪个数据库认证。

查询某个数据库下的用户db.system.users.find(); 
删除指定用户
    db.dropUser(“用户名”);
需要先切换到该用户所在的数据库。

MongoDB提供在数据库级别上的验证和授权，意味着用户存在于单个数据库的上下文中。

为了实现基本的身份验证，MongoDB把用户凭据存储在每个数据库名为system.users的集合中。

设置新的MongoDB实例的第一步是创建用户管理员和数据库管理员账户。

用户管理员具有在admin和其他数据库中创建用户账户的功能。
数据库管理员账户用来管理数据库、集群、复制和MongoDB的其他方面。

1. 创建用户管理员
用户管理员账号应只有创建用户的权限，而没有管理数据库或其他管理功能。
使数据库管理和用户账户管理完全分离。
用户管理账户应该以userAdminAnyDatabase作为唯一的角色来创建。

use  admin
db.createUser(
    { user:"useradmin",
      pwd:"test",
      roles:["userAdminAnyDatabase"],
    }
 )

创建数据库管理员
use  admin
db.createUser(
    {   user:"dbadmin",
        pwd:"test“,    
        roles:
             ["readWriteAnyDatabase",    
              "dbAdminAnyDatabase"]
   }
 );

## 创建、查看和删除数据库

创建数据库
``` 
>use dababase_name
若数据库存在，就连接它；若数据库不存在，就创建它
若只创建数据库，而不作任何操作，空数据库将被删除     
查看数据库
>show dbs
统计某数据库信息
>db.stats()
删除某数据库 
>db.dropDatabase()
查看数据库用户角色权限 
>show roles
查看当前数据库下集合名称 
>show collections 
>db.getCollectionNames()
删除指定数据库中的集合
>db.Collection_name.drop()
给指定数据库添加集合并添加文档
 >db.Collection_name.insert({key:”value”})
查询指定集合中的文档
> db.collection_name.find()  //返回所有文档
> db.collection_name.findOne()  //返回第一条文档
更新文档
db.collection_name.update({key:”value”},{$set:{key:value}})
删除文档
>db.collection_name.remove({key:”value”})
默认只删除第一条同值文档
数据库操作的帮助指令：
>db.help()
集合操作的帮助指令
>db.Collection_name.help()
文档操作的帮助指令
>db.Collection_name.find().help()
MongoDB shell内置JavaScript引擎可直接执行JS代码
>function insert(object)
    {
         db.getCollection(“Collection_name”).insert(object);
           }
>insert({key:”value”})
```

数据库实例建立后，就可以对数据库进行各种基本命令操作了。这里包括了创建、更新、删除、查询、索引、聚合、复制、分片等。

### Find详讲

命令说明:
在集合里查找文档记录。
语法:
   db.[Collection_Name].find ({ query }, {projection} )
  参数解释：
     query       //查询条件设置：查询选择器
     projection       //键指定，指定需要返回的字段：投影操作
数据准备persons.json

例如： 查询出所有数据的指定键(name ,age ,country)
         db.persons.find({  },{name:1,age:1,country:1,_id:0})
注意： _id默认是1，其他字段默认为0，即查询结果不返回。

语法:
db.collection_name.find({ query }, {projection} ) .pretty()

2.  查询条件
查询选择器
    选择器匹配                                                      
    范围查询  
    匹配数组                                                                     
    布尔操作符   
    正则表达式
    where操作符

投影                                 

语法：db.col.find({<key1,value>,{key2:value>,...})
db.user.find({last_name:"Banker"})
db.user.find({first_name:"Smith",age:40})

$lt	     <
$lte	<=
$gt	     >
$gte	>=

db.persons.find({age:{$gte:25,$lte:30}})

$in、$nin、$elemMatch（匹配数组）
若与任一搜索键匹配时，$in就该返回该文档。常用于ID列表
若与任一搜索键不匹配时，$nin才返回文档
若文档中的数组值至少有一个元素满足所有匹配条件，$elemMatch会返回文档

MongoDB的布尔操作符包括$ne,$not,$or,$and和$exists

$ne表示不等于，可以作用于单个值和数组；
例如：查询出所有国籍不是美国籍的学生      
   db.persons.find({country:{$ne:“USA"}},{_id:0,name:1,country:1})

（正则查询）$not对查询结果求反，但不能放于外层文档。注意：后面必须跟正则表达式或者文档(否则：出错$not needs a regex or a document)。
 例如：
查询出姓名里存在 ”li”的学生的信息  db.persons.find({name:/li/i})
查询出姓名里不存在 ”li”的学生的信息
  db.persons.find({name:{$not:/li/i}})

$or表示逻辑或关系:条件1尽可能匹配更多文档，才最为高效的
语法:db.col.find({$or:[{<key1:value>},{<key2,value>},...]})
查询语文成绩大于85或者英语大于90的学生信息
db.persons.find({$or:[{c:{$gte:85}},{e:{$gte:90}}]})

$or与$in:
例如：
查询国籍为中国或美国的学生
db.persons.find({$or:[{country:“China”},{country:“USA"}]})
db.persons.find({country:{$or:["China",“USA"]}})  
db.persons.find({country:{$in:["China”,”USA"]}})

$and主要用于连接具有and关系的复杂查询条件
语法:db.col.find({$and:[{<key1:value>},{<key2,value>},...]})

$exists用于查询集合中是否包含特定键的文档
db.persons.find({gender:{$exists:true}}) 返回键名含有gender的文档
db.persons.find({gender:{$exists:false}}) 返回键名不含有gender的文档

$all（匹配数组）：与所有搜索键匹配，返回文档。
例如：
查询喜欢看MONGOD和JS的学生 
db.persons.find({books:{$all:["MONGOBD","JS"]}})

index应用
例如：查询第二本书是JAVA的学习信息
db.persons.find({"books.1":"JAVA"})

$size（匹配数组）：查询指定长度数组,它不能与比较查询符一起使用(这是弊端)
例如：查询出书籍数量是4本的学生
db.persons.find({books:{$size:4}})

查询出喜欢的书籍数量大于4本的学生
1.  增加字段size
	db.persons.update({},{$set:{size:4}},false, true)
2. 改变书籍的更新方式,每次增加书籍的时候size增加1
    db.persons.update({查询器},{$push:{books:”ORACLE”},$inc:{size:1}}) 
3. 利用$gt查询
    db.persons.find({size:{$gt:4}})

利用shell查询出 lisi 喜欢看的书的数量
var persons = db.persons.find({name:“lisi"})
while(persons.hasNext()){
	obj = persons.next();
           print(obj.books.length)
} 

$regex为模糊查询的字符串提供正则表达式功能
{ <field>: { $regex: /pattern/, $options: '<options>' } }或
{ <field>: { $regex: 'pattern', $options: '<options>' } }或
{ <field>: { $regex: /pattern/<options> } }

正则表达式对象：{ <field>: /pattern/<options> }

$regex与正则表达式对象的区别：
在$in操作符中只能使用正则表达式对象
例如：{name:{$in:[/^joe/i,/^jack/}}
在使用隐式的$and操作符中，只能使用$regex
例如:{name:{$regex:/^zh/i, $nin:["zhang"]}}
当options选项中包含X或S选项时，只能使用$regex，
例如：{email:{$regex:/@qq./,$options:"si"}}

对于上述方法无法实现的复杂查询，可使用特殊的$where操作符，向任意查询中传入一个JavaScript表达式。
例如：查询年龄大于22岁,喜欢看JS书,在八一中学学校上过学的学生 
db.persons.find({age:{$gt:22},books:{$in:["JS"]},"schools.name":"八一中学"})
db.persons.find ({"$where":function () {
var books = this.books;
var schools = this.schools;
if(this.age > 22){
    for ( var i = 0; i < books.length; i++) {
        if(books[i] == "JS"){
              if(schools){
                   for (var j = 0; j < schools.length; j++)
                        If(schools[j].name == "八一中学"){
                                return true;

在查询结果集中，通常用返回的字段集合来定义投影。尤其是对于大文档数据，使用投影能最小化网络延时和序列化开销。
投影中的包含和排除字段: 包含字段设置为1，排除字段设置为0
db.persons.find({},{_id:0,name:1})
借助$slice返回保存在数组中的某个范围内的值
db.persons.find({},{schools:{$slice:2}})\\2: 正数返回的元素数
db.persons.find({},{schools:{$slice:-2}})\\-2: 倒数返回的元素数
实例：查询出“lisi”书架上的第2~4本书
db.persons.find({name: "lisi"},{_id:0,name:1,books:{$slice:[1,3]}})  
\\1: 跳过元素数，2: 返回元素数
实例：查询出“lisi”书架上的最后一本书
db.persons.find({name: "lisi"},{_id:0,name:1,books:{$slice:-1}})

注意：
$slice并不会阻止返回其他字段，如果希望文档中限制其他字段，必须显式的进行控制
db.persons.find({name:"John"},{schools:{$slice:2},"schools.name":1})

嵌套文档查询
db.persons.find({"schools.school":"八一小学"})
查询在“school1”学校上过学且成绩为”A”的学生。
db.persons.find({"schools.school":"school1","schools.score":"A"},{_id:0,name:1})   ?
db.persons.find({schools:{$elemMatch:{school:"school1",score:"A"}}},{_id:0,name:1})

查找null值字段（不仅仅可以查询出来某一字段值为null的记录，还可以查出来不存在某一字段的记录）
例如：把中国国籍的学生上增加新的键sex
db.persons.update({country:"China"},{$set:{sex:"m"}})
例如：查询出sex等于null的学生
db.persons.find({sex:null},{_id:0,country:1})

查询字段不存在的文档
db.persons.find({sex:{$exists:true}})

Sort方法     使用值1（升序）和-1（降序）来按指定属性排序。
例如：返回按照年龄排序的数据
db.persons.find({},{_id:0,name:1,age:1}).sort({age:1})    升序
db.persons.find({},{_id:0,name:1,age:1}).sort({age:-1})    降序

利用游标遍历查询数据
    var  persons = db.persons.find();  //声明游标
    while(persons.hasNext()){     //判断游标是否取到尽头
	    obj = persons.next();     //获取游标的下一个单元
          print(obj.name)
     } 或
var ShowCursor=db.persons.find() 
ShowCursor.forEach(printjson)

游标几个销毁条件
  客户端发来信息叫它销毁
  游标迭代完毕
  默认游标超过10分钟没用也会被清除

使用count方法获取文档数

例如：查询persons中美国学生的人数。
db.persons.find({country:"USA"}).count()

distinct
用法： 查找不同的字段值 
语法：distinct（key，[query]）
参数key是一个字符串，指定了要获取哪个字段的不同值。参数query指定要从哪些文档中获取不同的字段值。

查询出persons中一共有多少个国家分别是什么？
db.persons.distinct(" country ",{ })

db.runCommand(
    {distinct:"persons" , key:"country"}
).values

## 文档操作

### 插入文档

语法：db.collection_name.insert
(<document or array of documents>,//必填写字段
{ //选填字段
 writeConcern: <document>,
 ordered: <boolean>})
命令说明：
在集合里插入一条或多条文档。
db为数据库名（在shell平台用db代表当前数据库，在客户端用编程语言调用时，用具体的数据库名，如当前数据库名为“xscj”，则用xscj代替db）、collection_name为集合名、insert为插入文档命令。

有序插入多个文档
```
>db.goodsbaseinf.insert(
[
  {            item: "小学生教材",name:"《小学一年级语文》",price:12 },
  {            item: "高中生教材",name:"《高中一年级语文》",price:20},
  {           item: "外语教材",name:"《英语全解》",price:30}
]

)
```
ordered默认为true，是按数组中文档的先后顺序插入（而非必须明确指定id，且按文档id的先后顺序插入），如果某条文档插入失败（如：id与原有文档重复等等），就停止其后剩余文档的插入。
如果false，执行无序插入，如果错误发生在某个文档中，则继续处理数组中的剩余文档。

Save操作
db.collection_name.save(<document>)
  不同于insert操作之处在于，插入文档的_id与现有文档的_id相同时：
save用新文档替代原有同id的文档
insert则会报错

简化的插入命令
```
>db.collection.insertOne(文档)
>db.collection.insertMany(文档)
```
用变量方式插入文档
```
>document=({name:"《C语言编程》",price:32})//document为变量名
>db.goodsbaseinf.insert(document)
```

### 更新文档

语法：
 db.collection_name.update(
        <query>,  //update的查询条件
        <update>, //更新对象文档，含操作符功能使用
        //以下为可选参数：
       {upsert: <boolean>,
        multi: <boolean>,  
        writeConcern: document>，
        collation: <document>//3.4新增参数})
命令说明：
在集合里更新一条或多条文档记录。
db为数据库名、collection_name为集合名、update为更新命令。

insertOrUpdate操作 
目的:查询器查出来数据就执行更新操作,查不出来就插入操作
做法:db.[documentName].update({查询器},{修改器},true)

批量更新操作
当查询器查询出多条数据的时候默认就修改第一条数据实现批量修改  
db.[documentName].update({查询器},{修改器},false, true)  

db.order.update({_id:10}，{$min:{amount:50}}）

修改复杂文档
修改一条文档里的数组或嵌套文档
db.order.update({_id:2},
{$set:{"detail.1":{name:"大米",price:40},"overview.address":"天津市和平区成都道9号"}})
db.order.update({_id:2},{$set:{“detail.1.name”:”大米”}})

增加文档字段
db.order.update({_id:10},{$set:{detail:[{name:"西瓜",price:10}],unit:"元"}},true,false)

### 删除文档

语法：
db.collection.remove(<query>,{justOne: <boolean>,writeConcern: <document>,collation: <document>})
命令说明：
在集合里删除一条或多条符合条件的文档。
参数说明：
<query>,//必选项，设置删除文档条件
justOne: <boolean>, //可选项，false为默认值，删除符合条件的所有文档；true删除符合条件的一条文档
writeConcern: <document>,//可选项，自定义写出错确认级别。
collation: <document>//可选项，指定特定国家语言的删除归类规则。
返回值：
删除成功，返回WriteResult({ "nRemoved" : n})对象；
删除失败，返回结果中会包含WriteResult.writeConcernError对象字段内容（跟writeConcern配合使用）。

删除一个集合里的所有文档记录：
db.collection_name.remove({})
db.collection_name.drop({}):将整个集合和索引一起删除
删除符合条件的所有文档记录：
db.collection_name.remove({price:{$gt:3}})
删除符合条件的单个文档记录：
db.collection_name.remove({price:{$gt:3}},{justOne:true})
自定义写出错确认级别：
db.collection_name.remove({price:{$lt:3}},{witeConcern:{w:”majority”,wtimeout:5000})

 如果要清楚一个数据量十分庞大的集合，采用drop命令直接删除该集合并且重新建立索引的办法，比直接用remove的效率和高很多

## 分组(Group)

语法:
      db.runCommand({group:{
	      ns:集合名字,
        Key:分组的键对象,
        Initial:初始化累加器,
        $reduce:组分解器,
        Condition:条件,
        Finalize:组完成器
      }})
分组首先会按照key进行分组,每组的每一个文档全要执行$reduce的方法，它接收2个参数一个是组内本条记录,一个是累加器数据。

请查出persons中每个国家学生数学成绩最好的学生信息(必须在90以上)。
db.runCommand({group:{
    ns:"persons",
    key:{"country":true},
    initial:{m:0},
    $reduce:function(doc,prev){
        if(doc.m > prev.m){
            prev.m = doc.m;
            prev.name = doc.name;
            prev.country = doc.country;
        }
    },
    condition:{m:{$gt:90}}
}})   

## 聚合(Aggregation)

何为aggregate聚合操作？
MongoDB的聚合操作，接受一个名为pipeline的参数和一个可选参数。
 pipeline可以理解为流水线，一条流水线上可以有一个或多个工序。
所以，MongoDB的一次聚合操作就是对一个集合进行多个工序的加工，其中的每个工序都可以修改、增加、删除文档，最终产出我们需要的数据集合。

聚合管道
例如：请查出persons中每个国家学生数学成绩最好的学生信息(必须在90以上)
db.persons.aggregate({$match:{m:{$gte:90}}},{$group:{_id:{country:"$country",name:"$name",value:"$m"}}})

Map-Reduce
1. map部分
作用：用于分组的。
emit(param1, param2)
param1：需要分组的字段，this.字段名。
param2：需要进行统计的字段，this.字段名。

2. reduce部分
作用：处理需要统计的字段
var reduce = function(key, values){
    ......统计字段处理
}
key： 指分组字段（emit的param1）对应的值
values：指需要统计的字段（emit的param2）值组成的数组

3. options部分{ query: { age: {$lt: 25} }, out: "name_totals" }
query：先筛选符合条件的记录出来，再进行分组统计。
out：将分组统计后的结果输出到哪个集合当中。
默认情况下，out所指定的集合在数据库断开连接后再次打开时，依旧存在，并保留之前的所有记录的。

4. 执行分组统计>db.集合名.mapReduce( map, reduce, options )

统计不同地方（‘Guangzhou，Beijing，Shanghai’）的人的年龄总和
var map = function(){ emit(this.location, this.age); }  
var reduce = function( key, values ){ return Array.sum(values); }
var options = { out: "age_totals" }  
db.mythings.mapReduce( map, reduce, options )   

单一目标聚合方法，常见两种：
Count：返回匹配查询结果的数量

例：请查询persons中美国学生的人数.	 
db.persons.find({country:"USA"}).count()

Distinct：返回指定某个字段非重复值的列表

例：请查询出persons中一共有多少个国家分别是什么.
 db.runCommand({distinct:"persons",key:"country"}).values

## 复制(Replication)

Mongodb数据库在实际生产环境下，都是基于多服务器集群运行，并进行相应的数据分布式处理。由此，必须考虑数据读写的可用性和安全性，如一台服务器出故障时，应该能保证Mongodb数据处理的正常进行。
复制(在Mongodb数据库里又可以直接称”副本集“)就是为解决上述问题而产生的，通过复制功能可以实现：
多服务器的数据冗余备份操作；
使备份数据的服务器具备额外提供独立读访问请求的功能（分布式读取数据，可以解决高并发客户端读用户访问问题）；
当服务器出故障时，还能提供自动故障转移、自动数据恢复。

mongodb的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。
mongodb各个节点常见的搭配方式为：一主一从、一主多从。主节点记录在其上的所有操作oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

在执行复制动作前，首先在不同服务器安装Mongod实例（通常一个服务器安装一个mongod.exe程序)，一个节点看做是一台安装了mongod.exe的服务器
一个典型副本集有Primary(主节点)、Secondary(从节点)、Arbiter(仲裁节点)三种角色。
当从节点只有偶数个时，可增加仲裁节点，仲裁节点看做是一台只安装mongod.exe程序，不承担数据副本存储任务。通过心跳(Hearbeat)功能与其他节点联系，当主节点故障时，进行新主节点投票选举。

## 分片

MongoDB的复制集主要用以实现自动故障转移从而达到高可用的目的；
然而，随着业务规模的增长和时间的推移，业务数据量会越来越大，当前业务数据可能只有几百GB不到，一台DB服务器足以搞定所有的工作，而一旦业务数据量扩充大几个TB几百个TB时，就会产生一台服务器无法存储的情况
此时，需要将数据按照一定的规则分配到不同的服务器进行存储、查询等，即为分片集群。
分片集群要做到的事情就是数据分布式存储。

什么是分片？
高数据量和吞吐量的数据库应用会对单机的性能造成较大压力，大的查询量会将单机的 CPU 耗尽，大的数据量对单机的存储压力较大，最终会耗尽系统的内存而将压力转移到磁盘 IO 上。
MongoDB 分片是使用多个服务器存储数据的方法，以支持巨大的数据存储和对数据进行操作。分片技术可以满足 MongoDB 数据量大量增长的需求，当一台 MongoDB 服务器不足以存储海量数据或不足以提供可接受的读写吞吐量时，我们就可以通过在多台服务器上分割数据，使得数据库系统能存储和处理更多的数据。

分片的优势
使用分片减少了每个分片需要处理的请求数，因此，通过水平扩展，群集可以提高自己的存储容量。比如，当插入一条数据时，应用只需要访问存储这条数据的分片。
使用分片减少了每个分片村存储的数据
分片的优势在于提供类似线性增长的架构，提高数据可用性，提高大型数据库查询服务器的性能。当MongoDB单点数据库服务器存储成为瓶颈、单点数据库服务器的性能成为瓶颈或需要部署大型应用以充分利用内存时，可以使用分片技术。

mongodb采用将集合进行拆分，然后将拆分的数据均摊到几个片上的一种解决方案。

## Java连接MongoDB

```
public class MongoDBTest {
	public static void main(String[] args) {
		MongoClient mg=new MongoClient("127.0.0.1");
//		for(String name:mg.listDatabaseNames()) {
//			System.out.println("dbName:"+name);
//		}
		
		MongoDatabase db=mg.getDatabase("xscj");
		for(String name:db.listCollectionNames()) {
			System.out.println("CollectionName"+name);
		}
		MongoCollection<Document> users=db.getCollection("persons");
		FindIterable<Document> findIter=users.find();
		MongoCursor<Document> cur=findIter.iterator();
		while(cur.hasNext()) {
			Document doc=cur.next();
			System.out.println(doc.get("name")+">>"+doc.get("country"));
		}
		System.out.println(users.count());
		mg.close();
	}
}
```

```
public class MongoDB1 {
	MongoClient conn=null;
	static MongoDatabase db=null;
	
	public MongoDB1(String dbName) {
		conn=new MongoClient("127.0.0.1");
		db=conn.getDatabase(dbName);
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		MongoDB1 mongodb=new MongoDB1("xscj");
//		db.createCollection("stu");
		
//		Document doc=new Document();
//		doc.put("name", "zs");
//		doc.put("age", 25);
//		List<String> books=new ArrayList<String>();
//		books.add("C");
//		books.add("Java");
//		books.add("jsp");
//		doc.put("books1", books);
//		
//		MongoCollection<Document> coll1=db.getCollection("stu");
//		coll1.insertOne(doc);
		
//		Document doc1=new Document();
//		doc1.put("name", "jim");
//		doc1.put("age", 20);
//		Document doc2=new Document();
//		doc2.put("name","tom");
//		doc2.put("age", 25);
//		List<String> books=new ArrayList<String>();
//		books.add("C1");
//		books.add("Java1");
//		books.add("jsp1");
//		doc2.put("books2",books);
//		
//		List<Document> docs=new ArrayList<Document>();
//		docs.add(doc1);
//		docs.add(doc2);
//		MongoCollection<Document> coll1=db.getCollection("stu");
//		coll1.insertMany(docs);
		
//		Document param=new Document();
//		param.append("name","jim");
//		MongoCollection<Document> coll1=db.getCollection("stu");
//		coll1.deleteOne(param);
		
//		MongoCollection<Document> coll1=db.getCollection("stu");
//		Document update=new Document();
//		update.append("$set", new Document("email","tom@nuc.edu.cn"));
//		coll1.updateOne(Filters.eq("name","tom"),update);
//		
		MongoCollection<Document> coll1=db.getCollection("persons");
		Bson filter=Filters.and(Filters.gte("age", 26),
				Filters.lte("e", 80));
		
		
		FindIterable<Document> findIter=coll1.find();
		MongoCursor<Document> cur=findIter.iterator();
		while(cur.hasNext()) {
			Document doc=cur.next();
			System.out.println(doc.get("name")
					+">>"+doc.get("age")
					+">>"+doc.get("e"));
		}
		SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");//设置日期格式
		String date = df.format(new Date());
	}

}

```