---
layout: post
title:  "Spring y Quartz"
date:   2014-09-14 21:33:48
categories: spring, quartz
---

Maven con las dependencias:

{% highlight xml %}

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>net.prietopalacios.josemanuel</groupId>
    <artifactId>quartz-test</artifactId>
    <version>0.0.1</version>
  </parent>

  <groupId>net.prietopalacios.josemanuel.findjob</groupId>
  <artifactId>daemondTask</artifactId>

  <name>daemondTask</name>
  <description>peding...</description>

  <dependencies>
    <!-- QUARTZ -->
    <dependency>
      <groupId>org.quartz-scheduler</groupId>
      <artifactId>quartz</artifactId>
      <!-- <version>2.2.0</version> -->
      <version>1.8.6</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
      <version>${spring.version}</version>
    </dependency>
</project>

{% endhighlight %}

Integración de Spring con quartz:

{% highlight xml %}

<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:p="http://www.springframework.org/schema/p" xmlns:tx="http://www.springframework.org/schema/tx"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
  http://www.springframework.org/schema/tx
  http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context-3.0.xsd">

  <bean name="runJobs" class="org.springframework.scheduling.quartz.JobDetailBean">
    <property name="jobClass" value="net.prietopalacios.josemanuel.quartz.RunJobs" />
    <property name="jobDataAsMap">
      <map>
        <entry key="saverss" value-ref="saveRss" />
      </map>
    </property>
  </bean>

  <bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
    <property name="jobDetail" ref="runJobs" />
    <property name="cronExpression" value="${findjobs.daemon.scheduling.quartz.cronExpression}" />
  </bean>

  <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <property name="jobDetails">
      <list>
        <ref bean="runJobs" />
      </list>
    </property>
    <property name="triggers">
      <list>
        <ref bean="cronTrigger" />
      </list>
    </property>
  </bean>

</beans>

{% endhighlight %}

Clase Java que ejecuta la operación:

{% highlight java %}

package net.prietopalacios.josemanuel.quartz;

import net.prietopalacios.josemanuel.services.SaveRss;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.springframework.scheduling.quartz.QuartzJobBean;

public class RunJobs  extends QuartzJobBean {

  private SaveRss saverss;

  public void setSaverss(SaveRss task) {
    this.saverss = task;
  }

  protected void executeInternal(JobExecutionContext context)
    throws JobExecutionException {
    this.saverss.saveAllRss();
  }

}

{% endhighlight %}

Script cron de ejecución:

{% highlight text %}

findjobs.daemon.scheduling.quartz.cronExpression	= 0/5 * * * * ?
#findjobs.daemon.scheduling.quartz.cronExpression	= 0 0 20 * * ?

{% endhighlight %}
