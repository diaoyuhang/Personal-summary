# 什么是SPI

​	SPI的全名为Service Provider Interface,为了实现在模块装配的时候不需要在程序里动态的指明，需要一种服务发现机制。JAVA SPI就是提供这一机制：为某个接口寻找服务实现的机制。

​	具体的约定：当服务提供者提供了服务接口的一种实现，在jar包的META-INF/services/目录下创建一个以服务接口全类名的文件，文件中保存该接口实现类的全类名。

​	当外部程序装配这个模块的时候，就通过META-INF/services/中的配置文件加载具体的实现类名，并装载实例化。jdk中提供这种服务实现查找工具类：java.util.ServiceLoader。

![](images\SPI.png)

平时最常见、典型的例子就是java.sql.Driver:

![](images\SPI例子.png)

# JDBC案例分析

平时连接数据库最常见的写法：

```java
// 加载Class到AppClassLoader（系统类加载器），然后注册驱动类
// Class.forName("com.mysql.jdbc.Driver").newInstance(); 
String url = "jdbc:mysql://localhost:3306/testdb";    
// 通过java库获取数据库连接
Connection conn = java.sql.DriverManager.getConnection(url, "name", "password");
```

可是现在将Class.forName注释掉也是一样的使用，这里面就用到了上面的SPI机制，重点就在DriverManager.getConnection中。

```java
public class DriverManager {
    
    static {
        //加载初始化驱动程序
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }

    private static void loadInitialDrivers() {
            
        String drivers;
        try {
            drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
                public String run() {
                    return System.getProperty("jdbc.drivers");
                }
            });
        } catch (Exception ex) {
            drivers = null;
        }

        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
				//通过SPI加载驱动类，这里面用到ThreadContextClassLoader加载
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {

                }
                return null;
            }
        });

        if (drivers == null || drivers.equals("")) {
            return;
        }
        String[] driversList = drivers.split(":");
        println("number of Drivers:" + driversList.length);
        for (String aDriver : driversList) {
            try {
                //通过ClassLoader加载驱动类
                Class.forName(aDriver, true,
                        ClassLoader.getSystemClassLoader());
            } catch (Exception ex) {
                println("DriverManager.Initialize: load failed: " + ex);
            }
        }
    }
}
```

加载了驱动器之后

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    //
    // Register ourselves with the DriverManager
    //
    static {
        try {
            //注册驱动类，将Driver封装到DriverInfo里，用CopyOnWriteArrayList集合来保存
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

