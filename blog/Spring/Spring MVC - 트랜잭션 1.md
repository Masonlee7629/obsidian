태그: #Spring
연결문서: [Spring MVC - 트랜잭션 2](Spring%20MVC%20-%20트랜잭션%202.md)

# 트랜잭션(Transaction)

## 트랜잭션이란?

트랜잭션은 여러 개의 작업들을 하나의 그룹으로 묶어서 처리하는 처리 단위이다.  
  

무조건 여러 개의 작업을 그룹으로 묶는다고 해서 트랜잭션이라고 부를 수 있는게 아니라 물리적으로는 여러 개의 작업이지만 논리적으로는 마치 하나의 작업으로 인식해서 **전부 성공하든가 전부 실패하든가(All or Nothing)**의 둘 중 하나로만 처리되어야 트랜잭션의 의미를 가진다.  
  

**All or Nothing**이라는 트랜잭션 처리 방식은 애플리케이션에서 사용하는 데이터의 무결성을 보장하는 핵심적인 역할을 한다.

### ACID 원칙

트랜잭션의 특징을 이야기할 때는 일반적으로 ACID라는 원칙을 이용한다.

1.  **원자성(Atomicity)**  
    트랜잭션에서의 원자성은 작업을 더 이상 쪼갤 수 없음을 의미한다.  
    논리적으로 하나의 작업으로 인색해서 **둘 다 성공하든가 둘 다 실패하든가()** 중에서 하나로만 처리되는 것이 보장되어야 한다.
2.  **일관성(Consistency)**  
    일관성은 트랜잭션이 에러 없이 성공적으로 종료될 경우, 비즈니스 로직에서 의도하는대로 일관성있게 저장되거나 변경되는 것을 의미한다.
3.  **격리성(Isolation)**  
    격리성은 여러 개의 트랜잭션이 실행될 경우, 각각 독립적으로 실행이 되어야 함을 의미한다.  
    예시로 데이터베이스 성능 향상을 목적으로 한 개 이상의 트랜잭션을 번갈아가면서 처리할 수 있다. 이 때, 트랜잭션이 다른 트랜잭션에 영향을 주지 않고 독립적으로 실행이 되어야 한다는 것이다.
4.  **지속성(Durability)**  
    트랜잭션이 완료되면 그 결과가 지속되어야 한다는 것을 의미한다.  
    데이터베이스가 종료되어도 데이터는 물리적인 저장소에 저장되어 지속적으로 유지되어야 한다는 것이다.

### 트랜잭션 커밋(commit)과 롤백(rollback)

#### 커밋(commit)

-   커밋은 모든 작업을 최종적으로 데이터베이스에 반영하는 명령어로써 commit 명령을 수행하면 변경된 내용이 데이터베이스에 영구적으로 저장된다.
-   만약 commit 명령을 수행하지 않으면 작업의 결과가 데이터베이스에 최종적으로 반영되지 않는다.
-   commit 명령을 수행하면 하나의 트랜잭션 과정은 종료하게 된다.

#### 롤백(rollback)

-   롤백은 작업 중 문제가 발생했을 때, 트랜잭션 내에서 수행된 작업들을 취소한다.
-   트랜잭션 시작 이전의 상태로 되돌아가게 된다.

#### JPA의 tx.commit() 동작 과정

1.  **TransactionImpl**

```
package org.hibernate.engine.transaction.internal;

public class TransactionImpl implements TransactionImplementor {
            ...
            ...
            @Override
            public void commit() {
                ...
                ...
                try {
                    internalGetTransactionDriverControl().commit(); // (1)
                }
                catch (RuntimeException e) {
                    throw session.getExceptionConverter().convertCommitException( e );
                }
            }
}
```

JPA `EntityTransaction` 객체(tx)로 `tx.commit()`을 호출하는 것을 `EntityTransaction` 인터페이스의 구현 클래스인 `TransactionImpl` 클래스의 `commit()`을 호출하는 것이다.  
  

`TransactionImpl` 클래스에서는 (1)과 같이 물리적인 트랜잭션을 제어하기 위한 로컬 트랜잭션 드라이버 구현 객체(`TransactionDriverControlImpl`)를 얻은 후에 구현 메서드인 `commit()`을 다시 호출한다.

2.  **JdbcResourceLocalTransactionCoordinatorImpl** > **TransactionDriverControlImpl**

```
package org.hibernate.resource.transaction.backend.jdbc.internal;

public class JdbcResourceLocalTransactionCoordinatorImpl 
                                        implements TransactionCoordinator {
            ...
            ...
            public class TransactionDriverControlImpl implements TransactionDriver {
                        @Override
                        public void commit() {
                            try {
                                ...
                                ...
                                JdbcResourceLocalTransactionCoordinatorImpl
                                               .this.beforeCompletionCallback();
                                jdbcResourceTransaction.commit();  // (1)
                                JdbcResourceLocalTransactionCoordinatorImpl
                                        .this.afterCompletionCallback( true );
                            }
                            catch (RollbackException e) {
                                throw e;
                            }
                            catch (RuntimeException e) {
                                try {
                                    rollback();
                                }
                                catch (RuntimeException e2) {
                                    log.debug( "Encountered failure rolling back failed commit", e2 );
                                }
                                throw e;
                            }
                        }
            }
}
```

`TransactionDriverControlImpl`에서는 (1)과 같이 JDBC Connection의 액세스 방법을 제공하는 `JdbcResourceTransaction` 의 구현 객체인 `AbstractLogicalConnectionImplementor`의 `commit()`을 다시 호출합니다.

3.  **AbstractLogicalConnectionImplementor**

```
package org.hibernate.resource.jdbc.internal;

public abstract class AbstractLogicalConnectionImplementor 
                implements LogicalConnectionImplementor, PhysicalJdbcTransaction {
            ...
            ...
            @Override
            public void begin() {
                    ...    
            }

            @Override
            public void commit() {
                try {
                    getConnectionForTransactionManagement().commit(); // (1)
                    status = TransactionStatus.COMMITTED;

                }
                catch( SQLException e ) {
                    status = TransactionStatus.FAILED_COMMIT;
                }
                afterCompletion();
            }

            ...
            ...
            @Override
            public void rollback() {
                try {
                    getConnectionForTransactionManagement().rollback();
                    status = TransactionStatus.ROLLED_BACK;
                }
                catch( SQLException e ) {
                    status = TransactionStatus.FAILED_ROLLBACK;
                }

                afterCompletion();
            }
}
```

(1)에서 실제 물리적인 JDBC Connection을 얻은 후에 이 connection 객체의 commit()을 다시 호출한다.  
  

여기까지가 Hibernate ORM에서의 영역이다.  
  

이제 물리적인 JDBC Connection을 통해 데이터베이스와 인터랙션하기 위해서 JDBC API의 구현체인 JdbcConnection 영역으로 이동한다.

4.  **JdbcConnection**

```
// 인메모리 DB인 H2를 사용하기 있기 때문에 H2에서 제공하는 라이브러리를 사용
package org.h2.jdbc;

public class JdbcConnection extends TraceObject 
        implements Connection, JdbcConnectionBackwardsCompat, CastDataProvider {
            ...
            ...

            @Override
        public synchronized void commit() throws SQLException {
            try {
                ...
                            ...
                commit = prepareCommand("COMMIT", commit);  // (1)
                commit.executeUpdate(null);    // (2)
            } catch (Exception e) {
                throw logAndConvert(e);
            }
        }
}
```

(1)에서 데이터베이스에 commit 명령을 준비한 후, (2)에서 해당 명령을 실행한다.  
  

(1)과 (2)의 메서드 호출은 H2에서 지원하는 Command 클래스에서 이루어진다.

5.  **Command**

```
package org.h2.command;

public abstract class Command implements CommandInterface {
            ...
            ...
            @Override
        public ResultWithGeneratedKeys executeUpdate(Object generatedKeysRequest) {
            long start = 0;
            Database database = session.getDatabase();
            session.waitIfExclusiveModeEnabled();
            boolean callStop = true;
            //noinspection SynchronizationOnLocalVariableOrMethodParameter
            synchronized (session) {
                commitIfNonTransactional();  // (1) 
                SessionLocal.Savepoint rollback = session.setSavepoint();
                session.startStatementWithinTransaction(this);
                DbException ex = null;
                Session oldSession = session.setThreadLocalSession();
                try {
                      ...
                                    ...
                } catch (DbException e) {
                    e = e.addSQL(sql);
                    ...
                                    ...
                    try {
                        database.checkPowerOff();
                        if (s.getErrorCode() == ErrorCode.DEADLOCK_1) {
                            session.rollback();    // (2)
                        } else {
                            session.rollbackTo(rollback);
                        }
                    } catch (Throwable nested) {
                        e.addSuppressed(nested);
                    }
                    ex = e;
                    throw e;
                } finally {
                    ...
                                    ...
                }
            }
        }

        private void commitIfNonTransactional() {
            if (!isTransactional()) {
                boolean autoCommit = session.getAutoCommit();   // (3)
                session.commit(true);   // (4)
                if (!autoCommit && session.getAutoCommit()) {
                    session.begin();
                }
            }
        }
}
```

데이터베이스에 commit 명령을 전달하기 위해 (1)에서 `commitIfNonTransactional()` 메서드를 호출한다.  
  

`commitIfNonTransactional()` 메서드 내부에서는 (3)에서 auto commit 여부를 체크한 후, (4)와 같이 데이터베이스 세션에 해당하는 Session 객체를 통해 commit 명령을 수행하고 있습니다.  
  

만약에 commitIfNonTransactional() 수행 과정에서 예외가 발생하면 (2)와 같이 rollback을 수행하는 것을 볼 수 있다.

6.  **SessionLocal**

```
package org.h2.engine;

public final class SessionLocal extends Session 
            implements TransactionStore.RollbackListener {
            ...
            ...
            public void commit(boolean ddl) {
        beforeCommitOrRollback();
        if (hasTransaction()) {
            try {
                markUsedTablesAsUpdated();
                transaction.commit();      // (1)
                removeTemporaryLobs(true);
                endTransaction();
            } finally {
                transaction = null;
            }
            ...
                        ...
        }
    }
}
```

마지막으로 SessionLocal 클래스에서 (1)과 같이 트랜잭션에 대한 commit이 수행된다.