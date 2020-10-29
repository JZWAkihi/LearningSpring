第一章 引言

#### 1、EJB存在的问题

![EJB存在的问题](images\EJB存在的问题.png)

#### 2、什么是Spring

> Spring是一个轻量级的JavaEE解决方案，整合众多优秀的设计模式

- 轻量级

```markdown
1、 对于运行环境没有额外要求
	开源 tomcat resion jetty
	收费weblogic websphere
2、 代码移植性高
	不需要额外的接口
```



- JavaEE的解决方案

![Spring解决方案](images\Spring解决方案.png)

- 整合设计模式

```markdown
1、 工厂
2、 代理
3、 模板
4、 策略
```



#### 3、设计模式

```markdown
1. 广义概念
	面向对象设计中，解决特定问题的经典代码
2. 狭义模式
	GOF4人帮定义的23种设计模式：工厂、适配器、装饰器、门面、代理....
```



#### 4、工厂设计模式

##### 4.1、什么是工厂设计模式

```markdown
1. 概念：通过工厂类，创建对象
		User user = new User();
		UserDAO userDAO = new UserDAOImpl();
		
2. 好处：解耦合
	耦合：指定代码间的强关联关系，一方的改变会影响到另一方
	问题：不利于代码维护
	简单：把接口的实现类，硬编码在程序中
    	UserService userService = new UserServiceImpl();
```

##### 4.2、简单工厂的设计

```java
package basic;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class BeanFactory {
    private static Properties env = new Properties();

    static {
        try {
            InputStream inputStream = BeanFactory.class.getResourceAsStream("/applicationContext.properties");
            env.load(inputStream);
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }



    /*
    对象的创建方式
    1、直接调用构造方法： 创建对象 UserService userService = new ServiceImpl();
    2、通过反射的形式  创建对象 ，解耦合
        Class clazz = Class.forName("basic.UserServiceImpl");
        UserService userService = (UserService)clazz.newInstance();
     */
    public static UserService getUserService(){
        /*
        1、new 方法
         */
//        return new UserServiceImpl();


        /*
        2、反射方法
         */
        UserService userService = null;
        try {
            Class clazz = Class.forName(env.getProperty("userService"));
            userService = (UserService) clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

        return userService;

    }


    public static UserDAO getUserDAO(){
        UserDAO userDAO = null;
        try {
            Class clazz = Class.forName(env.getProperty("userDAO"));
            userDAO = (UserDAO) clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

        return userDAO;
    }
}
```

##### 4.3、通用工厂设计

```java
//简单工厂代码大量冗余
public class BeanFactory1 {
    private static Properties env = new Properties();

    static {
        try {
            InputStream inputStream = BeanFactory.class.getResourceAsStream("/applicationContext.properties");
            env.load(inputStream);
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static Object getBean(String key){
        Object ret = null;

        try {
            Class clazz = Class.forName(env.getProperty(key));

            ret = clazz.newInstance();

        }catch (Exception e){
            e.printStackTrace();
        }
        return ret;
    }
}
```

##### 4.4、通用工厂的使用方式

```markdown
1. 定义类型(类)
2. 通过配置文件的配置告知工厂(applicationContext.properties)
	key = value
3. 通过工厂获得类的对象
	object ret = BeanFactory.getBean("key");
```

```markdown
1. 定义类型
```

```java
public class Person {
}
```

```markdown
2. 配置文件
```

```properties
person = basic.Person
```

```markdown
3. 获得对象
```

```java
Person person = (Person)BeanFactory1.getBean("person");
System.out.println("person = " + person);
```

#### 5、总结

```markdown
spring本质：工厂 ApplicationContext.properties(applicationContext.xml)
```



## 第二章 第一个spring程序

#### 1. 软件版本

```markdown
1. JDK1.8+
2. Maven3.5+
3. IDEA2018+
4. SpringFramework 5.1.4
	官方网站 WWW.spring.io
```



#### 2.环境搭建

- Spring的jar包

  ```markdown
  # 设置pom 依赖
  <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.4.RELEASE</version>
  </dependency>
  ```

- Spring的配置文件

```markdown
1. 配置文件的防止位置：任意位置 没有硬性要求
2. 配置文件的命名   ：没有硬性要求，建议：applicationContext.xml

思考：日后应用Spring框架时，需要进行配置文件路劲的设置
```

![ApplcationContext创建](images\ApplcationContext创建.png)



#### 3. Spring的核心API

- ApplicationContext

```markdown
1. 作用：Spring提供的ApplicationContext这个工厂，用于对象的创建
2. 好处：解耦合
```

- ApplicationContext接口类型

  ```markdown
  1. 接口：屏蔽在不同场景下使用实现的差异
  2. 非Web环境： ClassPathXmlApplicationContext(main junit)
  3. Web环境  ： XmlApplicationContext
  ```

  ![SpringApi](images\SpringApi.png)

- 重量级资源

  ```markdown
  1. ApplicationContext工厂的对象占用大量内存
  2. 不会频繁的创建对象：一个应用指挥创建一个工厂对象
  3. ApplicationContext工厂：一定是线程安全的。(多线程并发访问)
  ```

  

#### 4. 程序开发

```markdown
1. 创建类型
2. 配置文件 applicationContext.xml
3. 通过工厂类，获得对象
	ApplicationContext
	
		|- ClassPathXmlApplicationContext
	ApplicationContext application = new ClassPathXmlApplicationContext("applicationContext.xml");
	Person person = (Person)application.getBean("person");
```





#### 5.细节分析

- 名词解释

  ```markdown
  Spring工厂创建的对象，叫做bean或者组件(componet)
  ```

- Spring工厂相关的方法

  ```java
  @Test
  public void test4() {
      ApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext.xml");
      //避免了强制类型转换
      Person person = ctx.getBean("person", Person.class);
      System.out.println("person = " + person);
      
  
      //当前Spring的配置文件中 只能有一个<bean class是Person类型
      Person person = ctx.getBean(Person.class);
      System.out.println("person = " + person);
      
  
      //获取的是 Spring工厂配置文件中所有bean标签的id值  person person1
      String[] beanDefinitionNames = ctx.getBeanDefinitionNames();
      for (String beanDefinitionName : beanDefinitionNames) {
          System.out.println("beanDefinitionName = " + beanDefinitionName);
      }
  
      //根据类型获得Spring配置文件中对应的id值
      String[] beanNamesForType = ctx.getBeanNamesForType(Person.class);
      for (String id : beanNamesForType) {
          System.out.println("id = " + id);
      }
  
      //用于判断是否存在指定id值得bean,不能判断name值
      if (ctx.containsBeanDefinition("person")) {
          System.out.println("true = " + true);
      }else{
          System.out.println("false = " + false);
      }
  
  
      //用于判断是否存在指定id值得bean,也可以判断name值
      if (ctx.containsBean("p")) {
          System.out.println("true = " + true);
      }else{
          System.out.println("false = " + false);
      }
  }
  ```

- 配置文件中需要注意的细节

  ```markdown
  1. 只配置class属性
  <bean class="basic.Person"/>
  a) 上述这种配置 有id值  id = basic.Person#0
  b) 应用场景 ：如果这个只需要使用一次，那么可以省略id值
  			如果这个只需要使用多次，或者被其他bean引用则需要设置id值
  			
  			
  			
  2. name 属性
  作用：用于在Spring的配置文件中，为bean对象定义别名
  相同：
  	1. application.getBean("id|name") --> object
  	2. <bean id="" class=""
  	    等效
  	    <bean name="" class=""
  区别：
  	1.别名可以定义多个，但id属性只有一个值
  	2.Xml的id属性的值。命名要求：必须以字母开头。数字，下划线，连字符 不能以特殊字符开头
  		name 属性的值，命名没有要求
  		
          但是XML发展到了今天：ID属性的限制，不存在 /person
          
      3. 代码
  	//不能判断name值
      if (ctx.containsBeanDefinition("person")) {
          System.out.println("true = " + true);
      }else{
          System.out.println("false = " + false);
      }
  
      //也可以判断name值
      if (ctx.containsBean("p")) {
          System.out.println("true = " + true);
      }else{
          System.out.println("false = " + false);
      }
  ```

  

#### 6.Spring工厂的底层实现原理(简易版)

![Spring原理](images\Spring原理.png)



#### 7.思考

```markdown
1. 问题：未来在开发中，是不是所有对象，都会交给Spring工厂来创建？
2. 回答：理论上 是的，但是有特例：实体对象(entity)
```



## 第三章、Spring5.x与日志框架的整合

```markdown
Spring与日志框架进行整合，日志框架就可以在控制台中，输出Spring框架运行进行中的一些重要的信息
好处：便于了解Spring框架的运行过程，利于程序调试
```

- Spring如果整合日志框架

  ```markdown
  默认：
  	Spring1.2.3早期都是于commons-logging.jar
  	Spring5.x默认整合的日志框架 logback log4j2
  	
  Spring5.x整合log4j
  	1.引入log4j jar包
  	2.引入log4.properties配置文件
  ```

- pom.xml

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.25</version>
    <scope>test</scope>
</dependency>
```

- log4j.properties

```properties
# resources # resources⽂件夹根⽬录下
### ### 配置根
log4j.rootLogger = debug,console
### ### ⽇志输出到控制台显示
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Target=System.out
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-ddHH:mm:ss} %-5p %c{1}:%L - %m%n
```



## 第四章、注入(Injection)

#### 4.1 什么是注入

```markdown
通过Spring工厂及配置文件，为所创建的成员变量赋值
```

##### 1.1 为什么要注入

```markdown
通过编码的方式，为成员变量进行赋值，存在耦合
```

```java
@Test
public void test5(){
    ApplicationContext context = new ClassPathXmlApplicationContext("/applicationContext.xml");

    Person person = (Person)context.getBean("person");

    person.setId(1);
    person.setName("jj");

    System.out.println(person);
}
```

##### 1.2 如何进行注入

- 类为成员变量提供set get方法

- 配置Spring的配置文件

```xml
<bean id="person" class="basic.Person">
    <property name="id" >
        <value>1</value>
    </property>
    <property name="name">
        <value>jj</value>
    </property>
</bean>
```

##### 1.3 注入好处

```markdown
解耦合
```



#### 2. Spring注入的原理分析(简易版)

![注入原理](images\注入原理.png)





## 第五章、Set注入详解

~~~markdown
针对于不同类型的成员变量，在<property>标签，需要嵌套其他标签。
<property>
	xxxx
</property>
~~~

![set注入](images\set注入.png)

#### 1. JDK类型

##### 1.1、String + 8种基本类型

```xml
<value>jj</value>
```

##### 1.2、数组

```xml
<property name="emails">
    <list>
        <value>asdad@jjj.com.cn</value>
        <value>qwrqr@jjj.com.cn</value>
    </list>
</property>
```

##### 1.3、set集合

```xml
<property name="tels">
    <set>
        <value>1111111</value>
        <value>2222222</value>
        <value>3333333</value>
        <!-- 重复元素会过滤掉   -->
        <value>1111111</value>
    </set>
</property>


注意：
在<set></set>中我们不一定是用<value></value>来赋值
如果set的泛型是Object
我们可以用<bean></bean>添加自定义的属性。也可以添加set属性的元素，<set></set>
<set>
	<ref></ref>
    <set>
    	<value></value>
    </set>
</set>
```

##### 1.4、List集合

```xml
<property name="addresses">
    <list>
        <value>apfa</value>
        <value>asfaf</value>
        <value>as</value>
        <value>as</value>
    </list>
</property>
```

##### 1.5、Map集合

```xml
<property name="pps">
    <map>
        <entry>
            <key><value>jj</value></key>
            <value>10</value>
        </entry>

        <entry>
            <key><value>jk</value></key>
            <value>11</value>
        </entry>
    </map>
</property>
```

##### 1.6、Properties

```xml
<property name="p">
    <props>
        <!--因为properties是 key = value形式-->
        <!--因为properties只能是字符串类型，所以不用加<value></value>-->
        <prop key="key1">value1</prop>
        <prop key="key2">value2</prop>
    </props>
</property>
```

##### 1.7、复杂的JDK类型(Date)

```markdown
需要程序员自定义类型转换器，处理
```



#### 2. 用户自定义类型

##### 2.1、第一种方式

- 为成员变量提供set get方法
- 配置文件中进行注入(赋值)

```xml
<bean id="userService" class="basic.UserServiceImpl">
    <property name="userDAO">
         <bean class="basic.UserDAOImpl"/>
    </property>
</bean>
```

##### 2.2、第二种方式

- 第一种赋值方式存在的问题

  ```markdown
  1. 配置文件代码冗余
  2. 被注入对象(UserDAO),多次创建，浪费(JVM)内存资源
  ```

- 为成员变量提供set get方法

- 配置文件中进行配置

![自定义类型注入分析](images\自定义类型注入分析.png)

```xml
<bean id="userDAO" class="basic.UserDAOImpl"/>

<bean id="userService" class="basic.UserServiceImpl">
    <property name="userDAO">
         <ref bean="userDAO"/>
    </property>
</bean>

# Spring 4.x 废除了<ref local="" /> 基本等效<ref bean=""/>
```



#### 3、Set注入的简化写法

##### 3.1、基于属性简化

```xml
<!--JDK基本属性-->
<property name="">
	<value></value>
</property>

<property name="" value=""/>
注意:value属性 只能简化 8 种基本数据类型 + String  注入标签

用户自定义类型
<property name="">
	<ref bean=""/>
</property>

<property name="" ref="" />
```

##### 3.2、基于p命名空间

```markdown
<!--JDK基本属性-->
<property name="">
	<value></value>
</property>

<bean id="person" class="basic.Person" p:id="1" p:name="jj"/>
注意:value属性 只能简化 8 种基本数据类型 + String  注入标签

用户自定义类型
<property name="">
	<ref bean=""/>
</property>

<bean id="userService" class="basic.UserServiceImpl" p:userDAO-ref="userDAO"/>
```



## 第六章、构造注入

```markdown
1. 注入:通过Spring的配置文件，为成员变量赋值
2. Set注入:Spring调用Set方法 通过配置文件 为成员变量赋值
3. 构造注入:Spring调用构造方法 通过配置文件 为成员变量赋值
```

##### 1.开发步骤

- 提供有参构造方法

```java
private String name;
private Integer age;

public Customer(String name, Integer age) {
    this.name = name;
    this.age = age;
}
```

- Spring配置文件

```xml
<bean id="customer" class="basic.constructer.Customer">
    <!--两个参数-->
    <constructor-arg>
        <value>jj</value>
    </constructor-arg>
    <constructor-arg>
        <value>11</value>
    </constructor-arg>
</bean>
```

##### 2、构造方法重载

###### 2.1 参数个数不同时

~~~markdown
通过控制<constructer-arg>标签的数量进行区分
~~~

```xml
<!--一个参数-->
<bean id="customer" class="basic.constructer.Customer">
    <constructor-arg>
        <value>jj</value>
    </constructor-arg>
</bean>
```

###### 2.2、构造参数个数相同时

~~~markdown
通过在标签引入type属性 进行类型的区分 <consructor-arg type=""/>
~~~

```xml
<bean id="customer" class="basic.constructer.Customer">
    <constructor-arg type="int">
        <value>11</value>
    </constructor-arg>
</bean>
```

##### 3、注入总结

~~~markdown
未来的实战中，应用set注入还是构造输入?
答案：	set注入更多
	1. 构造输入麻烦(重载)
	2. Spring框架底层 大量应用了ser注入
~~~

![注入总结](images\注入总结.png)



## 第七章、反转控制与依赖注入

#### 1. 反转控制(IOC Inverse of Control)

```markdown
控制： 对于成员变量赋值的控制权
反转控制：把对于成员变量赋值的控制权，从代码中反转到Spring工厂和配置文件中完成。
	好处：解耦合
底层实现： 工厂设计模式
```

![反转控制](images\反转控制.png)

#### 2. 依赖注入（Dependency Injection DI)

```markdown
注入：通过Spring 的工厂及配置文件，为对象(Bean 组件)的成员变量赋值。
依赖注入：当一个类需要另一个类时，就意味着依赖，一旦出现依赖，就可以把另一个类作为本类的成员变量，最终通过Spring配置文件进行注入(赋值)。

好处：解耦合
```

![DI](images\DI.png)

## 第八章、Spring工厂创建复杂对象

![复杂对象](images\复杂对象.png)

#### 1、什么是复杂对象

```markdown
复杂对象： 指的就是不能直接通过new构造方法创建的对象
	Connection
	SqlSessionFactory
```

## 2、Spring工厂创建复杂对象的3种方式

##### 2.1、FactoryBean接口

- 开发步骤

  - 实现FactoryBean接口

  ![FactoryBean接口](images\FactoryBean接口.png)

  ```java
  public class ConnectionFactoryBean implements FactoryBean<Connection> {
      //用与书写创建复杂对象的代码
      @Override
      public Connection getObject() throws Exception {
          Class.forName("com.mysql.jdbc.Driver");
          Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis","root","123456");
  
          return connection;
      }
  
      @Override
      public Class<Connection> getObjectType() {
          return Connection.class;
      }
  
      @Override
      public boolean isSingleton() {
          return false;
      }
  }
  ```

  - Spring配置文件的配置

  ```markdown
  # 如果Class中指定的类型是FactoryBean接口的实现类，
  # 那么通过id值获得的是这个类所创建的复杂对象
  
  Connection
  
  <bean id="conn" class="xxx.xxx.factorybean.ConnectionFactoryBean"/>
  
  ```



- 细节

  - 如果就想获得FactoryBean类型的对象

    ```markdown
    通过实现FactoryBean接口可以得到复杂对象
    当我们想要获得 ConnectionFactoryBean 对象
    context.getBean("&conn")
    这样就可以获得ConnectionFactoryBean对象
    ```

  - isSingleton

    ```markdown
    返回true只会创建一个复杂对象
    返回false每一次都会创建新的对象
    
    
    问题： 根据这个对象的特点，决定是返回true(SqlSessionFactory)还是false(Connection)
    ```

  - mysql⾼版本连接创建时，需要制定SSL证书，解决问题的⽅式

    ```markdown
    url ="jdbc:mysql://localhost:3306/mybatis?useSSL=false"
    ```

  - 依赖注入的体会(DI)

    ```markdown
    把ConnectionFactoryBean中依赖的4个字符串信息，并进行配置文件的注入
    好处： 解耦合
    ```

    ```java
    public class ConnectionFactoryBean implements FactoryBean<Connection> {
        private String driverClassName;
        private String url;
        private String user;
        private String password;
    }
    ```

    ```xml
    <bean id="conn" class="basic.factoryBean.ConnectionFactoryBean">
    
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false"/>
        <property name="user" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    ```

    

- FactoryBean的实现原理(简易版)

```markdown
接口回调
1. 为什么Spring规定FactoryBean接口 实现 并且 getObject()？
2. context.get("conn") 获得复杂对象 Connection  而没有 获得ConnectionFactoryBean

Spring内部运行流程
1. 通过conn获得ConnectionFactoryBean类的对象，进而通过instanceof判断出FactoryBean接口的实现类
2. Spring按照规定getObject() --->  Connection
3. 返回Connection
```

![FactoryBean原理](images\FactoryBean原理.png)



- FactoryBean总结

```markdown
Spring中⽤于创建复杂对象的⼀种⽅式，也是Spring原⽣提供的，后续讲解Spring整合其他框架，⼤量应⽤FactoryBean
```

##### 2.2、实例工厂

```markdown
使用实例工厂的原因
1. 避免Spring框架的侵入
2. 整合遗留系统
```

```java
public class ConnectionFactory {
    public Connection getConnection(){

        Connection conn = null;

        try {
            Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?useSSL=false","root","123456");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;
    }

}
```

```xml
<bean id="connFactory" class="basic.factoryBean.ConnectionFactory"/>

<bean id="conn" factory-bean="connFactory" factory-method="getConnection"/>
```

##### 2.3、静态工厂

```java
public class StaticConnectionFactory {
    public static Connection getConnection(){
        Connection conn = null;

        try {
            Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?useSSL=false","root","123456");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;

    }
}
```

```xml
<bean id="conn" class="basic.factoryBean.StaticConnectionFactory" factory-method="getConnection"/>
```

```markdown
实例工厂和静态工厂的区别：

实例工厂需要先创建ConnectionFactory对象 ，然后调用getConnection()方法

静态工厂因为是静态方法，不需要创建对象，直接调用静态方法
```

#### 3、Spring工厂创建对象的总结

![创建复杂对象总结](images\创建复杂对象总结.png)

## 第九章、控制Spring工厂创建对象的次数

#### 1、如何控制简单对象的创建次数

```xml
<bean id="account" scope="singleton|prototype" class="xxxx.Account"/>
sigleton:只会创建⼀次简单对象 默认值
prototype:每⼀次都会创建新的对象
```

#### 2、如何控制复杂对象的创建次数

```markdown
FactoryBean{
 	isSingleton(){
 		return true 只会创建⼀次
 		return false 每⼀次都会创建新的
 	}
}
如没有isSingleton⽅法 还是通过scope属性 进⾏对象创建次数的控制
```

#### 3、为什么要控制对象的创建次数

```markdown
好处： 节省不必要的内存浪费
```

- 什么样的对象只创建一次?

  ```markdown
  1. SqlsessionFactory
  2. DAO
  3. Service
  ```

- 什么样的对象 每一次都要创建新的?

  ```markdown
  1. Connection
  2. SqlSession|Session
  3. Strues2 Action
  ```

  













