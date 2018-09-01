## 5 In-Memory表达式

在IM列存储的上下文中，表达式是一个或多个值，运算符以及解析为值的SQL或PL / SQL函数（仅DETERMINISTIC ）的组合。

表达式统计存储（ESS）自动跟踪经常评估（“热”）表达式的结果。您可以使用DBMS_INMEMORY_ADMIN包来捕获热表达式，并将它们填充为隐藏的虚拟列，或删除其中的一些或全部。

此部分包含以下主题：

* 关于IM表达式

  默认情况下，DBMS_INMEMORY_ADMIN.IME_CAPTURE_EXPRESSIONS过程标识并填充“热”表达式，称为In-Memory表达式（IM表达式）。

* 配置IM表达式用法

  （可选）使用INMEMORY_EXPRESSIONS_USAGE 选择哪些类型的IM表达式有资格进行填充，或禁用所有IM表达式的填充。

* 捕获和填充IM表达式

  IME_CAPTURE_EXPRESSIONS过程捕获并填充指定时间范围内数据库中20个最常访问（“最热”）的表达式。IME_POPULATE_EXPRESSIONS过程强制在最近一次调用DBMS_INMEMORY_ADMIN.IME_CAPTURE_EXPRESSIONS中捕获的表达式。

* 删除IM表达式

  DBMS_INMEMORY_ADMIN.IME_DROP_ALL_EXPRESSIONS存储过程删除数据库中的所有SYS_IME表达式虚拟列。 DBMS_INMEMORY.IME_DROP_EXPRESSIONS存储过程从表中删除指定的一组SYS_IME虚拟列。
