# MyBatis总结

## 一、获取SqlSessionFactory对象

### SqlSeesionFactory初始化流程

#### 1、加载全局配置文件

![](\images\mybatis加载config配置文件信息.png)

#### 2、解析得到Configuration配置对象，包含全局配置和xxxMapper.xml配置信息

![](\images\返回加载完的mybatis配置的configuration.png)

![](\images\解析出Mapper文件的位置.png)

#### 3、一个MappedStatement代表了一个增删改查的详细信息

![](\images\Configuration中保存的Mapper内容.png)

![](\images\MyBatis中SqlSessionFactory初始化时序图.png)

## 二、获取sqlSession对象

![](\images\mybatis获取sqlSession时序图.png)

## 三、获取接口的代理对象

![](\images\mybatis获取接口的代理对象时序图.png)

代理对象中包含了DefaultSqlSession对象

![](\images\Mapper接口代理对象中包含了sqlSession对象.png)



## 四、执行增删改查方法

![](\images\mybatis执行sql时序图.png)



> #### StatementHandler：处理sql语句预编译，设置参数等相关工作； ParameterHandler：设置预编译参数用的
> ResultHandler：处理结果集
> TypeHandler：在整个过程中，进行数据库类型和javaBean类型的映射

![](\images\查询流程总结.png)

## 五、流程总结

> #### 1、根据配置文件（全局配置文件、sql映射）初始化出Configuration对象，将其包装在SqlSessionFactory（DefaultSqlSessionFactory）对象中
>
> #### 2、创建一个DefaultSqlSession对象，里面包含了Configuration以及Executor(根据全局配置文件的defaultExectuorType创建出对应的Executor)
>
> #### 3、DefaultSqlSession.getMapper(....)获取对应的接口的xxxMapperProxy对象
>
> #### 4、MapperProxy里面有（DefaultSqlSession）
>
> #### 5、执行增删改查方法：
>
> #### 	       	1）、调用DefaultSqlSession中的Execution对象的增删改查
>
> #### 	2）、会创建一个StatementHandler对象。（同时也会创建出ParameterHandler和ResultSetHandler）
>
> #### 	3)、调用StatementHandler预编译参数以及设置参数值；使用ParameterHandler来给sql设置参数	
>
> #### 	4）、调用StatementHandler的增删改查方法
>
> #### 	5）、ResultSetHandler封装结果
>
> #### 注：四大对象每个创建的时候都有一个interceptorChain.pluginAll(parameterHandler)
>
> 

