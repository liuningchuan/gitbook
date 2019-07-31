---
title: "Swagger"
catalog: true
date: 2019-03-23 10:51:24
header-img: "/images/archive-bg.png"
tags:
- Java
- Spring
catagories:
- Hexo
---
Swagger是一个流行的API开发框架，这个框架以“开放API声明”（OpenAPI Specification，OAS）为基础，对整个API的开发周期都提供了相应的解决方案，是一个非常庞大的项目（包括设计、编码和测试，几乎支持所有语言）。

OAS本身是一个API规范，它用于描述一整套API接口，包括一个接口是GET还是POST请求啊，有哪些参数哪些header啊，都会被包括在这个文件中。它在设计的时候通常是YAML格式，这种格式书写起来比较方便，而在网络中传输时又会以json形式居多，因为json的通用性比较强。

由于Spring的流行，Marty Pitt编写了一个基于Spring的组件swagger-springmvc，用于将swagger集成到springmvc中来。而springfox则是从这个组件发展而来，同时springfox也是一个新的项目，本文仍然是使用其中的一个组件springfox-swagger2。

pringfox-swagger2依然是依赖OSA规范文档，也就是一个描述API的json文件，而这个组件的功能就是帮助我们自动生成这个json文件，我们会用到的另外一个组件springfox-swagger-ui就是将这个json文件解析出来，用一种更友好的方式呈现出来。

在开始编码之前，我们先对配置的流程有个大致的了解。

在前言中，我们知道，我们的第一个任务就是生成一个满足OSA规范的json文件（当然，创建一个spring的项目就不说了）。对于这个任务，springfox为我们提供了一个Docket（摘要的意思）类，我们需要把它做成一个Bean注入到spring中，显然，我们需要一个配置文件，并通过一种方式（显然它会是一个注解）告诉程序，这是一个Swagger配置文件。

一个OSA规范文档需要许多信息来描述这个API，springfox允许我们将信息组合成一个ApiInfo的类，作为构造参数传给Docket（当然也可以不构造这个类，而直接使用null，但是你的这个API就太low了）。

接下来，我们要写控制器了，当然这不重要，不用springfox你依然要写控制器，重要的是要告诉springfox，这个控制器是一个需要他来收集API信息的控制器，不用说，这依然会采用注解的方式，同时，我们为了将配置文件与控制器结合起来，需要在配置文件中指明在什么位置收集可能是API的控制器的信息。

到这里，生成OSA规范的json文件的配置就结束了。虽然生成过程比我叙述的更复杂，但这些程序都会帮我们完成，我们可以通过类似http://localhost:8080/demo/v2/api-docs的路径来查看这个json文件。这个v2/api-docs就是springfox默认的生成文档的路径。

接下来，我们需要将它可视化显示出来，如果使用swagger-springmvc，我们需要单独去下载一个swagger ui的显示页面包，并将其中的路径改为上面的http://localhost:8080/demo/v2/api-docs，这里你就可以感受到，swagger ui就是在解析一个json文件了。你依然可以这么做，不过springfox专门提供了一个springfox-swagger-ui组件，不需要配置，我们只需要引入这个依赖的组件就可以看到最终的效果了，而这个路由会是http://localhost:8080/demo/swagger-ui.html。

#### Maven依赖配置

```xml
<!-- swaggr2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.1</version>
</dependency>
<!-- swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.1</version>
</dependency>
```

#### Configuration类配置

```java
@Configuration
@EnableSwagger2
@EnableWebMvc
public class SwaggerConfig {

    @Bean
    public Docket customDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage(
                        "com.liuning.springswagger.controller"))
                .build();
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact(
                "LiuNing",
                "https://springfox.github.io/springfox/",
                "liuning@cqfmbank.com"
        );

        return new ApiInfoBuilder()
                .title("开放平台接口")
                .description("open-api接口")
                .contact(contact)
                .version("1.1.0")
                .build();
    }
}
```

跑别人的demo，自己加上swagger，但是却访问不了，最后发现是因为默认的路径被 @EnableWebMvc覆盖掉 或者说是 继承WebMvcConfigurerAdapter类后，默认的静态资源路径对swagger-ui.html访问不起作用，
因此要重写  WebConfigurer（继承WebMvcConfigurerAdapter的那个类） 的一个方法。

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/");

        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");

    }
}
```

