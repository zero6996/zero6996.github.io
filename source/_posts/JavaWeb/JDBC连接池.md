---
title: JDBC连接池
date: 2019-5-25 23:59
categories: JavaWeb
tags: [JDBC]
description: 数据库连接和Spring JDBC
urlname: JDBCPool
---



## 1. 数据库连接池

其实就是一个容器(集合)，存放数据库连接的容器。当系统初始化好后，容器被创建，容器中会申请一些连接对象。当用户来访问数据库时，从容器中获取连接对象，用户访问完后，会将连接对象归还给容器。



<!--more-->



### 1.1 好处

1. 节约资源
2. 用户访问高效



### 1.2 实现

标准接口：`DataSource` ，`javax.sql`包下的，主要方法如下：

1. 获取连接：`getConnection()`
2. 归还连接：`close()`。 如果连接对象Connection是从连接池中获取的，那么调用`close()`方法，则不会关闭连接，而是归还连接。

一般我们不去实现它，有数据库厂商来实现

1. C3P0：数据库连接池技术
2. Druid：数据库连接池实现技术，由阿里巴巴提供的



## 2. C3P0数据库连接池技术

### 2.1 导入Jar包

[官方下载地址](https://sourceforge.net/projects/c3p0/)

下载完后解压，然后导入两个包：`c3p0-0.9.5.2.jar`和`mchange-commons-java-0.2.12.jar`

![c3p0](https://raw.githubusercontent.com/zero6996/GitNote-images/master/GitNote/2019/05/25/c3p0-1558796235077.jpg)

> Notice：不要忘记导入数据库驱动Jar包

### 2.2 定义配置文件

- 名称：`c3p0.properties`或者`c3p0-config.xml`
- 路径：将文件放在src目录下即可。



### 2.3 创建核心对象

- 数据库连接池对象：`ComboPooledDataSource`
- 基本使用如下：

```java
public class C3P0Demo1 {
    public static void main(String[] args) throws SQLException {
        // 1. 创建数据库连接池对象
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        // 2. 获取一个连接对象
        Connection conn = dataSource.getConnection();
        // 3. 进行具体操作
        System.out.println(conn);
    }
}
```



## 3. Druid：数据库连接池实现技术

### 3.1 使用步骤

1. 导入jar包：`druid-1.0.9.jar`
2. 定义配置文件：`properties`形式的，可以叫任意名称，可以放任意目录下
3. 加载配置文件：使用`Properties`对象来加载
4. 获取数据库连接池对象：通过工厂类`DruidDataSourceFactory`来获取
5. 获取连接：`getConnection`

### 3.2 基本使用示例

```java
// Druid演示
public class DruidDemo1 {
    public static void main(String[] args) throws Exception {
        // 1. 定义配置文件
        // 2. 加载配置文件
        Properties pro = new Properties();
        InputStream is = DruidDemo1.class.getClassLoader().getResourceAsStream("druid.properties");
        // 3. 获取连接池对象
        DataSource ds = DruidDataSourceFactory.createDataSource(pro);
        // 4. 获取连接
        Connection conn = ds.getConnection();
        // 5. 进行具体操作
        System.out.println(conn);
    }
}
```

### 3.3 定义工具类

1. 定义一个类
2. 提供静态代码块加载配置文件，初始化连接池对象
3. 提供方法
   1. 获取连接方法：通过数据库连接池获取连接
   2. 释放资源
   3. 获取连接池的方法

#### 代码实现

```java
// Druid连接池的工具类
public class DruidUtils {
    // 1. 定义成员变量
    private static DataSource ds;
    // 静态初始化代码块
    static{
        try {
            Properties pro = new Properties();
            pro.load(DruidUtils.class.getClassLoader().getResourceAsStream("druid.properties"));
            // 获取DataSource对象
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
    // 获取连接
    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }
    // 释放资源
    public static void close(Statement stmt,Connection conn){
        close(null,stmt,conn);
    }
    public static void close(ResultSet rs, Statement stmt, Connection conn){
        if (rs!=null){
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (stmt!=null){
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn!=null){
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    // 获取连接池方法
    public static DataSource getDataSource(){
        return ds;
    }
}
// 测试工具类
// 使用工具类
public class DruidDemo2 {
    public static void main(String[] args) {
        // 完成添加操作，给account表添加一条记录
        Connection conn = null;
        PreparedStatement pstmt = null;
        try {
            // 获取连接池对象
            conn = DruidUtils.getConnection();
            // 定义sql
            String sql = "insert into account values(null,?,?)";
            // 获取执行sql对象
            pstmt = conn.prepareStatement(sql);
            // 给？赋值
            pstmt.setString(1,"小六");
            pstmt.setDouble(2,2699);
            // 执行sql
            int count = pstmt.executeUpdate();
            System.out.println(count);
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            DruidUtils.close(pstmt,conn);
        }
    }
}
```



## 4. Spring JDBC

是Spring框架对JDBC的简单封装，提供了一个JDBCTemplate对象简化JDBC的开发

### 4.1 使用步骤

1. 导入Jar包
2. 创建`JDBCTemplate`对象，该对象依赖于数据源`DataSource`，例：`JdbcTemplate template = new JdbcTemplate(ds);`
3. 调用`JDBCTemaplate`的方法来完成CRUD的操作

**常用方法**：

| 方法名             | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| `update()`         | 执行DML语句。增、删、改语句                                  |
| `queryForMap()`    | 查询结果，将结果集封装为Map集合。列名作key，值作value，将这条记录封装为一个Map集合返回。注：该方法查询结果集长度只能是1 |
| `queryForList()`   | 查询结果，将结果集封装为List集合。将每一条记录封装为一个Map集合，再将Map集合装载到List集合中 |
| `queryForObject()` | 查询结果，将结果封装为对象。一般用于聚合函数的查询           |
| `query()`          | 查询结果，将结果封装为JavaBean对象。一般我们使用`new BeanPropertyRowMapper<类型>(类型.class)`来完成数据到JavaBean的自动封装。 |

### 4.2 使用示例

```java
// jdbcTemplate基础入门
public class JDBCTemplateDemo1 {
    public static void main(String[] args) {
        // 创建jdbcTemplate对象
        JdbcTemplate jt = new JdbcTemplate(DruidUtils.getDataSource());
        // 调用方法
        String sql = "update account set balance=3000 where id=?";
        int count = jt.update(sql, 6); // 返回值是影响的行数
        System.out.println(count); // 1
    }
}

```

### 4.3 练习

1. 修改6号数据的salary为10000
2. 添加一条记录
3. 删除刚刚添加的记录
4. 查询id为8的记录，将其封装为Map集合
5. 查询所有记录，将其封装为List
6. 查询所有记录，将其封装为Emp对象的List集合
7. 查询总记录数

```java
public class Exercise {
        // 获取JdbcTemplate对象
        // 使用Junit单元测试，让方法单独执行
        private JdbcTemplate jt = new JdbcTemplate(DruidUtils.getDataSource());
        // 编写sql语句并执行
        @Test
        public void test1(){
            // 1. 修改6号数据的salary为10000
            String update = "update staff set salary=10000 where id=?";
            jt.update(update,6);
        }
        @Test
        public void test2(){
            // 2. 添加一条记录,字段分别为id，name，性别，工资，入职时间，部门id
            jt.update("insert into staff(id,name,salary) values(?,?,?)",13,"冬兵",7889);
        }
        @Test
        public void test3(){
            // 3. 删除刚刚添加的记录
            jt.update("delete from staff where name=?","冬兵");
        }
        @Test
        public void test4(){
            // 4. 查询id为8的记录，将其封装为Map集合
            // 注意：这个方法查询的结果集只能是1
            Map<String, Object> stringObjectMap = jt.queryForMap("select * from staff where id=?", 8);
            System.out.println(stringObjectMap); // {id=8, name=美队, gender=男, salary=8000.0, join_date=2008-08-08, dept_id=2}
        }
        @Test
        public void test5(){
            // 5. 查询所有记录，将其封装为List
            List<Map<String, Object>> maplist = jt.queryForList("select * from staff");
            for (Map<String, Object> stringObjectMap : maplist) {
                System.out.println(stringObjectMap);
            }
        }
        @Test
        public void test6(){
            // 6. 查询所有记录，将其封装为Emp对象的List集合，自己完成实现类
            String sql = "select * from staff";
            List<Emp> list = jt.query(sql, new RowMapper<Emp>() {
                @Override
                public Emp mapRow(ResultSet rs, int i) throws SQLException {
                    Emp emp = new Emp();
                    int id = rs.getInt("id");
                    String name = rs.getString("name");
                    String gender = rs.getString("gender");
                    double salary = rs.getDouble("salary");
                    Date join_date = rs.getDate("join_date");
                    int dept_id = rs.getInt("dept_id");
                    emp.setId(id);
                    emp.setName(name);
                    emp.setGender(gender);
                    emp.setSalary(salary);
                    emp.setJoin_date(join_date);
                    emp.setDept_id(dept_id);
                    return emp;
                }
            });
            for (Emp emp : list) {
                System.out.println(emp);
            }
        }
        @Test
        public void test6_2() {
            // 6. 查询所有记录，将其封装为Emp对象的List集合,使用jdbc提供的实现类来完成
            String sql = "select * from staff";
            List<Emp> list = jt.query(sql, new BeanPropertyRowMapper<Emp>(Emp.class));
            for (Emp emp : list) {
                System.out.println(emp);
            }
            /*
            Failed to convert property value of type 'null' to required type 'int' for property 'dept_id'
            需要将Emp类中变量改为为引用数据类型,然后重新生成方法
             */
        }
        @Test
        public void test7(){
            // 7. 查询总记录数
            String sql = "select count(id) from staff";
            Long count = jt.queryForObject(sql, Long.class);
            System.out.println(count);
        }
}
```



