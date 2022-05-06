##### Spring,Spring MVC, Spring Boot
什么是Spring, Spring 就是一个大的框架。框架就是别人写好的函数，因此需要使用别人约定俗成的规则来写代码。

当初写这套框架的人是为了实现他们当时的业务需求，随着业务需求增多模块增多，而模块也就是别人为了后续开发的需求定制化的，比如spring mvc, spring cloud

用别人写好的模块使用，是需要写一些配置的，配置一般是以xml的格式在标签中解析，比如数据库的配置里会有用户密码等参数，如果你要在你的服务器启动，则必须修改自己的用户密码名

每个项目的配置不同，example配置如下:
```
spring.datasource.username=root
spring.datasource.pssword=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

```
##### Spring Boot
因为写配置很麻烦，写模块也很麻烦，因此有人把这些模块配置组装好了，对于新手很不友好很不方便，因此有了*spring boot*来方便化调用配置

比如我们调用*freemarker*插件，就可以直接调用别人写好的*spring boot freemarker-starter*。 除此之外还有很多starter，比如mongodb，redis等等，可以减少工作量。

> 写配置是学习框架，和写代码不同，因此需要学习的是框架知识

常用的配置:
- *freemarker*:用于渲染模板引擎，如果没有则是原始的*xml*配置
- *srping boot starter web* :将spring和spring mvc打包的web框架
- *mysql-connector-java*: 用于mysql连接数据库的
- *srping boot mybatis*: 用于简化crud的
- *tomcat*:所做的事情就是接受请求，解析请求，从请求拿取对应的参数后根据请求头(get or post)把数据丢给spring框架，再由spring框架决定应该传给什么router

##### Spring Boot 入口
入口是*@SpringBootApplication*，写法如下:

```  
@SpringBootApplication(scanBasePackages = {"org.linlinjava.litemall"})  

public class Application extends SpringBootServletInitializer {  
  
    public static void main(String[] args) throws Exception {  
        SpringApplication.run(Application.class, args);  
    }  

}
```

##### controller
> 用于返回页面, 需要通过@controller来标明，这种方式被称为注解

加入@controller后，程序会自动增加public Controller类，用于标明此类是controller类 。SpringBoot运行的时候，会扫描所有controller函数并且根据路径调用具体controller的函数，比如路径为"/admin"就会调用路由为"/admin"的函数

```
@RestController  
@RequestMapping("/admin")  
@Validated  
public class AdminAdminController {
  ...
}
```

> In Spring Boot, the _controller class_ is responsible for processing incoming REST API requests, preparing a model, and returning the view to be rendered as a response.

可见controller用于解析请求，返回渲染后的*response*(响应，一个controller可以对应多个路由，也就是对应多个路径。

根据请求会有post,get请求，因此会有*GetMapping*,*PostMapping*两套注解

##### ModelAndView
这是一个模板，可以传递参数，返回后渲染成我们所需要的页面，这是最简单的生成页面函数的*router*函数

这里的Model就是mvc里的model概念,我们用的*freemarker*渲染的模板为ftl格式，我们需要返回的ftl如下
```
ModelAndView mv = new ModelAndView("index");
mv.addObject("username", "guoba");
```

```
<!DOCTYPE html>
<body>
  <h1>欢迎${username}</h1>
</body>
</html>
```


##### RequestParm
get请求获取参数方式是在?后通过key,value对添加的,示例如下：

```
localhost:9000/demo?username=guoba
```
可以通过RequestParm提取到对应的参数，这里直接把username=的参数提取到了，并且渲染回一个网页

- ![avatar](https://images.weserv.nl/?url=https://article.biliimg.com/bfs/article/7df4ff199f6d41074b60d7fda359c4574d0a200f.png)

```
@GetMapping("/demo")  
public ModelAndView demoRoute(  
        @RequestParam(name="username", required=false, defaultValue="guoba") String username) {  
    ModelAndView mv = new ModelAndView("guoba");  
    mv.addObject("username", username);  
    return mv;  
}
```

还可以通过*HTTPServerletRequest*直接获取,这是另一种写法
```
String name = request.getParameter("username");
```

还有一种动态路由的方式截取参数，用法如下:


```
@GetMapping("/demo3/{username}")  
// 从 HttpServletRequest 里面取参数  
public ModelAndView demoRoute2(@PathVariable String username) {  
    ModelAndView mv = new ModelAndView("hello");  
    mv.addObject("username", username);  
    return mv;  
}
```

![avatar](https://images.weserv.nl/?url=https://article.biliimg.com/bfs/article/78f18de421f81ced8980ac3385ac054ecae1b43a.png)
demo3没有使用key=value方式，而是直接提取{username}即可

