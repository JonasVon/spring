## 初探 Spring IOC

学习 `Spring` 已经很长的一段时间了，基本的开发使用是没有太大问题的，但是作为一个上进的开发者，我觉得对一门技术的理解不应该停留在如何使用这个层面上了。所以说，为了自我提高或者说更好的理解 `Spring` 的思想，阅读源码无疑是最适合不过的了。

还记得刚接触 `Spring` 的时候，不管是官网的教程还是参考书，第一个提到的重要概念就是这里要说的 `IOC`，所以，我也从认识 `IOC` 开始翻阅源码。

## 一、什么是IOC

或许你可能已经对 `IOC` 有了自己的认知，虽然这里也不是基础教程，但是我还是想明确一个`IOC`这个糟老头到底是什么。

`IOC` 全称为 `Inversion of Control` 翻译为控制反转，它还有一个别名为 `DI` ，全称为 `Dependenccy Injection` ，翻译为依赖注入。从字面上的意思来看，这两个词语好像都不太容易理解，那么咱们就一个个来解析吧。

### 1. 控制反转

控制反转，可以拆分为两个词来理解，第一，控制；第二，反转。理解它的关键在于咱们需要思考以下这三个问题：

- 谁控制谁
- 为何是反转
- 哪些方面反转呢

先抛开这些问题来看看下面的例子（程序员找女朋友）：

1.首先，上面的问题涉及到程序员和女朋友两个基本类。

2.从程序员（男）的角度来说，如今社会结婚的标配无疑是房子和车了（其实并不想举这么现实的例子，但是没想到好的），那么`Coder` 类就这么来定义：

```java
/**
 * 年轻小伙子
 */
public class YoungMan {
    private BeautifulGirl beautifulGirl;

    YoungMan(){
        // 可能你比较牛逼，指腹为婚
        // beautifulGirl = new BeautifulGirl();
    }

    public void setBeautifulGirl(BeautifulGirl beautifulGirl) {
        this.beautifulGirl = beautifulGirl;
    }

    public static void main(String[] args){
        YoungMan you = new YoungMan();
        BeautifulGirl beautifulGirl = new BeautifulGirl("你的各种条件");
        beautifulGirl.setxxx("各种投其所好");

        // 然后你有女票了
        you.setBeautifulGirl(beautifulGirl);
    }
}
```

这是咱们没有使用 `Spring` 前做事的方式，如果我们需要某个对象，一般都是采用这种直接 `new` 的方式去创建，这个过程其实是复杂和繁琐的，可能你会反驳说不会呀，不就是多调用几个`set`方法嘛。如果对于那些基本类型来说似乎好似也是，但是如果对于引用类型来说就不是这么回事了。比如说，实例化A的时候因为它依赖某类型的对象B，那你就得先去 `new B()` ，假如 B 又依赖某两个对象C、D，这两个对象又跟其他对象有依赖关系，那么你就得先从这条链的最末端开始一个个的`new`，这显然非常不爽的。然而此时 `IOC` 的能耐就体现出来了。

**IOC ，就是由 Spring IOC 容器负责对象的生命周期和对象之间的关系** —— 翻译自官网文档

在使用 `IOC` 容器后，对象之间的关系以及创建我们都不需要去关心了，只需要“告诉” `IOC` 咱们的对象分别需要什么就行了，`IOC` 容器就会自动的将对象需要的东西注入进来了。这就是控制反转。

控制，就是 `IOC` 容器控制咱们的对象；反转，就是对象的获得权反转了，以前我们需要一个对象自己动手去 `new` ，如今我们把对象的依赖关系告诉 `IOC` 容器，它来将这些对象返回给我们，从主动获得变成了被动接收。

### 2. 依赖注入

前面提到的控制反转其实是一种理念，然而实现这个理念的方式就是依赖注入。依赖注入的方式有三种：构造器注入、setter注入和接口注入。具体的使用在此就不做阐述了，具体的实现原理在后面文章再来分析，毕竟这不是一件容易的事。

## 二、俯视 Spring

### 1. Resource 体系

`Resource` ，对资源的抽象，它的每个实现类都代表了一种资源的访问策略。

![](/images/spring-201805091003.jpg)

### 2. ResourceLoader 体系

在`Spring`中，资源的定义与加载是分开的，资源的加载时由 `ResourceLoader` 来进行统一资源加载。

![](/images/spring-201805091004.png)

### 3. BeanFactory 体系

`BeanFactory` 是一个非常纯粹的容器，它是 `IOC` 必备的数据结构，其中 `BeanDefinition` 是它的基本结构，内部维护这一个 `BeanDefinition map` ，可根据 `BeanDefinition` 的描述进行对 `Bean` 的创建和管理。

![](/images/spring-201805101001.png)

### 4. BeanDefinitionReader 体系

`BeanDefinitionReader` 的作用是读取 `Spring` 的配置文件，并将其转换成 `IOC` 容器内部的数据结构。

![](/images/spring-201805101003.png)
.png)

### 5. ApplicationContext 体系

这个就是 `Spring`容器，称为应用上下文。

![](/images/spring-201805101004.png)

上面的五个体系是 `Spring IOC` 中最核心的部分，啃源码就针对这些部分去深究原理。这此先对大体有个了解，做个思想准备，迎接 `Spring` 的洗礼吧。

 

## Spring 统一资源加载策略

在前面的文章提到过 `Spring` 的五大体系，这里就先拿个软柿子来捏捏，先从比较简单的资源体系入手 —— `Resource` 体系

### 一、统一资源 Resource

`Resource` 是 `org.springframework.core.io` 包下的一个接口，它是 `Spring` 框架所有资源的抽象和访问接口，它继承 `org.springframework.core.io.InputStreamSource` 接口，但该接口中只有一个方法 `getInputStream` 。作为所有资源的统一抽象，`Resource` 定义了一些通用的方法，由子类 `AbstractResource` 提供统一的默认实现。先来吃上一口 `Resource` 的源码：

```java
public interface Resource extends InputStreamSource {
    //判断资源是否存在
    boolean exists();
	//判断资源是否可读
    default boolean isReadable() {
        return this.exists();
    }
	//判断资源句柄是否被一个stream打开了
    default boolean isOpen() {
        return false;
    }
	//判断是否为File
    default boolean isFile() {
        return false;
    }
	//返回资源的URL
    URL getURL() throws IOException;
	//返回资源的URI
    URI getURI() throws IOException;
	//返回资源的File句柄
    File getFile() throws IOException;
	//返回ReadableByteChannel
    default ReadableByteChannel readableChannel() throws IOException {
        return Channels.newChannel(this.getInputStream());
    }
	//返回资源内容的长度
    long contentLength() throws IOException;
	//返回资源上一次被修改的时间
    long lastModified() throws IOException;
	//根据资源的相对路径创建新资源
    Resource createRelative(String var1) throws IOException;
	//返回资源的文件名
    @Nullable
    String getFilename();
	//返回资源的描述
    String getDescription();
}
```

第一次打开这个接口阅读的时候真的挺兴奋的，因为这里面的绝大部分方法都是见名知意的，咱们也不难理解。再来看看相关的类结构图：

![](/images/spring-201805091003.jpg)

从上图可以看出，`AbstractResource` 为统一的默认实现类，然后根据不同类型提供不同的具体实现：

- `FileSystemResource`：对`java.io.File`类型资源的封装，只要是跟`File`打交道的，基本上与`FileSystemResource`也可以打交道。支持文件和 `URL` 的形式，实现 `WritableResource`接口，且从 `Spring Framework5.0` 开始，`FileSystemResource` 使用 `NIO2` 进行读 /写交互。
- `ByteArrayResouce`：对字节数组提供的数据的封装。如果通过 `InputStream` 形式访问该类型的资源，该实现会根据字节数组的数据构造一个相应的 `ByteArrayInputStream`。
- `UrlResource`：对 `java.net.URL` 类型资源的封装。内部委派 `URL` 进行具体的资源操作。
- `ClassPathResource`：`classpath`类型资源的实现。使用给定的 `ClassLoader` 或者给定的 `Class` 来加载资源。
- `InputStreamResource` ：将给定的 `InputStream` 作为一种资源的 `Resource` 实现类。

`AbstractResource` 作为资源的默认实现，它实现了 `Resource` 接口的大部分方法，下面来看看具体的定义：

```java
public abstract class AbstractResource implements Resource {
    //只有一个空参构造方法
    public AbstractResource() {
    }
	//判断资源是否存在，上面的五个子类中，除了UrlResource没有实现该方法，其余的子类都有自己的实现。
    public boolean exists() {
        try {
            return this.getFile().exists();
        } catch (IOException var4) {
            try {
                this.getInputStream().close();
                return true;
            } catch (Throwable var3) {
                return false;
            }
        }
    }
	//以前的版本直接返回一个 true
    public boolean isReadable() {
        return this.exists();
    }
	//直接返回false，表示未被打开
    public boolean isOpen() {
        return false;
    }
	//直接返回false,表示不为File
    public boolean isFile() {
        return false;
    }
	//抛异常，具体逻辑交给子类实现
    public URL getURL() throws IOException {
        throw new FileNotFoundException(this.getDescription() + " cannot be resolved to URL");
    }
	//构建URI
    public URI getURI() throws IOException {
        URL url = this.getURL();

        try {
            return ResourceUtils.toURI(url);
        } catch (URISyntaxException var3) {
            throw new NestedIOException("Invalid URI [" + url + "]", var3);
        }
    }
	//抛出异常，交给子类实现
    public File getFile() throws IOException {
        throw new FileNotFoundException(this.getDescription() + " cannot be resolved to absolute file path");
    }
	//根据getInputStream()的返回结果构建ReadableByteChannel
    public ReadableByteChannel readableChannel() throws IOException {
        return Channels.newChannel(this.getInputStream());
    }
	//获取资源的长度，这个资源内容长度实际就是资源的字节长度，通过全部读取一遍来判断
    public long contentLength() throws IOException {
        InputStream is = this.getInputStream();

        try {
            long size = 0L;

            int read;
            for(byte[] buf = new byte[256]; (read = is.read(buf)) != -1; size += (long)read) {
            }

            long var6 = size;
            return var6;
        } finally {
            try {
                is.close();
            } catch (IOException var14) {
            }

        }
    }
	//返回资源的最后修改时间
    public long lastModified() throws IOException {
        File fileToCheck = this.getFileForLastModifiedCheck();
        long lastModified = fileToCheck.lastModified();
        if (lastModified == 0L && !fileToCheck.exists()) {
            throw new FileNotFoundException(this.getDescription() + " cannot be resolved in the file system for checking its last-modified timestamp");
        } else {
            return lastModified;
        }
    }
	
    protected File getFileForLastModifiedCheck() throws IOException {
        return this.getFile();
    }
	//抛出异常，交给子类实现
    public Resource createRelative(String relativePath) throws IOException {
        throw new FileNotFoundException("Cannot create a relative resource for " + this.getDescription());
    }
	//获取资源名称，默认返回null
    @Nullable
    public String getFilename() {
        return null;
    }
	
    public boolean equals(Object other) {
        return this == other || other instanceof Resource && ((Resource)other).getDescription().equals(this.getDescription());
    }

    public int hashCode() {
        return this.getDescription().hashCode();
    }

    public String toString() {
        return this.getDescription();
    }
}
```

这里面的绝大部分方法只是提供一个默认实现，具体的实现还是交给子类去完成的，不同的子类有着不同的实现。如果需要自定义 `Resource` ，记住不要实现 `Resource` 接口，而是继承这个 `AbstractResource` 抽象类，然后根据具体的资源特征覆盖相应的方法即可。

### 二、统一资源定位 ResourceLoader

`Spring` 将资源的定义和资源的加载是区分开的，前面已经提到资源的定义，接下来看看资源的加载器。资源的加载由 `ResourceLoader` 来统一定义。`org.springframework.core.io.ResourceLoader` 是 `Spring` 资源加载的统一抽象，具体的资源加载则由相应的实现类完成，所以可以将 `ResourceLoader` 称作为统一资源定位器。其定义如下：

```java
public interface ResourceLoader {
    String CLASSPATH_URL_PREFIX = "classpath:";

    Resource getResource(String location);

    @Nullable
    ClassLoader getClassLoader();
}
```

`ResourceLoader` 接口提供了两个方法：`getResource()` 和 `getClassLoader()`

`getResource()` 根据所提供资源的路径 `location` 返回 `Resource` 实例，但是它不确定该 `Resource` 一定存在，需要调用 `Resource.exist()` 方法判断。该方法支持以下模式的资源加载：

- `URL` 位置资源
- `ClassPath` 位置资源
- 相对路径资源

该方法主要实现是在其子类 `DefaultResourceLoader` 中实现的，具体过程在后面再详细说明。

`getClassLoader()` 返回 `ClassLoader` 实例。

作为统一的资源加载器，它提供了统一的抽象，具体的类结构如下：

![](/images/spring-201805091004.png)

#### 1. DefaultResourceLoader

`DefaultResourceLoader` 是 `ResourceLoader` 的默认实现，它接收 `ClassLoader` 作为构造函数的参数或者也可以使用空参的构造函数，如果使用空参的构造函数，使用的 `ClassLoader` 为默认的 `Thread.currentThread().getContextClassLoader()`，可以通过 `ClassUtils.getDefaultClassLoader()` 获取。

```java
public DefaultResourceLoader() {
    this.classLoader = ClassUtils.getDefaultClassLoader();
}

public DefaultResourceLoader(@Nullable ClassLoader classLoader) {
    this.classLoader = classLoader;
}

@Nullable
    public ClassLoader getClassLoader() {
        return this.classLoader != null ? this.classLoader : ClassUtils.getDefaultClassLoader();
    }
```

`ResourceLoader` 中最核心的方法是 `getResource()` ，它根据提供的 `location` 返回相应的 `Resource`，而 `DefaultResourceLoader` 对该方法提供了核心实现：

```java
public Resource getResource(String location) {
        Assert.notNull(location, "Location must not be null");
        Iterator var2 = this.protocolResolvers.iterator();

        Resource resource;
        do {
            if (!var2.hasNext()) {
                if (location.startsWith("/")) {
                    return this.getResourceByPath(location);
                }

                if (location.startsWith("classpath:")) {
                    return new ClassPathResource(location.substring("classpath:".length()), this.getClassLoader());
                }

                try {
                    URL url = new URL(location);
                    return (Resource)(ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
                } catch (MalformedURLException var5) {
                    return this.getResourceByPath(location);
                }
            }

            ProtocolResolver protocolResolver = (ProtocolResolver)var2.next();
            resource = protocolResolver.resolve(location, this);
        } while(resource == null);

        return resource;
    }
```

`protocolResolvers` 是该类的一个常量：

```java
private final Set<ProtocolResolver> protocolResolvers = new LinkedHashSet(4);
```

在该类中还有一个专门用于操作该常量的方法 `addProtocolResolver()`：

```java
public void addProtocolResolver(ProtocolResolver resolver) {
    Assert.notNull(resolver, "ProtocolResolver must not be null");
    this.protocolResolvers.add(resolver);
}
```

显而易见就是一个往集合添加元素的操作。回到上面的 `getResource()`中，一来先整出一个遍历器，遍历 `protocolResolvers` ：

- 如果 `location` 以 / 开头，则调用 `getResourceByPath()` 构造 `ClassPathResource`类型资源并返回。
- 如果 `locatoin` 以 `classpath:` 开头，则构造 `ClassPathResource` 类型资源并返回。
- 构造 `URL`，尝试通过它进行资源定位，若没有异常，则判断是否为 `FileURL`，如果是则构造 `FileUrlResource` 类型资源，否则构造 `UrlResource` 。如果出现异常，则委派`getResourceByPath()` 实现资源定位加载。

#### 2. FileSystemResourceLoader

在 `FileSystemResourceLoader` 类中，重写了 `getResourceByPath()` ：

```jav
@Override
protected Resource getResourceByPath(String path) {
	if (path.startsWith("/")) {
		path = path.substring(1);
	}
	return new FileSystemContextResource(path);
}
private static class FileSystemContextResource extends FileSystemResource implements ContextResource {

    public FileSystemContextResource(String path) {
    	super(path);
    }

    @Override
    public String getPathWithinContext() {
    	return getPath();
    }
}	
```

在构造器中也是调用了 `FileSystemResource` 的构造方法来构造 `FileSystemContextResource` 的。

#### 3. ResourcePatternResolver

`ResourceLoader` 的 `Resource getResource(String location)` 每次只能根据一个 `location` 返回一个 `Resource` ，当需要加载多个资源时，我们除了多次调用 `getResource()`外别无选择。 `ResourcePatternResolver` 是 `ResourceLoader` 的扩展，它支持根据指定资源路径匹配模式每次返回多个 `Resource` 。它是一个接口，定义如下：

```java
public interface ResourcePatternResolver extends ResourceLoader {
    String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
    
    Resource[] getResources(String locationPattern) throws IOException;
}
```

`ResourcePatternResolver` 在 `ResourceLoader` 的基础上增加了 `getResources(String locationPattern)` ，支持根据路径匹配模式返回多个 `Resource`实例，同时新增了一种协议前缀 `classpath*:` ，该协议前缀由具体的子类实现。

`PathMatchingResourcePatternResolver` 是 `ResourcePatternResolver` 最常用的子类，它除了支持 `ResourceLoader` 和 `ResourcePatternResolver` 新增的 `classpath*:` 前缀外，还支持 `Ant` 风格的路径匹配模式。先来看看构造方法：

```java
//空参构造器
public PathMatchingResourcePatternResolver() {
    this.resourceLoader = new DefaultResourceLoader();
}
//根据ResourceLoader实例化
public PathMatchingResourcePatternResolver(ResourceLoader resourceLoader) {
    Assert.notNull(resourceLoader, "ResourceLoader must not be null");
    this.resourceLoader = resourceLoader;
}
//根据ClassLoader实例化，如果不指定，就用默认的DefaultResourceLoader
public PathMatchingResourcePatternResolver(@Nullable ClassLoader classLoader) {
    this.resourceLoader = new DefaultResourceLoader(classLoader);
}
```

两个重要的方法：

```java
@Override
public Resource getResource(String location) {
    return getResourceLoader().getResource(location);
}

@Override
public Resource[] getResources(String locationPattern) throws IOException {
    Assert.notNull(locationPattern, "Location pattern must not be null");
    //以classpath*:开头
    if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
        // 路径包含通配符
        if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
            // a class path resource pattern
            return findPathMatchingResources(locationPattern);
        }
        else {
            // 不包含通配符
            return findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
        }
    }
    else {
        // Generally only look for a pattern after a prefix here,
        // and on Tomcat only after the "*/" separator for its "war:" protocol.
        int prefixEnd = (locationPattern.startsWith("war:") ? locationPattern.indexOf("*/") + 1 :
                         locationPattern.indexOf(':') + 1);
        if (getPathMatcher().isPattern(locationPattern.substring(prefixEnd))) {
            // a file pattern
            return findPathMatchingResources(locationPattern);
        }
        else {
            // a single resource with the given name
            return new Resource[] {getResourceLoader().getResource(locationPattern)};
        }
    }
}
```

`getResource()` 方法直接委托给相应的 `ResourceLoader` 来实现，所以如果我们在实例化的 `PathMatchingResourcePatternResolver` 的时候，如果没有指定 `ResourceLoader`，那么在加载资源时，其实就是 `DefaultResourceLoader` 的过程。`getResources(String locationPattern)` 也相同，只不过返回多个资源而已。处理逻辑图：

![](/images/spring-87106504.jpg)

由上图可见，当 `locationPattern` 以 `classpath*:` 开头但是不包含通配符，则调用 `findAllClassPathResource()` 方法加载资源。该方法返回 `classes` 路径下和所有 `jar` 包中的所有相匹配的资源。

```java
protected Resource[] findAllClassPathResources(String location) throws IOException {
    String path = location;
    if (path.startsWith("/")) {
        path = path.substring(1);
    }
    //执行加载
    Set<Resource> result = doFindAllClassPathResources(path);
    if (logger.isTraceEnabled()) {
        logger.trace("Resolved classpath location [" + location + "] to resources " + result);
    }
    return result.toArray(new Resource[0]);
}
```

真正执行加载的是 `doFindAllClassPathResources()` 方法：

```jav
protected Set<Resource> doFindAllClassPathResources(String path) throws IOException {
    Set<Resource> result = new LinkedHashSet<>(16);
    ClassLoader cl = getClassLoader();
    Enumeration<URL> resourceUrls = (cl != null ? cl.getResources(path) : ClassLoader.getSystemResources(path));
    while (resourceUrls.hasMoreElements()) {
        URL url = resourceUrls.nextElement();
        result.add(convertClassLoaderURL(url));
    }
    if ("".equals(path)) {
    	addAllClassLoaderJarRoots(cl, result);
    }
    return result;
}
```

`doFindAllClassPathResource()` 根据 `ClassLoader` 加载路径下的所有资源。在加载资源过程中，如果在构造 `PathMatchingResourcePatternResolver` 实例的时候如果传入了 `ClassLoader` ，则调用其 `getResource()` ，否则调用 `ClassLoader.getSystemResource(path)` 。

#### 4. 总结

- `Spring` 提供了 `Resource` 和 `ResourceLoader` 来统一抽象整个资源及其定位。使得资源与资源的定位有了一个更加清晰的界限，并且提供了合适的 Default 类，使得自定义实现更加方便和清晰。
- `DefaultResource` 为 `Resource` 的默认实现，它对 `Resource` 接口做了一个统一的实现，子类继承该类后只需要覆盖相应的方法即可，同时对于自定义的 `Resource` 我们也是继承该类。
- `DefaultResourceLoader` 同样也是 `ResourceLoader` 的默认实现，在自定 `ResourceLoader` 的时候我们除了可以继承该类外还可以实现 ProtocolResolver 接口来实现自定资源加载协议。
- `DefaultResourceLoader` 每次只能返回单一的资源，所以 `Spring` 针对这个提供了另外一个接口 `ResourcePatternResolver` ，该接口提供了根据指定的 `locationPattern` 返回多个资源的策略。其子类 `PathMatchingResourcePatternResolver` 是一个集大成者的 `ResourceLoader` ，因为它即实现了 `Resource getResource(String location)` 也实现了 `Resource[] getResources(String locationPattern)`。

