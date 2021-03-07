[TOC]

# TSF 异常、事务问题复现

表现为：

1. 不抛异常，代码能够接着走下去
2. 事务控制不生效

### 复现

#### 额外引入的依赖

``` xml
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>${mybatis-plus-boot-starter.version}</version>
            </dependency>

            <dependency>
                <!--公司的一个基础依赖-->
            </dependency>
```

#### 修改 `sqlSessionFactory` 的 bean 为 mybatis plus 提供

修改 biz 包下的 `db.xml` 文件：

原始的：

``` xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
```

修改后：

``` xml
<bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
    
    ...
    <bean class="xxx.mybatisplus.plugins.BatchUpdateInterceptor">
                    <description>SQL批量更新拦截器</description>
    </bean>
    <bean id="hsafGlobalConfig" class="com.baomidou.mybatisplus.core.config.GlobalConfig">
        <description>SQL全局配置</description>
        <property name="dbConfig" ref="hsafDbConfig"/>
        <property name="sqlInjector" ref="hsafLogicSqlInjector"/>
    </bean>
    <bean id="hsafDbConfig"
          class="com.baomidou.mybatisplus.core.config.GlobalConfig.DbConfig">
        <description>SQL全局数据库配置</description>
        <property name="idType" value="ASSIGN_UUID"/>
        <property name="logicDeleteValue" value="0"/>
        <property name="logicNotDeleteValue" value="1"/>
    </bean>
    <bean id="hsafLogicSqlInjector"
          class="xxx.mybatisplus.injector.LogicSqlInjector">
        <description>SQL注入器</description>
    </bean>
```

### 导入 nes 出问题的代码（test 包）

swagger 调用 `cn/hsa/mbs/nes/test` 接口，json 如下：

``` json
[
  {
    "id": 1,
    "status": 1,
    "username": "string"
  },
  {
    "id": 2,
    "status": 0,
    "username": "AAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBlksakjwqhekjahdskjwqsadBBBBBBBBBBCCCCCCCCC"
  },
  {
    "id": 3,
    "status": 0,
    "username": "string"
  }
]
```

未抛出异常：

![image-20201110205141691](TSF 异常、事务问题复现.assets/image-20201110205141691.png)

控制台无异常信息，bo 层后续代码被执行（红框标记）：

![image-20201110205234928](TSF 异常、事务问题复现.assets/image-20201110205234928.png)

数据被插入了（未被事务控制）：

![image-20201110205318192](TSF 异常、事务问题复现.assets/image-20201110205318192.png)

### 期望的正常情况

以开源环境为例，期望 TSF 能和开源环境运行效果一致。

能够返回异常：

![image-20201110210501624](TSF 异常、事务问题复现.assets/image-20201110210501624.png)

能够输出异常信息：

![image-20201110210809929](TSF 异常、事务问题复现.assets/image-20201110210809929.png)

事务能够回滚（未插入新数据）：

![image-20201110210841956](TSF 异常、事务问题复现.assets/image-20201110210841956.png)