conf: 

``` java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
@NacosPropertySource(dataId = "tgumi", groupId = "TGUMI", autoRefreshed = true) // nacos
```
