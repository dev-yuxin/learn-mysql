# 常用数据类型

## 整型

```sql
CREATE TABLE test_int(
  a INT,  -- 整型
  b INT UNSIGNED,  -- 无符号整型，非负整数
  c INT(3) ZEROFILL  -- 不足3位时前面补0，超过3位不补0，且ZEROFILL会开启UNSIGNED
);
```

## 小数型

```sql
CREATE TABLE test_decimal(
  a FLOAT,  -- 大约精确到小数点后8位
  b DOUBLE, -- 大约精确到小数点后16位
  -- (5, 2) 总计5位数字，小数点后保留2位，[-999.99, 999.99]，超过范围报错，在范围内会保留小数
  c FLOAT(5, 2),
  d DOUBLE(5, 2),
  -- 精确计算请使用 DECIMAL
  e DECIMAL(5, 2),
  f DECIMAL  -- 等价于 DECIMAL(10, 0)
);
```

## 字符串型

```sql
CREATE TABLE test_str(
  a CHAR,  -- 只能存1个字符
  b CHAR(10),  -- 最多能存10个字符，即使不足10个字符也会使用10个字符的空间
  c VARCHAR(10),  -- VARCHAR(n) 必须指定n，最多存n个字符，不足n个字符将调整存储空间
  d TEXT  -- 存储长文本
);
```

## 枚举和集合

```sql
CREATE TABLE test_enum_set(
  gender ENUM('男', '女', '保密'),  -- ENUM 里面选1个
  hobbies SET("唱", "跳", "篮球")  -- SET 里面选1或多个
);


INSERT INTO test_enum_set VALUES('男', "唱,跳,篮球");

-- 男 1, 女 2, 保密 3 
-- 唱 1, 跳 2, 蓝球 4, 6=2+4 
INSERT INTO test_enum_set VALUES(3, 6);
```

## 日期时间

```sql
CREATE TABLE test_datetime (
  a YEAR,  -- 存年份整数，范围比较小
  b DATE,  -- 存日期字符串
  c TIME,  -- 存时间字符串
  d DATETIME,  -- 存日期时间字符串
  e TIMESTAMP  -- 存时间戳，范围比较小（可以传入时间戳整数，也可以传入时间字符串底层会根据时区转换为时间戳）
);

INSERT INTO test_datetime VALUES(2026, '2026-01-01', '10:00:00', '2026-01-01 10:00:00', '2026-01-01 10:00:00');
```

# 基本运算

## 算术运算

```sql
-- 支持 +, -, *, /, div(整除), %
SELECT ename, salary, salary / 30, salary div 30, salary * 12
FROM t_employee;
```

## 比较运算

```sql
-- =(一个等号), !=, <>(<>等价于!=), <, <=, >, >=
-- 不能比较字符串
-- 不能和 NULL 比较
-- 查询非1号部门（不含没有部门）的员工姓名和部门号
SELECT ename, did
FROM t_employee
WHERE did <> 1;
```

```sql
-- 和NULL比较使用 IS, IS NOT, <=>(<=>等价于IS) 
-- 查询非1号部门（包括没有部门）的员工姓名和部门号
SELECT ename, did
FROM t_employee
WHERE did <> 1 OR did <=> NULL;
```

```sql
-- IN, NOT IN
-- 查询非 1, 2, 3 号部门（不含没有部门）的员工的姓名和部门号
SELECT ename, did
FROM t_employee
WHERE did NOT IN (1, 2, 3);
```

```sql
-- BETWEEN AND
-- 查询薪资在[10000, 20000]的员工姓名和薪资
SELECT ename, salary
FROM t_employee
WHERE salary BETWEEN 10000 AND 20000;
```

```sql
-- LIKE 字符串模糊匹配
-- % 匹配任意个字符，_ 匹配一个字符
-- 查询员工名字第2个字是冰的员工姓名
SELECT ename
FROM t_employee
WHERE ename LIKE '_冰%';
```

```sql
-- ALL 比所有
-- SOME 比其中一个
-- 查询李冰冰所在部门的员工（如果同名，列出所有）
SELECT * FROM t_employee
WHERE did=SOME(SELECT did FROM t_employee WHERE ename='李冰冰');

-- 查询比李冰冰薪资高的员工（如果同名，列出比他们都高的）
SELECT * FROM t_employee
WHERE salary>ALL(SELECT salary FROM t_employee WHERE ename='李冰冰');
```

## 逻辑运算

```sql
-- 与 and, &&
-- 或 or, ||
-- 非 not, !

-- 查询薪资不在[10000, 20000]范围的男员工
SELECT *
FROM t_employee
WHERE salary NOT BETWEEN 10000 AND 20000 AND gender='男';

SELECT *
FROM t_employee
WHERE !(salary BETWEEN 10000 AND 20000) && gender='男';
```

## 处理 NULL 值

```sql
-- NULL 值比较只能使用 IS, IS NOT, <=>
-- NULL 参与计算都是 NULL
-- IFNULL(字段, 值) 用指定值代替NULL值
SELECT ename AS 姓名, salary AS 基础薪资, commission_pct AS 绩效, salary * (1+IFNULL(commission_pct,0)) AS 实发薪资
FROM t_employee;
```

## 去重

```sql
-- DISTINCT 字段1, 字段2, 字段3 表示字段1+字段2+字段3组合值去重
-- 查询员工表所有部门编号
SELECT DISTINCT did
FROM t_employee;
```

# 内置函数

## 分组函数

分组函数，或者叫聚合函数，多行函数。多行数据一起计算，m 行数据得到的结果 <= m 行。

```sql
-- 查询最高、最低、总、平均薪资
SELECT MIN(salary), MAX(salary), SUM(salary), AVG(salary)
FROM t_employee;
```

```sql
-- 统计每个部门的平均薪资
-- 注意，如果使用了 GROUP BY ，SELECT 后面只能跟 GROUP BY 后面的字段以及分组函数
SELECT did, AVG(salary)
FROM t_employee
GROUP BY did;
```

```sql
-- COUNT(*) 统计所有个数
-- COUNT(常量) 统计所有个数
-- COUNT(字段) 统计该字段有多少个非 NULL 值个数
SELECT COUNT(*) FROM t_employee;
SELECT COUNT(1) FROM t_employee;
SELECT COUNT(eid) FROM t_employee;
```

```sql
-- 统计每个部门男女人数
SELECT did, gender, COUNT(*)
FROM t_employee
GROUP BY did, gender
-- WITH ROLLUP 会进行统计，根据n个字段分组会产生对n, n-1, ... ,1, 0(全表)个字段的统计行，总计n+1行
WITH ROLLUP;
```

```sql
-- 每个部门按男女统计平均薪资，只显示平均薪资>=10000的
SELECT did, gender, COUNT(*), AVG(salary)
FROM t_employee
GROUP BY did, gender
-- HAVING 是对分组后的结果进行统计
-- HAVING 和 WHERE 区别：
-- 1 HAVING 是对 GROUP BY 子句筛选，WHERE 是对 FROM 子句筛选
-- 2 WHERE 后面不能跟分组函数，HAVING 后面可以
HAVING AVG(salary) >= 10000;
```

## 单行函数

单行行数是一行一行计算，m 行数据得到的结果还是 m 行。

### 数学函数

```sql
-- 向上取整
SELECT CEIL(3.55);
-- 向下取整
SELECT FLOOR(3.55);
-- 保留一位小数（四舍五入）
SELECT ROUND(3.55, 1);
-- 保留一位小数（直接截断）
SELECT TRUNCATE(3.55, 1);
-- 取绝对值
SELECT ABS(-3);
```

### 字符串函数

```sql
 -- 字符串拼接
SELECT CONCAT('a', 'b', 'c');
-- 字符串使用指定字符连接
SELECT CONCAT_WS('-', 'a', 'b', 'c');

-- 字符串所占字节数
SELECT LENGTH('你好');
-- 字符串字符数
SELECT CHAR_LENGTH('你好');

-- 找某个子串在字符串中首次出现的位置（索引从1开始，没找到返回0，可指定从哪个索引开始找）
SELECT LOCATE('o', 'hello world', 6);

-- 取最左/右几个字符
SELECT LEFT('hello', 1);
SELECT RIGHT('hello', 1);

-- 去掉前后指定字符，默认空白符
SELECT TRIM(' hello ');
SELECT LTRIM(' hello ');
SELECT RTRIM(' hello ');
SELECT TRIM(BOTH '*' FROM '**hello**');
SELECT TRIM(LEADING '*' FROM '**hello**');
SELECT TRIM(TRAILING '*' FROM '**hello**');

-- 比较字符串大小，0 表示相等，负数表示小于，正数表示大于
SELECT STRCMP('hello', 'world');

-- 截取（索引从1开始），从第5个字符开始截取长度为5的子串
SELECT SUBSTRING('www.baidu.com', 5, 5);
-- 截取（索引从1开始），从左往右截取到第 2 个 .
SELECT SUBSTRING_INDEX('www.baidu.com', '.', 2);
-- 截取（索引从1开始），从右往左截取到第 2 个 .
SELECT SUBSTRING_INDEX('www.baidu.com', '.', -2);
```

### 日期时间函数

```sql
-- 获取当前日期/时间/日期加时间
SELECT CURDATE();
SELECT CURTIME();
SELECT NOW();

-- 获取某个时间的年、月、日、时、分、秒
-- 查询本月过生日的员工
SELECT ename, birthday FROM
t_employee
WHERE MONTH(CURDATE()) = MONTH(birthday);

-- 两个日期之间的间隔
-- 查询入职10年以上的老员工
SELECT ename, hiredate, DATEDIFF(CURDATE(), hiredate) FROM
t_employee
WHERE DATEDIFF(CURDATE(), hiredate) >= 10 * 365;
```

### 加密函数

```sql
CREATE TABLE t_user(
  username VARCHAR(20),
  `password` VARCHAR(256) 
);

INSERT INTO t_user VALUES('zhangsan', SHA2('123456', 256));

SELECT * FROM t_user WHERE username='zhangsan' and `password`=SHA2('123456', 256);
```

### 条件判断函数

```sql
-- IFNULL(字段, 值)
SELECT ename AS 姓名, salary AS 基础薪资, commission_pct AS 绩效, salary * (1+IFNULL(commission_pct,0)) AS 实发薪资
FROM t_employee;

-- IF(表达式, 值1, 值2)
SELECT
  ename AS 姓名,
  salary AS 基础薪资,
  commission_pct AS 绩效,
  salary * (1+IF(commission_pct IS NOT NULL, commission_pct, 0)) AS 实发薪资
FROM t_employee;

-- CASE WHEN 类似于 if-elif-else 语句
SELECT ename, salary,
CASE
  WHEN salary>=20000 THEN '高薪'
  WHEN salary>=15000 THEN '较高薪'
  WHEN salary>=10000 THEN '平均'
  ELSE '低于平均'
END AS 薪资情况
FROM t_employee;

-- CASE WHEN 类似于 match-case 语句
SELECT ename, work_place,
CASE LENGTH(work_place) - LENGTH(REPLACE(work_place, ',', '')) + 1
  WHEN 3 THEN 3
  WHEN 2 THEN 2
  WHEN 1 THEN 1
  WHEN 0 THEN 0
  ELSE '>=3'
END AS 工作城市数量
FROM t_employee;
```

## 窗口函数

窗口函数本质上是可以分组的单行函数。

### ROW_NUMBER, RANK, DENSE_RANK

```sql
-- 查询员工信息，并加上一列编号（从1开始）
SELECT ROW_NUMBER() OVER() AS 编号, eid, ename, salary
FROM t_employee;

-- 查询员工信息，并加上一列按部门分组的编号
SELECT ROW_NUMBER() OVER(PARTITION BY did) AS 编号, eid, ename, salary, did
FROM t_employee;

-- 查询员工信息，并加上一列按部门分组按薪资降序的编号
SELECT ROW_NUMBER() OVER(PARTITION BY did ORDER BY salary DESC) AS 编号, eid, ename, salary, did
FROM t_employee;

-- ROW_NUMBER() 遇到相同值是 1 2 3 4 ...
-- RANK() 遇到相同值是 1 1 1 4 ...
-- DENSE_RANK() 遇到相同值是 1 1 1 2 ...
-- 查询每个部门薪资前3的员工信息
WITH temp AS (
  SELECT DENSE_RANK() OVER(PARTITION BY did ORDER BY salary), eid, ename, did, salary
  FROM t_employee
  WHERE did IS NOT NULL
)
-- WHERE 后面不能使用分组函数、窗口函数、别名
SELECT * FROM temp;
```

### 分组函数 + OVER

```sql
-- SUM/AVG/MAX/MIN/COUNT 可以和分组函数一起使用，变为单行函数
-- 查询每个部门的平均薪资，全公司的平均薪资，以及差值
SELECT DISTINCT
  did,
  AVG(salary) OVER(PARTITION BY did) AS 部门平均薪资,
  AVG(salary) OVER() AS 公司平均薪资,
  AVG(salary) OVER(PARTITION BY did) - AVG(salary) OVER() AS 差值
FROM t_employee
WHERE did IS NOT NULL;
```

# DML 语句

## 添加语句

```sql
-- 添加一行数据（给全部字段赋值，要求值和表结构一一对应）
INSERT INTO t_department VALUES(7, '安保部', '负责安保工作');

-- 添加一行数据（只给部分字段赋值）
INSERT INTO t_department(did, dname) VALUES(8, '运营部');

-- 添加多行数据
INSERT INTO t_department VALUES(9, '法务部', '负责法务工作'), (10, '公关部', '负责公关工作');
INSERT INTO t_department(did, dname) VALUES(11, '招聘部'), (12, 'AI部');
```

## 删除语句

```sql
-- 删除公关部
DELETE FROM t_department WHERE dname='公关部';

-- 删除整张表数据，保留表结构，一行行删，效率低，事务中支持回滚
DELETE FROM `表名`;

-- 截断表，保留表结构，一下子全部清空表数据，效率高，但事务中不支持回滚
TRUNCATE `表名`;
```

## 修改语句

```sql
-- 给1号员工涨薪10%，且工作地点改为北京和上海
UPDATE t_employee SET salary=salary*1.1, work_place='北京,上海'
WHERE eid=1
```

## 简单查询

```sql
-- 查询一号部门的员工所有信息
SELECT *
FROM t_employee
WHERE did=1

-- DISTINCT 去重
SELECT DISTINCT did
FROM t_employee;

-- 别名（AS可省略）
SELECT eid AS 员工编号, ename AS 员工姓名
FROM t_employee AS 员工表;
```

## 关联查询

### 内连接

```sql
-- 内连接方式一：使用 [INNER] JOIN ON，INNER 可省略，JOIN 将产生笛卡尔积，再使用 ON 筛选
-- 内连接查询员工编号、员工姓名、部门编号、部门名称（只显示有部门的员工和有员工的部门）
SELECT eid, ename, t_employee.did, dname
FROM t_employee JOIN t_department ON t_employee.did=t_department.did;


-- 内连接方式二：使用 , 和 WHERE，`表1`,`表2`将产生笛卡尔积，再使用 WHERE 筛选
-- 内连接查询员工编号、员工姓名、部门编号、部门名称（只显示有部门的员工和有员工的部门）
SELECT eid, ename, t_employee.did, dname
FROM t_employee, t_department
WHERE t_employee.did=t_department.did;

-- 内连接查询员工编号、员工姓名、部门编号、部门名称、职位编号、职位名称
-- 3张表内连接，至少2个关联字段
SELECT eid, ename, t_employee.did, dname, job_id, jname
FROM
	t_employee 
	JOIN t_department ON t_employee.did=t_department.did
	JOIN t_job ON t_employee.job_id=t_job.jid;
	
-- 内连接查询员工编号、员工姓名、部门编号、部门名称、职位编号、职位名称
SELECT eid, ename, t_employee.did, dname, job_id, jname
FROM
	t_employee, t_department, t_job
WHERE
	t_employee.did=t_department.did AND t_employee.job_id=t_job.jid;
```

### 左外连接

```sql
-- 左外连接：使用 LEFT [OUTER] JOIN ON，OUTER 可以省略，ON 不能省略。
-- 查询员工编号、员工姓名、部门编号、部门名称（包括没有部门的员工）
SELECT eid, ename, t_employee.did, dname
-- 根据语义查员工，主表是 t_employee，放到 LEFT JOIN 左侧或 RIGHT JOIN 右侧
FROM t_employee LEFT JOIN t_department ON t_employee.did=t_department.did;

-- 查询员工编号、员工姓名、部门编号、部门名称、职位编号、职位名称（包括没有部门的员工和没有职位的员工）
SELECT eid, ename, t_employee.did, dname, job_id, jname
FROM 
  t_employee
  LEFT JOIN t_department ON t_employee.did=t_department.did
  LEFT JOIN t_job ON t_employee.job_id=t_job.jid;
  
-- 查询没有部门的员工，显示员工编号、员工姓名
SELECT eid, ename
FROM t_employee LEFT JOIN t_department ON t_employee.did=t_department.did
-- 推荐写 t_department.did，查孤立数据时推荐写 LEFT JOIN 的右表或 RIGHT JOIN 的左表
WHERE t_department.did IS NULL;

-- 查询没有部门的员工和没有职位的员工，显示员工编号、员工姓名、部门编号、部门名称、职位编号、职位名称
SELECT eid, ename, t_employee.did, dname, job_id, jname
FROM 
  t_employee
  LEFT JOIN t_department ON t_employee.did=t_department.did
  LEFT JOIN t_job ON t_employee.job_id=t_job.jid
WHERE t_department.did IS NULL OR t_job.jid IS NULL;
```

### 右外连接

```sql
-- 右外连接：使用 RIGHT [OUTER] JOIN ON，OUTER 可以省略，ON 不能省略。
-- 查询没有部门的员工，显示员工编号、员工姓名
SELECT eid, ename
FROM t_department RIGHT JOIN t_employee ON t_employee.did=t_department.did
WHERE t_department.did IS NULL;

-- 当 > 2 张表时不推荐使用右外连接，使用左外连接
```

### 全外连接

```sql
-- 使用 UNION 合并两个查询结果实现全外连接，要求合并的 2 个 SELECT 后面的字段一致
-- 全外连接查询所有员工和部门（包括没有部门的员工以及没有员工的部门），显示员工编号、员工姓名、部门编号、部门名称
SELECT eid, ename, t_employee.did, dname
FROM
	t_employee LEFT JOIN t_department ON t_employee.did=t_department.did
-- UNION ALL 交集部分不去重，UNION 交集部分去重
UNION
SELECT eid, ename, t_employee.did, dname
FROM t_department LEFT JOIN t_employee ON t_department.did=t_employee.did;

-- 查询没有部门的员工和没有员工的部门，显示员工编号、员工姓名、部门编号、部门名称
SELECT eid, ename, t_employee.did, dname
FROM t_employee LEFT JOIN t_department ON t_employee.did=t_department.did
WHERE t_department.did IS NULL
UNION
SELECT eid, ename, t_employee.did, dname
FROM t_department LEFT JOIN t_employee ON t_department.did=t_employee.did
WHERE t_employee.did IS NULL;
```

### 自连接

```sql
-- 查询所有员工的员工编号、员工姓名、以及他领导的编号和姓名
SELECT t_employee.eid, t_employee.ename, t_mgr.eid, t_mgr.ename
FROM t_employee LEFT JOIN t_employee AS t_mgr ON t_employee.mid=t_mgr.eid;
```

### 笛卡尔积

如果 `表1, 表2` 不写 `WHERE` ，以及 `表1 JOIN 表2` 不写 `ON` 就会产生笛卡尔积。假设表1有m行，表2有n行，那么最后会有m*n行。

实际开发中应正确去除笛卡尔积。

## SELECT 完整形式

书写必须按照以下顺序：

```sql
SELECT DISTINCT 字段或表达式
FROM 表1 LEFT / RIGHT JOIN 表2 ON 条件
WHERE 筛选条件
GROUP BY 分组字段 WITH ROLLUP
HAVING 分组筛选
ORDER BY 排序字段列表 
LIMIT m,n;
```

执行顺序：

1. 先执行 FROM 子句，产生笛卡尔积，再使用 ON 过滤。
2. 再执行 WHERE 对 FROM 子句过滤。
3. 再执行 GROUP BY 子句分组，如果有 WITH ROLLUP 则执行 WITH ROLLUP 进行分组汇总。
4. 再执行 HAVING 子句进行分组筛选。
5. 再执行 SELECT 子句，如果有  DISTINCT 则执行 DISTINCT 去重。
6. 再执行 ORDER BY 子句排序。
7. 再执行 LIMIT 子句分页。

```sql
-- 查询每个部门男员工的平均薪资，筛选出男员工平均薪资>=10000的，并且降序排列
SELECT t_department.did, dname, AVG(salary) AS 男员工平均薪资
FROM t_department LEFT JOIN t_employee ON t_department.did=t_employee.did
WHERE gender='男'
GROUP BY did, dname
HAVING 男员工平均薪资>=10000
-- 按照男员工平均薪资降序排列，如果相同则按部门号升序
ORDER BY 男员工平均薪资 DESC, did;

SELECT * FROM t_employee
-- LIMIT m,n 表示从第m行（行数从0开始）开始展示n条数据
LIMIT 0,5;

-- 假设要展示第5页（页码从1开始），每页6条，则 m=(5-1)*6, n=6
SELECT * FROM t_employee
LIMIT 24,6;
```

## 引号与别名

SQL 不区分大小写。

关于引号：

- 单引号：用于包裹字面量值，主要是字符串值和日期时间值。
- 反引号：用于包裹标识符（数据库名、表名、字段名），如果和关键字同名应加上以示区分。
- 双引号：在 MySQL 中等价于单引号，但其它数据库不一定，尽量别用。

使用 AS（AS 可省略）可以给表和字段起别名，别名可以使用中文（不建议），别名如果有空格必须使用反引号包裹。

- 表的别名：如果表起了别名，那么所有用到表的地方都必须使用别名。
- 字段的别名：如果字段取了别名，只有 WHERE 后面的子句才能使用别名，WHERE 及之前的子句不能使用字段别名。
- 如果两个表有相同名称的字段，要么起别名，要么使用表名.字段名区分。

```sql
-- 查询每个部门男员工的平均薪资，筛选出男员工平均薪资>=10000的
SELECT t_d.did AS 部门编号, dname AS 部门名称, AVG(salary) AS 男员工平均薪资
FROM t_department AS t_d LEFT JOIN t_employee AS t_e ON t_d.did=t_e.did
WHERE gender='男'
GROUP BY 部门编号, 部门名称
HAVING 男员工平均薪资>=10000;
```

## 子查询

### 在 SELECT 字段中嵌入子查询

```sql
-- 查询每个员工编号、姓名、薪资，以及他所在部门平均薪资
SELECT
  t1.eid,
  t1.ename,
  t1.salary,
  -- SELECT子查询需使用()包裹
  (SELECT AVG(t2.salary) FROM t_employee AS t2 WHERE t1.did=t2.did) AS 所在部门平均薪资
FROM t_employee AS t1;
```

### 在 WHERE/HAVING 中嵌入子查询

```sql
-- 查询李冰冰所在部门的员工（如果同名，列出所有）
SELECT * FROM t_employee
WHERE did=SOME(SELECT did FROM t_employee WHERE ename='李冰冰');

-- 查询比李冰冰薪资高的员工（如果同名，列出比他们都高的）
SELECT * FROM t_employee
WHERE salary>ALL(SELECT salary FROM t_employee WHERE ename='李冰冰');

-- 查询部门平均薪资>=公司平均薪资高的部门
SELECT t_department.did, dname, AVG(salary) AS 部门平均薪资
FROM t_department LEFT JOIN t_employee ON t_department.did=t_employee.did
GROUP BY t_department.did, dname
HAVING 部门平均薪资>=(SELECT AVG(salary) FROM t_employee);
```

### 在 FROM 中嵌套查询子查询

```sql
-- 查询每个部门的编号、名称、人数
-- 子查询代替关联查询可以避免产生大笛卡尔积，效率更高
SELECT t_department.did, dname, IFNULL(部门人数, 0) AS 部门人数
FROM t_department LEFT JOIN
(
  SELECT did, COUNT(*) AS 部门人数
  FROM t_employee
  GROUP BY did
) AS t2  -- 子查询出现在 FROM 后面必须起别名
ON t_department.did=t2.did;
```

### EXISTS 子查询

```sql
-- EXISTS 子查询外层和内层没有关联字段时，子查询为 TRUE 才执行外层操作，子查询为 FALSE 不执行外层操作
-- 当员工都有部门时才统计有多少员工，否则 COUNT(*) 不计算直接返回 0
SELECT COUNT(*)
FROM t_employee
WHERE NOT EXISTS(SELECT * FROM t_employee WHERE did is NULL);

-- EXISTS 子查询外层和内层有关联字段时，把外出每一行代入子查询，有结果就保留，无结果就舍弃，类似于双层循环
-- 查询没有员工的部门编号、部门名称
SELECT did, dname
FROM t_department
WHERE NOT EXISTS(SELECT * FROM t_employee WHERE t_employee.did=t_department.did);
```

### 在 INSERT/UPDATE/DELETE 语句中嵌入子查询

```sql
-- 研发部员工涨薪10%
UPDATE t_employee 
SET salary=salary*1.1
WHERE did=(
  SELECT did FROM t_department WHERE dname='研发部'
);

-- 1号员工所在部门全体涨薪10%
-- 注意，子查询的表和要进行写操作的表是同一张表时，需要再包一层产生一张临时表
UPDATE t_employee
SET salary=salary*1.1
WHERE did=(
  SELECT * FROM (SELECT did FROM t_employee WHERE eid=1) AS t
);

-- 删除没有员工的部门
DELETE FROM t_department
WHERE NOT EXISTS(SELECT * FROM t_employee WHERE t_department.did=t_employee.did);
```

### 在 CREATE/INSERT 语句中嵌入子查询

```sql
-- 表结构和数据一起复制
CREATE TABLE dept AS (SELECT * FROM t_dept);

-- 先创建表结构，再复制数据
CREATE TABLE t_dept2 LIKE t_department;
INSERT INTO t_dept2 (SELECT * FROM t_department);

-- 只复制部分表结构和数据
CREATE TABLE t_employee2 AS (SELECT eid, ename, gender FROM t_employee);
```

### WITH

```sql
-- 查询部门平均薪资高于公司平均薪资的部门信息
SELECT
  t_department.did,
  dname,
  AVG(salary)
FROM t_department LEFT JOIN t_employee ON t_department.did=t_employee.did
GROUP BY did
HAVING AVG(salary) > (
  SELECT AVG(salary) FROM t_employee
);

-- 可以使用 WITH 定义子查询
WITH
  t1 as (SELECT AVG(salary) AS 公司平均薪资 FROM t_employee),
  t2 as (SELECT did, AVG(salary) AS 部门平均薪资 FROM t_employee GROUP BY did)
SELECT
  t_department.did,
  dname,
  部门平均薪资,
  公司平均薪资
FROM
  t_department LEFT JOIN t2 ON t_department.did=t2.did
  JOIN t1
WHERE 部门平均薪资>=公司平均薪资;

-- 递归查询5号员工的所有领导
WITH RECURSIVE t AS (
  SELECT eid, ename, mid
  FROM t_employee
  WHERE eid=5
  UNION
  -- 递归是把新发现的行喂给自己，所以这里必须是 t_employee.xxx
  SELECT t_employee.eid, t_employee.ename, t_employee.mid
  FROM t JOIN t_employee ON t.mid=t_employee.eid
)
SELECT t.eid, t.ename, t.mid, t_employee.ename
FROM t LEFT JOIN t_employee ON t.mid=t_employee.eid;

```

# 约束

## 非空约束

```sql
-- 非空约束
-- 值不能为 NULL
-- 一个表可以有多个非空约束
-- 列级约束
CREATE TABLE t_stu(
  sid CHAR(20) NOT NULL,  
  sname VARCHAR(20) NOT NULL,
  phone CHAR(20)
);
```

## 默认值约束

```sql
-- 默认值约束
-- 添加数据时没指定具体值，将采用默认值
-- 一个表可以有多个默认值约束
-- 列级约束
CREATE TABLE t_stu(
  sid CHAR(20),  
  sname VARCHAR(20),
  gender ENUM('男', '女') DEFAULT '男'
);
```

## 检查约束

```sql
-- 检查约束
-- 定义了检查约束的字段在新增和修改值时受检查约束限制
-- 一个表可以有多个检查约束
-- 列级约束
CREATE TABLE t_stu(
  sid CHAR(20),  
  sname VARCHAR(20),
  age INT CHECK(age BETWEEN 0 AND 120)
);
```

## 唯一键约束

```sql
-- 唯一键约束
-- 值不能重复
-- 一个表可以有多个唯一键约束
-- 表级约束，会写入系统表
-- 键约束，会自动创建索引
CREATE TABLE t_stu_course(
	sid CHAR(20),
    sname VARCHAR(20),
    cid CHAR(20),
    cname VARCHAR(20) UNIQUE KEY,  -- 单个字段添加唯一键约束
    score FLOAT,
    UNIQUE KEY(sid, cid)  -- 复合唯一
);
```

## 主键约束

```sql
-- 主键约束
-- 值不能为 NULL 且不能重复
-- 一个表只能有一个主键约束，且每个表应该有一个主键约束
-- 表级约束，会写入系统表
-- 键约束，会自动创建索引
CREATE TABLE t_stu(
  sid CHAR(20) PRIMARY KEY,  -- 单个字段作为主键
  sname VARCHAR(20),
  phone CHAR(20)
);

CREATE TABLE t_stu_course(
	sid CHAR(20),
    sname VARCHAR(20),
    cid CHAR(20),
    cname VARCHAR(20)
    score FLOAT,
    PRIMARY KEY(sid, cid)  -- 复合主键
);
```

## 数据库设计三大范式

第一范式：每个列具有原子性。

第二范式：满足一范式基础上，一个表有且仅有一个主键，非主键列必须完全依赖主键，不能部分依赖主键。

第三范式：满足二范式基础上，非主键列必须直接依赖主键，不能有传递依赖。

## 自增属性

```sql
-- 定义自增属性必须是键约束（一般是主键、唯一键），数据类型必须是整数型
-- 一个表只能有一个自增属性
CREATE TABLE t_stu(
  sid INT PRIMARY KEY AUTO_INCREMENT,
  sname VARCHAR(20),
  phone CHAR(20)
);

-- NULL,不指定, 0 将会自增
INSERT INTO t_stu VALUES(NULL, '张三', '10086');
INSERT INTO t_stu(sname) VALUES('李四');
INSERT INTO t_stu(sid, sname) VALUES(0, '王五');
INSERT INTO t_stu(sid, sname) VALUES(0, '赵六');

-- 如果有指定值，则使用指定值
INSERT INTO t_stu(sid, sname) VALUES(-2, '熊大');
-- 未指定值，自增从当前最大值开始自增，如果没有z则从 1 开始
INSERT INTO t_stu(sname) VALUES('熊二');

-- 自增可能越界，这时你应该使用uuid等其他方案
```

## 外键约束

```sql
-- 外键约束时指一张表的键参考另一张表的键约束（一般是主键或唯一键）
-- 如果两张表有关联关系，就可以定义外键约束
-- 外键约束不限制关联查询，只会限制增删改操作
-- 表级约束，会写入系统表
-- 键约束，会自动创建索引
CREATE TABLE t_stu(
  sid INT PRIMARY KEY AUTO_INCREMENT,
  sname VARCHAR(20) NOT NULL
);

CREATE TABLE t_course(
  cid INT PRIMARY KEY AUTO_INCREMENT,
  cname VARCHAR(20) NOT NULL UNIQUE KEY
);

CREATE TABLE t_stu_course(
  sid INT,
  cid INT,
  score FLOAT CHECK(score BETWEEN 0 AND 100),
  PRIMARY KEY(sid, cid),
  -- 默认外键的 ON DELETE 和 ON UPDATE 策略是严格模式
  -- ON UPDATE/DELETE CASCADE 更新/删除被引用的键时，所有关联的地方都更新/删除
  -- ON UPDATE/DELETE SET NULL 更新/删除被引用的键时，所有关联的地方都设为 NULL
  -- ON UPDATE/DELETE NO ACTION 更新/删除被引用的键时，所有关联的地方都不动
  FOREIGN KEY(sid) REFERENCES t_stu(sid) ON UPDATE CASCADE ON DELETE CASCADE,
  FOREIGN KEY(cid) REFERENCES t_course(cid) ON UPDATE CASCADE ON DELETE CASCADE
);

INSERT INTO t_stu VALUES(1, '张三');
INSERT INTO t_stu VALUES(2, '李四');
INSERT INTO t_course VALUES(1, 'python');
INSERT INTO t_course VALUES(2, 'mysql');
-- 添加时，外键值必须在被引用的表中
INSERT INTO t_stu_course VALUES(1, 1, 80);
INSERT INTO t_stu_course VALUES(1, 2, 90);
INSERT INTO t_stu_course VALUES(2, 1, 88);
INSERT INTO t_stu_course VALUES(2, 2, 85);

DELETE FROM t_course WHERE cid=1;
UPDATE t_course SET cid=3 WHERE cid=2;
```

# 事务

## 基本概念

事务是指一组 sql 语句组成一个原子性单元，它们一起成功提交，或者一起失败回滚。

- A（原子性）
- C（一致性）
- I（隔离性）
- D（持久性）

mysql 事务的回滚只对 DML 语句有效，对 DDL 语句无效。

## 开启事务

```sql
-- 默认情况下，一条SQL语句就是一个事务，自动提交或回滚

-- 开启事务，把多条语句组成事务，需要手动提交或回滚
-- 这种方式仅限于客户端，不支持Java, Python等语言操作
START TRANSACTION;
-- SQL 语句1
-- SQL 语句2
COMMIT / ROLLBACK;

-- 后面的SQL都是手动提交模式
SET AUTOCOMMIT=FALSE;
-- SQL 语句3
-- SQL 语句4
COMMIT / ROLLBACK;
```

## 事务隔离级别

脏读：一个事务读取了另一个事务未提交数据。

不可重复读：针对同一行，一个事务两次查询结果不一样。

幻读：一个事务同样的查询，结果集行数不一样。

| 隔离级别                      | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| read-uncommitted              | 会出现脏读、不可重复读、幻读问题。                           |
| read-committed                | 避免脏读，但仍然会出现不可重复读、幻读问题。                 |
| repeatable-read（mysql 默认） | 避免脏读、不可重复读。但会有幻读（mysql 能极大程度避免幻读），使用行锁（不能同时对同一行写操作）。 |
| serializable                  | 完全没有任何问题，使用表锁（不能同时对同一张表进行写操作），性能十分低下。 |



