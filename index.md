@[TOC]
# 源码版本
- 2.3.x
# new Application
- 首先我们的SpringBoot的启动都是通过SpringApplication的静态方法run开始的，所以我们从run开始看起
![在这里插入图片描述](https://img-blog.csdnimg.cn/a0ec42a2685e414cb7d42e7b16e09277.png)
- 进入run方法发现里边调用了类内部的一个run方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/fca504cde0ba45cd9130fed705ca7ac8.png)
- 再进去以后发现new了一个SpringApplication然后调用了对象的run
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2c9d1381cf0459087e54f0109558299.png)
- 先看SpringApplication的构造方法做了什么事情
- 首先primarySources指向了primarySources也就是我们的启动类
- 然后webApplicationType获取启动类型
- 将getSpringFactoriesInstances(ApplicationContextInitializer.class)和getSpringFactoriesInstances(ApplicationListener.class)设置到自己的属性里
- 再将mainApplicationClass指向启动类
- 我们重点看getSpringFactoriesInstances()方法，两个语句执行了同一个方法，所以我们只需要看其中一个就可以，然后执行完以后将返回值设置到了initializers和listeners中
![在这里插入图片描述](https://img-blog.csdnimg.cn/a60ef49cf87a49e88f99b2a1cdbe7c8b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_15,color_FFFFFF,t_70,g_se,x_16)
## getSpringFactoriesInstances()
- 进来以后发现主要有两个方法SpringFactoriesLoader.loadFactoryNames()和createSpringFactoriesInstances()，我们分开看
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c4bcb1ed5c7419ea35d26c48863066a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_16,color_FFFFFF,t_70,g_se,x_16)
### SpringFactoriesLoader.loadFactoryNames()
- debug可以看到第一次进来cache是没有缓存的，所以会继续向下执行
- 然后classLoader是不为空的，所以URL地址是META-INF/spring.factories，在while循环中就是从这个URL加载配置进去的类最后封装到result这个map中然后再放入cache
- 刚才两个方法的参数分别是ApplicationContextInitializer.class和ApplicationListener.class，所以就是从META-INF/spring/factories中加载这两个接口的实现，也就是为什么实现这两个接口必须配置在META-INF/spring/factories中才会被SpringBoot所加载
- ApplicationListener接口虽然在这里会以加载文件的方式加载进来，但如果不在文件中配置的话这个类按理说是不会生效的，但是加上@Component注解这个类还是生效了，这块暂时还不太明白
![在这里插入图片描述](https://img-blog.csdnimg.cn/80d7d73a961d41008381ec16d3e4a12f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_18,color_FFFFFF,t_70,g_se,x_16)
### createSpringFactoriesInstances()
- 这个方法主要是通过反射实例化需要加载的类然后返回
- 然后我们知道了这个new SpringApplication做了什么事情以后回到实例完以后调用的run方法中去
![在这里插入图片描述](https://img-blog.csdnimg.cn/1946f059fa3a49a19f519179e2d9282a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_10,color_FFFFFF,t_70,g_se,x_16)


# run
- 这个方法是最重要的方法
- 实例化时间类计算启动时间
- new StopWatch()
- 获取监听器并在有事件时进行调用
- getRunListeners()
- 实例化参数类
- new DefaultAppicationArguments()
- 准备环境
- prepareEnvironment()
- 打印标志
- printBanner()
- 创建容器
- createApplicationContext()
- 准备工作
- prepareContext()
- 启动容器
- refreshContext()
- 启动之后
- afterRefresh()
- 调用扩展点
- callRunners()
![在这里插入图片描述](https://img-blog.csdnimg.cn/85b0ae0e0ec04cafa6796a0a699da17e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_14,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/719d9197617c4fe88eee11d2a0d5b823.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_15,color_FFFFFF,t_70,g_se,x_16)
## getRunListeners()
- 点进来这个方法发现主要是new了一个SpringApplicationRunListeners的对象，然后通过文件加载所有的SpringApplicationRunListener作为参数传入构造方法，所以SpringApplicationRunListener接口也是只能通过文件方式才可以被加载
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c5a5d43ab5d4e7eb11347fc973cad56.png)
- 进来以后发现这个类有一个List\<SpringApplicationRunListener>类型的参数，然后把所有的监听器放入到这个集合中
- 所以这里使用了监听器模式，被监听的对象改变然后监视者通知所有的监听者
![在这里插入图片描述](https://img-blog.csdnimg.cn/0a9851b02f2f437988608631233e11a0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_15,color_FFFFFF,t_70,g_se,x_16)
- 然后调用了监听器的starting()方法
- 就是通知所有的监听者
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a35c97f37ac4343bda8527daad0b376.png)
## new DefaultApplicationArguments()
- 就是创建出这个对象并把我们启动时传的参数封装起来
![在这里插入图片描述](https://img-blog.csdnimg.cn/cd1b475809c14871b025d545c2d98227.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_11,color_FFFFFF,t_70,g_se,x_16)
## prepareEnvironment()
- 这一步是准备环境
- 首先是创建环境类
- 然后是配置环境
- 调用监听器的environmentPrepared()方法
- 绑定环境到应用
- 环境这块没有深入研究就不说了
![在这里插入图片描述](https://img-blog.csdnimg.cn/c56ee860afb7494f8da5c2740bff5e0d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_16,color_FFFFFF,t_70,g_se,x_16)
## createApplicationContext()
- 这一步是创建容器，模板模式，根据不同的状态创建不同的容器
![在这里插入图片描述](https://img-blog.csdnimg.cn/a0872fc8a3434e189b91fcb97463711f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_16,color_FFFFFF,t_70,g_se,x_16)
## prepareContext
- 这一步是准备容器
- 设置环境context.setEnvironment()
- 调用ApplicationContextInitializer的initializer方法
- 调用监听器的contextPrepared()
- 初始化工厂
- 把参数类交给IOC管理
- 加载启动类到容器
- 调用监听器的ContextLoaded()方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/6823886b83dc4d0ea42c91964409210b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_17,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4934fa34a2d64e1aa6912aeac2326de8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_11,color_FFFFFF,t_70,g_se,x_16)
### applyInitializers()
- 这一步调用所有实现了ApplicationContextInitializer的initialize()方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/414474147c1e4deaa7eaee4b2d988f8b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_14,color_FFFFFF,t_70,g_se,x_16)
## refreshContext()
- 准备工作做完以后就要开始真正启动容器了
- 注册shutdownHook任务
- 调用refresh方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/bf12f3fb64e04cc89ecf375d5d6cac45.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_10,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/70cf8f43bc304c0c82b03cc8033c5476.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/afe43143bffe49f1bf3132f7c613d10f.png)
- 我们是web类型所以调用的是ServletWebServerApplicationContext的方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/2c807a44e449487db25cc9b43b766a41.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_11,color_FFFFFF,t_70,g_se,x_16)
- 调用了父类的refresh方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/baa141e12cd54e50a4dd55ef320fd654.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_11,color_FFFFFF,t_70,g_se,x_16)
- 这一步看过Spring源码的应该知道这是Spring的启动过程，这里就不多说
![在这里插入图片描述](https://img-blog.csdnimg.cn/550b556a8e434b4b9b3a4063c2a12701.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_11,color_FFFFFF,t_70,g_se,x_16)
- 重点看一下onRefresh()方法
- 这个类调用了父类的onRefresh方法后启动了Web容器
- 我们这里启动的是内置Tomcat
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a61a82aab884b958869c9c5617909a7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_12,color_FFFFFF,t_70,g_se,x_16)
### createWebServer()
- 在这一步获取web容器工厂然后根据状态创建实例
- 这也就是为什么SpringBoot能独立运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f425ff43ddc42abac71fa043d84fae6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_14,color_FFFFFF,t_70,g_se,x_16)
- 首先通过getWebServerFactory()获取web容器工厂，实现类分别有Jetty，Tomcat等
![在这里插入图片描述](https://img-blog.csdnimg.cn/833d1cd8bf5e4f22b3a537223aeb2ee8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_6,color_FFFFFF,t_70,g_se,x_16)
- 这里我们启动的是Tomcat，所以直接看TomcatServletWebServerFactory的源码
- 实例化Tomcat然后配置Connector，这里是NIO方式运行，然后在配置其他的一些属性等，然后通过getTomcatWebServer(tomcat)将其封装为一个WebServer实现
![在这里插入图片描述](https://img-blog.csdnimg.cn/44104d22ee14470db249490024f4b16b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_10,color_FFFFFF,t_70,g_se,x_16)
## afterRefresh()
- 这个方法在类中没有具体实现，应该是SpringBoot提供的一个扩展点
## callRunners()
- 对实现了ApplicationRunner和CommandLineRunner的类进行方法调用
- 后边就是对监听器的一些方法调用等
![在这里插入图片描述](https://img-blog.csdnimg.cn/39b409d1379f483dbfad066f19a52526.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aiD5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI5ZOI,size_12,color_FFFFFF,t_70,g_se,x_16)
# 总结
- SpringBoot的启动流程就是首先通过加载META-INF/spring.factories文件获取给定扩展出来的点进行加载，然后获取配置，准备启动容器，对Spring进行了封装，其实启动的还是Spring的容器然后将注解等方式注入的Bean通过处理器等方法加载到Spring中，然后通过内置启动Tomcat来进行独立运行并可以进行Web服务，最后在SpringBoot的扩展点等进行一下调用。
- 总的来说SpringBoot约定大于配置，独立运行，自动装配等功能让我们使用起来更加方便，让我们可以更好的专注业务而不是框架。
- 如果有不对的地方还望指出，以上只是个人理解
