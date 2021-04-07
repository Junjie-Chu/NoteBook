# SQL基础语法

## SQL的三类语句

●DDL（Data Definition Language，数据定义语言）    
用来创建或者删除存储数据用的数据库以及数据库中的表等对象。DDL 包含以下几种指令。  
CREATE： 创建数据库和表等对象  
DROP： 删除数据库和表等对象  
ALTER： 修改数据库和表等对象的结构  

●DML（Data Manipulation Language，数据操纵语言）  
用来查询或者变更表中的记录。DML 包含以下几种指令。  
SELECT：查询表中的数据  
INSERT：向表中插入新数据  
UPDATE：更新表中的数据  
DELETE：删除表中的数据  

●DCL（Data Control Language，数据控制语言）  
用来确认或者取消对数据库中的数据进行的变更。除此之外，还可以对RDBMS 的用户是否有权限操作数据库中的对象（数据库表等）进行设定。DCL 包含以下几种指令。  
COMMIT： 确认对数据库中的数据进行的变更  
ROLLBACK： 取消对数据库中的数据进行的变更  
GRANT： 赋予用户操作权限  
REVOKE： 取消用户的操作权限  

## DDL（Data Definition Language，数据定义语言）
### 创建数据库(CREATE)
CREATE DATABASE shop;   
创建一个名为shop的数据库  

### 创建表(CREATE)
CREATE TABLE Product  
(product_id     CHAR(4)      NOT NULL,  
 product_name   VARCHAR(100) NOT NULL,  
 product_type   VARCHAR(32)  NOT NULL,  
 sale_price     INTEGER      ,  
 purchase_price INTEGER      ,  
 regist_date    DATE         ,  
 PRIMARY KEY (product_id));  
创建一个名为Product的表，主键为product_id，包含六列，其中三列非空。    
每一列的数据类型（后述）是必须要指定的，数据类型包括：    
INTEGER 整数型    
NUMERIC ( 全体位数, 小数位数)    
CHAR 定长字符串    
VARCHAR 可变长字符串    
DATE 日期型    

### 删除表(DROP)
DROP TABLE Product;  
删除名为Product的表。  

### 表定义的更新(ALTER)
在表中增加一列(ADD COLUMN)  
ALTER TABLE Product ADD COLUMN product_name_pinyin VARCHAR(100);    
在Product表中，增加一列，名为product_name_pinyin，类型为VARCHAR(100)  

在表中删除一列(DROP COLUMN)  
ALTER TABLE Product DROP COLUMN product_name_pinyin;  
在Product表中，删除一列，名为product_name_pinyin    

变更表名(RENAME)    
RENAME TABLE Poduct to Product;    
将Poduct表重命名为Product表。  

## DCL（Data Control Language，数据控制语言）
BEGIN 或 START TRANSACTION 显式地开启一个事务；  
COMMIT 也可以使用 COMMIT WORK，不过二者是等价的。COMMIT 会提交事务，并使已对数据库进行的所有修改成为永久性的；  
ROLLBACK 也可以使用 ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；  
SAVEPOINT identifier，SAVEPOINT 允许在事务中创建一个保存点，一个事务中可以有多个 SAVEPOINT；  
RELEASE SAVEPOINT identifier 删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；  
ROLLBACK TO identifier 把事务回滚到标记点；  
SET TRANSACTION 用来设置事务的隔离级别。InnoDB 存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ 和 SERIALIZABLE。  

### 保存点：   
使用保留点 SAVEPOINT  
savepoint 是在数据库事务处理中实现“子事务”（subtransaction），也称为嵌套事务的方法。事务可以回滚到 savepoint 而不影响 savepoint 创建前的变化, 不需要放弃整个事务。  
ROLLBACK 回滚的用法可以设置保留点 SAVEPOINT，执行多条操作时，回滚到想要的那条语句之前。  

使用 SAVEPOINT：    
SAVEPOINT savepoint_name;    // 声明一个 savepoint  
ROLLBACK TO savepoint_name;  // 回滚到savepoint  

删除 SAVEPOINT：    
保留点再事务处理完成（执行一条 ROLLBACK 或 COMMIT）后自动释放。  
MySQL5 以来，可以用:  
RELEASE SAVEPOINT savepoint_name;  // 删除指定保留点   

### 创建事务(START TRANSACTION) - 提交处理(COMMIT)

START TRANSACTION;  
    -- 将运动T恤的销售单价降低1000日元  
    UPDATE Product  
       SET sale_price = sale_price - 1000  
     WHERE product_name = '运动T恤';  
    -- 将T恤衫的销售单价上浮1000日元  
    UPDATE Product  
       SET sale_price = sale_price + 1000  
     WHERE product_name = 'T恤衫';  
COMMIT;  

### 取消处理(ROLLBACK)

START TRANSACTION;  
    -- 将运动T恤的销售单价降低1000日元  
    UPDATE Product  
       SET sale_price = sale_price - 1000  
     WHERE product_name = '运动T恤';  
    -- 将T恤衫的销售单价上浮1000日元  
    UPDATE Product  
       SET sale_price = sale_price + 1000  
     WHERE product_name = 'T恤衫';  
ROLLBACK;  
