---
title: "The Value of Quarkus"
seoTitle: "The Value of Quarkus by Tristan Mahinay"
seoDescription: "The Value of Quarkus by Tristan Mahinay"
datePublished: Sat Aug 19 2023 11:11:39 GMT+0000 (Coordinated Universal Time)
cuid: cllhx5u3q000309megf2se3dm
slug: the-value-of-quarkus
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1692432595595/269d80a3-2a39-4531-b140-05ebf9edc8ab.png
tags: cloud, java, redhat, graalvm, quarkus

---

This blog is for my talk presented to [ING Hub Philippines](https://www.ing.com/) about the value of Quarkus in business and development. I've shared my personal experience in using this framework and the value it can give to their organization. As of the time of my talk, the framework version was 3.2.0.Final and this is the first LTS version.

The presentation can be accessed in my GitHub Repository: [The Value of Quarkus](https://github.com/rjtmahinay/presentations)

I've discussed the framework in the following sections:

* What is Quarkus?
    
* Quarkus' Industry Value
    
* Performance
    
    * Quarkus vs Spring Boot
        
    * JIT vs AOT
        
    * Metrics
        
    * AWS Lambda SnapStart
        
* Spring Framework Support
    
* Sustainability
    

# What is Quarkus?

The framework is called *Supersonic. Subatomic. Java.* It can be explained into 5 characteristics: *Kube-native*, *Cloud-native*, *Imperative and Reactive Code,* *Developer-centric*, and *Standards*.

## Kube-native

Quarkus is designed for Kubernetes-based development. It is a framework to leverage the characteristics of Kubernetes specifically for RedHat OpenShift. When developing applications in Quarkus, you're developing as if you're inside the Kubernetes platform due to the usage of YAML files. You can configure the environment directly in Quarkus. Thus, showing Single-step Deployment and Remote Development.

An example YAML file on how to deploy an application and connect to OpenShift automatically.

```yaml
"%prod":
  quarkus:
    kubernetes-client:
      trust-certs: true
      api-server-url: ${OPENSHIFT_URL}
      token: ${QUARKUS_KUBERNETES_CLIENT_TOKEN}
    openshift:
      route:
        expose: true
      env:
        secrets: mysql
    kubernetes:
      deploy: true
    s2i:
      base-jvm-image: registry.access.redhat.com/ubi8/openjdk-17:1.16-1
    datasource:
      db-kind: mysql
      jdbc:
        url: jdbc:mysql://<SQL Hostname>:3306/demo
      password: ${database-password}
      username: ${database-user}
```

## Cloud-native

Since it is a Kubernetes Java framework, it is optimized to be used in the cloud. It Shows fast startup times, minimal usage of reflection, and support for GraalVM Native compilation. This characteristic is related to the benefits of being a Greener framework.

## Imperative and Reactive Code

Quarkus supports both imperative and reactive programming whether in microservices, event-driven microservices, or serverless architectures. You can code HTTP Microservices, Reactive APIs, and Functions. Quarkus has all extensions for that.

Below are some examples of API coding in Quarkus.

**AWS Lambda code using API Gateway**

```java
@Named("/lambda1")
public class Lambda1 implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {
    
    @Override
    public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent request, Context context) {
        String httpMethod = request.getHttpMethod();
        String result = "";

        if ("GET".equals(httpMethod)) {
            // Business logic
        }

        return new APIGatewayProxyResponseEvent().withBody("Invoked Lambda")
                        .withStatusCode(200);
    }
}
```

**HTTP Microservices**

```java
@Path("/employee")
@ApplicationScoped
public class EmployeeResource {
    @Inject
    EmployeeRepository employeeRepository;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Employee> getEmployee() {
        return employeeRepository.listAll();
    }
}
```

**Reactive Microservices**

```java
@Path("/employee")
@ApplicationScoped
public class EmployeeResource {
    @Inject
    EmployeeRepository employeeRepository;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Multi<Employee> getEmployee() {
        return Multi.createFrom().items(employeeRepository.streamAll());
    }
}

```

## Developer-centric

Quarkus offers a seamless development experience for developers and users. It offers a variety of tools such as live reloading, remote development, Dev UI/Services, Continuous Testing, and Quarkus Extensions.

Dev UI is a local development website typically accessed via http://localhost:8080. It shows the dependency that you've used in your application and the ability to deploy to a cloud platform like RedHat OpenShift automatically.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692437604435/e80c93f0-a32d-40a9-8692-9c138663b8d7.png align="center")

Dev Services is Quarkus' ability to give an on-demand environment via TestContainers for common development tools like Databases, Kafka, RabbitMQ, etc. You can check out the documentation in [Dev Services](https://quarkus.io/guides/dev-services)

[Quarkus Extensions](https://quarkus.io/extensions/) are external libraries integrated into Quarkus CDI. These are from the [Quarkus Platform](https://github.com/quarkusio/quarkus-platform) and [Quarkiverse](https://github.com/quarkiverse)

## Standards

The framework is backed by industry standards and specifications. Some of these are Vert.x, Eclipse Microprofile, Micrometer, and Jakarta EE.

# Quarkus Industry Values

The industry values that it can give are based on cost savings, the reduction of the resources needed to deploy an application, and its environmental impact.

* **Cost Savings:** Less memory footprint, fast start-up time, and less disk footprint
    
* **Speed and Agility:** faster time to market using Quarkus Extension and crafted from well-known libraries which can be easily learned
    
* **Standard and Support:** [RedHat Build Support for Quarkus](https://code.quarkus.redhat.com/), an active community with regular releases in GitHub and based on [CNCF technologies](https://www.cncf.io/)
    
* **Sustainability:** Write Greener applications using Quarkus. Due to the minimal reflection and the small memory footprint of Quarkus, the framework shows characteristics of decarbonization.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692440003010/13ae75a7-6ff2-466c-9ab7-19988d722b44.png align="center")

## Performance Metrics

I've presented a comparison of Quarkus to Spring Boot. I compared it to the following metrics:

* Image Size
    
    * Spring Boot has a lesser size due to reflection. Quarkus performs build-time compilation causing increased image size.
        
* Startup Time
    
    * Quarkus doesn't need anymore to execute always dynamically the metadata of a program. Thus, improving startup times.
        
* Memory Usage
    
    * Quarkus is optimized to reduce its memory usage consumption. The CDI process of Quarkus uses Ahead of Time compilation to minimize the use of reflection.
        

# AWS Lambda SnapStart

The performance of Quarkus and Spring Boot in response time decreased significantly. Although, Spring Boot was 6x more efficient when integrated with SnapStart. While Quarkus had it 2x more efficient.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692442275248/1d9649d6-1b11-4ea3-aaf3-e10cbfc41a38.png align="center")

## Spring Framework Support

Quarkus support a number of annotations of Spring inside the Quarkus CDI (Quarkus ArC). The documentation can be accessed by clicking the link for each bullet. These can be listed as:

* [Spring Boot Properties](https://quarkus.io/guides/spring-boot-properties)
    
* [Spring Cache](https://quarkus.io/guides/spring-cache)
    
* [Spring Cloud](https://quarkus.io/guides/spring-cloud-config-client)
    
* [Spring Data](https://quarkus.io/guides/spring-data-jpa) & [Data Rest](https://quarkus.io/guides/spring-data-rest)
    
* [Spring Dependency Injection](https://quarkus.io/guides/spring-di)
    
* [Spring Scheduled](https://quarkus.io/guides/spring-scheduled)
    
* [Spring Security](https://quarkus.io/guides/spring-security)
    
* [Spring Web](https://quarkus.io/guides/spring-web)
    

## Sustainability

Quarkus offer writing greener applications as prescribed by [RedHat Greener Application Executive Summary](https://www.redhat.com/en/resources/greener-java-applications-detail). In this summary, they compare Quarkus to the energy consumption of a single light bulb, startup times, and the carbon dioxide equivalent (CO2-eq) respectively.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692442975902/b10241b0-67c1-4507-8daf-7d20192c4e7b.png align="center")

# Final Thoughts

In this presentation, I've discussed what will be the benefit of using Quarkus from a developer's perspective. It's not just a seamless development experience but the impact that a developer can give on their organization and the environment.

With the increased modernization and migration of legacy applications to the cloud. We can expect that they will consider a developer-friendly and sustainable framework that can optimize their applications in the cloud.