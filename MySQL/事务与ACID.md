# 事务与ACID

## \*ACID

* ### 原子性：一个事务必须被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚。对于一个事务而言，不可能只执行其中的一部分。
* ### 一致性：数据库总是从一个一致性的状态转移到另一个一致性的状态。比如银行转账，不会出现一个账户少了200元而一个账户没多200元。
* ### 隔离性：通常来说，一个事务所做的修改在最终提交以前，对其他事务是不可见的。具体看（隔离级别）。
* ### 持久性：一旦事务提交后，则其所做的修改就会永久保存到数据中。

## \*事务

### 事务内的语句，要么全部执行成功，要么全部执行失败。以一个银行应用为例：

### 可以用START TRANSACTION 语句开始一个事务，然后要么使用COMMIT提交事务将修改的数据持久保留，要么使用ROLLBACK撤销所有的修改。

```
START TRANSACTION;
SELECT balance FROM checking WHERE customer_id = 10233276;
UPDATE checking SET balance = balance - 200.00 WHERE customer_id = 10233276;
UPDATE savings SET balance = balance + 200.00 WHERE customer_id = 10233276;
COMMIT;
```



