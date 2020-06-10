# JSON(JavaScript  Object  Notation) 是一种轻量级的数据交换格式。
 
JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括C、C++、C#、Java、JavaScript、Perl、Python等）。

这些特性使JSON成为理想的数据交换语言。 易于阅读和编写，同时也易于机器解析和生成(一般用于提升网络传输速率)。

## JSON 语法规则
JSON 语法是 JavaScript 对象表示语法的子集。
数据在键值对中
数据由逗号分隔
花括号保存对象
方括号保存数组

## JSON 数据的书写格式是：键/值对。
名称/值对组合中的名称写在前面（在双引号中），值对写在后面(同样在双引号中)，中间用冒号   ：隔开
                   "firstName":"John“
等价于这条 JavaScript 语句
                   firstName="John"

## JSON 值可以是：
数字（整数或浮点数）
字符串（在双引号中）
逻辑值（true 或 false）
数组（在方括号中）
对象（在花括号中）
null

## JSON 结构有两种结构：JavaScript中的对象和数组
对象：
对象在JS中表示为“{ }”括起来的内容，数据结构为 {key:value,key:value,...}的键值对的结构。
在面向对象中，key为对象的属性，value为对应的属性值，通过对象.key 获取属性值，这个属性值的类型可以是数字、字符串、数组、对象几种。

数组：
数组在JS中是中括号“[  ]”括起来的内容，数据结构为 [“Java”,“JavaScript”,“vb”,...]。
取值方式和所有语言中一样，使用索引获取，字段值的类型可以是数字、字符串、数组、对象几种。
通过这两种结构可以表示各种复杂的结构。

简单地说 ，JSON 可将 JavaScript 对象中的一组数据转换为字符串。
转换后的字符串可在函数之间轻松传递，也可在异步应用程序中从Web 客户机传递给服务器端程序。
JavaScript很容易解释它，而且 JSON 可以表示比"名称 / 值对"更复杂的结构。

## JSON最基本的“名称 / 值对”表示如： {“firstName”:“Brett”}

但是，当多个“名称 / 值对”串在一起时，JSON可以创建包含多个“名称 / 值对”的 记录。
如：{"firstName":"Brett","lastName":"McLaughlin","email":"aaaa"}

这种情况下 JSON 可读性更好。
例如，它明确地表示以上三个值都是同一记录的一部分；花括号使这些值有了某种联系。

当表示一组值时，JSON 不但提高可读性，还可减少复杂性。
如果使用 JSON，就只需将多个带花括号的记录分组在一起

可以使用相同的语法表示多个值（每个值包含多个记录）：

掌握了 JSON 格式之后，在 JavaScript 中使用它就很简单了。
JSON 是JavaScript  原生格式，这意味着在 JavaScript 中处理 JSON 数据不需要任何特殊的 API 或工具包。

实际上，只需用点号表示法来表示数组元素。所以，要想访问 programmers 列表的第一个条目的姓氏，只需在 JavaScript 中使用下面这样的代码：
people.programmers[0].lastName;
注意，数组索引是从零开始的。所以，这行代码首先访问 people变量中的数据；然后移动到称为 programmers的条目，再移动到第一个记录（[0]）；最后，访问 lastName键的值。结果是字符串值 “McLaughlin”。

用点号和方括号访问数据，也可以按照同样的方式轻松地修改数据：
people.musicians[1].lastName="Rachmaninov";

因此，如果要处理大量 JavaScript 对象，那么 JSON 是一个好选择，这样就可以轻松地将数据转换为可以在请求中发送给服务器端程序的格式。

许多JavaScript树形控件使用JSON嵌套格式描述树形结构

值（value）可以是双引号括起来的字符串（string）、数值(number)、true、false、 null、对象（object）或者数组（array）。这些结构可以嵌套。
字符串（string）是由双引号包围的任意数量Unicode字符的集合，使用反斜线转义。字符串（string）与C或者Java的字符串非常相似。
数值（number）也与C或者Java的数值非常相似。除去未曾使用的八进制与十六进制格式。除去一些编码细节。

对象是一个无序的“‘名称/值’对”集合
（1）一个对象以“{”开始，“}”结束。
（2）每个“名称”后跟一个“:”；
（3）“‘名称/值’ 对”之间使用“,”分隔。

数组是值（value）的有序集合。
一个数组以“[”开始，“]”结束
值之间使用“,”分隔。

BSON( Binary Serialized Document Format) 支持内嵌的文档对象和数组对象，可以有效描述非结构化数据和结构化数据。
BSON是一种计算机数据交换格式，主要被用作MongoDB数据库中的数据存储和网络传输格式。它是一种二进制表示形式，能用来表示简单数据结构、关联数组（MongoDB中称为“对象”或“文档”）以及MongoDB中的各种数据类型。
BSON之名缘于JSON，含义为Binary JSON（二进制JSON）。

它和JSON一样，支持内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date和BinData类型。
BSON可以做为网络数据交换的一种存储形式，是一种schema-less的存储形式，它的优点是灵活性高，但它的缺点是空间利用率不是很理想。

### JSON和BSON对比

数据结构 　　JSON是像字符串一样存储的，BSON是按结构存储的（像数组 或者说struct）
存储空间 　　BSON > JSON
操作速度 　　BSON > JSON。比如，遍历查找：JSON需要扫字符串，而BSON可以直接定位。
修改　　 JSON要大动大移， BSON就不需要。

