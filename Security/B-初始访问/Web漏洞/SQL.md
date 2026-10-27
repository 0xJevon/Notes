# SQL（Structured Query Language）

用于管理（访问和处理）关系数据库管理系统（RBDMS）

包括数据插入、查询、更新和删除

单/双引号围绕文本值，数值字段不使用引号

## 数据库表（>=1）

每个表有唯一一个名字标识，表包含有数据记录（行）

## 语句（对大小写不敏感）

**SELECT** - 从数据库中提取数据

结果集：结果储存于结果表

SELECT (可选取部分数据或用 ***** 选取所有数据)FROM

**SELECT DISTINCT**-用于返回唯一不同值（去除重复值）

**WHERE** 子句- 用于提取满足指定条件的记录

​                 运算符    特殊：BETWEEN ,LIKE ,IN

​                                AND:两条件都成立，显示一条记录

​                                OR:两条件成立一，显示一条记录

​                                AND(    OR      )

ORDER BY 关键字 - 用于对结果集按照一列或多列排列

​                                  默认升序，可用DESC降序 

**UPDATE** - 更新数据库中的数据

UPDATE table_name

SET  value1=value2

WHERE                                         慎重对待此子句

**DELETE** - 从数据库中删除数据

DELETE FROM

WHERE                 需指定位置否则全删

**INSERT INTO** - 向数据库中插入新数据

INSERT INTO table_name(可指定插入列)

VALUES(插入值）（若不指定列名，则需给出每一列值)

**CREATE DATABASE** - 创建新数据库

**ALTER DATABASE** - 修改数据库

**CREATE TABLE** - 创建新表

**ALTER TABLE** - 变更（改变）数据库表

**DROP TABLE** - 删除表

**CREATE INDEX** - 创建索引（搜索键）

**DROP INDEX** - 删除索引