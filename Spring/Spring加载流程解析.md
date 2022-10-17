# Spring加载流程解析

## 项目地址

​    **git@gitee.com:fouyang/spring.git**

## 前置准备

1. 导入依赖

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
```

 上面这个spring-context包会依赖导入下面几个包

```java
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

```java
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
      
    <bean class="org.example.Student" id="student"/>
</beans>
```
3. 进入Spring入口类

```java
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

****

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

#### 核心方法refresh()分析

##### obtainFreshBeanFactory()方法

```java
 	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		return getBeanFactory();
	}
  protected abstract void refreshBeanFactory() throws BeansException, IllegalStateException;

	@Override
	public abstract ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;

```

  这里是一个模板方法设计模式，refreshBeanFactory()和getBeanFactory()在子类AbstractRefreshableApplicationContext实现

```
	@Override
	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
			loadBeanDefinitions(beanFactory);
			this.beanFactory = beanFactory;
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
```

 这个方法会在createBeanFactory()中new出一个DefaultListableBeanFactory对象，最后把这个BeanFactory对象赋给成员变量，这个方法其核心是loadBeanDefinitions(beanFactory)

```java
	@Override
	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// Create a new XmlBeanDefinitionReader for the given BeanFactory.
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// Configure the bean definition reader with this context's
		// resource loading environment.
		beanDefinitionReader.setEnvironment(this.getEnvironment());
    // 这里把ApplicationContext对象传进去
		beanDefinitionReader.setResourceLoader(this);
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		// Allow a subclass to provide custom initialization of the reader,
		// then proceed with actually loading the bean definitions.
		initBeanDefinitionReader(beanDefinitionReader);
		loadBeanDefinitions(beanDefinitionReader);
	}
```

这个方法中会创建一个XmlBeanDefinitionReader对象来读取XML文件配置的bean来封装为BeanDefinition对象，其核心是loadBeanDefinitions(beanDefinitionReader)方法

```
	protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
		Resource[] configResources = getConfigResources();
		if (configResources != null) {
			reader.loadBeanDefinitions(configResources);
		}
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			reader.loadBeanDefinitions(configLocations);
		}
	}
```

这里的getConfigResources()获取的Resource是在new ClassPathXmlApplicationContext()时作为构造函数参数传入赋值给ClassPathXmlApplicationContext类的成员变量private Resource[] configResources，这里我们new ClassPathXmlApplicationContext()时传入的是一个字符串"spring.xml"赋值给了AbstractRefreshableConfigApplicationContext类的成员变量private String[] configLocations，所以这里进入的是reader.loadBeanDefinitions(configLocations)方法

```
	@Override
	public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
		Assert.notNull(locations, "Location array must not be null");
		int count = 0;
		for (String location : locations) {
			count += loadBeanDefinitions(location);
		}
		return count;
	}
```

这个方法进入循环加载所有配置文件，主要在loadBeanDefinitions(location)完成

```
	@Override
	public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(location, null);
	}
	
	public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					"Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");
		}

		if (resourceLoader instanceof ResourcePatternResolver) {
			// Resource pattern matching available.
			try {
				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
				int count = loadBeanDefinitions(resources);
				if (actualResources != null) {
					Collections.addAll(actualResources, resources);
				}
				if (logger.isTraceEnabled()) {
					logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");
				}
				return count;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"Could not resolve bean definition resource pattern [" + location + "]", ex);
			}
		}
		else {
			// Can only load single resources by absolute URL.
			Resource resource = resourceLoader.getResource(location);
			int count = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			if (logger.isTraceEnabled()) {
				logger.trace("Loaded " + count + " bean definitions from location [" + location + "]");
			}
			return count;
		}
	}
	
```

进入这里后获取到的ResourceLoader是我们第一步new的ClassPathXmlApplicationContext对象，通过类继承关系可以明确ClassPathXmlApplicationContext实现了ResourcePatternResolver接口，所有这里进入if语句块，解析配置文件为Resource对象数组，进入loadBeanDefinitions(resources)方法

```
	@Override
	public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
		Assert.notNull(resources, "Resource array must not be null");
		int count = 0;
		for (Resource resource : resources) {
			count += loadBeanDefinitions(resource);
		}
		return count;
	}
```

然后进入loadBeanDefinitions(resource)方法

```
	@Override
	public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(new EncodedResource(resource));
	}
```

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isTraceEnabled()) {
			logger.trace("Loading XML bean definitions from " + encodedResource);
		}

		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();

		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
     // 最重要的方法块
		try (InputStream inputStream = encodedResource.getResource().getInputStream()) {
			InputSource inputSource = new InputSource(inputStream);
			if (encodedResource.getEncoding() != null) {
				inputSource.setEncoding(encodedResource.getEncoding());
			}
			return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}
```

在上面的方法中，会获取Resource的输入流构造InputSource,进入doLoadBeanDefinitions(inputSource, encodedResource.getResource())

```
	protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {

		try {
			Document doc = doLoadDocument(inputSource, resource);
			int count = registerBeanDefinitions(doc, resource);
			if (logger.isDebugEnabled()) {
				logger.debug("Loaded " + count + " bean definitions from " + resource);
			}
			return count;
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
```

在这里会进入SAX解析对配置文件进行解析

```
	public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		int countBefore = getRegistry().getBeanDefinitionCount();
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```

这里创建一个BeanDefinitionDocumentReader进行具体的解析工作，在documentReader.registerBeanDefinitions(doc, createReaderContext(resource));完成

```
	@Override
	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
		doRegisterBeanDefinitions(doc.getDocumentElement());
	}
	
	protected void doRegisterBeanDefinitions(Element root) {
		// Any nested <beans> elements will cause recursion in this method. In
		// order to propagate and preserve <beans> default-* attributes correctly,
		// keep track of the current (parent) delegate, which may be null. Create
		// the new (child) delegate with a reference to the parent for fallback purposes,
		// then ultimately reset this.delegate back to its original (parent) reference.
		// this behavior emulates a stack of delegates without actually necessitating one.
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);

		if (this.delegate.isDefaultNamespace(root)) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				// We cannot use Profiles.of(...) since profile expressions are not supported
				// in XML config. See SPR-12458 for details.
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					if (logger.isDebugEnabled()) {
						logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
								"] not matching: " + getReaderContext().getResource());
					}
					return;
				}
			}
		}

		preProcessXml(root);
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root);

		this.delegate = parent;
	}
```

在15行会创建一个BeanDefinitionParserDelegate来解析XML文件，delegate.isDefaultNamespace(root)会判断根node的nameSpaceURI是不是BEANS_NAMESPACE_URI = "http://www.springframework.org/schema/beans",这里我们的配置文件中根节点没有profile属性，在第二个if退出，进入parseBeanDefinitions(root, this.delegate)

```
	protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
						parseDefaultElement(ele, delegate);
					}
					else {
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
```

这里会逐个解析XML配置文件的元素，在默认的nameSpaceURI("http://www.springframework.org/schema/beans")下的<bean>标签、<import>标签、<alias>标签、<beans>标签声明的元素会通过parseDefaultElement(ele, delegate)方法解析，其它例如\<aop:aspectj\>或\<context:compontent-scan\>标签声明的delegate.parseCustomElement(ele)方法解析

> 这里为了后面的解析说明，先列一个BeanDefinition的定义
>
> ![BeanDifiniton定义](https://tva1.sinaimg.cn/large/008vxvgGly1h78orpti3hj316j0u0dmu.jpg)

  

```
	private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
		// 这个是<bean>标签的解析 非常重要
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}
```

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
   // 解析<bean>标签元素的代码，非常重要
   BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
   if (bdHolder != null) {
      // 这行代码可以不看 不太重要
      bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
      try {
         // Register the final decorated instance.
         BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error("Failed to register bean definition with name '" +
               bdHolder.getBeanName() + "'", ele, ex);
      }
      // Send registration event.
      getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
   }
}
```

delegate.parseBeanDefinitionElement(ele)会解析xml文件中的bean标签封装为BeanDefinition对象放到BeanDefinitionHolder中，由BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());注册到beanFactory

```
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean) {
		String id = ele.getAttribute(ID_ATTRIBUTE);
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
		List<String> aliases = new ArrayList<>();
		if (StringUtils.hasLength(nameAttr)) {
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList(nameArr));
		}
		String beanName = id;
		if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
			beanName = aliases.remove(0);
			if (logger.isTraceEnabled()) {
				logger.trace("No XML 'id' specified - using '" + beanName +
						"' as bean name and " + aliases + " as aliases");
			}
		}
		if (containingBean == null) {
			checkNameUniqueness(beanName, aliases, ele);
		}

		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
		if (beanDefinition != null) {
			if (!StringUtils.hasText(beanName)) {
				try {
					if (containingBean != null) {
						beanName = BeanDefinitionReaderUtils.generateBeanName(
								beanDefinition, this.readerContext.getRegistry(), true);
					}
					else {
						beanName = this.readerContext.generateBeanName(beanDefinition);
						// Register an alias for the plain bean class name, if still possible,
						// if the generator returned the class name plus a suffix.
						// This is expected for Spring 1.2/2.0 backwards compatibility.
						String beanClassName = beanDefinition.getBeanClassName();
						if (beanClassName != null &&
								beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
								!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
							aliases.add(beanClassName);
						}
					}
					if (logger.isTraceEnabled()) {
						logger.trace("Neither XML 'id' nor 'name' specified - " +
								"using generated bean name [" + beanName + "]");
					}
				}
				catch (Exception ex) {
					error(ex.getMessage(), ele);
					return null;
				}
			}
			String[] aliasesArray = StringUtils.toStringArray(aliases);
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
	}
```

这个方法就是通过标签的各种属性封装为beanDefinition

```java
@Nullable
	public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
		String namespaceUri = getNamespaceURI(ele);
		if (namespaceUri == null) {
			return null;
		}
		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
		if (handler == null) {
			error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
			return null;
		}
		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
	}
```

自定义标签的解析会不太一样，readerContext.getNamespaceHandlerResolver()会返回

### AnnotationConfigApplicationContext启动

​    
