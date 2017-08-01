---
title: 配置logback按日期输出
date: 2017-08-01 17:33:40
categories: "spring"
tags:
    - logback -
---
 logback是一个非常好的日志处理插件,据说比log4j处理效率高.目前spring boot的默认日志处理插件用的就是logback.这里简单说下我的logback配置.  
&#160; &#160; &#160; &#160;一般日志的输出分为控制台,文件和数据库输出,logback对三种方式都支持,不过我不推荐数据库方式,实际使用的时候需要先配置好数据表,而且日志产生的数据量很大,数据库增长速度特别快,查询的时候并没有显现出关系数据库的优点.所以我一般推荐使用控制台输出并对日志按日期进行持久化.  
&#160; &#160; &#160; &#160;因为spring boot默认自带logback,如果需要单独使用的时候,需要引入maven文件
<!--more--> 
```	
<dependency>
<groupId>ch.qos.logback</groupId>
	<artifactId>logback-access</artifactId>
	<version>1.2.0</version>
</dependency>
<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-classic</artifactId>
	<version>1.2.0</version>
</dependency>
<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-core</artifactId>
	<version>1.2.0</version>
</dependency>
```
然后设置logback.xml文件,在项目配置文件加入:

```
logging.config=classpath:logback.xml
```
我们重点看一下logback.xml里面的内容:

```
<configuration>

	<!-- 从高到地低 OFF 、 FATAL 、 ERROR 、 WARN 、 INFO 、 DEBUG 、 TRACE 、 ALL -->
	<!-- 日志输出规则 根据当前ROOT 级别，日志输出时，级别高于root默认的级别时 会输出 -->
 
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<!-- 只显示info级别内容 -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>DEBUG</level>
		</filter> 
		<encoder charset="UTF-8">
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} - %msg%n
			</pattern>
		 
		</encoder>
	</appender>
 
	<!-- 下面配置一些第三方包的日志过滤级别，用于避免刷屏 -->
	<logger name="org.springframework" level="WARN" /> 
	<!-- 指定包下的log级别 additivity设置不向上传递 -->
	<logger name="com.xiaoyu.modules.biz" level="DEBUG" additivity="false" />

	<!--生产环境 -->
	<appender name="FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>/home/logs/demo.out</file>

		<!-- 按时间输出 -->
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>/home/logs/%d{yyyy-MM,aux}/%d{yyyy-MM-dd}.log.zip
			</fileNamePattern>
			<!-- logs/%d{yyyy-MM,aux}/%d{yyyy-MM-dd_HH-mm}.log.zip -->
			<maxHistory>30</maxHistory>
		</rollingPolicy>
		<layout class="ch.qos.logback.classic.PatternLayout">
			<pattern> %d{HH:mm:ss.SSS}-%msg%n</pattern>
		</layout>
	 
	</appender>
	<!-- 默认级别和输出 -->
	<root level="DEBUG">
		<appender-ref ref="FILE" />
		<appender-ref ref="STDOUT" />
		<!-- <appender-ref ref="DB" /> -->
	</root>
</configuration>
```
&#160; &#160; &#160; &#160;这里可以看到整个配置文件大致分为 appender,logger,root三部分,appender是对输出方式的设置,这里设置了FILE和STDOUT俩种方式,logger则更具体的输出级别,而root表明了默认的项目里面哪些输出方式.
我们看一下FILE方式里面.  
1.file指定了日志输出在哪个文件里面,这里指定了/home/logs/demo.out,后面的持久化就会把每天的日志取出来,然后清空demo.out文件,重新记录日志.  
2.rollingPolicy为输出策略,这里按月输出每天的日志zip文件,实际会按月分为文件夹,里面是每一天的日志文件.  
3.layout为输出的格式.这里精确到秒,具体的符号的意思可以查询logback官网,里面有更详细的配置.  
&#160; &#160; &#160; &#160;好了,基本的配置和方法就这些,详细的可以参加我的github.开发中配置好日志输出对bug的检查还是非常重要的,实际项目中更应该重视日志的规范性.