# SpringData JPA

## JpaRepository继承关系

![JpaRepository继承关系](D:\0_LeargingSummary\SpringBoot\images\JpaRepository继承关系.png)



## springboot设置打印sql语句

> **spring.jpa.show-sql=true**

## 自定义查询

### 按照自定义名称查询

必须以find开头，By后面接查询条件，驼峰命名

+ And -- findByUsernameAndPassword(String user, Striang pwd);
+ Or -- findByUsernameOrAddress(String user, String addr);
+ Between -- findBySalaryBetween(int max, int min)；
+ LessThan -- 等价于 SQL 中的 "<",findBySalaryLessThan(int max)；
+ GreateThan -- 等价于 SQL 中的">"，findBySalaryGreaterThan(int min)；
+ IsNull -- findByUsernameIsNull()；
+ IsNotNull -- findByUsernameIsNotNull()；
+ NotLike  -- findByUsernameNotLike(String user)
+ OrderBy  -- findByUsernameOrderBySalaryAsc(String user)
+ Not -- 等价于 SQL 中的 "！ ="，比如 findByUsernameNot(String user)
+ In -- findByUsernameIn(Collection<String> userList),参数可以是数组，或不定长参数



### 自定义Sql-@Query

```java
//使用HQL语句，查询
//占位符?1表示选用第一个参数
@Query("select p from Person p where p.id < ?1")
	List<Person> selectById(Integer id);

//分页查询
//设置分页条件：PageRequest pageInfo = PageRequest.of(page, size);
	@Query("select p from Person p")
	List<Person> selectByPage(Pageable pageable);

//更新操作，依旧使用@Query注解，需要加上@Modifying,并添加事务@Transactional
	@Transactional
	@Modifying
	@Query("update Person p set p.username=?1 where p.id=?2")
	int upUsernameInfoById(String name, Integer id);

//删除操作，使用原始sql语句操作，需要早@Query中设置nativeQuery为true
	@Transactional
	@Modifying
	@Query(value = "delete from person where id=?1", nativeQuery = true)
	int delById(Integer id);
```





