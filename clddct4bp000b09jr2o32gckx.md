# Modular Monolithic in Practice

With the usage of microservices in application modernization, we have seen both the advantages and disadvantages of maintaining such software development styles.

When we create applications mostly in enterprise organizations, the first thing that comes to our mind now is how to decouple our applications.

But there will be times when creating too many microservices is not the best way and may cost you time and money.

Because of that, one alternative is to leverage the use of modules.

## Modules

Modularity is a concept that is used to build applications to separate logical units through modules, giving you *highly cohesive* code.

These modules are analogous to a compartment where you put a specific feature of your application.

Creating modules has different benefits during software development, whether ground-up or maintenance.

## Modular Monolithic

Modular Monolithic is an architectural style where your code is structured on the concept of modules.

It separates your functions into different logical units but still in a single application.

You can describe this as a composition of directories and those inner directories contain your working code.

In practice, this style is used when you don't want to manage different applications as seen in microservice development. Below are some reasons why to use Modular Monolithic.

* Single deployment - your code can be deployed to the cloud without configuring and maintaining too many applications. These can be seen in batch and some microservice applications.
    
* Issues of third-party dependency - imagine you have more than 5 applications with library violations using GitLab or JFrog Xray for example, these can be a big pain for the developers.
    
* Greenfield Development - when starting a new application we wouldn't want to build our applications immediately in microservice. This may cause over-engineering and high maintenance costs for no reason.
    
* Cost of developers - Too many services are costly to an organization. This can be seen when the organization is in the stage of budgeting and has few developers in place.
    

## Architecture

The architecture of the Modular Monolithic is quite simple. It's in a single project with different components separated into a well-defined logical directory. The modules or projects (for multi-module) define their way to prevent leaking of their implementation. The multi-module can define what to expose via the custom configuration of Gradle while Spring Modulith leverages the use of *internal* modules.

This can be deployed to the cloud using a single Dockerfile or a Multi-container approach using Docker Compose or Kubernetes-based technology. The application used in this demonstration is Spring and Gradle and a new module under Spring Framework called [Spring Modulith](https://spring.io/projects/spring-modulith)*.*

### Multi-Module

![Gradle Multi-module](https://foojay.io/wp-content/uploads/2023/01/modular-monolithic-gradle-700x455.png align="left")

If you want to create and deploy a modular monolithic application, you can use a common approach used by developers called multi-module approach. This can be done using the build management tools like Maven or Gradle. In this post, we will be dealing with Gradle.

In the architecture above we can see that the application consists of 3 different projects inside a main project called *modular-monolithic-gradle*. The 3 projects are called sub-projects and can define what to expose to other projects.

The *common* project also called *shared* project which contains global or redundant codes that can be seen when you have too many service modules.

The *service* project may contain codes for a specific domain. In this case, its sole responsibility is to manage the functionality regarding Employee. You can have many services based on your use case.

The *application* project is your main project that will invoke your services and behaves like a *Service Locator*. See a simple introduction to this pattern [here](https://www.geeksforgeeks.org/service-locator-pattern/). In Spring, the call for different services can be done using *@Qualifier* annotations and extending a base service.

Sub-modules themselves can have their Gradle lifecycle which is controlled by their build file. In there, you can define what to expose or not.

### Spring Modulith

![Spring Modulith Architecture](https://foojay.io/wp-content/uploads/2023/01/modular-monolithic-modulith-700x341.png align="left")

Spring Modulith is an experimental project by Spring that can be used for modular monolithic applications. Its feature is to have a well-defined modular structure for Spring Beans and have control over what to expose or not.

Here we have the same structure as the multi-module above but using different packages and a main project (main package) only. We can hide some functionalities like the code for repositories or the model classes. Thus, controlling how you will expose your beans. You can expose many services here too and hide their respective persistence and view layer via a package called *internal*.

For scenarios where you want to create another service, you can just create another one under the com.rjtmahinay (main package) and can be called to other services (other service packages).

Although under the hood, Spring Modulith uses the multi-module approach too in its respective sub-functionalities. This technology can help us improve the way we create modules, especially in Spring.

### Cloud Deployment

![Modular Monolithic Cloud Deployment](https://foojay.io/wp-content/uploads/2023/01/modular-monolithic-deployment.png align="left")

So far I've discussed the approaches to create a modular monolithic application using the common and a new approach. This time the architecture above shows the *single deployment* benefit of modular monolithic.

To deploy your application to the cloud, you can do the single Dockerfile deployment which contains a single image of your application or deploy it in a multi-container style which can be done by a single image (single jar) or multi-image (multiple jars). The multi-module approach can deploy the sub-projects as a multi-container deployment since it can have its image (jar file). For Spring Modulith, since internally it can be a multi-module, the modulith application can still be deployed in single or multiple deployments.

## Final Thoughts

Understandably, developers will opt for microservices since it gives the flexibility of technology to be used, style of deployment and decoupling of functionalities.

But we need to remember that all software requirements don't need to be created in a microservice fashion.

Problems arise when microservices become a Distributed Monolithic, which defeats its purpose.

In this post, I've presented an option called Modular Monolithic which is an alternative for the complexity seen in microservices and how modularity can be integrated into your software requirements.

## Further Reading

* [Spring Modulith Reference Documentation](https://docs.spring.io/spring-modulith/docs/current/reference/html/)
    
* [Modular Monolithic by JRebel](https://www.jrebel.com/blog/what-is-a-modular-monolith#when-to-use-modular-monoliths)
    
* [Gradle Multi-module](https://docs.gradle.org/current/userguide/multi_project_builds.html)
    

*Originally published on* [*Foojay.io*](https://foojay.io/today/modular-monolithic-in-practice/)