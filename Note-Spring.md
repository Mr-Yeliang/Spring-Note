### 1. Bean概述

Spring容器负责创建、管理所有的Java对象，这些Java对象被称为bean，一个bean包含以下元数据：

- 包限定的类名：被定义bean的实际实现类。
- bean行为的配置元素，在容器中bean以哪种状态表现（scope，lifecycle，callback，等等）。
- 需要一起工作的其它对象的引用，这些引用又被称作合作者（collaborator）或依赖。
- 设置到新创建对象中的其它配置，例如，在bean中使用的连接数，用于管理连接池或池的大小。

元数据转化成了一系列的属性，组成了每个bean的定义，这些bean定义表示为**BeanDefinition**对象  。 



#### 1.1 命名Bean

在Spring容器中一个bean通常只有一个标识符，也可以有多个，但必须都是唯一的。

##### bean命名规则

采用标准Java对字段的命名规则，即以小写字母开头，然后是驼峰式。例如：accountManage,  userDao, loginController等等。 

##### 在bean定义外给bean起别名

在XML配置中，可以使用<alias/>元素指定别名。

```
<alias name="fromName" alias="toName"/>
```

如上，同一个容器中这个bean叫**fromName**，在使用了别名定义也可以叫**toName**。

一个bean可以有多个别名，例如

```
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource" alias="myApp-dataSource" />
```

在一个应用中每个组件和主应用都可以通过使用**subsystemA-datasource**、**subsystemB-datasource**、**myApp-datasource**这三个名字引用同一个bean。



#### 1.2 实例化bean

##### 使用构造方法实例化

最常见的情况，在Spring底层调用Bean类的无参数构造器来创建实例。 

在XML中，可以像下面这样指定bean类：

```
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>123
```



##### 使用静态工厂方法实例化

使用静态工厂方法创建一个bean时，需要使用**class**属性指定包含静态工厂方法的类，并使用**factory-method**属性指定工厂方法的名字，调用这个方法后返回一个有效的对象，之后跟用构造方法创建的对象一样。

```
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>123
```

```
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```



##### 使用实例的工厂方法实例化（非静态）

此方法是调用一个已存在的bean的非静态方法来创建一个新的bean。使用这种机制需要把class属性置空，在**factory-bean**属性中指定被调用的用来创建对象的包含工厂方法的那个bean的名字，并**factory-method**属性中设置工厂方法的名字。

```
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>123456789
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();
    private DefaultServiceLocator() {}

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```



### 2.依赖

#### 2.1 依赖注入

**IoC** （Inversion of Control)，控制反转spring来负责控制对象的生命周期和对象间的关系。

**依赖注入**是一个对象定义其依赖的过程。

依赖注入有两种主要的方式，基于构造方法的依赖注入和基于setter方法的依赖注入。

##### 基于构造方法的依赖注入

基于构造方法的依赖注入，由容器调用带有参数的构造方法来完成，每个参数代表一个依赖。

```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...

}
```

##### 基于setter方法的依赖注入

基于setter方法的依赖注入，由容器在调用无参构造方法或无参静态工厂方法之后调用setter方法来实例化bean。

下面的例子展示了一个只能通过纯净的setter方法注入依赖的类。这个类符合Java的约定，它是一个没有依赖容器的特定接口、基类或注解的POJO。

```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...

}
```

##### 依赖注入的过程

容器按如下方式处理依赖注入：

- **ApplicationContext**被创建并初始化描述所有bean的配置元数据。配置元数据可以是XML、Java代码或注解。
- 对于每一个bean，它的依赖以属性、构造方法参数或静态工厂方法参数的形式表示。这些依赖在bean实际创建时被提供给它。
- 每一个属性或构造方法参数都是将被设置的值的实际定义，或容器中另一个bean的引用。
- 每一个值类型的属性或构造方法参数都会从特定的形式转化为它的实际类型。默认地，Spring可以把字符串形式的值转化为所有的内置类型，比如**int**, **long**, **String**, **boolean**，等等。

##### 依赖注入的示例

下面的例子使用XML配置setter方法注入。XML的部分配置如下：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```
public class ExampleBean {

    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }

}
```



#### 7.4.2 依赖与配置详解

如前所述，可以定义bean的属性和构造方法的参数引用其它的bean（合作者），或设置值。在XML配置中，可以使用<property/>或<constructor-arg/>的属性达到这个目的。

##### 直接设置值（原始类型，String，等等）

<property/>的value属性可以为属性或构造方法参数指定人类可读的字符串形式的值。Spring的转换器service会把这些值转换成属性或参数的实际类型。

```
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

##### 引用其它bean（合作者）

ref让一个bean的指定属性引用另一个bean（合作者）。被引用的bean就是这个bean的一个依赖，并且会在此属性设置前被初始化。

```
<ref bean="someBean"/>
```

通过parent属性可以指定对当前容器的父容器中的bean的引用。parent属性的值可能是目标bean的id属性或name属性中的一个，而且目标bean必须在当前容器的父容器中。这种引用形式主要出现在有容器继承并且想把父容器中存在的bean包装成同样名字的代理用于子容器的时候。

```
<!-- in the parent context -->
<bean id="accountService" class="com.foo.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

##### 内部bean

在<property/>或<constructor-arg/>元素内部定义的bean就是所谓的内部bean。

```
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean不需要定义id或name，即使指定了，容器也不会使用它作为标识符。

##### 集合

在<list/>, <set/>, <map/>和<props/>元素中，可以分别设置Java集合类型List, Set, Map和Properties的属性和参数。

```
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map的键值或set的值也可以是以下任何元素：

```
bean | ref | idref | list | set | map | props | value | null1
```



##### null和空字符串值

Spring把对属性的空参数作为空字符串。下面的XML片段把email属性的值设置成了空字符串值（”“）。

```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

上面的例子与下面的Java代码是等价的：

```
exampleBean.setEmail("")
```

<null/>元素处理null值。例如：

```
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

上面的配置与下面的Java代码等价：

```
exampleBean.setEmail(null)
```



##### 合成属性名

可以使用合成或嵌套的属性名设置bean的属性，只要路径中除了最后的属性值的所有的组件都不为null，考虑使用如下bean定义：

```
<bean id="foo" class="foo.Bar">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

foo有一个属性叫fred，fred有一个属性叫bob，bob有一个属性叫sammy，并且最后的sammy属性的值被设置为123。其中，foo的fred属性和fred的bob属性在bean构造后必须不为null，否则将会抛出空指针异常。

#### 7.4.3 使用depends-on

如果一个bean是另一个bean的依赖，那通常意味着这个bean被设置为另一个bean的属性。depends-on属性可以明确地强制一个或多个bean在使用它（们）的bean初始化之前被初始化。

下面的例子使用depends-on属性表示对一个单例bean的依赖：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表示对多个bean的依赖，可以为depends-on属性的值提供多个名字，使用逗号，空格或分号分割：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

depends-on属性能够同时指定初始化时依赖和通信销毁时依赖，这只能运用在单例bean的情况中。依赖的bean在给定的bean销毁之前被销毁。因此，depends-on也能够控制关闭的顺序。（初始化的时候依赖的对象先初始化才能注入，销毁时需要依赖的对象先销毁才能解绑）

#### 7.4.4 自动装配合作者

Spring容器可以相互合作的bean间自动装配其关系。可让Spring通过检查ApplicationContext的内容自动解决bean之间的依赖。自动装配有以下优点：

- 自动装配将极大地减少指定属性或构造方法参数的需要（在这点上，其它机制比如本章其它小节讲解的[bean模板](https://blog.csdn.net/tangtong1/article/details/51960382#beans-child-bean-definitions)也是有价值的）。
- 自动装配可以更新配置当你的对象进化时。例如，如果你需要为一个类添加一个依赖，那么不需要修改配置就可以自动满足。因此，自动装配在开发期间非常有用，但不否定在代码库变得更稳定的时候切换到显式的装配。

使用XML配置时，可以为带有autowire属性的bean定义指定自动装配的模式。自动装配的功能有四种模式。可以为每个自动装配的bean指定一种模式。 
**表7.2 自动装配的模式**

| 模式        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | 默认地没有自动自动装配。bean的引用必须通过ref元素定义。对于大型部署，不推荐更改默认设置，因为显式地指定合作者能够更好地控制且更清晰。在一定程度上，这也记录了系统的结构。 |
| byName      | 按属性名称自动装配。Spring为待装配的属性寻找同名的bean。例如，如果一个bean被设置为按属性名称自动装配，且它包含一个属性叫master（亦即，它有setMaster(…)方法），Spring会找到一个名叫master的bean并把它设置到这个属性中。 |
| byType      | 按属性的类型自动装配，如果这个属性的类型在容器中只存在一个bean。如果多于一个，则会抛出异常，这意味着不能为那个bean使用按类型装配。如果一个都没有，则什么事都不会发生，这个属性不会被装配。 |
| constructor | 与按类型装配类似，只不过用于构造方法的参数。如果这个构造方法的参数类型在容器中不存在明确的一个bean，将会抛出异常。 |



##### 自动装配的局限性和缺点

如果一个项目一直使用自动装配，它会运行得很好。如果只是用在一两个bean上，则会出现混乱。

自动装配有如下局限性和缺点：

- 在**property**和**constructor-arg**上显式设置的依赖总是覆盖自动装配。而且，不能自动装配所谓的简单属性，如基本类型、**String**和**Classes**（也包括简单属性的数组）。
- 自动装配没有显式装配精确。
- 从Spring容器生成文档的工具可能找不到装配信息。
- 容器中的多个bean定义可能会匹配setter方法或构造方法参数指定的类型。

在上述场景中，有如下几种选择：

- 放弃自动装配，改为显式地装配。
- 设置bean的**autowire-candidate**属性为**false**以避免自动装配。
- 设置**<bean/>**标签的**primary**属性为**true**，从而为其指定一个单独的bean作为主要的候选者。
- 使用基于注解的配置实现更细粒度的控制。

##### 避免bean自动装配

在单个bean上，可以避免自动装配。在xml形式中，设置**<bean>**元素的**autowire-candidate**属性为false，容器就不会让这个bean进入自动装配的架构中（包括注解形式的配置，比如**@Autowired**）。

也可以通过定义bean名称的匹配模式避免自动装配。顶级元素**<beans>**的属性**default-autowire-candidates**可以接受一个或多个模式。

#### 7.4.6 方法注入

在大部分应用场景中，容器中的大多数bean都是单例的。当一个单例bean需要与另一个单例bean合作，或者一个非单例bean需要与另外一个非单例bean合作的时候，仅仅定义一个bean作为另一个的属性就能解决它们的依赖关系。如果bean的生命周期不一致就会出现问题。假如，一个单例bean A 需要使用一个非单例bean B，也许在每次调用A的方法时都需要。容器仅仅创建这个单例bean A 一次，所以只有一次机会设置其属性。容器不能在 A 每次调用的时候都提供一个新的 B 的实例。

一种解决方法是放弃控制反转。可以通过实现**ApplicationContextAware**接口使 A 能连接到容器，并在每次 A 需要 B 的实例的时候调用容器的**getBean(“B”)**方法取得新的 B 的实例。下面是这种方式的一种案例：

```
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

##### 查找方法注入

查找方法注入是容器的一项能力，它可以重写容器管理的bean的方法，并返回另一个bean的查找结果。查找往往涉及到前面描述的那种原型（prototype）bean。spring利用CGLIB库动态地生成字节码子类，从而重写方法以实现查找方法注入。

- 为了能使动态的子类有效，被继承的类不能是**final**，且被重写的方法也不能是**final**。
- 单元测试一个具有抽象方法的类时，需要手动继承此类并重写其抽象方法。
- 组件扫描的具体方法也需要具体类。
- 一项关键限制是查找方法不能使用工厂方法和配置类中的**@Bean**方法，因为容器不会在运行时创建一个子类及其实例。
- 最后，方法注入的目标对象不能被序列化。

查看前面关于**CommandManager**类的代码片段，可以发现spring容器会动态生成**createCommand()**方法的实现。**CommandManager**不会有任何的spring依赖，如下所示：

```
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

客户端类包含了将被注入的方法（本例中的**CommandManager**），被注入的方法需要如下签名：

```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是抽象的，动态生成的子类会实现这个方法。另外，动态生成的子类也会重写原类中的具体方法。例如：

```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="command" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="command"/>
</bean>
```

*commandManager*这个bean会在任何它需要*command*实例的时候调用其**createCommand()**方法。如果实际需要，则必须把**command**声明为原型（prototype）。如果被声明为单例（singleton），则每次返回的都是同一个**command**实例。

##### 任意的方法替换

方法注入的一种很少使用的形式，它是使用另外的方法实现替换目标bean中的任意方法。

以xml形式为例，可以使用**replaced-method**元素指定用另一种实现替换bean中已存在的方法。如下例，我们希望重写其*computeValue*方法：

```
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...

}
```

一个实现了**org.springframework.beans.factory.support.MethodReplacer**接口的类提供了一个新的方法定义。

```
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

原类的bean定义可能看起来像这样：

```
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

可以使用一个或多个**<arg-type/>**元素指明原方法的参数签名。参数的签名仅仅当原方法有多个重载方法时才是必要的。为了方便，string类型的参数也可以是其全路径的子串，例如，以下方式均可匹配**java.lang.String**：

```
java.lang.String
String
Str
```

因为参数的个数往往就能够区分每一种可能了，这种简写方法可以少打几个字，所以你可以只输入最短的字符就能够匹配参数类型。



### 7.9 基于注解的容器配置

一种XML形式的替代方案是使用基于注解的配置，它依赖于字节码元数据，用于装配组件并可取代尖括号式的声明。

- 注解形式的注入在XML形式之前执行，因此，如果同时使用了两者，则XML形式的注入会覆盖注解形式的注入。

可以一个一个地注册这些bean，也可以隐式地注册它们，使用下面的配置即可（注意，请包含**context**命名空间）。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

（隐式注册的后置处理器包括**AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor, PersistenceAnnotationBeanPostProcessor**以及前面提到的**RequiredAnnotationBeanPostProcessor**。）

- **context:annotation-config/**只能寻找它所定义的上下文中的注解。这意味着，如果在一个**WebApplicationContext**中为一个**DispatcherServlete**配置了**context:annotation-config/**，那么它只会检测controller中的**@Autowired**，而不会检测service。

#### 7.9.1 @Required

- JSR-330的**@Inject**注解可以替换下面例子中Spring的**@Autowired**注解。

可以在构造方法上使用**@Autowired**：

```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...

}
```

- 从Spring 4.3开始，如果目标bean只有一个构造方法，则**@Autowired**的构造方法不再是必要的。如果有多个构造方法，那么至少一个必须被注解以便告诉容器使用哪个。

也可以在setter方法上使用**@Autowired**：

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...

}
```

也可以应用在具有任意名字和多个参数的方法上：

```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...

}
```

- 每个类只能有一个构造方法被标记为required，但可以有多个非必须的构造方法被注解。



#### 7.9.3 使用@Primary微调基于注解的自动装配

因为基于类型的自动装配可能会导致多个候选者，所以对这种过程通常需要更多的控制。一种方式是使用Spring的**@Primary**注解。它表示如果存在多个候选者且另一个bean只需要一个特定类型的bean依赖时，就使用标记了**@Primary**注解的那个依赖。如果只有一个候选者那就直接使用那个候选者即可。

我们假设下面的配置，定义**firstMovieCatalog**作为主要的（primary）**MovieCatalog**。

```
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...

}
```

使用这种配置，下面的**MovieRecommender**将装配**firstMovieCatalog**。

```
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...

}
```

通信的bean定义：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

#### 7.9.4 使用限定符微调基于注解的自动装配

当一个类型有几个实例时使用**@Primary**是一种有效的方式。当需要对选择过程做更多的控制时，那就需要用到Spring的**@Qualifier**注解了。为指定的参数绑定一个限定的值，可以缩小类型匹配的范围，使用这种方式，这个值可以是一个很普通的描述性的值：

```
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...

}
```

**@Qualifier**注解还能用在构造方法参数或方法参数上：

```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...

}
```

通信的bean定义如下所示。带有限定符“main”的bean会被装配到拥有同样值的构造方法参数上。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

也可创建自定义的限定符注解，只要定义一个注解并在其定义上添加**@Qualifier**即可：

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

然后就可以在字段或参数上使用自定义的限定符了：

```
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;
    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...

}
```

接着，提供候选的bean定义的信息。可以添加**<qualifier/>**标签作为**<bean/>**标签的子标签，并指定其**type**和**value**去匹配自定义的限定符注解。类型通过这个注解的全路径匹配，当然，如果没有风险的话也可以使用其类名。两种方式如下所示：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

在一些情况下，使用没有值的注解可能就足够了。这非常有用当注解提供了一种更通用的目的，并且可以运用到不同的依赖上。例如，当没有网络时可能会需要一种*offline*的类别。第一步定义这个简单的注解：

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```

然后，添加这个注解到将被自动装配的字段或属性上：

```
public class MovieRecommender {

    @Autowired
    @Offline
    private MovieCatalog offlineCatalog;

    // ...

}
```

然后，这个bean定义就只需要限定符类型：

```
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/>
    <!-- inject any dependencies required by this bean -->
</bean>
```

也可以自定义限定符注解，使它们带有命名的属性或者替代简单的**value**属性。如果多个属性值被指定在一个将被装配的字段或参数上，那么bean的定义必须匹配所有的属性值。例如，请看下面的注解定义：

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();

}
```

这种情况下**Format**是一个枚举：

```
public enum Format {
    VHS, DVD, BLURAY
}
```

被装配的字段使用这个自定义的限定符注解，它包含两个属性：**genre**和**format**。

```
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...

}
```

最后，这些bean的定义需要包含这些限定符。这个例子也展示了bean的*meta*属性可以使用**<qualifier/>**子元素代替。如果可以，优先使用**<qualifier/>**及其属性，但是，如果滑限定符自动装配机制会使用**<meta/>**标签提供的值，就像下面例子中最后两个bean的定义那样：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

#### 7.9.5 使用泛型作为自动装配限定符

除了**@Qualifier**注解，也可以使用Java的泛型类型作为一种显式的限定。例如，假设有如下配置：

```
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }

}
```

假设，上述bean实现了泛型接口，例如**Store<String>**和**Store<Integer>**，可以使用**@Autowired**装配**Store**接口，并且泛型会作为一种限定：

```
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

泛型限定符也可以用于自动装配的List、Map或数组：

```
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

#### 7.9.6 CustomAutowireConfigurer

**CustomAutowireConfigurer**是一个**BeanFactoryPostProcessor**，它可以注册自定义的限定符注解类型，甚至它们没有被Spring的**@Qualifier**注解所注解。

```
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

**AutowireCandidateResolver**通过以下方式决定了自动装配的候选者：

- 每个bean定义的**auto-candidate**值
- **<beans/>**元素上定义的**default-autowire-candidates**模式
- 使用了**@Qualifier**注解或任何在**CustomAutowireConfigurer**中注册的自定义注解

当多个bean被限定为候选者时，主要决定因素如下：如果这些候选者中有一个bean定义上明确地设置了**primay**属性为**true**，那么它将被选择。

#### 7.9.7 @Resource

Spring也支持使用JSR-250的**@Resource**注解在字段和setter方法上进行注入。这在Java EE 5和6中是一种通用的模式，例如，在JSF 1.2管理的bean或JAX-WS 2.0终端。Spring也同样支持这种模式来管理对象。

**@Resource**拥有一个name属性，默认地，Spring会把这个name属性的值解释为将要注入的bean的名字。换句话说，它按照名字的语法进行注入，如下例所示：

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder")
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

}
```

如果没有提供名字，默认的名字从字段名或setter方法名派生而来。对于一个字段，它会取字段的名字，对于stter方法，它会取bean的属性名。所以，下面的方法会使用名字为“movieFinder”的bean注入到setter方法中。

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

}
```

- 这个注解提供的名字会被**ApplicationContext**的**CommonAnnotationBeanPostProcessor**解析为一个bean的名字。如果显式地配置了Spring的**SimpleJndiBeanFactory**，这个名字也可以通过JNDI进行解析。但是，我们建议你使用默认的行为，简单地使用Spring的JNDI查找能力来间接寻址。

如果**@Resource**没有显式地指定名字，与**@Autowired**类似，它会寻找主要的（primary）类型匹配如果没有找到默认的名字，并解决众所周知的依赖：**BeanFactory, ApplicationContext, ResourceLoader, ApplicationEventPublisher,** 和**MessageSource**接口。

因此，下面的例子中**customerPreferenceDao**字段首先会寻找名字为*customerPreferenceDao*的bean，然后才会寻找接口**CustomerPreferenceDao**的主要的类型匹配。同样地，“context”字段会寻找**ApplicationContext**类型的已知的可解析的依赖。

```
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...

}
```

#### 7.9.8 @PostConstruct和@PreDestroy

**CommonAnnotationBeanPostProcessor**不仅能够识别到**@Resource**注解，还能识别到JSR-250的生命周期注解。这是在Spring 2.5引入的，这项支持为初始化回调及销毁回调又提供了一种替代方案。**CommonAnnotationBeanPostProcessor**是在**ApplicationContext**中注册的，因此带有这些注解的方法会与Spring自身的生命周期接口方法或显式声明的回调方法在同样调用。下面的例子中，缓存会在初始化的时候设置并在销毁时清除。

```
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }

}
```



### 7.10 类路径扫描及管理的组件

采用注解（比如**@Component**）、AspectJ表达式或自定义的过滤规则可以选择哪些类将被注册到容器中，而不再通过XML的形式执行bean的注册。

#### 7.10.1 @Component及其扩展注解

**@Repository**注解是一种用于标识存储类（也被称为数据访问对象或者DAO）的标记。异常的自动翻译是这个标记的用法之一。

Spring提供了一些扩展注解：**@Component， @Service**和**@Controller**。**@Component**可用于管理任何Spring的组件。**@Repository， @Service**和**@Controller**是**@Component**用于指定用例的特殊形式，比如，在持久层、服务层和表现层。使用**@Service**或**@Controller**能够让类更易于被合适的工具处理或与切面（aspect）关联。比如，这些注解可以使目标组件成为切入点。当然，**@Repository， @Service**和**@Controller**也能携带更多的语义。因此，如果你还在考虑使用**@Component**还是**@Service**用于注解service层，那么就选**@Service**吧，它更清晰。同样地，如前面所述，**@Repository**还能够用于在持久层标记自动异常翻译。

#### 7.10.2 元注解

Spring提供了很多注解可用于元注解。元注解即一种可用于别的注解之上的注解。例如，**@Service**就是一种被元注解**@Component**注解的注解：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Spring will see this and treat @Service in the same way as @Component
public @interface Service {

    // ....
}
```

元注解也可以组合起来形成组合注解。例如，**@RestController**注解是一种**@Controller**与**@ResponseBody**组合的注解。

另外，组合注解也可以重新定义来自元注解的属性。这在只想暴露元注解的部分属性值的时候非常有用。例如，Spring的**@SessionScope**注解把它的作用域硬编码为**session**，但是仍然允许自定义**proxyMode**。

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```

**@SessionScope**然后就可以使用了，而且不需要提供**proxyMode**，如下：

```
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

或者重写**proxyMode**的值，如下：

```
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

#### 7.10.3 自动检测类并注册bean定义

Spring能够自动检测被注解的类，并把它们注册到**ApplicationContext**中。例如，下面的两个会被自动检测到：

```
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

}
```

```
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

为了能够自动检测到这些类并注册它们，需要为**@Configuration**类添加**@ComponentScan**注解，并设置它的**basePackage**属性为这两个类所在的父包（替代方案，也可以使用逗号、分号、空格分割这两个类所在的包）。

```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

- 上面的配置也可以简单地使用这个注解的**value**属性，例如：**ComponentScan(“org.example”)**

也可以使用XML形式的配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>123456789101112
```

- 使用**context:component-scan**隐式地允许**context:annotation-config**的功能。因此，使用**context:component-scan**时一般就不需要再包含**context:annotation-config**元素了。
- 类路径扫描的包必须保证这些包出现在**classpath**中。。

另外，使用**component-scan**元素时默认也启用了**AutowiredAnnotationBeanPostProcessor**和**CommonAnnotationBeanPostProcessor**。这意味着这两个组件被自动检测到了且不需要在XML中配置任何元数据。

- 可以使用**annotation-config**元素并设置其属性为false来禁用**AutowiredAnnotationBeanPostProcessor**和**CommonAnnotationBeanPostProcessor**。

#### 7.10.4 使用过滤器自定义扫描

默认地，只有使用注解**@Component, @Repository, @Service, @Controller**或自定义注解注解的类才能被检测为候选组件。然而，我们可以使用自定义的过滤器修改并扩展这种行为。添加这些过滤器到**@ComponentScan**注解的*includeFilters*或*excludeFilters*参数即可（或**component-scan**元素的子元素*include-filter*或*exclude-filter*）。每个过滤器元素都需要**type**和**expression**属性。下表描述了相关的选项： 
**表7.5. 过滤器类型**

| 过滤器类型       | 表达式例子                 | 描述                                                     |
| ---------------- | -------------------------- | -------------------------------------------------------- |
| annotation(默认) | org.example.SomeAnnotation | 目标组件类级别的注解                                     |
| assignable       | org.example.SomeClass      | 目标组件继承或实现的类或接口                             |
| aspectj          | org.example..*Service+     | 用于匹配目标组件的AspecJ类型表达式                       |
| regex            | org.example.Default.*      | 用于匹配目标组件类名的正则表达式                         |
| custom           | org.example.MyTypeFilter   | org.springframework.core.type.TypeFilter接口的自定义实现 |

下面的例子展示了如何忽略掉所有的**@Repository**注解，并使用带有“stub”的Repository代替：

```
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

或者使用XML形式配置：

```
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

- 也可以设置这个注解的**useDefaultFilters=false**或为**<component-scan/>**元素提供属性**use-default-filters=”false”**忽略掉默认的过滤器。这将不会自动检测带有**@Component, @Repository, @Service, @Controller**或**@Configuration**注解的类。

#### 7.10.5 在组件内部定义bean元数据

Spring的组件也可以为容器贡献bean的定义元数据，只要在**@Component**注解的类内部使用**@Bean**注解即可。下面是一个简单的例子：

```
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }

}
```

- 除了扮演组件初始化的角色，**@Lazy**注解还可以放置在被**@Autowired**或**@Inject**标记的注入点。在这种情况下，它会使得注入使用延迟代理。

自动装配的字段和方法也可以像前面讨论的一样被支持，也可以支持**@Bean**方法的自动装配：

```
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters

    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }

}

```

上面的例子使用了另一个叫作**privateInstance**的bean的**Age**属性自动装配了**String**类型的参数**country**。Spring的表达式语言使用**#{}**的记法定义了这个属性的值。对于**@Value**注解，提前配置的表达式解析器会在需要解析表达式文本的时候寻找bean的名字。

#### 7.10.6 命名自动检测的组件

当一个组件被扫描过程自动检测到时，它的名字由**BeanNameGenerator**定义的策略生成。默认地，Spring的扩展注解（**@Component, @Repository, @Service**和**@Controller**）都包含一个**value**属性，这个**value**值会提供一个名字以便通信。

如果这样的注解没有明确地提供**value**值，或者另外一些检测到的组件（比如自定义过滤器扫描到的组件），那么默认生成器会返回一个首字母小写的短路径的类名。比如，下面两个组件，它们的名字分别为**myMovieLister**和**movieFinderImpl**：

```
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

```
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

- 如果不想遵循默认的名字生成策略，也可以提供自定义的策略。首先，需要实现**BeanNameGenerator**接口，并且要包含一个无参构造方法。然后，配置扫描器时为其指定这个自定义生成器的全路径：

  ```
  @Configuration
  @ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
  public class AppConfig {
      ...
  }
  ```

  ```
  <beans>
      <context:component-scan base-package="org.example"
          name-generator="org.example.MyNameGenerator" />
  </beans>
  ```

一般地，当其它的组件可能会明确地引用这个组件时为其注解提供一个名字是个很好地方式。另外，当容器装配时自动生成的名字足够用了。

#### 7.10.7 为自动检测的组件提供作用域

一般Spring管理的组件的作用域默认为**singleton**。但是，有时可能会需要不同的作用域，这时可以通过**@Scope**注解来声明：

```
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

- 也可以自定义策略处理作用域而不是依靠这种注解的方法，实现**ScopeMetadataResolver**接口，并包含一个默认的无参构造方法，然后在配置扫描器的时候提供其全路径即可。 
  `@Configuration @ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class) public class AppConfig { ... } `
  `<beans> <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver" /> </beans> `

当使用非单例作用域时，有必要为作用域内的对象生成代理。**component-scan**元素需要指明**scoped-proxy**属性。有三种可选值：无，接口和目标类。例如，下面的配置将使用标准的JDK动态代理：

```
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    ...
}
```

```
<beans>
    <context:component-scan base-package="org.example"
        scoped-proxy="interfaces" />
</beans>
```

#### 7.10.8 使用注解提供限定符

使用**qualifier**或**meta**子元素可以为bean提供限定符。同样地，也可以在类级别提供注解达到同样的效果。下面的三个例子展示了用法：

```
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

- 与大部分注解一样，请记住注解元数据是绑定到类定义本身的，然而XML形式允许为相同类型提供多个bean并绑定不同的限定符，因为XML的元数据是绑定到每个实例的而不是每个类。

### 7.11 使用JSR 330标准注解

从Spring 3.0开始，Spring开始支持JSR-330的标准注解用于依赖注入。这些注解与Spring自带的注解一样被扫描。仅仅只需要引入相关的jar包即可。

- 如果使用Maven，**javax.inject**也可以在标准Maven仓库中找到，添加如下配置到pom.xml即可。

  ```
  <dependency>
      <groupId>javax.inject</groupId>
      <artifactId>javax.inject</artifactId>
      <version>1</version>
  </dependency>
  ```

#### 7.11.1 使用@Inject和@Named依赖注入

可以像下面这样使用**@javax.inject.Inject**代替**@Autowired**：

```
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        ...
    }
}
```

与**@Autowired**一样，可以在字段级别、方法级别或构造参数级别使用**@Inject**。另外，也可以定义注入点为**Provider**，以便按需访问短作用域的bean或通过调用**Provider.get()**延迟访问其它的bean。上面例子的一种变体如下：

```
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        ...
    }
}
```

#### 7.11.2 @Named：与@Component注解等价

可以像下面这样使用**@javax.inject.Named**代替**@Component**：

```
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

通常使用**@Component**都不指定名字，同样地**@Named**也可以这么用：

```
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

使用**@Named**时，也可以像使用Spring注解一样使用组件扫描：

```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

- 与**@Component**不同的是，JSR-330的**@Named**注解不能组合成其它的注解，因此，如果需要构建自定义的注解，应使用Spring的注解。

#### 7.11.3 JSR-330标准注解的局限性

**表 7.6. Spring组件模型与JSR-330变种的对比**

| Spring              | javax.inject.*    | javax.inject的局限性                                         |
| ------------------- | ----------------- | ------------------------------------------------------------ |
| @Autowired          | @Inject           | @Inject没有require属性，可以使用Java 8的Optional代替。       |
| @Component          | @Named            | JSR-330没有提供组合模型，仅仅只是一种标识组件的方式          |
| @Scope(“singleton”) | @Singleton        | JSR-330默认的作用域类似于Spring的prototype。然而，为了与Spring一般的配置的默认值保持一致，JSR-330配置的bean在Spring中默认为singleton。为了使用singleton以外的作用域，必须使用Spring的@Scope注解。javax.inject也提供了一个@Scope注解，不过这仅仅被用于创建自己的注解。 |
| @Qualifier          | @Qualifier/@Named | javax.inject.Qualifier仅使用创建自定义的限定符。可以通过javax.inject.Named创建与Spring中@Qualifier一样的限定符 |
| @Value              | -                 | 无                                                           |
| @Required           | -                 | 无                                                           |
| @Lazy               | -                 | 无                                                           |
| ObjectFactory       | Provider          | javax.inject.Provider是对Spring的ObjectFactory的直接替代，仅仅使用简短的get()方法即可。它也可以与Spring的@Autowired或无注解的构造方法和setter方法一起使用。 |

### 7.12 基于Java的容器配置

#### 7.12.1 基本概念：@Bean和@Configuration

Spring中基于Java的配置的核心内容是**@Configuration**注解的类和**@Bean**注解的方法。

**@Bean**注解表示一个方法将会实例化、配置并初始化一个对象，且这个对象会被Spring容器管理。这就像在XML中**<beans/>**元素中**<bean/>**元素一样。**@Bean**注解可用于任何Spring的**@Component**注解的类中，但大部分都只用于**@Configuration**注解的类中。

注解了**@Configuration**的类表示这个类的目的就是作为bean定义的地方。另外，**@Configuration**类内部的bean可以调用本类中定义的其它bean作为依赖。最简单的配置大致如下：

```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }

}
```

上面的**AppConfig**类与下面的XML形式是等价的：

```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```



基于Java配置以不同的方式创建Spring的容器：

#### 7.12.2 使用AnnotationConfigApplicationContext实例化Spring容器

**AnnotationConfigApplicationContext**，这个**ApplicationContext**的实现不仅可以把**@Configuration**类作为输入，同样普通的**@Component**类和使用JSR-330注解的类也可以作为输入。

当使用**@Configuration**类作为输入时，这个类本身及其下面的所有**@Bean**方法都会被注册为bean。

当**@Component**和JSR-330类作为输入时，它们会被注册为bean，并且假设在必要的时候使用了**@Autowired**或**@Inject**。

##### 简单的构造方法

与使用**ClassPathXmlApplicationContext**注入XML文件一样，可以使用**AnnotationConfigApplicationContext**注入**@Configuration**类。这样就完全不用在Spring容器中使用XML了：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如前面所述，**AnnotationConfigApplicationContext**不限于只注入**@Configuration**类，任何**@Component**或JSR-330注解的类都能被提供给这个构造方法。例如：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

上面假设了**MyServiceImpl， Dependency1， Dependency2**使用了Spring的依赖注入注解比如**@Autowired**。

##### 使用register(Class）

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

##### 使用scan(String…)扫描组件

为了扫描组件，只要像下面这样配置**@Configuration**类即可：

```
@Configuration
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```

- 等价的XML形式配置： 
  `<beans> <context:component-scan base-package="com.acme"/> </beans> `

- **@Configuration**类是被**@Component**元注解注解的类，所以它们也会被扫描到。上面的例子中，假设**AppConfig**定义在**com.acme**包中（或更深的包中），调用**scan()**时它也会被扫描到，并且它里面配置的所有**@Bean**方法会在**refresh()**的时候被注册到容器中。

##### 使用AnnotationConfigWebApplicationContext支持web应用

一个**WebApplicationContext**与**AnnotationConfigApplicationContext**的变种是**AnnotationConfigWebApplicationContext**。这个实现可以用于配置Spring的**ContextLoaderListener**的servlet监听器、Spring MVC的**DispatcherServlet**等。下面是一个典型的配置Spring MVC web应用的片段。包含**contextClass**的context-param和init-param的用法：

```
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 7.12.3 使用@Bean注解

**@Bean**是方法级别的注解，它与XML中的**<bean/>**类似，同样地，也支持**<bean/>**的一些属性，比如，init-method, destroy-method, autowring和name。

可以在**@Configuration**或**@Component**注解的类中使用**@Bean**注解。

##### 声明bean

只要在方法上简单的加上**@Bean**注解就可以定义一个bean了，这样就在**ApplicationContext**中注册了一个类型为方法返回值的bean。默认地，bean的名字为方法的名称，如下所示：

```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

上面的方式与下面的XML形式等价：

```
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

两种方式都定义了一个名字为**transferService**的bean，且绑定了**TransferServiceImpl**的实例：

```
transferService -> com.acme.TransferServiceImpl1
```

##### Bean之间的依赖

**@Bean**注解的方法可以有任意个参数用于描述这个bean的依赖关系。比如，如果**TransferService**需要一个**AccountRepository**，我们可以通过方法参数实现这种依赖注入。

```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }

}
```

##### 接收生命周期回调

任何使用**@Bean**定义的类都有正常的生命周期回调，并且可以使用**@PostConstruct**和**@PreDestroy**注解。

正常的生命周期回调被完美支持， 如果一个bean实现了**InitializingBean, DisposableBean**或者**Lifecycle**，它们的方法将被容器依次调用。

同样地，也支持***Aware**系列的接口，比如BeanFactoryAware, BeanNameAware, MessageSourceAware, ApplicationContextAware等等。

**@Bean**注解也支持任意的初始化及销毁的回调方法，这与XML的init-method和destroy-method是非常类似的。

```
public class Foo {
    public void init() {
        // initialization logic
    }
}

public class Bar {
    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }

    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }

}
```

- 默认地，使用Java配置的方式如果一个bean包含了公共的close或shutdown方法，它们将会被自动地包含在销毁的回调中。如果有公共的close或shutdown方法，但是我们并不想使用它们，那么只要加上**@Bean(destroyMethod=”“)**就可以屏蔽掉默认的推测了。 
  

上面例子中的Foo，也可以直接在构造期间直接调用init()方法：

```
@Configuration
public class AppConfig {
    @Bean
    public Foo foo() {
        Foo foo = new Foo();
        foo.init();
        return foo;
    }

    // ...

}
```

- 在Java中可以用喜欢的方式直接操作对象，而不需要总是依赖容器的生命周期！

##### 指定bean的作用域

###### 使用@Scope注解

可以使用任何标准的方式为@Bean注解的bean指定一个作用域

默认地作用域为singleton，但是可以使用@Scope注解重写：

```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }

}
```

###### @Scope和scoped-proxy

Spring提供了一种简便地方式声明bean的作用域，它被称为scoped proxy。最简单地方式是创建那么一个代理，使用XML配置的形式则使用**<aop:scoped-proxy/>**元素。在Java中使用@Scope注解配置bean的方式提供了与XML代理模式属性同样的功能。默认是没有代理的(ScopedProxyMode.No)，但是可以指定ScopedProxyMode.TARGET_CLASS或ScopedProxyMode.INTERFACES。

如果把xml形式改写为Java形式，看起来如下：

```
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

##### 自定义bean的名称

默认地，使用@Bean默认的方法名为其bean的名字，然而这项功能可以使用name属性重写：

```
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }

}
```

##### bean的别名

有时候可以给同一个bean多个名字，亦即别名，@Bean注解的name属性就可以达到这样的目的， 可以为其提供一个String的数组。

```
@Configuration
public class AppConfig {

    @Bean(name = { "dataSource", "subsystemA-dataSource", "subsystemB-dataSource" })
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }

}
```

##### Bean描述

有时可能需要为一个bean提供更详细的描述。这对于监控bean很有用。

可以使用@Description注解为其添加一段描述：

```
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Foo foo() {
        return new Foo();
    }
```

#### 使用@Configuration注解

@Configuration是类级别的注解，这表示一个对象是bean定义的来源。@Configuration注解的类里面使用@Bean注解的方法声明bean。对其中的@Bean方法的调用也能实现内部bean的依赖。

##### 注入内部依赖

当@Bean的方法对另外一个有依赖时，简单地调用另外一个@Bean注解的方法即可表达这种依赖：

```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }

}
```

在上例中，foo这个bean通过构造函数注入接收了bar的引用。

- 这种方式仅仅适用于在@Configuration内部定义的@Bean方法。在普通的@Component类中不能声明内部依赖。

##### 查找方法注入

查找方法注入是一项很少使用到的先进的技术。它对于一个单例bean依赖另一个原型bean很有用。在Java中使用了一种很自然的方法来实现了这种模式。

```
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();

        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
    return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}12345678910111213
```

使用Java配置，可以创建一个CommandManager的子类，实现其createCommand()方法，这样就可以让它查找到新的原型command对象。

```
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with command() overridden
    // to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

#### 7.12.5 组合的Java配置

##### 使用@Import注解

与XML中使用**<import/>**一样用于模块化配置，@Import注解允许从另一个配置类中加载@Bean定义。

```
@Configuration
public class ConfigA {

     @Bean
    public A a() {
        return new A();
    }

}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }

}
```

现在，我们不需要同时指定ConfigA.class和ConfigB.class了，只要明确地指定ConfigB即可：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

这种方式简化了容器的实例化，只要处理一个类就行了，而不需要开发者记住大量的@Configuration类。

##### 在导入的@Bean定义上注入依赖

在大部分场景下，bean都会依赖另一个配置类中的bean。

@Bean方法可以有任意的参数用于描述其依赖。

有几个配置类，并且每个都依赖于其它的类：

```
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }

}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }

}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

@Configuration类也是容器中的一个bean：它们可以像其它bean一样使用@Autowired和@Value注解。



同样地，对于通过@Bean声明的BeanPostProcessor和BeanFactoryPostProcessor请谨慎对待。它们通常被声明为静态的@Bean方法，不会触发包含它们的配置类。另外，@Autowired和@Value在配置类本身上是不起作用的，因为它们太早被实例化了。

```
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }

}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    @Autowired
    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

- @Configuration类中的构造方法注入只在Spring 4.3以后才支持。另外请注意，如果目标bean只有一个构造方法也可以不指定@Autowried，在上例中，RepositoryConfig构造方法上的@Autowired是非必要的。

在上面的场景中，@Autowired可以很好地工作，并提供希望的结果，但是被装配的bean的定义声明是模糊不清的。在某些情况下，这种含糊是不被接受的，并且你希望可以在IDE中直接从一个@Configuration类到另一个，可以考虑装配配置类本身：

```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }

}
```

在上面的场景下，AccountRepository的定义就很明确了。然而，ServiceConfig与RepositoryConfig耦合了；这是一种折衷的方法。这种耦合某种程度上可以通过接口或抽象解决，如下：

```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();

}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }

}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class}) // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

现在ServiceConfig就与具体的DefaultRepositoryConfig松耦合了，并且内置的IDE工具也可以生效：对于 开发者可以很容易地获得RepositoryConfig实现类的继承体系。使用这种方式，操纵@Configuration类和它们的依赖与基于接口的代码没有区别。

##### 有条件地包含@Configuration类或@Bean方法

有时候有条件地包含或不包含一个@Configuration类或@Bean方法很有用，这基于特定的系统状态。一种通用的方法是使用@Profile注解去激活bean，仅当指定的配置文件包含在了Spring的环境中才有效。

@Profile注解实际是实现了一个更灵活的注解@Conditional。@Condition注解表明一个@Bean被注册之前会先询问@Condition。

Condition接口的实现只要简单地提供matches(…)方法，并返回true或false即可。例如，下面是一个实际的Condition实现用于@Profile：

```
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
        // Read the @Profile annotation attributes
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
    }
    return true;
}
```

更多信息请参考@Conditional的javadoc。

##### 绑定Java与XML配置

Spring的@Configuration类并不能100%地替代XML配置。一些情况下使用XML的命名空间仍然是最理想的方式来配置容器。在某些场景下，XML是很方便或必要的，你可以选择以XML为主，比如ClassPathXmlApplicationContext，或者以Java为主使用AnnotationConfigApplicationContext并在需要的时候使用@ImportResource注解导入XML配置。

###### XML为主使用@Configuration类

更受人喜爱的方法是从XML启动容器并包含@Configuration类。例如，在大型的已存在的系统中，以前是使用XML配置的，所以很容易地创建@Configuration类，并包含他们到XML文件中，下面我们会讲解以XML为主的案例。

记住@Configuration类仅仅用于bean的定义。在这个例子中，我们创建了一个叫做AppConfig的配置类，并把它包含到system-test-config.xml中。因为**context:annotation-config/**打开了，所以容器能够识别到@Configuration注解并处理其中的@Bean方法。

```
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }

}
```

system-test-config.xml:

```
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

jdbc.properties:

```
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=123
```

```
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

- 在上面的system-test-config.xml中，AppConfig的<bean/>并没有声明一个id。当然如果声明了也是可以授受的，但是对于没有被其它bean引用的bean并有必要声明id，并且它也没可能从容器获取一个明确的名字。同样地，DataSource也不需要一个明确的id，因为它是通过类型装配的。

因为@Configuration是被元注解@Component注解的，所以@Configuration注解的类也可以被自动扫描。同样使用上面的例子，我们可以重新定义system-test-config.xml来使用组件扫描。注意，这种情况下，我们就没必要明确地声明context:annotation-config/了，因为<context:component-scan/&tl;已经包含了同样的功能。

system-test-config.xml:

```
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

###### 以@Configuration类为主使用@ImportResource引入XML

在一些应用中，@Configuration类是主要的配置方式，也需要使用一些XML配置。在这些场景下，简单地使用@ImportResource并按需要定义XML文件即可。这种方式可以以Java为主，并保持少量的XML配置。

```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }

}
```



```
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

```
jdbc.properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=1234
```

```

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

