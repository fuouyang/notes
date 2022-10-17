# Spring加载流程解析



## 前置准备

1. 导入依赖

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
```

 上面这个spring-context包会依赖导入下面几个包

```
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>5.2.9.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>5.2.9.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>5.2.9.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
      <version>5.2.9.RELEASE</version>
      <scope>compile</scope>
    </dependency>
```

   一个Spring项目就只需要上面五个包就可以启动一个Spring容器了。

2. 新建Spring配置文件spring.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://mybatis.org/schema/mybatis-spring
        http://mybatis.org/schema/mybatis-spring.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>
</beans>
```
3. 进入Spring入口类

```
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

/**
 * Unit test for simple App.
 */
public class AppTest {

    @Test
    public void test1() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext("org.example");

    }

    @Test
    public void test2() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    }

    @Test
    public void test3() {
        ApplicationContext applicationContext = new FileSystemXmlApplicationContext("/Users/fuouyang/Code/Spring/src/main/resources/spring.xml");
    }

}
```

   上面列举了三个Spring的入口类，其中AnnotationConfigApplicationContext和ClassPathXmlApplicationContext是我们学习Spring时比较熟悉的两个类，下面会分别对这两个入口类分别做详细的解析

***



## 流程解析

###  ClassPathXmlApplicationContext启动

​        尤记得是一个单纯的小白学习Spring时新建一个Spring MVC项目，在web.xml配置各种Servlet、Intercepor，还有Spring配置文件时的样子，这种方式就是用的ClassPatXmlApplicationContext来启动Spring容器的，下面来具体看下这个类是如何启动Spring

```

	/**
	 * Create a new ClassPathXmlApplicationContext for bean-style configuration.
	 * @see #setConfigLocation
	 * @see #setConfigLocations
	 * @see #afterPropertiesSet()
	 */
	public ClassPathXmlApplicationContext() {
	}

	/**
	 * Create a new ClassPathXmlApplicationContext for bean-style configuration.
	 * @param parent the parent context
	 * @see #setConfigLocation
	 * @see #setConfigLocations
	 * @see #afterPropertiesSet()
	 */
	public ClassPathXmlApplicationContext(ApplicationContext parent) {
		super(parent);
	}

	/**
	 * Create a new ClassPathXmlApplicationContext, loading the definitions
	 * from the given XML file and automatically refreshing the context.
	 * @param configLocation resource location
	 * @throws BeansException if context creation failed
	 */
	public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}

	/**
	 * Create a new ClassPathXmlApplicationContext, loading the definitions
	 * from the given XML files and automatically refreshing the context.
	 * @param configLocations array of resource locations
	 * @throws BeansException if context creation failed
	 */
	public ClassPathXmlApplicationContext(String... configLocations) throws BeansException {
		this(configLocations, true, null);
	}
```

​    可以看到ClassPathXmlApplicationContext有很多的构造函数，通常我们用的是`public ClassPathXmlApplicationContext(String configLocation)`这个，传入Spring配置文件来启动，如下：


```
 ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
```

------

​       接下来分析一下这个类new出来后都做了什么事情，为了更好的理解方法之间的调用关系，我们先看下ClassPathXmlApplicationContext的类继承关系

![ClassPathXmlApplicationContext](https://tva1.sinaimg.cn/large/008vxvgGly1h754xerzdcj32230u0q7h.jpg)

1. 新建对象 ` new ClassPathXmlApplicationContext("spring.xml")`

   ```
   	public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
   		this(new String[] {configLocation}, true, null);
   	}
   ```

2. ` new ClassPathXmlApplicationContext("spring.xml")`会调用`public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)throws BeansException`

    ```
   public ClassPathXmlApplicationContext(
   			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
   			throws BeansException {
   		super(parent);
   		//设置配置文件路径
   		setConfigLocations(configLocations);
   		if (refresh) {
   		  //最重要的方法 执行容器启动
   			refresh();
   		}
   	}
    ```

3. 上面的方法中refresh()是核心方法，在这里完成Spring容器bean的加载，该方法在AbstractApplicationContext中声明

     ```
     @Override
     	public void refresh() throws BeansException, IllegalStateException {
     		synchronized (this.startupShutdownMonitor) {
     			// Prepare this context for refreshing.
     			prepareRefresh();
     
     			// Tell the subclass to refresh the internal bean factory.
     			// 非常重要的一个方法，
     			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
     
     			// Prepare the bean factory for use in this context.
     			prepareBeanFactory(beanFactory);
     
     			try {
     				// Allows post-processing of the bean factory in context subclasses.
     				postProcessBeanFactory(beanFactory);
     
     				// Invoke factory processors registered as beans in the context.
     				invokeBeanFactoryPostProcessors(beanFactory);
     
     				// Register bean processors that intercept bean creation.
     				registerBeanPostProcessors(beanFactory);
     
     				// Initialize message source for this context.
     				initMessageSource();
     
     				// Initialize event multicaster for this context.
     				initApplicationEventMulticaster();
     
     				// Initialize other special beans in specific context subclasses.
     				onRefresh();
     
     				// Check for listener beans and register them.
     				registerListeners();
     
     				// Instantiate all remaining (non-lazy-init) singletons.
     				finishBeanFactoryInitialization(beanFactory);
     
     				// Last step: publish corresponding event.
     				finishRefresh();
     			}
     
     			catch (BeansException ex) {
     				if (logger.isWarnEnabled()) {
     					logger.warn("Exception encountered during context initialization - " +
     							"cancelling refresh attempt: " + ex);
     				}
     
     				// Destroy already created singletons to avoid dangling resources.
     				destroyBeans();
     
     				// Reset 'active' flag.
     				cancelRefresh(ex);
     
     				// Propagate exception to caller.
     				throw ex;
     			}
     
     			finally {
     				// Reset common introspection caches in Spring's core, since we
     				// might not ever need metadata for singleton beans anymore...
     				resetCommonCaches();
     			}
     		}
     	}
     ```


### AnnotationConfigApplicationContext启动

​    
