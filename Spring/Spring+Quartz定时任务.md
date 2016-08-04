Spring+Quartz定时任务
====================

在开发过程中，经常需要一些定时任务。如定时同步数据库，定时清理数据库冗余数据等，每月月初/月末生成数据的统计报表等。  
Quartz是一个任务调度框架，使用起来比较简单。

### Quartz
Quartz在使用时会用到三个关键的概念Scheduler，Job，Trigger。

* Job. 封装具体的操作，比如同步数据库的操作。
* Trigger. 执行Job的触发器。有SimpleTrigger和CronTrigger两种
	
		SimpleTrigger: 每隔固定时间间隔触发
		CronTrigger: 通过Cron表达式定义复杂的时间规则触发(如每天的凌晨1点，每月最后一天的1点等)

* Scheduler. 调度器，负责调度注册在其中的Trigger和Job

### Spring使用
项目中引入Quartz

		<dependency>  
            <groupId>org.quartz-scheduler</groupId>  
            <artifactId>quartz</artifactId>  
            <version>1.8.4</version>  
        </dependency>

Spring2.5以后的版本使用Quartz需要

		<dependency>
		    <groupId>org.springframework</groupId>
		    <artifactId>spring-context-support</artifactId>
		    <version>${spring.version}</version>
			<scope>compile</scope>
		</dependency>
		
### Demo

##### 1. 定义Job

```java
package com.spring.quartz;

public class DataCleanQuartzJob {
	    
	public void cleanDB() {
   		System.out.println("Starting to clean expire data");
    	    	
    	//具体的业务逻辑

    	System.out.println("Finish to clean expire data");
    }	
}
```

##### 2. 配置Spring Bean

```xml
<bean id="dataCleanQuartzJob" class="com.spring.quartz.DataCleanQuartzJob" />
```

##### 3. 配置JobDetail

```xml
<bean id="quartzJob" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">  
	<property name="targetObject" ref="dataCleanQuartzJob" ></property>  
	<property name="targetMethod" value="cleanDB" ></property>  
</bean>
```
##### 4. 配置Trigger
##### 4.1 SimpleTrigger

```xml
<bean id="quartzTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerBean">  
	<property name="jobDetail" ref="quartzJob"></property>  
	<property name="repeatInterval" value="5000" />   <!-- 每个5s执行一次 -->
</bean>
```
##### 4.2 CronTrigger

```xml
<bean id="quartzTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">  
	<property name="jobDetail" ref="quartzJob" />      	 
	<property name="cronExpression">  
  		<value>0 0 2 * * ?</value>  <!-- 每天2点执行任务 -->
	</property>  
</bean>
```

##### 4.5 配置Scheduler

```xml
<bean id="startQuertz" lazy-init="false" autowire="no" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">  
	<property name="triggers">  
  		<list>  
    		<ref local="quartzTrigger" />  
  		</list>  
	</property>  
</bean>
```

##### 5. 运行

```java
public static void main(String[] args) {
	ApplicationContext context = new ClassPathXmlApplicationContext("grandcanal-beans.xml");
}
```