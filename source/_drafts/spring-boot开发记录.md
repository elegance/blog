---
title: spring-boot开发记录
tags:
---
## 官方文档
按照你对应的版本找Reference: [https://projects.spring.io/spring-boot/](https://projects.spring.io/spring-boot/)

比如[1.5.9RELEASE](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/)

#### 常见问题
1. `XXXproperties could not autowire ` 配置类无法autowire的问题？ (Idea 会在运行前智能的提示出来有错误)
这是因为`@SpringBootApplication`默认是扫描当前类所在包路径的，右边情况就是你的配置类`XXConfiguaratoin`不再扫描的包路径下，使用`@ComponentScan(basePackages = "com.xx")`包含其路径/或者指定多个扫描路径即可

#### 常见解决方式

###### 1. 项目中有一些类需外部配置
* 使用`spring-xxx-starter`项目，按照约定来配置，省去编码，此种方式适用于流行的开源框架，比如连数据、连接redis、连接kafka等等
* 编写项目的配置类，加载配置
    * 新增 `XXAppProperties.java`，使用`@ConfigurationProperties(prefix = "tzb.sms")`，加入你要设定的属性（如果系统配置较多，最好分类，即此对象中包括子的properties配置）
    * 新增 `XXAppConfiguaration.java`，使用 `@Configuaration`与`@EnableConfigurationProperties(XXAppProperties.class)`
    * 完成以上就可以使用了：配置文件`application.[properties|yaml]`中使用`tzb.sms`配置，类中使用Autowire注入即可
* 个别属性，不想使用对象，可以使用 `@Autowired ApplicationContext ctx`，然后使用`ctx.getEnviroment().getProperty("tzb.password")`
