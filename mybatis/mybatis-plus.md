---
title: mybatisplus整理
date: 2022-08-14
tags: [mybatisplus,mybatis]
toc: true
---

# 1、MyBatis-Plus

> mybatis 简化 JDBC，myBatisplus 简化 mybatis，是一种增强工具。
>
> 官方文档：https://baomidou.com/pages/24112f/#%E7%89%B9%E6%80%A7

MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

<!--more-->

##  1.1、特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑；
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作；
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求；
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错；
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题；
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作；
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）；
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用；
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询；
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库；
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询；
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作；

# 2、Hello, Mybatis-plus!

**使用步骤**

```
1、导入依赖
2、配置数据库文件
3、自定义Mapper实现BaseMapper接口
4、在主启动类上使用MapperScan注解扫描Mapper文件夹：@MapperScan("com.haining820.mapper")
5、在测试类中直接进行测试，增删改查有对应的方法自动实现
```

真实开发中的数据库字段：version（乐观锁）、deleted（逻辑删除）、gmt_create/gmt_modified（创建/更新时间）

## 2.1、增

```
insert
```

## 2.2、删

## 2.3、改

使用 updateById 方法可以更新对象，需要注意实体类中的各字段变量类型，以 age 属性为例，在数据库中它的类型是 int：

- 实体类中使用 int，在更新的对象中不设置 age 值的话默认是 0，会导数据库中的 age 也会变成 0;
- 实体类中使用 Integer，在更新的时候对象中不设置 age 值，mybatisplus 会自动进行 sql 的拼接，跳过 age，不会影响数据库中的字段。

```java
    @Test
    public void testUpdate() {
        Student student = new Student();
        student.setId(1);
//        student.setAge(8);
        student.setName("暖羊羊");
        int result = studentMapper.updateById(student);
        LOGGER.info("testUpdate,result->" + result);
    }
```

## 2.4、查

```
selectList
```

# 3、mybatisplus 使用技巧

## 3.1、主键生成策略

<font size=4 style="font-weight:bold;background:yellow;">主键生成策略</font>

- 主键 id 生成算法：uuid、自增 id、雪花算法、redis、zookeeper

  分布式系统唯一 id 生成方案：https://www.cnblogs.com/haoxinyue/p/5208136.html

- 实体类中的 id 字段对应数据库中的主键，未配置注解时

  - 当实体类主键使用 Long 时 mybatisplus 会自动生成主键，这里采用的是雪花算法；
  - 当实体类主键使用 int 时 mybatisplus 生成的默认值是 0。

<font size=4 style="font-weight:bold;background:yellow;">雪花算法</font>

- snowflake 是 Twitter 开源的分布式 ID 生成算法，结果是一个 long 型的 ID，可以保证几乎全球唯一。

- 核心思想：使用 41bit 作为毫秒数，10bit 作为机器的 ID（5 个 bit 是数据中心，5 个 bit 的机器 ID），12bit 作为毫秒内的流水号（意味着每个节点在每毫秒可以产生 4096 个 ID），最后还有一个符号位，永远是 0。具体实现的代码可以参看https://github.com/twitter/snowflake。雪花算法支持的 TPS 可以达到 419 万左右（2^22*1000）。

<font size=4 style="font-weight:bold;background:yellow;">@TableId 注解使用方式</font>

```java
@TableId(type = IdType.ID_WORKER)
private Long id;
```

<font size=4 style="font-weight:bold;background:yellow;">IdType 枚举类</font>

定义生成 id 的类型

- `AUTO(0)`：数据库 id 自增（数据库字段必须是自增的）
- `NONE(1)`：不使用主键，数据库未设置主键；
- `INPUT(2)`：手动输入，要自己配置 id，测试时不设置 id 传入的 id 是null；

以下3种类型、只有当插入对象 ID 为空，才自动填充。

- `ASSIGN_ID(3)`：分配 ID，使用雪花算法（主键类型为 number 或 string，@since 3.3.0）
- `ASSIGN_UUID(4)`：分配UUID（主键类型为 string）
- `ID_WORKER(3)/ID_WORKER_STR(3)/UUID(4);`：分别是雪花算法、字符串雪花算法、uuid，在测试的 3.3.1 版本中提示已经废弃，使用 ASSIGN_ID、ASSIGN_UUID 替代。

## 3.2、自动填充

所有的表都要配置 `创建时间/修改时间`（`gmt_create/gmt_modified`）等属性的操作自动化完成，不要手动去更新（alibaba 开发手册）！

<font size=4 style="font-weight:bold;background:yellow;">数据库级别</font>

在表中新增字段 create_time/update_time，进行默认值和更新的设置。

```sql
`create_time` datetime DEFAULT CURRENT_TIMESTAMP,
`update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
```

设置完之后测试插入，从日志中可以看到，执行的 sql 语句中没有与时间相关的内容，但是数据库表中的数据可以自动更新。

<font size=4 style="font-weight:bold;background:yellow;">代码级别</font>

> 首先删除刚刚设置的数据库时间字段的默认值还有更新的操作

- 在实体类字段上增加注解 @TableField（FieldFill 枚举类：DEFAULT、INSERT、UPDATE、INSERT_UPDATE）

    ```java
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
    ```

- 编写处理器处理 @TableField 注解

    ```java
    @Slf4j
    @Component  // 一定注意要把处理器加到ioc容器中
    public class MyHandler implements MetaObjectHandler {
        // 重写插入、更新时的填充策略，接口中有相关的默认方法可供使用
        @Override
        public void insertFill(MetaObject metaObject) {
            log.info("start insert fill ....");
            this.setFieldValByName("createTime", new Date(), metaObject);
            this.setFieldValByName("updateTime", new Date(), metaObject);
        }
        @Override
        public void updateFill(MetaObject metaObject) {
            log.info("start update fill ....");
        this.setFieldValByName("updateTime", new Date(), metaObject);
        }
    }
    ```
    
- 测试插入更新，观察数据库时间字段，在数据库层面没有进行设置仍然能够实现自动更新。

# 4、乐观锁

乐观锁：更新记录时，希望这条记录没有被别人更新，认为总是不会出现问题，无论干什么都不会上锁，如果出现问题就再次尝试更新；

悲观锁：认为总是会出现问题，无论干什么都会上锁后再去操作。

<font size=4 style="font-weight:bold;background:yellow;">version 版本号的实现</font>

- 取出记录时，获取当前 version；
- 更新时，带上这个 version；
- 执行更新时， `set version = newVersion where version = oldVersion`
- 如果 version 不对，就更新失败。

**说明**

- 支持的数据类型只有：int、Integer、long、Long、Date、Timestamp、LocalDateTime
- 整数类型下 `newVersion = oldVersion + 1`
- `newVersion` 会回写到 `entity` 中
- 仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法
- 在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用！

<font size=4 style="font-weight:bold;background:yellow;">乐观锁的配置</font>

比方说 A 线程现在正在进行操作，即将要对某用户进行更新，但是该用户被 B 线程抢先更新了，这个时候 version 就变，A 线程将无法进行更新。

- 首先在数据库中加上对应的 version 字段；

- 然后在实体类上添加对应的注解；

  ```java
  @Version
  private Integer version;
  ```

- 注册组件

  ```java
  @Configuration  // 配置类
  @EnableTransactionManagement    // 开启事务
  @MapperScan("com.haining820.mapper")
  public class MybatisPlusConfig {
      // 注册乐观锁插件
      @Bean
      public OptimisticLockerInterceptor optimisticLockerInterceptor(){
          return new OptimisticLockerInterceptor();
      }
  }
  ```


<font size=4 style="font-weight:bold;background:yellow;">测试</font>

- 重复提交的更新请求只有第一次提交会成功，因为 yhyy11 更新之后会将 version 设置为 version+1，yhyy22 更新时还是根据之前获取到的 version 进行查询（`update *** set ***=***,***=***... where id=*** and version=***;`），查询不到i就会更新失败；
- 在某些特殊情况下可能需要将会第二条信息也成功插入，这时候可以进行判断，可以将对象的版本号加 1，再次添加，或者是直接重新获取对象。

```java
@Test
public void testOptimisticUnderMultiThread() {
    Student student = studentMapper.selectById(2);
    student.setName("yhyy11");
    student.setContent("test version11");
    
    Student student2 = studentMapper.selectById(2);
    student2.setName("yhyy22");
    student2.setContent("test version22");
    studentMapper.updateById(student2);

    studentMapper.updateById(student);
}
```

**三个线程（A、B、main）同时对数据库中的同一行数据进行操作，先运行完的插入成功，其余两个线程会失败。**

```java
@Test
public void testOptimisticUnderMultiThread2() {
    new Thread(() -> {
        Student student = studentMapper.selectById(6);
        student.setName("yhyy in AAA");
        student.setAge(student.getAge() + 10);
        student.setContent("test version in AAA");
        int i = studentMapper.updateById(student);
        LOGGER.info(Thread.currentThread().getName() + "->" + i);
    }, "A").start();
    new Thread(() -> {
        Student student = studentMapper.selectById(6);
        student.setName("yhyy in BBB");
        student.setAge(student.getAge() - 3);
        student.setContent("test version in BBB");
        int j = studentMapper.updateById(student);
        LOGGER.info(Thread.currentThread().getName() + "->" + j);
    }, "B").start();
    Student student = studentMapper.selectById(6);
    student.setName("yhyy in main");
    student.setAge(student.getAge() + 80);
    student.setContent("test version in main");
    int k = studentMapper.updateById(student);
    LOGGER.info(Thread.currentThread().getName() + "->" + k);
}
```

# 5、分页查询

分页方式：limit、PageHelper...

**mybatisplus 配置分页插件**

```java
@Bean
public PaginationInterceptor paginationInterceptor() {
    return new PaginationInterceptor();
}
```

**测试**

```java
@Test
public void testPage() {
    // 设置当前页和页面大小
    Page<Student> page = new Page<>(1, 3);
    // queryWrapper参数与高级查询有关
    studentMapper.selectPage(page, null);
    for (Student s : page.getRecords()) {
        LOGGER.info(s.toString());
    }
    LOGGER.info("page.getTotal()->" + page.getTotal());
}
```













































