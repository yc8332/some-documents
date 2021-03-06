#### 1、`read uncommitted` 读未提交：所有事务都能看到其他未提交事务的修改
读取未提交数据，称之为脏读。如下表两个事务，A事务读到了B事务未提交的修改。   

| A事务 | B事务 |
| :-----------------------------------------------------------| :--------------------------------------------------- |
| begin;                                                      |                                                      |
| select nickname from users where id = 6; -> `haah`          |                                                      |
|                                                             | begin;                                               |
|                                                             | update users set nickname = 'zhangsan' where id = 6; |
| select nickname from users where id = 6; -> `zhangsan`      |                                                      |


#### 2、`read committed` 读已提交：读行的最新版本，如果行被锁定，则读取该行最新一个快照
一个事务两次查询同一行记录得到的数据不一致，事务A会读取到事务B提交的修改。   

| A事务 | B事务 |
| :-----------------------------------------------------------| :--------------------------------------------------- |
| begin;                                                      |                                                      |
| select nickname from users where id = 6; -> `haah`          |                                                      |
|                                                             | begin;                                               |
|                                                             | update users set nickname = 'zhangsan' where id = 6; |
|                                                             | commit; `提交事务，注意这就是与读未提交的区别`       |
| select nickname from users where id = 6; -> `zhangsan`      |                                                      |


#### 3、`repeatable read` 可重复读：一个事务两次查询同一行记录得到的数据一致
但是如果是两次查询一个范围的数据，可能会返回不同的行(其它事务在该范围插入了新行)，称之为幻读。在 `repeatable read` 级别下，InnoDB 引擎使用 `Next-Key Locking` 机制来解决幻读问题。   
普通的 `repeatable read` 级别执行结果如下，但MySQL的 `repeatable read` 是不会出现下面的幻读现象的。   

| A事务 | B事务 |
| :-----------------------------------------------------------| :--------------------------------------------------- |
| begin;                                                      |                                                      |
| select nickname from users where id < 7; -> `haah`          |                                                      |
|                                                             | begin;                                               |
|                                                             | insert into users values (5, 'oo00oo');              |
|                                                             | commit; `提交事务`                                   |
| select nickname from users where id < 7; -> `oo00oo, haah`  |                                                      |

#### 4、`serializable` 串行化：事务一个接一个地执行


#### 5、附 users 表如下图
![](http://wx4.sinaimg.cn/mw690/abf82c72gy1fekao9jd52j20gl051a9y.jpg)
