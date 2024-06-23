## Spring
### 基本格式
``` xml
<bean id="userDao" class="com.itheima.ioc.UserDaoImp"></bean>
```
``` java
package com.itheima.ioc;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class TestIOC {
    public static void main(String[] args) {
        // 1. 初始化spring容器，加载配置文件
        ApplicationContext applicationContext = new
        ClassPathXmlApplicationContext("ApplicationContext.xml");
        // 2. 通过容器获取userDao实例
        UserDao userDao = (UserDao)applicationContext.getBean("userDao");
        // 3. 调用实例方法
        userDao.say();
    }
}
```
### 依赖注入DI
#### 设值注入
> 需要set方法
``` xml
<property name="username" value="张三"></property>
```
#### 构造注入
> 使用构造函数设置属性值
``` xml
<constructor-arg index="0" value="tom" /> 
```
#### 注意事项
* 引用类型用ref,值类型用value
### 注解Annotation装配
``` java
package com.itheima.annotation;
import javax.annotation.Resource;
import org.springframework.stereotype.Controller;
@Controller("userController")
public class UserController {
    @Resource(name="userService")
    private UserService userService;
    public void save(){
        this.userService.save();
        System.out.println("save");
    }
    public void setUserService(UserService userService) {
    this.userService = userService;
    }
}
```
### 工厂方式实例化
#### 静态工厂
``` xml
<bean id="bean" class="BeanFactory" factory-method="static_method" />
```
#### 实例工厂
``` xml
<bean id="bean" factory-bean="beanFactory" factory-method="method" />
```
### 单例/原型
``` xml
    <bean id="bean" class="MYBean" scope="singleton/prototype"/> 
```
### 简答题
####  什么是IOC(控制反转)
它把传统上由程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓的“控制反转”概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器
#### 什么是DI(依赖注入)
组件之间的依赖关系由容器在应用系统运行期来决定，也就是由容器动态地将某种依赖关系的目标对象实例注入到应用系统中的各个关联的组件之中。组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系
#### 常见的依赖注入方式
构造器注入和Setter方法注入

## Spring AOP 面向切向编程
### 两种框架
* Spring AOP(狗屎一样)
* AspectJ
### AspectJ注解
| | | |
|-|-|-|
|@Aspect|用于定义一个切面|后面要跟@Component，因为在Spring中这个类算是一个组件
|@Pointcut|用于定义切入点表达式|需要定义一个签名(不用管)
|@Before|用于定义前置通知|需要指定切入点表达式，然后这个函数就能在切入点方法之前执行
|@AfterReturning|用于定义返回通知|和上面的一样，就是之后执行
|@Around|用于定义环绕通知|之前和之后都会调用
|@AfterThrowing|用于定义异常通知|抛出异常了会执行
|@After|用于定义返回通知|无论正常执行或者抛出异常了都会执行
## MyBatis 程序设计
### 简述MyBatis功能
提供数据库操作简化、SQL语句的分离、灵活的映射方式、缓存机制、多种数据库支持、与其他框架的集成、动态SQL的嵌套查询等功能
### MyBatis提供的关系映射包含哪几种类型
一对一、一对多、多对多
### MyBatis 如何采用缓存机制进行性能优化
如果同一个SqlSession对象多次执行完全相同的SQL语句，在第一次执行完成后，MyBatis会将查询结果写入到一次缓存中，此后，如果程序没有执行插入、更新、删除操作，当第二个执行相同的查询语句时，MyBatis会直接读取一级缓存中的数据，而不用再去数据库从而提高数据库的查询效率
### 映射文件
``` xml
<!-- mapper是配置文件的根元素 -->
<!-- namespace命名空间 -->
<mapper namespace="com.itheima.mapper.CustomerMapper">
    <!-- parameterType传递的参数类型 -->
    <!-- resultType返回的类型 -->
    <update id="updateCustomer" parameterType="com.itheima.po.Customer">
        update t_customer set
        username=#{username},jobs=#{jobs},phone=#{phone}
     where id=#{id}
    </update> 
 </mapper>
 ```
### 程序
``` java
public class MybatisTest {
    /**
    * 根据客户编号查询客户信息
    */
    @Test // 不是主函数就要加上这个注解
    public void findCustomerByIdTest() throws Exception {
        // 1、读取配置文件
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource); // 通过输入流读取配置文件
        // 2、根据配置文件构建SqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        // 3、通过SqlSessionFactory创建SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 4、SqlSession执行映射文件中定义的SQL，并返回映射结果
        Customer customer = sqlSession.selectOne("com.itheima.mapper" + ".CustomerMapper.findCustomerById", 1);
        // 打印输出结果
        System.out.println(customer.toString());
        // 5、关闭SqlSession
        sqlSession.close();
    }
}
```
## SpingMVC 程序设计
### web.xml
``` xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-config.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
 </servlet>
 <servlet-mapping>
 <servlet-name>DispatcherServlet</servlet-name>
 <url-pattern>/</url-pattern>
 </servlet-mapping>
```
### POJO 类型的数据绑定方法
就是一个变量有get和set方法
### 在SSM 框架整合方案中，Java EE应用通常分为哪几个层次
表现层、业务逻辑层、持久化层 
### Spring MVC、Spring和MyBatis 三个框架的分工是怎样的？
Spring MVC:负责处理用户请求，将请求路由至相应的控制器（Controller），控制器处理请求并返回ModelAndView对象，最终将数据渲染到视图中
Spring :负责管理JavaEE中各组件，提供IOC和DI
MyBatis:负责连接数据库