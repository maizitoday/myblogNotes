---
title:       "MyBatis系列-框架简单实现"
subtitle:    ""
description: ""
date:        2019-08-21
author:      "麦子"
image:       "https://c.pxhere.com/images/1f/8d/7a12ae7fcd86cdc43f624556db3b-1451061.jpg!d"
tags:        ["数据库", "mybatis系列"]
categories:  ["Tech" ]
---

[TOC]

**特别说明：以下代码来源网上，来源地址已经丢失。**

# 入口类



```java
public class APP {

    public static void main(String[] args){
        MySqlSession  sqlSession = new MySqlSession();
        // jdk代理
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.getUserById("1");
        System.out.println("  user:   "+user);
    }

}

```





# jdk代理



## MySqlSession.java

```java
public class MySqlSession {

    private Excutor  excutor = new MyExcutor();

    private MyConfiguration myConfiguration = new MyConfiguration();

    public <T> T selectOne(String sql, Object parameter)
    {
        return  excutor.query(sql,parameter);
    }


    public <T> T getMapper(Class<T> clas){

        return (T)Proxy.
              newProxyInstance(clas.getClassLoader(),
                               new Class[]{clas},
                               new MyMapperProxy(this,myConfiguration));
    }
}
```





## MyMapperProxy.java 

```java
public class MyMapperProxy implements InvocationHandler{


    private MySqlSession  mySqlSession;
    private MyConfiguration myConfiguration;

    public MyMapperProxy(MySqlSession mySqlSession, MyConfiguration myConfiguration) {
        this.mySqlSession = mySqlSession;
        this.myConfiguration = myConfiguration;
    }

    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 解析xml文档，封装到MapperBean对象中
        MapperBean mapperBean = myConfiguration.readMapper();
        // 判断这个接口类的路径和xml的nameSpace是否一致
        if (!method.getDeclaringClass().getName().equals(mapperBean.getInterfaceName()))
        {
            return null;
        }
        List<Function> list = mapperBean.getList();
        if (list != null && list.size() > 0)
        {
            for (Function function : list)
            {
                 // 这里就进行判断接口类里面的方法的名称和xml配置文件里面的id是否一致
                if (method.getName().equals(function.getFuncName()))
                {  
                    // 实际执行的sql语句
                    return mySqlSession.selectOne(function.getSql(),String.valueOf(args[0]));
                }
            }
        }
        return null;
    }
}
```



# 数据库连接





## MyConfiguration.java

处理数据库连接和xml配置文件的对象化封装。

```java
public class MyConfiguration {

    private static  String driver = "com.mysql.jdbc.Driver";
    private static String url = "jdbc:mysql://127.0.0.1:3306/sys?useUnicode=true&characterEncoding=utf-8&useSSL=false";
    private static String username = "root";
    private static String password = "123456";
    // 用于管理数据库连接池，ThreadLocal能够实现当前线程的操作都是用同一个Connection，保证了事务！
    private static ThreadLocal<Connection> connContainer = new ThreadLocal<Connection>();

    /**
     * 获取数据库连接
     * @return
     */
    public Connection getConnection() throws Exception{
        Connection conn = connContainer.get();
        Class.forName(driver);
        conn = DriverManager.getConnection(url,username,password);
        conn.setAutoCommit(false);
        return conn;
    }

    /***
     *  关闭链接
     */
    public void closeConnection() throws Exception
    {
        Connection conn = connContainer.get();
        if (conn != null)
        {
            conn.close();
        }
        connContainer.remove();
    }

    /***
     * 读取写入SQL的配置文件，获取他的SQL语句，返回值，传入的参数
     * @param path
     * @return
     */
    public MapperBean readMapper()
    {
        ClassLoader  loader = ClassLoader.getSystemClassLoader();
        MapperBean  mapperBean = new MapperBean();
        try {
            InputStream stream =  MyConfiguration.class.getResourceAsStream("/maizi/UserMapper.xml");
            SAXReader reader = new SAXReader();
            Document document = reader.read(stream);
            Element root = document.getRootElement();
            // 把nameSpace值为接口名
            mapperBean.setInterfaceName(root.attributeValue("nameSpace").trim());
            // 存储方法
            List<Function> list = new ArrayList<Function>();
            // 遍历根节点下面所有节点
            for(Iterator rootIter = root.elementIterator();rootIter.hasNext();)
            {
              Function fun = new Function();
              Element e = (Element) rootIter.next();
              String sqltype = e.getName().trim();
              String funcName = e.attributeValue("id").trim();
              String sql = e.getText().trim();
              String  resultType = e.attributeValue("resultType").trim();
              // sql 类型是 查询还是修改等
              fun.setSqlType(sqltype);
              fun.setFuncName(funcName);
              Object newInstance = null;
              // 在这里直接实例化返回的对象
              newInstance = Class.forName(resultType).newInstance();
              fun.setResultType(newInstance);
              fun.setSql(sql);
              System.out.println("   fun:  "+fun.toString());
              list.add(fun);
            }
            mapperBean.setList(list);
        }catch (Exception e){
           e.printStackTrace();
        }
        return mapperBean;
    }
}

```



# 配置文件处理





## UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mapper nameSpace="mapper.UserMapper">
    <select id="getUserById" resultType ="mapper.User">
        select * from user where id = ?
    </select>
</mapper>
```





## MapperBean.java

```java
@Data
public class MapperBean {
    private String interfaceName; // 接口名
    private List<Function> list; // 接口下所有方法
}
```





## Function.java

```java
@Data
@ToString
public class Function {
    private String sqlType;
    private String funcName;
    private String sql;
    private Object resultType;
    private String parameterType;
}
```





## User.java

```java
@Data
@ToString
public class User {
  private String id;
  private String userName;
  private String passWord;
}
```





# 接口类



## UserMapper.java

```java
public interface UserMapper {

    public User getUserById(String id);

}
```



# 实际执行sql语句类



## MyExcutor.java

```java
public class MyExcutor  implements Excutor{

    private MyConfiguration  myConfiguration = new MyConfiguration();

    @Override
    public <T> T query(String sql, Object parameter) {
        try {
            Connection connection = myConfiguration.getConnection();
            ResultSet set = null;
            PreparedStatement pre = null;
            pre = connection.prepareStatement(sql);
            pre.setString(1,parameter.toString());
            set = pre.executeQuery();
            User u = new User();
            while (set.next())
            {
                u.setId(set.getString(1));
                u.setUserName(set.getString(2));
                u.setPassWord(set.getString(3));
            }
            return (T)u;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```



# 总结



解析xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mapper nameSpace="mapper.UserMapper">
    <select id="getUserById" resultType ="mapper.User">
        select * from user where id = ?
    </select>
</mapper>
```

获取里面的nameSpace，然后根据前面的jdk代理传入的

```java
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
```

为了就是找到xml配置文件的**id和接口的方法对应**， 

```java
User user = mapper.getUserById("1");
```

然后在代理类里面进行实际的sql的操作和返回数据和对象的填充。