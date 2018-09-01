# 前言

本手册介绍了与Oracle Database In-Memory功能集相关的体系结构和任务。

本前言包含以下主题：

* 面向的读者
* 相关文档
* 约定

## 面向的读者

本文档面向管理In-Memory列存储（IM column store）的数据库管理员，以及优化使用Oracle数据库内存中功能的分析查询的开发人员

## 相关文档

本手册假定您熟悉Oracle数据库概念。 引用以下书籍：

* Oracle Database Data Warehousing Guide
* Oracle Database VLDB and Partitioning Guide
* Oracle Database SQL Tuning Guide
* Oracle Database SQL Language Reference
* Oracle Database Reference

本书中的许多示例使用sample schemas，这些示例在Oracle数据库选择基本安装选项时默认安装。 有关如何创建这些模式以及如何使用它们的信息，请参阅: Oracle Database Sample Schemas (http://docs.oracle.com/database/122/COMSC/introduction-to-sample-schemas.htm#COMSC005)。

## 约定

本文档中使用以下文本约定：

|惯例|含义|
|---|---|
粗体|粗体类型表示与动作相关联的图形用户界面元素，或者在文本或词汇表中定义的术语。
_斜体_|斜体类型表示您为其提供特定值的书名、重点或占位符变量。
等宽|等宽类型表示段落中的命令、URL、示例中的代码，屏幕上显示的文本或您输入的文本。
