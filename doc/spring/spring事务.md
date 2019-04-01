## 1. 事务的概念
理解事务之前，先讲一个你日常生活中最常干的事：取钱。 
比如你去ATM机取1000块钱，大体有两个步骤：首先输入密码金额，银行卡扣掉1000元钱；然后ATM出1000元钱。这两个步骤必须是要么都执行要么都不执行。
如果银行卡扣除了1000块但是ATM出钱失败的话，你将会损失1000元；如果银行卡扣钱失败但是ATM却出了1000块，那么银行将损失1000元。
所以，如果一个步骤成功另一个步骤失败对双方都不是好事，如果不管哪一个步骤失败了以后，整个取钱过程都能回滚，也就是完全取消所有操作的话，这对双方都是极好的。 
事务就是用来解决类似问题的。
事务是一系列的动作，它们综合在一起才是一个完整的工作单元，这些动作必须全部完成，如果有一个失败的话，那么事务就会回滚到最开始的状态，仿佛什么都没发生过一样。 
在企业级应用程序开发中，事务管理必不可少的技术，用来确保数据的完整性和一致性。 
### 事务有四个特性：ACID 
> 1. 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
> 2. 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
> 3. 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
> 4. 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

## 2. 事务属性的定义

```
package org.springframework.transaction;

public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    int TIMEOUT_DEFAULT = -1;

    int getPropagationBehavior(); //返回事务的传播行为

    int getIsolationLevel(); //返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据

    int getTimeout(); //返回事务必须在多少秒内完成

    boolean isReadOnly(); //返回事务是否只读，事务管理器能够根据这个返回值进行优化，确保事务是只读的

    String getName(); //回滚规则
}

```
### 2-1 事务的传播行为
|         传播行为        |                        含义                                                        |
|-|-|
|  PROPAGATION_REQUIRED  | 表示当前方法必须运行在事务中。如果当前事务存在，方法会在该事务中运行。否则，会启动一个新的事务。
|  PROPAGATION_SUPPORTS  | 表示当前方法不需要事务上下文，如果存在当前事务，方法会在该事务中运行。|
|  PROPAGATION_MANDATORY  |表示方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常。|
|  PROPAGATION_REQUIRES_NEW   | 表示方法必须运行在自己的事务中。一个新的事务被启动，如果存在当前事务，当前事务将会被挂起。|
|  PROPAGATION_NOT_SUPPORTED  | 表示该方法不应运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。 |
|  PROPAGATION_NEVER   | 表示方法不应该运行在事务中，如果存在当前事务，则抛异常。|
|  PROPAGATION_NESTED  | 表示当前如果存在一个事务，那么该方法会嵌套事务中执行。嵌套的事务可以独立的回滚和提交。如果当前事务不存在，那么和PROPAGATION_REQUIRED一样。|

### 2-2 事务的隔离级别
事务的第二个维度就是隔离级别（isolation level）。隔离级别定义了一个事务可能受其他并发事务影响的程度。

#### 并发引起的问题
```
1. 脏读（Dirty reads）——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。
2. 不可重复读（Nonrepeatable read）——不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。
3. 幻读（Phantom read）——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

```
#### 不可重复读和幻读的区别
不可重复读的重点是修改，幻读的重点在于新增或者删除。

#### 隔离级别
|   隔离级别   | 含义  |
|-|-|
|   ISOLATION_DEFAULT   | 使用后端数据库的默认隔离级别|
|   ISOLATION_READ_UNCOMMITTED   | 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读|
|   ISOLATION_READ_COMMITTED   | 允许读取并发事务已经提交的数据，可以阻止可以阻止脏读，但是幻读或不可重复读仍有可能发生|
|   ISOLATION_REPEATABLE_READ   | 对同一子段的多次读取结果都是一致的，除非数据本身事务自己修改，可以阻止脏读和不可重复读，幻读有可能发生|
|   ISOLATION_SERIALIZABLE   | 最高的隔离级别，完全服从acid的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的|

### 2-3 只读
事务的第三个特性是它是否为只读事务。如果事务只对后端的数据库进行该操作，数据库可以利用事务的只读特性来进行一些特定的优化。通过将事务设置为只读，你就可以给数据库一个机会，让它应用它认为合适的优化措施。

### 2-4 事务超时
为了使应用程序很好地运行，事务不能运行太长的时间。因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要的占用数据库资源。事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。

### 2-5 回滚滚则
定义了哪些异常会导致回滚那些不会回滚。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的） 
但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

## 3. 事务状态
```
package org.springframework.transaction;

import java.io.Flushable;

public interface TransactionStatus extends SavepointManager, Flushable {
    boolean isNewTransaction(); //是否是新的事务

    boolean hasSavepoint(); //是否有恢复点

    void setRollbackOnly(); //设置位为只回滚

    boolean isRollbackOnly(); //是否为只回滚

    void flush(); //

    boolean isCompleted(); //是否已完成
}

```

### 3-1 编程式事务
Spring提供两种方式的编程式事务管理，分别是：使用TransactionTemplate和直接使用PlatformTransactionManager。
#### 使用TransactionTemplate
```
    TransactionTemplate tt = new TransactionTemplate(); // 新建一个TransactionTemplate
    Object result = tt.execute(
        new TransactionCallback(){  
            public Object doTransaction(TransactionStatus status){  
                updateOperation();  
                return resultOfUpdateOperation();  
            }  
    }); // 执行execute方法进行事务管理
    
```
#### 使用PlatformTransactionManager
```
DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(); //定义一个某个框架平台                           TransactionManager，如JDBC、Hibernate
dataSourceTransactionManager.setDataSource(this.getJdbcTemplate().getDataSource()); // 设置数据源
DefaultTransactionDefinition transDef = new DefaultTransactionDefinition(); // 定义事务属性
transDef.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED); // 设置传播行为属性
TransactionStatus status = dataSourceTransactionManager.getTransaction(transDef); // 获得事务状态
try {
  // 数据库操作
  dataSourceTransactionManager.commit(status);// 提交
} catch (Exception e) {
  dataSourceTransactionManager.rollback(status);// 回滚
}

```

### 3-2 声明式事务

## 4. [详见博客] （https://blog.csdn.net/trigl/article/details/50968079）






