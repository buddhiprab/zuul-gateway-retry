# Configuring Auto Retry for Spring Boot-Netflix Zuul API Gateway

 When Netflix Zuul is used as a gateway to forward the incoming requests to backend services, there is always a chance that a request may fail to back end service.
 
 When a request fails, you may want to have the request be retried automatically. To do so when using Sping Cloud Netflix, you need to include Spring Retry on your application’s classpath. When Spring Retry is present, load-balanced Zuul automatically retry any failed requests (as below example configurations Zuul will retry 2 times if a backend service is down).
 
 
 ```
 Zuul, uses An Apache HTTP client which leverages Spring Retry to retry failed requests.
 two classes to explore
 public class RetryableRibbonLoadBalancingHttpClient
       extends RibbonLoadBalancingHttpClient
 public class RibbonLoadBalancedRetryPolicy implements LoadBalancedRetryPolicy 
 ```
 
 you can control the retry functionality by configuring certain Ribbon properties. 
 To do so, set the `ribbon.MaxAutoRetries`, `ribbon.MaxAutoRetriesNextServer`, and `ribbon.OkToRetryOnAllOperations` properties.
 
 application.yml
 ```
 server:
   port: 8080
 
 spring:
   application:
     name: gateway
   cloud:
     loadbalancer:
       retry:
         enabled: true
 
 eureka:
   client:
     serviceUrl:
       defaultZone: http://localhost:8085/eureka/
     fetch-registry: true
     register-with-eureka: true
   instance:
     prefer-ip-address: true
     lease-renewal-interval-in-seconds: 5
 
 zuul:
   prefix: /api
   retryable: true
 
 ribbon:
   OkToRetryOnAllOperations: true
   MaxAutoRetries: 2
 ```
 
 pom.xml
 ```
 <?xml version="1.0" encoding="UTF-8"?>
 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>2.3.0.RELEASE</version>
       <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.api</groupId>
    <artifactId>gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>gateway</name>
    <description>API Gateway project for Spring Boot</description>
 
    <properties>
       <java.version>11</java.version>
       <spring-cloud.version>Hoxton.SR4</spring-cloud.version>
    </properties>
 
    <dependencies>
       <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-jpa</artifactId>
       </dependency>
       <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
       </dependency>
       <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>
       <dependency>
          <groupId>org.springframework.retry</groupId>
          <artifactId>spring-retry</artifactId>
          <version>1.3.0</version>
       </dependency>
 
       <dependency>
          <groupId>com.h2database</groupId>
          <artifactId>h2</artifactId>
          <scope>runtime</scope>
       </dependency>
      
    </dependencies>
 
    <dependencyManagement>
       <dependencies>
          <dependency>
             <groupId>org.springframework.cloud</groupId>
             <artifactId>spring-cloud-dependencies</artifactId>
             <version>${spring-cloud.version}</version>
             <type>pom</type>
             <scope>import</scope>
          </dependency>
       </dependencies>
    </dependencyManagement>
 
    <build>
       <plugins>
          <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
       </plugins>
    </build>
 
 </project>
 ```

GatewayApplication.java
 ``` 
 package com.api.gateway;
 
 import org.springframework.boot.SpringApplication;
 import org.springframework.boot.autoconfigure.SpringBootApplication;
 import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
 import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
 
 @SpringBootApplication
 @EnableZuulProxy
 @EnableDiscoveryClient
 public class GatewayApplication {
 
    public static void main(String[] args) {
       SpringApplication.run(GatewayApplication.class, args);
    }
 
 }
  ```