---
title: spring注入static变量
date: 2017-03-20 21:04:43
categories: spring
tags: 
	- spring 
	- static
---

在spring中默认是不能注入static变量的,因为static是类变量,而spring是基于实例对象进行注入的.  
但是我们有时候需要static的变量进行操作.比如我们经常使用的jedis,如果每次调用jedis的时候都要实例化一遍,是非常麻烦的.  
<!--more-->
解决办法如下:
1. 我们先通过set方法注入一个jedisPool对象 ,然后再将这个对象赋值给static变量pool,于是我们就可以使用 pool了
```
@Component
public class JedisUtils {
	private static JedisPool pool;

	@Autowired
	@Qualifier("jedisPool")
	public void setPool(JedisPool jedisPool) {
		pool = jedisPool;
	}
}
```


2. 这是我在框架里面看到的.通过设置一个全局的applicationContext变量,在spring启动的时候就加载,然后利用context加载static变量

```
@Component
@Lazy(false)//启动便加载
public class SpringBeanUtils implements ApplicationContextAware, DisposableBean {
	private static ApplicationContext applicationContext = null;
	
	@SuppressWarnings("unchecked")
	public static <T> T getBean(String name) {
		return (T) applicationContext.getBean(name);
	}

	/**
	 * 从静态变量applicationContext中取得Bean, 自动转型为所赋值对象的类型.
	 */
	public static <T> T getBean(Class<T> requiredType) {
		return applicationContext.getBean(requiredType);
	}


	@Override
	public void setApplicationContext(ApplicationContext arg0) throws BeansException {
		SpringBeanUtils.applicationContext = arg0;
	}

	@Override
	public void destroy() throws Exception {
		applicationContext = null;
	}
}
```

然后我们这样:

```
//@Component 这时候就不能有这个注解了,因为已经无需注入了
public class JedisUtils {
	private static JedisPool pool=SpringBeanUtils.getBean(JedisPool.class);
}
```
