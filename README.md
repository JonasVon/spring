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

![](C:\Users\jonas\Desktop\spring-201805091003.jpg)

### 2. ResourceLoader 体系

在`Spring`中，资源的定义与加载是分开的，资源的加载时由 `ResourceLoader` 来进行统一资源加载。

![](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1555925618977.png)

### 3. BeanFactory 体系

`BeanFactory` 是一个非常纯粹的容器，它是 `IOC` 必备的数据结构，其中 `BeanDefinition` 是它的基本结构，内部维护这一个 `BeanDefinition map` ，可根据 `BeanDefinition` 的描述进行对 `Bean` 的创建和管理。

![](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1555925845977.png)

### 4. BeanDefinitionReader 体系

`BeanDefinitionReader` 的作用是读取 `Spring` 的配置文件，并将其转换成 `IOC` 容器内部的数据结构。

![](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1555925969406.png)

### 5. ApplicationContext 体系

这个就是 `Spring`容器，称为应用上下文。

![](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1555926064339.png)

上面的五个体系是 `Spring IOC` 中最核心的部分，啃源码就针对这些部分去深究原理。这此先对大体有个了解，做个思想准备，迎接 `Spring` 的洗礼吧。
