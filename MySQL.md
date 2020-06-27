# MySQL

## DQL语言

### 基础查询

语法：

```sql
select 查询列表
from 表名;

# 列表可以是：字段、常量值、表达式、函数
# 查询的结果是一个虚拟的表格

# 1.查询单个字段
select last_name from employees;

# 2.查询多个字段
select last_name, salary, email from employees;

# 3.查询所有字段
select * from employees;

# 4.查询常量
select 100;

# 5.查询表达式
select 100%2;

# 6.查询函数
select VERSION();

# 7.起别名
select 100 (as) num; // as可加可不加

# 8.去重，查有哪些部门编号
select department_id from employees;
--
select distinct department_id from employees;

# 9.+号的作用，只有运算的作用，操作数转数值如果失败则为0，下面返回0
# 查询员工姓名连成一个字段并显示为姓名
select last_name+first_name as name from employees; //xx

# 使用concat
select concat(last_name, first_name) as name from employees;

# IFNULL(字段, 为空时的默认值)
select ifnull(commission_pct, 0), commission_pct from employees;




```

### 条件查询

语法：

```sql
select 查询列表 from table where conditon;

-- 1. 按条件表达式筛选：> < = <>(!=) >= <=
select * from employees where salary > 12000;
select last_name, dept_id from employees where dept_id <> 90;
-- 2. 按逻辑表达式筛选：&& ^^ !  and or not
-- 用于连接条件表达式
select last_name,salary from employees where salary>=10000 and salary<=20000;
select * from employees where dept_id < 90 or dept_id>110 or salary > 15000;

-- 3. 模糊查询：like, between and, in, is null
select * from employees where last_name like '%a%';
-- like一般与通配符搭配：%匹配任意多个字符，_匹配任意一个字符
select * from employees where last_name like '__n_l';
select * from employees where last_name like '_\_n_l'; 第二个_转义
-- between and 左闭右闭，左右不可以颠倒顺序
select * from employees where employee_id between 0 and 100;

-- in
select * from employees where job_id in ('IT_PROT', 'AD_VP');

-- is null, =不能用于判断null值
select * from employees where commission_pct = NULL;  //xx 行不通
select * from employees where commission_pct is null;

-- 安全等于 <=>
-- 可以判断null值，容易混淆
select * from employees where commission_pct <=> null;

select * from employees where salary <=> 12000;


```

![image-20200606205549118](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200606205549118.png)

如果是or，那么结果一样（假设两个字段总有一个不为null）

### 排序查询

```sql
select 查询列表
from 表
【where】
order by 排序列表 【asc｜desc】 asc可以省略

-- 默认asc升序、desc降序

select * from employees order by salary desc;

select * from employees where department_id >= 90 order by hiredate asc;

select *,salary*12*(1+ifnull(commission_pct, 0)) 年薪 from employees order by 年薪;

-- 按姓名长度排序
select length(last_name) len, last_name from employees order by len;

-- 先按工资排序，再按员工编号排序
select * from employees order by salary asc, employee_id desc;
```

- Order by 子句中可以支持单个字段、多个字段、表达式、函数、别名
- order by子句一般是放在查询语句的最后面，limit子句除外

### 函数

#### 单行函数

```sql
-- 隐藏实现细节，提高代码的重用性

-- 直接调用 select func()...

-- 单行函数：concat length ifnull等
-- 分组函数：做统计使用

-- 1. 字符函数：
-- length
select length('abc'); // 3
select length('你好'); // 6

-- concat拼接字符串
select concat(last_name, first_name);

-- upper\lower 大小写
select upper('aaa');

-- substr/substring 截取，索引从1开始
select substr('aaaa',2); //aaa
select substr('aaaa',2, 2); //aa， 2为开始，2长度（字符长度）

-- 首字母大小，其他小写
select concat(upper(substr(last_mame,1,1)),'_',lower(substr(last_name,2))) as ..;

-- instr() 返回子串的起索引
select instr('aaabcjskdjabc','abc');

-- trim() 去左右空格
select trim('  aaa  ');
select trim('a' from 'aaaahhhhhaaaaa')..

-- lpad 左填充到指定长度，用*填aaa
select lpad('aaa', 10, '*');

-- rpad

-- replace 匹配到所有的都会替换
select replace('abcdfg','abc','zzz');


```

数学函数：

```sql
-- round(x) round(x, d) 四舍五入,d表示保留位数

-- ceil(x) 向上取整

-- floor(x) 向下取整

-- truncate(x,d) 截断

-- mod(n,m) 取余

```

日期函数：

```sql
-- now() 返回当前系统日期时间
-- curdate() 当前日期
-- curtime() 当前时间
-- year()获取年
-- month()
-- monthname()
-- day
-- hour
-- minute
-- second
-- str_to_date
-- date_format 日期转字符

-- 查询1992-4-3入职的
select * from employees where hiredate = '1992-4-3';
select * from employees where hiredate = str_to_date('4-4 1992', '%c-%d %Y');


```

![image-20200606215243456](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200606215243456.png)

![image-20200606215251848](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200606215251848.png)

其他函数：

```sql
-- version()
-- database();
-- user();
```

流程控制函数：

```sql
-- if
select if(10 < 5, '大','小')

-- case结构
case 字段/表达式
when 常量1 then 值 或语句;
when 常量2 then 值 或语句;
when 常量3 then 值 或语句;
else 值 或语句;
end

-- 查询员工的工资，如果dpid=30，显示工资1.1倍，=40，1.2，=50，1.3
select salary y, dp_id, 
case dpid
when 30 then salary*1.1
when 40 then salary*1.2
when 50 then salary*1.3
else salary
end as xin
from employees;

-- 多重case
case
when 条件1 then 值/语句
when 条件2 then 值/语句
when 条件3 then 值/语句
else 值/语句
end
```

#### 分组函数

```sql
-- sum\avg\max\min\count
-- count 统计非空
-- sum/avg 支持：数值型，忽略了null
-- max/min/count 支持：数值、字符串、日期，非空的，忽略null

-- count(distinct salary)  都可以与distinct结合使用

-- count(salary)  /忽略null
-- count(*) // 某个行只要有任何字段不为null就统计

-- count(1) // 相当于统计行数，写任何常量都可以

-- 效率：
-- MYISAM count(*)高，直接返回存储
-- INNODB count(*) count(1) 高，count(字段)的话需要判断是否null

-- 和分组函数 一同查询的字段有限制
select avg(salary), emipd from employees; //xx
-- 和分组一起用的要求是group by



-- 查相差天数
select datediff(max(), min()) ...
```

### 分组查询

GROUP BY，支持单个、多个字段，没有顺序要求，逗号隔开，也可以表达式或函数，相对用的少；也可以加排序，放在最后。

筛选条件：

- 分组前筛选：where，原始表已有字段
- 分组后筛选：having，分组后生成的表，分组函数做条件肯定放在having子句中
- 能用分组前筛选就优先分组前筛选

分组条件：

- 按字段

- 按表达式

```sql
select 分组函数，列（要求出现在groupby后面）
from table
where xxx
gourp by 分组列表
order byxx..;

-- 查询每个工种的最高工资
select max(salary), job_id from employees group by job_id;

-- 查询每个位置的部门个数
select count(*), location_id from departments group by location_id;

-- 查询邮箱中包含a的，每个部门的平均工资
select avg(salary), department_id
from employees
where email like '%a%'
group by department_id;

-- 查询有奖金的每个领导手下员工的最高工资
select max(salary), manager_id
from employees
where commission_pct is not null
group by manager_id;

-- 查询人》2，分组后的筛选，having
select department_id, count(*)
from employees
group by department_id
having count(*) > 2;

-- 查询每个工种有奖金的员工的最高工资大于12000的工种编号和最高工资
select job_id, max(salary)
from employees
where commission_pct is not null
group by job_id
having max(salary) > 12000;

-- 查询领导编号大于102的手下的最低工资大于5000的领导编号以及最低工资
select manager_id, min(salary)
from employees
where manager_id > 102
group by manager_id
having min(salary) > 5000;

-- 按姓名长度分组，查询每一组员工个数，筛选大于5个的

select count(*), length(last_name)
from employees
group by length(last_name)
having count(*) > 5;

-- 按多个字段分组
-- 查询每个部门每个工种的员工的平均工资
select avg(salary), department_id, job_id
from employees
group by department_id, job_id;
-- 添加排序
select avg(salary), department_id, job_id
from employees
where department_id is not null
group by department_id, job_id
order by avg(salary);
-- 可以用别名哦
```

### 连接查询

又多表查询、多表连接。

#### 笛卡尔积

完全连接，不加任何条件。

Select name, boyName from boys, beauty;

12x4

加上连接条件就可以避免。

#### emm

```sql
select name, boyName 
from boys, beauty
where boys.id = beauty.boyfrient_id;
```

分类：

- 内连接
  - 等值
  - 非等值
  - 自连接
- 外连接
  - 左外
  - 右外
  - 全外
- 交叉连接

标准：

- Sql192：仅仅内连接
- Sql199：支持内连接，左外和右外，交叉

#### sql92内连接

等值：

```sql
-- 等值
select name, boyName
from boys, beauty
where boys.id = beauty.boyfriend_id;

select last_name, department_name 
from employees,departments 
where employees.department_id = departments.department_id;

-- 查询员工名、工种号、工种名，用了别名再用原名不好使
select last_name, e.job_id, job_title 
from employees as e, jobs as j 
where e.job_id = j.job_id;

-- 加筛选
-- 查询有奖金的员工名和部门名
select last_name, department_name
from employees e, departments d
where e.department_id=d.department_id
and e.commission_pct is not null;

-- 查询城市名中第二个字符位o的部门名和城市名
select department_name, city
from departments d, locations l
where d.location_id=l.location_id
and city like '_o%';


-- 查每个城市的部门个数
select count(*) sum,city
from departements d, locations l
where d.location_id=l.location_id
group by city;

-- 查询有奖金的每个部门的部门名和部门领导编号和该部门的最低工资，manager——id有问题
select department_name, d.manager_id, min(salary)
from departments d, employees e
where d.department_id=e.department_id
and commission_pct is not null
group by d.department_name;

-- 加排序
-- 查询每个工种的工种名和员工个数，并排序
select job_title, count(*)
from employees e, jobs j
where e.job_id=j.job_id
group by job_title
order by count(*);

-- 三表连接
-- 查询员工名、部门名、和城市
select last_name,department_name,city
from employees e,departments d,locations l
where e.department_id=d.department_id
amd d.location_id=l.location_id;
```

非等值：

```sql
-- 连接条件非等于
-- 查询员工的工资和工资级别
select salary,grade_level
from employees e, job_grades g
where e.salary between g.lowest_sal and g.highest_sal;
```

自连接：

```sql
-- 相当于等值，在一个表内
-- 查询员工名、和上级的名字
select e.employee_id, e.last_name, m.employee_id, m.last_name
from employees e, employees m
where e.manager_id = m.employee_id;

```



#### sql99内连接与外连接

sql99的内连接与外连接的语法：

Select 查询列表

from 表1 别名 连接类型 

join 表2 别名 on 连接条件

[where 筛选条件]

[goup]

[having]

[order by]



分类：

- inner
- Left (outer)
- right(outer)
- full(outer)
- cross

```sql
-- 内连接，可以换顺序,inner可以省略
select 立标
from 表1 别名
inner join 表2 别名
on

-- 查询员工名、部门名
select last_name ,department_name
from employees e
inner join departments d
on e.department_id= d.department_id;

-- 查询名字中包含e的员工名和工种名
select last_name,job_title
from employees e
inner join jobs j
on e.job_id=j.job_id
where e.last_name like '%e%';

-- 查询部门个数>3的城市名和部门个数
select city,count(*)
from department d
inner join locations l
on d.location_id=l.location_id
group by city
having count(*) > 3;

-- 查询部门员工个数大于3的部门名和员工个数，并排序
select count(*), department_name
from employees e
inner join departments d
on e.department_id=d.department_id
group by department_name
having count(*) > 3
order by count(*);

-- 查询员工名、部门名、工种名
select last_name,department_name,job_title
from employees e
inner join departments d on e.department_id=d.department_id
inner join jobs j on e.job_id=j.job_id
order by department_name desc;

-- 非等值，，就不说了
-- 自连接也不说了
```

```sql
-- 左外（右外）：一般一个表，另一个表没有，拿一个表去匹配另一个表，主表（左/右）都要显示，从表匹配显示匹配值，没有匹配的，显示null。

select b.name, bo.*
from beauty b
left outer join boys bo
on b.boyfriend_id=bo.id;
[where bo.id is not null]

-- 全外,innodb不支持,各为主表，都要完全显示=内连接+表1有表2没+表2有表1没
select b.*,bo.*
from beauty b
full outer join boys bo
on b.boyfriend_id=bo.id;



```

#### 交叉连接

```sql
select b.*, bo.*
from beauty b
cross join boys bo;  相当于82的笛卡尔积

select b.*,bo.*
from beauty b, boys bo;
```

### 子查询

出现在其他语句中的select语句，称为子查询。

位置：

- select后面：标量子查询
- from后面：表子查询
- where或having后面：**标量**、**列**、行 
- exists后面（相关子查询）：表子查询

功能：

- 标量子查询：结果一行一列
- 列子查询：一列多行
- 行子查询：多行多列

- 表子查询：一般为多行多列

### 分页查询

limit offset, size；索引此处从0开始

### 联合查询

将多条查询语句的结果合并为一个结果。

#### where/having后面

- 标量子查询（单行）

```sql
-- 谁的工资比abel工资高
select salary
from employee
where last_name='Abel';

select * from employees
where salary > (
    select salary
	from employee
	where last_name='Abel';
);
```

- 列子查询（多行）![image-20200607195015080](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607195015080.png)

```sql

```



- 行子查询（多列多行）



## DML语言



## DOL语言

### 约束

六大约束：

- 非空约束：NOT NULL
- 默认约束：DEFAULT
- 主键约束：PRIMARY KEY
- 唯一约束：UNIQUE
- 检查约束：CHECK，mysql不支持，语法支持
- 外键约束：FOREIGN KEY，保证该字段的值必须来自于主表的关联列的值

列级约束：

- 六大约束都可以写，语法上支持，但外键约束没效果

表级约束：

- 除了非空、默认，都支持

![image-20200607203925197](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607203925197.png)

列级+表级



主键和唯一：

- 都可以保证唯一性
- 主键不允许为空，唯一允许空（不能两个空，只能一个空）
- 主键只能1个，唯一键可以多个
- 都可以使用组合键

## TCL语言

事务控制

- 隐式事务：比如insert、update、delete
- 显式事务：先禁用自动提交功能。set autocommit=0;

- set autocommit=0;
- Start transaction;// 可选
- 语句1
- 语句2.。
- commit;
- rollback;

## 视图



## 存储过程

## 一些原理（旧）

#### B+Tree原理（MySQL的索引结构）

##### 数据结构

B Tree指Balance Tree，也就是平衡树。平衡树是一颗查找树，并且所有的叶子节点位于同一层。

B+Tree是B树的变形，它是**更适实际应用中操作系统的文件索引和数据库索引**的一种用于查找的数据结构。B+Tree是基于B Tree和叶子节点顺序访问指针进行实现，它有BTree的平衡性，并且通过顺序访问指针来提高区间查询的性能。

定义：

- 除根节点外的内部节点，每个节点最多有m个关键字，最少有m/2向上取整个关键字，其中每个关键字对应一个子树（也就是最多有m颗子树，最少有m/2向上取整颗子树）
- 根节点要么没有子树，要么至少有两颗子树
- 所有叶子节点包含了全部的关键字以及这些关键字指向文件的指针，且：
  - 所有叶子节点中的关键字按大小顺序排序
  - 相邻的叶子节点顺序连接（相当于是构成了一个顺序链表）
  - 所有叶子节点在同一层

![image-20200211121017206](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200211121017206.png)

##### 操作

查找时，首先在根节点进行二分查找，找到一个key所在的指针，然后递归地在指针所只想的节点进行查找。直到查找到叶子节点，然后在叶子节点上进行二分查找，找出key所对应的data。

插入删除操作会破坏平衡树的平衡性，因此在插入删除操作之后，需要对树进行一个分裂、合并、旋转等操作来维护平衡性。

##### 与红黑树的比较

红黑树等平衡树也可以用来实现索引，但是文件系统及数据库普遍采用B+ Tree作为索引结构，原因：

1. 更少的查找次数：平衡树查找操作的时间复杂度和树高h有关，O(h)=O(logdn)，其中d为每个节点的出度。红黑树的出度为2，而B+ Tree的出度一般都非常大，所以红黑树的树高h很大，查找次数也就更多。
2. 利用磁盘预读特性：所有叶子节点相连，磁盘预读采用顺序读取，不需要寻道，速度飞快。数据库系统将索引的一个节点的大小设置为页的大小，使得一次IO能完全载入一个节点，利用预读特性，相邻节点能够被预先载入。

##### 额外的比较

为什么不用AVL树、红黑树呢

- 这两种结构在插入数据后，一般要调整树的结构来实现平衡，比较耗性能
- 最主要原因：二叉树查找的效率与树的高度相关，每次查找都是一次磁盘IO，而B+树h很低，查找次数很少。
- 结构特性，所有关键字和关键字的指针位于叶子节点，而叶子节点位于同一层，且相邻的叶子节点数按序链接，进行范围查找或者是全表查找（扫库）时非常方便；同时由于非叶子节点只存放索引信息，不存放数据信息，每个非叶子节点存放的索引信息就更多，一次磁盘IO就可以读取更多的索引信息到内存中，可以减少磁盘IO的次数。
- 查询效率稳定：任何关键字的查找必须走一条从根到叶子节点的路，路径长度相同，导致数据查询的效率相当
- ////////////

- **B+树的磁盘读写代价更低**，因为B+树的所有**非叶子节点只会存放索引信息**，而**真正的数据信息都只存放在叶子节点**中，这样一来，每个非叶子节点存放的索引信息就更多，一次磁盘IO就可以读取更多的索引信息到内存中，可以减少磁盘IO的次数。
- **B+树的查询效率更加稳定**，由于非叶子节点只存索引信息，而没有真正的数据信息，所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。
- **B+树更加适合在区间查询的情况**，由于B+树的数据都存储在叶子结点中，非叶子结点均为索引，只需要扫一遍叶子结点即可得到所有数据信息，但是B树因为其非叶子结点同样存储着数据，我们要找到具体的数据，需要进行一次中序遍历按序来扫，所以B+树更加适合在区间查询的情况，所以通常B+树用于数据库索引。

##### B树（B-树）

性质：多路搜索树（不是二叉）

1. 定义人意非叶子节点最多只有M个儿子；M>2
2. 根节点的儿子数为[2,M]；
3. 除根节点以外的非叶子节点的儿子数为[M/2，M]
4. 每个节点存放至少M/2-1向上取整和之多M-1个关键字（至少两个）
5. 非叶子节点的关键字个数=指向儿子的指针个数-1；
6. 非叶子节点的关键字：K[1]，K[2]，……，K[M-1]；且K[i]<K[i+1]；
7. 非叶子节点的指针：P[1]，P[2]，……，P[M]；其中P[1]指向关键字小于K[1]的子树，P[M]指向关键字大于K[M-1]的子树，其他P[i]指向属于(K[i-1]，K[i])的子树；
8. 所有叶子节点位于同一层

![image-20200211130427058](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200211130427058.png)

注意：红色方块代表文件的存储位置，即地址；P是指针，指针、关键字、以及关键字所代表的文件地址合起来构成B树的一个节点，这个节点存储在一个磁盘块上。

##### 搜索

搜索：从根节点开始，对节点内的关键字（有序）序列进行二分查找，如果命中则结束，否则进入查询关键字所属范围的儿子节点；重复，直到所对应的儿子节点指针为空，或已经是叶子节点；

##### B+树与B-树的区别

1. B+树内部节点中，关键字的个数与其子树的个数相同；B-树中子树的个数总比关键字个数多1；
2. B+树所有指向文件的关键字及其指针都在叶子节点中；B-树有的指向文件的关键字是在内部节点中。换句话说，B+树的内部节点仅仅起到索引的作用；
3. B+树在搜索过程中，如果查询和内部节点的关键字一致，那么搜索过程不停止，继续向下搜索这个分支；

#### MySQL索引

索引是在存储引擎层实现的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型实现。

##### B+Tree索引

是大多数 MySQL 存储引擎的默认索引类型。

因为不再需要进行全表扫描，只需要对树进行搜索即可，所以查找速度快很多。

因为 B+ Tree 的有序性，所以除了用于查找，还可以用于排序和分组。

可以指定多个列作为索引列，多个索引列共同组成键。

适用于全键值、键值范围和键前缀查找，其中键前缀查找只适用于最左前缀查找。如果不是按照索引列的顺序进行查找，则无法使用索引。

InnoDB 的 B+Tree 索引分为主索引和辅助索引。主索引的叶子节点 data 域记录着完整的数据记录，这种索引方式被称为聚簇索引。因为无法把数据行存放在两个不同的地方，所以一个表只能有一个聚簇索引。

![image-20200301175451545](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200301175451545.png)

辅助索引的叶子节点的 data 域记录着主键的值，因此在使用辅助索引进行查找时，需要先查找到主键值，然后再到主索引中进行查找。

![image-20200301175441064](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200301175441064.png)

##### 哈希索引

哈希索引能以 O(1) 时间进行查找，但是失去了有序性：

- 无法用于排序与分组；
- 只支持精确查找，无法用于部分查找和范围查找。

InnoDB 存储引擎有一个特殊的功能叫“自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+Tree 索引之上再创建一个哈希索引，这样就让 B+Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。

##### 全文索引

MyISAM 存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。

查找条件使用 MATCH AGAINST，而不是普通的 WHERE。

全文索引使用倒排索引实现，它记录着关键词到其所在文档的映射。

InnoDB 存储引擎在 MySQL 5.6.4 版本中也开始支持全文索引。

##### 空间数据索引

MyISAM存储引擎支持空间数据索引（R-Tree），可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

必须使用 GIS 相关的函数来维护数据。

#### 索引优化

##### 独立的列

在进行查询时，索引列不能是表达式的一部分，也不能是函数的参数，否则无法使用索引。

```
例如下面的查询不能使用actor_id列的索引
SELECT actor_id FROM askila.actor WHERE actor_id + 1 = 5;
```

##### 多列索引

在需要使用多个列作为条件进行查询时，使用多列索引比使用多个单列索引性能更好。例如下面的语句，最好把actor_id和film_id设置为多列索引。

```
SELECT film_id, actor_id FROM sakila.film_actor
WHERE actor_id = 1 AND film_id = 1;
```

##### 索引列的顺序

让选择性最强的索引列放在前面。

索引的选择性是指：不重复的索引值和记录总数的比值。最大值为1，此时每个记录都唯一的索引与其对应。选择性越高，每个记录的区分度越高，查询效率也越高。

例如下面显示的结果中customer_id的选择性比staf_id更高，因此最好把customer_id列放在多列索引的前面。

```
SELECT COUNT(DISTINCT staff_id)/COUNT(*) AS staff_id_selectivity,
COUNT(DISTINCT customer_id)/COUNT(*) AS customer_id_selectivity,
COUNT(*) FROM payment

staff_id_selectivity: 0.0001
customer_id_selectivity: 0.0373
               COUNT(*): 16049
```

##### 前缀索引

对于BLOB、TEXT和VARCHAR类型的列，必须使用前缀索引，只索引开始的部分字符。

前缀长度的选取需要根据索引选择性来确定。

##### 覆盖索引

索引包含所有需要查询的字段的值。

具有以下优点：

- 索引通常小于数据行的大小，只读取索引能大大减少数据访问量
- 一些存储引擎（例如MyISAM）在内存中只缓存索引，而数据依赖于操作系统来缓存。因此，只访问索引可以不使用系统调用（通常比较耗时）
- 对于InnoDB引擎，若辅助索引能够覆盖查询，则无需访问主索引

#### 索引的优点

- 大大减少了服务器需要扫描的数据行数。
- 帮助服务器避免进行排序和分组，以及避免创建临时表（B+Tree索引时有序的，可以用于ORDER BY和GROUP BY操作。临时表主要是在排序和分组过程中创建，不需要排序和分组，也就不需要创建临时表）
- 将随机I/O变为顺序I/O（B+Tree索引是有序的，会将相邻的数据都存储在一起）。

#### 索引的使用条件

- 对于非常小的表、大部分情况下简单的全表扫描比建立索引更高效
- 对于中到大型的表，索引就非常有效
- 但是对于特大的表，建立和维护索引的代价将会随之增长。这种情况下，需要用到一种技术可以直接区分出需要查询的一组数据，而不是一条记录一条记录地匹配，例如可以用分区技术。

#### 查询性能优化

##### 使用Explain进行分析

Explain用来分析SELECT查询语句，开发人员可以通过分析Explain结果来优化查询语句。

比较重要的字段：

- Select_type：查询类型，有简单查询、联合查询、子查询等
- key：使用的索引
- rows：扫描的行数

##### 优化数据访问

**减少请求的数据量**

- 只返回必要的列：最好不要使用 SELECT * 语句。
- 只返回必要的行：使用 LIMIT 语句来限制返回的数据。
- 缓存重复查询的数据：使用缓存可以避免在数据库中进行查询，特别在要查询的数据经常被重复查询时，缓存带来的查询性能提升将会是非常明显的。

**减少服务器端扫描的行数**

最有效的方式是使用索引来覆盖查询

##### 重构查询方式

**切分大查询**

一个大查询如果一次性执行的话，可能一次锁住很多数据、占满整个事务日志、耗尽系统资源、阻塞很多小的但很重要的查询。

```sql
DELETE FROM messages WHERE create < DATE_SUB(NOW(), INTERVAL 3 MONTH);
------------
rows_affected = 0
do {
    rows_affected = do_query(
    "DELETE FROM messages WHERE create  < DATE_SUB(NOW(), INTERVAL 3 MONTH) LIMIT 10000")
} while rows_affected > 0
```

**分解大连接查询**

将一个大连接查询分解成对每一个表进行一次单表查询，然后在应用程序中进行关联，这样做的好处是：

- 让缓存更高效。对于连接查询，如果其中一个表发生变化，那么整个查询缓存就无法使用。而分解后的多个查询，即使其中一个表发生变化，对其他表的查询缓存仍人可以使用。

- 分解成多个单表查询，这些单表查询的缓存结果可能被其他查询使用到，从而减少冗余记录的查询。

- 减少锁竞争。

- 在应用层进行连接，可以更容易对数据库进行拆分，从而更容易做到高性能和可伸缩性。

- 查询本身效率也可能会有所提神。例如下面的例子中，使用IN()代替连接查询，可以让MySQL按照ID顺序进行查询，这可能比随机的连接更高效。

  ```sql
  SELECT * FROM tag
  JOIN tag_post ON tag_post.tag_id=tag.id
  JOIN post ON tag_post.post_id=post.id
  WHERE tag.tag='mysql';
  ```

  ```sql
  SELECT * FROM tag WHERE tag='mysql';
  SELECT * FROM tag_post WHERE tag_id=1234;
  SELECT * FROM post WHERE post.id IN (123,456,567,9098,8904);
  ```

#### 存储引擎

##### InnoDB

是MySQL默认的事务型存储引擎，只有在需要它不支持的特性时，才考虑使用其他存储引擎。

实现了四个标准的隔离级别，默认级别是可重复读（REPEATABLE READ）。在可重复读隔离级别下，通过多版本并发控制（MVCC）+Next-Key Locks防止幻读。

主索引是聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有很大提升。

内部做了很多优化，包括从磁盘读取睡时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。

支持真正的在线热备份。其他存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取。

##### MyISAM

设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。

提供了大量的特性，包括压缩表、空间数据索引等。

不支持事务。

不支持行级锁，只能对整张表进行加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排他锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入（CONCURRENT INSERT）。

可以手工或者自动执行检查和修复操作，但是和事务恢复以及奔溃恢复不同，可能导致一些数据丢失，而且修复操作是非常慢的。

如果指定了DELAY_KEY_WRITE选项，在每次修改执行完成时，不会立即将修改的索引数据写入磁盘，而是会写到内存中的键缓冲区，只有在清理缓冲区或者关闭表的时候才会将对应的索引块写入磁盘。这种方式可以极大提升写入性能。但是在数据库或主机奔溃时会造成索引损坏，需要执行修复操作。

##### 比较

- 事务：InnoDB是事务型的，可以使用Commit和Rollback语句。
- 并发：MyISAM只支持表级锁，而InnoDB还支持行级锁。
- 外键：InnoDB支持外键
- 备份：InnoDB支持在线热备份
- 奔溃恢复：MyISAM奔溃后发生损坏的概率比InnoDB高很多，而且恢复的速度也很慢。
- 其他特性：MyISAM支持压缩表和空间数据索引。

#### 数据类型

##### 整型

TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT分别使用8、16、24、32、64位存储空间，一般情况下越小的列越好。

INT（11）中的数字只是规定了交互工具显示字符的个数，对于存储和计算来说是没有意义的。

##### 浮点数

FLOAT和DOUBLE为浮点类型，DECIMAL为高精度小数类型。CPU原生支持浮点运算，但是不支持DECIMAL类型的计算，因此DECIMAL的计算比浮点类型需要更高的代价。

FLOAT、DOUBLE和DECIMAL都可以指定列宽，例如DECIMAL（18，9）表示总18位，取9位存储小数，剩下9位存储整数部分。

##### 字符串

主要有CHAR和VARCHAR两种类型，一种是定长的，一种是变长的。

VARCHAR这种变长类型能够节省空间，因为只需要存储必要的内容。但是在执行UPDATE时可能会使行变得比原来长，当超出一个页所能容纳的大小时，就需要执行额外的操作。MyISAM会将行拆成不同的片段存储，而InnoDB则需要分裂页来使行放进页内。

在进行存储和检索时，会保留VARCHAR末尾的空格，而会删除CHAR末尾的空格。

#### 时间和日期

MySQL提供了两种相似的日期时间类型：DATETIME和TIMESTAMP。

##### DATETIME

能够保存从1000年到9999年的日期和时间，精度为秒，使用8字节的存储空间。

它与时区无关。

默认情况下，MySQL以一种可排序的、无歧义的格式显示DATETIME值，例如“2009-01-16 22:27:06"，这是ANSI标准定义的日期和时间表示方法。

##### TIMESTAMP

和UNIX时间戳相同，保存从1970年1月1日午夜（格林威治时间）以来的秒数，使用4个字节，只能表示从1970年到2038年。

它和时区有关，也就是说一个时间戳在不同的时区所代表的具体时间是不同的。

MySQL提供了FROM_UNIXTIME()函数把UNIX时间戳转换为日期，并提供了UNIX_TIMESTAMP()函数把日期转换为UNIX时间戳。

默认情况下，如果插入时没有指定TIMESTAMP列的值，会将这个值设置为当前时间。

应该尽量使用TIMESTAMP，因为它比DATETIME空间效率更高。

#### 切分

##### 水平切分

水平切分又称为Sharding，它是将同一个表中的记录拆分到多个结构相同的表中。

当一个表的数据不断增多时，Sharding是必然的选择，它可以降数据分布到集群的不同节点上，从而缓解单个数据库的压力。

![image-20200302105815155](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200302105815155.png)

##### 垂直切分

垂直切分是将一张表按列拆分成多个表，通常是按照列的关系密集程度进行切分，也可以利用垂直切分将经常被使用的列和不经常被使用的列切分到不同的表中。

在数据库的层面使用垂直切分将按数据库中表的密集程度部署到不同的库中，例如将原来的电商数据库垂直切分成商品数据库、用户数据库等。

##### ![image-20200302105958837](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200302105958837.png)

##### Sharding策略

- 哈希取模：hash(key) % N；
- 范围：可以是ID范围也可以是时间范围；
- 映射表：使用单独的一个数据库来存储映射关系。

##### Sharding存在的问题

**事务问题**

使用分布式事务来解决，比如XA接口。

**连接**

可以将原来的连接分解成多个单表查询，然后在用户程序中进行连接

ID**唯一性**

- 使用全局唯一ID（GUID）
- 为每个分片指定一个ID范围
- 分布式ID生成器（如Twitter的Snowflake算法）

#### 复制

##### 主从复制

主要涉及三个线程：binlog线程、I/O线程、SQL线程

- binlog：负责将主服务器上的数据更改写入二进制日志（Binary log）
- I/O：负责从主服务器上读取二进制日志，并写入从服务器的中继日志（Relay log）
- SQL：负责读取中继日志，解析出主服务器已经执行的数据更改并在从服务器中重放（Replay）。

![image-20200302110443944](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200302110443944.png)

##### 读写分离

主服务器处理写操作以及实时性要求比较的写操作，而从服务器处理读操作。

读写分离能提高性能的原因：

- 主服务器负责各自的读和写，极大程度缓解了锁的争用；
- 从服务器可以使用MyISAM，提升查询性能以及节约系统开销；
- 增加冗余，提高可用性。

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。

![image-20200302110657569](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200302110657569.png)

## MySQL高级

### MySQL架构

MySQL默认没密码，按照server提示，修改密码。

```shell
/usr/bin/mysqladmin -u root password 123456
```

自启动

```shell
service mysql start  //单次启动

chkconfig mysql on //设置自启动
```

文件位置

![image-20200607210942192](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607210942192.png)

修改配置文件位置：

My-hug(default).cnf ---->. /etc/my.cnf

修改字符集：

- 查看字符集：show variables like '%char%'

- 修改：

  ```shell
  [client]
  default-character-set=utf8
  
  [mysqld]
  character_set_server=utf8
  character_set_client=utf8
  collation-server=utf8_general_ci
  [mysql]
  defualt-character-set=utf8
  
  之后新建的库才是utf8
  
  ```

日志文件：

- 二进制日志：主从复制
- 错误日志
- 查询日志
- 数据文件
  - frm存放表结构
  - myd存数据
  - myi存索引

![image-20200607212353278](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607212353278.png)

连接层、服务层、引擎层、物理层

引擎：

- 查看：show engines;

![image-20200607213456267](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607213456267.png)

- 互联网公司：
  - 阿里，基于Percona原型改进，XtralDB引擎
  - AliSql+AliRedis

### 索引优化分析

#### 性能下降SQL慢

- 执行时间长
- 等待时间长

原因：

- 查询语句写的不好
- 索引失效
  - 单值索引：create index idx_user_name on user(name);
  - 复合索引：create index idx_user_nameRmail on user(name, email)
- 关联查询太多join（设计缺陷或不得已）
- 服务器调优及各个参数设置（缓冲、线程数等）

#### 常见的Join查询

SQL执行顺序：

- 手写
- 机读：先从from读

![image-20200607214503603](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607214503603.png)

join图![image-20200607214601627](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607214601627.png)

![image-20200607214810872](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607214810872.png)

![image-20200607214841167](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607214841167.png)

#### 索引

##### 什么是索引

索引是帮助MySQL高效获取数据的数据结构，索引的本质是**数据结构**。

目的在于提高查询效率，可以类比于字典。

**可以理解为“排好序的快速查找数据结构”。**

数据之外，数据库还维护着一个满足特定查找算法的数据结构，这些数据结构以某种方式指向数据，这样就可以在这些数据结构的基础上实现高级查找算法，这种数据结构是索引。

一般来说索引本身也很大，往往以索引文件存储在磁盘。

我们平时说的索引，如果没有特别指明，都是BTREE结构组织的索引。其中聚集索引、次要索引、覆盖索引、复合索引、前缀索引、唯一索引都默认BTREE，统称索引。

##### 索引的优势

- 提高数据检索效率，降低数据库的IO成本
- 通过索引对数据进行排序，降低数据排序的成本，降低了CPU的消耗

##### 索引的劣势

- 实际上也是一张表，保存了主键与索引字段，指向实体，也占空间
- 降低表更新的速度，更新表是需要跟新索引列的字段
- 需要**花时间**去建立最优秀的索引，时间成本

##### 索引分类

- 单值索引：一个索引值包含一个列，一个表可以有多个单值索引
- 唯一索引：索引列的值唯一，允许空值
- 复合索引：一个索引包含多列
- 语法：
  - CREATE [UNIQUE] INDEX indexName On table(columname(length))
  - ALTER table ADD [UNIQUE] INDEX [indexname] ON (columnname(length))
  - DROP INDEX [indexname] ON table;
  - SHOW INDEX FROM table;![image-20200607222307027](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607222307027.png)

##### 索引数据结构

- BTree索引：
  - ![image-20200607222408125](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607222408125.png)
  - 
- Hash索引
- full-text全文索引
- R-Tree索引

##### 创建索引的情况

- 主键自动建立唯一索引
- 频繁作为查询条件的字段应该创建索引
- 查询中与其他表关联的字段，外键关系建立索引
- 频繁更新的字段不适合创建索引
- where条件里用不到的字段不创建索引
- 单值和复合：倾向于复合
- 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
- 查询中统计或者分组字段

##### 不建索引的情况

- 表记录太少
- 经常增删改的表
- 数据重复且平均分布的表字段：索引的选择性，不同的值/所有

#### 性能分析

##### MySQL Query Optimizer

![image-20200607224449572](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607224449572.png)

##### 常见瓶颈

CPU：CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时

IO：磁盘IO瓶颈发生在主让入数据远大于内存容量的时候

服务器硬件

##### Explain

执行计划，使用EXPLAIN关键字可以模拟优化器执行SQL语句，从而知道MySQL是如何处理你的SQL语句的，分析你的查询语句或是表结构的性能瓶颈。

![image-20200607225308061](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200607225308061.png)

执行计划包含的信息：

- id：查询的序列号，表示操作顺序
  - id相同，顺序从上到下，对应不同表
  - id不同，按序号顺序，id越大越先执行，一般大的为子查询
  - id相同不同同时存在，id越大优先级越高，数字同的可以认为是一组，顺序执行

- select_type：
  - SIMPLE：简单查询
  - PRIMARY：包含子查询，最外层为PRIMARY
  - SUBQUERY：子查询
  - DERIVED：在FROM列表中包含的子查询，标记此，存放临时表
  - UNION：
  - UNION RESULT：
- table：表名
- type：
  - ALL：全表扫描
  - index：通过索引遍历，全表扫描
  - range：使用索引来选择行
  - ref
  - eq_ref
  - Const,system
    - system：表只有一行记录（等于系统表），特例
    - Const：表示通过索引一次就找到了，常出现在主键索引或唯一索引，where id = 1？
  - NULL
  - system>const>eq_ref>ref>range>index>all
  - 一般来说，保证至少达到range级别，最好能到ref
- possible_keys：可能用的索引，一个或多个，但不一定实际使用
- key：实际使用的索引。为NULL则没有使用索引。查询中若使用了覆盖索引，则该索引仅出现在key列表
- ken_len：表示索引中使用的字节数，显示的值为索引字段的最大可能长度，并非实际使用长度
- ref：显示索引的哪一列倍使用了
- rows：每张表所需要读取的行数，估算
- extra：额外信息
  - Using filesort：mysql会对数据使用一个外部的索引排序，即mysql无法利用索引完成的排序操作称为”文件排序“
  - Using tenporary：产生临时表，常见于order by和group by
  - Using index：表示select使用了覆盖索引，避免了访问了表的数据行，效率不错
  - Using where
  - using join buffer
  - impossible where：where的值永远为false？
  - select tables optimized away
  - distinct

#### 索引优化

##### 索引分析

- 单表：

- 两表：左连接，在右表建立索引；右连接，在左表建立索引

- 三表：

join‘的优化：

- 妗可能减少join语句的循环总次数：永远用小表去驱动大表
- 优先优化内层循环
- 保证join语句中被驱动表上join条件字段已经被索引
- 当无法保证驱动表的join字段被索引切内存资源充足的前提下，不要太吝啬joinBuffer的设置

##### 索引失效——尽量避免

1. 全值匹配我最爱，怎么建怎么用。（顺序无所谓，优化器自动调整）
2. 最佳最前缀法则：🈯️查询从索引的最左字段开始，且不跳过中间列
3. 不在索引列上做任何操作（计算、函数、类型转换），导致索引失效转向全表扫描
4. 存储引起不能使用索引中范围条件右边的列，截断
5. 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致），减少select *
6. 使用不等于的时候无法使用索引，会导致全表扫面
7. isnull、is not null也无法使用索引
8. like以通配符开头，索引失效变全表扫描：如何解决？利用覆盖索引
9. 字符串不加单引号索引失效：（不加单引号需要强转）
10. 少用or，使用or也会失效

建议：

- 单值索引，尽量选择针对当前query过滤性更好的索引
- 选择组合索引时，当前query中过滤性最好的字段在索引字段顺序中，位置越靠前越好
- 在选择组合索引时，尽量选择可以能够包含当前quert中where子句中更对字段的索引
- 尽可能通过分析统计信息和调整query的写法来到达选择合适索引的目的

### 查询截取分析

1. explain
2. 观察，看看生成的慢sql情况
3. 开启慢查询日志，设置阈值，住区出来
4. explain+慢查询分析
5. show profile
6. 运维、DBA进行数据库的参数调优



#### 查询优化

- 永远小表驱动大表![image-20200608152723427](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608152723427.png)![image-20200608152907427](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608152907427.png)
- order by优化
  - 尽量使用index排序，避免filesort
  - 尽可能在索引列本身完成排序，按最左匹配
    - order by满足最左
    - 使用where于order by组合满足最左
  - 如果不在索引列上，filesort两种算法：
    - 双路排序：4.1之前，两次扫描磁盘
    - 单路排序：4.1之后，改进算法，在buffer排序，排完输出，少读一次
      - 由于单路后出，比双路好，但是有一次buffer装不完的可能，产生多次排序和合并
      - 优化：增大sort_buffer_size参数，增大max_length_for_sort_data，减少select*的使用
- group by：与order by基本一致
  - 实质是先排序后分组，按照最左
  - 当无法使用索引列，增大max_length_for_sort_data+增大sort_buffer_size
  - where高于having，能写在where的尽量写在where

#### 慢查询日志

日志记录，记录响应时间超过阈值的语句，超过long_query_time的sql，会被记录到慢查询日志中。默认为10秒。

默认，没有开启，需要手动设置，平时不建议启动。

查看是否开启：show varibales like '%slow_query_log%'

开启：set global slow_query_log=1；只对当前有效

查看时间：show variables like 'long_query_time%';

设置时间：set global long_query_time=3; // 重开一个会话才能看到修改

mysqldumpslow：日志分析工具![image-20200608183759488](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608183759488.png)



#### 批量数据集

![image-20200608184411296](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608184411296.png)

#### Show Profile

分析当前绘画中语句执行的资源消耗情况，可以用于SQL调优的测量，默认情况下，参数处于关闭状态，并保存最近15次的运行结果。

```
show profile;
show profile cpu,block ip for query 10.
```

#### 全局查询日志

测试时用，永远不要在生产环节打开。![image-20200608193256223](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608193256223.png)

### MySQL锁机制

对数据操作分类：

- 读（共享锁）：读共享，与写互斥，但是锁了一个表，不能读其他表的，
- 写（排他锁）：能读能写锁住的表，不能读其他表，不能写其他表

操作粒度分类：

- 表锁（偏读）
  - 偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低
- 行锁（偏写）
  - innodb，开销大，粒度小
  - 索引失效引行锁升级表锁。
  - 间隙锁：
  - ![image-20200608213907703](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608213907703.png)
  - 
- 页锁

```
show open tables;看哪些表被加锁了

```

![image-20200608211124395](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608211124395.png)



### 主从复制

基本原理：

- slave从master读取binlog来进行数据同步
- ![image-20200608214223393](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608214223393.png)

基本规则：

- 每个salve只有一个master
- 每个slave只能有一个唯一的服务器id

- 每个master可以有多个slave

最大问题：延迟

一主一从配置：

- 版本一致，后台运行
- 主从配置都在[mysqld]节点下
- 配置：
  - id唯一，binlog开启：server-id=1
  - log-bin=目录
  - log-err=目录，可选
  - basedir可选
  - tmpdir可选
  - datadir可选
  - 设置不要复制的数据库，可选
  - 设置需要复制的数据库，可选
- 重启数据库服务
- 主机上建立账户并授权slave![image-20200608220654970](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608220654970.png)flush privileges;
- show master status
- 从机上配置需要复制的主机![image-20200608221019767](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200608221019767.png)
- Start slave; show slave status;
- 主机建库，从机复制
- 如何停止从服务复制功能？stop slave