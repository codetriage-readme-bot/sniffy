JDBC Sniffer
============
[![CI Status](https://travis-ci.org/bedrin/jdbc-sniffer.svg?branch=develop)](https://travis-ci.org/bedrin/jdbc-sniffer)

JDBC Sniffer counts the number of executed SQL queries and provides an API for validating it and reseting to 0
It is very useful in unit tests and allows you to test if particular method doesn't make more than N SQL queries

Maven
============
JDBC Sniffer is available from Maven Central repository
```xml
<dependency>
    <groupId>com.github.bedrin</groupId>
    <artifactId>jdbc-sniffer</artifactId>
    <version>1.3</version>
</dependency>
```

Download
============
- [jdbc-sniffer-1.3.jar](https://github.com/bedrin/jdbc-sniffer/releases/download/1.3/jdbc-sniffer-1.1.jar)
- [jdbc-sniffer-1.3-sources.jar](https://github.com/bedrin/jdbc-sniffer/releases/download/1.3/jdbc-sniffer-1.1-sources.jar)
- [jdbc-sniffer-1.3-javadoc.jar](https://github.com/bedrin/jdbc-sniffer/releases/download/1.3/jdbc-sniffer-1.1-javadoc.jar)

Setup
============
Simply add jdbc-sniffer.jar to your classpath and add `sniffer:` prefix to the JDBC connection url
For example `jdbc:h2:~/test` should be changed to `sniffer:jdbc:h2:~/test`

Validating the number of queries
============
The number of executed queries is available via static methods of two classes:
`com.github.bedrin.jdbc.sniffer.Sniffer` and `com.github.bedrin.jdbc.sniffer.ThreadLocalSniffer`

First one holds the number of SQL queries executed by all threads, while the later holds the number of SQL queries generated by current thread only

```java
@Test
public void testExecuteStatement() throws ClassNotFoundException, SQLException {
    // Just add sniffer: in front of your JDBC connection URL in order to enable sniffer
    Connection connection = DriverManager.getConnection("sniffer:jdbc:h2:~/test", "sa", "sa");
    connection.createStatement().execute("SELECT 1 FROM DUAL");
    // Sniffer.executedStatements() returns count of execute queries
    assertEquals(1, Sniffer.executedStatements());
    // Sniffer.verifyNotMoreThanOne() throws an IllegalStateException if more than one query was executed; it also resets the counter to 0
    Sniffer.verifyNotMoreThanOne();
    // Sniffer.verifyNotMore() throws an IllegalStateException if any query was executed
    Sniffer.verifyNotMore();
}
```

JUnit Integration
============
JDBC Sniffer supports integration with JUnit framework via `@Rule`

Add a `QueryCounter` rule to your test and assert the maximum number of queries allowed for particular test using `@AllowedQueries(n)` and `@NotAllowedQueries` annotations

```java
package com.github.bedrin.jdbc.sniffer.junit;

import org.junit.BeforeClass;
import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ExpectedException;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class QueryCounterTest {

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Rule
    public QueryCounter queryCounter = new QueryCounter();

    @BeforeClass
    public static void loadDriver() throws ClassNotFoundException {
        Class.forName("com.github.bedrin.jdbc.sniffer.MockDriver");
    }

    @Test
    @AllowedQueries(1)
    public void testAllowedOneQuery() throws SQLException {
        Connection connection = DriverManager.getConnection("sniffer:jdbc:h2:~/test", "sa", "sa");
        connection.createStatement().execute("SELECT 1 FROM DUAL");
    }

    @Test
    @NotAllowedQueries
    public void testNotAllowedQueries() throws SQLException {
        Connection connection = DriverManager.getConnection("sniffer:jdbc:h2:~/test", "sa", "sa");
        connection.createStatement().execute("SELECT 1 FROM DUAL");
        thrown.expect(IllegalStateException.class);
    }

}```

Building
============
JDBC sniffer is built using JDK8+ and Maven 3+ - just checkout the project and type `mvn install`
