Java
====

JDBC
----

The [JDBC API](http://download.oracle.com/javase/tutorial/jdbc/index.html)
has a class called
[`PreparedStatement`](http://download.oracle.com/javase/6/docs/api/java/sql/PreparedStatement.html)
which allows the programmer to safely insert user-supplied data
into a SQL query.  The location of each input value in the query
string is marked with a question mark.  The various `set*()` methods
are then used to safely perform the insertion.

    String name = //user input
    int age = //user input
    Connection connection = DriverManager.getConnection(...);
    PreparedStatement statement = connection.prepareStatement(
            "SELECT * FROM people WHERE lastName = ? AND age > ?" );
    statement.setString(1, name); //lastName is a VARCHAR
    statement.setInt(2, age); //age is an INT
    ResultSet rs = statement.executeQuery();
    while (rs.next()){
        //...
    }


Once a `PreparedStatement` object has been created, it can be reused
multiple times for multiple queries (for example, when using the
same query to update multiple rows in a table).  However, they are
**not thread-safe** because of the many method calls involved in
setting the parameters and executing the query.  Therefore, you
should only define `PreparedStatement` objects as method-level
variables (as opposed to class-level variables) to avoid concurrency
issues.

    List<Person>; people = //user input
    Connection connection = DriverManager.getConnection(...);
    connection.setAutoCommit(false);
    try {
        PreparedStatement statement = connection.prepareStatement(
                "UPDATE people SET lastName = ?, age = ? WHERE id = ?");
        for (Person person : people){
            statement.setString(1, person.getLastName());
            statement.setInt(2, person.getAge());
            statement.setInt(3, person.getId());
            statement.execute();
        }
        connection.commit();
    } catch (SQLException e) {
        connection.rollback();
    }

More information on `PreparedStatement` can be found in the
[Oracle JDBC tutorial](http://download.oracle.com/javase/tutorial/jdbc/basics/prepared.html).

Hibernate
---------

[Hibernate](http://www.hibernate.org/) uses named parameters to
safely insert data into a query.  A named parameter consists of a
colon, followed by a unique name for the parameter.

    String name = //user input
    int age = //user input
    Session session = //...
    Query query = session.createQuery("from People where lastName = :name and age > :age");
    query.setString("name", name);
    query.setInteger("age", age);
    Iterator people = query.iterate();

Hibernate also supports positional parameters like `PreparedStatement`,
but named parameters are generally preferred because they make the
query a little easier to read.

See the
[Hibernate Manual](http://docs.jboss.org/hibernate/stable/core/reference/en/html/objectstate.html#objectstate-querying-executing-parameters)
for more information on named parameters.

MyBatis
-------

[MyBatis](http://www.mybatis.org/) is a database framework that
hides a lot of the JDBC code from the developer, allowing him or
her to focus on writing SQL.  The SQL statements are typically
stored in XML files.

MyBatis automatically creates `PreparedStatement`s behind the scenes.
Nothing extra needs to be done by the programmer.

To give you some context, here's an example showing how a basic
query is called with MyBatis.  The input data is passed into the
`PeopleMapper` instance and then it gets inserted into the
"selectPeopleByNameAndAge" query.

XML mapping document
====================

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.bobbytables.mybatis.PeopleMapper">
    <select id="selectPeopleByNameAndAge" resultType="list">
        <!-- lastName and age are automatically sanitized --->
        SELECT * FROM people WHERE lastName = #{lastName} AND age > #{age}
    </select>
    </mapper>

Mapper class
============

    public interface PeopleMapper {
        List<Person> selectPeopleByNameAndAge(@Param("lastName") String name, @Param("age") int age);
    }

Invoking the query
==================

    String name = //user input
    int age = //user input
    SqlSessionFactory sqlMapper = //...
    SqlSession session = sqlMapper.openSession();
    try {
        PeopleMapper mapper = session.getMapper(PeopleMapper.class);
        List<Person> people = mapper.selectPeopleByNameAndAge(name, age); //data is automatically sanitized
        for (Person person : people) {
            //...
        }
    } finally {
        session.close();
    }