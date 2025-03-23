---
layout: post
title:  使用Spring JdbcTemplate的存储过程
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

在本教程中，我们将讨论[Spring JDBC框架](https://www.baeldung.com/spring-jdbc-jdbctemplate)的JdbcTemplate类执行数据库存储过程的能力。数据库存储过程类似于函数，函数支持输入参数并具有返回类型，而存储过程则支持输入和输出参数。

## 2. 先决条件

让我们考虑[PostgreSQL](https://www.postgresql.org/)数据库中的一个简单的存储过程：

```sql
CREATE OR REPLACE PROCEDURE sum_two_numbers(
    IN num1 INTEGER,
    IN num2 INTEGER,
    OUT result INTEGER
)
LANGUAGE plpgsql
AS '
BEGIN
    sum_result := num1 + num2;
END;
';
```

**存储过程sum_two_numbers接收两个输入数字，并在输出参数sum_result中返回它们的和**。通常，存储过程可以支持多个输入和输出参数。但是，对于此示例，我们只考虑一个输出参数。

## 3. 使用JdbcTemplate#call()方法

让我们看看如何使用[JdbcTemplate#call()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html#call(org.springframework.jdbc.core.CallableStatementCreator,java.util.List))方法调用数据库存储过程：

```java
void givenStoredProc_whenCallableStatement_thenExecProcUsingJdbcTemplateCallMethod() {
    List<SqlParameter> procedureParams = List.of(new SqlParameter("num1", Types.INTEGER),
            new SqlParameter("num2", Types.NUMERIC),
            new SqlOutParameter("result", Types.NUMERIC)
    );

    Map<String, Object> resultMap = jdbcTemplate.call(new CallableStatementCreator() {
        @Override
        public CallableStatement createCallableStatement(Connection con) throws SQLException {
            CallableStatement callableStatement = con.prepareCall("call sum_two_numbers(?, ?, ?)");
            callableStatement.registerOutParameter(3, Types.NUMERIC);
            callableStatement.setInt(1, 4);
            callableStatement.setInt(2, 5);

            return callableStatement;
        }
    }, procedureParams);

    assertEquals(new BigDecimal(9), resultMap.get("result"));
}
```

**首先，我们借助[SqlParameter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/SqlParameter.html)类定义存储过程sum_two_numbers()的IN参数num1和num2。然后，我们借助[SqlOutParameter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/SqlOutParameter.html)定义OUT参数result**。

随后，我们将CallableStatementCreator对象和procedureParams中的List<SqlParameter\>传递给JdbcTemplate#call()方法。

在CallableStatementCreator#createCallableStatement()方法中，我们通过调用Connection#prepareCall()方法创建[CallableStatement](https://docs.oracle.com/en/java/javase/21/docs/api/java.sql/java/sql/CallableStatement.html)对象。与PreparedStatement类似，我们在CallableStatement对象中设置IN参数。

但是，我们必须使用registerOutParameter()方法注册OUT参数。

最后，我们从resultMap中的Map对象中检索结果。

## 4. 使用JdbcTemplate#execute()方法

在某些情况下，我们需要对CallableStatement进行更多控制。因此，**Spring框架提供了类似于[PreparedStatementCallback](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/PreparedStatementCallback.html)的[CallableStatementCallback](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/CallableStatementCallback.html)接口**，让我们看看如何在[JdbcTemplate#execute()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html#execute(org.springframework.jdbc.core.CallableStatementCreator,org.springframework.jdbc.core.CallableStatementCallback))方法中使用它：

```java
void givenStoredProc_whenCallableStatement_thenExecProcUsingJdbcTemplateExecuteMethod() {
    String command = jdbcTemplate.execute(new CallableStatementCreator() {
        @Override
        public CallableStatement createCallableStatement(Connection con) throws SQLException {
            CallableStatement callableStatement = con.prepareCall("call sum_two_numbers(?, ?, ?)");
            return callableStatement;
        }
    }, new CallableStatementCallback<String>() {
        @Override
        public String doInCallableStatement(CallableStatement cs) throws SQLException, DataAccessException {
            cs.setInt(1, 4);
            cs.setInt(2, 5);
            cs.registerOutParameter(3, Types.NUMERIC);
            cs.execute();
            BigDecimal result = cs.getBigDecimal(3);
            assertEquals(new BigDecimal(9), result);

            String command = "4 + 5 = " + cs.getBigDecimal(3);
            return command;
        }
    });
    assertEquals("4 + 5 = 9", command);
}
```

CallableStatementCreator对象参数创建CallableStatement对象，稍后，该对象可在CallableStatementCallback#doInCallableStatement()方法中使用。

在这个方法中，我们设置CallableStatement对象中的IN和OUT参数，然后调用CallableStatement#execute()。最后，我们获取结果并形成命令4 + 5 = 9。

**我们可以在doInCallableStatement()方法中多次重用CallableStatement对象来执行具有不同参数的存储过程**。

## 5. 使用SimpleJdbcCall

[SimpleJdbcCall](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/simple/SimpleJdbcCall.html)类在内部使用JdbcTemplate来执行存储过程和函数，它还支持流式方法链，使其更易于理解和使用。

此外，**SimpleJdbcCall专为多线程场景而设计。因此，它允许多个线程进行安全的并发访问，而无需任何外部同步**。

让我们看看如何借助此类调用存储过程sum_two_numbers：

```java
void givenStoredProc_whenJdbcTemplate_thenCreateSimpleJdbcCallAndExecProc() {
    SimpleJdbcCall simpleJdbcCall = new SimpleJdbcCall(jdbcTemplate).withProcedureName("sum_two_numbers");

    Map<String, Integer> inParams = new HashMap<>();
    inParams.put("num1", 4);
    inParams.put("num2", 5);
    Map<String, Object> resultMap = simpleJdbcCall.execute(inParams);
    assertEquals(new BigDecimal(9), resultMap.get("result"));
}
```

首先，我们通过将JdbcTemplate对象传递给其构造函数来实例化SimpleJdbcCall类，在底层，这个JdbcTemplate对象执行存储过程。然后，我们将存储过程名称传递给SimpleJdbcCall#withProcedureName()方法。

最后，我们将Map中的输入参数传递给SimpleJdbcCall#execute()方法，从而在Map对象中获取结果。结果将根据OUT参数名称的键进行存储。

有趣的是，**不需要定义存储过程参数的元数据，因为SimpleJdbcCall类可以读取数据库元数据**。此支持仅限于少数数据库，例如Derby、MySQL、Microsoft SQLServer、Oracle、DB2、Sybase和PostgreSQL。

因此，对于其他情况，我们需要在SimpleJdbcCall#declareParameters()方法中明确定义参数：

```java
@Test
void givenStoredProc_whenJdbcTemplateAndDisableMetadata_thenCreateSimpleJdbcCallAndExecProc() {
    SimpleJdbcCall simpleJdbcCall = new SimpleJdbcCall(jdbcTemplate)
        .withProcedureName("sum_two_numbers")
        .withoutProcedureColumnMetaDataAccess();
    simpleJdbcCall.declareParameters(new SqlParameter("num1", Types.NUMERIC),
        new SqlParameter("num2", Types.NUMERIC),
        new SqlOutParameter("result", Types.NUMERIC));

    Map<String, Integer> inParams = new HashMap<>();
    inParams.put("num1", 4);
    inParams.put("num2", 5);
    Map<String, Object> resultMap = simpleJdbcCall.execute(inParams);
    assertEquals(new BigDecimal(9), resultMap.get("result"));
}
```

我们通过调用SimpleJdbcCall#withoutProcedureColumnMetaDataAccess()方法禁用了数据库元数据处理，其余步骤与以前相同。

## 6. 使用StoredProcedure

**[StoredProcedure](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/object/StoredProcedure.html)是一个抽象类，我们可以重写它的execute()方法进行额外的处理**：

```java
public class StoredProcedureImpl extends StoredProcedure {
    public StoredProcedureImpl(JdbcTemplate jdbcTemplate, String procName) {
        super(jdbcTemplate, procName);
    }

    private String doSomeProcess(Object procName) {
        //do some processing
        return null;
    }

    @Override
    public Map<String, Object> execute(Map<String, ?> inParams) throws DataAccessException {
        doSomeProcess(inParams);
        return super.execute(inParams);
    }
}
```

让我们看看如何使用这个类：

```java
@Test
void givenStoredProc_whenJdbcTemplate_thenCreateStoredProcedureAndExecProc() {
    StoredProcedure storedProcedure = new StoredProcedureImpl(jdbcTemplate, "sum_two_numbers");
    storedProcedure.declareParameter(new SqlParameter("num1", Types.NUMERIC));
    storedProcedure.declareParameter(new SqlParameter("num2", Types.NUMERIC));
    storedProcedure.declareParameter(new SqlOutParameter("result", Types.NUMERIC));

    Map<String, Integer> inParams = new HashMap<>();
    inParams.put("num1", 4);
    inParams.put("num2", 5);

    Map<String, Object> resultMap = storedProcedure.execute(inParams);
    assertEquals(new BigDecimal(9), resultMap.get("result"));
}
```

与SimpleJdbcCall类似，我们首先通过传入JdbcTemplate和存储过程名称来实例化StoredProcedure的子类。然后设置参数，执行存储过程，并在Map中获取结果。

此外，**我们必须记住按照将参数传递给存储过程的相同顺序声明SqlParameter对象**。

## 7. 总结

在本文中，我们讨论了JdbcTemplate执行存储过程的能力。JdbcTemplate是处理数据库中数据操作的核心类，它可以直接使用，也可以借助SimpleJdbcCall和StoredProcedure等包装类隐式使用。