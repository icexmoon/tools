# Spring：核心技术——IoC容器

**说明：本文翻译自Spring官方文档[Core Technologies --- 核心技术 (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)。**

> Version 6.0.7

This part of the reference documentation covers all the technologies that are absolutely integral to the Spring Framework.
参考文档的这一部分涵盖了Spring框架中不可或缺的所有技术。

Foremost amongst these is the Spring Framework’s Inversion of Control (IoC) container. A thorough treatment of the Spring Framework’s IoC container is closely followed by comprehensive coverage of Spring’s Aspect-Oriented Programming (AOP) technologies. The Spring Framework has its own AOP framework, which is conceptually easy to understand and which successfully addresses the 80% sweet spot of AOP requirements in Java enterprise programming.
其中最重要的是Spring框架的控制反转（Inversion of Control，IoC）容器。在全面介绍Spring框架的IoC容器之后，紧接着是对Spring的面向方面编程（AOP）技术的全面介绍。Spring框架有自己的AOP框架，它在概念上易于理解，并且成功地解决了Java企业编程中80%的AOP需求。

Coverage of Spring’s integration with AspectJ (currently the richest — in terms of features — and certainly most mature AOP implementation in the Java enterprise space) is also provided.
本文还介绍了Spring与AspectJ的集成（就特性而言，目前是最丰富的，当然也是Java企业领域中最成熟的AOP实现）。

AOT processing can be used to optimize your application ahead-of-time. It is typically used for native image deployment using GraalVM.
AOT处理可用于提前优化应用程序。它通常用于使用GraalVM的本机映像部署。

## 1. IoC容器

This chapter covers Spring’s Inversion of Control (IoC) container.
本章介绍Spring的控制反转（Inversion of Control，IoC）容器。

### 1.1. Spring IoC容器和Bean简介

This chapter covers the Spring Framework implementation of the Inversion of Control (IoC) principle. IoC is also known as dependency injection (DI). It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.
本章介绍了控制反转（IoC）原则的Spring框架实现。IoC也被称为依赖注入（DI）。在此过程中，对象仅通过构造函数参数、工厂方法的参数或在对象实例构造或从工厂方法返回后在对象实例上设置的属性来定义其依赖项（即，与它们一起工作的其他对象）。然后，容器在创建bean时注入这些依赖项。这个过程基本上是bean本身的逆过程（因此称为控制反转），通过使用类的直接构造或诸如服务定位器模式之类的机制来控制其依赖项的实例化或位置。

The `org.springframework.beans` and `org.springframework.context` packages are the basis for Spring Framework’s IoC container. The [`BeanFactory`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/beans/factory/BeanFactory.html) interface provides an advanced configuration mechanism capable of managing any type of object. [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/ApplicationContext.html) is a sub-interface of `BeanFactory`. It adds:
`org.springframework.beans` 和 `org.springframework.context` 包是Spring框架的IoC容器的基础。 `BeanFactory` 接口提供了能够管理任何类型对象的高级配置机制。 `ApplicationContext` 是 `BeanFactory` 的子接口。它添加了（额外功能）：

- Easier integration with Spring’s AOP features
  更容易与Spring的AOP特性集成

- Message resource handling (for use in internationalization)
  消息资源处理（用于国际化）

- Event publication

  事件发布

- Application-layer specific contexts such as the `WebApplicationContext` for use in web applications.
  应用程序层特定上下文，如用于Web应用程序的 `WebApplicationContext` 。

In short, the `BeanFactory` provides the configuration framework and basic functionality, and the `ApplicationContext` adds more enterprise-specific functionality. The `ApplicationContext` is a complete superset of the `BeanFactory` and is used exclusively in this chapter in descriptions of Spring’s IoC container. For more information on using the `BeanFactory` instead of the `ApplicationContext,` see the section covering the [`BeanFactory` API](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory).
简而言之， `BeanFactory` 提供了配置框架和基本功能，而 `ApplicationContext` 添加了更多特定于企业的功能。 `ApplicationContext` 是 `BeanFactory` 的完整超集，在本章描述Spring的IoC容器时专门使用。有关使用 `BeanFactory` 而不是 `ApplicationContext` 的详细信息，请参见 `BeanFactory` API部分。

In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.
在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。这正是bean有别于应用程序中其他对象的地方。Bean以及它们之间的依赖关系反映在容器使用的配置元数据中。

### 1.2. 容器概述

The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata. The configuration metadata is represented in XML, Java annotations, or Java code. It lets you express the objects that compose your application and the rich interdependencies between those objects.
`org.springframework.context.ApplicationContext` 接口表示Spring IoC容器，负责实例化、配置和组装bean。容器通过阅读配置元数据来获取关于实例化、配置和组装哪些对象的指令。配置元数据以XML、Java注释或Java代码表示。它允许您表示组成应用程序的对象以及这些对象之间丰富的相互依赖关系。

Several implementations of the `ApplicationContext` interface are supplied with Spring. In stand-alone applications, it is common to create an instance of [`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) or [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html). While XML has been the traditional format for defining configuration metadata, you can instruct the container to use Java annotations or code as the metadata format by providing a small amount of XML configuration to declaratively enable support for these additional metadata formats.
Spring提供了 `ApplicationContext` 接口的几个实现。在独立应用程序中，通常创建 `ClassPathXmlApplicationContext` 或 `FileSystemXmlApplicationContext` 的实例。虽然XML一直是定义配置元数据的传统格式，但您可以通过提供少量XML配置以声明方式启用对这些附加元数据格式的支持，来指示容器使用Java注释或代码作为元数据格式。

In most application scenarios, explicit user code is not required to instantiate one or more instances of a Spring IoC container. For example, in a web application scenario, a simple eight (or so) lines of boilerplate web descriptor XML in the `web.xml` file of the application typically suffices (see [Convenient ApplicationContext Instantiation for Web Applications](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-create)). If you use the [Spring Tools for Eclipse](https://spring.io/tools) (an Eclipse-powered development environment), you can easily create this boilerplate configuration with a few mouse clicks or keystrokes.
在大多数应用程序场景中，实例化Spring IoC容器的一个或多个实例不需要显式用户代码。例如，在Web应用程序场景中，应用程序的 `web.xml` 文件中简单的八行（左右）样板Web描述符XML通常就足够了（请参见Web应用程序的便捷ApplicationContext实例化）。如果您使用Spring Tools for Eclipse（一个由Eclipse支持的开发环境），您可以通过几次鼠标点击或击键轻松地创建这个样板配置。

The following diagram shows a high-level view of how Spring works. Your application classes are combined with configuration metadata so that, after the `ApplicationContext` is created and initialized, you have a fully configured and executable system or application.
下图显示了Spring工作方式的高级视图。您的应用程序类与配置元数据相结合，这样，在创建并初始化 `ApplicationContext` 之后，您就拥有了一个完全配置好的可执行系统或应用程序。

![container magic](https://docs.spring.io/spring-framework/docs/current/reference/html/images/container-magic.png)

Figure 1. The Spring IoC container
图1. Spring IoC容器

#### 1.2.1. 配置元数据

As the preceding diagram shows, the Spring IoC container consumes a form of configuration metadata. This configuration metadata represents how you, as an application developer, tell the Spring container to instantiate, configure, and assemble the objects in your application.
如上图所示，Spring IoC容器使用一种形式的配置元数据。此配置元数据表示您作为应用程序开发人员，告诉Spring容器如何实例化、配置和组装应用程序中的对象。

Configuration metadata is traditionally supplied in a simple and intuitive XML format, which is what most of this chapter uses to convey key concepts and features of the Spring IoC container.
配置元数据通常以简单直观的XML格式提供，本章大部分内容都使用这种格式来传达Spring IoC容器的关键概念和特性。

>XML-based metadata is not the only allowed form of configuration metadata. The Spring IoC container itself is totally decoupled from the format in which this configuration metadata is actually written. These days, many developers choose [Java-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) for their Spring applications.
>基于XML的元数据不是配置元数据的唯一允许形式。SpringIoC容器本身与实际编写配置元数据的格式完全分离。目前，许多开发人员选择基于Java的配置来配置Spring应用程序。

For information about using other forms of metadata with the Spring container, see:
有关对Spring容器使用其它形式的元数据的信息，请参见：

- [Annotation-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config): define beans using annotation-based configuration metadata.
  基于注解的配置：使用基于注释的配置元数据定义bean。
- [Java-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java): define beans external to your application classes by using Java rather than XML files. To use these features, see the [`@Configuration`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/annotation/Import.html), and [`@DependsOn`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/annotation/DependsOn.html) annotations.
  基于Java的配置：使用Java而不是XML文件定义应用程序类外部的bean。要使用这些功能，请参见 `@Configuration` 、 `@Bean` 、 `@Import` 和 `@DependsOn` 注释。

Spring configuration consists of at least one and typically more than one bean definition that the container must manage. XML-based configuration metadata configures these beans as `<bean/>` elements inside a top-level `<beans/>` element. Java configuration typically uses `@Bean`-annotated methods within a `@Configuration` class.
Spring配置由容器必须管理的至少一个、通常多个bean定义组成。基于XML的配置元数据将这些bean配置为顶层 `<beans/>` 元素中的 `<bean/>` 元素。Java配置通常在 `@Configuration` 类中使用 `@Bean` 注释的方法。

These bean definitions correspond to the actual objects that make up your application. Typically, you define service layer objects, persistence layer objects such as repositories or data access objects (DAOs), presentation objects such as Web controllers, infrastructure objects such as a JPA `EntityManagerFactory`, JMS queues, and so forth. Typically, one does not configure fine-grained domain objects in the container, because it is usually the responsibility of repositories and business logic to create and load domain objects.
这些bean定义对应于组成应用程序的实际对象。通常，您定义服务层对象、持久层对象（如存储库或数据访问对象（DAO））、表示对象（如Web控制器）、基础结构对象（如JPA `EntityManagerFactory` ）、JMS队列等。通常，用户不会在容器中配置细粒度的域对象，因为创建和加载域对象通常是存储库和业务逻辑的职责。

The following example shows the basic structure of XML-based configuration metadata:
以下示例显示了基于XML的配置元数据的基本结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="..."> (1) (2)
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

1. The `id` attribute is a string that identifies the individual bean definition.
   `id` 属性是标识单个bean定义的字符串。
2. The `class` attribute defines the type of the bean and uses the fully qualified class name.
   `class` 属性定义bean的类型，并使用完全限定的类名。

The value of the `id` attribute can be used to refer to collaborating objects. The XML for referring to collaborating objects is not shown in this example. See [Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies) for more information.
`id` 属性的值可用于引用协作对象。本例中没有显示用于引用协作对象的XML。有关详细信息，请参见依赖项。

#### 1.2.2. 实例化容器

The location path or paths supplied to an `ApplicationContext` constructor are resource strings that let the container load configuration metadata from a variety of external resources, such as the local file system, the Java `CLASSPATH`, and so on.
提供给 `ApplicationContext` 构造函数的一个或多个位置路径是资源字符串，它允许容器从各种外部资源（如本地文件系统、Java `CLASSPATH` 等）加载配置元数据。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

> After you learn about Spring’s IoC container, you may want to know more about Spring’s `Resource` abstraction (as described in [Resources](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources)), which provides a convenient mechanism for reading an InputStream from locations defined in a URI syntax. In particular, `Resource` paths are used to construct applications contexts, as described in [Application Contexts and Resource Paths](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-app-ctx).
> 在了解Spring的IoC容器之后，您可能希望了解有关Spring的 `Resource` 抽象（如[资源](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources)中所述）的更多信息，它提供了一种方便的机制，用于从URI语法中定义的位置读入InputStream。特别地， `Resource` 路径用于构造应用上下文，如[应用上下文和资源路径](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-app-ctx)中所述。

The following example shows the service layer objects `(services.xml)` configuration file:
以下示例显示了服务层对象 `(services.xml)` 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

The following example shows the data access objects `daos.xml` file:
以下示例显示数据访问对象 `daos.xml` 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

In the preceding example, the service layer consists of the `PetStoreServiceImpl` class and two data access objects of the types `JpaAccountDao` and `JpaItemDao` (based on the JPA Object-Relational Mapping standard). The `property name` element refers to the name of the JavaBean property, and the `ref` element refers to the name of another bean definition. This linkage between `id` and `ref` elements expresses the dependency between collaborating objects. For details of configuring an object’s dependencies, see [Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies).
在前面的示例中，服务层由 `PetStoreServiceImpl` 类和 `JpaAccountDao` 和 `JpaItemDao` 类型的两个数据访问对象（基于JPA对象关系映射标准）组成。 `property name` 元素引用JavaBean属性的名称， `ref` 元素引用另一个bean定义的名称。 `id` 和 `ref` 元素之间的这种链接表达了协作对象之间的依赖性。有关配置对象依赖关系的详细信息，请参阅[依赖关系](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies)。

##### 合成基于XML的配置元数据

It can be useful to have bean definitions span multiple XML files. Often, each individual XML configuration file represents a logical layer or module in your architecture.
让bean定义跨越多个XML文件是很有用的。通常，每个单独的XML配置文件都表示体系结构中的一个逻辑层或模块。

You can use the application context constructor to load bean definitions from all these XML fragments. This constructor takes multiple `Resource` locations, as was shown in the [previous section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-instantiation). Alternatively, use one or more occurrences of the `<import/>` element to load bean definitions from another file or files. The following example shows how to do so:
您可以使用应用程序上下文构造器从所有这些XML片段加载bean定义。此构造函数接受多个 `Resource` 位置，如上一节所示。或者，通过使用一个或多个 `<import/>` 标签定义从另一个或多个文件加载bean定义。下面的示例说明了如何执行此操作：

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

In the preceding example, external bean definitions are loaded from three files: `services.xml`, `messageSource.xml`, and `themeSource.xml`. All location paths are relative to the definition file doing the importing, so `services.xml` must be in the same directory or classpath location as the file doing the importing, while `messageSource.xml` and `themeSource.xml` must be in a `resources` location below the location of the importing file. As you can see, a leading slash is ignored. However, given that these paths are relative, it is better form not to use the slash at all. The contents of the files being imported, including the top level `<beans/>` element, must be valid XML bean definitions, according to the Spring Schema.
在前面的示例中，从三个文件加载外部Bean定义：`services.xml`、`messageSource.xml`和 `themeSource.xml`.。所有位置路径都相对于执行导入的定义文件，因此 `services.xml` 必须与执行导入的文件位于同一目录或类路径位置，而 `messageSource.xml` 和 `themeSource.xml` 必须位于导入文件位置下方的 `resources` 位置。如您所见，前导斜杠被忽略。然而，考虑到这些路径是相对的，最好不要使用斜线。根据Spring Schema，导入的文件内容（包括顶层 `<beans/>` 元素）必须是有效的XMLbean定义。

> It is possible, but not recommended, to reference files in parent directories using a relative "../" path. Doing so creates a dependency on a file that is outside the current application. In particular, this reference is not recommended for `classpath:` URLs (for example, `classpath:../services.xml`), where the runtime resolution process chooses the “nearest” classpath root and then looks into its parent directory. Classpath configuration changes may lead to the choice of a different, incorrect directory.
> 可以（但不建议）使用相对“../”引用父目录中的文件“path。这样做会创建对当前应用程序外部文件的依赖项。特别是，不建议将此引用用于 `classpath:` URL（例如， `classpath:../services.xml` ），在这些URL中，运行时解析进程选择“最近的”类路径根，然后查看其父目录。类路径配置更改可能导致选择不同的、不正确的目录。
>
> You can always use fully qualified resource locations instead of relative paths: for example, `file:C:/config/services.xml` or `classpath:/config/services.xml`. However, be aware that you are coupling your application’s configuration to specific absolute locations. It is generally preferable to keep an indirection for such absolute locations — for example, through "${…}" placeholders that are resolved against JVM system properties at runtime.
> 您可以始终使用完全限定的资源位置，而不是相对路径：例如 `file:C:/config/services.xml` 或 `classpath:/config/services.xml` 。但是，请注意，您将应用程序的配置耦合到特定的绝对位置。通常最好为这样的绝对位置保留一个间接寻址--例如，通过“${...}”占位符，在运行时根据JVM系统属性解析这些占位符。

The namespace itself provides the import directive feature. Further configuration features beyond plain bean definitions are available in a selection of XML namespaces provided by Spring — for example, the `context` and `util` namespaces.
命名空间本身提供了import指令功能。除了普通bean定义之外，Spring提供的XML命名空间中还提供了更多配置特性——例如， `context` 和 `util` 命名空间。

##### Groovy Bean定义DSL

As a further example for externalized configuration metadata, bean definitions can also be expressed in Spring’s Groovy Bean Definition DSL, as known from the Grails framework. Typically, such configuration live in a ".groovy" file with the structure shown in the following example:
作为外部化配置元数据的另一个示例，bean定义也可以用Spring的Groovy Bean Definition DSL来表示，这在Grails框架中是已知的。通常，这样的配置存在于一个.groovy文件中，其结构如下例所示：

```groovy
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

This configuration style is largely equivalent to XML bean definitions and even supports Spring’s XML configuration namespaces. It also allows for importing XML bean definition files through an `importBeans` directive.
这种配置风格在很大程度上等同于XMLbean定义，甚至支持Spring的XML配置名称空间。它还允许通过 `importBeans` 指令导入XML bean定义文件。

#### 1.2.3. 使用容器

The `ApplicationContext` is the interface for an advanced factory capable of maintaining a registry of different beans and their dependencies. By using the method `T getBean(String name, Class<T> requiredType)`, you can retrieve instances of your beans.
`ApplicationContext` 是高级工厂的接口，该工厂能够维护不同bean及其依赖项的注册表。通过使用方法 `T getBean(String name, Class<T> requiredType)` ，您可以检索bean的实例。

The `ApplicationContext` lets you read bean definitions and access them, as the following example shows:
`ApplicationContext` 允许您读取和访问Bean定义，如以下示例所示：

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

With Groovy configuration, bootstrapping looks very similar. It has a different context implementation class which is Groovy-aware (but also understands XML bean definitions). The following example shows Groovy configuration:
使用Groovy配置时，引导看起来非常相似。它有一个不同的上下文实现类，它是Groovy感知的（但也理解XMLbean定义）。下面的示例显示了Groovy配置：

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

The most flexible variant is `GenericApplicationContext` in combination with reader delegates — for example, with `XmlBeanDefinitionReader` for XML files, as the following example shows:
最灵活的变体是 `GenericApplicationContext` 与reader委托的组合——例如，对于XML文件，使用 `XmlBeanDefinitionReader` ，如下例所示：

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

You can also use the `GroovyBeanDefinitionReader` for Groovy files, as the following example shows:
您还可以将 `GroovyBeanDefinitionReader` 用于Groovy文件，如下例所示：

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

You can mix and match such reader delegates on the same `ApplicationContext`, reading bean definitions from diverse configuration sources.
您可以在同一个 `ApplicationContext` 上混合和匹配这样的读取器委托，从不同的配置源阅读bean定义。

You can then use `getBean` to retrieve instances of your beans. The `ApplicationContext` interface has a few other methods for retrieving beans, but, ideally, your application code should never use them. Indeed, your application code should have no calls to the `getBean()` method at all and thus have no dependency on Spring APIs at all. For example, Spring’s integration with web frameworks provides dependency injection for various web framework components such as controllers and JSF-managed beans, letting you declare a dependency on a specific bean through metadata (such as an autowiring annotation).
然后可以使用 `getBean` 检索bean的实例。 `ApplicationContext` 接口有一些其他的方法来检索bean，但是，理想情况下，应用程序代码不应该使用它们。实际上，您的应用程序代码根本不应该调用 `getBean()` 方法，因此根本不依赖于SpringAPI。例如，Spring与Web框架的集成为各种Web框架组件（如控制器和JSF管理的bean）提供了依赖注入，允许您通过元数据（如自动装配注释）声明对特定bean的依赖。

### 1.3. Bean概述

A Spring IoC container manages one or more beans. These beans are created with the configuration metadata that you supply to the container (for example, in the form of XML `<bean/>` definitions).
Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据（例如，以XML `<bean/>` 定义的形式）创建的。

Within the container itself, these bean definitions are represented as `BeanDefinition` objects, which contain (among other information) the following metadata:
在容器本身中，这些bean定义表示为 `BeanDefinition` 对象，其中包含（以及其他信息）以下元数据：

- A package-qualified class name: typically, the actual implementation class of the bean being defined.
  包限定的类名：通常是正在定义的bean的实际实现类。
- Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).
  Bean行为配置元素，这些元素规定了Bean在容器中的行为方式（范围、生命周期回调等）。
- References to other beans that are needed for the bean to do its work. These references are also called collaborators or dependencies.
  对Bean执行其工作所需的其他Bean的引用。这些引用也称为协作者或依赖项。
- Other configuration settings to set in the newly created object — for example, the size limit of the pool or the number of connections to use in a bean that manages a connection pool.
  要在新创建的对象中设置的其他配置设置——例如，池的大小限制或要在管理连接池的Bean中使用的连接数。

This metadata translates to a set of properties that make up each bean definition. The following table describes these properties:
此元数据转换为一组属性，这些属性构成了每个bean定义。下表介绍了这些属性：

表1. Bean定义

| Property                                | Explained in… 解释于...                                      |
| :-------------------------------------- | :----------------------------------------------------------- |
| Class                                   | [Instantiating Beans 实例化Bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| Name                                    | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname) |
| Scope                                   | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
| Constructor arguments 构造函数参数      | [Dependency Injection 依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Properties                              | [Dependency Injection 依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Autowiring mode                         | [Autowiring Collaborators 自动装配协作者](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) |
| Lazy initialization mode 惰性初始化模式 | [Lazy-initialized Beans 延迟初始化Bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
| Initialization method 初始化方法        | [Initialization Callbacks 初始化回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method 销毁方法             | [Destruction Callbacks 销毁回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) |

In addition to bean definitions that contain information on how to create a specific bean, the `ApplicationContext` implementations also permit the registration of existing objects that are created outside the container (by users). This is done by accessing the ApplicationContext’s `BeanFactory` through the `getBeanFactory()` method, which returns the `DefaultListableBeanFactory` implementation. `DefaultListableBeanFactory` supports this registration through the `registerSingleton(..)` and `registerBeanDefinition(..)` methods. However, typical applications work solely with beans defined through regular bean definition metadata.
除了包含如何创建特定bean的信息的bean定义之外， `ApplicationContext` 实现还允许注册在容器之外（由用户）创建的现有对象。这是通过 `getBeanFactory()` 方法访问ApplicationContext的 `BeanFactory` 来完成的，该方法返回 `DefaultListableBeanFactory` 实现。 `DefaultListableBeanFactory` 通过 `registerSingleton(..)` 和 `registerBeanDefinition(..)` 方法支持此注册。但是，典型的应用程序只使用通过常规bean定义元数据定义的bean。

> Bean metadata and manually supplied singleton instances need to be registered as early as possible, in order for the container to properly reason about them during autowiring and other introspection steps. While overriding existing metadata and existing singleton instances is supported to some degree, the registration of new beans at runtime (concurrently with live access to the factory) is not officially supported and may lead to concurrent access exceptions, inconsistent state in the bean container, or both.
> Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他内省步骤中正确地推理它们。虽然在某种程度上支持覆盖现有的元数据和现有的单例实例，但在运行时注册新bean（与对工厂的实时访问并发）并不受正式支持，这可能会导致并发访问异常、bean容器中的不一致状态，或者两者兼而有之。

#### 1.3.1. 命名Bean

Every bean has one or more identifiers. These identifiers must be unique within the container that hosts the bean. A bean usually has only one identifier. However, if it requires more than one, the extra ones can be considered aliases.
每个bean都有一个或多个标识符。这些标识符在承载bean的容器中必须是唯一的。bean通常只有一个标识符。但是，如果它需要多个别名，则可以将多余的别名视为别名。

In XML-based configuration metadata, you use the `id` attribute, the `name` attribute, or both to specify bean identifiers. The `id` attribute lets you specify exactly one `id`. Conventionally, these names are alphanumeric ('myBean', 'someService', etc.), but they can contain special characters as well. If you want to introduce other aliases for the bean, you can also specify them in the `name` attribute, separated by a comma (`,`), semicolon (`;`), or white space. Although the `id` attribute is defined as an `xsd:string` type, bean `id` uniqueness is enforced by the container, though not by XML parsers.
在基于XML的配置元数据中，可以使用 `id` 属性、 `name` 属性或同时使用这两个属性来指定Bean标识符。 `id` 属性允许您仅指定一个 `id` 。传统上，这些名称是字母数字（“myBean”、“someService”等），但是它们也可以包含特殊字符。如果要为bean引入其他别名，也可以在 `name` 属性中指定它们，用逗号（ `,` ）、分号（ `;` ）或空格分隔。尽管 `id` 属性被定义为 `xsd:string` 类型，但是容器强制bean `id` 的唯一性，而不是XML解析器。

You are not required to supply a `name` or an `id` for a bean. If you do not supply a `name` or `id` explicitly, the container generates a unique name for that bean. However, if you want to refer to that bean by name, through the use of the `ref` element or a Service Locator style lookup, you must provide a name. Motivations for not supplying a name are related to using [inner beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-inner-beans) and [autowiring collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire).
您不需要为Bean提供 `name` 或 `id` 。如果没有显式地提供 `name` 或 `id` ，容器将为该bean生成一个唯一的名称。但是，如果您希望通过使用 `ref` 元素或ServiceLocator样式查找按名称引用该bean，则必须提供名称。不提供名称的动机与使用[内部bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-inner-beans)和[自动装配协作者](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)有关。

> Bean Naming Conventions 
> Bean命名约定
>
> The convention is to use the standard Java convention for instance field names when naming beans. That is, bean names start with a lowercase letter and are camel-cased from there. Examples of such names include `accountManager`, `accountService`, `userDao`, `loginController`, and so forth.
> 约定是在命名bean时对实例字段名使用标准Java约定。也就是说，bean名称以小写字母开头，并从那里开始采用驼峰式大小写。这样的名称的示例包括 `accountManager` 、 `accountService` 、 `userDao` 、 `loginController` 等。
>
> Naming beans consistently makes your configuration easier to read and understand. Also, if you use Spring AOP, it helps a lot when applying advice to a set of beans related by name.
> 一致地命名bean使您的配置更易于阅读和理解。此外，如果您使用SpringAOP，那么在将建议应用于一组名称相关的bean时，它会有很大帮助。

> With component scanning in the classpath, Spring generates bean names for unnamed components, following the rules described earlier: essentially, taking the simple class name and turning its initial character to lower-case. However, in the (unusual) special case when there is more than one character and both the first and second characters are upper case, the original casing gets preserved. These are the same rules as defined by `java.beans.Introspector.decapitalize` (which Spring uses here).
> 通过在类路径中进行组件扫描，Spring会按照前面描述的规则为未命名的组件生成bean名称：本质上就是取简单的类名并将其首字符转换为小写。但是，在（不常见的）特殊情况下，当有多个字符并且第一个和第二个字符都是大写时，原始大小写将被保留。这些规则与 `java.beans.Introspector.decapitalize` （Spring在这里使用的）定义的规则相同。

##### 在Bean定义之外为Bean添加别名

In a bean definition itself, you can supply more than one name for the bean, by using a combination of up to one name specified by the `id` attribute and any number of other names in the `name` attribute. These names can be equivalent aliases to the same bean and are useful for some situations, such as letting each component in an application refer to a common dependency by using a bean name that is specific to that component itself.
在Bean定义本身中，您可以为Bean提供多个名称，方法是使用 `id` 属性指定的最多一个名称和 `name` 属性中任意数量的其他名称的组合。这些名称可以是同一个bean的等效别名，并且在某些情况下非常有用，例如，通过使用特定于应用程序中的每个组件本身的bean名称，使该组件引用一个公共依赖项。

Specifying all aliases where the bean is actually defined is not always adequate, however. It is sometimes desirable to introduce an alias for a bean that is defined elsewhere. This is commonly the case in large systems where configuration is split amongst each subsystem, with each subsystem having its own set of object definitions. In XML-based configuration metadata, you can use the `<alias/>` element to accomplish this. The following example shows how to do so:
但是，指定实际定义bean的所有别名并不总是足够的。有时需要为在其他地方定义的bean引入别名。这在大型系统中是常见的情况，其中配置在每个子系统之间被分割，每个子系统具有其自己的对象定义集。在基于XML的配置元数据中，可以使用 `<alias/>` 元素来完成此操作。下面的示例说明了如何执行此操作：

```xml
<alias name="fromName" alias="toName"/>
```

In this case, a bean (in the same container) named `fromName` may also, after the use of this alias definition, be referred to as `toName`.
在这种情况下，名为 `fromName` 的bean（在同一个容器中）在使用这个别名定义之后也可以称为 `toName` 。

For example, the configuration metadata for subsystem A may refer to a DataSource by the name of `subsystemA-dataSource`. The configuration metadata for subsystem B may refer to a DataSource by the name of `subsystemB-dataSource`. When composing the main application that uses both these subsystems, the main application refers to the DataSource by the name of `myApp-dataSource`. To have all three names refer to the same object, you can add the following alias definitions to the configuration metadata:
例如，子系统A的配置元数据可以通过名称 `subsystemA-dataSource` 来引用数据源。子系统B的配置元数据可以引用名称为 `subsystemB-dataSource` 的数据源。当组合使用这两个子系统的主应用程序时，主应用程序通过名称 `myApp-dataSource` 引用DataSource。要使所有三个名称都引用同一对象，可以将以下别名定义添加到配置元数据中：

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

Now each component and the main application can refer to the dataSource through a name that is unique and guaranteed not to clash with any other definition (effectively creating a namespace), yet they refer to the same bean.
现在，每个组件和主应用程序都可以通过一个唯一的名称引用dataSource，并保证不会与任何其他定义冲突（有效地创建了一个名称空间），但它们引用的是同一个bean。

> Java-configuration 
> Java配置
>
> If you use Java Configuration, the `@Bean` annotation can be used to provide aliases. See [Using the `@Bean` Annotation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-bean-annotation) for details.
> 如果使用Java配置，则 `@Bean` 注释可用于提供别名。有关详细信息，请参见[使用 `@Bean` 注解](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-bean-annotation)。

#### 1.3.2. 实例化Bean

A bean definition is essentially a recipe for creating one or more objects. The container looks at the recipe for a named bean when asked and uses the configuration metadata encapsulated by that bean definition to create (or acquire) an actual object.
bean定义本质上是创建一个或多个对象的方法。当被询问时，容器查看命名bean的配置，并使用该bean定义封装的配置元数据来创建（或获取）实际对象。

If you use XML-based configuration metadata, you specify the type (or class) of object that is to be instantiated in the `class` attribute of the `<bean/>` element. This `class` attribute (which, internally, is a `Class` property on a `BeanDefinition` instance) is usually mandatory. (For exceptions, see [Instantiation by Using an Instance Factory Method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-instance-factory-method) and [Bean Definition Inheritance](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions).) You can use the `Class` property in one of two ways:
如果使用基于XML的配置元数据，则在 `<bean/>` 元素的 `class` 属性中指定要实例化的对象的类型（或类）。这个 `class` 属性（在内部，它是 `BeanDefinition` 实例上的 `Class` 属性）通常是强制的。(例外情况，请参见[使用实例工厂方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-instance-factory-method)和[Bean定义继承进行实例化](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions)。）可以通过以下两种方式之一使用 `Class` 属性：

- Typically, to specify the bean class to be constructed in the case where the container itself directly creates the bean by calling its constructor reflectively, somewhat equivalent to Java code with the `new` operator.
  通常，在容器本身通过反射调用其构造函数直接创建bean的情况下，指定要构造的bean类，这在某种程度上相当于Java代码中的 `new` 操作符。
- To specify the actual class containing the `static` factory method that is invoked to create the object, in the less common case where the container invokes a `static` factory method on a class to create the bean. The object type returned from the invocation of the `static` factory method may be the same class or another class entirely.
  指定包含 `static` 工厂方法的实际类，该方法被调用来创建对象，这种情况不太常见，即容器调用类上的 `static` 工厂方法来创建Bean。从 `static` 工厂方法的调用返回的对象类型可以是同一个类或完全是另一个类。

> Nested class names 嵌套类名
>
> If you want to configure a bean definition for a nested class, you may use either the binary name or the source name of the nested class.
> 如果要为嵌套类配置Bean定义，可以使用嵌套类的二进制名称或源名称。
>
> For example, if you have a class called `SomeThing` in the `com.example` package, and this `SomeThing` class has a `static` nested class called `OtherThing`, they can be separated by a dollar sign (`$`) or a dot (`.`). So the value of the `class` attribute in a bean definition would be `com.example.SomeThing$OtherThing` or `com.example.SomeThing.OtherThing`.
> 例如，如果在 `com.example` 包中有一个名为 `SomeThing` 的类，而这个 `SomeThing` 类有一个名为 `OtherThing` 的 `static` 嵌套类，则可以用美元符号（ `$` ）或点（ `.` ）分隔它们。因此，bean定义中 `class` 属性的值应该是 `com.example.SomeThing$OtherThing` 或 `com.example.SomeThing.OtherThing` 。

##### 使用构造函数实例化

When you create a bean by the constructor approach, all normal classes are usable by and compatible with Spring. That is, the class being developed does not need to implement any specific interfaces or to be coded in a specific fashion. Simply specifying the bean class should suffice. However, depending on what type of IoC you use for that specific bean, you may need a default (empty) constructor.
当您通过构造函数方法创建bean时，所有普通类都可以被Spring使用并与之兼容。也就是说，正在开发的类不需要实现任何特定的接口，也不需要以特定的方式进行编码。只需指定bean类就足够了。但是，根据您为特定bean使用的IoC类型，您可能需要一个默认（空）构造函数。

The Spring IoC container can manage virtually any class you want it to manage. It is not limited to managing true JavaBeans. Most Spring users prefer actual JavaBeans with only a default (no-argument) constructor and appropriate setters and getters modeled after the properties in the container. You can also have more exotic non-bean-style classes in your container. If, for example, you need to use a legacy connection pool that absolutely does not adhere to the JavaBean specification, Spring can manage it as well.
SpringIoC容器几乎可以管理您希望它管理的任何类。它并不局限于管理真正的JavaBean。大多数Spring用户更喜欢实际的JavaBeans，它只有一个默认的（无参数的）构造函数，以及按照容器中的属性建模的适当的setter和getter。您还可以在容器中包含更多的非bean样式的类。例如，如果您需要使用一个完全不符合JavaBean规范的遗留连接池，Spring也可以管理它。

With XML-based configuration metadata you can specify your bean class as follows:
使用基于XML的配置元数据，可以按如下方式指定Bean类：

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

For details about the mechanism for supplying arguments to the constructor (if required) and setting object instance properties after the object is constructed, see [Injecting Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators).
有关向构造函数提供参数（如果需要）以及在构造对象后设置对象实例属性的机制的详细信息，请参[见注入依赖项](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators)。

##### 使用静态工厂方法实例化

When defining a bean that you create with a static factory method, use the `class` attribute to specify the class that contains the `static` factory method and an attribute named `factory-method` to specify the name of the factory method itself. You should be able to call this method (with optional arguments, as described later) and return a live object, which subsequently is treated as if it had been created through a constructor. One use for such a bean definition is to call `static` factories in legacy code.
在定义使用静态工厂方法创建的Bean时，请使用 `class` 属性指定包含 `static` 工厂方法的类，并使用名为 `factory-method` 的属性指定工厂方法本身的名称。您应该能够调用此方法（使用可选参数，如后面所述）并返回一个活动对象，该对象随后被视为通过构造函数创建的对象。这种bean定义的一个用途是调用遗留代码中的 `static` 工厂。

The following bean definition specifies that the bean will be created by calling a factory method. The definition does not specify the type (class) of the returned object, but rather the class containing the factory method. In this example, the `createInstance()` method must be a `static` method. The following example shows how to specify a factory method:
下面的bean定义指定将通过调用工厂方法来创建bean。该定义不指定返回对象的类型（类），而是指定包含工厂方法的类。在本例中， `createInstance()` 方法必须是 `static` 方法。下面的示例说明如何指定工厂方法：

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

The following example shows a class that would work with the preceding bean definition:
下面的示例显示了一个将与前面的Bean定义一起使用的类：

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

For details about the mechanism for supplying (optional) arguments to the factory method and setting object instance properties after the object is returned from the factory, see [Dependencies and Configuration in Detail](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-properties-detailed).
有关在对象从工厂返回后向工厂方法提供（可选）参数和设置对象实例属性的机制的详细信息，请参见[依赖项和配置的详细信息](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-properties-detailed)。

##### 使用实例工厂方法进行实例化

Similar to instantiation through a [static factory method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-static-factory-method), instantiation with an instance factory method invokes a non-static method of an existing bean from the container to create a new bean. To use this mechanism, leave the `class` attribute empty and, in the `factory-bean` attribute, specify the name of a bean in the current (or parent or ancestor) container that contains the instance method that is to be invoked to create the object. Set the name of the factory method itself with the `factory-method` attribute. The following example shows how to configure such a bean:
与通过静态工厂方法实例化类似，使用实例工厂方法实例化从容器调用现有bean的非静态方法来创建新bean。要使用此机制，请将 `class` 属性保留为空，并在 `factory-bean` 属性中指定当前（或父代或祖先）容器中的bean的名称，该容器包含要调用以创建对象的实例方法。使用 `factory-method` 属性设置工厂方法本身的名称。以下示例说明如何配置此类Bean：

```java
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

The following example shows the corresponding class:
下面的示例显示了相应的类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

One factory class can also hold more than one factory method, as the following example shows:
一个工厂类还可以包含多个工厂方法，如下例所示：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

The following example shows the corresponding class:
下面的示例显示了相应的类：

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

This approach shows that the factory bean itself can be managed and configured through dependency injection (DI). See [Dependencies and Configuration in Detail](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-properties-detailed).
这种方法表明工厂bean本身可以通过依赖注入（DI）来管理和配置。请参阅[相关性和配置的详细信息](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-properties-detailed)。

> In Spring documentation, "factory bean" refers to a bean that is configured in the Spring container and that creates objects through an [instance](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-instance-factory-method) or [static](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-static-factory-method) factory method. By contrast, `FactoryBean` (notice the capitalization) refers to a Spring-specific [`FactoryBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-factorybean) implementation class.
> 在Spring文档中，“factory bean”指的是在Spring容器中配置的bean，它通过实例或静态工厂方法创建对象。相比之下， `FactoryBean` （注意大写）引用特定于Spring的 `FactoryBean` 实现类。

##### 确定Bean的运行时类型

The runtime type of a specific bean is non-trivial to determine. A specified class in the bean metadata definition is just an initial class reference, potentially combined with a declared factory method or being a `FactoryBean` class which may lead to a different runtime type of the bean, or not being set at all in case of an instance-level factory method (which is resolved via the specified `factory-bean` name instead). Additionally, AOP proxying may wrap a bean instance with an interface-based proxy with limited exposure of the target bean’s actual type (just its implemented interfaces).
确定特定bean的运行时类型并非易事。bean元数据定义中指定的类只是一个初始类引用，可能与声明的工厂方法结合，或者是 `FactoryBean` 类，这可能导致bean的不同运行时类型，或者在实例级工厂方法的情况下根本不设置（而是通过指定的 `factory-bean` 名称解析）。此外，AOP代理可以用基于接口的代理包装bean实例，并限制目标bean实际类型的公开（仅限于其实现的接口）。

The recommended way to find out about the actual runtime type of a particular bean is a `BeanFactory.getType` call for the specified bean name. This takes all of the above cases into account and returns the type of object that a `BeanFactory.getBean` call is going to return for the same bean name.
要了解特定bean的实际运行时类型，推荐的方法是对指定的bean名称进行 `BeanFactory.getType` 调用。这考虑了上述所有情况，并返回 `BeanFactory.getBean` 调用将为相同bean名称返回的对象类型。

### 1.4. 依赖

A typical enterprise application does not consist of a single object (or bean in the Spring parlance). Even the simplest application has a few objects that work together to present what the end-user sees as a coherent application. This next section explains how you go from defining a number of bean definitions that stand alone to a fully realized application where objects collaborate to achieve a goal.
一个典型的企业应用程序不是由单个对象（或者Spring术语中的bean）组成的。即使是最简单的应用程序也有几个对象，这些对象一起工作以呈现最终用户所看到的一致的应用程序。下一节将解释如何从定义多个独立的bean定义发展到一个完全实现的应用程序，在这个应用程序中，对象协作来实现一个目标。

#### 1.4.1. 依赖注入

Dependency injection (DI) is a process whereby objects define their dependencies (that is, the other objects with which they work) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies on its own by using direct construction of classes or the Service Locator pattern.
依赖注入（DI）是一个过程，对象仅通过构造函数参数、工厂方法的参数或在对象实例被构造或从工厂方法返回后在对象实例上设置的属性来定义它们的依赖关系（即，与它们一起工作的其他对象）。然后，容器在创建bean时注入这些依赖项。这个过程基本上是bean本身的逆过程（因此得名“控制反转”），bean本身通过使用类的直接构造或服务定位器模式来控制其依赖项的实例化或位置。

Code is cleaner with the DI principle, and decoupling is more effective when objects are provided with their dependencies. The object does not look up its dependencies and does not know the location or class of the dependencies. As a result, your classes become easier to test, particularly when the dependencies are on interfaces or abstract base classes, which allow for stub or mock implementations to be used in unit tests.
DI原则的代码更简洁，并且当对象提供了依赖项时，解耦更有效。对象不查找其依赖项，也不知道依赖项的位置或类。因此，类变得更易于测试，特别是当依赖项位于接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

DI exists in two major variants: [Constructor-based dependency injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection) and [Setter-based dependency injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-setter-injection).
DI存在两种主要变体：[基于构造函数的依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection)和[基于Setter的依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-setter-injection)。

##### 基于构造函数的依赖注入

Constructor-based DI is accomplished by the container invoking a constructor with a number of arguments, each representing a dependency. Calling a `static` factory method with specific arguments to construct the bean is nearly equivalent, and this discussion treats arguments to a constructor and to a `static` factory method similarly. The following example shows a class that can only be dependency-injected with constructor injection:
基于构造函数的DI是通过容器调用具有多个参数的构造函数来实现的，每个参数表示一个依赖项。这与调用带有特定参数的 `static` 工厂方法来构造bean的做法几乎是等效的，并且本讨论以类似的方式处理构造函数的参数和 `static` 工厂方法的参数。下面的示例显示只能使用构造函数注入进行依赖项注入的类：

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

Notice that there is nothing special about this class. It is a POJO that has no dependencies on container specific interfaces, base classes, or annotations.
请注意，这个类没有什么特别之处。它是一个POJO，不依赖于容器特定的接口、基类或注释。

###### 构造函数参数解析

Constructor argument resolution matching occurs by using the argument’s type. If no potential ambiguity exists in the constructor arguments of a bean definition, the order in which the constructor arguments are defined in a bean definition is the order in which those arguments are supplied to the appropriate constructor when the bean is being instantiated. Consider the following class:
构造函数参数解析匹配通过使用参数的类型进行。如果Bean定义的构造函数参数中不存在潜在的歧义，则在Bean定义中定义构造函数参数的顺序就是实例化Bean时将这些参数提供给相应构造函数的顺序。考虑以下类：

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

Assuming that the `ThingTwo` and `ThingThree` classes are not related by inheritance, no potential ambiguity exists. Thus, the following configuration works fine, and you do not need to specify the constructor argument indexes or types explicitly in the `<constructor-arg/>` element.
假设 `ThingTwo` 和 `ThingThree` 类不通过继承相关，则不存在潜在的模糊性。因此，下面的配置工作正常，您不需要在 `<constructor-arg/>` 元素中显式地指定构造函数参数索引或类型。

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

When another bean is referenced, the type is known, and matching can occur (as was the case with the preceding example). When a simple type is used, such as `<value>true</value>`, Spring cannot determine the type of the value, and so cannot match by type without help. Consider the following class:
当引用另一个bean时，类型是已知的，并且可以进行匹配（与前面的示例相同）。当使用简单类型时，比如 `<value>true</value>` ，Spring不能确定值的类型，因此没有帮助就不能按类型匹配。考虑以下类：

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private final int years;

    // The Answer to Life, the Universe, and Everything
    private final String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

In the preceding scenario, the container can use type matching with simple types if you explicitly specify the type of the constructor argument by using the `type` attribute, as the following example shows:
在前面的方案中，如果使用 `type` 属性显式指定构造函数参数的类型，则容器可以使用与简单类型匹配的类型，如下面的示例所示：

```java
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

You can use the `index` attribute to specify explicitly the index of constructor arguments, as the following example shows:
可以使用 `index` 属性显式指定构造函数参数的索引，如下面的示例所示：

```java
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

In addition to resolving the ambiguity of multiple simple values, specifying an index resolves ambiguity where a constructor has two arguments of the same type.
除了解决多个简单值的多义性外，指定索引还解决构造函数具有两个相同类型的参数时的多义性。

> The index is 0-based. 该索引从0开始。

You can also use the constructor parameter name for value disambiguation, as the following example shows:
也可以使用构造函数参数名称来消除值的歧义，如下例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

Keep in mind that, to make this work out of the box, your code must be compiled with the debug flag enabled so that Spring can look up the parameter name from the constructor. If you cannot or do not want to compile your code with the debug flag, you can use the [@ConstructorProperties](https://download.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html) JDK annotation to explicitly name your constructor arguments. The sample class would then have to look as follows:
请记住，要使其开箱即用，您的代码必须在编译时启用调试标志，以便Spring可以从构造函数中查找参数名。如果无法或不想使用debug标志编译代码，可以使用JDK注解 [@ConstructorProperties](https://download.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html) 显式命名构造函数参数。然后，示例类必须如下所示：

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

##### 基于Setter的依赖注入

Setter-based DI is accomplished by the container calling setter methods on your beans after invoking a no-argument constructor or a no-argument `static` factory method to instantiate your bean.
基于setter的DI是由容器在调用无参数构造函数或无参数 `static` 工厂方法实例化bean之后调用bean上的setter方法来完成的。

The following example shows a class that can only be dependency-injected by using pure setter injection. This class is conventional Java. It is a POJO that has no dependencies on container specific interfaces, base classes, or annotations.
下面的示例演示只能使用纯setter注入进行依赖项注入的类。这个类是传统的Java。它是一个POJO，不依赖于容器特定的接口、基类或注释。

```java
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

The `ApplicationContext` supports constructor-based and setter-based DI for the beans it manages. It also supports setter-based DI after some dependencies have already been injected through the constructor approach. You configure the dependencies in the form of a `BeanDefinition`, which you use in conjunction with `PropertyEditor` instances to convert properties from one format to another. However, most Spring users do not work with these classes directly (that is, programmatically) but rather with XML `bean` definitions, annotated components (that is, classes annotated with `@Component`, `@Controller`, and so forth), or `@Bean` methods in Java-based `@Configuration` classes. These sources are then converted internally into instances of `BeanDefinition` and used to load an entire Spring IoC container instance.
`ApplicationContext` 为它管理的bean支持基于构造函数和基于setter函数的DI。在通过构造函数方法注入了一些依赖项之后，它还支持基于setter的DI。您以 `BeanDefinition` 的形式配置依赖项，它与 `PropertyEditor` 实例一起使用，将属性从一种格式转换为另一种格式。然而，大多数Spring用户并不直接使用这些类（即以编程方式），而是使用XML `bean` 定义、带注释的组件（即用 `@Component` 、 `@Controller` 等注释的类）或基于Java的 `@Configuration` 类中的 `@Bean` 方法。然后，这些源代码在内部转换为 `BeanDefinition` 的实例，并用于加载整个Spring IoC容器实例。

> Constructor-based or setter-based DI?
> 基于构造函数还是基于设置函数的DI？
>
> Since you can mix constructor-based and setter-based DI, it is a good rule of thumb to use constructors for mandatory dependencies and setter methods or configuration methods for optional dependencies. Note that use of the [@Autowired](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation) annotation on a setter method can be used to make the property be a required dependency; however, constructor injection with programmatic validation of arguments is preferable.
> 由于可以混合使用基于构造函数和基于setter的DI，因此一个很好的经验法则是将构造函数用于强制依赖项，将setter方法或配置方法用于可选依赖项。注意，在setter方法上使用 [@Autowired](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation) 注释可以使属性成为必需的依赖项；但是，更推荐使用有Java语法检查的构造函数进行注入。
>
> The Spring team generally advocates constructor injection, as it lets you implement application components as immutable objects and ensures that required dependencies are not `null`. Furthermore, constructor-injected components are always returned to the client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.
> Spring团队通常提倡构造函数注入，因为它允许您将应用程序组件实现为不可变对象，并确保所需的依赖关系不是 `null` 。此外，构造函数注入的组件总是以完全初始化的状态返回到客户端（调用）代码。顺便说一句，大量的构造函数参数是一种糟糕的代码气味，这意味着类可能有太多的责任，应该进行重构，以更好地解决关注点的适当分离。
>
> Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later. Management through [JMX MBeans](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jmx) is therefore a compelling use case for setter injection.
> Setter注入应该主要只用于可以在类中分配合理默认值的可选依赖项。否则，必须在代码使用依赖项的所有地方执行非空检查。setter注入的一个好处是，setter方法使该类的对象可以在以后重新配置或重新注入。因此，通过JMX MBean进行管理是setter注入的一个引人注目的用例。
>
> Use the DI style that makes the most sense for a particular class. Sometimes, when dealing with third-party classes for which you do not have the source, the choice is made for you. For example, if a third-party class does not expose any setter methods, then constructor injection may be the only available form of DI.
> 使用对特定类最有意义的DI样式。有时，当处理您没有源代码的第三方类时，会为您做出选择。例如，如果第三方类不公开任何setter方法，那么构造函数注入可能是DI的唯一可用形式。

##### 依赖关系解决过程

The container performs bean dependency resolution as follows:
容器执行Bean依赖关系解析，如下所示：

- The `ApplicationContext` is created and initialized with configuration metadata that describes all the beans. Configuration metadata can be specified by XML, Java code, or annotations.
  创建 `ApplicationContext` 并使用描述所有bean的配置元数据进行初始化。配置元数据可以由XML、Java代码或注释指定。
- For each bean, its dependencies are expressed in the form of properties, constructor arguments, or arguments to the static-factory method (if you use that instead of a normal constructor). These dependencies are provided to the bean, when the bean is actually created.
  对于每个bean，它的依赖关系以属性、构造函数参数或静态工厂方法的参数（如果使用静态工厂方法而不是普通构造函数）的形式表示。在实际创建bean时，这些依赖项将提供给bean。
- Each property or constructor argument is an actual definition of the value to set, or a reference to another bean in the container.
  每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个bean的引用。
- Each property or constructor argument that is a value is converted from its specified format to the actual type of that property or constructor argument. By default, Spring can convert a value supplied in string format to all built-in types, such as `int`, `long`, `String`, `boolean`, and so forth.
  每个值属性或构造函数参数都从其指定格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring可以将以字符串格式提供的值转换为所有内置类型，如 `int` 、 `long` 、 `String` 、 `boolean` 等。

The Spring container validates the configuration of each bean as the container is created. However, the bean properties themselves are not set until the bean is actually created. Beans that are singleton-scoped and set to be pre-instantiated (the default) are created when the container is created. Scopes are defined in [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes). Otherwise, the bean is created only when it is requested. Creation of a bean potentially causes a graph of beans to be created, as the bean’s dependencies and its dependencies' dependencies (and so on) are created and assigned. Note that resolution mismatches among those dependencies may show up late — that is, on first creation of the affected bean.
Spring容器在创建容器时验证每个bean的配置。但是，直到实际创建了bean之后，才设置bean属性本身。在创建容器时，将创建单例作用域并设置为预实例化（缺省）的Bean。作用域在[Bean作用域](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes)中定义。否则，仅在请求Bean时才创建它。bean的创建可能会导致创建bean图，因为bean的依赖项及其依赖项的依赖项（等等）被创建和分配。请注意，这些依赖项之间的解析不匹配可能会在较晚的时候出现——也就是说，在受影响的bean的第一次创建时。

> Circular dependencies 循环依赖
>
> If you use predominantly constructor injection, it is possible to create an unresolvable circular dependency scenario.
> 如果您主要使用构造函数注入，则有可能创建无法解析的循环依赖场景。
>
> For example: Class A requires an instance of class B through constructor injection, and class B requires an instance of class A through constructor injection. If you configure beans for classes A and B to be injected into each other, the Spring IoC container detects this circular reference at runtime, and throws a `BeanCurrentlyInCreationException`.
> 例如：类A通过构造函数注入需要类B的实例，类B通过构造函数注入需要类A的实例。如果您将类A和B的bean配置为相互注入，那么Spring IoC容器会在运行时检测到这种循环引用，并抛出 `BeanCurrentlyInCreationException` 。
>
> One possible solution is to edit the source code of some classes to be configured by setters rather than constructors. Alternatively, avoid constructor injection and use setter injection only. In other words, although it is not recommended, you can configure circular dependencies with setter injection.
> 一个可能的解决方案是编辑某些类的源代码，使其由setter而不是构造函数来配置。或者，避免构造函数注入，只使用setter注入。换句话说，尽管不推荐，但您可以使用setter注入配置循环依赖项。
>
> Unlike the typical case (with no circular dependencies), a circular dependency between bean A and bean B forces one of the beans to be injected into the other prior to being fully initialized itself (a classic chicken-and-egg scenario).
> 与典型情况（没有循环依赖关系）不同，bean A和bean B之间的循环依赖关系强制其中一个bean在完全初始化自身之前注入到另一个bean中（典型的先有鸡还是先有蛋的场景）。

You can generally trust Spring to do the right thing. It detects configuration problems, such as references to non-existent beans and circular dependencies, at container load-time. Spring sets properties and resolves dependencies as late as possible, when the bean is actually created. This means that a Spring container that has loaded correctly can later generate an exception when you request an object if there is a problem creating that object or one of its dependencies — for example, the bean throws an exception as a result of a missing or invalid property. This potentially delayed visibility of some configuration issues is why `ApplicationContext` implementations by default pre-instantiate singleton beans. At the cost of some upfront time and memory to create these beans before they are actually needed, you discover configuration issues when the `ApplicationContext` is created, not later. You can still override this default behavior so that singleton beans initialize lazily, rather than being eagerly pre-instantiated.
您通常可以相信Spring会做正确的事情。它在容器加载时检测配置问题，例如引用不存在的bean和循环依赖关系。Spring设置属性和解析依赖关系的时间越晚越好，直到bean实际创建的时候。这意味着，如果创建对象或其依赖项之一时出现问题（例如，Bean由于缺少属性或属性无效而引发异常），则正确加载的Spring容器以后可能会在请求对象时生成异常。这种潜在的延迟的配置问题可见性就是 `ApplicationContext` 实现默认预实例化单例bean的原因。在实际需要这些bean之前，您需要花费一些前期时间和内存来创建它们，因此您在创建 `ApplicationContext` 时就发现了配置问题，而不是之后。 当然，您仍然可以覆盖此默认行为，以便单例bean可以延迟初始化，而不是急切地进行预实例化。

If no circular dependencies exist, when one or more collaborating beans are being injected into a dependent bean, each collaborating bean is totally configured prior to being injected into the dependent bean. This means that, if bean A has a dependency on bean B, the Spring IoC container completely configures bean B prior to invoking the setter method on bean A. In other words, the bean is instantiated (if it is not a pre-instantiated singleton), its dependencies are set, and the relevant lifecycle methods (such as a [configured init method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) or the [InitializingBean callback method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean)) are invoked.
如果不存在循环依赖关系，则当一个或多个协作bean被注入到依赖bean中时，每个协作bean在被注入到依赖bean之前被完全配置。这意味着，如果bean A依赖于bean B，那么Spring IoC容器在调用bean A上的setter方法之前会完全配置bean B。换句话说，bean被实例化（如果它不是一个已经预先实例化了的单例），它的依赖关系被设置，相关的生命周期方法（如配置的init方法或InitializingBean回调方法）被调用。

##### 依赖项注入的示例

The following example uses XML-based configuration metadata for setter-based DI. A small part of a Spring XML configuration file specifies some bean definitions as follows:
下面的示例将基于XML的配置元数据用于基于setter的DI。Spring XML配置文件的一小部分指定了一些bean定义，如下所示：

```xml
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

The following example shows the corresponding `ExampleBean` class:
下面的示例显示相应的 `ExampleBean` 类：

```java
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

In the preceding example, setters are declared to match against the properties specified in the XML file. The following example uses constructor-based DI:
在前面的示例中，声明setter以匹配XML文件中指定的属性。下面的示例使用基于构造函数的DI：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

The following example shows the corresponding `ExampleBean` class:
下面的示例显示相应的 `ExampleBean` 类：

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

The constructor arguments specified in the bean definition are used as arguments to the constructor of the `ExampleBean`.
bean定义中指定的构造函数参数用作 `ExampleBean` 的构造函数的参数。

Now consider a variant of this example, where, instead of using a constructor, Spring is told to call a `static` factory method to return an instance of the object:
现在考虑这个例子的一个变体，其中，Spring被告知调用 `static` 工厂方法来返回对象的实例，而不是使用构造函数：

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

The following example shows the corresponding `ExampleBean` class:
下面的示例显示相应的 `ExampleBean` 类：

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

Arguments to the `static` factory method are supplied by `<constructor-arg/>` elements, exactly the same as if a constructor had actually been used. The type of the class being returned by the factory method does not have to be of the same type as the class that contains the `static` factory method (although, in this example, it is). An instance (non-static) factory method can be used in an essentially identical fashion (aside from the use of the `factory-bean` attribute instead of the `class` attribute), so we do not discuss those details here.
`static` 工厂方法的参数由 `<constructor-arg/>` 元素提供，就像实际使用了构造函数一样。由工厂方法返回的类的类型不必与包含 `static` 工厂方法的类的类型相同（尽管在本例中是相同的）。实例（非静态）工厂方法可以以本质上相同的方式使用（除了使用 `factory-bean` 属性而不是 `class` 属性之外），因此我们不在这里讨论这些细节。

#### 1.4.2. 依赖项和配置的详细信息

As mentioned in the [previous section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators), you can define bean properties and constructor arguments as references to other managed beans (collaborators) or as values defined inline. Spring’s XML-based configuration metadata supports sub-element types within its `<property/>` and `<constructor-arg/>` elements for this purpose.
如前一节所述，您可以将bean属性和构造函数参数定义为对其他受管bean（协作者）的引用或内联定义的值。Spring基于XML的配置元数据支持在 `<property/>` 和 `<constructor-arg/>` 元素中使用子元素类型来实现这一目的。

##### 直接值（基元、字符串等）

The `value` attribute of the `<property/>` element specifies a property or constructor argument as a human-readable string representation. Spring’s [conversion service](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core-convert-ConversionService-API) is used to convert these values from a `String` to the actual type of the property or argument. The following example shows various values being set:
`<property/>` 元素的 `value` 属性将属性或构造函数参数指定为人类可读的字符串表示形式。Spring的转换服务用于将这些值从 `String` 转换为属性或参数的实际类型。以下示例显示了正在设置的各种值：

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```

The following example uses the [p-namespace](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-p-namespace) for even more succinct XML configuration:
以下示例使用[p名称空间](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-p-namespace)进行更简洁的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="misterkaoli"/>

</beans>
```

The preceding XML is more succinct. However, typos are discovered at runtime rather than design time, unless you use an IDE (such as [IntelliJ IDEA](https://www.jetbrains.com/idea/) or the [Spring Tools for Eclipse](https://spring.io/tools)) that supports automatic property completion when you create bean definitions. Such IDE assistance is highly recommended.
前面的XML更加简洁。但是，拼写错误是在运行时而不是设计时发现的，除非您使用的IDE（如IntelliJ IDEA或Spring Tools for Eclipse）支持在创建bean定义时自动完成属性。强烈建议提供此类IDE协助。

You can also configure a `java.util.Properties` instance, as follows:
您还可以配置 `java.util.Properties` 实例，如下所示：

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

The Spring container converts the text inside the `<value/>` element into a `java.util.Properties` instance by using the JavaBeans `PropertyEditor` mechanism. This is a nice shortcut, and is one of a few places where the Spring team do favor the use of the nested `<value/>` element over the `value` attribute style.
Spring容器使用JavaBeans `PropertyEditor` 机制将 `<value/>` 元素中的文本转换为 `java.util.Properties` 实例。这是一个很好的捷径，也是Spring团队倾向于使用嵌套的 `<value/>` 元素而不是 `value` 属性样式的少数地方之一。

######  `idref` 元素

The `idref` element is simply an error-proof way to pass the `id` (a string value - not a reference) of another bean in the container to a `<constructor-arg/>` or `<property/>` element. The following example shows how to use it:
`idref` 元素只是一种防错的方式，用于将容器中另一个bean的 `id` （字符串值——不是引用）传递给 `<constructor-arg/>` 或 `<property/>` 元素。下面的示例说明如何使用它：

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

The preceding bean definition snippet is exactly equivalent (at runtime) to the following snippet:
前面的bean定义代码段（在运行时）与以下代码段完全等效：

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

The first form is preferable to the second, because using the `idref` tag lets the container validate at deployment time that the referenced, named bean actually exists. In the second variation, no validation is performed on the value that is passed to the `targetName` property of the `client` bean. Typos are only discovered (with most likely fatal results) when the `client` bean is actually instantiated. If the `client` bean is a [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) bean, this typo and the resulting exception may only be discovered long after the container is deployed.
第一种形式比第二种形式更可取，因为使用 `idref` 标记可以让容器在部署时验证引用的命名bean是否实际存在。在第二个变体中，不对传递给 `client` bean的 `targetName` 属性的值执行验证。只有在实际实例化 `client` bean时才会发现拼写错误（最有可能导致致命的结果）。如果 `client` bean是一个原型bean，那么这个拼写错误和由此产生的异常可能只有在部署容器很久之后才能被发现。

> The `local` attribute on the `idref` element is no longer supported in the 4.0 beans XSD, since it does not provide value over a regular `bean` reference any more. Change your existing `idref local` references to `idref bean` when upgrading to the 4.0 schema.
> 4.0 bean XSD不再支持 `idref` 元素上的 `local` 属性，因为它不再通过常规的 `bean` 引用提供值。升级到4.0模式时，将现有的 `idref local` 引用更改为 `idref bean` 。

A common place (at least in versions earlier than Spring 2.0) where the `<idref/>` element brings value is in the configuration of [AOP interceptors](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-pfb-1) in a `ProxyFactoryBean` bean definition. Using `<idref/>` elements when you specify the interceptor names prevents you from misspelling an interceptor ID.
`<idref/>` 元素带来价值的一个常见地方（至少在Spring 2.0之前的版本中）是 `ProxyFactoryBean` bean定义中的[AOP拦截器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-pfb-1)配置。在指定拦截器名称时使用 `<idref/>` 元素可以防止拼写错误拦截器ID。

##### 对其他Bean的引用（协作者）

The `ref` element is the final element inside a `<constructor-arg/>` or `<property/>` definition element. Here, you set the value of the specified property of a bean to be a reference to another bean (a collaborator) managed by the container. The referenced bean is a dependency of the bean whose property is to be set, and it is initialized on demand as needed before the property is set. (If the collaborator is a singleton bean, it may already be initialized by the container.) All references are ultimately a reference to another object. Scoping and validation depend on whether you specify the ID or name of the other object through the `bean` or `parent` attribute.
`ref` 元素是 `<constructor-arg/>` 或 `<property/>` 定义元素中的最后一个元素。这里，您将bean的指定属性的值设置为对容器管理的另一个bean（协作者）的引用。被引用的bean是要设置其属性的bean的依赖项，并且在设置属性之前根据需要初始化它。(如果协作者是一个单例bean，它可能已经被容器初始化了。）所有引用最终都是对另一个对象的引用。作用域和验证取决于您是通过 `bean` 还是 `parent` 属性指定其他对象的ID或名称。

Specifying the target bean through the `bean` attribute of the `<ref/>` tag is the most general form and allows creation of a reference to any bean in the same container or parent container, regardless of whether it is in the same XML file. The value of the `bean` attribute may be the same as the `id` attribute of the target bean or be the same as one of the values in the `name` attribute of the target bean. The following example shows how to use a `ref` element:
通过 `<ref/>` 标记的 `bean` 属性指定目标bean是最常见的形式，它允许创建对同一容器或父容器中任何bean的引用，而不管它是否在同一XML文件中。 `bean` 属性的值可以与目标bean的 `id` 属性的值相同，也可以与目标bean的 `name` 属性的值之一相同。下面的示例说明如何使用 `ref` 元素：

```xml
<ref bean="someBean"/>
```

Specifying the target bean through the `parent` attribute creates a reference to a bean that is in a parent container of the current container. The value of the `parent` attribute may be the same as either the `id` attribute of the target bean or one of the values in the `name` attribute of the target bean. The target bean must be in a parent container of the current one. You should use this bean reference variant mainly when you have a hierarchy of containers and you want to wrap an existing bean in a parent container with a proxy that has the same name as the parent bean. The following pair of listings shows how to use the `parent` attribute:
通过 `parent` 属性指定目标bean将创建对当前容器的父容器中的bean的引用。 `parent` 属性的值可以与目标bean的 `id` 属性相同，也可以与目标bean的 `name` 属性中的某个值相同。目标Bean必须位于当前Bean的父容器中。当您具有容器层次结构，并且希望使用与父容器同名的代理将现有Bean包装在父容器中时，应主要使用此Bean引用变体。以下两个清单显示了如何使用 `parent` 属性：

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

> The `local` attribute on the `ref` element is no longer supported in the 4.0 beans XSD, since it does not provide value over a regular `bean` reference any more. Change your existing `ref local` references to `ref bean` when upgrading to the 4.0 schema.
> 4.0 bean XSD不再支持 `ref` 元素上的 `local` 属性，因为它不再通过常规的 `bean` 引用提供值。升级到4.0模式时，将现有的 `ref local` 引用更改为 `ref bean` 。

##### 内嵌 Bean

A `<bean/>` element inside the `<property/>` or `<constructor-arg/>` elements defines an inner bean, as the following example shows:
`<property/>` 或 `<constructor-arg/>` 元素内的 `<bean/>` 元素定义内部Bean，如下例所示：

```xml
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

An inner bean definition does not require a defined ID or name. If specified, the container does not use such a value as an identifier. The container also ignores the `scope` flag on creation, because inner beans are always anonymous and are always created with the outer bean. It is not possible to access inner beans independently or to inject them into collaborating beans other than into the enclosing bean.
内部Bean定义不需要定义的ID或名称。如果指定，则容器不使用这样的值作为标识符。容器在创建时也会忽略 `scope` 标志，因为内部bean总是匿名的，并且总是与外部bean一起创建。不可能独立访问内部bean，也不可能将它们注入到协作bean中，只能注入到封闭bean中。

As a corner case, it is possible to receive destruction callbacks from a custom scope — for example, for a request-scoped inner bean contained within a singleton bean. The creation of the inner bean instance is tied to its containing bean, but destruction callbacks let it participate in the request scope’s lifecycle. This is not a common scenario. Inner beans typically simply share their containing bean’s scope.
作为一种极端情况，可以从自定义范围接收析构回调——例如，对于单例bean中包含的请求范围的内部bean。内部bean实例的创建与它的包含bean绑定，但是析构回调允许它参与请求范围的生命周期。这种情况并不常见。内部bean通常只是共享其包含bean的作用域。

##### Collections

The <list/>, <set/>, <map/>, and <props/> elements set the properties and arguments of the Java Collection types List, Set, Map, and Properties, respectively. The following example shows how to use <list/> 、 <set/> 、 <map/> 和 <props/> 元素分别设置Java Collection 类型 List 、 Set 、 Map 和 Properties 的属性和参数。下面的示例说明如何使用它们：

```xml
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
            <entry key="a ref" value-ref="myDataSource"/>
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

The value of a map key or value, or a set value, can also be any of the following elements:
映射键或值的值或集合值也可以是以下任何元素：

```xml
bean | ref | idref | list | set | map | props | value | null
```

###### 集合合并

The Spring container also supports merging collections. An application developer can define a parent <list/>, <map/>, <set/> or <props/> element and have child <list/>, <map/>, <set/> or <props/> elements inherit and override values from the parent collection. That is, the child collection’s values are the result of merging the elements of the parent and child collections, with the child’s collection elements overriding values specified in the parent collection.Spring容器还支持合并集合。应用程序开发者可以定义父 <list/> 、 <map/> 、 <set/> 或 <props/> 元素，并使子 <list/> 、 <map/> 、 <set/> 或 <props/> 元素继承和覆盖来自父集合的值。也就是说，子集合的值是父集合和子集合的元素合并的结果，其中子集合的元素重写父集合中指定的值。

This section on merging discusses the parent-child bean mechanism. Readers unfamiliar with parent and child bean definitions may wish to read the [relevant section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions) before continuing.
关于合并的这一节讨论父——子bean机制。不熟悉父bean和子bean定义的读者可能希望在继续之前阅读[相关部分](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions)。

The following example demonstrates collection merging:
下面的示例演示集合合并：

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

Notice the use of the `merge=true` attribute on the `<props/>` element of the `adminEmails` property of the `child` bean definition. When the `child` bean is resolved and instantiated by the container, the resulting instance has an `adminEmails` `Properties` collection that contains the result of merging the child’s `adminEmails` collection with the parent’s `adminEmails` collection. The following listing shows the result:
注意 `child` bean定义的 `adminEmails` 属性的 `<props/>` 元素上的 `merge=true` 属性的使用。当容器解析并实例化 `child` bean时，得到的实例有一个 `adminEmails` `Properties` 集合，其中包含子对象的 `adminEmails` 集合与父对象的 `adminEmails` 集合合并的结果。以下清单显示了结果：

```xml
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

The child `Properties` collection’s value set inherits all property elements from the parent `<props/>`, and the child’s value for the `support` value overrides the value in the parent collection.
子级 `Properties` 集合的值集继承父级 `<props/>` 的所有属性元素，并且子级的 `support` 值将覆盖父级集合中的值。

This merging behavior applies similarly to the <list/>, <map/>, and <set/> collection types. In the specific case of the <list/> element, the semantics associated with the List collection type (that is, the notion of an ordered collection of values) is maintained. The parent’s values precede all of the child list’s values. In the case of the Map, Set, and Properties collection types, no ordering exists. Hence, no ordering semantics are in effect for the collection types that underlie the associated Map, Set, and Properties implementation types that the container uses internally.
此合并行为同样适用于 <list/> 、 <map/> 和 <set/> 集合类型。在 <list/> 元素的特定情况下，保持与 List 集合类型相关联的语义（即，值的 ordered 集合的概念）。父列表的值位于所有子列表的值之前。对于 Map 、 Set 和 Properties 集合类型，不存在排序。因此，对于容器内部使用的相关 Map 、 Set 和 Properties 实现类型所基于的集合类型，排序语义是无效的。

###### 集合合并的限制

You cannot merge different collection types (such as a `Map` and a `List`). If you do attempt to do so, an appropriate `Exception` is thrown. The `merge` attribute must be specified on the lower, inherited, child definition. Specifying the `merge` attribute on a parent collection definition is redundant and does not result in the desired merging.
不能合并不同的集合类型（如 `Map` 和 `List` ）。如果您尝试这样做，则会引发相应的 `Exception` 。必须在较低的继承子定义上指定 `merge` 属性。在父集合定义上指定 `merge` 属性是多余的，并且不会产生所需的合并。

###### 强类型集合

Thanks to Java’s support for generic types, you can use strongly typed collections. That is, it is possible to declare a `Collection` type such that it can only contain (for example) `String` elements. If you use Spring to dependency-inject a strongly-typed `Collection` into a bean, you can take advantage of Spring’s type-conversion support such that the elements of your strongly-typed `Collection` instances are converted to the appropriate type prior to being added to the `Collection`. The following Java class and bean definition show how to do so:
由于Java支持泛型类型，因此可以使用强类型集合。也就是说，可以声明 `Collection` 类型，使得它只能包含（例如） `String` 元素。如果您使用Spring将强类型 `Collection` 依赖注入到bean中，那么您可以利用Spring的类型转换支持，以便在将强类型 `Collection` 实例的元素添加到 `Collection` 之前将其转换为适当的类型。以下Java类和Bean定义说明了如何执行此操作：

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

When the `accounts` property of the `something` bean is prepared for injection, the generics information about the element type of the strongly-typed `Map<String, Float>` is available by reflection. Thus, Spring’s type conversion infrastructure recognizes the various value elements as being of type `Float`, and the string values (`9.99`, `2.75`, and `3.99`) are converted into an actual `Float` type.
当 `something` bean的 `accounts` 属性准备好注入时，关于强类型 `Map<String, Float>` 的元素类型的泛型信息可以通过反射获得。因此，Spring的类型转换基础设施将各种value元素识别为 `Float` 类型，字符串值（ `9.99` 、 `2.75` 和 `3.99` ）被转换为实际的 `Float` 类型。

##### Null和空字符串值

Spring treats empty arguments for properties and the like as empty `Strings`. The following XML-based configuration metadata snippet sets the `email` property to the empty `String` value ("").
Spring将属性等的空参数视为空的 `Strings` 。以下基于XML的配置元数据代码段将 `email` 属性设置为空的 `String` 值（“”）。

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

The preceding example is equivalent to the following Java code:
上面的示例等效于以下Java代码：

```java
exampleBean.setEmail("");
```

The `<null/>` element handles `null` values. The following listing shows an example:
`<null/>` 元素处理 `null` 值。下面的清单显示了一个示例：

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

The preceding configuration is equivalent to the following Java code:
上述配置等效于以下Java代码：

```java
exampleBean.setEmail(null);
```

##### 具有p名称空间的XML快捷方式

The p-namespace lets you use the `bean` element’s attributes (instead of nested `<property/>` elements) to describe your property values collaborating beans, or both.
p-namespace允许您使用 `bean` 元素的属性（而不是嵌套的 `<property/>` 元素）来描述协作bean的属性值，或者两者都使用。

Spring supports extensible configuration formats [with namespaces](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core.appendix.xsd-schemas), which are based on an XML Schema definition. The `beans` configuration format discussed in this chapter is defined in an XML Schema document. However, the p-namespace is not defined in an XSD file and exists only in the core of Spring.
Spring支持带有名称空间的可扩展配置格式，这些格式基于XMLSchema定义。本章讨论的 `beans` 配置格式在XML模式文档中定义。但是，p名称空间不是在XSD文件中定义的，它只存在于Spring的核心中。

The following example shows two XML snippets (the first uses standard XML format and the second uses the p-namespace) that resolve to the same result:
以下示例显示解析为相同结果的两个XML代码段（第一个使用标准XML格式，第二个使用p命名空间）：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

The example shows an attribute in the p-namespace called `email` in the bean definition. This tells Spring to include a property declaration. As previously mentioned, the p-namespace does not have a schema definition, so you can set the name of the attribute to the property name.
该示例显示了bean定义中名为 `email` 的p名称空间中的一个属性。这将告诉Spring包含一个属性声明。如前所述，p名称空间没有模式定义，因此可以将属性的名称设置为字段名。

This next example includes two more bean definitions that both have a reference to another bean:
下一个示例包括另外两个bean定义，它们都引用了另一个bean：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

This example includes not only a property value using the p-namespace but also uses a special format to declare property references. Whereas the first bean definition uses `<property name="spouse" ref="jane"/>` to create a reference from bean `john` to bean `jane`, the second bean definition uses `p:spouse-ref="jane"` as an attribute to do the exact same thing. In this case, `spouse` is the property name, whereas the `-ref` part indicates that this is not a straight value but rather a reference to another bean.
这个例子不仅包含了一个使用p命名空间的属性值，而且还使用了一种特殊的格式来声明属性引用。第一个bean定义使用 `<property name="spouse" ref="jane"/>` 来创建从bean `john` 到bean `jane` 的引用，而第二个bean定义使用 `p:spouse-ref="jane"` 作为属性来做完全相同的事情。在本例中， `spouse` 是属性名，而 `-ref` 部分表示这不是一个直接的值，而是对另一个bean的引用。

> The p-namespace is not as flexible as the standard XML format. For example, the format for declaring property references clashes with properties that end in `Ref`, whereas the standard XML format does not. We recommend that you choose your approach carefully and communicate this to your team members to avoid producing XML documents that use all three approaches at the same time.
> p名称空间不如标准XML格式灵活。例如，声明属性引用的格式与以 `Ref` 结尾的属性冲突，而标准XML格式则不会。我们建议您仔细选择方法，并与团队成员进行沟通，以避免生成同时使用所有三种方法的XML文档。

##### 具有c命名空间的XML快捷方式

Similar to the [XML Shortcut with the p-namespace](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-p-namespace), the c-namespace, introduced in Spring 3.1, allows inlined attributes for configuring the constructor arguments rather then nested `constructor-arg` elements.
与带有p名称空间的XML Shortcut类似，Spring 3.1中引入的c名称空间允许内联属性来配置构造函数参数，而不是嵌套 `constructor-arg` 元素。

The following example uses the `c:` namespace to do the same thing as the from [Constructor-based Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection):
下面的示例使用 `c:` 命名空间执行与[基于构造函数的依赖项注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection)相同的操作：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

The `c:` namespace uses the same conventions as the `p:` one (a trailing `-ref` for bean references) for setting the constructor arguments by their names. Similarly, it needs to be declared in the XML file even though it is not defined in an XSD schema (it exists inside the Spring core).
`c:` 名称空间使用与 `p:` 名称空间相同的约定（bean引用的尾部 `-ref` ），通过名称设置构造函数参数。类似地，它需要在XML文件中声明，即使它没有在XSD模式中定义（它存在于Spring核心中）。

For the rare cases where the constructor argument names are not available (usually if the bytecode was compiled without debugging information), you can use fallback to the argument indexes, as follows:
对于构造函数参数名不可用的极少数情况（通常是在没有调试信息的情况下编译字节码），可以使用回退到参数索引，如下所示：

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

> Due to the XML grammar, the index notation requires the presence of the leading `_`, as XML attribute names cannot start with a number (even though some IDEs allow it). A corresponding index notation is also available for `<constructor-arg>` elements but not commonly used since the plain order of declaration is usually sufficient there.
> 由于XML语法的原因，索引表示法需要以 `_` 开头，因为XML属性名不能以数字开头（尽管有些IDE允许这样做）。对应的索引符号也可用于 `<constructor-arg>` 元素，但不常用，因为在那里声明的简单顺序通常就足够了。

In practice, the constructor resolution [mechanism](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-ctor-arguments-resolution) is quite efficient in matching arguments, so unless you really need to, we recommend using the name notation throughout your configuration.
在实践中，构造函数解析[机制](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-ctor-arguments-resolution)在匹配参数方面非常有效，因此除非确实需要，否则我们建议在整个配置中使用名称表示法。

##### 复合属性名称

You can use compound or nested property names when you set bean properties, as long as all components of the path except the final property name are not `null`. Consider the following bean definition:
设置Bean属性时，可以使用复合或嵌套属性名，只要路径中除最后属性名之外的所有组成部分不是 `null` 即可。请考虑以下Bean定义：

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

The `something` bean has a `fred` property, which has a `bob` property, which has a `sammy` property, and that final `sammy` property is being set to a value of `123`. In order for this to work, the `fred` property of `something` and the `bob` property of `fred` must not be `null` after the bean is constructed. Otherwise, a `NullPointerException` is thrown.
`something` bean具有 `fred` 属性， `fred` 属性具有 `bob` 属性， `sammy` 属性具有 `sammy` 属性，并且最后的 `sammy` 属性被设置为 `123` 的值。为了使其正常工作，在构造bean之后， `something` 的 `fred` 属性和 `fred` 的 `bob` 属性不能为 `null` 。否则，将引发 `NullPointerException` 。

#### 1.4.3. 使用 `depends-on`

If a bean is a dependency of another bean, that usually means that one bean is set as a property of another. Typically you accomplish this with the [` element`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-ref-element) in XML-based configuration metadata. However, sometimes dependencies between beans are less direct. An example is when a static initializer in a class needs to be triggered, such as for database driver registration. The `depends-on` attribute can explicitly force one or more beans to be initialized before the bean using this element is initialized. The following example uses the `depends-on` attribute to express a dependency on a single bean:
如果一个bean是另一个bean的依赖项，这通常意味着一个bean被设置为另一个bean的属性。通常，您可以使用基于XML的配置元数据中的 `<ref/>` 元素来完成此操作。但是，有时bean之间的依赖关系不那么直接。一个例子是当类中的静态初始化器需要被触发时，比如数据库驱动程序注册。 `depends-on` 属性可以显式地强制一个或多个bean在初始化使用该元素的bean之前进行初始化。以下示例使用 `depends-on` 属性表示对单个Bean的依赖关系：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

To express a dependency on multiple beans, supply a list of bean names as the value of the `depends-on` attribute (commas, whitespace, and semicolons are valid delimiters):
要表示对多个bean的依赖关系，请提供bean名称列表作为 `depends-on` 属性的值（逗号、空格和分号是有效的分隔符）：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

> The `depends-on` attribute can specify both an initialization-time dependency and, in the case of [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) beans only, a corresponding destruction-time dependency. Dependent beans that define a `depends-on` relationship with a given bean are destroyed first, prior to the given bean itself being destroyed. Thus, `depends-on` can also control shutdown order.
> `depends-on` 属性既可以指定初始化时依赖关系，也可以指定相应的析构时依赖关系（仅限单例bean）。定义了与给定bean的 `depends-on` 关系的依赖bean在给定bean本身被销毁之前首先被销毁。因此， `depends-on` 也可以控制停机顺序。

#### 1.4.4. 延迟初始化Bean

By default, `ApplicationContext` implementations eagerly create and configure all [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) beans as part of the initialization process. Generally, this pre-instantiation is desirable, because errors in the configuration or surrounding environment are discovered immediately, as opposed to hours or even days later. When this behavior is not desirable, you can prevent pre-instantiation of a singleton bean by marking the bean definition as being lazy-initialized. A lazy-initialized bean tells the IoC container to create a bean instance when it is first requested, rather than at startup.
默认情况下， `ApplicationContext` 实现会急切地创建和配置所有单例bean，作为初始化过程的一部分。通常，这种预先实例化是期望的，因为配置或周围环境中的错误被立即发现，而不是在几小时甚至几天之后。如果不希望出现这种行为，可以通过将bean定义标记为延迟初始化来防止单例bean的预实例化。延迟初始化的bean告诉IoC容器在第一次请求bean实例时创建bean实例，而不是在启动时创建。

In XML, this behavior is controlled by the `lazy-init` attribute on the `<bean/>` element, as the following example shows:
在XML中，此行为由 `<bean/>` 元素上的 `lazy-init` 属性控制，如下例所示：

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

When the preceding configuration is consumed by an `ApplicationContext`, the `lazy` bean is not eagerly pre-instantiated when the `ApplicationContext` starts, whereas the `not.lazy` bean is eagerly pre-instantiated.
当前面的配置被 `ApplicationContext` 使用时， `lazy` bean在 `ApplicationContext` 启动时不会被急切地预实例化，而 `not.lazy` bean会被急切地预实例化。

However, when a lazy-initialized bean is a dependency of a singleton bean that is not lazy-initialized, the `ApplicationContext` creates the lazy-initialized bean at startup, because it must satisfy the singleton’s dependencies. The lazy-initialized bean is injected into a singleton bean elsewhere that is not lazy-initialized.
但是，当延迟初始化的bean是未延迟初始化的单例bean的依赖项时， `ApplicationContext` 会在启动时创建延迟初始化的bean，因为它必须满足单例bean的依赖项。延迟初始化的bean被注入到没有延迟初始化的单例bean中。

You can also control lazy-initialization at the container level by using the `default-lazy-init` attribute on the `<beans/>` element, as the following example shows:
您还可以使用 `<beans/>` 元素上的 `default-lazy-init` 属性在容器级别控制延迟初始化，如下例所示：

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

#### 1.4.5. 自动装配协作者

The Spring container can autowire relationships between collaborating beans. You can let Spring resolve collaborators (other beans) automatically for your bean by inspecting the contents of the `ApplicationContext`. Autowiring has the following advantages:
Spring容器可以自动连接协作bean之间的关系。通过检查 `ApplicationContext` 的内容，您可以让Spring自动为您的bean解析协作者（其他bean）。自动装配具有以下优点：

- Autowiring can significantly reduce the need to specify properties or constructor arguments. (Other mechanisms such as a bean template [discussed elsewhere in this chapter](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions) are also valuable in this regard.)
  自动装配可以显著减少指定属性或构造函数参数的需要。（其他机制，如本章其他地方讨论的bean模板，在这方面也很有价值。）
- Autowiring can update a configuration as your objects evolve. For example, if you need to add a dependency to a class, that dependency can be satisfied automatically without you needing to modify the configuration. Thus autowiring can be especially useful during development, without negating the option of switching to explicit wiring when the code base becomes more stable.
  自动装配可以随着对象的发展更新配置。例如，如果需要向类添加依赖项，则无需修改配置即可自动满足该依赖项。因此，自动装配在开发过程中特别有用，当代码库变得更加稳定时，不会否定切换到显式连接的选择。

When using XML-based configuration metadata (see [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators)), you can specify the autowire mode for a bean definition with the `autowire` attribute of the `<bean/>` element. The autowiring functionality has four modes. You specify autowiring per bean and can thus choose which ones to autowire. The following table describes the four autowiring modes:
使用基于XML的配置元数据时（请参见[依赖注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators)），可以使用 `<bean/>` 元素的 `autowire` 属性为bean定义指定自动连接模式。自动装配功能有四种方式。您可以为每个bean指定自动装配，这样就可以选择要自动装配的bean。下表介绍了四种自动装配模式：

| Mode          | Explanation                                                  |
| :------------ | :----------------------------------------------------------- |
| `no`          | (Default) No autowiring. Bean references must be defined by `ref` elements. Changing the default setting is not recommended for larger deployments, because specifying collaborators explicitly gives greater control and clarity. To some extent, it documents the structure of a system. （默认）不自动装配。Bean引用必须由 `ref` 元素定义。对于较大的部署，不建议更改默认设置，因为显式指定协作者可以提供更好的控制和清晰度。在某种程度上，它记录了系统的结构。 |
| `byName`      | Autowiring by property name. Spring looks for a bean with the same name as the property that needs to be autowired. For example, if a bean definition is set to autowire by name and it contains a `master` property (that is, it has a `setMaster(..)` method), Spring looks for a bean definition named `master` and uses it to set the property. 按属性名称自动装入。Spring查找与需要自动连接的属性同名的bean。例如，如果bean定义被设置为按名称自动连接，并且它包含 `master` 属性（即，它有 `setMaster(..)` 方法），Spring将查找名为 `master` 的bean定义并使用它来设置属性。 |
| `byType`      | Lets a property be autowired if exactly one bean of the property type exists in the container. If more than one exists, a fatal exception is thrown, which indicates that you may not use `byType` autowiring for that bean. If there are no matching beans, nothing happens (the property is not set). 如果容器中恰好存在一个属性类型的bean，则允许自动连接属性。如果存在多个bean，就会抛出一个致命异常，这表明您不能对该bean使用 `byType` 自动装配。如果没有匹配的bean，则不执行任何操作（不设置属性）。 |
| `constructor` | Analogous to `byType` but applies to constructor arguments. If there is not exactly one bean of the constructor argument type in the container, a fatal error is raised. 类似于 `byType` ，但适用于构造函数参数。如果容器中不正好有一个构造函数参数类型的bean，则会引发致命错误。 |

With `byType` or `constructor` autowiring mode, you can wire arrays and typed collections. In such cases, all autowire candidates within the container that match the expected type are provided to satisfy the dependency. You can autowire strongly-typed `Map` instances if the expected key type is `String`. An autowired `Map` instance’s values consist of all bean instances that match the expected type, and the `Map` instance’s keys contain the corresponding bean names.
使用 `byType` 或 `constructor` 自动装配模式，可以装配数组和类型化集合。在这种情况下，将提供容器中与预期类型匹配的所有自动装配候选项以满足依赖项。如果期望的键类型为 `String` ，则可以自动装配强类型 `Map` 实例。自动装配的 `Map` 实例的值由所有与预期类型匹配的bean实例组成， `Map` 实例的键包含相应的bean名称。

##### 自动装配的局限性和缺点

Autowiring works best when it is used consistently across a project. If autowiring is not used in general, it might be confusing to developers to use it to wire only one or two bean definitions.
当在项目中一致地使用自动装配时，它的效果最好。如果通常不使用自动装配，那么开发人员可能会对使用它仅装配一个或两个bean定义感到困惑。

Consider the limitations and disadvantages of autowiring:
考虑自动装配的限制和缺点：

- Explicit dependencies in `property` and `constructor-arg` settings always override autowiring. You cannot autowire simple properties such as primitives, `Strings`, and `Classes` (and arrays of such simple properties). This limitation is by-design.
  `property` 和 `constructor-arg` 设置中的显式依赖项始终覆盖自动装配。不能自动关联简单属性，如基元、 `Strings` 和 `Classes` （以及此类简单属性的数组）。此限制是设计所致。
- Autowiring is less exact than explicit wiring. Although, as noted in the earlier table, Spring is careful to avoid guessing in case of ambiguity that might have unexpected results. The relationships between your Spring-managed objects are no longer documented explicitly.
  自动连接不如显式连接精确。尽管如此，正如前面的表中所指出的，Spring还是小心地避免猜测，以免出现可能产生意外结果的歧义。Spring管理的对象之间的关系不再被显式地记录。
- Wiring information may not be available to tools that may generate documentation from a Spring container.
  连接信息对于可能从Spring容器生成文档的工具可能不可用。
- Multiple bean definitions within the container may match the type specified by the setter method or constructor argument to be autowired. For arrays, collections, or `Map` instances, this is not necessarily a problem. However, for dependencies that expect a single value, this ambiguity is not arbitrarily resolved. If no unique bean definition is available, an exception is thrown.
  容器中的多个bean定义可能与setter方法或构造函数参数指定的类型相匹配，以便自动装配。对于数组、集合或 `Map` 实例，这不一定是个问题。但是，对于期望单个值的依赖项，这种不明确性不是任意解决的。如果没有唯一的bean定义可用，则会引发异常。

In the latter scenario, you have several options:
在后一种情况下，您有几个选项：

- Abandon autowiring in favor of explicit wiring.
  放弃自动装配，采用显式装配。
- Avoid autowiring for a bean definition by setting its `autowire-candidate` attributes to `false`, as described in the [next section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire-candidate).
  通过将bean定义的 `autowire-candidate` 属性设置为 `false` 来避免自动装配bean定义，如[下一节](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire-candidate)所述。
- Designate a single bean definition as the primary candidate by setting the `primary` attribute of its `<bean/>` element to `true`.
  通过将一个bean定义的 `<bean/>` 元素的 `primary` 属性设置为 `true` ，将其指定为主要候选对象。
- Implement the more fine-grained control available with annotation-based configuration, as described in [Annotation-based Container Configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config).
  如[基于注解的容器配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config)中所述，用基于注解的配置实现更细粒度的控制。

##### 从自动装配中排除Bean

On a per-bean basis, you can exclude a bean from autowiring. In Spring’s XML format, set the `autowire-candidate` attribute of the `<bean/>` element to `false`. The container makes that specific bean definition unavailable to the autowiring infrastructure (including annotation style configurations such as [`@Autowired`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation)).
在每个bean的基础上，您可以从自动装配中排除bean。在Spring的XML格式中，将 `<bean/>` 元素的 `autowire-candidate` 属性设置为 `false` 。容器使得自动装配基础设施（包括注解样式配置，如 `@Autowired` ）无法使用特定的bean定义。

> The `autowire-candidate` attribute is designed to only affect type-based autowiring. It does not affect explicit references by name, which get resolved even if the specified bean is not marked as an autowire candidate. As a consequence, autowiring by name nevertheless injects a bean if the name matches.
> `autowire-candidate` 属性设计为仅影响基于类型的自动装配。它不影响按名称的显式引用，即使指定的bean没有标记为自动连接候选对象，也会解析这些显式引用。因此，如果名称匹配，按名称自动装配仍然会注入bean。

You can also limit autowire candidates based on pattern-matching against bean names. The top-level `<beans/>` element accepts one or more patterns within its `default-autowire-candidates` attribute. For example, to limit autowire candidate status to any bean whose name ends with `Repository`, provide a value of `*Repository`. To provide multiple patterns, define them in a comma-separated list. An explicit value of `true` or `false` for a bean definition’s `autowire-candidate` attribute always takes precedence. For such beans, the pattern matching rules do not apply.
您还可以根据与bean名称的模式匹配来限制自动装配候选对象。顶级 `<beans/>` 元素在其 `default-autowire-candidates` 属性中接受一个或多个模式。例如，要将自动装配候选状态限制为名称以 `Repository` 结尾的任何Bean，请提供值 `*Repository` 。要提供多个模式，请在逗号分隔的列表中定义它们。bean定义的 `autowire-candidate` 属性的显式值 `true` 或 `false` 始终优先。对于此类bean，模式匹配规则不适用。

These techniques are useful for beans that you never want to be injected into other beans by autowiring. It does not mean that an excluded bean cannot itself be configured by using autowiring. Rather, the bean itself is not a candidate for autowiring other beans.
这些技术对于您永远不希望通过自动装配注入到其他bean中的bean非常有用。这并不意味着被排除的bean本身不能使用自动装配进行配置。相反，bean本身不是自动装配其他bean的候选对象。

#### 1.4.6. 方法注入

In most application scenarios, most beans in the container are [singletons](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton). When a singleton bean needs to collaborate with another singleton bean or a non-singleton bean needs to collaborate with another non-singleton bean, you typically handle the dependency by defining one bean as a property of the other. A problem arises when the bean lifecycles are different. Suppose singleton bean A needs to use non-singleton (prototype) bean B, perhaps on each method invocation on A. The container creates the singleton bean A only once, and thus only gets one opportunity to set the properties. The container cannot provide bean A with a new instance of bean B every time one is needed.
在大多数应用程序场景中，容器中的大多数bean都是单例。当一个单例bean需要与另一个单例bean协作，或者一个非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。当bean的生命周期不同时就会出现问题。假设单例bean A需要使用非单例（原型）bean B，可能是在A的每个方法调用上。容器只创建单例bean A一次，因此只有一次机会设置属性。容器不能在每次需要bean B的实例时都为bean A提供新实例。

A solution is to forego some inversion of control. You can [make bean A aware of the container](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) by implementing the `ApplicationContextAware` interface, and by [making a `getBean("B")` call to the container](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-client) ask for (a typically new) bean B instance every time bean A needs it. The following example shows this approach:
一个解决办法是放弃一些控制反转。通过实现 `ApplicationContextAware` 接口，并在每次Bean A需要Bean B实例（通常是新实例）时对容器进行 `getBean("B")` 调用，可以使Bean A知道容器。以下示例说明了此方法：

```java
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

/**
 * A class that uses a stateful Command-style class to perform
 * some processing.
 */
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

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

The preceding is not desirable, because the business code is aware of and coupled to the Spring Framework. Method Injection, a somewhat advanced feature of the Spring IoC container, lets you handle this use case cleanly.
前面的情况并不理想，因为业务代码知道Spring框架并与之耦合。方法注入是SpringIoC容器的一个比较高级的特性，可以让您干净地处理这种用例。

> You can read more about the motivation for Method Injection in [this blog entry](https://spring.io/blog/2004/08/06/method-injection/).
> 你可以在[这个博客](https://spring.io/blog/2004/08/06/method-injection/)中读到更多关于方法注入的动机。

##### 查找方法注入

Lookup method injection is the ability of the container to override methods on container-managed beans and return the lookup result for another named bean in the container. The lookup typically involves a prototype bean, as in the scenario described in [the preceding section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection). The Spring Framework implements this method injection by using bytecode generation from the CGLIB library to dynamically generate a subclass that overrides the method.
查找方法注入是容器覆盖容器管理的bean上的方法并返回容器中另一个命名bean的查找结果的能力。查找通常涉及原型bean，如前一节中描述的场景。Spring框架通过使用CGLIB库中的字节码生成来动态生成覆盖该方法的子类，从而实现了这种方法注入。

> - For this dynamic subclassing to work, the class that the Spring bean container subclasses cannot be `final`, and the method to be overridden cannot be `final`, either.
>   要使这种动态子类化工作，Springbean容器子类的类不能是 `final` ，要覆盖的方法也不能是 `final` 。
> - Unit-testing a class that has an `abstract` method requires you to subclass the class yourself and to supply a stub implementation of the `abstract` method.
>   对具有 `abstract` 方法的类进行单元测试需要您自己将该类子类化，并提供 `abstract` 方法的存根实现。
> - Concrete methods are also necessary for component scanning, which requires concrete classes to pick up.
>   具体的方法对于组件扫描也是必要的，它需要具体的类来拾取。
> - A further key limitation is that lookup methods do not work with factory methods and in particular not with `@Bean` methods in configuration classes, since, in that case, the container is not in charge of creating the instance and therefore cannot create a runtime-generated subclass on the fly.
>   另一个关键限制是查找方法不能与工厂方法一起工作，尤其是不能与配置类中的 `@Bean` 方法一起工作，因为在这种情况下，容器不负责创建实例，因此不能动态地创建运行时生成的子类。

In the case of the `CommandManager` class in the previous code snippet, the Spring container dynamically overrides the implementation of the `createCommand()` method. The `CommandManager` class does not have any Spring dependencies, as the reworked example shows:
对于前面代码片段中的 `CommandManager` 类，Spring容器动态地覆盖 `createCommand()` 方法的实现。 `CommandManager` 类没有任何Spring依赖项，如修改后的示例所示：

```java
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

In the client class that contains the method to be injected (the `CommandManager` in this case), the method to be injected requires a signature of the following form:
在包含要注入的方法（本例中为 `CommandManager` ）的客户端类中，要注入的方法需要以下形式的签名：

```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

If the method is `abstract`, the dynamically-generated subclass implements the method. Otherwise, the dynamically-generated subclass overrides the concrete method defined in the original class. Consider the following example:
如果方法为 `abstract` ，则动态生成的子类实现该方法。否则，动态生成的子类将重写原始类中定义的具体方法。考虑以下示例：

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

The bean identified as `commandManager` calls its own `createCommand()` method whenever it needs a new instance of the `myCommand` bean. You must be careful to deploy the `myCommand` bean as a prototype if that is actually what is needed. If it is a [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton), the same instance of the `myCommand` bean is returned each time.
标识为 `commandManager` 的bean在需要 `myCommand` bean的新实例时调用它自己的 `createCommand()` 方法。如果确实需要的话，您必须小心地将 `myCommand` bean部署为原型。如果是单例，则每次返回 `myCommand` bean的相同实例。

Alternatively, within the annotation-based component model, you can declare a lookup method through the `@Lookup` annotation, as the following example shows:
或者，在基于注解的组件模型中，可以通过 `@Lookup` 注释声明查找方法，如下例所示：

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

Or, more idiomatically, you can rely on the target bean getting resolved against the declared return type of the lookup method:
或者，更习惯地说，您可以依赖于根据查找方法的声明返回类型解析目标bean：

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract Command createCommand();
}
```

Note that you should typically declare such annotated lookup methods with a concrete stub implementation, in order for them to be compatible with Spring’s component scanning rules where abstract classes get ignored by default. This limitation does not apply to explicitly registered or explicitly imported bean classes.
注意，通常应该使用具体的桩实现来声明这样的带注解的查找方法，以便它们与Spring的组件扫描规则兼容，在组件扫描规则中，抽象类默认被忽略。此限制不适用于显式注册或显式导入的Bean类。

> Another way of accessing differently scoped target beans is an `ObjectFactory`/ `Provider` injection point. See [Scoped Beans as Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection).
> 访问不同作用域的目标bean的另一种方法是 `ObjectFactory` / `Provider` 注入点。请参见[作为依赖项的作用域Bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)。
>
> You may also find the `ServiceLocatorFactoryBean` (in the `org.springframework.beans.factory.config` package) to be useful.
> 您可能还会发现 `ServiceLocatorFactoryBean` （在 `org.springframework.beans.factory.config` 包中）非常有用。

##### 任意方法替换

A less useful form of method injection than lookup method injection is the ability to replace arbitrary methods in a managed bean with another method implementation. You can safely skip the rest of this section until you actually need this functionality.
与查找方法注入相比，方法注入的一种不太有用的形式是用另一个方法实现替换托管bean中的任意方法的能力。您可以放心地跳过本节的其余部分，直到您实际需要此功能。

With XML-based configuration metadata, you can use the `replaced-method` element to replace an existing method implementation with another, for a deployed bean. Consider the following class, which has a method called `computeValue` that we want to override:
使用基于XML的配置元数据，您可以使用 `replaced-method` 元素为已部署的bean用另一个方法实现替换现有方法实现。考虑下面的类，它有一个名为 `computeValue` 的方法，我们想覆盖它：

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

A class that implements the `org.springframework.beans.factory.support.MethodReplacer` interface provides the new method definition, as the following example shows:
实现 `org.springframework.beans.factory.support.MethodReplacer` 接口的类提供新的方法定义，如下例所示：

```java
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

The bean definition to deploy the original class and specify the method override would resemble the following example:
部署原始类并指定方法覆盖的Bean定义将类似于以下示例：

```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

You can use one or more `<arg-type/>` elements within the `<replaced-method/>` element to indicate the method signature of the method being overridden. The signature for the arguments is necessary only if the method is overloaded and multiple variants exist within the class. For convenience, the type string for an argument may be a substring of the fully qualified type name. For example, the following all match `java.lang.String`:
可以在 `<replaced-method/>` 元素中使用一个或多个 `<arg-type/>` 元素来指示被重写方法的方法签名。仅当重载方法并且类中存在多个变量时，才需要参数的签名。为方便起见，参数的类型字符串可以是完全限定类型名的子字符串。例如，以下所有项都与 `java.lang.String` 匹配：

```xml
java.lang.String
String
Str
```

Because the number of arguments is often enough to distinguish between each possible choice, this shortcut can save a lot of typing, by letting you type only the shortest string that matches an argument type.
由于参数的数量通常足以区分每个可能的选择，因此此快捷方式可以让您仅键入与参数类型匹配的最短字符串，从而保存大量键入工作。

### 1.5. Bean作用域

When you create a bean definition, you create a recipe for creating actual instances of the class defined by that bean definition. The idea that a bean definition is a recipe is important, because it means that, as with a class, you can create many object instances from a single recipe.
当您创建bean定义时，您创建了一个方法，用于创建由该bean定义定义的类的实际实例。bean定义是一个配方的想法很重要，因为这意味着，与类一样，您可以从单个配方创建许多对象实例。

You can control not only the various dependencies and configuration values that are to be plugged into an object that is created from a particular bean definition but also control the scope of the objects created from a particular bean definition. This approach is powerful and flexible, because you can choose the scope of the objects you create through configuration instead of having to bake in the scope of an object at the Java class level. Beans can be defined to be deployed in one of a number of scopes. The Spring Framework supports six scopes, four of which are available only if you use a web-aware `ApplicationContext`. You can also create [a custom scope.](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-custom)
您不仅可以控制要插入到根据特定Bean定义创建的对象中的各种依赖关系和配置值，还可以控制根据特定Bean定义创建的对象的范围。这种方法功能强大且灵活，因为您可以选择通过配置创建的对象的作用域，而不必在Java类级别的对象作用域中进行bake。可以将Bean定义为部署在多个作用域之一中。Spring框架支持六个作用域，其中四个作用域只有在使用Web感知的 `ApplicationContext` 时才可用。您也可以创建[自定义作用域](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-custom)。

The following table describes the supported scopes:
下表介绍了支持的作用域：

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container. （默认）将单个bean定义限定为每个Spring IoC容器的单个对象实例。 |
| [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. 将单个bean定义的范围限定为任意数量的对象实例。 |
| [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. 将单个bean定义的范围限定为单个HTTP请求的生命周期。也就是说，每个HTTP请求都有自己的bean实例，该实例是在单个bean定义的基础上创建的。仅在支持Web的Spring `ApplicationContext` 的上下文中有效。 |
| [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. 将单个bean定义的范围限定为HTTP `Session` 的生命周期。仅在支持Web的Spring `ApplicationContext` 的上下文中有效。 |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`. 将单个bean定义的范围限定为 `ServletContext` 的生命周期。仅在支持Web的Spring `ApplicationContext` 的上下文中有效。 |
| [websocket](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. 将单个bean定义的范围限定为 `WebSocket` 的生命周期。仅在支持Web的Spring `ApplicationContext` 的上下文中有效。 |

> A thread scope is available but is not registered by default. For more information, see the documentation for [`SimpleThreadScope`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/support/SimpleThreadScope.html). For instructions on how to register this or any other custom scope, see [Using a Custom Scope](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-custom-using).
> thread作用域可用，但默认情况下未注册。有关详细信息，请参阅 [`SimpleThreadScope`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/context/support/SimpleThreadScope.html) 的文档。有关如何注册此自定义范围或任何其他自定义范围的说明，请参见[使用自定义作用域](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-custom-using)。

#### 1.5.1. 单例作用域

Only one shared instance of a singleton bean is managed, and all requests for beans with an ID or IDs that match that bean definition result in that one specific bean instance being returned by the Spring container.
一个单例bean只有一个共享实例被管理，并且对具有与该bean定义相匹配的ID的bean的所有请求都会导致Spring容器返回一个特定的bean实例。

To put it another way, when you define a bean definition and it is scoped as a singleton, the Spring IoC container creates exactly one instance of the object defined by that bean definition. This single instance is stored in a cache of such singleton beans, and all subsequent requests and references for that named bean return the cached object. The following image shows how the singleton scope works:
换句话说，当您定义一个bean定义并且它的作用域是单例时，SpringIoC容器只创建该bean定义所定义的对象的一个实例。这个单一实例存储在这种单例bean的缓存中，并且该命名bean的所有后续请求和引用都返回缓存的对象。下图显示了单例作用域的工作方式：

![singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/images/singleton.png)

Spring’s concept of a singleton bean differs from the singleton pattern as defined in the Gang of Four (GoF) patterns book. The GoF singleton hard-codes the scope of an object such that one and only one instance of a particular class is created per ClassLoader. The scope of the Spring singleton is best described as being per-container and per-bean. This means that, if you define one bean for a particular class in a single Spring container, the Spring container creates one and only one instance of the class defined by that bean definition. The singleton scope is the default scope in Spring. To define a bean as a singleton in XML, you can define a bean as shown in the following example:
Spring的单例bean概念不同于Gang of Four（GoF）模式一书中定义的单例模式。GoF singleton硬编码对象的作用域，这样每个ClassLoader创建一个且仅一个特定类的实例。Spring单例的范围最好描述为每个容器和每个bean。这意味着，如果您在单个Spring容器中为特定类定义了一个Bean，则Spring容器将创建且仅创建由该Bean定义所定义的类的一个实例。单例作用域是Spring中的默认作用域。要将Bean定义为XML中的单例，可以按以下示例所示定义Bean：

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

#### 1.5.2. 原型作用域

The non-singleton prototype scope of bean deployment results in the creation of a new bean instance every time a request for that specific bean is made. That is, the bean is injected into another bean or you request it through a `getBean()` method call on the container. As a rule, you should use the prototype scope for all stateful beans and the singleton scope for stateless beans.
bean部署的非单例原型作用域导致每次对特定bean发出请求时都创建一个新bean实例。也就是说，bean被注入到另一个bean中，或者您通过对容器的 `getBean()` 方法调用来请求它。通常，您应该对所有有状态bean使用原型作用域，对无状态bean使用单例作用域。

The following diagram illustrates the Spring prototype scope:
下图说明了Spring原型的作用域：

![prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/images/prototype.png)

(A data access object (DAO) is not typically configured as a prototype, because a typical DAO does not hold any conversational state. It was easier for us to reuse the core of the singleton diagram.)
（数据访问对象（DAO）通常不配置为原型，因为典型的DAO不保存任何对话状态。我们更容易重用单例图的核心。）

The following example defines a bean as a prototype in XML:
以下示例将Bean定义为XML中的原型：

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

In contrast to the other scopes, Spring does not manage the complete lifecycle of a prototype bean. The container instantiates, configures, and otherwise assembles a prototype object and hands it to the client, with no further record of that prototype instance. Thus, although initialization lifecycle callback methods are called on all objects regardless of scope, in the case of prototypes, configured destruction lifecycle callbacks are not called. The client code must clean up prototype-scoped objects and release expensive resources that the prototype beans hold. To get the Spring container to release resources held by prototype-scoped beans, try using a custom [bean post-processor](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp), which holds a reference to beans that need to be cleaned up.
与其他作用域相比，Spring不管理原型bean的完整生命周期。容器实例化、配置和组装原型对象，并将其传递给客户机，而不进一步记录该原型实例。因此，尽管初始化生命周期回调方法可以在所有对象上调用，而不管其作用域如何，但在原型的情况下，不会调用配置的析构生命周期回调。客户端代码必须清除原型范围内的对象，并释放原型bean所拥有的昂贵资源。要让Spring容器释放由原型范围的bean所持有的资源，请尝试使用自定义[bean后处理器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp)，它持有对需要清理的bean的引用。

In some respects, the Spring container’s role in regard to a prototype-scoped bean is a replacement for the Java `new` operator. All lifecycle management past that point must be handled by the client. (For details on the lifecycle of a bean in the Spring container, see [Lifecycle Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle).)
在某些方面，原型作用域bean在Spring容器中的角色是Java `new` 操作符的替代。超过该点的所有生命周期管理都必须由客户端处理。(有关Spring容器中bean生命周期的详细信息，请参见[生命周期回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)。）

#### 1.5.3. 具有原型Bean依赖关系的单例Bean

When you use singleton-scoped beans with dependencies on prototype beans, be aware that dependencies are resolved at instantiation time. Thus, if you dependency-inject a prototype-scoped bean into a singleton-scoped bean, a new prototype bean is instantiated and then dependency-injected into the singleton bean. The prototype instance is the sole instance that is ever supplied to the singleton-scoped bean.
当您使用依赖于原型bean的单例作用域bean时，请注意依赖关系是在实例化时解析的。因此，如果您将一个原型范围的bean依赖性注入到一个单例范围的bean中，则会实例化一个新的原型bean，然后将依赖性注入到单例bean中。原型实例是提供给单例作用域bean的唯一实例。

However, suppose you want the singleton-scoped bean to acquire a new instance of the prototype-scoped bean repeatedly at runtime. You cannot dependency-inject a prototype-scoped bean into your singleton bean, because that injection occurs only once, when the Spring container instantiates the singleton bean and resolves and injects its dependencies. If you need a new instance of a prototype bean at runtime more than once, see [Method Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection).
但是，假设您希望单例作用域Bean在运行时重复获取原型作用域Bean的新实例。不能将原型作用域的bean依赖性注入到单例bean中，因为这种注入只发生一次，即Spring容器实例化单例bean并解析和注入其依赖性时。如果在运行时需要原型bean的新实例不止一次，请参见[方法注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection)。

#### 1.5.4. 请求、会话、应用程序和WebSocket作用域

The `request`, `session`, `application`, and `websocket` scopes are available only if you use a web-aware Spring `ApplicationContext` implementation (such as `XmlWebApplicationContext`). If you use these scopes with regular Spring IoC containers, such as the `ClassPathXmlApplicationContext`, an `IllegalStateException` that complains about an unknown bean scope is thrown.
`request` 、 `session` 、 `application` 和 `websocket` 作用域仅在您使用Web感知Spring `ApplicationContext` 实现（如 `XmlWebApplicationContext` ）时可用。如果您将这些作用域与常规Spring IoC容器（如 `ClassPathXmlApplicationContext` ）一起使用，则会抛出一个 `IllegalStateException` ，抱怨未知的bean作用域。

#####  初始Web配置

To support the scoping of beans at the `request`, `session`, `application`, and `websocket` levels (web-scoped beans), some minor initial configuration is required before you define your beans. (This initial setup is not required for the standard scopes: `singleton` and `prototype`.)
要支持 `request` 、 `session` 、 `application` 和 `websocket` 级别的bean（Web范围的bean）的范围，在定义bean之前需要进行一些小的初始配置。(这些标准内作用域不需要初始设置：`singleton`和`prototype`。）

How you accomplish this initial setup depends on your particular Servlet environment.
如何完成此初始设置取决于特定的Servlet环境。

If you access scoped beans within Spring Web MVC, in effect, within a request that is processed by the Spring `DispatcherServlet`, no special setup is necessary. `DispatcherServlet` already exposes all relevant state.
如果您在SpringWebMVC中访问scopedbean，实际上，在Spring `DispatcherServlet` 处理的请求中，不需要特殊的设置。 `DispatcherServlet` 已公开所有相关状态。

If you use a Servlet web container, with requests processed outside of Spring’s `DispatcherServlet` (for example, when using JSF), you need to register the `org.springframework.web.context.request.RequestContextListener` `ServletRequestListener`. This can be done programmatically by using the `WebApplicationInitializer` interface. Alternatively, add the following declaration to your web application’s `web.xml` file:
如果您使用Servlet Web容器，并且请求在Spring的 `DispatcherServlet` 之外处理（例如，使用JSF时），则需要注册 `org.springframework.web.context.request.RequestContextListener` `ServletRequestListener` 。这可以通过使用 `WebApplicationInitializer` 接口以编程方式完成。或者，将以下声明添加到Web应用程序的 `web.xml` 文件中：

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

Alternatively, if there are issues with your listener setup, consider using Spring’s `RequestContextFilter`. The filter mapping depends on the surrounding web application configuration, so you have to change it as appropriate. The following listing shows the filter part of a web application:
或者，如果侦听器设置有问题，请考虑使用Spring的 `RequestContextFilter` 。过滤器映射取决于周围的Web应用程序配置，因此您必须根据需要进行更改。以下清单显示了Web应用程序的筛选器部分：

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet`, `RequestContextListener`, and `RequestContextFilter` all do exactly the same thing, namely bind the HTTP request object to the `Thread` that is servicing that request. This makes beans that are request- and session-scoped available further down the call chain.
`DispatcherServlet` 、 `RequestContextListener` 和 `RequestContextFilter` 都执行完全相同的操作，即将HTTP请求对象绑定到为该请求提供服务的 `Thread` 。这使得请求和会话作用域的bean在调用链的下游可用。

##### Request 作用域

Consider the following XML configuration for a bean definition:
请考虑Bean定义的以下XML配置：

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

The Spring container creates a new instance of the `LoginAction` bean by using the `loginAction` bean definition for each and every HTTP request. That is, the `loginAction` bean is scoped at the HTTP request level. You can change the internal state of the instance that is created as much as you want, because other instances created from the same `loginAction` bean definition do not see these changes in state. They are particular to an individual request. When the request completes processing, the bean that is scoped to the request is discarded.
Spring容器通过对每个HTTP请求使用 `loginAction` bean定义来创建 `LoginAction` bean的新实例。也就是说， `loginAction` bean的作用域在HTTP请求级别。您可以随意更改所创建实例的内部状态，因为从同一 `loginAction` bean定义创建的其他实例看不到这些状态更改。它们专门针对个人请求。当请求完成处理时，作用域为该请求的bean将被丢弃。

When using annotation-driven components or Java configuration, the `@RequestScope` annotation can be used to assign a component to the `request` scope. The following example shows how to do so:
当使用注释驱动的组件或Java配置时， `@RequestScope` 注释可用于将组件分配到 `request` 作用域。下面的示例说明了如何执行此操作：

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

##### Session 作用域

Consider the following XML configuration for a bean definition:
请考虑Bean定义的以下XML配置：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

The Spring container creates a new instance of the `UserPreferences` bean by using the `userPreferences` bean definition for the lifetime of a single HTTP `Session`. In other words, the `userPreferences` bean is effectively scoped at the HTTP `Session` level. As with request-scoped beans, you can change the internal state of the instance that is created as much as you want, knowing that other HTTP `Session` instances that are also using instances created from the same `userPreferences` bean definition do not see these changes in state, because they are particular to an individual HTTP `Session`. When the HTTP `Session` is eventually discarded, the bean that is scoped to that particular HTTP `Session` is also discarded.
Spring容器通过在单个HTTP `Session` 的生命周期内使用 `userPreferences` bean定义来创建 `UserPreferences` bean的新实例。换句话说， `userPreferences` bean的作用域实际上是在HTTP `Session` 级别。与请求作用域bean一样，您可以根据需要更改所创建实例的内部状态，因为您知道其他HTTP `Session` 实例（也使用从同一 `userPreferences` bean定义创建的实例）不会看到这些状态更改，因为它们是特定于单个HTTP `Session` 的。当HTTP `Session` 最终被丢弃时，作用域为该特定HTTP `Session` 的bean也将被丢弃。

When using annotation-driven components or Java configuration, you can use the `@SessionScope` annotation to assign a component to the `session` scope.
在使用注释驱动的组件或Java配置时，可以使用 `@SessionScope` 注释将组件分配给 `session` 作用域。

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

##### Application 作用域

Consider the following XML configuration for a bean definition:
请考虑Bean定义的以下XML配置：

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

The Spring container creates a new instance of the `AppPreferences` bean by using the `appPreferences` bean definition once for the entire web application. That is, the `appPreferences` bean is scoped at the `ServletContext` level and stored as a regular `ServletContext` attribute. This is somewhat similar to a Spring singleton bean but differs in two important ways: It is a singleton per `ServletContext`, not per Spring `ApplicationContext` (for which there may be several in any given web application), and it is actually exposed and therefore visible as a `ServletContext` attribute.
Spring容器通过对整个Web应用程序使用一次 `appPreferences` bean定义来创建 `AppPreferences` bean的新实例。也就是说， `appPreferences` bean的作用域在 `ServletContext` 级别，并存储为常规的 `ServletContext` 属性。这有点类似于Spring单例bean，但在两个重要方面有所不同：它是每个 `ServletContext` 的单例，而不是每个Spring `ApplicationContext` 的单例（Spring `ApplicationContext` 在任何给定的web应用程序中可能有多个），并且它实际上是公开的，因此作为 `ServletContext` 属性可见。

When using annotation-driven components or Java configuration, you can use the `@ApplicationScope` annotation to assign a component to the `application` scope. The following example shows how to do so:
在使用注释驱动的组件或Java配置时，可以使用 `@ApplicationScope` 注释将组件分配给 `application` 作用域。下面的示例说明了如何执行此操作：

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

##### WebSocket 作用域

WebSocket scope is associated with the lifecycle of a WebSocket session and applies to STOMP over WebSocket applications, see [WebSocket scope](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) for more details.
WebSocket作用域与WebSocket会话的生命周期相关联，并适用于WebSocket应用程序上的STOMP，有关详细信息，请参阅[WebSocket作用域](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope)。

##### 作为依赖项的作用域Bean

The Spring IoC container manages not only the instantiation of your objects (beans), but also the wiring up of collaborators (or dependencies). If you want to inject (for example) an HTTP request-scoped bean into another bean of a longer-lived scope, you may choose to inject an AOP proxy in place of the scoped bean. That is, you need to inject a proxy object that exposes the same public interface as the scoped object but that can also retrieve the real target object from the relevant scope (such as an HTTP request) and delegate method calls onto the real object.
Spring IoC容器不仅管理对象（bean）的实例化，还管理协作者（或依赖项）的连接。例如，如果您想将一个HTTP请求范围的bean注入到另一个更长生存期范围的bean中，您可以选择注入一个AOP代理来代替范围bean。也就是说，您需要注入一个代理对象，该代理对象公开与作用域对象相同的公共接口，但也可以从相关作用域（如HTTP请求）检索真实的目标对象，并将方法调用委托给实际对象。

> You may also use `<aop:scoped-proxy/>` between beans that are scoped as `singleton`, with the reference then going through an intermediate proxy that is serializable and therefore able to re-obtain the target singleton bean on deserialization.
> 您还可以在作用域为 `singleton` 的bean之间使用 `<aop:scoped-proxy/>` ，然后引用通过一个可序列化的中间代理，从而能够在反序列化时重新获得目标单例bean。
>
> When declaring `<aop:scoped-proxy/>` against a bean of scope `prototype`, every method call on the shared proxy leads to the creation of a new target instance to which the call is then being forwarded.
> 当针对范围为 `prototype` 的bean声明 `<aop:scoped-proxy/>` 时，共享代理上的每个方法调用都会导致创建一个新的目标实例，然后调用将被转发到该实例。
>
> Also, scoped proxies are not the only way to access beans from shorter scopes in a lifecycle-safe fashion. You may also declare your injection point (that is, the constructor or setter argument or autowired field) as `ObjectFactory<MyTargetBean>`, allowing for a `getObject()` call to retrieve the current instance on demand every time it is needed — without holding on to the instance or storing it separately.
> 此外，作用域代理并不是以生命周期安全的方式从较短的作用域访问bean的唯一方法。您也可以将注入点（即构造函数或setter参数或autowired字段）声明为 `ObjectFactory<MyTargetBean>` ，这样就可以在每次需要时，通过 `getObject()` 调用来检索当前实例，而无需保留实例或单独存储它。
>
> As an extended variant, you may declare `ObjectProvider<MyTargetBean>` which delivers several additional access variants, including `getIfAvailable` and `getIfUnique`.
> 作为一个扩展变量，您可以声明 `ObjectProvider<MyTargetBean>` ，它提供了几个额外的访问变量，包括 `getIfAvailable` 和 `getIfUnique` 。
>
> The JSR-330 variant of this is called `Provider` and is used with a `Provider<MyTargetBean>` declaration and a corresponding `get()` call for every retrieval attempt. See [here](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-standard-annotations) for more details on JSR-330 overall.
> 它的JSR-330变体称为 `Provider` ，每次检索尝试都要使用 `Provider<MyTargetBean>` 声明和相应的 `get()` 调用。有关JSR-330整体的更多详细信息，请参见此处。

The configuration in the following example is only one line, but it is important to understand the “why” as well as the “how” behind it:
以下示例中的配置只有一行，但了解其背后的“原因”和“方式”非常重要：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> (1)
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

1.  The line that defines the proxy. 定义代理的行。

To create such a proxy, you insert a child `<aop:scoped-proxy/>` element into a scoped bean definition (see [Choosing the Type of Proxy to Create](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection-proxies) and [XML Schema-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core.appendix.xsd-schemas)). Why do definitions of beans scoped at the `request`, `session` and custom-scope levels require the `<aop:scoped-proxy/>` element? Consider the following singleton bean definition and contrast it with what you need to define for the aforementioned scopes (note that the following `userPreferences` bean definition as it stands is incomplete):
要创建这样的代理，您需要将子 `<aop:scoped-proxy/>` 元素插入到作用域Bean定义中（请参见[选择要创建的代理类型](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection-proxies)和[基于XML Schema的配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core.appendix.xsd-schemas)）。为什么作用域在 `request` 、 `session` 和自定义作用域级别的bean的定义需要 `<aop:scoped-proxy/>` 元素？考虑下面的单例bean定义，并将其与您需要为前面提到的作用域定义的内容进行对比（注意，下面的 `userPreferences` bean定义是不完整的）：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

In the preceding example, the singleton bean (`userManager`) is injected with a reference to the HTTP `Session`-scoped bean (`userPreferences`). The salient point here is that the `userManager` bean is a singleton: it is instantiated exactly once per container, and its dependencies (in this case only one, the `userPreferences` bean) are also injected only once. This means that the `userManager` bean operates only on the exact same `userPreferences` object (that is, the one with which it was originally injected).
在前面的示例中，单例bean（ `userManager` ）被注入了对HTTP `Session` 作用域bean（ `userPreferences` ）的引用。这里突出的一点是 `userManager` bean是一个单例：它在每个容器中实例化一次，并且它的依赖项（在本例中只有一个， `userPreferences` bean）也只注入一次。这意味着 `userManager` bean只对完全相同的 `userPreferences` 对象（即，最初注入它的对象）进行操作。

This is not the behavior you want when injecting a shorter-lived scoped bean into a longer-lived scoped bean (for example, injecting an HTTP `Session`-scoped collaborating bean as a dependency into singleton bean). Rather, you need a single `userManager` object, and, for the lifetime of an HTTP `Session`, you need a `userPreferences` object that is specific to the HTTP `Session`. Thus, the container creates an object that exposes the exact same public interface as the `UserPreferences` class (ideally an object that is a `UserPreferences` instance), which can fetch the real `UserPreferences` object from the scoping mechanism (HTTP request, `Session`, and so forth). The container injects this proxy object into the `userManager` bean, which is unaware that this `UserPreferences` reference is a proxy. In this example, when a `UserManager` instance invokes a method on the dependency-injected `UserPreferences` object, it is actually invoking a method on the proxy. The proxy then fetches the real `UserPreferences` object from (in this case) the HTTP `Session` and delegates the method invocation onto the retrieved real `UserPreferences` object.
这不是将较短生存期的作用域Bean注入较长生存期的作用域Bean时所希望的行为（例如，将HTTP `Session` 作用域的协作Bean作为依赖项注入单例Bean）。相反，您需要一个 `userManager` 对象，并且在HTTP `Session` 的生存期内，您需要一个特定于HTTP `Session` 的 `userPreferences` 对象。因此，容器创建了一个公开与 `UserPreferences` 类完全相同的公共接口的对象（理想情况下，该对象是 `UserPreferences` 实例），它可以从作用域机制（HTTP请求、 `Session` 等）获取真实的的 `UserPreferences` 对象。容器将这个代理对象注入到 `userManager` bean中，而 `userManager` bean并不知道这个 `UserPreferences` 引用是一个代理。在这个例子中，当 `UserManager` 实例调用依赖注入的 `UserPreferences` 对象上的方法时，它实际上是在调用代理上的方法。 然后代理从HTTP `Session` 获取真实的 `UserPreferences` 对象（在本例中），并将方法调用委托给检索到的真实 `UserPreferences` 对象。

Thus, you need the following (correct and complete) configuration when injecting `request-` and `session-scoped` beans into collaborating objects, as the following example shows:
因此，在将 `request-` 和 `session-scoped` bean注入协作对象时，需要以下（正确且完整）配置，如以下示例所示：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

###### 选择要创建的代理类型

By default, when the Spring container creates a proxy for a bean that is marked up with the `<aop:scoped-proxy/>` element, a CGLIB-based class proxy is created.
默认情况下，当Spring容器为用 `<aop:scoped-proxy/>` 元素标记的bean创建代理时，将创建一个基于CGLIB的类代理。

> CGLIB proxies intercept only public method calls! Do not call non-public methods on such a proxy. They are not delegated to the actual scoped target object.
> CGLIB代理只拦截公共方法调用！不要在这样的代理上调用非公共方法。它们不会委托给实际的作用域目标对象。

Alternatively, you can configure the Spring container to create standard JDK interface-based proxies for such scoped beans, by specifying `false` for the value of the `proxy-target-class` attribute of the `<aop:scoped-proxy/>` element. Using JDK interface-based proxies means that you do not need additional libraries in your application classpath to affect such proxying. However, it also means that the class of the scoped bean must implement at least one interface and that all collaborators into which the scoped bean is injected must reference the bean through one of its interfaces. The following example shows a proxy based on an interface:
或者，您可以配置Spring容器，通过为 `<aop:scoped-proxy/>` 元素的 `proxy-target-class` 属性值指定 `false` ，为此类作用域Bean创建基于标准JDK接口的代理。使用基于JDK接口的代理意味着应用程序类路径中不需要其他库来影响此类代理。但是，这也意味着作用域bean的类必须实现至少一个接口，并且所有注入了作用域bean的协作者都必须通过它的一个接口引用该bean。下面的示例显示了基于接口的代理：

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

For more detailed information about choosing class-based or interface-based proxying, see [Proxying Mechanisms](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-proxying).
有关选择基于类或基于接口的代理的详细信息，请参见[代理机制](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-proxying)。

#### 1.5.5. 自定义作用域

The bean scoping mechanism is extensible. You can define your own scopes or even redefine existing scopes, although the latter is considered bad practice and you cannot override the built-in `singleton` and `prototype` scopes.
bean作用域机制是可扩展的。您可以定义自己的作用域，甚至重新定义现有的作用域，尽管后者被认为是不好的做法，并且您不能覆盖内置的 `singleton` 和 `prototype` 作用域。

##### 创建自定义范围

To integrate your custom scopes into the Spring container, you need to implement the `org.springframework.beans.factory.config.Scope` interface, which is described in this section. For an idea of how to implement your own scopes, see the `Scope` implementations that are supplied with the Spring Framework itself and the [`Scope`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/beans/factory/config/Scope.html) javadoc, which explains the methods you need to implement in more detail.
要将自定义作用域集成到Spring容器中，需要实现 `org.springframework.beans.factory.config.Scope` 接口，本节将对此进行描述。要了解如何实现自己的作用域，请参阅Spring框架本身提供的 `Scope` 实现和[ `Scope`](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/beans/factory/config/Scope.html) javadoc，后者更详细地解释了需要实现的方法。

The `Scope` interface has four methods to get objects from the scope, remove them from the scope, and let them be destroyed.
`Scope` 接口有四个方法，用于从作用域获取对象、从作用域移除对象以及销毁对象。

The session scope implementation, for example, returns the session-scoped bean (if it does not exist, the method returns a new instance of the bean, after having bound it to the session for future reference). The following method returns the object from the underlying scope:
例如，session作用域实现了返回session范围的bean（如果它不存在，则在将bean绑定到session以供将来引用之后，该方法返回bean的新实例）。下面的方法从底层范围返回对象：

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

The session scope implementation, for example, removes the session-scoped bean from the underlying session. The object should be returned, but you can return `null` if the object with the specified name is not found. The following method removes the object from the underlying scope:
例如，session作用域实现从底层session中删除session范围的bean。应返回该对象，但如果找不到具有指定名称的对象，则可以返回 `null` 。以下方法从底层范围中删除该对象：

```java
Object remove(String name)
```

The following method registers a callback that the scope should invoke when it is destroyed or when the specified object in the scope is destroyed:
下面的方法注册一个回调，当作用域被销毁或作用域中的指定对象被销毁时，作用域应调用该回调：

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

See the [javadoc](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/beans/factory/config/Scope.html#registerDestructionCallback) or a Spring scope implementation for more information on destruction callbacks.
有关销毁回调的更多信息，请参见[javadoc](https://docs.spring.io/spring-framework/docs/6.0.7/javadoc-api/org/springframework/beans/factory/config/Scope.html#registerDestructionCallback)或Spring作用域实现。

The following method obtains the conversation identifier for the underlying scope:
下面的方法获取底层范围的会话标识符：

```java
String getConversationId()
```

This identifier is different for each scope. For a session scoped implementation, this identifier can be the session identifier.
此标识符对于每个作用域都是不同的。对于session范围的实现，此标识符可以是会话标识符。

##### 使用自定义范围

After you write and test one or more custom `Scope` implementations, you need to make the Spring container aware of your new scopes. The following method is the central method to register a new `Scope` with the Spring container:
在编写和测试一个或多个定制 `Scope` 实现之后，需要让Spring容器知道您的新作用域。以下方法是向Spring容器注册新的 `Scope` 的核心方法：

```java
void registerScope(String scopeName, Scope scope);
```

This method is declared on the `ConfigurableBeanFactory` interface, which is available through the `BeanFactory` property on most of the concrete `ApplicationContext` implementations that ship with Spring.
此方法在 `ConfigurableBeanFactory` 接口上声明，该接口可通过Spring附带的大多数具体 `ApplicationContext` 实现上的 `BeanFactory` 属性获得。

The first argument to the `registerScope(..)` method is the unique name associated with a scope. Examples of such names in the Spring container itself are `singleton` and `prototype`. The second argument to the `registerScope(..)` method is an actual instance of the custom `Scope` implementation that you wish to register and use.
`registerScope(..)` 方法的第一个参数是与作用域关联的唯一名称。Spring容器本身中的此类名称的示例是 `singleton` 和 `prototype` 。 `registerScope(..)` 方法的第二个参数是您希望注册和使用的自定义 `Scope` 实现的实际实例。

Suppose that you write your custom `Scope` implementation, and then register it as shown in the next example.
假设您编写了自定义 `Scope` 实现，然后按下一个示例所示注册它。

> The next example uses `SimpleThreadScope`, which is included with Spring but is not registered by default. The instructions would be the same for your own custom `Scope` implementations.
> 下一个示例使用 `SimpleThreadScope` ，它包含在Spring中，但默认情况下未注册。对于您自己的自定义 `Scope` 实现，这些说明是相同的。

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

You can then create bean definitions that adhere to the scoping rules of your custom `Scope`, as follows:
然后，可以创建遵循自定义 `Scope` 的作用域规则的Bean定义，如下所示：

```xml
<bean id="..." class="..." scope="thread">
```

With a custom `Scope` implementation, you are not limited to programmatic registration of the scope. You can also do the `Scope` registration declaratively, by using the `CustomScopeConfigurer` class, as the following example shows:
使用自定义 `Scope` 实现，您将不限于以编程方式注册范围。您还可以使用 `CustomScopeConfigurer` 类以声明方式执行 `Scope` 注册，如下例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

> When you place `<aop:scoped-proxy/>` within a `<bean>` declaration for a `FactoryBean` implementation, it is the factory bean itself that is scoped, not the object returned from `getObject()`.
> 当您将 `<aop:scoped-proxy/>` 放置在 `FactoryBean` 实现的 `<bean>` 声明中时，作用域是工厂Bean本身，而不是从 `getObject()` 返回的对象。

### 1.6. 定制Bean的性质

The Spring Framework provides a number of interfaces you can use to customize the nature of a bean. This section groups them as follows:
Spring框架提供了许多接口，您可以使用这些接口来定制bean的特性。本节将它们分组如下：

- [Lifecycle Callbacks 生命周期回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)
- [`ApplicationContextAware` and `BeanNameAware ApplicationContextAware 和 BeanNameAware `](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)
- [Other `Aware` Interfaces 其他 `Aware` 接口](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aware-list)

#### 1.6.1. 生命周期回调

To interact with the container’s management of the bean lifecycle, you can implement the Spring `InitializingBean` and `DisposableBean` interfaces. The container calls `afterPropertiesSet()` for the former and `destroy()` for the latter to let the bean perform certain actions upon initialization and destruction of your beans.

要与容器对bean生命周期的管理进行交互，可以实现Spring `InitializingBean` 和 `DisposableBean` 接口。容器为前者调用 `afterPropertiesSet()` ，为后者调用 `destroy()` ，以使bean在初始化和销毁bean时执行某些操作。

> The JSR-250 `@PostConstruct` and `@PreDestroy` annotations are generally considered best practice for receiving lifecycle callbacks in a modern Spring application. Using these annotations means that your beans are not coupled to Spring-specific interfaces. For details, see [Using `@PostConstruct` and `@PreDestroy`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations).
> JSR-250 `@PostConstruct` 和 `@PreDestroy` 注释通常被认为是在现代Spring应用程序中接收生命周期回调的最佳实践。使用这些注释意味着您的bean没有耦合到Spring特定的接口。有关详细信息，请参见使用 `@PostConstruct` 和 `@PreDestroy` 。
>
> If you do not want to use the JSR-250 annotations but you still want to remove coupling, consider `init-method` and `destroy-method` bean definition metadata.
> 如果您不想使用JSR-250注释，但仍想去除耦合，请考虑 `init-method` 和 `destroy-method` bean定义元数据。

Internally, the Spring Framework uses `BeanPostProcessor` implementations to process any callback interfaces it can find and call the appropriate methods. If you need custom features or other lifecycle behavior Spring does not by default offer, you can implement a `BeanPostProcessor` yourself. For more information, see [Container Extension Points](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension).
在内部，Spring框架使用 `BeanPostProcessor` 实现来处理它能找到的任何回调接口并调用适当的方法。如果您需要Spring默认不提供的定制特性或其他生命周期行为，您可以自己实现 `BeanPostProcessor` 。有关更多信息，请参见[容器扩展点](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension)。

In addition to the initialization and destruction callbacks, Spring-managed objects may also implement the `Lifecycle` interface so that those objects can participate in the startup and shutdown process, as driven by the container’s own lifecycle.
除了初始化和销毁回调之外，Spring管理的对象还可以实现 `Lifecycle` 接口，以便这些对象可以参与启动和关闭过程，这由容器自己的生命周期驱动。

The lifecycle callback interfaces are described in this section.
本节介绍生命周期回调接口。

##### 初始化回调

The `org.springframework.beans.factory.InitializingBean` interface lets a bean perform initialization work after the container has set all necessary properties on the bean. The `InitializingBean` interface specifies a single method:
`org.springframework.beans.factory.InitializingBean` 接口允许bean在容器设置了bean的所有必要属性之后执行初始化工作。 `InitializingBean` 接口指定单个方法：

```java
void afterPropertiesSet() throws Exception;
```

We recommend that you do not use the `InitializingBean` interface, because it unnecessarily couples the code to Spring. Alternatively, we suggest using the [`@PostConstruct`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations) annotation or specifying a POJO initialization method. In the case of XML-based configuration metadata, you can use the `init-method` attribute to specify the name of the method that has a void no-argument signature. With Java configuration, you can use the `initMethod` attribute of `@Bean`. See [Receiving Lifecycle Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-lifecycle-callbacks). Consider the following example:
我们建议您不要使用 `InitializingBean` 接口，因为它不必要地将代码耦合到Spring。或者，我们建议使用 [`@PostConstruct`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations) 注释或指定POJO初始化方法。对于基于XML的配置元数据，可以使用 `init-method` 属性指定具有void无参数签名的方法的名称。通过Java配置，您可以使用 `@Bean` 的 `initMethod` 属性。请参阅：[接收生命周期回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-lifecycle-callbacks)考虑以下示例：

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

The preceding example has almost exactly the same effect as the following example (which consists of two listings):
前面的示例与下面的示例（由两个清单组成）的效果几乎完全相同：

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

However, the first of the two preceding examples does not couple the code to Spring.
但是，前面两个示例中的第一个没有将代码耦合到Spring。

##### 销毁回调

Implementing the `org.springframework.beans.factory.DisposableBean` interface lets a bean get a callback when the container that contains it is destroyed. The `DisposableBean` interface specifies a single method:
实现 `org.springframework.beans.factory.DisposableBean` 接口可以让bean在包含它的容器被销毁时获得回调。 `DisposableBean` 接口指定了一个方法：

```java
void destroy() throws Exception;
```

We recommend that you do not use the `DisposableBean` callback interface, because it unnecessarily couples the code to Spring. Alternatively, we suggest using the [`@PreDestroy`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations) annotation or specifying a generic method that is supported by bean definitions. With XML-based configuration metadata, you can use the `destroy-method` attribute on the `<bean/>`. With Java configuration, you can use the `destroyMethod` attribute of `@Bean`. See [Receiving Lifecycle Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-lifecycle-callbacks). Consider the following definition:
我们建议您不要使用 `DisposableBean` 回调接口，因为它不必要地将代码耦合到Spring。或者，我们建议使用 `@PreDestroy` 注释或指定bean定义支持的泛型方法。使用基于XML的配置元数据，您可以在 `<bean/>` 上使用 `destroy-method` 属性。使用Java配置，您可以使用 `@Bean` 的 `destroyMethod` 属性。请参阅接收生命周期回调。考虑以下定义：

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

The preceding definition has almost exactly the same effect as the following definition:
前面的定义与下面的定义几乎具有完全相同的效果：

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

However, the first of the two preceding definitions does not couple the code to Spring.
但是，前面两个定义中的第一个并没有将代码耦合到Spring。

> You can assign the `destroy-method` attribute of a `<bean>` element a special `(inferred)` value, which instructs Spring to automatically detect a public `close` or `shutdown` method on the specific bean class. (Any class that implements `java.lang.AutoCloseable` or `java.io.Closeable` would therefore match.) You can also set this special `(inferred)` value on the `default-destroy-method` attribute of a `<beans>` element to apply this behavior to an entire set of beans (see [Default Initialization and Destroy Methods](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-default-init-destroy-methods)). Note that this is the default behavior with Java configuration.
> 您可以为 `<bean>` 元素的 `destroy-method` 属性分配一个特殊的 `(inferred)` 值，该值指示Spring自动检测特定bean类上的公共 `close` 或 `shutdown` 方法。(任何实现了 `java.lang.AutoCloseable` 或 `java.io.Closeable` 的类将因此匹配）。您还可以在 `<beans>` 元素的 `default-destroy-method` 属性上设置这个特殊的 `(inferred)` 值，以将此行为应用于整个bean集（请参阅[默认初始化和销毁方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-default-init-destroy-methods)）。注意，这是Java配置的默认行为。

##### 默认初始化和销毁方法

When you write initialization and destroy method callbacks that do not use the Spring-specific `InitializingBean` and `DisposableBean` callback interfaces, you typically write methods with names such as `init()`, `initialize()`, `dispose()`, and so on. Ideally, the names of such lifecycle callback methods are standardized across a project so that all developers use the same method names and ensure consistency.
当你编写不使用Spring特有的 `InitializingBean` 和 `DisposableBean` 回调接口的初始化和销毁方法回调时，你通常使用诸如 `init()` 、 `initialize()` 、 `dispose()` 等名称来编写方法。理想情况下，这些生命周期回调方法的名称在整个项目中是标准化的，这样所有开发人员都使用相同的方法名称，并确保一致性。

You can configure the Spring container to “look” for named initialization and destroy callback method names on every bean. This means that you, as an application developer, can write your application classes and use an initialization callback called `init()`, without having to configure an `init-method="init"` attribute with each bean definition. The Spring IoC container calls that method when the bean is created (and in accordance with the standard lifecycle callback contract [described previously](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)). This feature also enforces a consistent naming convention for initialization and destroy method callbacks.
您可以配置Spring容器来“查找”命名初始化并销毁每个bean上的回调方法名称。这意味着，作为应用程序开发人员，您可以编写应用程序类并使用名为 `init()` 的初始化回调，而不必为每个bean定义配置 `init-method="init"` 属性。SpringIoC容器在创建bean时调用该方法（并根据前面描述的标准生命周期回调契约）。这个特性还为初始化和销毁方法回调强制了一致的命名约定。

Suppose that your initialization callback methods are named `init()` and your destroy callback methods are named `destroy()`. Your class then resembles the class in the following example:
假设初始化回调方法命名为 `init()` ，销毁回调方法命名为 `destroy()` 。然后，您的类将类似于以下示例中的类：

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

You could then use that class in a bean resembling the following:
然后，您可以在类似于以下内容的bean中使用该类：

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

The presence of the `default-init-method` attribute on the top-level `<beans/>` element attribute causes the Spring IoC container to recognize a method called `init` on the bean class as the initialization method callback. When a bean is created and assembled, if the bean class has such a method, it is invoked at the appropriate time.
顶层 `<beans/>` 元素属性上的 `default-init-method` 属性会导致Spring IoC容器将bean类上的一个名为 `init` 的方法识别为初始化方法回调。当创建并组装bean时，如果bean类具有这样的方法，则会在适当的时间调用它。

You can configure destroy method callbacks similarly (in XML, that is) by using the `default-destroy-method` attribute on the top-level `<beans/>` element.
您可以通过使用顶级 `<beans/>` 元素上的 `default-destroy-method` 属性来类似地配置destroy方法回调（即在XML中）。

Where existing bean classes already have callback methods that are named at variance with the convention, you can override the default by specifying (in XML, that is) the method name by using the `init-method` and `destroy-method` attributes of the `<bean/>` itself.
如果现有的bean类已经有了命名与约定不同的回调方法，则可以通过使用 `<bean/>` 本身的 `init-method` 和 `destroy-method` 属性指定（即在XML中）方法名称来覆盖默认值。

The Spring container guarantees that a configured initialization callback is called immediately after a bean is supplied with all dependencies. Thus, the initialization callback is called on the raw bean reference, which means that AOP interceptors and so forth are not yet applied to the bean. A target bean is fully created first and then an AOP proxy (for example) with its interceptor chain is applied. If the target bean and the proxy are defined separately, your code can even interact with the raw target bean, bypassing the proxy. Hence, it would be inconsistent to apply the interceptors to the `init` method, because doing so would couple the lifecycle of the target bean to its proxy or interceptors and leave strange semantics when your code interacts directly with the raw target bean.
Spring容器保证在bean提供所有依赖项后立即调用配置的初始化回调。因此，初始化回调是在原始bean引用上调用的，这意味着AOP拦截器等尚未应用于bean。首先完全创建目标bean，然后应用AOP代理（例如）及其拦截器链。如果目标bean和代理是单独定义的，那么您的代码甚至可以绕过代理与原始目标bean交互。因此，将拦截器应用于 `init` 方法是不协调的，因为这样做会将目标bean的生命周期耦合到其代理或拦截器，并在代码直接与原始目标bean交互时留下奇怪的语义。

##### 组合生命周期机制

As of Spring 2.5, you have three options for controlling bean lifecycle behavior:
从Spring 2.5开始，您有三个选项来控制bean生命周期行为：

- The [`InitializingBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) and [`DisposableBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) callback interfaces
  `InitializingBean` 和 `DisposableBean` 回调接口
- Custom `init()` and `destroy()` methods
  自定义 `init()` 和 `destroy()` 方法
- The [`@PostConstruct` and `@PreDestroy` annotations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations). You can combine these mechanisms to control a given bean.
  `@PostConstruct` 和 `@PreDestroy` 注释。您可以组合这些机制来控制给定的bean。

> If multiple lifecycle mechanisms are configured for a bean and each mechanism is configured with a different method name, then each configured method is run in the order listed after this note. However, if the same method name is configured — for example, `init()` for an initialization method — for more than one of these lifecycle mechanisms, that method is run once, as explained in the [preceding section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-default-init-destroy-methods).
> 如果为bean配置了多个生命周期机制，并且每个机制都配置了不同的方法名称，那么每个配置的方法都将按照本说明后面列出的顺序运行。但是，如果为多个生命周期机制配置了相同的方法名称（例如，初始化方法为 `init()` ），则该方法将运行一次，如前一节所述。

Multiple lifecycle mechanisms configured for the same bean, with different initialization methods, are called as follows:
为同一bean配置的多个生命周期机制，使用不同的初始化方法，调用如下：

1. Methods annotated with `@PostConstruct`
   使用 `@PostConstruct` 注释的方法
2. `afterPropertiesSet()` as defined by the `InitializingBean` callback interface
   `InitializingBean` 回调接口定义的 `afterPropertiesSet()`
3. A custom configured `init()` method
   自定义配置的 `init()` 方法

Destroy methods are called in the same order:
销毁方法的调用顺序相同：

1. Methods annotated with `@PreDestroy`
   使用 `@PreDestroy` 注释的方法
2. `destroy()` as defined by the `DisposableBean` callback interface
   `DisposableBean` 回调接口定义的 `destroy()`
3. A custom configured `destroy()` method
   自定义配置的 `destroy()` 方法

##### 启动和关闭回调

The `Lifecycle` interface defines the essential methods for any object that has its own lifecycle requirements (such as starting and stopping some background process):
`Lifecycle` 接口定义了任何有自己生命周期需求的对象的基本方法（例如启动和停止一些后台进程）：

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

Any Spring-managed object may implement the `Lifecycle` interface. Then, when the `ApplicationContext` itself receives start and stop signals (for example, for a stop/restart scenario at runtime), it cascades those calls to all `Lifecycle` implementations defined within that context. It does this by delegating to a `LifecycleProcessor`, shown in the following listing:
任何Spring管理的对象都可以实现 `Lifecycle` 接口。然后，当 `ApplicationContext` 本身接收到开始和停止信号时（例如，对于运行时的停止/重新启动场景），它将这些调用级联到该上下文中定义的所有 `Lifecycle` 实现。它通过委托给一个 `LifecycleProcessor` 来实现这一点，如下面的清单所示：

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

Notice that the `LifecycleProcessor` is itself an extension of the `Lifecycle` interface. It also adds two other methods for reacting to the context being refreshed and closed.
请注意， `LifecycleProcessor` 本身就是 `Lifecycle` 接口的扩展。它还添加了另外两个方法，用于对正在刷新和关闭的上下文做出反应。

> Note that the regular `org.springframework.context.Lifecycle` interface is a plain contract for explicit start and stop notifications and does not imply auto-startup at context refresh time. For fine-grained control over auto-startup of a specific bean (including startup phases), consider implementing `org.springframework.context.SmartLifecycle` instead.
> 请注意，常规的 `org.springframework.context.Lifecycle` 接口是用于显式启动和停止通知的普通契约，并且不意味着在上下文刷新时自动启动。对于特定bean的自动启动（包括启动阶段）的细粒度控制，可以考虑实现 `org.springframework.context.SmartLifecycle` 。
>
> Also, please note that stop notifications are not guaranteed to come before destruction. On regular shutdown, all `Lifecycle` beans first receive a stop notification before the general destruction callbacks are being propagated. However, on hot refresh during a context’s lifetime or on stopped refresh attempts, only destroy methods are called.
> 此外，请注意，停止通知不能保证在销毁之前到达。在常规关闭时，所有 `Lifecycle` bean在传播常规销毁回调之前首先接收停止通知。但是，在上下文生存期内进行热刷新或停止刷新尝试时，只调用销毁方法。

The order of startup and shutdown invocations can be important. If a “depends-on” relationship exists between any two objects, the dependent side starts after its dependency, and it stops before its dependency. However, at times, the direct dependencies are unknown. You may only know that objects of a certain type should start prior to objects of another type. In those cases, the `SmartLifecycle` interface defines another option, namely the `getPhase()` method as defined on its super-interface, `Phased`. The following listing shows the definition of the `Phased` interface:
启动和关闭调用的顺序可能很重要。如果任何两个对象之间存在“依赖”关系，则依赖方在其依赖项之后开始，并在其依赖项之前停止。然而，有时，直接依赖性是未知的。您可能只知道某个类型的对象应该在另一个类型的对象之前开始。在这些情况下， `SmartLifecycle` 接口定义了另一个选项，即在其超接口 `Phased` 上定义的 `getPhase()` 方法。下面的清单显示了 `Phased` 接口的定义：

```java
public interface Phased {

    int getPhase();
}
```

The following listing shows the definition of the `SmartLifecycle` interface:
下面的清单显示了 `SmartLifecycle` 接口的定义：

```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

When starting, the objects with the lowest phase start first. When stopping, the reverse order is followed. Therefore, an object that implements `SmartLifecycle` and whose `getPhase()` method returns `Integer.MIN_VALUE` would be among the first to start and the last to stop. At the other end of the spectrum, a phase value of `Integer.MAX_VALUE` would indicate that the object should be started last and stopped first (likely because it depends on other processes to be running). When considering the phase value, it is also important to know that the default phase for any “normal” `Lifecycle` object that does not implement `SmartLifecycle` is `0`. Therefore, any negative phase value indicates that an object should start before those standard components (and stop after them). The reverse is true for any positive phase value.
启动时，相位最低的对象首先启动。当停止时，遵循相反的顺序。因此，实现了 `SmartLifecycle` 且其 `getPhase()` 方法返回 `Integer.MIN_VALUE` 的对象将是第一个启动和最后一个停止的对象。在频谱的另一端，相位值 `Integer.MAX_VALUE` 将指示对象应该最后启动并且首先停止（可能是因为它取决于要运行的其他进程）。在考虑相位值时，还必须知道任何不实现 `SmartLifecycle` 的“正常” `Lifecycle` 对象的默认相位是 `0` 。因此，任何负相位值都表示对象应在这些标准组件之前开始（并在它们之后停止）。对于任何正相位值，反之亦然。

The stop method defined by `SmartLifecycle` accepts a callback. Any implementation must invoke that callback’s `run()` method after that implementation’s shutdown process is complete. That enables asynchronous shutdown where necessary, since the default implementation of the `LifecycleProcessor` interface, `DefaultLifecycleProcessor`, waits up to its timeout value for the group of objects within each phase to invoke that callback. The default per-phase timeout is 30 seconds. You can override the default lifecycle processor instance by defining a bean named `lifecycleProcessor` within the context. If you want only to modify the timeout, defining the following would suffice:
`SmartLifecycle` 定义的stop方法接受回调。任何实现都必须在该实现的关闭过程完成后调用该回调的 `run()` 方法。这将在必要时启用异步关闭，因为 `LifecycleProcessor` 接口的默认实现 `DefaultLifecycleProcessor` 将等待每个阶段中的对象组的超时值，以调用该回调。默认的每阶段超时为30秒。您可以通过在上下文中定义名为 `lifecycleProcessor` 的bean来覆盖默认的生命周期处理器实例。如果只想修改超时，定义以下内容即可：

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

As mentioned earlier, the `LifecycleProcessor` interface defines callback methods for the refreshing and closing of the context as well. The latter drives the shutdown process as if `stop()` had been called explicitly, but it happens when the context is closing. The 'refresh' callback, on the other hand, enables another feature of `SmartLifecycle` beans. When the context is refreshed (after all objects have been instantiated and initialized), that callback is invoked. At that point, the default lifecycle processor checks the boolean value returned by each `SmartLifecycle` object’s `isAutoStartup()` method. If `true`, that object is started at that point rather than waiting for an explicit invocation of the context’s or its own `start()` method (unlike the context refresh, the context start does not happen automatically for a standard context implementation). The `phase` value and any “depends-on” relationships determine the startup order as described earlier.
如前所述， `LifecycleProcessor` 接口还定义了用于刷新和关闭上下文的回调方法。后者驱动关闭过程，就像 `stop()` 被显式调用一样，但它发生在上下文关闭时。另一方面，'refresh'回调启用了 `SmartLifecycle` beans的另一个特性。当上下文被刷新时（在所有对象都被实例化和初始化之后），该回调被调用。此时，默认的生命周期处理器检查每个 `SmartLifecycle` 对象的 `isAutoStartup()` 方法返回的布尔值。如果是 `true` ，则该对象在该点启动，而不是等待显式调用上下文或其自己的 `start()` 方法（与上下文刷新不同，对于标准上下文实现，上下文启动不会自动发生）。如前所述， `phase` 值和任何“依赖于”关系确定启动顺序。

##### 在非Web应用程序中优雅地关闭Spring IoC容器

> This section applies only to non-web applications. Spring’s web-based `ApplicationContext` implementations already have code in place to gracefully shut down the Spring IoC container when the relevant web application is shut down.
> 本节仅适用于非Web应用程序。Spring的基于web的 `ApplicationContext` 实现已经有了代码，可以在相关的web应用程序关闭时优雅地关闭Spring IoC容器。

If you use Spring’s IoC container in a non-web application environment (for example, in a rich client desktop environment), register a shutdown hook with the JVM. Doing so ensures a graceful shutdown and calls the relevant destroy methods on your singleton beans so that all resources are released. You must still configure and implement these destroy callbacks correctly.
如果您在非Web应用程序环境中使用Spring的IoC容器（例如，在富客户端桌面环境中），请向JVM注册一个shutdown钩子。这样做可以确保正常关闭，并在单例bean上调用相关的destroy方法，以便释放所有资源。您仍然必须正确地配置和实现这些销毁回调。

To register a shutdown hook, call the `registerShutdownHook()` method that is declared on the `ConfigurableApplicationContext` interface, as the following example shows:
要注册一个shutdown钩子，调用在 `ConfigurableApplicationContext` 接口上声明的 `registerShutdownHook()` 方法，如下例所示：

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

#### 1.6.2. `ApplicationContextAware` 和 `BeanNameAware`

When an `ApplicationContext` creates an object instance that implements the `org.springframework.context.ApplicationContextAware` interface, the instance is provided with a reference to that `ApplicationContext`. The following listing shows the definition of the `ApplicationContextAware` interface:
当一个 `ApplicationContext` 创建一个实现了 `org.springframework.context.ApplicationContextAware` 接口的对象实例时，该实例将被提供一个对该 `ApplicationContext` 的引用。下面的清单显示了 `ApplicationContextAware` 接口的定义：

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

Thus, beans can programmatically manipulate the `ApplicationContext` that created them, through the `ApplicationContext` interface or by casting the reference to a known subclass of this interface (such as `ConfigurableApplicationContext`, which exposes additional functionality). One use would be the programmatic retrieval of other beans. Sometimes this capability is useful. However, in general, you should avoid it, because it couples the code to Spring and does not follow the Inversion of Control style, where collaborators are provided to beans as properties. Other methods of the `ApplicationContext` provide access to file resources, publishing application events, and accessing a `MessageSource`. These additional features are described in [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction).
因此，bean可以通过编程方式操纵创建它们的 `ApplicationContext` ，通过 `ApplicationContext` 接口或通过将引用转换到此接口的已知子类（例如 `ConfigurableApplicationContext` ，它公开了其他功能）。一种用途是编程检索其他bean。有时这种能力是有用的。但是，一般来说，您应该避免它，因为它将代码耦合到Spring，并且不遵循控制反转样式，其中协作者作为属性提供给bean。 `ApplicationContext` 的其他方法提供对文件资源的访问、发布应用程序事件和访问 `MessageSource` 。这些附加功能在[ `ApplicationContext` 的附加功能](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction)中进行了说明。

Autowiring is another alternative to obtain a reference to the `ApplicationContext`. The *traditional* `constructor` and `byType` autowiring modes (as described in [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)) can provide a dependency of type `ApplicationContext` for a constructor argument or a setter method parameter, respectively. For more flexibility, including the ability to autowire fields and multiple parameter methods, use the annotation-based autowiring features. If you do, the `ApplicationContext` is autowired into a field, constructor argument, or method parameter that expects the `ApplicationContext` type if the field, constructor, or method in question carries the `@Autowired` annotation. For more information, see [Using `@Autowired`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation).
自动装配是获取对 `ApplicationContext` 的引用的另一种替代方法。传统的 `constructor` 和 `byType` 自动装配模式（如自动装配协作者中所述）可以分别为构造函数参数或setter方法参数提供类型为 `ApplicationContext` 的依赖项。要获得更大的灵活性，包括自动关联字段和多参数方法的能力，请使用基于注释的自动装配功能。如果你这样做， `ApplicationContext` 会自动连接到一个字段、构造函数参数或方法参数中，如果该字段、构造函数或方法带有 `@Autowired` 注释，则该字段、构造函数参数或方法参数需要 `ApplicationContext` 类型。有关更多信息，请参阅[使用 `@Autowired`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation) 。

When an `ApplicationContext` creates a class that implements the `org.springframework.beans.factory.BeanNameAware` interface, the class is provided with a reference to the name defined in its associated object definition. The following listing shows the definition of the BeanNameAware interface:
当一个 `ApplicationContext` 创建一个实现 `org.springframework.beans.factory.BeanNameAware` 接口的类时，该类将被提供一个对其关联对象定义中定义的名称的引用。下面的清单显示了BeanNameAware接口的定义：

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

The callback is invoked after population of normal bean properties but before an initialization callback such as `InitializingBean.afterPropertiesSet()` or a custom init-method.
在填充普通bean属性之后，但在初始化回调（如 `InitializingBean.afterPropertiesSet()` 或自定义init-method）之前调用回调。

#### 1.6.3.其他 `Aware` 接口

Besides `ApplicationContextAware` and `BeanNameAware` (discussed [earlier](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)), Spring offers a wide range of `Aware` callback interfaces that let beans indicate to the container that they require a certain infrastructure dependency. As a general rule, the name indicates the dependency type. The following table summarizes the most important `Aware` interfaces:
除了 `ApplicationContextAware` 和 `BeanNameAware` （前面讨论过），Spring还提供了广泛的 `Aware` 回调接口，让bean向容器指示它们需要特定的基础设施依赖。作为一般规则，名称指示依赖项类型。下表总结了最重要的 `Aware` 接口：

| Name                             | Injected Dependency 注入的依赖性                             | Explained in… 解释于...                                      |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ApplicationContextAware`        | Declaring `ApplicationContext`. 声明 `ApplicationContext` 。 | [`ApplicationContextAware` and `BeanNameAware ApplicationContextAware 和 BeanNameAware `](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | Event publisher of the enclosing `ApplicationContext`. 封闭 `ApplicationContext` 的事件发布者。 | [Additional Capabilities of the `ApplicationContext ApplicationContext 的附加功能 `](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction) |
| `BeanClassLoaderAware`           | Class loader used to load the bean classes. 用于加载bean类的类加载器。 | [Instantiating Beans 实例化Bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| `BeanFactoryAware`               | Declaring `BeanFactory`. 声明 `BeanFactory` 。               | [The `BeanFactory` API `BeanFactory` API](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory) |
| `BeanNameAware`                  | Name of the declaring bean. 声明bean的名称。                 | [`ApplicationContextAware` and `BeanNameAware ApplicationContextAware 和 BeanNameAware `](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `LoadTimeWeaverAware`            | Defined weaver for processing class definition at load time. 定义了在加载时处理类定义的织入器。 | [Load-time Weaving with AspectJ in the Spring Framework 在Spring框架中使用AspectJ进行加载时编织](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw) |
| `MessageSourceAware`             | Configured strategy for resolving messages (with support for parameterization and internationalization). 已配置的消息解析策略（支持参数化和国际化）。 | [Additional Capabilities of the `ApplicationContext ApplicationContext 的附加功能 `](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction) |
| `NotificationPublisherAware`     | Spring JMX notification publisher. Spring JMX通知发布器。    | [Notifications](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jmx-notifications) |
| `ResourceLoaderAware`            | Configured loader for low-level access to resources. 配置的加载程序用于对资源的低级访问。 | [Resources](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources) |
| `ServletConfigAware`             | Current `ServletConfig` the container runs in. Valid only in a web-aware Spring `ApplicationContext`. 容器运行的当前 `ServletConfig` 。仅在Web感知Spring `ApplicationContext` 中有效。 | [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc) |
| `ServletContextAware`            | Current `ServletContext` the container runs in. Valid only in a web-aware Spring `ApplicationContext`. 容器运行的当前 `ServletContext` 。仅在Web感知Spring `ApplicationContext` 中有效。 | [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc) |

Note again that using these interfaces ties your code to the Spring API and does not follow the Inversion of Control style. As a result, we recommend them for infrastructure beans that require programmatic access to the container.
再次注意，使用这些接口将您的代码绑定到Spring API，并且不遵循控制反转风格。因此，我们建议将它们用于需要对容器进行编程访问的基础设施bean。

### 1.7. Bean定义继承

A bean definition can contain a lot of configuration information, including constructor arguments, property values, and container-specific information, such as the initialization method, a static factory method name, and so on. A child bean definition inherits configuration data from a parent definition. The child definition can override some values or add others as needed. Using parent and child bean definitions can save a lot of typing. Effectively, this is a form of templating.
bean定义可以包含大量配置信息，包括构造函数参数、属性值和容器特定信息，如初始化方法、静态工厂方法名称等。子bean定义从父定义继承配置数据。子定义可以根据需要覆盖某些值或添加其他值。使用父bean和子bean定义可以保存大量的输入。实际上，这是一种模板化的形式。

If you work with an `ApplicationContext` interface programmatically, child bean definitions are represented by the `ChildBeanDefinition` class. Most users do not work with them on this level. Instead, they configure bean definitions declaratively in a class such as the `ClassPathXmlApplicationContext`. When you use XML-based configuration metadata, you can indicate a child bean definition by using the `parent` attribute, specifying the parent bean as the value of this attribute. The following example shows how to do so:
如果以编程方式使用 `ApplicationContext` 接口，则子bean定义由 `ChildBeanDefinition` 类表示。大多数用户在这个级别上不使用它们。相反，它们在一个类中以声明方式配置bean定义，例如 `ClassPathXmlApplicationContext` 。当您使用基于XML的配置元数据时，您可以通过使用 `parent` 属性来指示子bean定义，并将父bean指定为该属性的值。下面的示例说明如何执行此操作：

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  (1)
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

> Note the `parent` attribute. 注意 `parent` 属性。

A child bean definition uses the bean class from the parent definition if none is specified but can also override it. In the latter case, the child bean class must be compatible with the parent (that is, it must accept the parent’s property values).
如果未指定任何Bean类，则子Bean定义将使用父Bean定义中的Bean类，但也可以覆盖父Bean类。在后一种情况下，子Bean类必须与父Bean类兼容（即，它必须接受父Bean类的属性值）。

A child bean definition inherits scope, constructor argument values, property values, and method overrides from the parent, with the option to add new values. Any scope, initialization method, destroy method, or `static` factory method settings that you specify override the corresponding parent settings.
子bean定义从父bean继承作用域、构造函数参数值、属性值和方法重写，并可以选择添加新值。您指定的任何范围、初始化方法、销毁方法或 `static` 工厂方法设置将覆盖相应的父设置。

The remaining settings are always taken from the child definition: depends on, autowire mode, dependency check, singleton, and lazy init.
其余设置始终从子定义中获取：depends on、autowire模式、dependency check、singleton和lazy init。

The preceding example explicitly marks the parent bean definition as abstract by using the `abstract` attribute. If the parent definition does not specify a class, explicitly marking the parent bean definition as `abstract` is required, as the following example shows:
前面的示例使用 `abstract` 属性显式地将父bean定义标记为抽象。如果父定义没有指定类，则需要显式地将父bean定义标记为 `abstract` ，如以下示例所示：

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

The parent bean cannot be instantiated on its own because it is incomplete, and it is also explicitly marked as `abstract`. When a definition is `abstract`, it is usable only as a pure template bean definition that serves as a parent definition for child definitions. Trying to use such an `abstract` parent bean on its own, by referring to it as a ref property of another bean or doing an explicit `getBean()` call with the parent bean ID returns an error. Similarly, the container’s internal `preInstantiateSingletons()` method ignores bean definitions that are defined as abstract.
父bean不能自己实例化，因为它是不完整的，并且它也被显式地标记为 `abstract` 。当一个定义是 `abstract` 时，它只能作为一个纯模板bean定义，作为子定义的父定义。尝试单独使用这样的 `abstract` 父bean，将其作为另一个bean的ref属性引用，或者使用父bean ID进行显式的 `getBean()` 调用，都会返回错误。类似地，容器的内部 `preInstantiateSingletons()` 方法忽略定义为抽象的bean定义。

> `ApplicationContext` pre-instantiates all singletons by default. Therefore, it is important (at least for singleton beans) that if you have a (parent) bean definition which you intend to use only as a template, and this definition specifies a class, you must make sure to set the *abstract* attribute to *true*, otherwise the application context will actually (attempt to) pre-instantiate the `abstract` bean.
> `ApplicationContext` 默认预实例化所有单例。因此，重要的是（至少对于单例bean），如果您有一个（父）bean定义，您打算仅用作模板，并且此定义指定了一个类，则必须确保将abstract属性设置为true，否则应用程序上下文实际上将（尝试）预实例化 `abstract` bean。

### 1.8.容器扩展点

Typically, an application developer does not need to subclass `ApplicationContext` implementation classes. Instead, the Spring IoC container can be extended by plugging in implementations of special integration interfaces. The next few sections describe these integration interfaces.
通常，应用程序开发人员不需要子类化 `ApplicationContext` 实现类。相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。接下来的几节将描述这些集成接口。

#### 1.8.1.使用 `BeanPostProcessor` 自定义Bean

The `BeanPostProcessor` interface defines callback methods that you can implement to provide your own (or override the container’s default) instantiation logic, dependency resolution logic, and so forth. If you want to implement some custom logic after the Spring container finishes instantiating, configuring, and initializing a bean, you can plug in one or more custom `BeanPostProcessor` implementations.
`BeanPostProcessor`接口定义了回调方法，您可以实现这些方法来提供自己的（或覆盖容器的默认）实例化逻辑、依赖项解析逻辑等。如果你想在Spring容器完成实例化、配置和初始化bean之后实现一些自定义逻辑，你可以插入一个或多个自定义 `BeanPostProcessor` 实现。

You can configure multiple `BeanPostProcessor` instances, and you can control the order in which these `BeanPostProcessor` instances run by setting the `order` property. You can set this property only if the `BeanPostProcessor` implements the `Ordered` interface. If you write your own `BeanPostProcessor`, you should consider implementing the `Ordered` interface, too. For further details, see the javadoc of the [`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html) and [`Ordered`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/Ordered.html) interfaces. See also the note on [programmatic registration of `BeanPostProcessor` instances](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-programmatically-registering-beanpostprocessors).
您可以配置多个 `BeanPostProcessor` 实例，也可以通过设置 `order` 属性来控制这些 `BeanPostProcessor` 实例的运行顺序。只有当 `BeanPostProcessor` 实现了 `Ordered` 接口时，才可以设置此属性。如果你自己编写了 `BeanPostProcessor` ，你也应该考虑实现 [`Ordered`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/Ordered.html) 接口。有关更多详细信息，请参阅 [`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html) 和 [`Ordered`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/Ordered.html) 接口的javadoc。另请参阅[ `BeanPostProcessor` 实例的编程注册说明](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-programmatically-registering-beanpostprocessors)。

> `BeanPostProcessor` instances operate on bean (or object) instances. That is, the Spring IoC container instantiates a bean instance and then `BeanPostProcessor` instances do their work.
> `BeanPostProcessor` 实例对bean（或object）实例进行操作。也就是说，Spring IoC容器实例化一个bean实例，然后 `BeanPostProcessor` 实例完成它们的工作。
>
> `BeanPostProcessor` instances are scoped per-container. This is relevant only if you use container hierarchies. If you define a `BeanPostProcessor` in one container, it post-processes only the beans in that container. In other words, beans that are defined in one container are not post-processed by a `BeanPostProcessor` defined in another container, even if both containers are part of the same hierarchy.
> `BeanPostProcessor` 实例的作用域为每个容器。只有在使用容器层次结构时，这才是相关的。如果在一个容器中定义 `BeanPostProcessor` ，它只对该容器中的bean进行后处理。换句话说，在一个容器中定义的bean不会被另一个容器中定义的 `BeanPostProcessor` 进行后处理，即使两个容器都是同一层次结构的一部分。
>
> To change the actual bean definition (that is, the blueprint that defines the bean), you instead need to use a `BeanFactoryPostProcessor`, as described in [Customizing Configuration Metadata with a `BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-factory-postprocessors).
> 要更改实际的bean定义（即定义bean的蓝图），则需要使用 `BeanFactoryPostProcessor` ，如[使用 `BeanFactoryPostProcessor` 自定义配置元数据](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-factory-postprocessors)中所述。

The `org.springframework.beans.factory.config.BeanPostProcessor` interface consists of exactly two callback methods. When such a class is registered as a post-processor with the container, for each bean instance that is created by the container, the post-processor gets a callback from the container both before container initialization methods (such as `InitializingBean.afterPropertiesSet()` or any declared `init` method) are called, and after any bean initialization callbacks. The post-processor can take any action with the bean instance, including ignoring the callback completely. A bean post-processor typically checks for callback interfaces, or it may wrap a bean with a proxy. Some Spring AOP infrastructure classes are implemented as bean post-processors in order to provide proxy-wrapping logic.
`org.springframework.beans.factory.config.BeanPostProcessor` 接口由两个回调方法组成。当这样的类作为后处理器注册到容器中时，对于容器创建的每个bean实例，后处理器在调用容器初始化方法（例如 `InitializingBean.afterPropertiesSet()` 或任何声明的 `init` 方法）之前和任何bean初始化回调之后都会从容器中获得回调。后处理器可以对bean实例执行任何操作，包括完全忽略回调。bean后处理器通常检查回调接口，或者它可以用代理包装bean。一些SpringAOP基础设施类被实现为bean后处理器，以提供代理包装逻辑。

An `ApplicationContext` automatically detects any beans that are defined in the configuration metadata that implement the `BeanPostProcessor` interface. The `ApplicationContext` registers these beans as post-processors so that they can be called later, upon bean creation. Bean post-processors can be deployed in the container in the same fashion as any other beans.
`ApplicationContext` 自动检测在配置元数据中定义的实现 `BeanPostProcessor` 接口的任何bean。 `ApplicationContext` 将这些bean注册为后处理器，以便稍后在创建bean时调用它们。Bean后处理器可以以与任何其他Bean相同的方式部署在容器中。

Note that, when declaring a `BeanPostProcessor` by using an `@Bean` factory method on a configuration class, the return type of the factory method should be the implementation class itself or at least the `org.springframework.beans.factory.config.BeanPostProcessor` interface, clearly indicating the post-processor nature of that bean. Otherwise, the `ApplicationContext` cannot autodetect it by type before fully creating it. Since a `BeanPostProcessor` needs to be instantiated early in order to apply to the initialization of other beans in the context, this early type detection is critical.
请注意，当在配置类上使用 `@Bean` 工厂方法声明 `BeanPostProcessor` 时，工厂方法的返回类型应该是实现类本身或至少是 `org.springframework.beans.factory.config.BeanPostProcessor` 接口，清楚地表明该bean的后处理器性质。否则， `ApplicationContext` 无法在完全创建之前根据类型自动检测它。由于需要尽早实例化 `BeanPostProcessor` 以应用于上下文中其他bean的初始化，因此早期类型检测是至关重要的。

> Programmatically registering `BeanPostProcessor` instances
> 以编程方式注册 `BeanPostProcessor` 实例
>
> While the recommended approach for `BeanPostProcessor` registration is through `ApplicationContext` auto-detection (as described earlier), you can register them programmatically against a `ConfigurableBeanFactory` by using the `addBeanPostProcessor` method. This can be useful when you need to evaluate conditional logic before registration or even for copying bean post processors across contexts in a hierarchy. Note, however, that `BeanPostProcessor` instances added programmatically do not respect the `Ordered` interface. Here, it is the order of registration that dictates the order of execution. Note also that `BeanPostProcessor` instances registered programmatically are always processed before those registered through auto-detection, regardless of any explicit ordering.
> 虽然推荐的 `BeanPostProcessor` 注册方法是通过 `ApplicationContext` 自动检测（如前所述），但您可以通过使用 `addBeanPostProcessor` 方法以编程方式针对 `ConfigurableBeanFactory` 注册它们。当您需要在注册之前评估条件逻辑时，或者甚至在层次结构中跨上下文复制bean后处理器时，这可能很有用。但是，请注意，以编程方式添加的 `BeanPostProcessor` 实例不支持 `Ordered` 接口。在这里，登记的顺序决定了执行的顺序。还请注意，以编程方式注册的 `BeanPostProcessor` 实例总是在通过自动检测注册的实例之前处理，无论任何显式顺序如何。

> `BeanPostProcessor` instances and AOP auto-proxying
> `BeanPostProcessor` 实例和AOP自动代理
>
> Classes that implement the `BeanPostProcessor` interface are special and are treated differently by the container. All `BeanPostProcessor` instances and beans that they directly reference are instantiated on startup, as part of the special startup phase of the `ApplicationContext`. Next, all `BeanPostProcessor` instances are registered in a sorted fashion and applied to all further beans in the container. Because AOP auto-proxying is implemented as a `BeanPostProcessor` itself, neither `BeanPostProcessor` instances nor the beans they directly reference are eligible for auto-proxying and, thus, do not have aspects woven into them.
> 实现了 `BeanPostProcessor` 接口的类是特殊的，并且被容器区别对待。它们直接引用的所有 `BeanPostProcessor` 实例和bean都在启动时实例化，作为 `ApplicationContext` 的特殊启动阶段的一部分。接下来，所有 `BeanPostProcessor` 实例都以排序的方式注册，并应用于容器中的所有其他bean。因为AOP自动代理是作为 `BeanPostProcessor` 本身实现的，所以 `BeanPostProcessor` 实例和它们直接引用的bean都不符合自动代理的条件，因此，没有将方面织入其中。
>
> For any such bean, you should see an informational log message: `Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)`.
> 对于任何这样的bean，您都应该看到一条信息性日志消息： `Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)` 。
>
> If you have beans wired into your `BeanPostProcessor` by using autowiring or `@Resource` (which may fall back to autowiring), Spring might access unexpected beans when searching for type-matching dependency candidates and, therefore, make them ineligible for auto-proxying or other kinds of bean post-processing. For example, if you have a dependency annotated with `@Resource` where the field or setter name does not directly correspond to the declared name of a bean and no name attribute is used, Spring accesses other beans for matching them by type.
> 如果您通过使用自动装配或 `@Resource` （可能会退回到自动装配）将bean连接到您的 `BeanPostProcessor` 中，Spring可能会在搜索类型匹配依赖候选项时访问意外的bean，因此，使它们不符合自动代理或其他类型的bean后处理。例如，如果您有一个使用 `@Resource` 注释的依赖项，其中字段或setter名称不直接对应于bean的声明名称，并且没有使用name属性，则Spring访问其他bean以按类型匹配它们。

The following examples show how to write, register, and use `BeanPostProcessor` instances in an `ApplicationContext`.
以下示例显示如何在 `ApplicationContext` 中写入、注册和使用 `BeanPostProcessor` 实例。

##### 示例：Hello World， `BeanPostProcessor` -style

This first example illustrates basic usage. The example shows a custom `BeanPostProcessor` implementation that invokes the `toString()` method of each bean as it is created by the container and prints the resulting string to the system console.
第一个示例说明了基本用法。该示例显示了一个自定义的 `BeanPostProcessor` 实现，它在容器创建每个bean时调用其 `toString()` 方法，并将结果字符串打印到系统控制台。

The following listing shows the custom `BeanPostProcessor` implementation class definition:
下面的清单显示了自定义的 `BeanPostProcessor` 实现类定义：

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

The following `beans` element uses the `InstantiationTracingBeanPostProcessor`:
下面的 `beans` 元素使用 `InstantiationTracingBeanPostProcessor` ：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        https://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

Notice how the `InstantiationTracingBeanPostProcessor` is merely defined. It does not even have a name, and, because it is a bean, it can be dependency-injected as you would any other bean. (The preceding configuration also defines a bean that is backed by a Groovy script. The Spring dynamic language support is detailed in the chapter entitled [Dynamic Language Support](https://docs.spring.io/spring-framework/docs/current/reference/html/languages.html#dynamic-language).)
注意 `InstantiationTracingBeanPostProcessor` 是如何定义的。它甚至没有名称，而且因为它是一个bean，所以可以像对任何其他bean一样进行依赖注入。(The前面的配置还定义了一个由Groovy脚本支持的bean。Spring动态语言支持在[动态语言支持](https://docs.spring.io/spring-framework/docs/current/reference/html/languages.html#dynamic-language)一章中详细介绍。

The following Java application runs the preceding code and configuration:
以下Java应用程序运行上述代码和配置：

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = ctx.getBean("messenger", Messenger.class);
        System.out.println(messenger);
    }

}
```

The output of the preceding application resembles the following:
上述应用程序的输出类似于以下内容：

```
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```

##### 示例： `AutowiredAnnotationBeanPostProcessor`

Using callback interfaces or annotations in conjunction with a custom `BeanPostProcessor` implementation is a common means of extending the Spring IoC container. An example is Spring’s `AutowiredAnnotationBeanPostProcessor` — a `BeanPostProcessor` implementation that ships with the Spring distribution and autowires annotated fields, setter methods, and arbitrary config methods.
使用回调接口或注释以及自定义的 `BeanPostProcessor` 实现是扩展Spring IoC容器的常用方法。一个例子是Spring的 `AutowiredAnnotationBeanPostProcessor` -一个与Spring发行版一起发布的 `BeanPostProcessor` 实现，以及autowires注释字段、setter方法和任意配置方法。

#### 1.8.2.使用 `BeanFactoryPostProcessor` 自定义配置元数据

The next extension point that we look at is the `org.springframework.beans.factory.config.BeanFactoryPostProcessor`. The semantics of this interface are similar to those of the `BeanPostProcessor`, with one major difference: `BeanFactoryPostProcessor` operates on the bean configuration metadata. That is, the Spring IoC container lets a `BeanFactoryPostProcessor` read the configuration metadata and potentially change it *before* the container instantiates any beans other than `BeanFactoryPostProcessor` instances.
我们要看的下一个扩展点是 `org.springframework.beans.factory.config.BeanFactoryPostProcessor` 。此接口的语义与 `BeanPostProcessor` 的语义相似，但有一个主要区别： `BeanFactoryPostProcessor` 操作bean配置元数据。也就是说，Spring IoC容器允许 `BeanFactoryPostProcessor` 读取配置元数据，并可能在容器实例化除 `BeanFactoryPostProcessor` 实例之外的任何bean之前对其进行更改。

You can configure multiple `BeanFactoryPostProcessor` instances, and you can control the order in which these `BeanFactoryPostProcessor` instances run by setting the `order` property. However, you can only set this property if the `BeanFactoryPostProcessor` implements the `Ordered` interface. If you write your own `BeanFactoryPostProcessor`, you should consider implementing the `Ordered` interface, too. See the javadoc of the [`BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html) and [`Ordered`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/Ordered.html) interfaces for more details.
您可以配置多个 `BeanFactoryPostProcessor` 实例，也可以通过设置 `order` 属性来控制这些 `BeanFactoryPostProcessor` 实例的运行顺序。但是，只有当 `BeanFactoryPostProcessor` 实现了 `Ordered` 接口时，才可以设置此属性。如果你自己编写了 `BeanFactoryPostProcessor` ，你也应该考虑实现 `Ordered` 接口。有关更多详细信息，请参阅 [`BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html) 和 [`Ordered`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/Ordered.html) 接口的javadoc。

>  If you want to change the actual bean instances (that is, the objects that are created from the configuration metadata), then you instead need to use a `BeanPostProcessor` (described earlier in [Customizing Beans by Using a `BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp)). While it is technically possible to work with bean instances within a `BeanFactoryPostProcessor` (for example, by using `BeanFactory.getBean()`), doing so causes premature bean instantiation, violating the standard container lifecycle. This may cause negative side effects, such as bypassing bean post processing. 
>  如果要更改实际的Bean实例（即从配置元数据创建的对象），则需要使用 `BeanPostProcessor` （在前面的使用 `BeanPostProcessor` 自定义Bean中描述）。虽然在技术上可以使用 `BeanFactoryPostProcessor` 中的bean实例（例如，通过使用 `BeanFactory.getBean()` ），但这样做会导致过早的bean实例化，违反了标准的容器生命周期。这可能会导致负面的副作用，例如绕过bean后处理。
>
>  Also, `BeanFactoryPostProcessor` instances are scoped per-container. This is only relevant if you use container hierarchies. If you define a `BeanFactoryPostProcessor` in one container, it is applied only to the bean definitions in that container. Bean definitions in one container are not post-processed by `BeanFactoryPostProcessor` instances in another container, even if both containers are part of the same hierarchy. 
>  此外， `BeanFactoryPostProcessor` 实例的作用域为每个容器。这仅在使用容器层次结构时才相关。如果你在一个容器中定义了一个 `BeanFactoryPostProcessor` ，那么它只应用于该容器中的bean定义。一个容器中的Bean定义不会被另一个容器中的 `BeanFactoryPostProcessor` 实例进行后处理，即使两个容器都是同一层次结构的一部分。

A bean factory post-processor is automatically run when it is declared inside an `ApplicationContext`, in order to apply changes to the configuration metadata that define the container. Spring includes a number of predefined bean factory post-processors, such as `PropertyOverrideConfigurer` and `PropertySourcesPlaceholderConfigurer`. You can also use a custom `BeanFactoryPostProcessor` — for example, to register custom property editors.
bean工厂后处理器在 `ApplicationContext` 中声明时会自动运行，以便将更改应用于定义容器的配置元数据。Spring包含许多预定义的bean工厂后处理器，例如 `PropertyOverrideConfigurer` 和 `PropertySourcesPlaceholderConfigurer` 。您还可以使用自定义 `BeanFactoryPostProcessor` -例如，注册自定义属性编辑器。

An `ApplicationContext` automatically detects any beans that are deployed into it that implement the `BeanFactoryPostProcessor` interface. It uses these beans as bean factory post-processors, at the appropriate time. You can deploy these post-processor beans as you would any other bean.
`ApplicationContext` 会自动检测部署到其中并实现了 `BeanFactoryPostProcessor` 接口的任何bean。它在适当的时候使用这些bean作为bean工厂的后处理器。您可以像部署任何其他bean一样部署这些后处理器bean。

> As with `BeanPostProcessor`s , you typically do not want to configure `BeanFactoryPostProcessor`s for lazy initialization. If no other bean references a `Bean(Factory)PostProcessor`, that post-processor will not get instantiated at all. Thus, marking it for lazy initialization will be ignored, and the `Bean(Factory)PostProcessor` will be instantiated eagerly even if you set the `default-lazy-init` attribute to `true` on the declaration of your `<beans />` element.
> 与 `BeanPostProcessor` 一样，您通常不希望将 `BeanFactoryPostProcessor` 配置为惰性初始化。如果没有其他bean引用 `Bean(Factory)PostProcessor` ，则该后处理器根本不会被实例化。因此，将其标记为惰性初始化将被忽略，并且即使您在 `<beans />` 元素的声明中将 `default-lazy-init` 属性设置为 `true` ， `Bean(Factory)PostProcessor` 也将被立即实例化。

##### 示例：类名替换 `PropertySourcesPlaceholderConfigurer`

You can use the `PropertySourcesPlaceholderConfigurer` to externalize property values from a bean definition in a separate file by using the standard Java `Properties` format. Doing so enables the person deploying an application to customize environment-specific properties, such as database URLs and passwords, without the complexity or risk of modifying the main XML definition file or files for the container.
您可以使用标准的Java `Properties` 格式，使用 `PropertySourcesPlaceholderConfigurer` 将bean定义中的属性值外部化到单独的文件中。这样做使部署应用程序的人员能够自定义特定于环境的属性（如数据库URL和密码），而无需修改容器的一个或多个主XML定义文件，既不复杂，也没有风险。

Consider the following XML-based configuration metadata fragment, where a `DataSource` with placeholder values is defined:
考虑以下基于XML的配置元数据片段，其中定义了具有占位符值的 `DataSource` ：

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

The example shows properties configured from an external `Properties` file. At runtime, a `PropertySourcesPlaceholderConfigurer` is applied to the metadata that replaces some properties of the DataSource. The values to replace are specified as placeholders of the form `${property-name}`, which follows the Ant and log4j and JSP EL style.
该示例显示了从外部 `Properties` 文件配置的属性。在运行时，将 `PropertySourcesPlaceholderConfigurer` 应用于元数据，以替换DataSource的某些属性。要替换的值被指定为格式为 `${property-name}` 的占位符，它遵循Ant、log4j和JSPEL样式。

The actual values come from another file in the standard Java `Properties` format:
实际值来自另一个标准Java `Properties` 格式的文件：

```properties
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

Therefore, the `${jdbc.username}` string is replaced at runtime with the value, 'sa', and the same applies for other placeholder values that match keys in the properties file. The `PropertySourcesPlaceholderConfigurer` checks for placeholders in most properties and attributes of a bean definition. Furthermore, you can customize the placeholder prefix and suffix.
因此， `${jdbc.username}` 字符串在运行时被替换为值“sa”，这同样适用于与属性文件中的键匹配的其他占位符值。 `PropertySourcesPlaceholderConfigurer` 检查bean定义的大多数属性和特性中的占位符。此外，还可以自定义占位符前缀和后缀。

With the `context` namespace introduced in Spring 2.5, you can configure property placeholders with a dedicated configuration element. You can provide one or more locations as a comma-separated list in the `location` attribute, as the following example shows:
使用Spring 2.5中引入的 `context` 命名空间，您可以使用专用的configuration元素配置属性占位符。您可以在 `location` 属性中以逗号分隔的列表形式提供一个或多个位置，如下例所示：

```xml
<context:property-placeholder location="classpath:com/something/jdbc.properties"/>
```

The `PropertySourcesPlaceholderConfigurer` not only looks for properties in the `Properties` file you specify. By default, if it cannot find a property in the specified properties files, it checks against Spring `Environment` properties and regular Java `System` properties.
`PropertySourcesPlaceholderConfigurer` 不仅在您指定的 `Properties` 文件中查找属性。默认情况下，如果在指定的属性文件中找不到属性，它会检查Spring `Environment` 属性和常规Java `System` 属性。

> You can use the `PropertySourcesPlaceholderConfigurer` to substitute class names, which is sometimes useful when you have to pick a particular implementation class at runtime. The following example shows how to do so:
> 您可以使用 `PropertySourcesPlaceholderConfigurer` 来替换类名，当您必须在运行时选择特定的实现类时，这有时很有用。下面的示例说明如何执行此操作：
>
> ```xml
> <bean class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">
>     <property name="locations">
>         <value>classpath:com/something/strategy.properties</value>
>     </property>
>     <property name="properties">
>         <value>custom.strategy.class=com.something.DefaultStrategy</value>
>     </property>
> </bean>
> 
> <bean id="serviceStrategy" class="${custom.strategy.class}"/>
> ```
>
> If the class cannot be resolved at runtime to a valid class, resolution of the bean fails when it is about to be created, which is during the `preInstantiateSingletons()` phase of an `ApplicationContext` for a non-lazy-init bean.
> 如果类在运行时不能被解析为有效类，那么在将要创建bean时，对bean的解析将失败，这是在非惰性初始化bean的 `ApplicationContext` 的 `preInstantiateSingletons()` 阶段。

##### 示例： `PropertyOverrideConfigurer`

The `PropertyOverrideConfigurer`, another bean factory post-processor, resembles the `PropertySourcesPlaceholderConfigurer`, but unlike the latter, the original definitions can have default values or no values at all for bean properties. If an overriding `Properties` file does not have an entry for a certain bean property, the default context definition is used.
另一个bean工厂后处理器 `PropertyOverrideConfigurer` 与 `PropertySourcesPlaceholderConfigurer` 类似，但与后者不同的是，原始定义可以为bean属性提供默认值或根本没有值。如果覆盖的 `Properties` 文件没有特定bean属性的条目，则使用默认上下文定义。

Note that the bean definition is not aware of being overridden, so it is not immediately obvious from the XML definition file that the override configurer is being used. In case of multiple `PropertyOverrideConfigurer` instances that define different values for the same bean property, the last one wins, due to the overriding mechanism.
请注意，bean定义并不知道被覆盖，因此从XML定义文件中并不能立即看出正在使用覆盖配置器。如果多个 `PropertyOverrideConfigurer` 实例为同一个bean属性定义了不同的值，由于覆盖机制，最后一个实例获胜。

Properties file configuration lines take the following format:
属性文件配置行采用以下格式：

```properties
beanName.property=value
```

The following listing shows an example of the format:
下面的清单显示了该格式的示例：

```properties
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```

This example file can be used with a container definition that contains a bean called `dataSource` that has `driver` and `url` properties.
此示例文件可以与容器定义一起使用，该容器定义包含名为 `dataSource` 的bean，该bean具有 `driver` 和 `url` 属性。

Compound property names are also supported, as long as every component of the path except the final property being overridden is already non-null (presumably initialized by the constructors). In the following example, the `sammy` property of the `bob` property of the `fred` property of the `tom` bean is set to the scalar value `123`:
也支持复合属性名，只要路径的每个组件（除了被覆盖的final属性）都已经是非空的（可能是由构造函数初始化的）。在下面的示例中， `tom` bean的 `fred` 属性的 `bob` 属性的 `sammy` 属性被设置为标量值 `123` ：

```properties
tom.fred.bob.sammy=123
```

> Specified override values are always literal values. They are not translated into bean references. This convention also applies when the original value in the XML bean definition specifies a bean reference.
> 指定的覆盖值始终为文字值。它们不会被转换为bean引用。当XML bean定义中的原始值指定bean引用时，此约定也适用。

With the `context` namespace introduced in Spring 2.5, it is possible to configure property overriding with a dedicated configuration element, as the following example shows:
使用Spring 2.5中引入的 `context` 命名空间，可以使用专用的configuration元素配置property overriding，如以下示例所示：

```xml
<context:property-override location="classpath:override.properties"/>
```

#### 1.8.3.使用 `FactoryBean` 自定义实例化逻辑

You can implement the `org.springframework.beans.factory.FactoryBean` interface for objects that are themselves factories.
您可以为本身就是工厂的对象实现 `org.springframework.beans.factory.FactoryBean` 接口。

The `FactoryBean` interface is a point of pluggability into the Spring IoC container’s instantiation logic. If you have complex initialization code that is better expressed in Java as opposed to a (potentially) verbose amount of XML, you can create your own `FactoryBean`, write the complex initialization inside that class, and then plug your custom `FactoryBean` into the container.
`FactoryBean` 接口是可插入Spring IoC容器的实例化逻辑的一个点。如果您有复杂的初始化代码，而不是（可能）冗长的XML，那么您可以创建自己的 `FactoryBean` ，在该类中编写复杂的初始化，然后将自定义的 `FactoryBean` 插入容器。

The `FactoryBean<T>` interface provides three methods:
`FactoryBean<T>` 接口提供三种方法：

- `T getObject()`: Returns an instance of the object this factory creates. The instance can possibly be shared, depending on whether this factory returns singletons or prototypes.
  `T getObject()` ：返回此工厂创建的对象的实例。这个实例可能是共享的，这取决于这个工厂返回的是单例还是原型。
- `boolean isSingleton()`: Returns `true` if this `FactoryBean` returns singletons or `false` otherwise. The default implementation of this method returns `true`.
  `boolean isSingleton()` ：如果此 `FactoryBean` 返回单例，则返回 `true` ，否则返回 `false` 。这个方法的默认实现返回 `true` 。
- `Class<?> getObjectType()`: Returns the object type returned by the `getObject()` method or `null` if the type is not known in advance.
  `Class<?> getObjectType()` ：返回由 `getObject()` 方法返回的对象类型，如果类型事先未知，则返回 `null` 。

The `FactoryBean` concept and interface are used in a number of places within the Spring Framework. More than 50 implementations of the `FactoryBean` interface ship with Spring itself.
`FactoryBean` 概念和接口在Spring框架中的许多地方使用。超过50个 `FactoryBean` 接口的实现与Spring本身一起发布。

When you need to ask a container for an actual `FactoryBean` instance itself instead of the bean it produces, prefix the bean’s `id` with the ampersand symbol (`&`) when calling the `getBean()` method of the `ApplicationContext`. So, for a given `FactoryBean` with an `id` of `myBean`, invoking `getBean("myBean")` on the container returns the product of the `FactoryBean`, whereas invoking `getBean("&myBean")` returns the `FactoryBean` instance itself.
当您需要向容器请求实际的 `FactoryBean` 实例本身而不是它生成的bean时，在调用 `ApplicationContext` 的 `getBean()` 方法时，使用&符号（ `&` ）前缀bean的 `id` 。因此，对于给定的 `FactoryBean` 和 `myBean` 的 `id` ，在容器上调用 `getBean("myBean")` 返回 `FactoryBean` 的产品，而调用 `getBean("&myBean")` 返回 `FactoryBean` 实例本身。

### 1.9.基于注解的容器配置

> Are annotations better than XML for configuring Spring?
> 注解比XML更适合配置Spring吗？
>
> The introduction of annotation-based configuration raised the question of whether this approach is “better” than XML. The short answer is “it depends.” The long answer is that each approach has its pros and cons, and, usually, it is up to the developer to decide which strategy suits them better. Due to the way they are defined, annotations provide a lot of context in their declaration, leading to shorter and more concise configuration. However, XML excels at wiring up components without touching their source code or recompiling them. Some developers prefer having the wiring close to the source while others argue that annotated classes are no longer POJOs and, furthermore, that the configuration becomes decentralized and harder to control.
> 引入基于注解的配置提出了这样一个问题：这种方法是否比XML“更好”。简短的回答是“看情况”详细的答案是，每种方法都有其优点和缺点，通常由开发人员决定哪种策略更适合他们。由于它们的定义方式，注解在其声明中提供了大量的上下文，从而使配置更短、更简洁。但是，XML擅长于在不接触组件的源代码或重新编译它们的情况下将组件连接起来。一些开发人员更喜欢在接近源代码的地方进行连接，而另一些人则认为带注解的类不再是POJO，而且配置变得分散且更难控制。
>
> No matter the choice, Spring can accommodate both styles and even mix them together. It is worth pointing out that through its [JavaConfig](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) option, Spring lets annotations be used in a non-invasive way, without touching the target components' source code and that, in terms of tooling, all configuration styles are supported by [Spring Tools](https://spring.io/tools) for Eclipse, Visual Studio Code, and Theia.
> 无论选择什么，Spring都可以容纳这两种风格，甚至可以将它们混合在一起。值得指出的是，通过其JavaConfig选项，Spring允许以非侵入性的方式使用注解，而无需接触目标组件的源代码，并且在工具方面，Spring Tools for Eclipse，Visual Studio Code和Theia支持所有配置样式。

An alternative to XML setup is provided by annotation-based configuration, which relies on bytecode metadata for wiring up components instead of XML declarations. Instead of using XML to describe a bean wiring, the developer moves the configuration into the component class itself by using annotations on the relevant class, method, or field declaration. As mentioned in [Example: The `AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp-examples-aabpp), using a `BeanPostProcessor` in conjunction with annotations is a common means of extending the Spring IoC container. For example, the [`@Autowired`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation) annotation provides the same capabilities as described in [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) but with more fine-grained control and wider applicability. In addition, Spring provides support for JSR-250 annotations, such as `@PostConstruct` and `@PreDestroy`, as well as support for JSR-330 (Dependency Injection for Java) annotations contained in the `jakarta.inject` package such as `@Inject` and `@Named`. Details about those annotations can be found in the [relevant section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-standard-annotations).
基于注解的配置提供了XML设置的替代方案，它依赖于字节码元数据而不是XML声明来连接组件。开发人员不使用XML来描述bean连接，而是通过在相关的类、方法或字段声明上使用注解来将配置移动到组件类本身。如示例中所述： `AutowiredAnnotationBeanPostProcessor` ，使用 `BeanPostProcessor` 结合注解是扩展Spring IoC容器的常见方法。例如， `@Autowired` 注释提供了与Autowiring Collaborators中所述相同的功能，但具有更细粒度的控制和更广泛的适用性。此外，Spring还支持JSR-250注释，例如 `@PostConstruct` 和 `@PreDestroy` ，以及 `jakarta.inject` 包中包含的JSR-330（Java依赖注入）注释，例如 `@Inject` 和 `@Named` 。有关这些注释的详细信息，请参见[相关章节](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-standard-annotations)。

> Annotation injection is performed before XML injection. Thus, the XML configuration overrides the annotations for properties wired through both approaches.
> 注解注入在XML注入之前执行。因此，XML配置覆盖了通过这两种方法连接的属性的方式。

As always, you can register the post-processors as individual bean definitions, but they can also be implicitly registered by including the following tag in an XML-based Spring configuration (notice the inclusion of the `context` namespace):
与往常一样，您可以将后处理器注册为单独的bean定义，但也可以通过在基于XML的Spring配置中包含以下标记来隐式注册它们（请注意包含 `context` 命名空间）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

The `<context:annotation-config/>` element implicitly registers the following post-processors:
`<context:annotation-config/>` 元素隐式注册以下后处理器：

- [`ConfigurationClassPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)
- [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)
- [`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html)
- [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html)
- [`EventListenerMethodProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/event/EventListenerMethodProcessor.html)

> `<context:annotation-config/>` only looks for annotations on beans in the same application context in which it is defined. This means that, if you put `<context:annotation-config/>` in a `WebApplicationContext` for a `DispatcherServlet`, it only checks for `@Autowired` beans in your controllers, and not your services. See [The DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet) for more information.
> `<context:annotation-config/>` 只在定义它的应用程序上下文中查找bean上的注释。这意味着，如果您将 `<context:annotation-config/>` 放入 `WebApplicationContext` 中以获取 `DispatcherServlet` ，它只会检查控制器中的 `@Autowired` bean，而不会检查服务。有关详细信息，请参阅[The DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)。

#### 1.9.1.使用 `@Autowired`

> JSR 330’s `@Inject` annotation can be used in place of Spring’s `@Autowired` annotation in the examples included in this section. See [here](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-standard-annotations) for more details.
> JSR 330的 `@Inject` annotation可以用来代替Spring的 `@Autowired` annotation。更多详情请看这里。

You can apply the `@Autowired` annotation to constructors, as the following example shows:
您可以将 `@Autowired` 注解应用于构造函数，如以下示例所示：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

> As of Spring Framework 4.3, an `@Autowired` annotation on such a constructor is no longer necessary if the target bean defines only one constructor to begin with. However, if several constructors are available and there is no primary/default constructor, at least one of the constructors must be annotated with `@Autowired` in order to instruct the container which one to use. See the discussion on [constructor resolution](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation-constructor-resolution) for details.
> 从Spring Framework 4.3开始，如果目标bean只定义了一个构造函数，那么在这样的构造函数上的 `@Autowired` 注释就不再需要了。但是，如果有几个构造函数可用，并且没有主/默认构造函数，则必须使用 `@Autowired` 注释至少一个构造函数，以便指示容器使用哪一个。有关详细信息，请参见构造函数解析的讨论。

You can also apply the `@Autowired` annotation to *traditional* setter methods, as the following example shows:
您还可以将 `@Autowired` 注解应用于传统的setter方法，如以下示例所示：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

You can also apply the annotation to methods with arbitrary names and multiple arguments, as the following example shows:
您还可以将注解应用于具有任意名称和多个参数的方法，如以下示例所示：

```java
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

You can apply `@Autowired` to fields as well and even mix it with constructors, as the following example shows:
您也可以将 `@Autowired` 应用于字段，甚至将其与构造函数混合使用，如以下示例所示：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

> Make sure that your target components (for example, `MovieCatalog` or `CustomerPreferenceDao`) are consistently declared by the type that you use for your `@Autowired`-annotated injection points. Otherwise, injection may fail due to a "no type match found" error at runtime.
> 确保您的目标组件（例如 `MovieCatalog` 或 `CustomerPreferenceDao` ）由您用于 `@Autowired` 注解注入点的类型一致地声明。否则，注入可能会由于运行时出现“找不到类型匹配”错误而失败。
>
> For XML-defined beans or component classes found via classpath scanning, the container usually knows the concrete type up front. However, for `@Bean` factory methods, you need to make sure that the declared return type is sufficiently expressive. For components that implement several interfaces or for components potentially referred to by their implementation type, consider declaring the most specific return type on your factory method (at least as specific as required by the injection points referring to your bean).
> 对于通过类路径扫描找到的XML定义的bean或组件类，容器通常预先知道具体类型。但是，对于 `@Bean` 工厂方法，您需要确保声明的返回类型具有足够的表达性。对于实现多个接口的组件或可能由其实现类型引用的组件，请考虑在工厂方法上声明最具体的返回类型（至少与引用bean的注入点所需的返回类型一样具体）。

You can also instruct Spring to provide all beans of a particular type from the `ApplicationContext` by adding the `@Autowired` annotation to a field or method that expects an array of that type, as the following example shows:
您还可以通过将 `@Autowired` 注解添加到期望该类型的数组的字段或方法来指示Spring从 `ApplicationContext` 提供特定类型的所有bean，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

The same applies for typed collections, as the following example shows:
这同样适用于类型化集合，如下例所示：

```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

> Your target beans can implement the `org.springframework.core.Ordered` interface or use the `@Order` or standard `@Priority` annotation if you want items in the array or list to be sorted in a specific order. Otherwise, their order follows the registration order of the corresponding target bean definitions in the container.
> 如果您希望数组或列表中的项以特定顺序排序，则目标bean可以实现 `org.springframework.core.Ordered` 接口或使用 `@Order` 或标准 `@Priority` 注释。否则，它们的顺序将遵循容器中相应目标bean定义的注册顺序。
>
> You can declare the `@Order` annotation at the target class level and on `@Bean` methods, potentially for individual bean definitions (in case of multiple definitions that use the same bean class). `@Order` values may influence priorities at injection points, but be aware that they do not influence singleton startup order, which is an orthogonal concern determined by dependency relationships and `@DependsOn` declarations.
> 您可以在目标类级别和 `@Bean` 方法上声明 `@Order` 注释，可能是针对单个bean定义（如果多个定义使用相同的bean类）。 `@Order` 值可能会影响注入点的优先级，但要注意它们不会影响单例启动顺序，这是由依赖关系和 `@DependsOn` 声明确定的正交关注点。
>
> Note that the standard `jakarta.annotation.Priority` annotation is not available at the `@Bean` level, since it cannot be declared on methods. Its semantics can be modeled through `@Order` values in combination with `@Primary` on a single bean for each type.
> 请注意，标准的 `jakarta.annotation.Priority` 注释在 `@Bean` 级别不可用，因为它不能在方法上声明。它的语义可以通过 `@Order` 值结合 `@Primary` 在每个类型的单个bean上建模。

Even typed `Map` instances can be autowired as long as the expected key type is `String`. The map values contain all beans of the expected type, and the keys contain the corresponding bean names, as the following example shows:
只要预期的密钥类型是 `String` ，即使是类型化的 `Map` 实例也可以自动连接。map值包含预期类型的所有bean，而key包含相应的bean名称，如以下示例所示：

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

By default, autowiring fails when no matching candidate beans are available for a given injection point. In the case of a declared array, collection, or map, at least one matching element is expected.
默认情况下，当给定注入点没有匹配的候选bean可用时，自动装配失败。如果是已声明的数组、集合或映射，则至少需要一个匹配元素。

The default behavior is to treat annotated methods and fields as indicating required dependencies. You can change this behavior as demonstrated in the following example, enabling the framework to skip a non-satisfiable injection point through marking it as non-required (i.e., by setting the `required` attribute in `@Autowired` to `false`):
默认行为是将带注释的方法和字段视为指示所需的依赖项。您可以更改此行为，如以下示例所示，通过将其标记为非必需（即，通过将 `@Autowired` 中的 `required` 属性设置为 `false` ）：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

> A non-required method will not be called at all if its dependency (or one of its dependencies, in case of multiple arguments) is not available. A non-required field will not get populated at all in such cases, leaving its default value in place.
> 如果一个非必需的方法的依赖项（或者在有多个参数的情况下，它的一个依赖项）不可用，则根本不会调用该方法。在这种情况下，根本不会填充非必填字段，而是保留其默认值。
>
> In other words, setting the `required` attribute to `false` indicates that the corresponding property is *optional* for autowiring purposes, and the property will be ignored if it cannot be autowired. This allows properties to be assigned default values that can be optionally overridden via dependency injection.
> 换句话说，将 `required` 属性设置为 `false` 表示相应的属性对于自动布线目的是可选的，并且如果属性不能自动布线，则将忽略该属性。这允许为属性分配默认值，可以通过依赖项注入可选地重写这些值。

Injected constructor and factory method arguments are a special case since the `required` attribute in `@Autowired` has a somewhat different meaning due to Spring’s constructor resolution algorithm that may potentially deal with multiple constructors. Constructor and factory method arguments are effectively required by default but with a few special rules in a single-constructor scenario, such as multi-element injection points (arrays, collections, maps) resolving to empty instances if no matching beans are available. This allows for a common implementation pattern where all dependencies can be declared in a unique multi-argument constructor — for example, declared as a single public constructor without an `@Autowired` annotation.
注入的构造函数和工厂方法参数是一种特殊情况，因为由于Spring的构造函数解析算法可能会处理多个构造函数， `@Autowired` 中的 `required` 属性具有不同的含义。默认情况下，构造函数和工厂方法参数是必需的，但在单构造函数场景中有一些特殊规则，例如如果没有匹配的bean可用，则多元素注入点（数组，集合，映射）解析为空实例。这允许一种通用的实现模式，其中所有依赖项都可以在唯一的多参数构造函数中声明-例如，声明为没有 `@Autowired` 注释的单个公共构造函数。

> Only one constructor of any given bean class may declare `@Autowired` with the `required` attribute set to `true`, indicating *the* constructor to autowire when used as a Spring bean. As a consequence, if the `required` attribute is left at its default value `true`, only a single constructor may be annotated with `@Autowired`. If multiple constructors declare the annotation, they will all have to declare `required=false` in order to be considered as candidates for autowiring (analogous to `autowire=constructor` in XML). The constructor with the greatest number of dependencies that can be satisfied by matching beans in the Spring container will be chosen. If none of the candidates can be satisfied, then a primary/default constructor (if present) will be used. Similarly, if a class declares multiple constructors but none of them is annotated with `@Autowired`, then a primary/default constructor (if present) will be used. If a class only declares a single constructor to begin with, it will always be used, even if not annotated. Note that an annotated constructor does not have to be public.
> 任何给定bean类的只有一个构造函数可以声明 `@Autowired` ，并将 `required` 属性设置为 `true` ，指示用作Spring bean时要自动连接的构造函数。因此，如果 `required` 属性保留其默认值 `true` ，则只有单个构造函数可以用 `@Autowired` 注释。如果有多个构造函数声明注解，它们都必须声明 `required=false` ，以便被认为是自动装配的候选者（类似于XML中的 `autowire=constructor` ）。将选择具有最大数量的依赖项的构造函数，这些依赖项可以通过匹配Spring容器中的bean来满足。如果没有候选者可以被满足，则将使用主/默认构造器（如果存在的话）。类似地，如果一个类声明了多个构造函数，但没有一个用 `@Autowired` 注释，那么将使用主/默认构造函数（如果存在）。如果一个类只声明了一个构造函数，那么它将始终被使用，即使没有注释。 请注意，带注释的构造函数不必是公共的。

Alternatively, you can express the non-required nature of a particular dependency through Java 8’s `java.util.Optional`, as the following example shows:
或者，您可以通过Java 8的 `java.util.Optional` 来表达特定依赖项的非必需性质，如以下示例所示：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

As of Spring Framework 5.0, you can also use a `@Nullable` annotation (of any kind in any package — for example, `javax.annotation.Nullable` from JSR-305) or just leverage Kotlin built-in null-safety support:
从Spring Framework 5.0开始，您还可以使用 `@Nullable` 注释（任何包中的任何类型-例如，JSR-305中的 `javax.annotation.Nullable` ）或仅利用Kotlin内置的null-safety支持：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

You can also use `@Autowired` for interfaces that are well-known resolvable dependencies: `BeanFactory`, `ApplicationContext`, `Environment`, `ResourceLoader`, `ApplicationEventPublisher`, and `MessageSource`. These interfaces and their extended interfaces, such as `ConfigurableApplicationContext` or `ResourcePatternResolver`, are automatically resolved, with no special setup necessary. The following example autowires an `ApplicationContext` object:
您还可以将 `@Autowired` 用于已知的可解析依赖项接口： `BeanFactory` 、 `ApplicationContext` 、 `Environment` 、 `ResourceLoader` 、 `ApplicationEventPublisher` 和 `MessageSource` 。这些接口及其扩展接口（如 `ConfigurableApplicationContext` 或 `ResourcePatternResolver` ）会自动解析，无需特殊设置。以下示例自动关联 `ApplicationContext` 对象：

```java
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

> The `@Autowired`, `@Inject`, `@Value`, and `@Resource` annotations are handled by Spring `BeanPostProcessor` implementations. This means that you cannot apply these annotations within your own `BeanPostProcessor` or `BeanFactoryPostProcessor` types (if any). These types must be 'wired up' explicitly by using XML or a Spring `@Bean` method.
> `@Autowired` 、 `@Inject` 、 `@Value` 和 `@Resource` 注释由Spring `BeanPostProcessor` 实现处理。这意味着您不能在自己的 `BeanPostProcessor` 或 `BeanFactoryPostProcessor` 类型（如果有的话）中应用这些注释。这些类型必须通过使用XML或Spring `@Bean` 方法显式地“连接”。

#### 1.9.2.使用 `@Primary` 微调基于注释的自动装配

Because autowiring by type may lead to multiple candidates, it is often necessary to have more control over the selection process. One way to accomplish this is with Spring’s `@Primary` annotation. `@Primary` indicates that a particular bean should be given preference when multiple beans are candidates to be autowired to a single-valued dependency. If exactly one primary bean exists among the candidates, it becomes the autowired value.
由于按类型自动装配可能会导致多个候选项，因此通常需要对选择过程进行更多控制。实现这一点的一种方法是使用Spring的 `@Primary` 注释。 `@Primary` 表示当多个bean是自动连接到单值依赖项的候选者时，应该优先考虑特定的bean。如果候选对象中恰好存在一个主bean，则它将成为自动连接的值。

Consider the following configuration that defines `firstMovieCatalog` as the primary `MovieCatalog`:
考虑以下将 `firstMovieCatalog` 定义为主 `MovieCatalog` 的配置：

```java
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

With the preceding configuration, the following `MovieRecommender` is autowired with the `firstMovieCatalog`:
使用上述配置，以下 `MovieRecommender` 与 `firstMovieCatalog` 自动连接：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

The corresponding bean definitions follow:
对应的Bean定义如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

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

#### 1.9.3.基于修饰符的标注自动装配算法

`@Primary` is an effective way to use autowiring by type with several instances when one primary candidate can be determined. When you need more control over the selection process, you can use Spring’s `@Qualifier` annotation. You can associate qualifier values with specific arguments, narrowing the set of type matches so that a specific bean is chosen for each argument. In the simplest case, this can be a plain descriptive value, as shown in the following example:
当可以确定一个主要候选时， `@Primary` 是使用具有多个实例的按类型自动装配的有效方法。当您需要更多地控制选择过程时，可以使用Spring的 `@Qualifier` 注释。您可以将限定符值与特定的参数相关联，从而缩小类型匹配的范围，以便为每个参数选择特定的bean。在最简单的情况下，这可以是一个简单的描述性值，如下例所示：

```java
public class MovieRecommender {

    private final MovieCatalog movieCatalog;

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

The following example shows corresponding bean definitions.
下面的示例显示了相应的bean定义。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/> (1)

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/> (2)

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

1. The bean with the `main` qualifier value is wired with the constructor argument that is qualified with the same value.
   具有 `main` 限定符值的bean与具有相同值限定的构造函数参数连接。
2. The bean with the `action` qualifier value is wired with the constructor argument that is qualified with the same value.
   具有 `action` 限定符值的bean与具有相同值限定的构造函数参数连接。

For a fallback match, the bean name is considered a default qualifier value. Thus, you can define the bean with an `id` of `main` instead of the nested qualifier element, leading to the same matching result. However, although you can use this convention to refer to specific beans by name, `@Autowired` is fundamentally about type-driven injection with optional semantic qualifiers. This means that qualifier values, even with the bean name fallback, always have narrowing semantics within the set of type matches. They do not semantically express a reference to a unique bean `id`. Good qualifier values are `main` or `EMEA` or `persistent`, expressing characteristics of a specific component that are independent from the bean `id`, which may be auto-generated in case of an anonymous bean definition such as the one in the preceding example.
对于回退匹配，bean名称被视为默认限定符值。因此，您可以使用 `id` 为 `main` 而不是嵌套的限定符元素来定义bean，从而得到相同的匹配结果。然而，尽管您可以使用这个约定通过名称引用特定的bean，但是 `@Autowired` 基本上是关于使用可选语义限定符的类型驱动注入。这意味着限定符值，即使使用bean名称回退，在类型匹配集中也总是具有缩小语义。它们在语义上不表示对唯一bean `id` 的引用。好的限定符值是 `main` 或 `EMEA` 或 `persistent` ，表示独立于bean `id` 的特定组件的特征，在匿名bean定义的情况下，例如在前面的示例中，可以自动生成这些特征。

Qualifiers also apply to typed collections, as discussed earlier — for example, to `Set<MovieCatalog>`. In this case, all matching beans, according to the declared qualifiers, are injected as a collection. This implies that qualifiers do not have to be unique. Rather, they constitute filtering criteria. For example, you can define multiple `MovieCatalog` beans with the same qualifier value “action”, all of which are injected into a `Set<MovieCatalog>` annotated with `@Qualifier("action")`.
限定符也适用于类型化集合，如前所述-例如，适用于 `Set<MovieCatalog>` 。在这种情况下，根据声明的限定符，所有匹配的bean都作为一个集合注入。这意味着限定符不必是唯一的。相反，它们构成过滤标准。例如，您可以使用相同的限定符值“action”定义多个 `MovieCatalog` bean，所有这些bean都被注入到使用 `@Qualifier("action")` 注释的 `Set<MovieCatalog>` 中。

> Letting qualifier values select against target bean names, within the type-matching candidates, does not require a `@Qualifier` annotation at the injection point. If there is no other resolution indicator (such as a qualifier or a primary marker), for a non-unique dependency situation, Spring matches the injection point name (that is, the field name or parameter name) against the target bean names and chooses the same-named candidate, if any.
> 在类型匹配的候选对象中，让限定符值针对目标bean名称进行选择，不需要在注入点使用 `@Qualifier` 注释。如果没有其他的解析指示符（比如一个限定符或者一个主标记），对于一个非唯一的依赖情况，Spring会将注入点名称（也就是字段名或者参数名）与目标bean名称进行匹配，并选择相同名称的候选项（如果有的话）。

That said, if you intend to express annotation-driven injection by name, do not primarily use `@Autowired`, even if it is capable of selecting by bean name among type-matching candidates. Instead, use the JSR-250 `@Resource` annotation, which is semantically defined to identify a specific target component by its unique name, with the declared type being irrelevant for the matching process. `@Autowired` has rather different semantics: After selecting candidate beans by type, the specified `String` qualifier value is considered within those type-selected candidates only (for example, matching an `account` qualifier against beans marked with the same qualifier label).
也就是说，如果您打算通过名称来表示注释驱动的注入，不要主要使用 `@Autowired` ，即使它能够通过bean名称在类型匹配的候选者中进行选择。相反，使用JSR-250 `@Resource` 注释，它在语义上定义为通过其唯一名称标识特定的目标组件，声明的类型与匹配过程无关。 `@Autowired` 有不同的语义：在按类型选择候选bean之后，指定的 `String` 限定符值仅在那些类型选择的候选中被考虑（例如，将 `account` 限定符与标记有相同限定符标签的bean匹配）。

For beans that are themselves defined as a collection, `Map`, or array type, `@Resource` is a fine solution, referring to the specific collection or array bean by unique name. That said, as of 4.3, you can match collection, `Map`, and array types through Spring’s `@Autowired` type matching algorithm as well, as long as the element type information is preserved in `@Bean` return type signatures or collection inheritance hierarchies. In this case, you can use qualifier values to select among same-typed collections, as outlined in the previous paragraph.
对于本身定义为集合、 `Map` 或数组类型的bean， `@Resource` 是一个很好的解决方案，通过唯一的名称引用特定的集合或数组bean。也就是说，从4.3开始，您也可以通过Spring的 `@Autowired` 类型匹配算法来匹配collection， `Map` 和数组类型，只要元素类型信息保留在 `@Bean` 返回类型签名或集合继承层次结构中。在这种情况下，可以使用限定符值在相同类型的集合中进行选择，如前一段所述。

As of 4.3, `@Autowired` also considers self references for injection (that is, references back to the bean that is currently injected). Note that self injection is a fallback. Regular dependencies on other components always have precedence. In that sense, self references do not participate in regular candidate selection and are therefore in particular never primary. On the contrary, they always end up as lowest precedence. In practice, you should use self references as a last resort only (for example, for calling other methods on the same instance through the bean’s transactional proxy). Consider factoring out the affected methods to a separate delegate bean in such a scenario. Alternatively, you can use `@Resource`, which may obtain a proxy back to the current bean by its unique name.
从4.3开始， `@Autowired` 还考虑了注入的自引用（即，对当前注入的bean的引用）。请注意，自注入是一种回退。对其他组件的常规依赖项始终具有优先级。从这个意义上说，自引用不参与常规的候选选择，因此特别地从来不是主要的。相反，它们总是以最低优先级结束。在实践中，应该将自引用作为最后的手段（例如，通过bean的事务代理调用同一实例上的其他方法）。在这种情况下，请考虑将受影响的方法分解到单独的委托bean中。或者，您可以使用 `@Resource` ，它可以通过当前bean的唯一名称获取代理。

> Trying to inject the results from `@Bean` methods on the same configuration class is effectively a self-reference scenario as well. Either lazily resolve such references in the method signature where it is actually needed (as opposed to an autowired field in the configuration class) or declare the affected `@Bean` methods as `static`, decoupling them from the containing configuration class instance and its lifecycle. Otherwise, such beans are only considered in the fallback phase, with matching beans on other configuration classes selected as primary candidates instead (if available).
> 尝试将 `@Bean` 方法的结果注入到同一个配置类上也是一个有效的自引用场景。要么在实际需要的方法签名中延迟解析此类引用（与配置类中的autowired字段相反），要么将受影响的 `@Bean` 方法声明为 `static` ，将它们与包含的配置类实例及其生命周期解耦。否则，只在回退阶段考虑此类bean，而将其他配置类上的匹配bean选择为主要候选项（如果可用）。

`@Autowired` applies to fields, constructors, and multi-argument methods, allowing for narrowing through qualifier annotations at the parameter level. In contrast, `@Resource` is supported only for fields and bean property setter methods with a single argument. As a consequence, you should stick with qualifiers if your injection target is a constructor or a multi-argument method.
`@Autowired` 适用于字段，构造函数和多参数方法，允许在参数级别通过限定符注释进行缩小。相比之下， `@Resource` 只支持字段和带有单个参数的bean属性setter方法。因此，如果注入目标是构造函数或多参数方法，则应坚持使用限定符。

You can create your own custom qualifier annotations. To do so, define an annotation and provide the `@Qualifier` annotation within your definition, as the following example shows:
您可以创建自己的自定义限定符注释。为此，定义一个注释并在定义中提供 `@Qualifier` 注释，如下例所示：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

Then you can provide the custom qualifier on autowired fields and parameters, as the following example shows:
然后，可以为自动关联字段和参数提供自定义限定符，如下例所示：

```java
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

Next, you can provide the information for the candidate bean definitions. You can add `<qualifier/>` tags as sub-elements of the `<bean/>` tag and then specify the `type` and `value` to match your custom qualifier annotations. The type is matched against the fully-qualified class name of the annotation. Alternately, as a convenience if no risk of conflicting names exists, you can use the short class name. The following example demonstrates both approaches:
接下来，您可以提供候选bean定义的信息。您可以添加 `<qualifier/>` 标记作为 `<bean/>` 标记的子元素，然后指定 `type` 和 `value` 以匹配自定义限定符注释。类型与注释的完全限定类名匹配。或者，为了方便起见，如果不存在名称冲突的风险，可以使用短类名。下面的示例演示了这两种方法：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

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

In [Classpath Scanning and Managed Components](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-classpath-scanning), you can see an annotation-based alternative to providing the qualifier metadata in XML. Specifically, see [Providing Qualifier Metadata with Annotations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-scanning-qualifiers).
在[类路径扫描和托管组件中](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-classpath-scanning)，您可以看到一种基于注释的替代方法，可以在XML中提供限定符元数据。具体请参见[提供带有注释的限定符元数据](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-scanning-qualifiers)。

In some cases, using an annotation without a value may suffice. This can be useful when the annotation serves a more generic purpose and can be applied across several different types of dependencies. For example, you may provide an offline catalog that can be searched when no Internet connection is available. First, define the simple annotation, as the following example shows:
在某些情况下，使用没有值的注释可能就足够了。当注释服务于更通用的目的时，这可能很有用，并且可以跨几种不同类型的依赖项应用。例如，您可以提供一个脱机目录，当没有Internet连接可用时可以搜索该目录。首先，定义简单注释，如以下示例所示：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {
}
```

Then add the annotation to the field or property to be autowired, as shown in the following example:
然后将批注添加到要自动关联的字段或属性，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    @Offline (1)
    private MovieCatalog offlineCatalog;

    // ...
}
```

1. This line adds the `@Offline` annotation.
   这一行添加了 `@Offline` 注释。

Now the bean definition only needs a qualifier `type`, as shown in the following example:
现在bean定义只需要一个限定符 `type` ，如以下示例所示：

```xml
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/> (1)
    <!-- inject any dependencies required by this bean -->
</bean>
```

1.  This element specifies the qualifier. 此元素指定限定符。

You can also define custom qualifier annotations that accept named attributes in addition to or instead of the simple `value` attribute. If multiple attribute values are then specified on a field or parameter to be autowired, a bean definition must match all such attribute values to be considered an autowire candidate. As an example, consider the following annotation definition:
您还可以定义自定义限定符注释，这些注释接受除了简单的 `value` 属性之外的命名属性，或者接受命名属性来代替简单的 `value` 属性。如果随后在要自动连接的字段或参数上指定了多个属性值，则bean定义必须匹配所有此类属性值才能被视为自动连接候选项。例如，考虑以下注释定义：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

In this case `Format` is an enum, defined as follows:
在这种情况下， `Format` 是一个枚举，定义如下：

```java
public enum Format {
    VHS, DVD, BLURAY
}
```

The fields to be autowired are annotated with the custom qualifier and include values for both attributes: `genre` and `format`, as the following example shows:
要自动关联的字段使用自定义限定符进行注释，并包括两个属性的值： `genre` 和 `format` ，如下例所示：

```java
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

Finally, the bean definitions should contain matching qualifier values. This example also demonstrates that you can use bean meta attributes instead of the `<qualifier/>` elements. If available, the `<qualifier/>` element and its attributes take precedence, but the autowiring mechanism falls back on the values provided within the `<meta/>` tags if no such qualifier is present, as in the last two bean definitions in the following example:
最后，bean定义应该包含匹配的限定符值。此示例还演示了您可以使用bean的元属性来代替 `<qualifier/>` 元素。如果可用，则 `<qualifier/>` 元素及其属性优先，但如果不存在此类限定符，则自动装配机制回退到 `<meta/>` 标记中提供的值，如以下示例中的最后两个bean定义所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

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

#### 1.9.4.使用泛型作为自动装配限定符

In addition to the `@Qualifier` annotation, you can use Java generic types as an implicit form of qualification. For example, suppose you have the following configuration:
除了 `@Qualifier` 注释之外，您还可以使用Java泛型类型作为限定的隐式形式。例如，假设您具有以下配置：

```java
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

Assuming that the preceding beans implement a generic interface, (that is, `Store<String>` and `Store<Integer>`), you can `@Autowire` the `Store` interface and the generic is used as a qualifier, as the following example shows:
假设前面的bean实现了一个泛型接口（即 `Store<String>` 和 `Store<Integer>` ），您可以对 `Store` 接口进行 `@Autowire` ，并将泛型用作限定符，如以下示例所示：

```java
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

Generic qualifiers also apply when autowiring lists, `Map` instances and arrays. The following example autowires a generic `List`:
通用限定符也适用于自动装配列表、 `Map` 实例和数组。以下示例自动连接通用 `List` ：

```java
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

#### 1.9.5.使用 `CustomAutowireConfigurer`

[`CustomAutowireConfigurer`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/annotation/CustomAutowireConfigurer.html) is a `BeanFactoryPostProcessor` that lets you register your own custom qualifier annotation types, even if they are not annotated with Spring’s `@Qualifier` annotation. The following example shows how to use `CustomAutowireConfigurer`:
[`CustomAutowireConfigurer`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/annotation/CustomAutowireConfigurer.html) 是一个 `BeanFactoryPostProcessor` ，它允许您注册自己的自定义限定符注释类型，即使它们没有使用Spring的 `@Qualifier` 注释进行注释。下面的例子展示了如何使用 `CustomAutowireConfigurer`：

```xml
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

The `AutowireCandidateResolver` determines autowire candidates by:
`AutowireCandidateResolver` 通过以下方式确定自动连接候选：

- The `autowire-candidate` value of each bean definition
  每个bean定义的 `autowire-candidate` 值
- Any `default-autowire-candidates` patterns available on the `<beans/>` element
  `<beans/>` 元素上可用的任何 `default-autowire-candidates` 模式
- The presence of `@Qualifier` annotations and any custom annotations registered with the `CustomAutowireConfigurer`
  `@Qualifier` 注释和任何自定义注释的存在与 `CustomAutowireConfigurer`

When multiple beans qualify as autowire candidates, the determination of a “primary” is as follows: If exactly one bean definition among the candidates has a `primary` attribute set to `true`, it is selected.
当多个bean有资格作为autowire候选时，“主要”的确定如下所示：如果候选项中恰好有一个bean定义的 `primary` 属性设置为 `true` ，则选择它。

#### 1.9.6.用 `@Resource`注入

Spring also supports injection by using the JSR-250 `@Resource` annotation (`jakarta.annotation.Resource`) on fields or bean property setter methods. This is a common pattern in Jakarta EE: for example, in JSF-managed beans and JAX-WS endpoints. Spring supports this pattern for Spring-managed objects as well.
Spring还通过在字段或bean属性setter方法上使用JSR-250 `@Resource` annotation（ `jakarta.annotation.Resource` ）来支持注入。这是Jakarta EE中的常见模式：例如，在JSF管理的bean和JAX-WS端点中。Spring也支持Spring管理的对象的这种模式。

`@Resource` takes a name attribute. By default, Spring interprets that value as the bean name to be injected. In other words, it follows by-name semantics, as demonstrated in the following example:
`@Resource` 带有name属性。默认情况下，Spring将该值解释为要注入的bean名称。换句话说，它遵循by-name语义，如以下示例所示：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") (1)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

1.  This line injects a `@Resource`. 该行注入 `@Resource` 。

If no name is explicitly specified, the default name is derived from the field name or setter method. In case of a field, it takes the field name. In case of a setter method, it takes the bean property name. The following example is going to have the bean named `movieFinder` injected into its setter method:
如果未显式指定名称，则默认名称派生自字段名称或setter方法。如果是字段，则使用字段名。如果是setter方法，它采用bean属性名。下面的例子将把名为 `movieFinder` 的bean注入到它的setter方法中：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

> The name provided with the annotation is resolved as a bean name by the `ApplicationContext` of which the `CommonAnnotationBeanPostProcessor` is aware. The names can be resolved through JNDI if you configure Spring’s [`SimpleJndiBeanFactory`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/jndi/support/SimpleJndiBeanFactory.html) explicitly. However, we recommend that you rely on the default behavior and use Spring’s JNDI lookup capabilities to preserve the level of indirection.
> 注释提供的名称由 `ApplicationContext` 解析为bean名称， `CommonAnnotationBeanPostProcessor` 知道该名称。如果您显式地配置Spring的 `SimpleJndiBeanFactory` ，则可以通过JNDI解析名称。但是，我们建议您依赖默认行为并使用Spring的JNDI查找功能来保留间接层级。

In the exclusive case of `@Resource` usage with no explicit name specified, and similar to `@Autowired`, `@Resource` finds a primary type match instead of a specific named bean and resolves well known resolvable dependencies: the `BeanFactory`, `ApplicationContext`, `ResourceLoader`, `ApplicationEventPublisher`, and `MessageSource` interfaces.
在没有明确指定名称的 `@Resource` 使用的独占情况下，类似于 `@Autowired` ， `@Resource` 查找主类型匹配而不是特定命名的bean，并解析众所周知的可解析依赖关系： `BeanFactory` 、 `ApplicationContext` 、 `ResourceLoader` 、 `ApplicationEventPublisher` 和 `MessageSource` 接口。

Thus, in the following example, the `customerPreferenceDao` field first looks for a bean named "customerPreferenceDao" and then falls back to a primary type match for the type `CustomerPreferenceDao`:
因此，在下面的示例中， `customerPreferenceDao` 字段首先查找名为“customerPreferenceDao”的bean，然后回退到类型 `CustomerPreferenceDao` 的主类型匹配：

```java
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; (1)

    public MovieRecommender() {
    }

    // ...
}
```

1. The `context` field is injected based on the known resolvable dependency type: `ApplicationContext`.
   `context` 字段基于已知的可解析依赖类型被注入：`ApplicationContext`。

#### 1.9.7.使用 `@Value`

`@Value` is typically used to inject externalized properties:
`@Value` 通常用于注入外部化属性：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

With the following configuration:
具有以下配置：

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

And the following `application.properties` file:
下面是 `application.properties` 文件：

```properties
catalog.name=MovieCatalog
```

In that case, the `catalog` parameter and field will be equal to the `MovieCatalog` value.
在这种情况下， `catalog` 参数和字段将等于 `MovieCatalog` 值。

A default lenient embedded value resolver is provided by Spring. It will try to resolve the property value and if it cannot be resolved, the property name (for example `${catalog.name}`) will be injected as the value. If you want to maintain strict control over nonexistent values, you should declare a `PropertySourcesPlaceholderConfigurer` bean, as the following example shows:
Spring提供了一个默认的宽松嵌入值解析器。它将尝试解析属性值，如果无法解析，则属性名称（例如 `${catalog.name}` ）将作为值注入。如果你想保持对不存在的值的严格控制，你应该声明一个 `PropertySourcesPlaceholderConfigurer` bean，如下面的例子所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

> When configuring a `PropertySourcesPlaceholderConfigurer` using JavaConfig, the `@Bean` method must be `static`.
> 使用JavaConfig配置 `PropertySourcesPlaceholderConfigurer` 时， `@Bean` 方法必须是 `static` 。

Using the above configuration ensures Spring initialization failure if any `${}` placeholder could not be resolved. It is also possible to use methods like `setPlaceholderPrefix`, `setPlaceholderSuffix`, or `setValueSeparator` to customize placeholders.
使用上面的配置可以确保在任何 `${}` 占位符无法解析时Spring初始化失败。也可以使用 `setPlaceholderPrefix` 、 `setPlaceholderSuffix` 或 `setValueSeparator` 等方法来自定义占位符。

> Spring Boot configures by default a `PropertySourcesPlaceholderConfigurer` bean that will get properties from `application.properties` and `application.yml` files.
> Spring Boot默认配置了一个 `PropertySourcesPlaceholderConfigurer` bean，它将从 `application.properties` 和 `application.yml` 文件中获取属性。

Built-in converter support provided by Spring allows simple type conversion (to `Integer` or `int` for example) to be automatically handled. Multiple comma-separated values can be automatically converted to `String` array without extra effort.
Spring提供的内置转换器支持允许自动处理简单的类型转换（例如到 `Integer` 或 `int` ）。多个逗号分隔的值可以自动转换为 `String` 数组，而无需额外的努力。

It is possible to provide a default value as following:
可以提供如下默认值：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
        this.catalog = catalog;
    }
}
```

A Spring `BeanPostProcessor` uses a `ConversionService` behind the scenes to handle the process for converting the `String` value in `@Value` to the target type. If you want to provide conversion support for your own custom type, you can provide your own `ConversionService` bean instance as the following example shows:
Spring `BeanPostProcessor` 在幕后使用 `ConversionService` 来处理将 `@Value` 中的 `String` 值转换为目标类型的过程。如果你想为你自己的自定义类型提供转换支持，你可以提供你自己的 `ConversionService` bean实例，如下例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public ConversionService conversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        conversionService.addConverter(new MyCustomConverter());
        return conversionService;
    }
}
```

When `@Value` contains a [`SpEL` expression](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions) the value will be dynamically computed at runtime as the following example shows:
当 `@Value` 包含 [`SpEL` 表达式](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions)时，该值将在运行时动态计算，如以下示例所示：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {
        this.catalog = catalog;
    }
}
```

SpEL also enables the use of more complex data structures:
SpEL还支持使用更复杂的数据结构：

```java
@Component
public class MovieRecommender {

    private final Map<String, Integer> countOfMoviesPerCatalog;

    public MovieRecommender(
            @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {
        this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
    }
}
```

#### 1.9.8.使用 `@PostConstruct` 和 `@PreDestroy`

The `CommonAnnotationBeanPostProcessor` not only recognizes the `@Resource` annotation but also the JSR-250 lifecycle annotations: `jakarta.annotation.PostConstruct` and `jakarta.annotation.PreDestroy`. Introduced in Spring 2.5, the support for these annotations offers an alternative to the lifecycle callback mechanism described in [initialization callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) and [destruction callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean). Provided that the `CommonAnnotationBeanPostProcessor` is registered within the Spring `ApplicationContext`, a method carrying one of these annotations is invoked at the same point in the lifecycle as the corresponding Spring lifecycle interface method or explicitly declared callback method. In the following example, the cache is pre-populated upon initialization and cleared upon destruction:
`CommonAnnotationBeanPostProcessor` 不仅可以识别 `@Resource` 注释，还可以识别JSR-250生命周期注释： `jakarta.annotation.PostConstruct` 和 `jakarta.annotation.PreDestroy` 。Spring 2.5中引入了对这些注释的支持，为初始化回调和销毁回调中描述的生命周期回调机制提供了一种替代方案。假设 `CommonAnnotationBeanPostProcessor` 在Spring `ApplicationContext` 中注册，则携带这些注释之一的方法将在生命周期中与相应的Spring生命周期接口方法或显式声明的回调方法相同的点被调用。在以下示例中，该高速缓存在初始化时预填充，在销毁时清除：

```java
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

For details about the effects of combining various lifecycle mechanisms, see [Combining Lifecycle Mechanisms](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-combined-effects).
有关组合各种生命周期机制的效果的详细信息，请参阅[组合生命周期机制](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-combined-effects)。

> Like `@Resource`, the `@PostConstruct` and `@PreDestroy` annotation types were a part of the standard Java libraries from JDK 6 to 8. However, the entire `javax.annotation` package got separated from the core Java modules in JDK 9 and eventually removed in JDK 11. As of Jakarta EE 9, the package lives in `jakarta.annotation` now. If needed, the `jakarta.annotation-api` artifact needs to be obtained via Maven Central now, simply to be added to the application’s classpath like any other library.
> 与 `@Resource` 一样， `@PostConstruct` 和 `@PreDestroy` 注释类型是JDK 6到8的标准Java库的一部分。然而，整个 `javax.annotation` 包在JDK 9中与核心Java模块分离，并最终在JDK 11中被删除。从Jakarta EE 9开始，该软件包现在位于 `jakarta.annotation` 。如果需要，现在就需要通过Maven Central获取 `jakarta.annotation-api` 工件，只需像任何其他库一样将其添加到应用程序的类路径即可。

### 1.10.类路径扫描和托管组件

Most examples in this chapter use XML to specify the configuration metadata that produces each `BeanDefinition` within the Spring container. The previous section ([Annotation-based Container Configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config)) demonstrates how to provide a lot of the configuration metadata through source-level annotations. Even in those examples, however, the "base" bean definitions are explicitly defined in the XML file, while the annotations drive only the dependency injection. This section describes an option for implicitly detecting the candidate components by scanning the classpath. Candidate components are classes that match against a filter criteria and have a corresponding bean definition registered with the container. This removes the need to use XML to perform bean registration. Instead, you can use annotations (for example, `@Component`), AspectJ type expressions, or your own custom filter criteria to select which classes have bean definitions registered with the container.
本章中的大多数示例都使用XML来指定在Spring容器中生成每个 `BeanDefinition` 的配置元数据。上一节（[基于注释的容器配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config)）演示了如何通过源代码级别的注释提供大量配置元数据。但是，即使在这些示例中，“基本”bean定义也是在XML文件中显式定义的，而注释只驱动依赖项注入。本节描述一个通过扫描类路径隐式检测候选组件的选项。候选组件是与过滤器条件匹配的类，并且具有向容器注册的对应bean定义。这样就不需要使用XML来执行bean注册。相反，您可以使用注释（例如 `@Component` ）、AspectJ类型表达式或您自己的自定义过滤器条件来选择哪些类具有注册到容器中的bean定义。

> You can define beans using Java rather than using XML files. Take a look at the `@Configuration`, `@Bean`, `@Import`, and `@DependsOn` annotations for examples of how to use these features.
> 您可以使用Java而不是XML文件来定义bean。查看 `@Configuration` 、 `@Bean` 、 `@Import` 和 `@DependsOn` 注释，了解如何使用这些功能的示例。

#### 1.10.1. `@Component` 和其他模式化注释

The `@Repository` annotation is a marker for any class that fulfills the role or stereotype of a repository (also known as Data Access Object or DAO). Among the uses of this marker is the automatic translation of exceptions, as described in [Exception Translation](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#orm-exception-translation).
`@Repository` 注释是任何满足存储库（也称为数据访问对象或DAO）的角色或原型的类的标记。此标记的用途之一是自动转换异常，如[异常转换](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#orm-exception-translation)中所述。

Spring provides further stereotype annotations: `@Component`, `@Service`, and `@Controller`. `@Component` is a generic stereotype for any Spring-managed component. `@Repository`, `@Service`, and `@Controller` are specializations of `@Component` for more specific use cases (in the persistence, service, and presentation layers, respectively). Therefore, you can annotate your component classes with `@Component`, but, by annotating them with `@Repository`, `@Service`, or `@Controller` instead, your classes are more properly suited for processing by tools or associating with aspects. For example, these stereotype annotations make ideal targets for pointcuts. `@Repository`, `@Service`, and `@Controller` can also carry additional semantics in future releases of the Spring Framework. Thus, if you are choosing between using `@Component` or `@Service` for your service layer, `@Service` is clearly the better choice. Similarly, as stated earlier, `@Repository` is already supported as a marker for automatic exception translation in your persistence layer.
Spring提供了进一步的原型注释： `@Component` 、 `@Service` 和 `@Controller` 。 `@Component` 是任何Spring管理的组件的通用原型。 `@Repository` 、 `@Service` 和 `@Controller` 是 `@Component` 针对更具体用例的专门化（分别在持久层、服务层和表示层）。因此，您可以使用 `@Component` 注释组件类，但是，通过使用 `@Repository` 、 `@Service` 或 `@Controller` 注释它们，您的类更适合于由工具处理或与方面关联。例如，这些构造型注释是切入点的理想目标。 `@Repository` 、 `@Service` 和 `@Controller` 还可以在Spring框架的未来版本中携带额外的语义。因此，如果您在使用 `@Component` 或 `@Service` 作为服务层之间进行选择， `@Service` 显然是更好的选择。类似地，如前所述， `@Repository` 已经被支持作为持久层中自动异常转换的标记。

#### 1.10.2.使用元注释和合成注释

Many of the annotations provided by Spring can be used as meta-annotations in your own code. A meta-annotation is an annotation that can be applied to another annotation. For example, the `@Service` annotation mentioned [earlier](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-stereotype-annotations) is meta-annotated with `@Component`, as the following example shows:
Spring提供的许多注释都可以在您自己的代码中用作元注释。元注释是可以应用于另一个注释的注释。例如，前面提到的 `@Service` 注释使用 `@Component` 进行元注释，如下例所示：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component (1)
public @interface Service {

    // ...
}
```

1.  The `@Component` causes `@Service` to be treated in the same way as `@Component`. `@Component` 导致 `@Service` 以与 `@Component` 相同的方式处理。

You can also combine meta-annotations to create “composed annotations”. For example, the `@RestController` annotation from Spring MVC is composed of `@Controller` and `@ResponseBody`.
您还可以组合元注释以创建“组合注释”。例如，Spring MVC中的 `@RestController` 注释由 `@Controller` 和 `@ResponseBody` 组成。

In addition, composed annotations can optionally redeclare attributes from meta-annotations to allow customization. This can be particularly useful when you want to only expose a subset of the meta-annotation’s attributes. For example, Spring’s `@SessionScope` annotation hard codes the scope name to `session` but still allows customization of the `proxyMode`. The following listing shows the definition of the `SessionScope` annotation:
此外，组合注释可以选择性地从元注释重新声明属性，以允许自定义。当您只想公开元注释属性的一个子集时，这可能特别有用。例如，Spring的 `@SessionScope` 注释将作用域名称硬编码为 `session` ，但仍然允许定制 `proxyMode` 。下面的清单显示了 `SessionScope` 注释的定义：

```java
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

You can then use `@SessionScope` without declaring the `proxyMode` as follows:
然后，您可以使用 `@SessionScope` 而不声明 `proxyMode` ，如下所示：

```java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

You can also override the value for the `proxyMode`, as the following example shows:
也可以覆盖 `proxyMode` 的值，如下例所示：

```java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

For further details, see the [Spring Annotation Programming Model](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model) wiki page.
有关更多详细信息，请参阅[Spring Annotation Programming Model](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model) wiki页面。

#### 1.10.3.自动检测类和注册Bean定义

Spring can automatically detect stereotyped classes and register corresponding `BeanDefinition` instances with the `ApplicationContext`. For example, the following two classes are eligible for such autodetection:
Spring可以自动检测原型类，并使用 `ApplicationContext` 注册相应的 `BeanDefinition` 实例。例如，以下两个类适合于这种自动检测：

```java
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```java
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

To autodetect these classes and register the corresponding beans, you need to add `@ComponentScan` to your `@Configuration` class, where the `basePackages` attribute is a common parent package for the two classes. (Alternatively, you can specify a comma- or semicolon- or space-separated list that includes the parent package of each class.)
要自动检测这些类并注册相应的bean，您需要将 `@ComponentScan` 添加到 `@Configuration` 类，其中 `basePackages` 属性是两个类的公共父包。（或者，您可以指定逗号、分号或空格分隔的列表，其中包含每个类的父包。）

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

> For brevity, the preceding example could have used the `value` attribute of the annotation (that is, `@ComponentScan("org.example")`).
> 为了简洁起见，前面的示例可能使用了注释的 `value` 属性（即 `@ComponentScan("org.example")` ）。

The following alternative uses XML:
以下替代方法使用XML：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

> The use of `<context:component-scan>` implicitly enables the functionality of `<context:annotation-config>`. There is usually no need to include the `<context:annotation-config>` element when using `<context:component-scan>`.
> `<context:component-scan>` 的使用隐含地启用 `<context:annotation-config>` 的功能。使用 `<context:component-scan>` 时，通常不需要包含 `<context:annotation-config>` 元素。

> The scanning of classpath packages requires the presence of corresponding directory entries in the classpath. When you build JARs with Ant, make sure that you do not activate the files-only switch of the JAR task. Also, classpath directories may not be exposed based on security policies in some environments — for example, standalone apps on JDK 1.7.0_45 and higher (which requires 'Trusted-Library' setup in your manifests — see https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources).
> 扫描类路径包需要类路径中存在相应的目录条目。当您使用Ant构建JAR时，请确保您没有激活JAR任务的仅文件开关。此外，在某些环境中，类路径目录可能不会根据安全策略公开-例如，JDK 1.7.0_45及更高版本上的独立应用程序（这需要在清单中设置“Trusted-Library”-请参阅 https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources ）。
>
> On JDK 9’s module path (Jigsaw), Spring’s classpath scanning generally works as expected. However, make sure that your component classes are exported in your `module-info` descriptors. If you expect Spring to invoke non-public members of your classes, make sure that they are 'opened' (that is, that they use an `opens` declaration instead of an `exports` declaration in your `module-info` descriptor).
> 在JDK 9的模块路径（Jigsaw）上，Spring的类路径扫描通常按预期工作。但是，请确保您的组件类在 `module-info` 描述符中导出。如果您希望Spring调用类的非公共成员，请确保它们是“打开的”（也就是说，它们在 `module-info` 描述符中使用 `opens` 声明，而不是 `exports` 声明）。

Furthermore, the `AutowiredAnnotationBeanPostProcessor` and `CommonAnnotationBeanPostProcessor` are both implicitly included when you use the component-scan element. That means that the two components are autodetected and wired together — all without any bean configuration metadata provided in XML.
此外，当您使用component-scan元素时， `AutowiredAnnotationBeanPostProcessor` 和 `CommonAnnotationBeanPostProcessor` 都被隐式地包含在内。这意味着这两个组件被自动检测并连接在一起--所有这些都不需要XML中提供的任何bean配置元数据。

> You can disable the registration of `AutowiredAnnotationBeanPostProcessor` and `CommonAnnotationBeanPostProcessor` by including the `annotation-config` attribute with a value of `false`.
> 您可以通过包含值为 `false` 的 `annotation-config` 属性来禁用 `AutowiredAnnotationBeanPostProcessor` 和 `CommonAnnotationBeanPostProcessor` 的注册。

#### 1.10.4.使用过滤器自定义扫描

By default, classes annotated with `@Component`, `@Repository`, `@Service`, `@Controller`, `@Configuration`, or a custom annotation that itself is annotated with `@Component` are the only detected candidate components. However, you can modify and extend this behavior by applying custom filters. Add them as `includeFilters` or `excludeFilters` attributes of the `@ComponentScan` annotation (or as `<context:include-filter />` or `<context:exclude-filter />` child elements of the `<context:component-scan>` element in XML configuration). Each filter element requires the `type` and `expression` attributes. The following table describes the filtering options:
默认情况下，使用 `@Component` 、 `@Repository` 、 `@Service` 、 `@Controller` 、 `@Configuration` 注释的类或本身使用 `@Component` 注释的自定义注释是唯一检测到的候选组件。但是，您可以通过应用自定义筛选器来修改和扩展此行为。将它们添加为 `@ComponentScan` 注释的 `includeFilters` 或 `excludeFilters` 属性（或者作为XML配置中 `<context:component-scan>` 元素的 `<context:include-filter />` 或 `<context:exclude-filter />` 子元素）。每个过滤器元素都需要 `type` 和 `expression` 属性。下表介绍了筛选选项：

| Filter Type                       | Example Expression 示例表达式 | Description                                                  |
| :-------------------------------- | :---------------------------- | :----------------------------------------------------------- |
| annotation (default) 注释（默认） | `org.example.SomeAnnotation`  | An annotation to be *present* or *meta-present* at the type level in target components. 要在目标组件中的类型级别呈现或元呈现的注释。 |
| assignable                        | `org.example.SomeClass`       | A class (or interface) that the target components are assignable to (extend or implement). 目标组件可分配（扩展或实现）到的类（或接口）。 |
| aspectj                           | `org.example..*Service+`      | An AspectJ type expression to be matched by the target components. 目标组件要匹配的AspectJ类型表达式。 |
| regex                             | `org\.example\.Default.*`     | A regex expression to be matched by the target components' class names. 要与目标组件的类名匹配的正则表达式。 |
| custom                            | `org.example.MyTypeFilter`    | A custom implementation of the `org.springframework.core.type.TypeFilter` interface. `org.springframework.core.type.TypeFilter` 接口的自定义实现。 |

The following example shows the configuration ignoring all `@Repository` annotations and using “stub” repositories instead:
下面的示例显示了忽略所有 `@Repository` 注释并使用“stub”存储库的配置：

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    // ...
}
```

The following listing shows the equivalent XML:
下面的清单显示了等效的XML：

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

> You can also disable the default filters by setting `useDefaultFilters=false` on the annotation or by providing `use-default-filters="false"` as an attribute of the `<component-scan/>` element. This effectively disables automatic detection of classes annotated or meta-annotated with `@Component`, `@Repository`, `@Service`, `@Controller`, `@RestController`, or `@Configuration`.
> 您还可以通过在注释上设置 `useDefaultFilters=false` 或提供 `use-default-filters="false"` 作为 `<component-scan/>` 元素的属性来禁用默认过滤器。这有效地禁用了用 `@Component` 、 `@Repository` 、 `@Service` 、 `@Controller` 、 `@RestController` 或 `@Configuration` 注释或元注释的类的自动检测。

#### 1.10.5.在组件中定义Bean元数据

Spring components can also contribute bean definition metadata to the container. You can do this with the same `@Bean` annotation used to define bean metadata within `@Configuration` annotated classes. The following example shows how to do so:
Spring组件还可以向容器提供bean定义元数据。您可以使用与在 `@Configuration` 注释类中定义bean元数据相同的 `@Bean` 注释来实现这一点。下面的示例说明如何执行此操作：

```java
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

The preceding class is a Spring component that has application-specific code in its `doWork()` method. However, it also contributes a bean definition that has a factory method referring to the method `publicInstance()`. The `@Bean` annotation identifies the factory method and other bean definition properties, such as a qualifier value through the `@Qualifier` annotation. Other method-level annotations that can be specified are `@Scope`, `@Lazy`, and custom qualifier annotations.
前面的类是一个Spring组件，在其 `doWork()` 方法中具有特定于应用程序的代码。但是，它也提供了一个bean定义，该bean定义具有引用方法 `publicInstance()` 的工厂方法。 `@Bean` 注释标识工厂方法和其他bean定义属性，例如通过 `@Qualifier` 注释的限定符值。可以指定的其他方法级注释有 `@Scope` 、 `@Lazy` 和自定义限定符注释。

> In addition to its role for component initialization, you can also place the `@Lazy` annotation on injection points marked with `@Autowired` or `@Inject`. In this context, it leads to the injection of a lazy-resolution proxy. However, such a proxy approach is rather limited. For sophisticated lazy interactions, in particular in combination with optional dependencies, we recommend `ObjectProvider<MyTargetBean>` instead.
> 除了用于组件初始化之外，您还可以将 `@Lazy` 注释放置在标记为 `@Autowired` 或 `@Inject` 的注入点上。在这种情况下，它会导致注入一个延迟解析代理。然而，这样的代理方法是相当有限的。对于复杂的惰性交互，特别是与可选依赖项结合使用时，我们建议使用 `ObjectProvider<MyTargetBean>` 。

Autowired fields and methods are supported, as previously discussed, with additional support for autowiring of `@Bean` methods. The following example shows how to do so:
如前所述，支持自动绑定字段和方法，并额外支持 `@Bean` 方法的自动绑定。下面的示例说明如何执行此操作：

```java
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

The example autowires the `String` method parameter `country` to the value of the `age` property on another bean named `privateInstance`. A Spring Expression Language element defines the value of the property through the notation `#{ <expression> }`. For `@Value` annotations, an expression resolver is preconfigured to look for bean names when resolving expression text.
该示例将 `String` 方法参数 `country` 自动连接到另一个名为 `privateInstance` 的bean上的 `age` 属性的值。Spring Expression Language元素通过符号 `#{ <expression> }` 定义属性的值。对于 `@Value` 注释，表达式解析器被预配置为在解析表达式文本时查找bean名称。

As of Spring Framework 4.3, you may also declare a factory method parameter of type `InjectionPoint` (or its more specific subclass: `DependencyDescriptor`) to access the requesting injection point that triggers the creation of the current bean. Note that this applies only to the actual creation of bean instances, not to the injection of existing instances. As a consequence, this feature makes most sense for beans of prototype scope. For other scopes, the factory method only ever sees the injection point that triggered the creation of a new bean instance in the given scope (for example, the dependency that triggered the creation of a lazy singleton bean). You can use the provided injection point metadata with semantic care in such scenarios. The following example shows how to use `InjectionPoint`:
从Spring Framework 4.3开始，您还可以声明类型为 `InjectionPoint` 的工厂方法参数（或其更具体的子类： `DependencyDescriptor` ）来访问触发当前bean创建的请求注入点。注意，这只适用于bean实例的实际创建，而不适用于现有实例的注入。因此，该特性对于原型范围的bean最有意义。对于其他作用域，工厂方法只能看到在给定作用域中触发创建新bean实例的注入点（例如，触发创建惰性单例bean的依赖项）。在这样的场景中，您可以使用提供的注入点元数据，但要注意语义。下面的例子展示了如何使用 `InjectionPoint` ：

```java
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```

The `@Bean` methods in a regular Spring component are processed differently than their counterparts inside a Spring `@Configuration` class. The difference is that `@Component` classes are not enhanced with CGLIB to intercept the invocation of methods and fields. CGLIB proxying is the means by which invoking methods or fields within `@Bean` methods in `@Configuration` classes creates bean metadata references to collaborating objects. Such methods are not invoked with normal Java semantics but rather go through the container in order to provide the usual lifecycle management and proxying of Spring beans, even when referring to other beans through programmatic calls to `@Bean` methods. In contrast, invoking a method or field in a `@Bean` method within a plain `@Component` class has standard Java semantics, with no special CGLIB processing or other constraints applying.
常规Spring组件中的 `@Bean` 方法与Spring `@Configuration` 类中的对应方法的处理方式不同。区别在于 `@Component` 类没有使用CGLIB来增强以拦截方法和字段的调用。CGLIB代理是调用 `@Configuration` 类中的 `@Bean` 方法中的方法或字段来创建对协作对象的bean元数据引用的方法。这些方法不是用普通的Java语义调用的，而是通过容器来提供Spring bean的常规生命周期管理和代理，即使是通过编程调用 `@Bean` 方法引用其他bean时也是如此。相比之下，在普通的 `@Component` 类中调用 `@Bean` 方法中的方法或字段具有标准的Java语义，没有特殊的CGLIB处理或其他约束。

> You may declare `@Bean` methods as `static`, allowing for them to be called without creating their containing configuration class as an instance. This makes particular sense when defining post-processor beans (for example, of type `BeanFactoryPostProcessor` or `BeanPostProcessor`), since such beans get initialized early in the container lifecycle and should avoid triggering other parts of the configuration at that point.
> 您可以将 `@Bean` 方法声明为 `static` ，允许在不将其包含的配置类创建为实例的情况下调用它们。这在定义后处理器bean（例如，类型为 `BeanFactoryPostProcessor` 或 `BeanPostProcessor` 的bean）时特别有意义，因为此类bean在容器生命周期的早期初始化，并且应该避免在该点触发配置的其他部分。
>
> Calls to static `@Bean` methods never get intercepted by the container, not even within `@Configuration` classes (as described earlier in this section), due to technical limitations: CGLIB subclassing can override only non-static methods. As a consequence, a direct call to another `@Bean` method has standard Java semantics, resulting in an independent instance being returned straight from the factory method itself.
> 由于技术限制，对静态 `@Bean` 方法的调用永远不会被容器拦截，即使在 `@Configuration` 类中也不会（如本节前面所述）：CGLIB子类只能覆盖非静态方法。因此，对另一个 `@Bean` 方法的直接调用具有标准的Java语义，导致直接从工厂方法本身返回一个独立的实例。
>
> The Java language visibility of `@Bean` methods does not have an immediate impact on the resulting bean definition in Spring’s container. You can freely declare your factory methods as you see fit in non-`@Configuration` classes and also for static methods anywhere. However, regular `@Bean` methods in `@Configuration` classes need to be overridable — that is, they must not be declared as `private` or `final`.
> `@Bean` 方法的Java语言可见性不会对Spring容器中的结果bean定义产生直接影响。你可以在非 `@Configuration` 类中自由声明你的工厂方法，也可以在任何地方声明静态方法。然而，在 `@Configuration` 类中的常规 `@Bean` 方法需要是可重写的-也就是说，它们不能被声明为 `private` 或 `final` 。
>
> `@Bean` methods are also discovered on base classes of a given component or configuration class, as well as on Java 8 default methods declared in interfaces implemented by the component or configuration class. This allows for a lot of flexibility in composing complex configuration arrangements, with even multiple inheritance being possible through Java 8 default methods as of Spring 4.2.
> `@Bean` 方法也可以在给定组件或配置类的基类上发现，以及在组件或配置类实现的接口中声明的Java 8默认方法上发现。这允许在组成复杂的配置安排时有很大的灵活性，甚至可以通过Spring 4.2的Java 8默认方法实现多继承。
>
> Finally, a single class may hold multiple `@Bean` methods for the same bean, as an arrangement of multiple factory methods to use depending on available dependencies at runtime. This is the same algorithm as for choosing the “greediest” constructor or factory method in other configuration scenarios: The variant with the largest number of satisfiable dependencies is picked at construction time, analogous to how the container selects between multiple `@Autowired` constructors.
> 最后，一个类可以为同一个bean保存多个 `@Bean` 方法，作为多个工厂方法的安排，以在运行时根据可用的依赖关系使用。这与在其他配置场景中选择“最贪婪”的构造函数或工厂方法的算法相同：具有最大数量的可满足依赖项的变体在构造时被挑选，类似于容器如何在多个 `@Autowired` 构造器之间进行选择。

#### 1.10.6.命名自动检测到的组件

When a component is autodetected as part of the scanning process, its bean name is generated by the `BeanNameGenerator` strategy known to that scanner. By default, any Spring stereotype annotation (`@Component`, `@Repository`, `@Service`, and `@Controller`) that contains a name `value` thereby provides that name to the corresponding bean definition.
当组件作为扫描过程的一部分被自动检测时，其bean名称由该扫描程序已知的 `BeanNameGenerator` 策略生成。默认情况下，任何包含名称 `value` 的Spring原型注释（ `@Component` 、 `@Repository` 、 `@Service` 和 `@Controller` ）都会将该名称提供给相应的bean定义。

If such an annotation contains no name `value` or for any other detected component (such as those discovered by custom filters), the default bean name generator returns the uncapitalized non-qualified class name. For example, if the following component classes were detected, the names would be `myMovieLister` and `movieFinderImpl`:
如果这样的注释不包含名称 `value` 或任何其他检测到的组件（例如自定义过滤器发现的组件），则默认的bean名称生成器返回未大写的非限定类名。例如，如果检测到以下组件类，则名称将为 `myMovieLister` 和 `movieFinderImpl` ：

```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

```java
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

If you do not want to rely on the default bean-naming strategy, you can provide a custom bean-naming strategy. First, implement the [`BeanNameGenerator`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/support/BeanNameGenerator.html) interface, and be sure to include a default no-arg constructor. Then, provide the fully qualified class name when configuring the scanner, as the following example annotation and bean definition show.
如果不想依赖默认的bean命名策略，可以提供自定义bean命名策略。首先，实现 `BeanNameGenerator` 接口，并确保包含默认的无参数构造函数。然后，在配置扫描器时提供完全限定的类名，如下面的示例注释和bean定义所示。

> if you run into naming conflicts due to multiple autodetected components having the same non-qualified class name (i.e., classes with identical names but residing in different packages), you may need to configure a `BeanNameGenerator` that defaults to the fully qualified class name for the generated bean name. As of Spring Framework 5.2.3, the `FullyQualifiedAnnotationBeanNameGenerator` located in package `org.springframework.context.annotation` can be used for such purposes.
> 如果由于多个自动检测到的组件具有相同的非限定类名（即，类具有相同的名称，但驻留在不同的包中），您可能需要配置一个 `BeanNameGenerator` ，它默认为生成的bean名称的完全限定类名。从Spring Framework 5.2.3开始，位于包 `org.springframework.context.annotation` 中的 `FullyQualifiedAnnotationBeanNameGenerator` 可以用于此目的。

```java
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    // ...
}
```

```xml
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

As a general rule, consider specifying the name with the annotation whenever other components may be making explicit references to it. On the other hand, the auto-generated names are adequate whenever the container is responsible for wiring.
作为一般规则，考虑在其他组件可能显式引用它时使用注释指定名称。另一方面，当容器负责连接时，自动生成的名称就足够了。

#### 1.10.7.为自动检测的组件提供作用域

As with Spring-managed components in general, the default and most common scope for autodetected components is `singleton`. However, sometimes you need a different scope that can be specified by the `@Scope` annotation. You can provide the name of the scope within the annotation, as the following example shows:
与Spring管理的组件一样，自动检测组件的默认和最常见的作用域是 `singleton` 。但是，有时候您需要一个可以由 `@Scope` 注释指定的不同作用域。您可以在注释中提供作用域的名称，如下例所示：

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

> `@Scope` annotations are only introspected on the concrete bean class (for annotated components) or the factory method (for `@Bean` methods). In contrast to XML bean definitions, there is no notion of bean definition inheritance, and inheritance hierarchies at the class level are irrelevant for metadata purposes.
> `@Scope` 注释仅在具体bean类（对于注释的组件）或工厂方法（对于 `@Bean` 方法）上进行内省。与XML bean定义相反，没有bean定义继承的概念，并且类级别的继承层次结构与元数据目的无关。

For details on web-specific scopes such as “request” or “session” in a Spring context, see [Request, Session, Application, and WebSocket Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other). As with the pre-built annotations for those scopes, you may also compose your own scoping annotations by using Spring’s meta-annotation approach: for example, a custom annotation meta-annotated with `@Scope("prototype")`, possibly also declaring a custom scoped-proxy mode.
有关Spring上下文中特定于web的作用域（如“request”或“session”）的详细信息，请参阅[Request，Session，Application和WebSocket作用域](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other)。与这些作用域的预构建注释一样，您也可以通过使用Spring的元注释方法来编写自己的作用域注释：例如，用 `@Scope("prototype")` 元注释的定制注释，可能还声明定制范围代理模式。

> To provide a custom strategy for scope resolution rather than relying on the annotation-based approach, you can implement the [`ScopeMetadataResolver`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/ScopeMetadataResolver.html) interface. Be sure to include a default no-arg constructor. Then you can provide the fully qualified class name when configuring the scanner, as the following example of both an annotation and a bean definition shows:
> 要为范围解析提供自定义策略，而不是依赖于基于注释的方法，您可以实现 `ScopeMetadataResolver` 接口。确保包含默认的无参数构造函数。然后，您可以在配置扫描器时提供完全限定的类名，如以下注释和bean定义的示例所示：

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    // ...
}
```

```xml
<beans>
    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>
```

When using certain non-singleton scopes, it may be necessary to generate proxies for the scoped objects. The reasoning is described in [Scoped Beans as Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection). For this purpose, a scoped-proxy attribute is available on the component-scan element. The three possible values are: `no`, `interfaces`, and `targetClass`. For example, the following configuration results in standard JDK dynamic proxies:
当使用某些非单例作用域时，可能需要为作用域对象生成代理。在[Scoped Beans as Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)中描述了推理过程。为此，component-scan元素上提供了一个scoped-proxy属性。三个可能的值是： `no` 、 `interfaces` 和 `targetClass` 。例如，以下配置会导致标准JDK动态代理：

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    // ...
}
```

```xml
<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

#### 1.10.8.提供带有注释的限定符元数据

The `@Qualifier` annotation is discussed in [Fine-tuning Annotation-based Autowiring with Qualifiers](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation-qualifiers). The examples in that section demonstrate the use of the `@Qualifier` annotation and custom qualifier annotations to provide fine-grained control when you resolve autowire candidates. Because those examples were based on XML bean definitions, the qualifier metadata was provided on the candidate bean definitions by using the `qualifier` or `meta` child elements of the `bean` element in the XML. When relying upon classpath scanning for auto-detection of components, you can provide the qualifier metadata with type-level annotations on the candidate class. The following three examples demonstrate this technique:
`@Qualifier` 注释将在使用限定符微调基于注释的自动装配中讨论。该部分中的示例演示了如何使用 `@Qualifier` 注释和自定义限定符注释在解析自动连接候选项时提供细粒度的控制。因为这些示例基于XMLbean定义，所以通过使用XML中 `bean` 元素的 `qualifier` 或 `meta` 子元素，在候选bean定义上提供限定符元数据。当依赖于类路径扫描来自动检测组件时，可以为候选类提供带有类型级别注释的限定符元数据。以下三个示例演示了此技术：

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

> As with most annotation-based alternatives, keep in mind that the annotation metadata is bound to the class definition itself, while the use of XML allows for multiple beans of the same type to provide variations in their qualifier metadata, because that metadata is provided per-instance rather than per-class.
> 与大多数基于注释的替代方案一样，请记住注释元数据绑定到类定义本身，而XML的使用允许相同类型的多个bean在其限定符元数据中提供变体，因为该元数据是按实例而不是按类提供的。

#### 1.10.9.生成候选组件的索引

While classpath scanning is very fast, it is possible to improve the startup performance of large applications by creating a static list of candidates at compilation time. In this mode, all modules that are targets of component scanning must use this mechanism.
虽然类路径扫描非常快，但通过在编译时创建候选项的静态列表，可以提高大型应用程序的启动性能。在此模式下，作为组件扫描目标的所有模块都必须使用此机制。

> Your existing `@ComponentScan` or `<context:component-scan/>` directives must remain unchanged to request the context to scan candidates in certain packages. When the `ApplicationContext` detects such an index, it automatically uses it rather than scanning the classpath.
> 您现有的 `@ComponentScan` 或 `<context:component-scan/>` 指令必须保持不变，以请求上下文扫描某些包中的候选项。当 `ApplicationContext` 检测到这样的索引时，它会自动使用它，而不是扫描类路径。

To generate the index, add an additional dependency to each module that contains components that are targets for component scan directives. The following example shows how to do so with Maven:
若要生成索引，请向包含作为组件扫描指令目标的组件的每个模块添加一个附加依赖项。下面的示例展示了如何使用Maven执行此操作：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>6.0.8</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

With Gradle 4.5 and earlier, the dependency should be declared in the `compileOnly` configuration, as shown in the following example:
对于Gradle 4.5及更早版本，应在 `compileOnly` 配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    compileOnly "org.springframework:spring-context-indexer:6.0.8"
}
```

With Gradle 4.6 and later, the dependency should be declared in the `annotationProcessor` configuration, as shown in the following example:
对于Gradle 4.6及更高版本，应在 `annotationProcessor` 配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    annotationProcessor "org.springframework:spring-context-indexer:6.0.8"
}
```

The `spring-context-indexer` artifact generates a `META-INF/spring.components` file that is included in the jar file.
`spring-context-indexer` 工件生成一个包含在jar文件中的 `META-INF/spring.components` 文件。

> When working with this mode in your IDE, the `spring-context-indexer` must be registered as an annotation processor to make sure the index is up-to-date when candidate components are updated.
> 在IDE中使用此模式时，必须将 `spring-context-indexer` 注册为注释处理器，以确保更新候选组件时索引是最新的。

> The index is enabled automatically when a `META-INF/spring.components` file is found on the classpath. If an index is partially available for some libraries (or use cases) but could not be built for the whole application, you can fall back to a regular classpath arrangement (as though no index were present at all) by setting `spring.index.ignore` to `true`, either as a JVM system property or via the [`SpringProperties`](https://docs.spring.io/spring-framework/docs/current/reference/html/appendix.html#appendix-spring-properties) mechanism.
> 当在类路径中找到 `META-INF/spring.components` 文件时，索引会自动启用。如果一个索引对于某些库（或用例）是部分可用的，但不能为整个应用程序构建，那么您可以通过将 `spring.index.ignore` 设置为 `true` ，或者作为JVM系统属性，或者通过 [`SpringProperties`](https://docs.spring.io/spring-framework/docs/current/reference/html/appendix.html#appendix-spring-properties) 机制，返回到常规的类路径安排（就像根本不存在索引一样）。

### 1.11.使用JSR 330标准注释

Spring offers support for JSR-330 standard annotations (Dependency Injection). Those annotations are scanned in the same way as the Spring annotations. To use them, you need to have the relevant jars in your classpath.
Spring提供了对JSR-330标准注释（依赖注入）的支持。这些注释的扫描方式与Spring注释相同。要使用它们，您需要在类路径中拥有相关的jar。

> If you use Maven, the `jakarta.inject` artifact is available in the standard Maven repository ( https://repo.maven.apache.org/maven2/jakarta/inject/jakarta.inject-api/2.0.0/). You can add the following dependency to your file pom.xml:
> 如果您使用Maven，那么 `jakarta.inject` 工件在标准Maven存储库（ https://repo.maven.apache.org/maven2/jakarta/inject/jakarta.inject-api/2.0.0/ ）中可用。可以将以下依赖项添加到文件pom.xml中：
>
> ```xml
> <dependency>
>     <groupId>jakarta.inject</groupId>
>     <artifactId>jakarta.inject-api</artifactId>
>     <version>2.0.0</version>
> </dependency>
> ```

#### 1.11.1.使用 `@Inject` 和 `@Named` 进行依赖注入

Instead of `@Autowired`, you can use `@jakarta.inject.Inject` as follows:
你可以使用 `@jakarta.inject.Inject` 来代替 `@Autowired` ，如下所示：

```java
import jakarta.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        // ...
    }
}
```

As with `@Autowired`, you can use `@Inject` at the field level, method level and constructor-argument level. Furthermore, you may declare your injection point as a `Provider`, allowing for on-demand access to beans of shorter scopes or lazy access to other beans through a `Provider.get()` call. The following example offers a variant of the preceding example:
与 `@Autowired` 一样，你可以在字段级别、方法级别和构造函数参数级别使用 `@Inject` 。此外，您可以将您的注入点声明为 `Provider` ，允许按需访问范围较短的bean或通过 `Provider.get()` 调用延迟访问其他bean。下面的示例提供了前面示例的变体：

```java
import jakarta.inject.Inject;
import jakarta.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        // ...
    }
}
```

If you would like to use a qualified name for the dependency that should be injected, you should use the `@Named` annotation, as the following example shows:
如果您希望为应该注入的依赖项使用限定名，则应该使用 `@Named` 注释，如以下示例所示：

```java
import jakarta.inject.Inject;
import jakarta.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

As with `@Autowired`, `@Inject` can also be used with `java.util.Optional` or `@Nullable`. This is even more applicable here, since `@Inject` does not have a `required` attribute. The following pair of examples show how to use `@Inject` and `@Nullable`:
与 `@Autowired` 一样， `@Inject` 也可以与 `java.util.Optional` 或 `@Nullable` 一起使用。这在这里甚至更适用，因为 `@Inject` 没有 `required` 属性。下面的两个例子展示了如何使用 `@Inject` 和 `@Nullable` ：

```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        // ...
    }
}
```

```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        // ...
    }
}
```

#### 1.11.2. `@Named` 和 `@ManagedBean` ： `@Component` 注释的标准等同物

Instead of `@Component`, you can use `@jakarta.inject.Named` or `jakarta.annotation.ManagedBean`, as the following example shows:
您可以使用 `@jakarta.inject.Named` 或 `jakarta.annotation.ManagedBean` 代替 `@Component` ，如下例所示：

```java
import jakarta.inject.Inject;
import jakarta.inject.Named;

@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

It is very common to use `@Component` without specifying a name for the component. `@Named` can be used in a similar fashion, as the following example shows:
使用 `@Component` 而不指定组件名称是非常常见的。 `@Named` 可以类似的方式使用，如下例所示：

```java
import jakarta.inject.Inject;
import jakarta.inject.Named;

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

When you use `@Named` or `@ManagedBean`, you can use component scanning in the exact same way as when you use Spring annotations, as the following example shows:
当您使用 `@Named` 或 `@ManagedBean` 时，您可以使用与使用Spring annotations完全相同的方式使用组件扫描，如以下示例所示：

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

> In contrast to `@Component`, the JSR-330 `@Named` and the JSR-250 `@ManagedBean` annotations are not composable. You should use Spring’s stereotype model for building custom component annotations.
> 与 `@Component` 相反，JSR-330 `@Named` 和JSR-250 `@ManagedBean` 注释是不可组合的。您应该使用Spring的构造型模型来构建自定义组件注释。

####  1.11.3. JSR-330标准注释的局限性

When you work with standard annotations, you should know that some significant features are not available, as the following table shows:
使用标准注释时，您应该知道一些重要功能不可用，如下表所示：

| Spring              | jakarta.inject.*      | jakarta.inject restrictions / comments                       |
| :------------------ | :-------------------- | :----------------------------------------------------------- |
| @Autowired          | @Inject               | `@Inject` has no 'required' attribute. Can be used with Java 8’s `Optional` instead. `@Inject` 没有'required'属性。可以与Java 8的 `Optional` 一起使用。 |
| @Component          | @Named / @ManagedBean | JSR-330 does not provide a composable model, only a way to identify named components. JSR-330没有提供可组合的模型，只提供了一种识别命名组件的方法。 |
| @Scope("singleton") | @Singleton            | The JSR-330 default scope is like Spring’s `prototype`. However, in order to keep it consistent with Spring’s general defaults, a JSR-330 bean declared in the Spring container is a `singleton` by default. In order to use a scope other than `singleton`, you should use Spring’s `@Scope` annotation. `jakarta.inject` also provides a `jakarta.inject.Scope` annotation: however, this one is only intended to be used for creating custom annotations. JSR-330的默认作用域类似于Spring的 `prototype` 。然而，为了与Spring的一般默认值保持一致，Spring容器中声明的JSR-330 bean默认为 `singleton` 。为了使用除 `singleton` 之外的作用域，您应该使用Spring的 `@Scope` 注释。 `jakarta.inject` 还提供了一个 `jakarta.inject.Scope` 注释：但是，此注释仅用于创建自定义注释。 |
| @Qualifier          | @Qualifier / @Named   | `jakarta.inject.Qualifier` is just a meta-annotation for building custom qualifiers. Concrete `String` qualifiers (like Spring’s `@Qualifier` with a value) can be associated through `jakarta.inject.Named`. `jakarta.inject.Qualifier` 只是一个用于构建自定义限定符的元注释。具体的 `String` 限定符（如Spring的带值的 `@Qualifier` ）可以通过 `jakarta.inject.Named` 关联。 |
| @Value              | -                     | no equivalent                                                |
| @Lazy               | -                     | no equivalent                                                |
| ObjectFactory       | Provider              | `jakarta.inject.Provider` is a direct alternative to Spring’s `ObjectFactory`, only with a shorter `get()` method name. It can also be used in combination with Spring’s `@Autowired` or with non-annotated constructors and setter methods. `jakarta.inject.Provider` 是Spring的 `ObjectFactory` 的直接替代品，只是有一个更短的 `get()` 方法名称。它还可以与Spring的 `@Autowired` 或无注释的构造函数和setter方法结合使用。 |

### 1.12.基于Java的容器配置

This section covers how to use annotations in your Java code to configure the Spring container. It includes the following topics:
本节介绍如何在Java代码中使用注释来配置Spring容器。它包括以下主题：

- [基本概念： @Bean 和 @Configuration `](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-basic-concepts)
- [使用 AnnotationConfigApplicationContext 实例化Spring容器`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-instantiating-container)
- [使用 `@Bean` 注释](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-bean-annotation)
- [使用 `@Configuration` 注释](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-configuration-annotation)
- [组合基于Java的配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-composing-configuration-classes)
- [Bean定义配置文件](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles)
- [`PropertySource` Abstraction](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction)
- [使用 `@PropertySource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-using-propertysource)
- [语句中的占位符解析](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-placeholder-resolution-in-statements)

#### 1.12.1.基本概念： `@Bean` 和 `@Configuration`

The central artifacts in Spring’s Java configuration support are `@Configuration`-annotated classes and `@Bean`-annotated methods.
Spring的Java配置支持中的核心构件是用 `@Configuration` 注释的类和 用`@Bean` 注释的方法。

The `@Bean` annotation is used to indicate that a method instantiates, configures, and initializes a new object to be managed by the Spring IoC container. For those familiar with Spring’s `<beans/>` XML configuration, the `@Bean` annotation plays the same role as the `<bean/>` element. You can use `@Bean`-annotated methods with any Spring `@Component`. However, they are most often used with `@Configuration` beans.
`@Bean` 注释用于指示方法实例化，配置和初始化要由Spring IoC容器管理的新对象。对于那些熟悉Spring的 `<beans/>` XML配置的人来说， `@Bean` 注释扮演着与 `<bean/>` 元素相同的角色。您可以在任何Spring `@Component` 中使用 `@Bean` 注释方法。但是，它们最常用于 `@Configuration` bean。

Annotating a class with `@Configuration` indicates that its primary purpose is as a source of bean definitions. Furthermore, `@Configuration` classes let inter-bean dependencies be defined by calling other `@Bean` methods in the same class. The simplest possible `@Configuration` class reads as follows:
用 `@Configuration` 注释一个类表示它的主要用途是作为bean定义的源。此外， `@Configuration` 类允许通过调用同一个类中的其他 `@Bean` 方法来定义bean间的依赖关系。最简单的 `@Configuration` 类如下所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public MyServiceImpl myService() {
        return new MyServiceImpl();
    }
}
```

The preceding `AppConfig` class is equivalent to the following Spring `<beans/>` XML:
前面的 `AppConfig` 类等效于以下Spring `<beans/>` XML：

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

> Full @Configuration vs “lite” @Bean mode?
> Full @Configuration与“lite”@Bean模式？
>
> When `@Bean` methods are declared within classes that are not annotated with `@Configuration`, they are referred to as being processed in a “lite” mode. Bean methods declared in a `@Component` or even in a plain old class are considered to be “lite”, with a different primary purpose of the containing class and a `@Bean` method being a sort of bonus there. For example, service components may expose management views to the container through an additional `@Bean` method on each applicable component class. In such scenarios, `@Bean` methods are a general-purpose factory method mechanism.
> 当在没有用 `@Configuration` 注释的类中声明 `@Bean` 方法时，它们被称为以“精简”模式处理。在 `@Component` 甚至普通的旧类中声明的Bean方法被认为是“精简”的，包含类的主要用途不同，而 `@Bean` 方法是一种额外的好处。例如，服务组件可以通过每个适用组件类上的附加 `@Bean` 方法向容器公开管理视图。在这种情况下， `@Bean` 方法是一种通用的工厂方法机制。
>
> Unlike full `@Configuration`, lite `@Bean` methods cannot declare inter-bean dependencies. Instead, they operate on their containing component’s internal state and, optionally, on arguments that they may declare. Such a `@Bean` method should therefore not invoke other `@Bean` methods. Each such method is literally only a factory method for a particular bean reference, without any special runtime semantics. The positive side-effect here is that no CGLIB subclassing has to be applied at runtime, so there are no limitations in terms of class design (that is, the containing class may be `final` and so forth).
> 与完整的 `@Configuration` 不同，lite `@Bean` 方法不能声明bean间的依赖关系。相反，它们对包含它们的组件的内部状态进行操作，并且可选地，对它们可能声明的参数进行操作。因此，这样的 `@Bean` 方法不应该调用其他 `@Bean` 方法。每个这样的方法实际上只是特定bean引用的工厂方法，没有任何特殊的运行时语义。这里的积极副作用是，在运行时不必应用CGLIB子类，因此在类设计方面没有限制（也就是说，包含的类可以是 `final` 等等）。
>
> In common scenarios, `@Bean` methods are to be declared within `@Configuration` classes, ensuring that “full” mode is always used and that cross-method references therefore get redirected to the container’s lifecycle management. This prevents the same `@Bean` method from accidentally being invoked through a regular Java call, which helps to reduce subtle bugs that can be hard to track down when operating in “lite” mode.
> 在常见的场景中， `@Bean` 方法将在 `@Configuration` 类中声明，确保始终使用“full”模式，因此跨方法引用将被重定向到容器的生命周期管理。这可以防止同一个 `@Bean` 方法通过常规Java调用意外地被调用，这有助于减少在“精简”模式下操作时难以跟踪的细微错误。

The `@Bean` and `@Configuration` annotations are discussed in depth in the following sections. First, however, we cover the various ways of creating a spring container by using Java-based configuration.
以下章节将深入讨论 `@Bean` 和 `@Configuration` 注释。但是，首先，我们将介绍使用基于Java的配置创建spring容器的各种方法。

#### 1.12.2.使用 `AnnotationConfigApplicationContext` 实例化Spring容器

The following sections document Spring’s `AnnotationConfigApplicationContext`, introduced in Spring 3.0. This versatile `ApplicationContext` implementation is capable of accepting not only `@Configuration` classes as input but also plain `@Component` classes and classes annotated with JSR-330 metadata.
下面的部分记录了Spring 3.0中引入的Spring的 `AnnotationConfigApplicationContext` 。这个通用的 `ApplicationContext` 实现不仅能够接受 `@Configuration` 类作为输入，还能够接受普通的 `@Component` 类和用JSR-330元数据注释的类。

When `@Configuration` classes are provided as input, the `@Configuration` class itself is registered as a bean definition and all declared `@Bean` methods within the class are also registered as bean definitions.
当提供 `@Configuration` 类作为输入时， `@Configuration` 类本身被注册为bean定义，类中所有声明的 `@Bean` 方法也被注册为bean定义。

When `@Component` and JSR-330 classes are provided, they are registered as bean definitions, and it is assumed that DI metadata such as `@Autowired` or `@Inject` are used within those classes where necessary.
当提供 `@Component` 和JSR-330类时，它们被注册为bean定义，并且假设在必要时在这些类中使用诸如 `@Autowired` 或 `@Inject` 的DI元数据。

##### 简单结构

In much the same way that Spring XML files are used as input when instantiating a `ClassPathXmlApplicationContext`, you can use `@Configuration` classes as input when instantiating an `AnnotationConfigApplicationContext`. This allows for completely XML-free usage of the Spring container, as the following example shows:
与实例化 `ClassPathXmlApplicationContext` 时使用Spring XML文件作为输入的方式大致相同，您可以在实例化 `AnnotationConfigApplicationContext` 时使用 `@Configuration` 类作为输入。这允许Spring容器的完全无XML使用，如以下示例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

As mentioned earlier, `AnnotationConfigApplicationContext` is not limited to working only with `@Configuration` classes. Any `@Component` or JSR-330 annotated class may be supplied as input to the constructor, as the following example shows:
如前所述， `AnnotationConfigApplicationContext` 并不局限于只使用 `@Configuration` 类。任何 `@Component` 或JSR-330注释的类都可以作为输入提供给构造函数，如下例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

The preceding example assumes that `MyServiceImpl`, `Dependency1`, and `Dependency2` use Spring dependency injection annotations such as `@Autowired`.
前面的示例假设 `MyServiceImpl` 、 `Dependency1` 和 `Dependency2` 使用Spring依赖注入标注，如 `@Autowired` 。

##### 使用 `register(Class<?>…)` 以编程方式构建容器

You can instantiate an `AnnotationConfigApplicationContext` by using a no-arg constructor and then configure it by using the `register()` method. This approach is particularly useful when programmatically building an `AnnotationConfigApplicationContext`. The following example shows how to do so:
您可以使用无参数构造函数实例化 `AnnotationConfigApplicationContext` ，然后使用 `register()` 方法配置它。这种方法在以编程方式构建 `AnnotationConfigApplicationContext` 时特别有用。下面的示例说明如何执行此操作：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

##### 使用 `scan(String…)` 启用组件扫描

To enable component scanning, you can annotate your `@Configuration` class as follows:
要启用组件扫描，您可以按如下方式注释 `@Configuration` 类：

```java
@Configuration
@ComponentScan(basePackages = "com.acme") (1)
public class AppConfig  {
    // ...
}
```

1.  This annotation enables component scanning. 此注释启用元件扫描。

> Experienced Spring users may be familiar with the XML declaration equivalent from Spring’s `context:` namespace, shown in the following example:
> 有经验的Spring用户可能熟悉Spring的 `context:` 命名空间的XML声明，如以下示例所示：
>
> ```xml
> <beans>
>     <context:component-scan base-package="com.acme"/>
> </beans>
> ```

In the preceding example, the `com.acme` package is scanned to look for any `@Component`-annotated classes, and those classes are registered as Spring bean definitions within the container. `AnnotationConfigApplicationContext` exposes the `scan(String…)` method to allow for the same component-scanning functionality, as the following example shows:
在前面的示例中，扫描 `com.acme` 包以查找任何 `@Component` 注释的类，并且这些类在容器中注册为Spring bean定义。 `AnnotationConfigApplicationContext` 公开 `scan(String…)` 方法以允许相同的组件扫描功能，如下例所示：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

> Remember that `@Configuration` classes are [meta-annotated](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations) with `@Component`, so they are candidates for component-scanning. In the preceding example, assuming that `AppConfig` is declared within the `com.acme` package (or any package underneath), it is picked up during the call to `scan()`. Upon `refresh()`, all its `@Bean` methods are processed and registered as bean definitions within the container.
> 请记住， `@Configuration` 类使用 `@Component` 进行了元注释，因此它们是组件扫描的候选对象。在前面的例子中，假设在 `com.acme` 包（或其下的任何包）中声明了 `AppConfig` ，则在调用 `scan()` 时会拾取它。在 `refresh()` 时，它的所有 `@Bean` 方法都被处理并注册为容器中的bean定义。

##### 使用 `AnnotationConfigWebApplicationContext` 支持Web应用程序

A `WebApplicationContext` variant of `AnnotationConfigApplicationContext` is available with `AnnotationConfigWebApplicationContext`. You can use this implementation when configuring the Spring `ContextLoaderListener` servlet listener, Spring MVC `DispatcherServlet`, and so forth. The following `web.xml` snippet configures a typical Spring MVC web application (note the use of the `contextClass` context-param and init-param):
`AnnotationConfigApplicationContext` 的 `WebApplicationContext` 变体可用于 `AnnotationConfigWebApplicationContext` 。您可以在配置Spring `ContextLoaderListener` servlet listener、Spring MVC `DispatcherServlet` 等时使用此实现。下面的 `web.xml` 代码片段配置了一个典型的Spring MVC Web应用程序（注意 `contextClass` context-param和init-param的使用）：

```xml
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

> For programmatic use cases, a `GenericWebApplicationContext` can be used as an alternative to `AnnotationConfigWebApplicationContext`. See the [`GenericWebApplicationContext`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/web/context/support/GenericWebApplicationContext.html) javadoc for details.
> 对于编程用例， `GenericWebApplicationContext` 可以用作 `AnnotationConfigWebApplicationContext` 的替代方案。请参阅  [`GenericWebApplicationContext`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/web/context/support/GenericWebApplicationContext.html)javadoc了解详细信息。

#### 1.12.3.使用 `@Bean` 注释

`@Bean` is a method-level annotation and a direct analog of the XML `<bean/>` element. The annotation supports some of the attributes offered by `<bean/>`, such as:
`@Bean` 是一个方法级注释，是XML `<bean/>` 元素的直接模拟。注释支持 `<bean/>` 提供的一些属性，例如：

- [init-method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean)
- [destroy-method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean)
- [autowiring](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)
- `name`.

You can use the `@Bean` annotation in a `@Configuration`-annotated or in a `@Component`-annotated class.
您可以在带 `@Configuration` 注释的类或带 `@Component` 注释的类中使用 `@Bean` 注释。

##### 声明Bean

To declare a bean, you can annotate a method with the `@Bean` annotation. You use this method to register a bean definition within an `ApplicationContext` of the type specified as the method’s return value. By default, the bean name is the same as the method name. The following example shows a `@Bean` method declaration:
要声明一个bean，你可以用 `@Bean` 注释来注释一个方法。您可以使用此方法在指定为方法返回值的类型的 `ApplicationContext` 中注册bean定义。默认情况下，bean名称与方法名称相同。下面的示例显示了一个 `@Bean` 方法声明：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

The preceding configuration is exactly equivalent to the following Spring XML:
前面的配置与下面的Spring XML完全等效：

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

Both declarations make a bean named `transferService` available in the `ApplicationContext`, bound to an object instance of type `TransferServiceImpl`, as the following text image shows:
这两个声明都使名为 `transferService` 的bean在 `ApplicationContext` 中可用，绑定到类型为 `TransferServiceImpl` 的对象实例，如以下文本图像所示：

```
transferService -> com.acme.TransferServiceImpl
```

You can also use default methods to define beans. This allows composition of bean configurations by implementing interfaces with bean definitions on default methods.
您还可以使用默认方法来定义bean。这允许通过在默认方法上实现具有bean定义的接口来组合bean配置。

```java
public interface BaseConfig {

    @Bean
    default TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}

@Configuration
public class AppConfig implements BaseConfig {

}
```

You can also declare your `@Bean` method with an interface (or base class) return type, as the following example shows:
你也可以用接口（或基类）返回类型声明你的 `@Bean` 方法，如下面的例子所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

However, this limits the visibility for advance type prediction to the specified interface type (`TransferService`). Then, with the full type (`TransferServiceImpl`) known to the container only once the affected singleton bean has been instantiated. Non-lazy singleton beans get instantiated according to their declaration order, so you may see different type matching results depending on when another component tries to match by a non-declared type (such as `@Autowired TransferServiceImpl`, which resolves only once the `transferService` bean has been instantiated).
然而，这将高级类型预测的可见性限制到指定的接口类型（ `TransferService` ）。然后，只有当受影响的单例bean被实例化时，容器才知道完整的类型（ `TransferServiceImpl` ）。非惰性单例bean根据其声明顺序进行实例化，因此您可能会看到不同的类型匹配结果，这取决于另一个组件何时尝试通过未声明的类型进行匹配（例如 `@Autowired TransferServiceImpl` ，它只在 `transferService` bean实例化后才解析）。

> If you consistently refer to your types by a declared service interface, your `@Bean` return types may safely join that design decision. However, for components that implement several interfaces or for components potentially referred to by their implementation type, it is safer to declare the most specific return type possible (at least as specific as required by the injection points that refer to your bean).
> 如果你始终通过声明的服务接口引用你的类型，你的 `@Bean` 返回类型可以安全地加入那个设计决策。但是，对于实现多个接口的组件或可能由其实现类型引用的组件，尽可能声明最具体的返回类型（至少与引用Bean的注入点所要求的一样具体）会更安全。

##### Bean依赖

A `@Bean`-annotated method can have an arbitrary number of parameters that describe the dependencies required to build that bean. For instance, if our `TransferService` requires an `AccountRepository`, we can materialize that dependency with a method parameter, as the following example shows:
一个 `@Bean` 注释的方法可以有任意数量的参数来描述构建该bean所需的依赖关系。例如，如果我们的 `TransferService` 需要一个 `AccountRepository` ，我们可以用一个方法参数来具体化这个依赖关系，如下面的例子所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

The resolution mechanism is pretty much identical to constructor-based dependency injection. See [the relevant section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection) for more details.
解析机制与基于构造函数的依赖注入非常相似。有关详细信息，请参阅[相关章节](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection)。

##### 接收生命周期回调

Any classes defined with the `@Bean` annotation support the regular lifecycle callbacks and can use the `@PostConstruct` and `@PreDestroy` annotations from JSR-250. See [JSR-250 annotations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations) for further details.
使用 `@Bean` 注释定义的任何类都支持常规的生命周期回调，并且可以使用JSR-250中的 `@PostConstruct` 和 `@PreDestroy` 注释。更多细节请参见[JSR-250注释](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)。

The regular Spring [lifecycle](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-nature) callbacks are fully supported as well. If a bean implements `InitializingBean`, `DisposableBean`, or `Lifecycle`, their respective methods are called by the container.
也完全支持常规的Spring生命周期回调。如果bean实现了 `InitializingBean` 、 `DisposableBean` 或 `Lifecycle` ，容器将调用它们各自的方法。

The standard set of `*Aware` interfaces (such as [BeanFactoryAware](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory), [BeanNameAware](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware), [MessageSourceAware](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-messagesource), [ApplicationContextAware](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware), and so on) are also fully supported.
还完全支持标准的 `*Aware` 接口集（例如BeanFactoryAware、BeanNameAware、MessageSourceAware、ApplicationContextAware等）。

The `@Bean` annotation supports specifying arbitrary initialization and destruction callback methods, much like Spring XML’s `init-method` and `destroy-method` attributes on the `bean` element, as the following example shows:
`@Bean` annotation支持指定任意的初始化和析构回调方法，就像Spring XML在 `bean` 元素上的 `init-method` 和 `destroy-method` 属性一样，如下例所示：

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

> By default, beans defined with Java configuration that have a public `close` or `shutdown` method are automatically enlisted with a destruction callback. If you have a public `close` or `shutdown` method and you do not wish for it to be called when the container shuts down, you can add `@Bean(destroyMethod = "")` to your bean definition to disable the default `(inferred)` mode.
> 默认情况下，使用Java配置定义的具有公共 `close` 或 `shutdown` 方法的bean会自动使用析构回调进行登记。如果你有一个公共的 `close` 或 `shutdown` 方法，并且你不希望它在容器关闭时被调用，你可以将 `@Bean(destroyMethod = "")` 添加到你的bean定义中以禁用默认的 `(inferred)` 模式。
>
> You may want to do that by default for a resource that you acquire with JNDI, as its lifecycle is managed outside the application. In particular, make sure to always do it for a `DataSource`, as it is known to be problematic on Jakarta EE application servers.
> 对于使用JNDI获取的资源，您可能希望在默认情况下这样做，因为其生命周期是在应用程序外部管理的。特别是，请确保始终对 `DataSource` 执行此操作，因为众所周知，在Jakarta EE应用程序服务器上这是有问题的。
>
> The following example shows how to prevent an automatic destruction callback for a `DataSource`:
> 以下示例显示如何防止 `DataSource` 的自动销毁回调：
>
> ```java
> @Bean(destroyMethod = "")
> public DataSource dataSource() throws NamingException {
>     return (DataSource) jndiTemplate.lookup("MyDS");
> }
> ```
>
> Also, with `@Bean` methods, you typically use programmatic JNDI lookups, either by using Spring’s `JndiTemplate` or `JndiLocatorDelegate` helpers or straight JNDI `InitialContext` usage but not the `JndiObjectFactoryBean` variant (which would force you to declare the return type as the `FactoryBean` type instead of the actual target type, making it harder to use for cross-reference calls in other `@Bean` methods that intend to refer to the provided resource here).
> 此外，对于 `@Bean` 方法，您通常使用编程JNDI查找，通过使用Spring的 `JndiTemplate` 或 `JndiLocatorDelegate` helper或直接使用JNDI `InitialContext` 而不是 `JndiObjectFactoryBean` 变体（这将迫使您将返回类型声明为 `FactoryBean` 类型，而不是实际的目标类型，这使得它更难用于其他 `@Bean` 方法中的交叉引用调用，这些方法旨在引用此处提供的资源）。

In the case of `BeanOne` from the example above the preceding note, it would be equally valid to call the `init()` method directly during construction, as the following example shows:
在前面注释中的示例中的 `BeanOne` 的情况下，在构造过程中直接调用 `init()` 方法同样有效，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        BeanOne beanOne = new BeanOne();
        beanOne.init();
        return beanOne;
    }

    // ...
}
```

> When you work directly in Java, you can do anything you like with your objects and do not always need to rely on the container lifecycle.
> 当你直接在Java中工作时，你可以对你的对象做任何你喜欢的事情，而不总是需要依赖于容器的生命周期。

##### 指定Bean范围

Spring includes the `@Scope` annotation so that you can specify the scope of a bean.
Spring包含了 `@Scope` 注解，这样你就可以指定bean的作用域。

###### 使用 `@Scope` 注释

You can specify that your beans defined with the `@Bean` annotation should have a specific scope. You can use any of the standard scopes specified in the [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) section.
您可以指定使用 `@Bean` 注释定义的bean应该具有特定的范围。您可以使用[Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes)部分中指定的任何标准范围。

The default scope is `singleton`, but you can override this with the `@Scope` annotation, as the following example shows:
默认的作用域是 `singleton` ，但是你可以用 `@Scope` 注释覆盖它，如下例所示：

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

######  `@Scope` 和 `scoped-proxy`

Spring offers a convenient way of working with scoped dependencies through [scoped proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection). The easiest way to create such a proxy when using the XML configuration is the `<aop:scoped-proxy/>` element. Configuring your beans in Java with a `@Scope` annotation offers equivalent support with the `proxyMode` attribute. The default is `ScopedProxyMode.DEFAULT`, which typically indicates that no scoped proxy should be created unless a different default has been configured at the component-scan instruction level. You can specify `ScopedProxyMode.TARGET_CLASS`, `ScopedProxyMode.INTERFACES` or `ScopedProxyMode.NO`.
Spring提供了一种通过[作用域代理](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)处理作用域依赖项的方便方法。在使用XML配置时，创建此类代理的最简单方法是使用 `<aop:scoped-proxy/>` 元素。在Java中使用 `@Scope` 注释配置bean提供了与 `proxyMode` 属性等效的支持。默认值是 `ScopedProxyMode.DEFAULT` ，它通常表示不应创建任何作用域代理，除非在组件扫描指令级别配置了不同的默认值。您可以指定 `ScopedProxyMode.TARGET_CLASS` 、 `ScopedProxyMode.INTERFACES` 或 `ScopedProxyMode.NO` 。

If you port the scoped proxy example from the XML reference documentation (see [scoped proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)) to our `@Bean` using Java, it resembles the following:
如果您使用Java将XML参考文档中的作用域代理示例（请参阅[作用域代理](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)）移植到我们的 `@Bean` ，它将类似于以下内容：

```java
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

##### 自定义Bean命名

By default, configuration classes use a `@Bean` method’s name as the name of the resulting bean. This functionality can be overridden, however, with the `name` attribute, as the following example shows:
默认情况下，配置类使用 `@Bean` 方法的名称作为结果bean的名称。但是，可以使用 `name` 属性覆盖此功能，如下例所示：

```java
@Configuration
public class AppConfig {

    @Bean("myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

##### Bean别名

As discussed in [Naming Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname), it is sometimes desirable to give a single bean multiple names, otherwise known as bean aliasing. The `name` attribute of the `@Bean` annotation accepts a String array for this purpose. The following example shows how to set a number of aliases for a bean:
正如命名Bean中所讨论的，有时需要为一个bean指定多个名称，也称为bean别名。 `@Bean` 注释的 `name` 属性为此接受String数组。以下示例显示如何为Bean设置多个别名：

```java
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

##### Bean Description

Sometimes, it is helpful to provide a more detailed textual description of a bean. This can be particularly useful when beans are exposed (perhaps through JMX) for monitoring purposes.
有时候，提供bean的更详细的文本描述是很有帮助的。当为了监视目的而公开bean时（可能通过JMX），这可能特别有用。

To add a description to a `@Bean`, you can use the [`@Description`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/Description.html) annotation, as the following example shows:
要向 `@Bean` 添加描述，可以使用 `@Description` 注释，如下例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

#### 1.12.4.使用 `@Configuration` 注释

`@Configuration` is a class-level annotation indicating that an object is a source of bean definitions. `@Configuration` classes declare beans through `@Bean`-annotated methods. Calls to `@Bean` methods on `@Configuration` classes can also be used to define inter-bean dependencies. See [Basic Concepts: `@Bean` and `@Configuration`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-basic-concepts) for a general introduction.
`@Configuration` 是一个类级别的注释，指示一个对象是bean定义的源。 `@Configuration` 类通过 `@Bean` 注释的方法声明bean。调用 `@Configuration` 类上的 `@Bean` 方法也可以用来定义bean间的依赖关系。参见[基本概念： `@Bean` 和 `@Configuration`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-basic-concepts) 的基本介绍。

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

In the preceding example, `beanOne` receives a reference to `beanTwo` through constructor injection.
在前面的示例中， `beanOne` 通过构造函数注入接收到对 `beanTwo` 的引用。

> This method of declaring inter-bean dependencies works only when the `@Bean` method is declared within a `@Configuration` class. You cannot declare inter-bean dependencies by using plain `@Component` classes.
> 这种声明bean间依赖关系的方法只有在 `@Bean` 方法在 `@Configuration` 类中声明时才有效。你不能使用普通的 `@Component` 类来声明bean间的依赖关系。

##### 查找方法注入

As noted earlier, [lookup method injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection) is an advanced feature that you should use rarely. It is useful in cases where a singleton-scoped bean has a dependency on a prototype-scoped bean. Using Java for this type of configuration provides a natural means for implementing this pattern. The following example shows how to use lookup method injection:
如前所述，[查找方法注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection)是一个高级特性，您应该很少使用。在单例作用域bean依赖于原型域bean的情况下，它很有用。将Java用于这种类型的配置为实现这种模式提供了一种自然的方法。以下示例显示如何使用查找方法注入：

```java
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

By using Java configuration, you can create a subclass of `CommandManager` where the abstract `createCommand()` method is overridden in such a way that it looks up a new (prototype) command object. The following example shows how to do so:
通过使用Java配置，您可以创建 `CommandManager` 的子类，其中抽象的 `createCommand()` 方法被重写，以便查找新的（原型）命令对象。下面的示例说明如何执行此操作：

```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

##### 有关基于Java的配置如何在内部工作的更多信息

Consider the following example, which shows a `@Bean` annotated method being called twice:
考虑以下示例，其中显示了一个被调用两次的 `@Bean` 注释方法：

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

`clientDao()` has been called once in `clientService1()` and once in `clientService2()`. Since this method creates a new instance of `ClientDaoImpl` and returns it, you would normally expect to have two instances (one for each service). That definitely would be problematic: In Spring, instantiated beans have a `singleton` scope by default. This is where the magic comes in: All `@Configuration` classes are subclassed at startup-time with `CGLIB`. In the subclass, the child method checks the container first for any cached (scoped) beans before it calls the parent method and creates a new instance.
`clientDao()` 已在 `clientService1()` 和 `clientService2()` 中被调用一次。由于此方法创建了一个新的 `ClientDaoImpl` 实例并返回它，因此通常需要两个实例（每个服务一个）。这肯定会有问题：在Spring中，实例化的bean默认有一个 `singleton` 作用域。这就是魔法的由来：所有 `@Configuration` 类在启动时都被 `CGLIB` 子类化。在子类中，子方法在调用父方法并创建新实例之前，首先检查容器中是否有任何缓存（作用域）bean。

> The behavior could be different according to the scope of your bean. We are talking about singletons here.
> 根据bean的作用域，行为可能会有所不同。我们在这里谈论的是单例。

> It is not necessary to add CGLIB to your classpath because CGLIB classes are repackaged under the `org.springframework.cglib` package and included directly within the `spring-core` JAR.
> 没有必要将CGLIB添加到类路径中，因为CGLIB类在 `org.springframework.cglib` 包下重新打包，并直接包含在 `spring-core` JAR中。

> There are a few restrictions due to the fact that CGLIB dynamically adds features at startup-time. In particular, configuration classes must not be final. However, any constructors are allowed on configuration classes, including the use of `@Autowired` or a single non-default constructor declaration for default injection.
> 由于CGLIB在启动时动态添加特性，因此存在一些限制。特别是，配置类不能是final的。但是，配置类上允许任何构造函数，包括使用 `@Autowired` 或单个非默认构造函数声明进行默认注入。
>
> If you prefer to avoid any CGLIB-imposed limitations, consider declaring your `@Bean` methods on non-`@Configuration` classes (for example, on plain `@Component` classes instead) or by annotating your configuration class with `@Configuration(proxyBeanMethods = false)`. Cross-method calls between `@Bean` methods are then not intercepted, so you have to exclusively rely on dependency injection at the constructor or method level there.
> 如果你更喜欢避免任何CGLIB强加的限制，考虑在非 `@Configuration` 类上声明你的 `@Bean` 方法（例如，在普通的 `@Component` 类上），或者用 `@Configuration(proxyBeanMethods = false)` 注释你的配置类。 `@Bean` 方法之间的跨方法调用不会被拦截，因此您必须完全依赖于构造函数或方法级别的依赖注入。

#### 1.12.5.组合基于Java的配置

Spring’s Java-based configuration feature lets you compose annotations, which can reduce the complexity of your configuration.
Spring的基于Java的配置特性允许您编写注释，这可以降低配置的复杂性。

##### 使用 `@Import` 注释

Much as the `<import/>` element is used within Spring XML files to aid in modularizing configurations, the `@Import` annotation allows for loading `@Bean` definitions from another configuration class, as the following example shows:
就像在Spring XML文件中使用 `<import/>` 元素来帮助模块化配置一样， `@Import` 注释允许从另一个配置类加载 `@Bean` 定义，如以下示例所示：

```java
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

Now, rather than needing to specify both `ConfigA.class` and `ConfigB.class` when instantiating the context, only `ConfigB` needs to be supplied explicitly, as the following example shows:
现在，在实例化上下文时不需要同时指定 `ConfigA.class` 和 `ConfigB.class` ，只需要显式提供 `ConfigB` ，如以下示例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

This approach simplifies container instantiation, as only one class needs to be dealt with, rather than requiring you to remember a potentially large number of `@Configuration` classes during construction.
这种方法简化了容器实例化，因为只需要处理一个类，而不需要在构造过程中记住大量的 `@Configuration` 类。

> As of Spring Framework 4.2, `@Import` also supports references to regular component classes, analogous to the `AnnotationConfigApplicationContext.register` method. This is particularly useful if you want to avoid component scanning, by using a few configuration classes as entry points to explicitly define all your components.
> 从Spring Framework 4.2开始， `@Import` 也支持对常规组件类的引用，类似于 `AnnotationConfigApplicationContext.register` 方法。如果您希望通过使用一些配置类作为入口点来显式定义所有组件，从而避免组件扫描，这一点特别有用。

###### 在导入的 `@Bean` 定义上注入依赖项

The preceding example works but is simplistic. In most practical scenarios, beans have dependencies on one another across configuration classes. When using XML, this is not an issue, because no compiler is involved, and you can declare `ref="someBean"` and trust Spring to work it out during container initialization. When using `@Configuration` classes, the Java compiler places constraints on the configuration model, in that references to other beans must be valid Java syntax.
前面的例子可以工作，但过于简单。在大多数实际的场景中，bean在配置类中相互依赖。当使用XML时，这不是问题，因为不涉及编译器，并且您可以声明 `ref="someBean"` 并信任Spring在容器初始化期间解决它。当使用 `@Configuration` 类时，Java编译器对配置模型进行约束，因为对其他bean的引用必须是有效的Java语法。

Fortunately, solving this problem is simple. As [we already discussed](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-dependencies), a `@Bean` method can have an arbitrary number of parameters that describe the bean dependencies. Consider the following more real-world scenario with several `@Configuration` classes, each depending on beans declared in the others:
幸运的是，解决这个问题很简单。正如我们已经讨论过的，一个 `@Bean` 方法可以有任意数量的参数来描述bean的依赖关系。考虑以下更真实的场景，其中有几个 `@Configuration` 类，每个类都依赖于其他类中声明的bean：

```java
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

There is another way to achieve the same result. Remember that `@Configuration` classes are ultimately only another bean in the container: This means that they can take advantage of `@Autowired` and `@Value` injection and other features the same as any other bean.
还有另一种方法可以达到同样的效果。请记住， `@Configuration` 类最终只是容器中的另一个bean：这意味着它们可以利用 `@Autowired` 和 `@Value` 注入以及与任何其他bean相同的其他特性。

> Make sure that the dependencies you inject that way are of the simplest kind only. `@Configuration` classes are processed quite early during the initialization of the context, and forcing a dependency to be injected this way may lead to unexpected early initialization. Whenever possible, resort to parameter-based injection, as in the preceding example.
> 请确保您以这种方式注入的依赖项是最简单的类型。 `@Configuration` 类在上下文的初始化过程中很早就被处理了，强制依赖项以这种方式注入可能会导致意外的早期初始化。只要有可能，请采用基于参数的注入，如上例所示。
>
> Also, be particularly careful with `BeanPostProcessor` and `BeanFactoryPostProcessor` definitions through `@Bean`. Those should usually be declared as `static @Bean` methods, not triggering the instantiation of their containing configuration class. Otherwise, `@Autowired` and `@Value` may not work on the configuration class itself, since it is possible to create it as a bean instance earlier than [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html).
> 此外，要特别注意 `BeanPostProcessor` 和 `BeanFactoryPostProcessor` 到 `@Bean` 的定义。这些方法通常应该声明为 `static @Bean` 方法，而不是触发其包含的配置类的实例化。否则， `@Autowired` 和 `@Value` 可能无法在配置类本身上工作，因为可以在 `AutowiredAnnotationBeanPostProcessor` 之前将其创建为bean实例。

The following example shows how one bean can be autowired to another bean:
下面的示例显示了如何将一个bean自动连接到另一个bean：

```java
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

> Constructor injection in `@Configuration` classes is only supported as of Spring Framework 4.3. Note also that there is no need to specify `@Autowired` if the target bean defines only one constructor.
> `@Configuration` classes中的构造函数注入仅在Spring Framework 4.3中支持。还要注意，如果目标bean只定义了一个构造函数，则不需要指定 `@Autowired` 。

In the preceding scenario, using `@Autowired` works well and provides the desired modularity, but determining exactly where the autowired bean definitions are declared is still somewhat ambiguous. For example, as a developer looking at `ServiceConfig`, how do you know exactly where the `@Autowired AccountRepository` bean is declared? It is not explicit in the code, and this may be just fine. Remember that the [Spring Tools for Eclipse](https://spring.io/tools) provides tooling that can render graphs showing how everything is wired, which may be all you need. Also, your Java IDE can easily find all declarations and uses of the `AccountRepository` type and quickly show you the location of `@Bean` methods that return that type.
在前面的场景中，使用 `@Autowired` 工作得很好，并提供了所需的模块性，但确定autowired bean定义的确切位置仍然有些模糊。例如，作为一名开发人员，您如何确切地知道 `@Autowired AccountRepository` bean在哪里声明？它在代码中并不显式，这可能没问题。请记住，SpringToolsforEclipse提供了可以呈现图形的工具，这些图形可以显示所有内容是如何连接的，这可能就是您所需要的。此外，Java IDE可以轻松地找到 `AccountRepository` 类型的所有声明和使用，并快速显示返回该类型的 `@Bean` 方法的位置。

In cases where this ambiguity is not acceptable and you wish to have direct navigation from within your IDE from one `@Configuration` class to another, consider autowiring the configuration classes themselves. The following example shows how to do so:
如果这种不明确性是不可接受的，并且您希望在IDE中从一个 `@Configuration` 类直接导航到另一个 `@Configuration` 类，请考虑自动装配配置类本身。下面的示例说明如何执行此操作：

```java
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

In the preceding situation, where `AccountRepository` is defined is completely explicit. However, `ServiceConfig` is now tightly coupled to `RepositoryConfig`. That is the tradeoff. This tight coupling can be somewhat mitigated by using interface-based or abstract class-based `@Configuration` classes. Consider the following example:
在前面的情况下，其中 `AccountRepository` 被定义是完全明确的。然而， `ServiceConfig` 现在与 `RepositoryConfig` 紧密耦合。这就是权衡。这种紧密耦合可以通过使用基于接口或基于抽象类的 `@Configuration` 类来缓解。考虑以下示例：

```java
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
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
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

Now `ServiceConfig` is loosely coupled with respect to the concrete `DefaultRepositoryConfig`, and built-in IDE tooling is still useful: You can easily get a type hierarchy of `RepositoryConfig` implementations. In this way, navigating `@Configuration` classes and their dependencies becomes no different than the usual process of navigating interface-based code.
现在 `ServiceConfig` 相对于具体的 `DefaultRepositoryConfig` 是松散耦合的，内置的IDE工具仍然有用：你可以很容易地得到 `RepositoryConfig` 实现的类型层次结构。通过这种方式，导航 `@Configuration` 类及其依赖项与导航基于接口的代码的常规过程没有什么不同。

> If you want to influence the startup creation order of certain beans, consider declaring some of them as `@Lazy` (for creation on first access instead of on startup) or as `@DependsOn` certain other beans (making sure that specific other beans are created before the current bean, beyond what the latter’s direct dependencies imply).
> 如果要影响某些Bean的启动创建顺序，请考虑将其中一些Bean声明为 `@Lazy` （用于在第一次访问时创建，而不是在启动时创建）或 `@DependsOn` 某些其他Bean（确保在当前Bean之前创建特定的其他Bean，超出了后者的直接依赖关系所暗示的范围）。

##### 有条件地包含 `@Configuration` 类或 `@Bean` 方法

It is often useful to conditionally enable or disable a complete `@Configuration` class or even individual `@Bean` methods, based on some arbitrary system state. One common example of this is to use the `@Profile` annotation to activate beans only when a specific profile has been enabled in the Spring `Environment` (see [Bean Definition Profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles) for details).
根据某些任意的系统状态，有条件地启用或禁用一个完整的 `@Configuration` 类，甚至是单个的 `@Bean` 方法，这通常很有用。一个常见的例子是使用 `@Profile` 注解仅在Spring `Environment` 中启用了特定的配置文件时才激活bean（有关详细信息，请参阅 [Bean Definition Profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles) ）。

The `@Profile` annotation is actually implemented by using a much more flexible annotation called [`@Conditional`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/Conditional.html). The `@Conditional` annotation indicates specific `org.springframework.context.annotation.Condition` implementations that should be consulted before a `@Bean` is registered.
`@Profile` 注释实际上是通过使用一个更灵活的注释 `@Conditional` 来实现的。 `@Conditional` 注释指出了在注册 `@Bean` 之前应该咨询的特定 `org.springframework.context.annotation.Condition` 实现。

Implementations of the `Condition` interface provide a `matches(…)` method that returns `true` or `false`. For example, the following listing shows the actual `Condition` implementation used for `@Profile`:
`Condition` 接口的实现提供了一个返回 `true` 或 `false` 的 `matches(…​)` 方法。例如，下面的清单显示了用于 `@Profile` 的实际 `Condition` 实现：

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
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
    return true;
}
```

See the [`@Conditional`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/Conditional.html) javadoc for more detail.
请参阅  [`@Conditional`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/Conditional.html) javadoc了解更多细节。

##### 结合Java和XML配置

Spring’s `@Configuration` class support does not aim to be a 100% complete replacement for Spring XML. Some facilities, such as Spring XML namespaces, remain an ideal way to configure the container. In cases where XML is convenient or necessary, you have a choice: either instantiate the container in an “XML-centric” way by using, for example, `ClassPathXmlApplicationContext`, or instantiate it in a “Java-centric” way by using `AnnotationConfigApplicationContext` and the `@ImportResource` annotation to import XML as needed.
Spring的 `@Configuration` 类支持并不打算100%完全替代Spring XML。一些工具，比如SpringXML名称空间，仍然是配置容器的理想方法。在XML方便或必要的情况下，您可以选择：或者通过使用例如 `ClassPathXmlApplicationContext` 以“以XML为中心”的方式实例化容器，或者通过使用 `AnnotationConfigApplicationContext` 和 `@ImportResource` 注释以“以Java为中心”的方式实例化容器以根据需要导入XML。

###### 以XML为中心使用 `@Configuration` 类

It may be preferable to bootstrap the Spring container from XML and include `@Configuration` classes in an ad-hoc fashion. For example, in a large existing codebase that uses Spring XML, it is easier to create `@Configuration` classes on an as-needed basis and include them from the existing XML files. Later in this section, we cover the options for using `@Configuration` classes in this kind of “XML-centric” situation.
最好从XML引导Spring容器，并以ad-hoc方式包含 `@Configuration` 类。例如，在使用Spring XML的大型现有代码库中，更容易根据需要创建 `@Configuration` 类，并从现有XML文件中包含它们。在本节的后面，我们将介绍在这种“以XML为中心”的情况下使用 `@Configuration` 类的选项。

Declaring `@Configuration` classes as plain Spring `<bean/>` elements
将 `@Configuration` 类声明为普通Spring `<bean/>` 元素

Remember that `@Configuration` classes are ultimately bean definitions in the container. In this series examples, we create a `@Configuration` class named `AppConfig` and include it within `system-test-config.xml` as a `<bean/>` definition. Because `<context:annotation-config/>` is switched on, the container recognizes the `@Configuration` annotation and processes the `@Bean` methods declared in `AppConfig` properly.
请记住， `@Configuration` 类最终是容器中的bean定义。在本系列示例中，我们创建了一个名为 `AppConfig` 的 `@Configuration` 类，并将其作为 `<bean/>` 定义包含在 `system-test-config.xml` 中。因为 `<context:annotation-config/>` 是打开的，所以容器识别 `@Configuration` 注释并正确处理 `AppConfig` 中声明的 `@Bean` 方法。

```java
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

The following example shows part of a sample `system-test-config.xml` file:
以下示例显示了示例 `system-test-config.xml` 文件的一部分：

```xml
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

The following example shows a possible `jdbc.properties` file:
以下示例显示了可能的 `jdbc.properties` 文件：

```properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

> In `system-test-config.xml` file, the `AppConfig` `<bean/>` does not declare an `id` element. While it would be acceptable to do so, it is unnecessary, given that no other bean ever refers to it, and it is unlikely to be explicitly fetched from the container by name. Similarly, the `DataSource` bean is only ever autowired by type, so an explicit bean `id` is not strictly required.
> 在 `system-test-config.xml` 文件中， `AppConfig` `<bean/>` 未声明 `id` 元素。虽然这样做是可以接受的，但这是不必要的，因为没有其他bean引用它，并且不太可能通过名称显式地从容器中获取它。类似地， `DataSource` bean只会根据类型自动连接，因此并不严格要求显式的bean `id` 。

Using <context:component-scan/> to pick up `@Configuration` classes
用<context:component-scan/>接 `@Configuration` 班

Because `@Configuration` is meta-annotated with `@Component`, `@Configuration`-annotated classes are automatically candidates for component scanning. Using the same scenario as described in the previous example, we can redefine `system-test-config.xml` to take advantage of component-scanning. Note that, in this case, we need not explicitly declare `<context:annotation-config/>`, because `<context:component-scan/>` enables the same functionality.
因为 `@Configuration` 是用 `@Component` 进行元注释的，所以 `@Configuration` 注释的类自动成为组件扫描的候选。使用与前面示例中描述的相同的场景，我们可以重新定义 `system-test-config.xml` 以利用组件扫描。注意，在这种情况下，我们不需要显式声明 `<context:annotation-config/>` ，因为 `<context:component-scan/>` 启用了相同的功能。

The following example shows the modified `system-test-config.xml` file:
以下示例显示了修改后的 `system-test-config.xml` 文件：

```xml
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

######  `@Configuration` 使用XML以类为中心 `@ImportResource`

In applications where `@Configuration` classes are the primary mechanism for configuring the container, it is still likely necessary to use at least some XML. In these scenarios, you can use `@ImportResource` and define only as much XML as you need. Doing so achieves a “Java-centric” approach to configuring the container and keeps XML to a bare minimum. The following example (which includes a configuration class, an XML file that defines a bean, a properties file, and the `main` class) shows how to use the `@ImportResource` annotation to achieve “Java-centric” configuration that uses XML as needed:
在应用程序中，如果 `@Configuration` 类是配置容器的主要机制，则可能仍然需要使用至少一些XML。在这些场景中，您可以使用 `@ImportResource` ，并且只定义所需的XML。这样做可以实现一种“以Java为中心”的方法来配置容器，并将XML保持在最低限度。下面的示例（包括一个配置类、一个定义bean的XML文件、一个属性文件和 `main` 类）展示了如何使用 `@ImportResource` 注释来实现根据需要使用XML的“以Java为中心”的配置：

```java
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

```xml
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

```properties
jdbc.properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

### 1.13.环境抽象

The [`Environment`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/env/Environment.html) interface is an abstraction integrated in the container that models two key aspects of the application environment: [profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles) and [properties](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction).
`Environment` 接口是集成在容器中的抽象，它对应用程序环境的两个关键方面进行建模：配置文件和属性。

A profile is a named, logical group of bean definitions to be registered with the container only if the given profile is active. Beans may be assigned to a profile whether defined in XML or with annotations. The role of the `Environment` object with relation to profiles is in determining which profiles (if any) are currently active, and which profiles (if any) should be active by default.
概要文件是一个命名的、逻辑的bean定义组，只有当给定的概要文件处于活动状态时，它才会注册到容器中。可以将Bean分配给一个配置文件，无论是用XML定义的还是用注释定义的。与配置相关的 `Environment` 对象的作用是确定哪些配置（如果有的话）当前是活动的，以及哪些配置（如果有的话）在默认情况下应该是活动的。

Properties play an important role in almost all applications and may originate from a variety of sources: properties files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc `Properties` objects, `Map` objects, and so on. The role of the `Environment` object with relation to properties is to provide the user with a convenient service interface for configuring property sources and resolving properties from them.
属性在几乎所有的应用中都起着重要的作用，并且可能来自各种来源：properties文件、JVM系统属性、系统环境变量、JNDI、servlet上下文参数、ad-hoc `Properties` 对象、 `Map` 对象等，与属性相关的 `Environment` 对象的作用是为用户提供一个方便的服务接口，用于配置属性源并从中解析属性。

#### 1.13.1. Bean定义配置文件

Bean definition profiles provide a mechanism in the core container that allows for registration of different beans in different environments. The word, “environment,” can mean different things to different users, and this feature can help with many use cases, including:
Bean定义概要文件在核心容器中提供了一种机制，允许在不同的环境中注册不同的Bean。“环境”这个词对不同的用户来说可能意味着不同的东西，这个功能可以帮助许多用例，包括：

- Working against an in-memory datasource in development versus looking up that same datasource from JNDI when in QA or production.
  在开发中使用内存中的数据源，而在QA或生产中从JNDI查找相同的数据源。
- Registering monitoring infrastructure only when deploying an application into a performance environment.
  仅在将应用程序部署到性能环境中时注册监视基础结构。
- Registering customized implementations of beans for customer A versus customer B deployments.
  为客户A和客户B的部署注册bean的自定义实现。

Consider the first use case in a practical application that requires a `DataSource`. In a test environment, the configuration might resemble the following:
考虑实际应用中需要 `DataSource` 的第一个用例。在测试环境中，配置可能类似于以下内容：

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

Now consider how this application can be deployed into a QA or production environment, assuming that the datasource for the application is registered with the production application server’s JNDI directory. Our `dataSource` bean now looks like the following listing:
现在考虑如何将此应用程序部署到QA或生产环境中，假设应用程序的数据源已注册到生产应用程序服务器的JNDI目录中。我们的 `dataSource` bean现在看起来像下面的清单：

```java
@Bean(destroyMethod = "")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

The problem is how to switch between using these two variations based on the current environment. Over time, Spring users have devised a number of ways to get this done, usually relying on a combination of system environment variables and XML `<import/>` statements containing `${placeholder}` tokens that resolve to the correct configuration file path depending on the value of an environment variable. Bean definition profiles is a core container feature that provides a solution to this problem.
问题是如何根据当前环境在使用这两种变体之间切换。随着时间的推移，Spring用户已经设计了许多方法来完成此任务，通常依赖于系统环境变量和包含 `${placeholder}` 令牌的XML `<import/>` 语句的组合，这些令牌根据环境变量的值解析到正确的配置文件路径。Bean定义概要文件是一个核心容器特性，它为这个问题提供了一个解决方案。

If we generalize the use case shown in the preceding example of environment-specific bean definitions, we end up with the need to register certain bean definitions in certain contexts but not in others. You could say that you want to register a certain profile of bean definitions in situation A and a different profile in situation B. We start by updating our configuration to reflect this need.
如果我们概括前面特定于环境的bean定义的示例中所示的用例，我们最终需要在某些上下文中注册某些bean定义，而不是在其他上下文中注册。您可以说，您希望在情况A中注册bean定义的某个概要文件，而在情况B中注册另一个概要文件。我们首先更新我们的配置以反映这种需要。

##### 使用 `@Profile`

The [`@Profile`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/Profile.html) annotation lets you indicate that a component is eligible for registration when one or more specified profiles are active. Using our preceding example, we can rewrite the `dataSource` configuration as follows:
当一个或多个指定的配置文件处于活动状态时，[`@Profile`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/Profile.html)注释允许您指示组件符合注册条件。使用我们前面的例子，我们可以重写 `dataSource` 配置如下：

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

```java
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod = "") (1)
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

1. `@Bean(destroyMethod = "")` disables default destroy method inference.
   `@Bean(destroyMethod = "")` 禁用默认销毁方法推断。

> As mentioned earlier, with `@Bean` methods, you typically choose to use programmatic JNDI lookups, by using either Spring’s `JndiTemplate`/`JndiLocatorDelegate` helpers or the straight JNDI `InitialContext` usage shown earlier but not the `JndiObjectFactoryBean` variant, which would force you to declare the return type as the `FactoryBean` type.
> 如前所述，对于 `@Bean` 方法，您通常选择使用编程JNDI查找，通过使用Spring的 `JndiTemplate` / `JndiLocatorDelegate` helpers或前面显示的直接JNDI `InitialContext` 用法，而不是 `JndiObjectFactoryBean` 变体，这将迫使您将返回类型声明为 `FactoryBean` 类型。

The profile string may contain a simple profile name (for example, `production`) or a profile expression. A profile expression allows for more complicated profile logic to be expressed (for example, `production & us-east`). The following operators are supported in profile expressions:
配置文件字符串可以包含简单的配置文件名称（例如 `production` ）或配置文件表达式。配置文件表达式允许表达更复杂的配置文件逻辑（例如 `production & us-east` ）。配置文件表达式中支持以下运算符：

- `!`: A logical `NOT` of the profile
  `!` ：配置文件的逻辑 `NOT`
- `&`: A logical `AND` of the profiles
  `&` ：配置文件的逻辑 `AND`
- `|`: A logical `OR` of the profiles
  `|` ：配置文件的逻辑 `OR`

> You cannot mix the `&` and `|` operators without using parentheses. For example, `production & us-east | eu-central` is not a valid expression. It must be expressed as `production & (us-east | eu-central)`.
> 如果不使用括号，则不能混合使用 `&` 和 `|` 运算符。例如， `production & us-east | eu-central` 不是有效的表达式。它必须表示为 `production & (us-east | eu-central)` 。

You can use `@Profile` as a [meta-annotation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations) for the purpose of creating a custom composed annotation. The following example defines a custom `@Production` annotation that you can use as a drop-in replacement for `@Profile("production")`:
您可以使用 `@Profile` 作为[元注释](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations)来创建自定义组合注释。下面的示例定义了一个自定义的 `@Production` 注释，您可以将其用作 `@Profile("production")` 的直接替换：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

> If a `@Configuration` class is marked with `@Profile`, all of the `@Bean` methods and `@Import` annotations associated with that class are bypassed unless one or more of the specified profiles are active. If a `@Component` or `@Configuration` class is marked with `@Profile({"p1", "p2"})`, that class is not registered or processed unless profiles 'p1' or 'p2' have been activated. If a given profile is prefixed with the NOT operator (`!`), the annotated element is registered only if the profile is not active. For example, given `@Profile({"p1", "!p2"})`, registration will occur if profile 'p1' is active or if profile 'p2' is not active.
> 如果 `@Configuration` 类被标记为 `@Profile` ，则与该类相关联的所有 `@Bean` 方法和 `@Import` 注释都将被绕过，除非一个或多个指定的配置文件处于活动状态。如果 `@Component` 或 `@Configuration` 类被标记为 `@Profile({"p1", "p2"})` ，则除非配置文件'p1'或'p2'已被激活，否则不注册或处理该类。如果给定的配置文件以NOT操作符（ `!` ）为前缀，则仅当配置文件不活动时才注册带注释的元素。例如，给定 `@Profile({"p1", "!p2"})` ，如果简档“p1”是活动的或者如果简档“p2”不是活动的，则将发生注册。

`@Profile` can also be declared at the method level to include only one particular bean of a configuration class (for example, for alternative variants of a particular bean), as the following example shows:
`@Profile` 也可以在方法级别声明，以仅包含配置类的一个特定bean（例如，用于特定bean的替代变体），如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") (1)
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") (2)
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

1. The `standaloneDataSource` method is available only in the `development` profile.
   `standaloneDataSource` 方法仅在 `development` 配置文件中可用。
2.  The `jndiDataSource` method is available only in the `production` profile. `jndiDataSource` 方法仅在 `production` 配置文件中可用。

> With `@Profile` on `@Bean` methods, a special scenario may apply: In the case of overloaded `@Bean` methods of the same Java method name (analogous to constructor overloading), a `@Profile` condition needs to be consistently declared on all overloaded methods. If the conditions are inconsistent, only the condition on the first declaration among the overloaded methods matters. Therefore, `@Profile` can not be used to select an overloaded method with a particular argument signature over another. Resolution between all factory methods for the same bean follows Spring’s constructor resolution algorithm at creation time.
> 对于 `@Profile` on `@Bean` 方法，可以应用特殊场景：在重载相同Java方法名的 `@Bean` 方法的情况下（类似于构造函数重载），需要在所有重载方法上一致地声明 `@Profile` 条件。如果条件不一致，则只有重载方法中第一个声明上的条件才重要。因此， `@Profile` 不能用于选择具有特定参数签名的重载方法而不是其他方法。同一bean的所有工厂方法之间的解析在创建时遵循Spring的构造函数解析算法。
>
> If you want to define alternative beans with different profile conditions, use distinct Java method names that point to the same bean name by using the `@Bean` name attribute, as shown in the preceding example. If the argument signatures are all the same (for example, all of the variants have no-arg factory methods), this is the only way to represent such an arrangement in a valid Java class in the first place (since there can only be one method of a particular name and argument signature).
> 如果您想定义具有不同配置文件条件的替代bean，请使用不同的Java方法名称，通过使用 `@Bean` name属性来指向相同的bean名称，如前面的示例所示。如果参数签名都是相同的（例如，所有的变量都有无参数的工厂方法），这是在一个有效的Java类中首先表示这种安排的唯一方法（因为只能有一个特定名称和参数签名的方法）。

##### XML Bean定义配置文件

The XML counterpart is the `profile` attribute of the `<beans>` element. Our preceding sample configuration can be rewritten in two XML files, as follows:
XML对应项是 `<beans>` 元素的 `profile` 属性。我们前面的示例配置可以在两个XML文件中重写，如下所示：

```xml
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

```xml
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

It is also possible to avoid that split and nest `<beans/>` elements within the same file, as the following example shows:
也可以避免在同一个文件中拆分和嵌套 `<beans/>` 元素，如以下示例所示：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

The `spring-bean.xsd` has been constrained to allow such elements only as the last ones in the file. This should help provide flexibility without incurring clutter in the XML files. 

`spring-bean.xsd` 已被限制为仅允许此类元素作为文件中的最后一个元素。这应该有助于提供灵活性，而不会导致XML文件混乱。

> The XML counterpart does not support the profile expressions described earlier. It is possible, however, to negate a profile by using the `!` operator. It is also possible to apply a logical “and” by nesting the profiles, as the following example shows: 
>
> XML对应部分不支持前面描述的配置文件表达式。但是，可以通过使用 `!` 运算符来否定配置文件。也可以通过嵌套配置文件来应用逻辑“and”，如以下示例所示：
>
> ```xml
> <beans xmlns="http://www.springframework.org/schema/beans"
>     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>     xmlns:jdbc="http://www.springframework.org/schema/jdbc"
>     xmlns:jee="http://www.springframework.org/schema/jee"
>     xsi:schemaLocation="...">
> 
>     <!-- other bean definitions -->
> 
>     <beans profile="production">
>         <beans profile="us-east">
>             <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
>         </beans>
>     </beans>
> </beans>
> ```
>
> In the preceding example, the `dataSource` bean is exposed if both the `production` and `us-east` profiles are active.
> 在前面的示例中，如果 `production` 和 `us-east` 配置文件都是活动的，则暴露 `dataSource` bean。

##### 激活配置文件

Now that we have updated our configuration, we still need to instruct Spring which profile is active. If we started our sample application right now, we would see a `NoSuchBeanDefinitionException` thrown, because the container could not find the Spring bean named `dataSource`.
现在我们已经更新了配置，我们仍然需要指示Spring哪个配置文件是活动的。如果我们现在启动我们的示例应用程序，我们会看到一个 `NoSuchBeanDefinitionException` 抛出，因为容器找不到名为 `dataSource` 的Spring bean。

Activating a profile can be done in several ways, but the most straightforward is to do it programmatically against the `Environment` API which is available through an `ApplicationContext`. The following example shows how to do so:
激活配置文件可以通过多种方式完成，但最直接的方式是通过 `ApplicationContext` 以编程方式对 `Environment` API进行激活。下面的示例说明如何执行此操作：

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

In addition, you can also declaratively activate profiles through the `spring.profiles.active` property, which may be specified through system environment variables, JVM system properties, servlet context parameters in `web.xml`, or even as an entry in JNDI (see [`PropertySource` Abstraction](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction)). In integration tests, active profiles can be declared by using the `@ActiveProfiles` annotation in the `spring-test` module (see [context configuration with environment profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-env-profiles)).
此外，您还可以通过 `spring.profiles.active` 属性以声明方式激活配置文件，该属性可以通过系统环境变量、JVM系统属性、 `web.xml` 中的servlet上下文参数来指定，甚至可以作为JNDI中的一个条目（参见 `PropertySource` 抽象）。在集成测试中，可以使用 `spring-test` 模块中的 `@ActiveProfiles` 注释来声明活动概要文件（参见环境概要文件的上下文配置）。

Note that profiles are not an “either-or” proposition. You can activate multiple profiles at once. Programmatically, you can provide multiple profile names to the `setActiveProfiles()` method, which accepts `String…` varargs. The following example activates multiple profiles:
请注意，配置文件不是“非此即彼”的命题。您可以一次激活多个配置文件。通过编程，您可以为 `setActiveProfiles()` 方法提供多个配置文件名称，该方法接受 `String…​` 参数。以下示例激活多个配置文件：

```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

Declaratively, `spring.profiles.active` may accept a comma-separated list of profile names, as the following example shows:
声明性地， `spring.profiles.active` 可以接受以逗号分隔的配置文件名称列表，如以下示例所示：

```properties
-Dspring.profiles.active="profile1,profile2"
```

##### 默认配置

The default profile represents the profile that is enabled by default. Consider the following example:
默认配置文件表示默认启用的配置文件。考虑以下示例：

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

If no profile is active, the `dataSource` is created. You can see this as a way to provide a default definition for one or more beans. If any profile is enabled, the default profile does not apply.
如果没有活动的配置文件，则创建 `dataSource` 。您可以将此视为为一个或多个bean提供默认定义的一种方式。如果启用了任何配置文件，则不应用默认配置文件。

You can change the name of the default profile by using `setDefaultProfiles()` on the `Environment` or, declaratively, by using the `spring.profiles.default` property.
您可以通过在 `Environment` 上使用 `setDefaultProfiles()` 来更改默认配置文件的名称，或者以声明方式使用 `spring.profiles.default` 属性。

#### 1.13.2. `PropertySource` 抽象

Spring’s `Environment` abstraction provides search operations over a configurable hierarchy of property sources. Consider the following listing:
Spring的 `Environment` 抽象在属性源的可配置层次结构上提供搜索操作。考虑以下列表：

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

In the preceding snippet, we see a high-level way of asking Spring whether the `my-property` property is defined for the current environment. To answer this question, the `Environment` object performs a search over a set of [`PropertySource`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/env/PropertySource.html) objects. A `PropertySource` is a simple abstraction over any source of key-value pairs, and Spring’s [`StandardEnvironment`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/env/StandardEnvironment.html) is configured with two PropertySource objects — one representing the set of JVM system properties (`System.getProperties()`) and one representing the set of system environment variables (`System.getenv()`).
在前面的代码片段中，我们看到了一种高级的方式来询问Spring是否为当前环境定义了 `my-property` 属性。为了回答这个问题， `Environment` 对象在一组 `PropertySource` 对象上执行搜索。 `PropertySource` 是对任何键值对源的简单抽象，Spring的 `StandardEnvironment` 配置了两个PropertySource对象--一个表示JVM系统属性集（ `System.getProperties()` ），一个表示系统环境变量集（ `System.getenv()` ）。

> These default property sources are present for `StandardEnvironment`, for use in standalone applications. [`StandardServletEnvironment`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/web/context/support/StandardServletEnvironment.html) is populated with additional default property sources including servlet config, servlet context parameters, and a [`JndiPropertySource`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/jndi/JndiPropertySource.html) if JNDI is available.
> 这些默认属性源存在于 `StandardEnvironment` 中，用于独立应用程序。 `StandardServletEnvironment` 填充了额外的默认属性源，包括servlet配置、servlet上下文参数和 `JndiPropertySource` （如果JNDI可用）。

Concretely, when you use the `StandardEnvironment`, the call to `env.containsProperty("my-property")` returns true if a `my-property` system property or `my-property` environment variable is present at runtime.
具体地说，当你使用 `StandardEnvironment` 时，如果运行时存在 `my-property` 系统属性或 `my-property` 环境变量，那么对 `env.containsProperty("my-property")` 的调用将返回true。

> The search performed is hierarchical. By default, system properties have precedence over environment variables. So, if the `my-property` property happens to be set in both places during a call to `env.getProperty("my-property")`, the system property value “wins” and is returned. Note that property values are not merged but rather completely overridden by a preceding entry.
> 执行的搜索是分层的。默认情况下，系统属性优先于环境变量。因此，如果在调用 `env.getProperty("my-property")` 期间，在两个位置都设置了 `my-property` 属性，则系统属性值“wins”并返回。请注意，属性值不会合并，而是完全被前面的条目覆盖。
>
> For a common `StandardServletEnvironment`, the full hierarchy is as follows, with the highest-precedence entries at the top:
> 对于常见的 `StandardServletEnvironment` ，完整的层次结构如下所示，最高优先级的条目位于顶部：
>
> 1. ServletConfig parameters (if applicable — for example, in case of a `DispatcherServlet` context)
>    ServletConfig参数（如果适用-例如，在 `DispatcherServlet` 上下文的情况下）
> 2. ServletContext parameters (web.xml context-param entries)
>    ServletContext参数（web.xml context-param条目）
> 3. JNDI environment variables (`java:comp/env/` entries)
>    JNDI环境变量（ `java:comp/env/` 个条目）
> 4. JVM system properties (`-D` command-line arguments)
>    JVM系统属性（ `-D` 命令行参数）
> 5. JVM system environment (operating system environment variables)
>    JVM系统环境（操作系统环境变量）

Most importantly, the entire mechanism is configurable. Perhaps you have a custom source of properties that you want to integrate into this search. To do so, implement and instantiate your own `PropertySource` and add it to the set of `PropertySources` for the current `Environment`. The following example shows how to do so:
最重要的是，整个机制是可配置的。也许您有一个自定义的属性源，希望将其集成到此搜索中。为此，实现并实例化您自己的 `PropertySource` ，并将其添加到当前 `Environment` 的 `PropertySources` 集合中。下面的示例说明如何执行此操作：

```java
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

In the preceding code, `MyPropertySource` has been added with highest precedence in the search. If it contains a `my-property` property, the property is detected and returned, in favor of any `my-property` property in any other `PropertySource`. The [`MutablePropertySources`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/core/env/MutablePropertySources.html) API exposes a number of methods that allow for precise manipulation of the set of property sources.
在前面的代码中， `MyPropertySource` 已被添加为搜索中的最高优先级。如果它包含 `my-property` 属性，则检测并返回该属性，以支持任何其他 `PropertySource` 中的任何 `my-property` 属性。 `MutablePropertySources` API公开了许多方法，这些方法允许对属性源集进行精确操作。

#### 1.13.3.使用 `@PropertySource`

The [`@PropertySource`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/annotation/PropertySource.html) annotation provides a convenient and declarative mechanism for adding a `PropertySource` to Spring’s `Environment`.
`@PropertySource` 注释提供了一个方便的声明性机制，用于将 `PropertySource` 添加到Spring的 `Environment` 中。

Given a file called `app.properties` that contains the key-value pair `testbean.name=myTestBean`, the following `@Configuration` class uses `@PropertySource` in such a way that a call to `testBean.getName()` returns `myTestBean`:
给定一个名为 `app.properties` 的文件，其中包含键值对 `testbean.name=myTestBean` ，下面的 `@Configuration` 类使用 `@PropertySource` 的方式是调用 `testBean.getName()` 返回 `myTestBean` ：

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

Any `${…}` placeholders present in a `@PropertySource` resource location are resolved against the set of property sources already registered against the environment, as the following example shows:
存在于 `@PropertySource` 资源位置中的任何 `${…}` 占位符都将根据已在环境中注册的属性源集进行解析，如下例所示：

```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

Assuming that `my.placeholder` is present in one of the property sources already registered (for example, system properties or environment variables), the placeholder is resolved to the corresponding value. If not, then `default/path` is used as a default. If no default is specified and a property cannot be resolved, an `IllegalArgumentException` is thrown.
假设 `my.placeholder` 存在于已注册的属性源之一（例如，系统属性或环境变量）中，占位符将解析为相应的值。如果不是，则使用 `default/path` 作为默认值。如果没有指定默认值，并且无法解析属性，则会抛出 `IllegalArgumentException` 。

> The `@PropertySource` annotation is repeatable, according to Java 8 conventions. However, all such `@PropertySource` annotations need to be declared at the same level, either directly on the configuration class or as meta-annotations within the same custom annotation. Mixing direct annotations and meta-annotations is not recommended, since direct annotations effectively override meta-annotations.
> 根据Java 8约定， `@PropertySource` 注释是可重复的。然而，所有这样的 `@PropertySource` 注释都需要在同一级别上声明，要么直接在配置类上声明，要么作为同一自定义注释中的元注释声明。不推荐混合使用直接注释和元注释，因为直接注释有效地覆盖了元注释。

#### 1.13.4.语句中的占位符解析

Historically, the value of placeholders in elements could be resolved only against JVM system properties or environment variables. This is no longer the case. Because the `Environment` abstraction is integrated throughout the container, it is easy to route resolution of placeholders through it. This means that you may configure the resolution process in any way you like. You can change the precedence of searching through system properties and environment variables or remove them entirely. You can also add your own property sources to the mix, as appropriate.
从历史上看，元素中占位符的值只能根据JVM系统属性或环境变量来解析。现在情况已不再如此。因为 `Environment` 抽象被集成在整个容器中，所以很容易通过它路由占位符的解析。这意味着你可以以任何你喜欢的方式配置解析过程。您可以更改搜索系统属性和环境变量的优先级，也可以将其完全删除。您还可以根据需要将自己的属性源添加到组合中。

Concretely, the following statement works regardless of where the `customer` property is defined, as long as it is available in the `Environment`:
具体地说，以下语句无论 `customer` 属性在哪里定义都有效，只要它在 `Environment` 中可用：

```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

### 1.14.注册 `LoadTimeWeaver`

The `LoadTimeWeaver` is used by Spring to dynamically transform classes as they are loaded into the Java virtual machine (JVM).
Spring使用 `LoadTimeWeaver` 在类加载到Java虚拟机（JVM）时动态转换类。

To enable load-time weaving, you can add the `@EnableLoadTimeWeaving` to one of your `@Configuration` classes, as the following example shows:
要启用加载时织入，可以将 `@EnableLoadTimeWeaving` 添加到 `@Configuration` 类之一，如下例所示：

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

Alternatively, for XML configuration, you can use the `context:load-time-weaver` element:
或者，对于XML配置，您可以使用 `context:load-time-weaver` 元素：

```xml
<beans>
    <context:load-time-weaver/>
</beans>
```

Once configured for the `ApplicationContext`, any bean within that `ApplicationContext` may implement `LoadTimeWeaverAware`, thereby receiving a reference to the load-time weaver instance. This is particularly useful in combination with [Spring’s JPA support](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#orm-jpa) where load-time weaving may be necessary for JPA class transformation. Consult the [`LocalContainerEntityManagerFactoryBean`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/orm/jpa/LocalContainerEntityManagerFactoryBean.html) javadoc for more detail. For more on AspectJ load-time weaving, see [Load-time Weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw).
一旦配置了 `ApplicationContext` ， `ApplicationContext` 中的任何bean都可以实现 `LoadTimeWeaverAware` ，从而接收对加载时weaver实例的引用。这在与[Spring的JPA支持](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#orm-jpa)结合使用时特别有用，因为JPA类转换可能需要加载时编织。更多细节请参考  [`LocalContainerEntityManagerFactoryBean`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/orm/jpa/LocalContainerEntityManagerFactoryBean.html)javadoc。有关AspectJ加载时编织的更多信息，请参见[Spring框架中的AspectJ加载时编织](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw)。

### 1.15. `ApplicationContext` 的附加功能

As discussed in the [chapter introduction](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans), the `org.springframework.beans.factory` package provides basic functionality for managing and manipulating beans, including in a programmatic way. The `org.springframework.context` package adds the [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/ApplicationContext.html) interface, which extends the `BeanFactory` interface, in addition to extending other interfaces to provide additional functionality in a more application framework-oriented style. Many people use the `ApplicationContext` in a completely declarative fashion, not even creating it programmatically, but instead relying on support classes such as `ContextLoader` to automatically instantiate an `ApplicationContext` as part of the normal startup process of a Jakarta EE web application.
正如本章介绍中所讨论的， `org.springframework.beans.factory` 包提供了管理和操作bean的基本功能，包括以编程方式。 `org.springframework.context` 包添加了 `ApplicationContext` 接口，该接口扩展了 `BeanFactory` 接口，此外还扩展了其他接口，以更加面向应用程序框架的风格提供额外的功能。许多人以完全声明的方式使用 `ApplicationContext` ，甚至不以编程方式创建它，而是依靠支持类（如 `ContextLoader` ）自动实例化 `ApplicationContext` ，作为Jakarta EE Web应用程序正常启动过程的一部分。

To enhance `BeanFactory` functionality in a more framework-oriented style, the context package also provides the following functionality:
为了以更加面向框架的风格增强 `BeanFactory` 功能，上下文包还提供了以下功能：

- Access to messages in i18n-style, through the `MessageSource` interface.
  通过 `MessageSource` 接口访问i18n风格的消息。
- Access to resources, such as URLs and files, through the `ResourceLoader` interface.
  通过 `ResourceLoader` 接口访问资源，如URL和文件。
- Event publication, namely to beans that implement the `ApplicationListener` interface, through the use of the `ApplicationEventPublisher` interface.
  事件发布，即通过使用 `ApplicationEventPublisher` 接口发布到实现 `ApplicationListener` 接口的bean。
- Loading of multiple (hierarchical) contexts, letting each be focused on one particular layer, such as the web layer of an application, through the `HierarchicalBeanFactory` interface.
  通过 `HierarchicalBeanFactory` 接口加载多个（分层）上下文，让每个上下文集中在一个特定的层，例如应用程序的Web层。

#### 1.15.1.使用 `MessageSource` 进行国际化

The `ApplicationContext` interface extends an interface called `MessageSource` and, therefore, provides internationalization (“i18n”) functionality. Spring also provides the `HierarchicalMessageSource` interface, which can resolve messages hierarchically. Together, these interfaces provide the foundation upon which Spring effects message resolution. The methods defined on these interfaces include:
`ApplicationContext` 接口扩展了名为 `MessageSource` 的接口，因此提供了国际化（“i18n”）功能。Spring还提供了 `HierarchicalMessageSource` 接口，可以分层解析消息。这些接口一起提供了Spring实现消息解析的基础。在这些接口上定义的方法包括：

- `String getMessage(String code, Object[] args, String default, Locale loc)`: The basic method used to retrieve a message from the `MessageSource`. When no message is found for the specified locale, the default message is used. Any arguments passed in become replacement values, using the `MessageFormat` functionality provided by the standard library.
  `String getMessage(String code, Object[] args, String default, Locale loc)` ：用于从 `MessageSource` 检索消息的基本方法。如果找不到指定区域设置的消息，则使用默认消息。使用标准库提供的 `MessageFormat` 功能，传入的任何参数都将成为替换值。
- `String getMessage(String code, Object[] args, Locale loc)`: Essentially the same as the previous method but with one difference: No default message can be specified. If the message cannot be found, a `NoSuchMessageException` is thrown.
  `String getMessage(String code, Object[] args, Locale loc)` ：基本上与之前的方法相同，但有一点不同：无法指定默认消息。如果找不到消息，则抛出 `NoSuchMessageException` 。
- `String getMessage(MessageSourceResolvable resolvable, Locale locale)`: All properties used in the preceding methods are also wrapped in a class named `MessageSourceResolvable`, which you can use with this method.
  `String getMessage(MessageSourceResolvable resolvable, Locale locale)` ：前面的方法中使用的所有属性也被包装在一个名为 `MessageSourceResolvable` 的类中，您可以在此方法中使用该类。

When an `ApplicationContext` is loaded, it automatically searches for a `MessageSource` bean defined in the context. The bean must have the name `messageSource`. If such a bean is found, all calls to the preceding methods are delegated to the message source. If no message source is found, the `ApplicationContext` attempts to find a parent containing a bean with the same name. If it does, it uses that bean as the `MessageSource`. If the `ApplicationContext` cannot find any source for messages, an empty `DelegatingMessageSource` is instantiated in order to be able to accept calls to the methods defined above.
当加载 `ApplicationContext` 时，它会自动搜索上下文中定义的 `MessageSource` bean。Bean的名称必须为 `messageSource` 。如果找到这样的bean，则将对前面方法的所有调用委托给消息源。如果没有找到消息源， `ApplicationContext` 会尝试查找包含同名bean的父节点。如果是，则使用该bean作为 `MessageSource` 。如果 `ApplicationContext` 找不到任何消息源，则实例化空的 `DelegatingMessageSource` ，以便能够接受对上面定义的方法的调用。

Spring provides three `MessageSource` implementations, `ResourceBundleMessageSource`, `ReloadableResourceBundleMessageSource` and `StaticMessageSource`. All of them implement `HierarchicalMessageSource` in order to do nested messaging. The `StaticMessageSource` is rarely used but provides programmatic ways to add messages to the source. The following example shows `ResourceBundleMessageSource`:
Spring提供了三个 `MessageSource` 实现： `ResourceBundleMessageSource` 、 `ReloadableResourceBundleMessageSource` 和 `StaticMessageSource` 。它们都实现了 `HierarchicalMessageSource` ，以便进行嵌套消息传递。 `StaticMessageSource` 很少使用，但提供了将消息添加到源的编程方法。以下示例显示了 `ResourceBundleMessageSource` ：

```xml
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

The example assumes that you have three resource bundles called `format`, `exceptions` and `windows` defined in your classpath. Any request to resolve a message is handled in the JDK-standard way of resolving messages through `ResourceBundle` objects. For the purposes of the example, assume the contents of two of the above resource bundle files are as follows:
这个例子假设在类路径中定义了三个资源包，分别是 `format` 、 `exceptions` 和 `windows` 。任何解析消息的请求都以JDK标准的方式处理，即通过 `ResourceBundle` 对象解析消息。出于示例的目的，假设上述资源束文件中的两个的内容如下：

```properties
# in format.properties
message=Alligators rock!
```

```properties
# in exceptions.properties
argument.required=The {0} argument is required.
```

The next example shows a program to run the `MessageSource` functionality. Remember that all `ApplicationContext` implementations are also `MessageSource` implementations and so can be cast to the `MessageSource` interface.
下一个示例显示了运行 `MessageSource` 功能的程序。请记住，所有 `ApplicationContext` 实现也是 `MessageSource` 实现，因此可以转换为 `MessageSource` 接口。

```java
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
    System.out.println(message);
}
```

The resulting output from the above program is as follows:
上述程序的结果输出如下：

```
Alligators rock!
```

To summarize, the `MessageSource` is defined in a file called `beans.xml`, which exists at the root of your classpath. The `messageSource` bean definition refers to a number of resource bundles through its `basenames` property. The three files that are passed in the list to the `basenames` property exist as files at the root of your classpath and are called `format.properties`, `exceptions.properties`, and `windows.properties`, respectively.
总而言之， `MessageSource` 是在名为 `beans.xml` 的文件中定义的，该文件位于类路径的根目录。 `messageSource` bean定义通过其 `basenames` 属性引用了许多资源包。在列表中传递给 `basenames` 属性的三个文件作为类路径根目录中的文件存在，分别称为 `format.properties` 、 `exceptions.properties` 和 `windows.properties` 。

The next example shows arguments passed to the message lookup. These arguments are converted into `String` objects and inserted into placeholders in the lookup message.
下一个示例显示传递给消息查找的参数。这些参数被转换为 `String` 对象并插入到查找消息中的占位符中。

```xml
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.something.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```

```java
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", Locale.ENGLISH);
        System.out.println(message);
    }
}
```

The resulting output from the invocation of the `execute()` method is as follows:
调用 `execute()` 方法的结果输出如下：

```
The userDao argument is required.
```

With regard to internationalization (“i18n”), Spring’s various `MessageSource` implementations follow the same locale resolution and fallback rules as the standard JDK `ResourceBundle`. In short, and continuing with the example `messageSource` defined previously, if you want to resolve messages against the British (`en-GB`) locale, you would create files called `format_en_GB.properties`, `exceptions_en_GB.properties`, and `windows_en_GB.properties`, respectively.
关于国际化（“i18n”），Spring的各种 `MessageSource` 实现遵循与标准JDK `ResourceBundle` 相同的locale解析和回退规则。简而言之，继续前面定义的示例 `messageSource` ，如果您希望根据英国（ `en-GB` ）地区解析消息，则应分别创建名为 `format_en_GB.properties` 、 `exceptions_en_GB.properties` 和 `windows_en_GB.properties` 的文件。

Typically, locale resolution is managed by the surrounding environment of the application. In the following example, the locale against which (British) messages are resolved is specified manually:
通常，区域设置解析由应用程序的周围环境管理。在下面的示例中，手动指定了解析（英国）邮件所依据的区域设置：

```properties
# in exceptions_en_GB.properties
argument.required=Ebagum lad, the ''{0}'' argument is required, I say, required.
```

```java
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

The resulting output from the running of the above program is as follows:
运行上述程序的结果输出如下：

```
Ebagum lad, the 'userDao' argument is required, I say, required.
```

You can also use the `MessageSourceAware` interface to acquire a reference to any `MessageSource` that has been defined. Any bean that is defined in an `ApplicationContext` that implements the `MessageSourceAware` interface is injected with the application context’s `MessageSource` when the bean is created and configured.
您还可以使用 `MessageSourceAware` 接口获取对已定义的任何 `MessageSource` 的引用。在实现了 `MessageSourceAware` 接口的 `ApplicationContext` 中定义的任何bean在创建和配置bean时都会注入应用程序上下文的 `MessageSource` 。

> Because Spring’s `MessageSource` is based on Java’s `ResourceBundle`, it does not merge bundles with the same base name, but will only use the first bundle found. Subsequent message bundles with the same base name are ignored.
> 因为Spring的 `MessageSource` 基于Java的 `ResourceBundle` ，所以它不会合并具有相同基本名称的bundle，而是只使用找到的第一个bundle。具有相同基本名称的后续消息bundles将被忽略。

> As an alternative to `ResourceBundleMessageSource`, Spring provides a `ReloadableResourceBundleMessageSource` class. This variant supports the same bundle file format but is more flexible than the standard JDK based `ResourceBundleMessageSource` implementation. In particular, it allows for reading files from any Spring resource location (not only from the classpath) and supports hot reloading of bundle property files (while efficiently caching them in between). See the [`ReloadableResourceBundleMessageSource`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/support/ReloadableResourceBundleMessageSource.html) javadoc for details.
> 作为 `ResourceBundleMessageSource` 的替代，Spring提供了一个 `ReloadableResourceBundleMessageSource` 类。此变体支持相同的bundle文件格式，但比基于标准JDK的 `ResourceBundleMessageSource` 实现更灵活。特别是，它允许从任何Spring资源位置阅读文件（不仅仅是从类路径），并支持bundle属性文件的热重新加载（同时有效地缓存它们）。详情请参阅 [`ReloadableResourceBundleMessageSource`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/support/ReloadableResourceBundleMessageSource.html)javadoc。

#### 1.15.2.标准和自定义事件

Event handling in the `ApplicationContext` is provided through the `ApplicationEvent` class and the `ApplicationListener` interface. If a bean that implements the `ApplicationListener` interface is deployed into the context, every time an `ApplicationEvent` gets published to the `ApplicationContext`, that bean is notified. Essentially, this is the standard Observer design pattern.
`ApplicationContext` 中的事件处理通过 `ApplicationEvent` 类和 `ApplicationListener` 接口提供。如果一个实现了 `ApplicationListener` 接口的bean被部署到上下文中，那么每当一个 `ApplicationEvent` 被发布到 `ApplicationContext` 时，这个bean就会被通知。本质上，这是标准的观察者设计模式。

> As of Spring 4.2, the event infrastructure has been significantly improved and offers an [annotation-based model](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-events-annotation) as well as the ability to publish any arbitrary event (that is, an object that does not necessarily extend from `ApplicationEvent`). When such an object is published, we wrap it in an event for you.
> 从Spring 4.2开始，事件基础设施得到了显著的改进，并提供了一个[基于注释的模型](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-events-annotation)以及发布任何任意事件的能力（即，不一定从 `ApplicationEvent` 扩展的对象）。当这样的对象发布时，我们会将其包装在一个事件中。

| Event                        | Explanation                                                  |
| :--------------------------- | :----------------------------------------------------------- |
| `ContextRefreshedEvent`      | Published when the `ApplicationContext` is initialized or refreshed (for example, by using the `refresh()` method on the `ConfigurableApplicationContext` interface). Here, “initialized” means that all beans are loaded, post-processor beans are detected and activated, singletons are pre-instantiated, and the `ApplicationContext` object is ready for use. As long as the context has not been closed, a refresh can be triggered multiple times, provided that the chosen `ApplicationContext` actually supports such “hot” refreshes. For example, `XmlWebApplicationContext` supports hot refreshes, but `GenericApplicationContext` does not. 在初始化或刷新 `ApplicationContext` 时发布（例如，通过在 `ConfigurableApplicationContext` 接口上使用 `refresh()` 方法）。在这里，“初始化”意味着加载所有bean，检测并激活后处理器bean，预实例化单例，并且准备使用 `ApplicationContext` 对象。只要上下文尚未关闭，就可以多次触发刷新，只要所选的 `ApplicationContext` 实际上支持这种“热”刷新。例如， `XmlWebApplicationContext` 支持热刷新，但 `GenericApplicationContext` 不支持。 |
| `ContextStartedEvent`        | Published when the `ApplicationContext` is started by using the `start()` method on the `ConfigurableApplicationContext` interface. Here, “started” means that all `Lifecycle` beans receive an explicit start signal. Typically, this signal is used to restart beans after an explicit stop, but it may also be used to start components that have not been configured for autostart (for example, components that have not already started on initialization). 使用 `ConfigurableApplicationContext` 接口上的 `start()` 方法启动 `ApplicationContext` 时发布。这里，“started”意味着所有 `Lifecycle` bean都接收到一个显式的启动信号。通常，此信号用于在显式停止后重新启动Bean，但也可以用于启动尚未配置为自动启动的组件（例如，初始化时尚未启动的组件）。 |
| `ContextStoppedEvent`        | Published when the `ApplicationContext` is stopped by using the `stop()` method on the `ConfigurableApplicationContext` interface. Here, “stopped” means that all `Lifecycle` beans receive an explicit stop signal. A stopped context may be restarted through a `start()` call. 使用 `ConfigurableApplicationContext` 接口上的 `stop()` 方法停止 `ApplicationContext` 时发布。这里，“stopped”意味着所有 `Lifecycle` bean都接收到一个显式的停止信号。停止的上下文可以通过 `start()` 调用重新启动。 |
| `ContextClosedEvent`         | Published when the `ApplicationContext` is being closed by using the `close()` method on the `ConfigurableApplicationContext` interface or via a JVM shutdown hook. Here, "closed" means that all singleton beans will be destroyed. Once the context is closed, it reaches its end of life and cannot be refreshed or restarted. 当使用 `ConfigurableApplicationContext` 接口上的 `close()` 方法或通过JVM关闭钩子关闭 `ApplicationContext` 时发布。在这里，“closed”意味着所有singleton bean都将被销毁。一旦上下文关闭，它就达到了生命周期的终点，并且无法刷新或重新启动。 |
| `RequestHandledEvent`        | A web-specific event telling all beans that an HTTP request has been serviced. This event is published after the request is complete. This event is only applicable to web applications that use Spring’s `DispatcherServlet`. 一个特定于Web的事件，告诉所有Bean HTTP请求已得到服务。此事件在请求完成后发布。此事件仅适用于使用Spring的 `DispatcherServlet` 的Web应用程序。 |
| `ServletRequestHandledEvent` | A subclass of `RequestHandledEvent` that adds Servlet-specific context information. `RequestHandledEvent` 的一个子类，添加Servlet特定的上下文信息。 |

You can also create and publish your own custom events. The following example shows a simple class that extends Spring’s `ApplicationEvent` base class:
您还可以创建和发布自己的自定义事件。下面的例子展示了一个简单的类，它扩展了Spring的 `ApplicationEvent` 基类：

```java
public class BlockedListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlockedListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

To publish a custom `ApplicationEvent`, call the `publishEvent()` method on an `ApplicationEventPublisher`. Typically, this is done by creating a class that implements `ApplicationEventPublisherAware` and registering it as a Spring bean. The following example shows such a class:
要发布自定义 `ApplicationEvent` ，请在 `ApplicationEventPublisher` 上调用 `publishEvent()` 方法。通常，这是通过创建一个实现 `ApplicationEventPublisherAware` 的类并将其注册为Spring bean来完成的。下面的示例显示了这样一个类：

```java
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blockedList;
    private ApplicationEventPublisher publisher;

    public void setBlockedList(List<String> blockedList) {
        this.blockedList = blockedList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blockedList.contains(address)) {
            publisher.publishEvent(new BlockedListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

At configuration time, the Spring container detects that `EmailService` implements `ApplicationEventPublisherAware` and automatically calls `setApplicationEventPublisher()`. In reality, the parameter passed in is the Spring container itself. You are interacting with the application context through its `ApplicationEventPublisher` interface.
在配置时，Spring容器检测到 `EmailService` 实现了 `ApplicationEventPublisherAware` 并自动调用 `setApplicationEventPublisher()` 。实际上，传入的参数是Spring容器本身。您通过应用程序上下文的 `ApplicationEventPublisher` 接口与应用程序上下文进行交互。

To receive the custom `ApplicationEvent`, you can create a class that implements `ApplicationListener` and register it as a Spring bean. The following example shows such a class:
要接收自定义的 `ApplicationEvent` ，您可以创建一个实现 `ApplicationListener` 的类并将其注册为Spring bean。下面的示例显示了这样一个类：

```java
public class BlockedListNotifier implements ApplicationListener<BlockedListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

Notice that `ApplicationListener` is generically parameterized with the type of your custom event (`BlockedListEvent` in the preceding example). This means that the `onApplicationEvent()` method can remain type-safe, avoiding any need for downcasting. You can register as many event listeners as you wish, but note that, by default, event listeners receive events synchronously. This means that the `publishEvent()` method blocks until all listeners have finished processing the event. One advantage of this synchronous and single-threaded approach is that, when a listener receives an event, it operates inside the transaction context of the publisher if a transaction context is available. If another strategy for event publication becomes necessary, see the javadoc for Spring’s [`ApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/event/ApplicationEventMulticaster.html) interface and [`SimpleApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/event/SimpleApplicationEventMulticaster.html) implementation for configuration options.
请注意， `ApplicationListener` 一般使用自定义事件的类型进行参数化（上例中的 `BlockedListEvent` ）。这意味着 `onApplicationEvent()` 方法可以保持类型安全，避免任何向下转换的需要。您可以注册任意多个事件侦听器，但请注意，默认情况下，事件侦听器同步接收事件。这意味着 `publishEvent()` 方法会阻塞，直到所有侦听器都完成了对事件的处理。这种同步和单线程方法的一个优点是，当侦听器接收到事件时，如果事务上下文可用，它将在发布者的事务上下文内操作。如果需要其他的事件发布策略，请参阅javadoc for Spring的 [`ApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/event/ApplicationEventMulticaster.html) 接口和 [`SimpleApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/event/SimpleApplicationEventMulticaster.html) 实现以获取配置选项。

The following example shows the bean definitions used to register and configure each of the classes above:
以下示例显示了用于注册和配置上述每个类的Bean定义：

```xml
<bean id="emailService" class="example.EmailService">
    <property name="blockedList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blockedListNotifier" class="example.BlockedListNotifier">
    <property name="notificationAddress" value="blockedlist@example.org"/>
</bean>
```

Putting it all together, when the `sendEmail()` method of the `emailService` bean is called, if there are any email messages that should be blocked, a custom event of type `BlockedListEvent` is published. The `blockedListNotifier` bean is registered as an `ApplicationListener` and receives the `BlockedListEvent`, at which point it can notify appropriate parties.
总之，当调用 `emailService` bean的 `sendEmail()` 方法时，如果有任何应该被阻止的电子邮件消息，则会发布一个类型为 `BlockedListEvent` 的自定义事件。 `blockedListNotifier` bean注册为 `ApplicationListener` 并接收 `BlockedListEvent` ，此时它可以通知适当的各方。

> Spring’s eventing mechanism is designed for simple communication between Spring beans within the same application context. However, for more sophisticated enterprise integration needs, the separately maintained [Spring Integration](https://projects.spring.io/spring-integration/) project provides complete support for building lightweight, [pattern-oriented](https://www.enterpriseintegrationpatterns.com/), event-driven architectures that build upon the well-known Spring programming model.
> Spring的事件机制是为同一应用程序上下文中的Springbean之间的简单通信而设计的。然而，对于更复杂的企业集成需求，单独维护的[Spring Integration](https://projects.spring.io/spring-integration/)项目为构建轻量级，[面向模式](https://www.enterpriseintegrationpatterns.com/)，事件驱动的架构提供了完整的支持，这些架构构建在众所周知的Spring编程模型上。

##### 基于注释的事件侦听器

You can register an event listener on any method of a managed bean by using the `@EventListener` annotation. The `BlockedListNotifier` can be rewritten as follows:
您可以使用 `@EventListener` 注释在托管bean的任何方法上注册事件侦听器。 `BlockedListNotifier` 可以重写如下：

```java
public class BlockedListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlockedListEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

The method signature once again declares the event type to which it listens, but, this time, with a flexible name and without implementing a specific listener interface. The event type can also be narrowed through generics as long as the actual event type resolves your generic parameter in its implementation hierarchy.
方法签名再次声明了它所监听的事件类型，但这一次使用了灵活的名称，并且没有实现特定的监听器接口。只要实际的事件类型在其实现层次结构中解析泛型参数，就可以通过泛型缩小事件类型的范围。

If your method should listen to several events or if you want to define it with no parameter at all, the event types can also be specified on the annotation itself. The following example shows how to do so:
如果你的方法需要监听多个事件，或者你想定义一个不带任何参数的方法，那么事件类型也可以在注解本身指定。下面的示例说明如何执行此操作：

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```

It is also possible to add additional runtime filtering by using the `condition` attribute of the annotation that defines a [`SpEL` expression](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions), which should match to actually invoke the method for a particular event.
还可以通过使用注释的 `condition` 属性来添加额外的运行时过滤，该属性定义了 `SpEL` 表达式，该表达式应该匹配以实际调用特定事件的方法。

The following example shows how our notifier can be rewritten to be invoked only if the `content` attribute of the event is equal to `my-event`:
下面的例子展示了如何重写我们的通知程序，使其仅在事件的 `content` 属性等于 `my-event` 时才被调用：

```java
@EventListener(condition = "#blEvent.content == 'my-event'")
public void processBlockedListEvent(BlockedListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```

Each `SpEL` expression evaluates against a dedicated context. The following table lists the items made available to the context so that you can use them for conditional event processing:
每个 `SpEL` 表达式都针对专用上下文进行计算。下表列出了可用于上下文的项，以便您可以将其用于条件事件处理：

表8。事件SpEL可用元数据

| Name            | Location                    | Description                                                  | Example                                                      |
| :-------------- | :-------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Event           | root object                 | The actual `ApplicationEvent`. 真实的 `ApplicationEvent`     | `#root.event` or `event` `#root.event` 或 `event`            |
| Arguments array | root object                 | The arguments (as an object array) used to invoke the method. 用于调用方法的参数（作为对象数组）。 | `#root.args` or `args`; `args[0]` to access the first argument, etc. `#root.args` 或 `args` ; `args[0]` 访问第一个参数，依此类推。 |
| *Argument name* | evaluation context 评价语境 | The name of any of the method arguments. If, for some reason, the names are not available (for example, because there is no debug information in the compiled byte code), individual arguments are also available using the `#a<#arg>` syntax where `<#arg>` stands for the argument index (starting from 0). 任何方法参数的名称。如果由于某种原因，名称不可用（例如，因为编译的字节码中没有调试信息），则使用 `#a<#arg>` 语法也可以使用单个参数，其中 `<#arg>` 代表参数索引（从0开始）。 |                                                              |

Note that `#root.event` gives you access to the underlying event, even if your method signature actually refers to an arbitrary object that was published.
请注意， `#root.event` 允许您访问基础事件，即使您的方法签名实际上引用了已发布的任意对象。

If you need to publish an event as the result of processing another event, you can change the method signature to return the event that should be published, as the following example shows:
如果需要发布一个事件作为处理另一个事件的结果，则可以更改方法签名以返回应发布的事件，如下例所示：

```java
@EventListener
public ListUpdateEvent handleBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```

> This feature is not supported for [asynchronous listeners](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-events-async).
> [异步侦听器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-events-async)不支持此功能。

The `handleBlockedListEvent()` method publishes a new `ListUpdateEvent` for every `BlockedListEvent` that it handles. If you need to publish several events, you can return a `Collection` or an array of events instead.
`handleBlockedListEvent()` 方法为它处理的每个 `BlockedListEvent` 发布新的 `ListUpdateEvent` 。如果需要发布多个事件，可以返回 `Collection` 或事件数组。

##### 异步监听器

If you want a particular listener to process events asynchronously, you can reuse the [regular `@Async` support](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#scheduling-annotation-support-async). The following example shows how to do so:
如果你想让一个特定的监听器异步处理事件，你可以重用[常规的 `@Async` 支持](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#scheduling-annotation-support-async)。下面的示例说明如何执行此操作：

```java
@EventListener
@Async
public void processBlockedListEvent(BlockedListEvent event) {
    // BlockedListEvent is processed in a separate thread
}
```

Be aware of the following limitations when using asynchronous events:
使用异步事件时，请注意以下限制：

- If an asynchronous event listener throws an `Exception`, it is not propagated to the caller. See [`AsyncUncaughtExceptionHandler`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/aop/interceptor/AsyncUncaughtExceptionHandler.html) for more details.
  如果异步事件侦听器抛出 `Exception` ，则不会将其传播到调用方。更多详情请参见 [`AsyncUncaughtExceptionHandler`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/aop/interceptor/AsyncUncaughtExceptionHandler.html)。
- Asynchronous event listener methods cannot publish a subsequent event by returning a value. If you need to publish another event as the result of the processing, inject an [`ApplicationEventPublisher`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/ApplicationEventPublisher.html) to publish the event manually.
  异步事件侦听器方法不能通过返回值来发布后续事件。如果您需要发布另一个事件作为处理的结果，请注入 [`ApplicationEventPublisher`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/context/ApplicationEventPublisher.html) 以手动发布事件。

##### 排序监听器

If you need one listener to be invoked before another one, you can add the `@Order` annotation to the method declaration, as the following example shows:
如果你需要一个监听器在另一个监听器之前被调用，你可以在方法声明中添加 `@Order` 注解，如下面的例子所示：

```java
@EventListener
@Order(42)
public void processBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

##### 泛型事件

You can also use generics to further define the structure of your event. Consider using an `EntityCreatedEvent<T>` where `T` is the type of the actual entity that got created. For example, you can create the following listener definition to receive only `EntityCreatedEvent` for a `Person`:
您还可以使用泛型来进一步定义事件的结构。考虑使用 `EntityCreatedEvent<T>` ，其中 `T` 是创建的实际实体的类型。例如，您可以创建以下侦听器定义，以便仅接收 `Person` 的 `EntityCreatedEvent` ：

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```

Due to type erasure, this works only if the event that is fired resolves the generic parameters on which the event listener filters (that is, something like `class PersonCreatedEvent extends EntityCreatedEvent<Person> { … }`).
由于类型擦除，只有当触发的事件解析了事件侦听器过滤的泛型参数（即类似 `class PersonCreatedEvent extends EntityCreatedEvent<Person> { …​ }` 的参数）时，这才有效。

In certain circumstances, this may become quite tedious if all events follow the same structure (as should be the case for the event in the preceding example). In such a case, you can implement `ResolvableTypeProvider` to guide the framework beyond what the runtime environment provides. The following event shows how to do so:
在某些情况下，如果所有事件都遵循相同的结构（如上例中的事件），这可能会变得非常乏味。在这种情况下，您可以实现 `ResolvableTypeProvider` 来指导框架超出运行时环境提供的范围。以下事件说明了如何执行此操作：

```java
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getSource()));
    }
}
```

> This works not only for `ApplicationEvent` but any arbitrary object that you send as an event.
> 这不仅适用于 `ApplicationEvent` ，还适用于任何作为事件发送的任意对象。

#### 1.15.3.方便地访问低级资源

For optimal usage and understanding of application contexts, you should familiarize yourself with Spring’s `Resource` abstraction, as described in [Resources](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources).
为了最佳地使用和理解应用程序上下文，您应该熟悉Spring的 `Resource` 抽象，如[参考资料](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources)中所述。

An application context is a `ResourceLoader`, which can be used to load `Resource` objects. A `Resource` is essentially a more feature rich version of the JDK `java.net.URL` class. In fact, the implementations of the `Resource` wrap an instance of `java.net.URL`, where appropriate. A `Resource` can obtain low-level resources from almost any location in a transparent fashion, including from the classpath, a filesystem location, anywhere describable with a standard URL, and some other variations. If the resource location string is a simple path without any special prefixes, where those resources come from is specific and appropriate to the actual application context type.
应用程序上下文是 `ResourceLoader` ，可用于加载 `Resource` 对象。 `Resource` 本质上是JDK `java.net.URL` 类的功能更丰富的版本。事实上， `Resource` 的实现在适当的情况下包装了 `java.net.URL` 的实例。 `Resource` 可以以透明的方式从几乎任何位置获取低级资源，包括从类路径、文件系统位置、任何可以用标准URL描述的位置，以及其他一些变体。如果资源位置字符串是一个没有任何特殊前缀的简单路径，则这些资源的来源是特定的，并且适合于实际的应用程序上下文类型。

You can configure a bean deployed into the application context to implement the special callback interface, `ResourceLoaderAware`, to be automatically called back at initialization time with the application context itself passed in as the `ResourceLoader`. You can also expose properties of type `Resource`, to be used to access static resources. They are injected into it like any other properties. You can specify those `Resource` properties as simple `String` paths and rely on automatic conversion from those text strings to actual `Resource` objects when the bean is deployed.
您可以配置部署到应用程序上下文中的bean来实现特殊的回调接口 `ResourceLoaderAware` ，以便在初始化时自动回调，并将应用程序上下文本身作为 `ResourceLoader` 传入。您还可以公开类型为 `Resource` 的属性，用于访问静态资源。它们像任何其他属性一样被注入其中。您可以将这些 `Resource` 属性指定为简单的 `String` 路径，并在部署bean时依赖于从这些文本字符串到实际 `Resource` 对象的自动转换。

The location path or paths supplied to an `ApplicationContext` constructor are actually resource strings and, in simple form, are treated appropriately according to the specific context implementation. For example `ClassPathXmlApplicationContext` treats a simple location path as a classpath location. You can also use location paths (resource strings) with special prefixes to force loading of definitions from the classpath or a URL, regardless of the actual context type.
提供给 `ApplicationContext` 构造函数的一个或多个位置路径实际上是资源字符串，并且以简单的形式根据特定的上下文实现进行适当的处理。例如， `ClassPathXmlApplicationContext` 将简单的位置路径视为类路径位置。您还可以使用带有特殊前缀的位置路径（资源字符串）来强制从类路径或URL加载定义，而不考虑实际的上下文类型。

#### 1.15.4.应用程序启动跟踪

The `ApplicationContext` manages the lifecycle of Spring applications and provides a rich programming model around components. As a result, complex applications can have equally complex component graphs and startup phases.
`ApplicationContext` 管理Spring应用程序的生命周期，并围绕组件提供丰富的编程模型。因此，复杂的应用程序可以具有同样复杂的组件图和启动阶段。

Tracking the application startup steps with specific metrics can help understand where time is being spent during the startup phase, but it can also be used as a way to better understand the context lifecycle as a whole.
使用特定指标跟踪应用程序启动步骤可以帮助了解启动阶段的时间花费情况，但也可以将其用作更好地了解整个上下文生命周期的方法。

The `AbstractApplicationContext` (and its subclasses) is instrumented with an `ApplicationStartup`, which collects `StartupStep` data about various startup phases:
`AbstractApplicationContext` （及其子类）使用 `ApplicationStartup` 进行检测，它收集关于各种启动阶段的 `StartupStep` 数据：

- application context lifecycle (base packages scanning, config classes management)
  应用程序上下文生命周期（基础包扫描、配置类管理）
- beans lifecycle (instantiation, smart initialization, post processing)
  beans生命周期（实例化、智能初始化、后期处理）
- application events processing 应用程序事件处理

Here is an example of instrumentation in the `AnnotationConfigApplicationContext`:
以下是 `AnnotationConfigApplicationContext` 中的仪器示例：

```java
// create a startup step and start recording
StartupStep scanPackages = this.getApplicationStartup().start("spring.context.base-packages.scan");
// add tagging information to the current step
scanPackages.tag("packages", () -> Arrays.toString(basePackages));
// perform the actual phase we're instrumenting
this.scanner.scan(basePackages);
// end the current step
scanPackages.end();
```

The application context is already instrumented with multiple steps. Once recorded, these startup steps can be collected, displayed and analyzed with specific tools. For a complete list of existing startup steps, you can check out the [dedicated appendix section](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core.appendix.application-startup-steps).
应用程序上下文已经通过多个步骤进行了检测。一旦记录下来，这些启动步骤就可以用特定的工具收集、显示和分析。有关现有启动步骤的完整列表，您可以查看[专用附录部分](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core.appendix.application-startup-steps)。

The default `ApplicationStartup` implementation is a no-op variant, for minimal overhead. This means no metrics will be collected during application startup by default. Spring Framework ships with an implementation for tracking startup steps with Java Flight Recorder: `FlightRecorderApplicationStartup`. To use this variant, you must configure an instance of it to the `ApplicationContext` as soon as it’s been created.
默认的 `ApplicationStartup` 实现是一个no-op变体，用于最小的开销。这意味着默认情况下，在应用程序启动期间不会收集任何指标。Spring Framework附带了一个实现，用于跟踪Java Flight Recorder的启动步骤：`FlightRecorderApplicationStartup`。要使用这个变体，您必须在它创建后立即将它的实例配置到 `ApplicationContext` 。

Developers can also use the `ApplicationStartup` infrastructure if they’re providing their own `AbstractApplicationContext` subclass, or if they wish to collect more precise data.
如果开发人员提供了自己的 `AbstractApplicationContext` 子类，或者希望收集更精确的数据，他们也可以使用 `ApplicationStartup` 基础设施。

> `ApplicationStartup` is meant to be only used during application startup and for the core container; this is by no means a replacement for Java profilers or metrics libraries like [Micrometer](https://micrometer.io/).
> `ApplicationStartup` 仅在应用程序启动期间使用，并用于核心容器;这绝不是Java分析器或像Micrometer这样的度量库的替代品。

To start collecting custom `StartupStep`, components can either get the `ApplicationStartup` instance from the application context directly, make their component implement `ApplicationStartupAware`, or ask for the `ApplicationStartup` type on any injection point.
要开始收集自定义 `StartupStep` ，组件可以直接从应用程序上下文获取 `ApplicationStartup` 实例，使其组件实现 `ApplicationStartupAware` ，或者在任何注入点上请求 `ApplicationStartup` 类型。

> Developers should not use the `"spring.*"` namespace when creating custom startup steps. This namespace is reserved for internal Spring usage and is subject to change.
> 开发人员在创建自定义启动步骤时不应使用 `"spring.*"` 命名空间。此命名空间保留给内部Spring使用，可能会发生更改

#### 1.15.5.方便的Web应用程序ApplicationContext实例化

You can create `ApplicationContext` instances declaratively by using, for example, a `ContextLoader`. Of course, you can also create `ApplicationContext` instances programmatically by using one of the `ApplicationContext` implementations.
您可以通过使用声明方式创建 `ApplicationContext` 实例，例如，使用 `ContextLoader` 。当然，您也可以通过使用 `ApplicationContext` 实现之一以编程方式创建 `ApplicationContext` 实例。

You can register an `ApplicationContext` by using the `ContextLoaderListener`, as the following example shows:
您可以使用 `ContextLoaderListener` 注册 `ApplicationContext` ，如下例所示：

```java
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

The listener inspects the `contextConfigLocation` parameter. If the parameter does not exist, the listener uses `/WEB-INF/applicationContext.xml` as a default. When the parameter does exist, the listener separates the `String` by using predefined delimiters (comma, semicolon, and whitespace) and uses the values as locations where application contexts are searched. Ant-style path patterns are supported as well. Examples are `/WEB-INF/*Context.xml` (for all files with names that end with `Context.xml` and that reside in the `WEB-INF` directory) and `/WEB-INF/**/*Context.xml` (for all such files in any subdirectory of `WEB-INF`).
监听器检查 `contextConfigLocation` 参数。如果该参数不存在，监听器将使用 `/WEB-INF/applicationContext.xml` 作为默认值。当参数确实存在时，侦听器使用预定义的分隔符（逗号、分号和空格）分隔 `String` ，并使用这些值作为搜索应用程序上下文的位置。也支持蚂蚁样式的路径模式。例如 `/WEB-INF/*Context.xml` （针对所有文件名以 `Context.xml` 结尾，并且位于 `WEB-INF` 目录中）和 `/WEB-INF/**/*Context.xml` （针对 `WEB-INF` 的任何子目录中的所有此类文件）。

#### 1.15.6.将Spring `ApplicationContext` 部署为Jakarta EE RAR文件

It is possible to deploy a Spring `ApplicationContext` as a RAR file, encapsulating the context and all of its required bean classes and library JARs in a Jakarta EE RAR deployment unit. This is the equivalent of bootstrapping a stand-alone `ApplicationContext` (only hosted in Jakarta EE environment) being able to access the Jakarta EE servers facilities. RAR deployment is a more natural alternative to a scenario of deploying a headless WAR file — in effect, a WAR file without any HTTP entry points that is used only for bootstrapping a Spring `ApplicationContext` in a Jakarta EE environment.
可以将Spring `ApplicationContext` 部署为RAR文件，将上下文及其所有必需的bean类和库JAR封装在Jakarta EE RAR部署单元中。这相当于引导独立的 `ApplicationContext` （仅托管在Jakarta EE环境中）能够访问Jakarta EE服务器设施。RAR部署是部署无头WAR文件的场景的更自然的替代方案-实际上，没有任何HTTP入口点的WAR文件仅用于在Jakarta EE环境中引导Spring `ApplicationContext` 。

RAR deployment is ideal for application contexts that do not need HTTP entry points but rather consist only of message endpoints and scheduled jobs. Beans in such a context can use application server resources such as the JTA transaction manager and JNDI-bound JDBC `DataSource` instances and JMS `ConnectionFactory` instances and can also register with the platform’s JMX server — all through Spring’s standard transaction management and JNDI and JMX support facilities. Application components can also interact with the application server’s JCA `WorkManager` through Spring’s `TaskExecutor` abstraction.
RAR部署非常适合不需要HTTP入口点但仅包含消息端点和计划作业的应用程序上下文。在这样的上下文中，Bean可以使用应用程序服务器资源，例如JTA事务管理器和JNDI绑定的JDBC `DataSource` 实例和JMS `ConnectionFactory` 实例，并且还可以向平台的JMX服务器注册-所有这些都通过Spring的标准事务管理和JNDI和JMX支持设施进行。应用程序组件还可以通过Spring的 `TaskExecutor` 抽象与应用程序服务器的JCA `WorkManager` 交互。

See the javadoc of the [`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html) class for the configuration details involved in RAR deployment.
RAR部署涉及的配置细节，请参见 [`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html)类的javadoc。

For a simple deployment of a Spring ApplicationContext as a Jakarta EE RAR file:
对于Spring ApplicationContext作为Jakarta EE RAR文件的简单部署：

1. Package all application classes into a RAR file (which is a standard JAR file with a different file extension).
   将所有应用程序类打包到一个RAR文件中（这是一个具有不同文件扩展名的标准JAR文件）。
2. Add all required library JARs into the root of the RAR archive.
   将所有需要的库JAR添加到RAR存档的根目录中。
3. Add a `META-INF/ra.xml` deployment descriptor (as shown in the [javadoc for `SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/6.0.8/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html)) and the corresponding Spring XML bean definition file(s) (typically `META-INF/applicationContext.xml`).
   添加一个 `META-INF/ra.xml` 部署描述符（如 `SpringContextResourceAdapter` 的javadoc所示）和相应的SpringXMLbean定义文件（通常为 `META-INF/applicationContext.xml` ）。
4. Drop the resulting RAR file into your application server’s deployment directory.
   将生成的RAR文件拖放到应用程序服务器的部署目录中。

> Such RAR deployment units are usually self-contained. They do not expose components to the outside world, not even to other modules of the same application. Interaction with a RAR-based `ApplicationContext` usually occurs through JMS destinations that it shares with other modules. A RAR-based `ApplicationContext` may also, for example, schedule some jobs or react to new files in the file system (or the like). If it needs to allow synchronous access from the outside, it could (for example) export RMI endpoints, which may be used by other application modules on the same machine.
> 这样的RAR部署单元通常是自包含的。它们不向外界公开组件，甚至不向同一应用程序的其他模块公开组件。与基于RAR的 `ApplicationContext` 的交互通常通过它与其他模块共享的JMS目的地发生。基于RAR的 `ApplicationContext` 还可以例如调度一些作业或对文件系统中的新文件作出反应（等）。如果它需要允许从外部同步访问，它可以（例如）导出RMI端点，这些端点可以由同一机器上的其他应用程序模块使用。

### 1.16. `BeanFactory` API

The `BeanFactory` API provides the underlying basis for Spring’s IoC functionality. Its specific contracts are mostly used in integration with other parts of Spring and related third-party frameworks, and its `DefaultListableBeanFactory` implementation is a key delegate within the higher-level `GenericApplicationContext` container.
`BeanFactory` API为Spring的IoC功能提供了底层基础。它的特定合约主要用于与Spring的其他部分和相关的第三方框架集成，它的 `DefaultListableBeanFactory` 实现是更高级别的 `GenericApplicationContext` 容器中的关键委托。

`BeanFactory` and related interfaces (such as `BeanFactoryAware`, `InitializingBean`, `DisposableBean`) are important integration points for other framework components. By not requiring any annotations or even reflection, they allow for very efficient interaction between the container and its components. Application-level beans may use the same callback interfaces but typically prefer declarative dependency injection instead, either through annotations or through programmatic configuration.
`BeanFactory` 和相关接口（如 `BeanFactoryAware` 、 `InitializingBean` 、 `DisposableBean` ）是其他框架组件的重要集成点。由于不需要任何注释甚至反射，它们允许容器及其组件之间非常有效的交互。应用程序级别的bean可以使用相同的回调接口，但通常更喜欢声明性依赖注入，无论是通过注释还是通过编程配置。

Note that the core `BeanFactory` API level and its `DefaultListableBeanFactory` implementation do not make assumptions about the configuration format or any component annotations to be used. All of these flavors come in through extensions (such as `XmlBeanDefinitionReader` and `AutowiredAnnotationBeanPostProcessor`) and operate on shared `BeanDefinition` objects as a core metadata representation. This is the essence of what makes Spring’s container so flexible and extensible.
请注意，核心 `BeanFactory` API级别及其 `DefaultListableBeanFactory` 实现并没有对要使用的配置格式或任何组件注释进行假设。所有这些风格都是通过扩展（例如 `XmlBeanDefinitionReader` 和 `AutowiredAnnotationBeanPostProcessor` ）引入的，并在共享的 `BeanDefinition` 对象上作为核心元数据表示进行操作。这就是使Spring的容器如此灵活和可扩展的本质。

#### 1.16.1. `BeanFactory` 还是 `ApplicationContext` ？

This section explains the differences between the `BeanFactory` and `ApplicationContext` container levels and the implications on bootstrapping.
本节解释 `BeanFactory` 和 `ApplicationContext` 容器级别之间的差异以及对引导的影响。

You should use an `ApplicationContext` unless you have a good reason for not doing so, with `GenericApplicationContext` and its subclass `AnnotationConfigApplicationContext` as the common implementations for custom bootstrapping. These are the primary entry points to Spring’s core container for all common purposes: loading of configuration files, triggering a classpath scan, programmatically registering bean definitions and annotated classes, and (as of 5.0) registering functional bean definitions.
你应该使用 `ApplicationContext` ，除非你有很好的理由不这样做， `GenericApplicationContext` 和它的子类 `AnnotationConfigApplicationContext` 作为自定义引导的常见实现。这些是Spring核心容器的主要入口点，用于所有常见目的：加载配置文件，触发类路径扫描，以编程方式注册bean定义和带注释的类，以及（从5.0开始）注册函数bean定义。

Because an `ApplicationContext` includes all the functionality of a `BeanFactory`, it is generally recommended over a plain `BeanFactory`, except for scenarios where full control over bean processing is needed. Within an `ApplicationContext` (such as the `GenericApplicationContext` implementation), several kinds of beans are detected by convention (that is, by bean name or by bean type — in particular, post-processors), while a plain `DefaultListableBeanFactory` is agnostic about any special beans.
因为 `ApplicationContext` 包含了 `BeanFactory` 的所有功能，所以通常建议使用它而不是普通的 `BeanFactory` ，除非需要对bean处理进行完全控制。在 `ApplicationContext` （比如 `GenericApplicationContext` 实现）中，几种bean是按照约定检测的（也就是说，通过bean名称或bean类型-特别是后处理器），而普通的 `DefaultListableBeanFactory` 对任何特殊的bean都是不可知的。

For many extended container features, such as annotation processing and AOP proxying, the [`BeanPostProcessor` extension point](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp) is essential. If you use only a plain `DefaultListableBeanFactory`, such post-processors do not get detected and activated by default. This situation could be confusing, because nothing is actually wrong with your bean configuration. Rather, in such a scenario, the container needs to be fully bootstrapped through additional setup.
对于许多扩展的容器特性，比如注释处理和AOP代理， `BeanPostProcessor` 扩展点是必不可少的。如果您只使用普通的 `DefaultListableBeanFactory` ，则默认情况下不会检测和激活此类后处理器。这种情况可能令人困惑，因为您的bean配置实际上没有任何错误。相反，在这样的场景中，容器需要通过附加设置完全自举。

The following table lists features provided by the `BeanFactory` and `ApplicationContext` interfaces and implementations.
下表列出了 `BeanFactory` 和 `ApplicationContext` 接口提供的功能和实现。

 在表9中。特性矩阵

| Feature                                                      | `BeanFactory` | `ApplicationContext` |
| :----------------------------------------------------------- | :------------ | :------------------- |
| Bean instantiation/wiring Bean实例化/连接                    | Yes           | Yes                  |
| Integrated lifecycle management 集成生命周期管理             | No            | Yes                  |
| Automatic `BeanPostProcessor` registration 自动 `BeanPostProcessor` 注册 | No            | Yes                  |
| Automatic `BeanFactoryPostProcessor` registration 自动 `BeanFactoryPostProcessor` 注册 | No            | Yes                  |
| Convenient `MessageSource` access (for internationalization) 方便的 `MessageSource` 访问（用于国际化） | No            | Yes                  |
| Built-in `ApplicationEvent` publication mechanism 内置 `ApplicationEvent` 发布机制 | No            | Yes                  |

To explicitly register a bean post-processor with a `DefaultListableBeanFactory`, you need to programmatically call `addBeanPostProcessor`, as the following example shows:
要使用 `DefaultListableBeanFactory` 显式注册bean后处理器，您需要以编程方式调用 `addBeanPostProcessor` ，如以下示例所示：

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

To apply a `BeanFactoryPostProcessor` to a plain `DefaultListableBeanFactory`, you need to call its `postProcessBeanFactory` method, as the following example shows:
要将 `BeanFactoryPostProcessor` 应用于普通的 `DefaultListableBeanFactory` ，您需要调用其 `postProcessBeanFactory` 方法，如以下示例所示：

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertySourcesPlaceholderConfigurer cfg = new PropertySourcesPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```

In both cases, the explicit registration steps are inconvenient, which is why the various `ApplicationContext` variants are preferred over a plain `DefaultListableBeanFactory` in Spring-backed applications, especially when relying on `BeanFactoryPostProcessor` and `BeanPostProcessor` instances for extended container functionality in a typical enterprise setup.
在这两种情况下，显式注册步骤都不方便，这就是为什么在Spring-backed应用程序中，各种 `ApplicationContext` 变体比普通的 `DefaultListableBeanFactory` 更受欢迎，特别是在典型的企业设置中依赖于 `BeanFactoryPostProcessor` 和 `BeanPostProcessor` 实例来扩展容器功能时。

> An `AnnotationConfigApplicationContext` has all common annotation post-processors registered and may bring in additional processors underneath the covers through configuration annotations, such as `@EnableTransactionManagement`. At the abstraction level of Spring’s annotation-based configuration model, the notion of bean post-processors becomes a mere internal container detail.
> `AnnotationConfigApplicationContext` 注册了所有常见的注释后处理器，并且可以通过配置注释（例如 `@EnableTransactionManagement` ）在盖子下面引入额外的处理器。在Spring的基于注释的配置模型的抽象层，bean后处理器的概念变成了一个纯粹的内部容器细节。