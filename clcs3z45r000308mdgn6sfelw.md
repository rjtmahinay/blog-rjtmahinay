# Micronaut: Yet Another Java  Framework

## Introduction

With the rise of application modernization in the cloud, Java position itself by having different frameworks that can produce lightweight applications in the cloud to minimize memory consumption and quick application start-up.

In this post, I'm gonna explain the codes presented in my demonstration:

[Micronaut Demo](https://github.com/rjtmahinay/micronaut-java)

### What is Micronaut?

Micronaut is a full-stack framework for building microservice and serverless applications. It leverages the use of compile-time inversion of control that results in quick start-up and low memory footprint.

Significant features of this framework to others like Spring Boot:

* Reduced Memory Footprint
    
* Quick startup time
    
* Minimal use of runtime reflection and proxies
    
* Leverages AOT (Ahead of Time) compilation
    
* Doesn't depend any more to the size of the codebase
    

As of the writing, the version of Micronaut is 3.8.0. Also, Micronaut's annotation design is very similar to Spring Boot. Thus, a minimal learning curve for Spring Developers 😁

## 👨🏻‍💻 Create your application!

To create a Micronaut application, you can create it via CLI or the Launcher site.

### **CLI**

You can start by downloading Micronaut CLI [here](https://micronaut.io/download/). After installing you can create your app using the command below:

```bash
mn create-app  example-app --features <additional dependencies>
```

the *\--features* option lets you add dependencies other than the Micronaut native features. After creating the app a micronaut-cli.yml file is created where it shows the information about the project you've created. Below is an example of the contents of the YAML file.

```yaml
applicationType: default
defaultPackage: com.rjtmahinay
testFramework: junit
sourceLanguage: java
buildTool: gradle
features: [annotation-api, app-name, data, data-jpa, gradle, h2, http-client, jackson-databind, java, java-application, jdbc-hikari, junit, logback, micronaut-build, netty-server, openapi, readme, shade, swagger-ui, yaml]
```

### **Micronaut Launcher**

In this approach, you will be shown a user interface to create your Micronaut application with ease. The launcher can be found [here](https://micronaut.io/launch)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673463870586/e0fb006b-8ef4-4b5e-b950-739839cfb1a2.png align="center")

As you can see the launcher design used is familiar to us and is also similar to Spring Initialzr's website. You can see that it has a feature button where you can add your project's dependencies. A preview is also shown for you to have a glimpse of what files are created. When generating your project either you can download the file or push it to a GitHub repository.

## 👨🏻‍💻 Let's start coding!

The main entry application of Micronaut will be a class that invokes the Micronaut class. Below is a simple entry code to run the application.

```java
public class Application {

    public static void main(String[] args) {
        Micronaut.run(Application.class, args);
    }
}
```

If you want to access the different arguments that you can pass to Micronaut's application context. You can use the builder approach.

```java
public class Application {

    public static void main(String[] args) {
        Micronaut.build(args)
                .mainClass(Application.class)
                .eagerInitSingletons(true)
                .start();
        
    }
}
```

### Creating a Bean

In Micronaut you can create your beans inside a configuration called Factory. This factory class will load your specified beans. It is the counterpart of **Configuration** in Spring. Below are some used annotations for bean creation.

```java
@Factory // The configuration annotation to create your beans
public class ApplicationConfiguration {

    @Singleton // Create a single instance of a bean
    public EmployeeBean singletonBean() {
        return new EmployeeBean();
    }

    @Bean // Create a copy of bean for every invocation
    public EmployeeBean bean() {
        return new EmployeeBean();
    }

    @Prototype // Similar to @Bean
    public EmployeeBean prototypeBean() {
        return new EmployeeBean();
    }

    @Context // Create a bean upon start of ApplicationContext
    public EmployeeBean contextBean() {
        return new EmployeeBean();
    } 
}
```

### Autowire a bean

In Spring we use the Autowired annotation but in Micronaut it leverages the use of **Inject** of *jakarta* annotations library.

```java
@Inject // Autowires your bean in the ApplicationContext
private EmployeeBean singletonBean;
```

### Creating Properties

Just like in Spring Boot, we want to configure our custom property class. It uses the same syntax in Spring called **ConfigurationProperties**

```java
// The value of the annotation is the name of your property
@ConfigurationProperties("example")
public class EmployeeProperties {
    private String value;
}
```

You can also get the value directly

```java
// Using an expression language, you can get the value of a property
@Value("${example.value}")
private String value;
```

### Define your DTO

You can use a DTO to map your JSON request and pass it to persistence layer model. Here we encounter the annotations **Serdeable** and **Introspected.**

In Java 17, you can now use records as your request/response model. This removes the use of [Lombok](https://projectlombok.org/) library for most cases.

Other annotations like **NotEmpty** and **DecimalMin** are part of *javax* validations.

```java
@Serdeable // Used for JSON serialization
@Introspected // Used for introspecting your beans in compile-time
public record EmployeeDto(@JsonProperty @NotEmpty String name, @JsonProperty String address, @JsonProperty String position, @JsonProperty @DecimalMin("1.0") BigDecimal salary) {
}
```

### The Persistence Layer

The persistence layer is composed of the repository and its entity. The library used is the Micronaut Data JPA

#### Define your entity

Defining your entity is similar to using the basic JPA format where you initialize it using **Entity** and **Table.** The difference here is you must annotate your class using **Serdeable** for compile-time serialization.

```java
@Serdeable // Used for compile-time serialization
@Setter
@Getter
@Entity // Defines your class as a database entity
@Table(name = "EMPLOYEE") // Annotation to map the class as your table
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "ID", nullable = false, insertable = false, updatable = false)
    private Long id;

    @Column(name = "NAME")
    private String name;
 
    // Removed other attributes for brevity   
}
```

Define your repository

Micronaut supports Synchronous and Reactive repositories. These can be extended by the following interfaces. The annotation is similar to Spring's repository.

```java
@Repository // Used for defining the class as repository
public interface SynchronousEmployeeRepository extends JpaRepository<Employee, Long> { 
 
     @Query("SELECT e FROM Employee e") // Use to define your own query
     List<Employee> findAllEmployee();
}
```

Below can return a type of Reactor's Flux (for list and streams) or Mono (single object).

```java
@Repository // Used for defining the class as repository
public interface ReactiveEmployeeRepository extends ReactorCrudRepository<Employee, Long> {
    
    @Query("SELECT e FROM Employee e") // Use to define your own query
    Flux<Employee> findAllEmployee();
}
```

### The Service Layer

The concept of the service layer is mostly used in the time of Monolithic Architecture which uses the MVC Design Pattern. Although there is no **Service** annotation in Micronaut, you can use the existing **Singleton** to create your service. After all, services are used as beans.

```java
public interface SynchronousEmployeeService {
    Optional<Employee> getEmployee(Long id);
    
    // Removed other functions for brevity
}
```

```java
@Singleton // Make your service as a bean
public class SynchronousEmployeeServiceImpl implements SynchronousEmployeeService {
    @Inject
    private SynchronousEmployeeRepository synchronousEmployeeRepository;

    @Override
    public Optional<Employee> getEmployee(Long id) {
        return synchronousEmployeeRepository.findById(id);
    }
    
    // Removed other logic for brevity
}
```

### The Controller Layer

This consists of your API definitions in Micronaut. It is similar to Spring's **Controller** but the difference is you need an additional **RestController** for your return type to be serialized. In Micronaut, it is already done by one annotation.

```java
// Used for making the API managed by Micronaut threading
@ExecuteOn(TaskExecutors.IO)
// The controller annotation with the define root URI
@Controller("/v1/synchronous/employee")
public class SynchronousEmployeeController {
    // Removed logic for brevity
}
```

#### GET

This will return a serialized response and may be used using a Query Parameter or Path Parameter. Notice that you don't need an extra annotation for defining a Path Parameter, Micronaut already handles it.

```java
@Get(uri = "/{id}", produces = MediaType.APPLICATION_JSON)
public Optional<Employee> getEmployee(Long id) {
    return employeeService.getEmployee(id);
}
```

#### POST

This will invoke an add operation to your persistence layer and custom response. The additional **Body** annotation can be optionally removed since Micronaut knows that it is your request body. The **Valid** annotation asserts that your request body should be validated.

```java
@Post(uri = "/add", consumes = MediaType.APPLICATION_JSON, produces = MediaType.APPLICATION_JSON)
public Employee createEmployee(@Body @Valid EmployeeDto employeeDto) {
    return employeeService.addEmployee(employeeDto);
}
```

#### PUT

This will invoke an update to your data. In the case of combining a Path Parameter and Request Body. You can define it using **PathVariable** (similar to Spring) and **Body**.

```java
@Put(uri = "/update/{id}", consumes = MediaType.APPLICATION_JSON, produces = MediaType.APPLICATION_JSON)
public Employee updateEmployee(@PathVariable Long id, @Body @Valid EmployeeDto employeeDto) {
    return employeeService.updateEmployee(id, employeeDto);
}
```

#### DELETE

This will invoke a deletion of your data.

```java
@Delete(uri = "/delete/{id}", consumes = MediaType.APPLICATION_JSON, produces = MediaType.APPLICATION_JSON)
public String deleteEmployee(Long id) {
    employeeService.deleteEmployee(id);
    return "Successfully deleted";
}
```

### Caching

Sometimes you want to put a cache for method calls in your API. This is usually applied to your API methods. This can be done in Micronaut using the annotations below.

```java
// Stores the data in the specified cache name
@CachePut("synchronous-employee")

// Invalidates all the stored cache by name
@CacheInvalidate("synchronous-employee")
```

## 👨🏻‍💻 Running the application

You can run the application using the build management tool. For this demo, I've used Gradle as my build management tool. You can run your newly created application via the command below:

```bash
./gradlew run
```

After running you can see that the startup time of Micronaut is as twice low as Spring Boot. This is the power of Ahead of Time compilation.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673467859535/0aba03f7-c650-4fcf-b69c-aa498c268c95.png align="center")

Micronaut supports docker build images in Gradle Plugin which you can use

For standard docker image

```bash
./gradlew dockerBuild
```

For GraalVM Native Image

```bash
./gradlew dockerBuildNative
```

## 💭 Final Thoughts

Micronaut is a promising microservice and serverless framework in the Java space. It leverages the use of compile-time inversion of control with very minimal reflection. We want a lightweight and quick startup time application when deploying to the cloud. If you're creating a new service in your organization, Micronaut will be one of the good choices.

## 📝Further Reading

Check out the Micronaut's documentation for more information

* [Micronaut Documentation](https://docs.micronaut.io/latest/guide/)
    
* [Micronaut Configuration References](https://docs.micronaut.io/3.8.0/guide/configurationreference.html)
    
* [Official Micronaut Blogs](https://micronaut.io/blog/)