# SQL 语句从编写完成后到返回结果时的流程

1. 词法分析

作用：语句到Token

Token分类：

- 注释
- 关键字（不关心是什么关键字）
- 操作符
- 开闭合标志（为了支持字语句）
- 占位符
- 空格
- 引号包括的文本、数字、字段
- 方言语法

MySqlParser：https://github.com/antlr/grammars-v4/blob/master/sql/mysql/Positive-Technologies/MySqlParser.g4

Antlr4: 

左推导/右推导，

通过不断执行正则匹配函数，将一个SQL语句，不断的切分成一个个Token，

2. 语法分析

3. 错误检测

4. 语义分析

