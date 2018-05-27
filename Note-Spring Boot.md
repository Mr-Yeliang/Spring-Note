## 1 构建系统



### 1.1 依赖管理

每个版本的Spring Boot提供了一个它所支持的依赖关系列表。 不需要为构建配置文件中的这些依赖关系提供版本，Spring Boot会管理这些依赖的版本。升级Spring Boot本身时，这些依赖关系也将以一致的进行升级。如果有必要，仍然可以指定一个版本并覆盖Spring Boot建议的版本。

### 1.2 启动器

启动器是一组方便的依赖关系描述符，可以包含在应用程序中。用于获得所需的所有Spring和相关技术的一站式服务，无需通过示例代码搜索和复制粘贴依赖配置。例如，如果要开始使用Spring和JPA进行数据库访问，那么只需在项目中包含spring-boot-starter-data-jpa依赖关系即可。

所有正式起动器都遵循类似的命名模式： spring-boot-starter- * ，其中 * 是特定类型的应用程序。  启动器包含许多依赖关系，包括需要使项目快速启动并运行，并具有一致的受支持的依赖传递关系。



## 2. 构建代码

### 查找主应用程序类

通常建议将应用程序主类放到其他类之上的根包(root package)中。 @EnableAutoConfiguration注解通常放置在的主类上，它隐式定义了某些项目的基本“搜索包”。 例如，如果正在编写JPA应用程序，则@EnableAutoConfiguration注解类的包将用于搜索@Entity项。

使用根包(root package)还可以使用@ComponentScan注释，而不需要指定basePackage属性。 如果的主类在根包中，也可以使用@SpringBootApplication注释。

这是一个典型的布局：

```
com
 +- example
     +- myproject
         +- Application.java
         |
         +- domain
         |   +- Customer.java
         |   +- CustomerRepository.java
         |
         +- service
         |   +- CustomerService.java
         |
         +- web
             +- CustomerController.java
```

Application.java文件将声明main方法以及基本的@Configuration。

```
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```



## 3. 配置类

通常建议主source是@Configuration类。

### 3.1 导入其他配置类

 @Import注解可用于导入其他配置类。 或者，可以使用@ComponentScan自动扫描所有Spring组件，包括@Configuration类。

### 3.2 导入XML配置

建议从@Configuration类开始，使用@ImportResource注释来加载XML配置文件。



## 4. 自动配置

Spring Boot 会根据添加的jar依赖关系自动配置Spring应用程序。例如，如果HSQLDB在类路径上，并且没有手动配置任何数据库连接bean，那么将自动配置内存数据库。

需要通过将@EnableAutoConfiguration或@SpringBootApplication注解添加到一个@Configuration类中来选择自动配置。只添加一个@EnableAutoConfiguration注解时通常建议将其添加到主@Configuration类中。

### 4.1逐渐取代自动配置

自动配置是非侵入式的，可以随时定义自己的配置来替换自动配置。 例如，如果添加自己的 DataSource bean，则默认的嵌入式数据库支持将会退回。

### 4.2 禁用指定的自动配置

如果正在使用一些不需要的自动配置类，可以使用@EnableAutoConfiguration的exclude属性来禁用它们。

```
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

如果类不在classpath路径上，则可以使用注释的excludeName属性，并指定全限定名(fully qualified name)。 最后，还可以通过spring.autoconfigure.exclude属性控制要排除的自动配置类列表。

> 注解和使用属性(property)定义都可以指定要排除的自动配置类。



## 5. Spring Beans 和 依赖注入

建议使用@ComponentScan搜索bean，结合@Autowired构造函数(constructor)注入。

将应用程序类放在根包(root package)中构建代码，则可以使用 @ComponentScan而不使用任何参数， 所有应用程序组件（@Component，@Service，@Repository，@Controller等）将自动注册为Spring Bean。

以下是一个@Service Bean的例子，可以使用构造函数注入获取RiskAssessor bean。

```
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

如果一个bean 只有一个构造函数，则可以省略@Autowired。

```
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

> 注意，如何使用构造函数注入允许将RiskAssessor字段标记为final，表示不能更改。



## 6. @SpringBootApplication注解

Spring Boot提供了一个方便的@SpringBootApplication注解来标注主类，以此替代@Configuration，@EnableAutoConfiguration和@ComponentScan。

```
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

> @SpringBootApplication还提供了别名来定制@EnableAutoConfiguration和@ComponentScan的属性。



## 7. 运行应用程序

将应用程序打包成jar并使用嵌入式HTTP服务器可以按照任何方式运行应用程序。调试Spring Boot应用程序也很容易; 不需要任何专门的IDE插件或扩展。

### 7.1 作为已打包应用程序运行

如果使用Spring Boot 的 Maven或Gradle插件创建可执行jar，则可以使用java -jar运行应用程序。 例如：

```
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar
```

也可以启用远程调试支持运行打包的应用程序。 这允许将调试器添加到打包的应用程序中：

```
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myproject-0.0.1-SNAPSHOT.jar
```

### 7.2 使用 Maven 插件

Spring Boot Maven 插件包含一个运行目标(goal )，可用于快速编译和运行应用程序。 应用程序以exploded的形式运行，就像在IDE中一样。

```
$ mvn spring-boot:run
```

使用操作系统环境变量：

```
$ export MAVEN_OPTS=-Xmx1024m -XX:MaxPermSize=128M
```



## 8. 开发工具

 spring-boot-devtools模块可以包含在任何项目中，以提供额外的开发时功能。 要包含devtools支持，只需将模块依赖关系添加到构建中：

**Maven：**

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

### 8.1 自动重启

使用spring-boot-devtools的应用程序将在类路径上的文件发生更改时自动重新启动。

**触发重启**

当DevTools监视类路径资源时，触发重新启动的唯一方法是更新类路径中的文件时。 导致类路径更新的方式取决于正在使用的IDE。 在Eclipse中，保存修改的文件将导致类路径被更新并触发重新启动。 在IntelliJ IDEA中，构建项目（Build→Make Project）将具有相同的效果。

#### 8.1.1 排除资源

在类路径下，某些资源在更改时不一定需要触发重新启动。 例如，Thymeleaf模板可以直接编辑不需重启。 
默认情况下，有一些排除项，更改 /META-INF/maven，/META-INF/resources，/resources，/static，/public或/templates中的资源不会触发重新启动，但会触发 实时重新加载。
如果要自定义这些排除项，可以使用spring.devtools.restart.exclude属性。 例如，要仅排除 /static和 /public，可以设置：

```
spring.devtools.restart.exclude=static/**,public/**
```

> 如果要保留这些默认值并添加其他排除项，应改用spring.devtools.restart.additional-exclude属性。

#### 8.1.2 监视额外的路径

有时当对不在类路径中的文件进行更改时，需要重新启动或重新加载应用程序。为此，应使用spring.devtools.restart.additional-paths属性来配置其他路径以监视更改。 

可以使用上述的spring.devtools.restart.exclude属性来控制附加路径下的更改是否会触发完全重新启动或只是实时重新加载。

#### 8.1.3 禁用重启

如果不想使用重新启动功能，可以使用spring.devtools.restart.enabled属性来禁用它。 在大多数情况下，可以在application.properties中设置此项（这仍将初始化重新启动类加载器，但不会监视文件更改）。

例如，如果需要完全禁用重新启动支持，因为它在一些特定库中不能正常运行，则需要在调用SpringApplication.run（…）之前设置System属性。 例如：

```
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

#### 8.1.4 使用触发文件

如果使用IDE工具编写代码，更改文件，则可能希望仅在特定时间触发重新启动。 为此，可以使用“触发文件”，这是一个特殊文件，当要实际触发重新启动检查时，必须修改它。 更改文件只会触发检查，只有在Devtools检测到它必须执行某些操作时才会重新启动。 触发文件可以手动更新，也可以通过IDE插件更新。

要使用触发器文件，应使用spring.devtools.restart.trigger-file属性。

### 8.2 全局设置

可以通过向 $HOME 文件夹添加名为.spring-boot-devtools.properties的文件来配置全局devtools设置（应注意文件名以“.”开头）。 添加到此文件的任何属性将适用于的计算机上使用devtools的所有Spring Boot应用程序。 例如，要配置重新启动以始终使用触发器文件，可以添加以下内容：

**~/.spring-boot-devtools.properties.**

```
spring.devtools.reload.trigger-file=.reloadtrigger
```

### 8.3 远程应用

Spring Boot开发工具不仅限于本地开发。 远程运行应用程序时也可以使用多种功能。 远程支持是可选择的，要使其能够确保重新打包的存档中包含devtools：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

那么需要设置一个spring.devtools.remote.secret属性，例如：

```
spring.devtools.remote.secret=mysecret
```

远程devtools支持分为两部分： 有一个接受连接的服务器端和在IDE中运行的客户端应用程序。 当spring.devtools.remote.secret属性设置时，服务器组件将自动启用。 客户端组件必须手动启动。

#### 8.3.1 运行远程客户端应用程序

远程客户端应用程序旨在从IDE中运行。 需要使用与要连接的远程项目相同的类路径运行org.springframework.boot.devtools.RemoteSpringApplication。 传递给应用程序的必选参数应该是要连接到的远程URL。

例如，如果使用Eclipse或STS，并且有一个名为my-app的项目已部署到Cloud Foundry，则可以执行以下操作：

- 从Run 菜单中选择Run Configurations…。
- 创建一个新的Java Application “launch configuration”。
- 浏览my-app项目。
- 使用org.springframework.boot.devtools.RemoteSpringApplication作为主类。
- 将https://myapp.cfapps.io添加到程序参数（或任何远程URL）中。

运行的远程客户端将如下所示：

```
  .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote :: 1.5.2.RELEASE

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```

> 由于远程客户端正在使用与实际应用程序相同的类路径，因此可以直接读取应用程序属性。 这是spring.devtools.remote.secret属性如何读取并传递到服务器进行身份验证。
>
> 建议使用https//作为连接协议，以便流量被加密，防止密码被拦截。
>
> 如果需要使用代理访问远程应用程序，应配置spring.devtools.remote.proxy.host和spring.devtools.remote.proxy.port属性。

#### 8.3.2 远程更新

远程客户端将以与本地相同的方式监视应用程序类路径的更改。 任何更新的资源将被推送到远程应用程序，并且（如果需要的话）触发重新启动。 如果正在迭代使用当地没有的云服务的功能，这可能会非常有用。 通常，远程更新和重新启动比完全重建和部署周期要快得多。

> 仅在远程客户端运行时才监视文件。 如果在启动远程客户端之前更改文件，则不会将其推送到远程服务器。



## 9. SpringApplication

SpringApplication类提供了一种方便的方法来将Spring应用程序从main()方法启动。 在许多情况下，只需授权静态方法SpringApplication.run()：

```
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

当的应用程序启动时，应该看到类似于以下内容：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v1.5.2.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatEmbeddedServletContainerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)123456789101112
```

默认情况下，将显示INFO 级别log消息，包括用户启动应用程序一些相关的启动细节。

### 9.1 启动失败

如果的应用程序无法启动，则注册的FailureAnalyzers会提供专门的错误消息和具体操作来解决问题。 例如，如果在端口8080上启动Web应用程序，并且该端口已在使用中，则应该会看到类似于以下内容的内容：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```



如果没有故障分析器(analyzers)能够处理异常，仍然可以显示完整的自动配置报告，以更好地了解出现的问题。 为此，需要启用debug属性或启用org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer的DEBUG日志。

例如，如果使用java -jar运行应用程序，则可以按如下方式启用 debug：

```
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### 9.2 自定义Banner

可以通过在的类路径中添加一个 banner.txt 文件，或者将banner.location设置到banner文件的位置来更改启动时打印的banner。 如果文件有一些不常用的编码，你可以设置banner.charset（默认为UTF-8）。除了文本文件，还可以将banner.gif，banner.jpg或banner.png图像文件添加到的类路径中，或者设置一个banner.image.location属性。 图像将被转换成ASCII艺术表现，并打印在任何文字banner上方。

可以在banner.txt文件中使用以下占位符：

#### 表23.1. banner变量

| 变量名                                                       | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ${application.version}                                       | 在MANIFEST.MF中声明的应用程序的版本号。例如， Implementation-Version: 1.0 被打印为 1.0. |
| ${application.formatted-version}                             | 在MANIFEST.MF中声明的应用程序版本号的格式化显示（用括号括起来，以v为前缀）。 例如 (v1.0)。 |
| ${spring-boot.version}                                       | 正在使用的Spring Boot版本。 例如1.5.2.RELEASE。              |
| ${spring-boot.formatted-version}                             | 正在使用格式化显示的Spring Boot版本（用括号括起来，以v为前缀）。 例如（v1.5.2.RELEASE）。 |
| Ansi.NAME(orAnsi.NAME(or{AnsiColor.NAME}, AnsiBackground.NAME,AnsiBackground.NAME,{AnsiStyle.NAME}) | 其中NAME是ANSI转义码的名称。 有关详细信息，应参阅 [AnsiPropertySource](https://github.com/spring-projects/spring-boot/tree/v1.5.2.RELEASE/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)。 |
| ${application.title}                                         | 的应用程序的标题在MANIFEST.MF中声明。 例如Implementation-Title：MyApp打印为MyApp。 |

> 如果要以编程方式生成banner，则可以使用SpringApplication.setBanner()方法。 使用org.springframework.boot.Banner 如接口，并实现自己的printBanner() 方法。

还可以使用spring.main.banner-mode属性来决定是否必须在System.out（控制台）上打印banner，使用配置的logger（log）或不打印（off）。

### 9.3 定制SpringApplication

如果SpringApplication默认值不符合的想法，可以创建本地实例并进行自定义。 例如，关闭banner：

```
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

> 传递给SpringApplication的构造函数参数是spring bean的配置源。 在大多数情况下，这些将引用@Configuration类，但它们也可以引用XML配置或应扫描的包。

也可以使用application.properties文件配置SpringApplication。

### 9.4 流式构建 API

如果需要构建一个ApplicationContext层次结构（具有父/子关系的多个上下文），或者如果只想使用“流式（fluent）”构建器API，则可以使用SpringApplicationBuilder。

SpringApplicationBuilder允许链式调用多个方法，并包括允许创建层次结构的父和子方法。

例如：

```
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```

> 创建ApplicationContext层次结构时有一些限制，例如 Web组件必须包含在子上下文中，并且相同的环境将用于父和子上下文。

### 9.5 Application events and listeners

除了常见的Spring Framework事件（如**ContextRefreshedEvent** ）之外，SpringApplication还会发送一些其他应用程序事件。

> 在创建ApplicationContext之前，实际上触发了一些事件，因此不能在@Bean上注册一个监听器。 可以通过SpringApplication.addListeners(…) 或SpringApplicationBuilder.listeners(…)方法注册它们。
>
> 如果希望自动注册这些侦听器，无论创建应用程序的方式如何，都可以将META-INF / spring.factories文件添加到项目中，并使用org.springframework.context.ApplicationListener引用的侦听器。 
> org.springframework.context.ApplicationListener=com.example.project.MyListener

当的应用程序运行时，事件按照以下顺序发送：

1. ApplicationStartingEvent在运行开始时发送，但在注册侦听器和注册初始化器之后。
2. 当已经知道要使用的上下文(context)环境，并在context创建之前，将发送ApplicationEnvironmentPreparedEvent。
3. ApplicationPreparedEvent在启动刷新(refresh)之前发送，但在加载了bean定义之后。
4. ApplicationReadyEvent在刷新之后被发送，并且处理了任何相关的回调以指示应用程序准备好服务应求。
5. 如果启动时发生异常，则发送ApplicationFailedEvent。

> 一般不需要使用应用程序事件，但可以方便地知道它们存在。 在内部，Spring Boot使用事件来处理各种任务。

### 9.6 Web 环境

SpringApplication将尝试代表创建正确类型的ApplicationContext。 默认情况下，将使用AnnotationConfigApplicationContext或AnnotationConfigEmbeddedWebApplicationContext，具体取决于是否正在开发Web应用程序。

用于确定“Web环境”的算法是相当简单的（基于几个类的存在）。 如果需要覆盖默认值，可以使用setWebEnvironment（boolean webEnvironment）。

也可以通过调用setApplicationContextClass() 对ApplicationContext完全控制。

> 在JUnit测试中使用SpringApplication时，通常需要调用setWebEnvironment()

### 9.7 访问应用程序参数

如果需要访问传递给SpringApplication.run()的应用程序参数，则可以注入org.springframework.boot.ApplicationArguments bean。 ApplicationArguments接口提供对原始String []参数以及解析选项和非选项参数的访问：

```
import org.springframework.boot.*
import org.springframework.beans.factory.annotation.*
import org.springframework.stereotype.*

@Component
public class MyBean {

    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }

}
```

> Spring Boot还将向Spring Environment 注册一个CommandLinePropertySource。 这允许也使用@Value注解注入应用程序参数。

### 9.8 使用ApplicationRunner或CommandLineRunner

SpringApplication启动时如果需要运行一些特定的代码，就可以实现ApplicationRunner或CommandLineRunner接口。 两个接口都以相同的方式工作，并提供一个单独的运行方法，这将在SpringApplication.run（…）完成之前调用。

CommandLineRunner接口提供对应用程序参数的访问（简单的字符串数组），而ApplicationRunner使用上述的ApplicationArguments接口。

```
@Component
public class MyBean implements CommandLineRunner {

    public void run(String... args) {
        // Do something...
    }

}
```

如果定义了若干CommandLineRunner或ApplicationRunner bean，这些bean必须按特定顺序调用，可以实现org.springframework.core.Ordered接口，也可以使用org.springframework.core.annotation.Order注解。

### 9.9 Application exit

每个SpringApplication将注册一个JVM关闭钩子，以确保ApplicationContext在退出时正常关闭。 可以使用所有标准的Spring生命周期回调（例如DisposableBean接口或@PreDestroy注释）。

另外，如果希望在应用程序结束时返回特定的退出代码，那么bean可以实现org.springframework.boot.ExitCodeGenerator接口。

### 9.10 管理功能

可以通过指定spring.application.admin.enabled属性来为应用程序启用与管理相关的功能。 这会在平台MBeanServer上暴露SpringApplicationAdminMXBean。 可以使用此功能来远程管理的Spring Boot应用程序。 这对于任何服务包装器(service wrapper)实现也是有用的。
