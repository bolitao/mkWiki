# 入门

## 默认配置

对于一个没有任何额外配置的 controller，默认会使用 spring security 进行保护：

![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210320121559.png)

项目启动时会给出一个随机生成的密码：

``` log
Using generated security password: 2fb8492d-410e-4333-998f-5fa5f68ed52f
```

使用用户名 `user` 和密码 `2fb8492d-410e-4333-998f-5fa5f68ed52f` 即可进行登录。

相关源码：

`UserDetailsServiceAutoConfiguration` 类中：

``` java
private String getOrDeducePassword(SecurityProperties.User user, PasswordEncoder encoder) {
	String password = user.getPassword();
	if (user.isPasswordGenerated()) {
		logger.info(String.format("%n%nUsing generated security password: %s%n", user.getPassword()));
	}
	if (encoder != null || PASSWORD_ALGORITHM_PATTERN.matcher(password).matches()) {
		return password;
	}
	return NOOP_PASSWORD_PREFIX + password;
}
```

## 设置一个密码

在 spring 配置文件添加如下：

``` properties
spring.security.user.name=bolitao
spring.security.user.password=bolitao
```
