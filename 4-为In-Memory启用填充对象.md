# 4 为In-Memory填充(population)启用对象

本章介绍如何在IM列存储中启用和禁用填充对象，包括设置压缩和优先级选项。

本章包含以下主题：

* 关于 In-Memory

填充当数据库从磁盘读取现有行格式数据，将其转换为列格式，然后将其存储在IM列存储中时，发生In-Memory 填充 (population)。只有具有  INMEMORY属性的对象才有资格进行填充。

* 启用和禁用IM列存储的表

通过在CREATE TABLE 或 ALTER TABLE 语句中包含 INMEMORY 子句来启用IM列存储的表。通过在CREATE TABLE 或 ALTER TABLE 语句中包含 NO INMEMORY 子句来禁用IM列存储的表。

* 启用和禁用内存表的列

您可以为单独的列指定 INMEMORY 子句。非虚拟列和In-Memory虚拟列（IM虚拟列）都有资格进入IM列存储。

* 启用和禁用IM列存储的表空间

您可以启用或禁用IM列存储的表空间。

* 启用和禁用IM列存储的物化视图

您可以为IM列存储启用和禁用物化视图。

* In-Memory对象的强制填充：教程

启用In-Memory填充的对象不会立即填充该对象。

* 为IM列存储启用ADO

信息生命周期管理（ILM）是一组用于管理从创建到归档或删除的数据的过程和策略。
