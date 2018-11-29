---
title: 重温数据库操作JDBC
date: 2018-11-11 13:07:57
tags: 
- JDBC 
- MySQL 
- SQL 
categories: 
- 数据库
---
# 前言
        在日常的开发当中，我们总是用框架去连接数据库，去完成业务中的需求，但是不可避免的有时候我们需要对数据进行额外的处理，
    批量的修改或查询，这时候用框架就显得有点杀鸡用牛刀了，在这里我们将重温原始的JDBC操作，好记性不如烂笔头。
# JDBC操作步骤
        1.注册驱动
        2.获取连接
        3.获取预处理语句
        4.添加占位符
        5.结果遍历或者查询修改记录条数
        6.释放资源
# 代码
        1.导入数据库连接需要的jar包，这里使用maven
```
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
```
        2.增删改查的测试
```
        import org.junit.Test;

        import java.sql.*;

        public class DBTest {

            /**
            * 增
            *
            * @throws ClassNotFoundException
            * @throws SQLException
            */
            @Test
            public void test01() throws ClassNotFoundException, SQLException {
                //1.注册驱动
                Class.forName("com.mysql.jdbc.Driver");
                //2.获取连接
                Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "asdfmin");
                //3.获得预处理语句
                String sql = "insert into user (name, sex, age) values (? , ?, ?)";
                PreparedStatement preparedStatement = conn.prepareStatement(sql);
                //4.添加占位符
                preparedStatement.setString(1, "jack");
                preparedStatement.setString(2, "男");
                preparedStatement.setString(3, "18");
                //5.执行预处理语句
                int i = preparedStatement.executeUpdate();
                System.out.println("添加新记录条数：" + i);
                //6.释放资源
                preparedStatement.close();
                conn.close();
            }

            /**
            * 更
            *
            * @throws ClassNotFoundException
            * @throws SQLException
            */
            @Test
            public void test02() throws ClassNotFoundException, SQLException {
                //1.注册驱动
                Class.forName("com.mysql.jdbc.Driver");
                //2.获取连接
                Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "asdfmin");
                //3.获取预处理语句
                String sql = "update user set age = ? where name = ?";
                PreparedStatement preparedStatement = conn.prepareStatement(sql);
                //4.添加占位符
                preparedStatement.setString(1, "19");
                preparedStatement.setString(2, "jack");
                //5.执行预处理语句
                int i = preparedStatement.executeUpdate();
                System.out.println("更新记录条数：" + i);
                //6.释放资源
                preparedStatement.close();
                conn.close();
            }

            /**
            * 删
            *
            * @throws ClassNotFoundException
            * @throws SQLException
            */
            @Test
            public void test03() throws ClassNotFoundException, SQLException {
                //1.注册驱动
                Class.forName("com.mysql.jdbc.Driver");
                //2.获取连接
                Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "asdfmin");
                //3.获取预处理语句
                String sql = "delete from user where id = ?";
                PreparedStatement preparedStatement = conn.prepareStatement(sql);
                //4.添加占位符
                preparedStatement.setString(1, "1");
                //5.执行预处理
                int i = preparedStatement.executeUpdate();
                System.out.println("删除记录条数：" + i);
                //6.释放资源
                preparedStatement.close();
                conn.close();
            }

            /**
            * 查
            * @throws ClassNotFoundException
            * @throws SQLException
            */
            @Test
            public void test04() throws ClassNotFoundException, SQLException {
                //1.注册驱动
                Class.forName("com.mysql.jdbc.Driver");
                //2.获取连接
                Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "asdfmin");
                //3.获取预处理语句
                //String sql = "select * from user";
                String sql = "select * from user where name = ?";
                PreparedStatement preparedStatement = conn.prepareStatement(sql);
                //4.添加站位符
                preparedStatement.setString(1, "jack");
                //5.执行预处理
                ResultSet resultSet = preparedStatement.executeQuery();
                //6.遍历结果集
                while (resultSet.next()) {
                    int id = resultSet.getInt("id");
                    String name = resultSet.getString("name");
                    String sex = resultSet.getString("sex");
                    String age = resultSet.getString("age");
                    System.out.println("id:" + id + "   name:" + name + "  set:" + sex + "  age:" + age);
                }
                //7.关闭资源
                resultSet.close();
                preparedStatement.close();
                conn.close();
            }

        }

```
        3.说明
        这里用的是预处理语句PreparedStatement（防止sql注入攻击）
# 对于JDBC的简单封装
        对于频繁的注册驱动获取连接我们可以进行简单的封装
```
        import java.sql.Connection;
        import java.sql.DriverManager;
        import java.sql.SQLException;

        public class JDBCUtils {

            private static final String DRIVERNAME = "com.mysql.jdbc.Driver";
            private static final String URL = "jdbc:mysql://localhost:3306/test";
            private static final String USER = "root";
            private static final String PASSWORD = "asdfmin";

            static {
                try {
                    //1.注册驱动
                    Class.forName(DRIVERNAME);
                } catch (ClassNotFoundException e) {
                    System.out.println("注册驱动失败");
                    e.printStackTrace();
                }
            }

            public static Connection getConn() throws SQLException {
                //2.获取连接
                Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
                //返回连接
                return conn;
            }

        }
```
# 测试封装的JDBC工具类
```
        import org.junit.Test;

        import java.sql.Connection;
        import java.sql.PreparedStatement;
        import java.sql.ResultSet;
        import java.sql.SQLException;

        public class DBUtilTest {

            /**
            * 测试封装的JDBCUtil
            */
            @Test
            public void test01() {
                Connection conn = null;
                PreparedStatement preparedStatement = null;
                ResultSet resultSet = null;
                try {
                    //1.获取连接
                    conn = JDBCUtils.getConn();
                    //2.获取执行语句
                    String sql = "select * from user";
                    preparedStatement = conn.prepareStatement(sql);
                    //3.添加占位符
                    //4.执行预处理语句
                    resultSet = preparedStatement.executeQuery();
                    //5.遍历结果集
                    while (resultSet.next()) {
                        String name = resultSet.getString("name");
                        System.out.println("name:" + name);
                    }
                } catch (SQLException e) {
                    e.printStackTrace();
                } finally {
                    //6.释放资源
                    try {
                        resultSet.close();
                        preparedStatement.close();
                        conn.close();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }
                }
            }

        }
```    
# API的一些简单说明
        执行预处理语句的API
        int executeUpdate(); --执行insert update delete语句.
        ResultSet executeQuery(); --执行select语句.
        boolean execute(); --执行select返回true 执行其他的语句返回false.
# 后记
        温故而知新。
        

        