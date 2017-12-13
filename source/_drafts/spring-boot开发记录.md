---
title: spring-boot开发记录
tags:
---
#### 常见问题
1. `XXXproperties could not autowire ` 配置类无法autowire的问题？ (Idea 会在运行前智能的提示出来有错误)
这是因为`@SpringBootApplication`默认是扫描当前类所在包路径的，右边情况就是你的配置类`XXConfiguaratoin`不再扫描的包路径下，使用`@ComponentScan(basePackages = "com.xx")`包含其路径/或者指定多个扫描路径即可