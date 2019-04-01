## 1. 事务的概念
理解事务之前，先讲一个你日常生活中最常干的事：取钱。 
比如你去ATM机取1000块钱，大体有两个步骤：首先输入密码金额，银行卡扣掉1000元钱；然后ATM出1000元钱。这两个步骤必须是要么都执行要么都不执行。
如果银行卡扣除了1000块但是ATM出钱失败的话，你将会损失1000元；如果银行卡扣钱失败但是ATM却出了1000块，那么银行将损失1000元。
所以，如果一个步骤成功另一个步骤失败对双方都不是好事，如果不管哪一个步骤失败了以后，整个取钱过程都能回滚，也就是完全取消所有操作的话，这对双方都是极好的。 
事务就是用来解决类似问题的。
事务是一系列的动作，它们综合在一起才是一个完整的工作单元，这些动作必须全部完成，如果有一个失败的话，那么事务就会回滚到最开始的状态，仿佛什么都没发生过一样。 
在企业级应用程序开发中，事务管理必不可少的技术，用来确保数据的完整性和一致性。 
事务有四个特性：ACID 
> ### 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
> ### 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
> ### 隔离性（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
> ### 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

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
## 3. 事务的传播行为
|         传播行为        |                        含义                                                        |
|  PROPAGATION_REQUIRED  | 表示当前方法必须运行在事务中。如果当前事务存在，方法会在该事务中运行。否则，会启动一个新的事务。
|  PROPAGATION_SUPPORTS  | 表示当前方法不需要事务上下文，如果存在当前事务，方法会在该事务中运行。|
|  PROPAGATION_MANDATORY  |表示方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常。|
|  PROPAGATION_REQUIRES_NEW   | 表示方法必须运行在自己的事务中。一个新的事务被启动，如果存在当前事务，当前事务将会被挂起。|
|  PROPAGATION_NOT_SUPPORTED  | 表示该方法不应运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。 |
|  PROPAGATION_NEVER   | 表示方法不应该运行在事务中，如果存在当前事务，则抛异常。|
|  PROPAGATION_NESTED  | 表示当前如果存在一个事务，那么该方法会嵌套事务中执行。嵌套的事务可以独立的回滚和提交。如果当前事务不存在，那么和PROPAGATION_REQUIRED一样。|

## 4. 事务的隔离级别

int ISOLATION_DEFAULT = -1;
int ISOLATION_READ_UNCOMMITTED = 1;
int ISOLATION_READ_COMMITTED = 2;
int ISOLATION_REPEATABLE_READ = 4;
int ISOLATION_SERIALIZABLE = 8;
int TIMEOUT_DEFAULT = -1;
