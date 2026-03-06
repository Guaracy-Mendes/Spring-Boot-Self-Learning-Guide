
# Spring Boot Self Learning Guide

[Spring Boot](https://spring.io/projects/spring-boot) is a popular framework for building Java enterprise applications.
Spring Boot provides support for a wide range of features 
and integrations with most of the libraries to build different types of applications.

For beginners this can be overwhelming. This guide will help you get started with Spring Boot.

![spring-boot-self-learning-guide.png](images/spring-boot-self-learning-guide.png)

The goal of this self-learning guide is to provide a step-by-step approach to understanding and mastering Spring Boot. 
It covers essential concepts, best practices, and practical examples to help you build robust and scalable applications.

## Prerequisites
- Knowledge of Java
- Knowledge of Spring (nice to have)
- Basic knowledge of Databases and SQL
- Basic knowledge of Maven
- IDE (preferably [Visual Studio Code](https://code.visualstudio.com/))

## How to use this guide?
- Create a new GitHub repository to implement the sample application(jblogger)
- As you go through the guide, implement the assignments, push your changes to the repository
- Use branches to keep different versions of the code (In-memory repo, JdbcClient, Spring Data JPA based repos, etc.)

# Introduction

- Spring framework was created as an alternative to J2EE (J2EE was renamed to Java EE and then to Jakarta EE)
- Spring provides Dependency Injection, Aspect Oriented Programming, and many other features to simplify Java development.
- Spring provides many modules (Jdbc, ORM, Security, Batch, etc) to build enterprise applications.
- Spring initially provided XML based configuration, but later introduced Java based configuration and annotations.
- However, configuring Spring applications were still challenging and usually involved a lot of boilerplate code copy-pasted from previous projects.
- Spring Boot is created to simplify the process of creating Spring applications.
- Spring Boot provides opinionated defaults, auto-configuration, and starter dependencies to reduce boilerplate code and make development faster.
- Spring Boot also provides embedded servers support to run applications as fat/uber jar.

**I strongly believe that the best way to learn a new technology is to build something with it.**

**Throughout this guide, you will learn how to use Spring Boot by building a Blog REST API (called jblogger) progressively.
In this guide, each section will explain a specific feature of Spring Boot and will give an assignment to build jblogger**

## jblogger
- Admin user can create, update, delete posts.
- Users can register and login.
- Anonymous users can view posts.
- Authenticated users can add comments.
- Admin users can view and delete comments.

Though it is a very simple REST API, it covers most of the Spring Boot features and best practices.

### Database
- We will use the PostgreSQL database for this guide. But you can use any database of your choice.
- We will use Flyway for database migrations.
- We will run PostgreSQL in a Docker container using Docker Compose.

#### USERS Table

| Column     | Type         | Constraints                 |
|------------|--------------|-----------------------------|
| ID         | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |
| FULL_NAME  | VARCHAR(255) | NOT NULL                    |
| EMAIL      | VARCHAR(255) | NOT NULL, UNIQUE            |
| PASSWORD   | VARCHAR(255) | NOT NULL                    |
| ROLE       | VARCHAR(50)  | NOT NULL                    |
| CREATED_AT | TIMESTAMP    | NOT NULL                    |
| UPDATED_AT | TIMESTAMP    |                             |

#### POSTS Table

| Column     | Type         | Constraints                 |
|------------|--------------|-----------------------------|
| ID         | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |
| TITLE      | VARCHAR(255) | NOT NULL                    |
| SLUG       | VARCHAR(255) | NOT NULL, UNIQUE            |
| CONTENT    | TEXT         | NOT NULL                    |
| AUTHOR_ID  | BIGINT       | NOT NULL                    |
| STATUS     | VARCHAR(50)  | NOT NULL                    |
| CREATED_AT | TIMESTAMP    | NOT NULL                    |
| UPDATED_AT | TIMESTAMP    |                             |

#### COMMENTS Table

| Column     | Type         | Constraints                 |
|------------|--------------|-----------------------------|
| ID         | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |
| POST_ID    | BIGINT       | NOT NULL                    |
| AUTHOR_ID  | BIGINT       | NOT NULL                    |
| CONTENT    | TEXT         | NOT NULL                    |
| CREATED_AT | TIMESTAMP    | NOT NULL                    |
| UPDATED_AT | TIMESTAMP    |                             |

### REST API Endpoints

#### Login

- `POST /api/v1/login`: Authenticate a user and return a JWT token.

#### Register

- `POST /api/v1/register`: Register a new user.

#### Create Post

- `POST /api/v1/posts`: Create a new post.

#### Get Posts

- `GET /api/v1/posts?page=1&size=10`: Retrieve a page of posts.

#### Get Post By Slug

- `GET /api/v1/posts/{slug}`: Retrieve a post by its slug.

#### Update Post

- `PUT /api/v1/posts/{slug}`: Update a post by its slug.

#### Delete Post

- `DELETE /api/v1/posts/{slug}`: Delete a post by its slug.

#### Add Comment

- `POST /api/v1/posts/{slug}/comments`: Add a comment to a post.

#### Get Comments By a Post Slug

- `GET /api/v1/posts/{slug}/comments`: Retrieve comments for a post by its slug.

#### Delete Comment

- `DELETE /api/v1/comments/{id}`: Delete a comment by its ID.


# Getting Started

## Table of Contents
- [Prerequisites](#prerequisites)
- [Running a Postgres Database](#running-a-postgres-database)
- [Creating a Spring Boot Project](#creating-a-spring-boot-project)

## Prerequisites
To follow along with this guide, you will need to install the following:

- JDK 21 or later (prefer JDK 25)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Docker](https://www.docker.com/) & Docker Compose Or you can use Podman ...
- [Git](https://git-scm.com/)

I would recommend using:

- [SDKMAN](https://sdkman.io/) to install JDK
- Docker Desktop for Mac/Windows Or Pdoman Desktop ...

## Running a Postgres Database
Once you have installed these, you can start a Postgres database using Docker Compose.

Create a file called `docker-compose.yml` with the following contents:

```yaml
services:
  postgres:
    image: postgres:18
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
```

Run `docker compose up -d` to start the database.

## Creating a Spring Boot Project
You can create a new Spring Boot project using Visual Studio Code or by going to https://start.spring.io/.

Generate a new Spring Boot project by using the following details:

```shell
Language: Java
Build tool: Maven
Spring Boot version: 4.0.1
Group: com.dgw
Artifact: jblogger
Name: jblogger
Package name: com.dgw.jblogger
Configuration: Properties
Java version: 25
Dependencies: Web
```

Generate the project, unzip and import it into Visual Studio Code.


# Spring Boot Core Concepts

## Table of Contents
- [Bean Configuration Examples](#bean-configuration-examples)
- [Spring Component Scan](#spring-component-scan)

- A Spring bean is a Java object that is managed by the Spring IoC container.
- The Spring IoC container is responsible for creating, configuring, and managing the lifecycle of Spring beans.
- You can use the generic `@Component` annotation to mark a class as a Spring bean.
- But there are more specific annotations that you can use based on the type of bean you want to create.
  - `@Service` for business logic 
  - `@Repository` for data access
  - `@Controller` for web controllers
  - `@RestController` for REST controllers
- You can also use the `@Bean` annotation to register a bean definition with the container programmatically.
- You can also use the `@Configuration` annotation to mark a class as a Spring configuration class.
- You can inject one bean into another bean using the `@Autowired` annotation.
- You can use constructor injection, setter injection, and field injection.
- It is recommended to use constructor injection for all beans.
- If there is only one constructor for a class, you don't need to annotate the constructor with `@Autowired`.

## Bean Configuration Examples

Annotation based dependency injection example:

```java
@Repository
class UserRepository {
    // data access logic methods
}

@Service
class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // business logic methods
}

@Component
class UserMapper {
    // data mapping logic methods
}

@RestController
class UserController {
    private final UserService userService;
    private final UserMapper userMapper;
    
    public UserController(UserService userService, UserMapper userMapper) {
        this.userService = userService;
        this.userMapper = userMapper;
    }
    
    // request handler methods
}
```

Registering a beans using Java Configuration:

```java
@Configuration
class AppConfig {
    @Bean
    public UserReposiotry userReposiotry() {
        return new UserReposiotry();
    }
    
    @Bean
    public UserService userService(UserReposiotry userReposiotry) {
        return new UserService(userReposiotry);
    }
    
    @Bean
    public UserMapper userMapper() {
        return new UserMapper();
    }
}
```

**NOTE:** Usually Java Configuration is used to configure beans of types from external libraries 
as we can't annotate them with Spring annotations.

## Spring Component Scan
You can use the `@ComponentScan` annotation to configure the package(s) where Spring should look for components.

```java
@Configuration
@ComponentScan("com.dgw.jblogger")
class AppConfig {
    
}
```

In Spring Boot applications, the main entry point is the class with `@SpringBootApplication` annotation.

```java
package com.dgw.jblogger;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

The `@SpringBootApplication` annotation is a meta-annotated with `@Configuration` and `@ComponentScan` annotations.
So, all the components from the `com.dgw.jblogger` package and its sub-packages will be registered with the Spring IoC container.

**IMPORTANT:** You should keep all the classes in the same package or sub-packages of the main entry point class.
Otherwise, Spring will not be able to find them by default.


# Spring MVC REST APIs

## Table of Contents
- [What is REST?](#what-is-rest)
- [Importance of HTTP Methods, RESTful URLs, and Status Codes](#importance-of-http-methods-restful-urls-and-status-codes)
- [REST API Examples: Blog Application](#rest-api-examples-blog-application)
- [Writing HTTP Request Handlers with Spring MVC](#writing-http-request-handlers-with-spring-mvc)
- [Mapping HTTP Methods](#mapping-http-methods)
- [Spring MVC Annotations](#spring-mvc-annotations)
- [Using ResponseEntity](#using-responseentity)
- [Validation](#validation)
- [Error Handling](#error-handling)
- [Assignment](#assignment)

## What is REST?

REST (Representational State Transfer) is an architectural style for designing networked applications. 
It relies on stateless, client-server communication using standard HTTP methods. 
In REST, resources are identified by URLs, and operations are performed using HTTP methods like GET, POST, PUT, and DELETE.

Key principles of REST:
- **Client-Server Architecture**: Separation of concerns between client and server
- **Stateless**: Each request contains all information needed to process it
- **Cacheable**: Responses can be cached to improve performance
- **Uniform Interface**: Consistent way to interact with resources using standard HTTP methods

## Importance of HTTP Methods, RESTful URLs, and Status Codes

### HTTP Methods
Using proper HTTP methods provides semantic meaning to API operations:
- **GET**: Retrieve resources (safe and idempotent)
- **POST**: Create new resources
- **PUT**: Update existing resources (idempotent)
- **PATCH**: Partial update of resources
- **DELETE**: Remove resources (idempotent)

### RESTful URLs
Well-designed URLs make APIs intuitive and self-documenting:
- Use nouns for resources, not verbs
- Use plural names for collections
- Use hierarchical structure for relationships
- Keep URLs clean and predictable

### HTTP Status Codes
Status codes communicate the result of operations clearly:
- **2xx**: Success (200 OK, 201 Created, 204 No Content)
- **3xx**: Redirection
- **4xx**: Client errors (400 Bad Request, 404 Not Found, 401 Unauthorized)
- **5xx**: Server errors (500 Internal Server Error)

## REST API Examples: Blog Application

Here are examples of RESTful endpoints for a blog application:

### Resource: Posts
```
GET    /api/v1/posts              - Get all posts
GET    /api/v1/posts/{id}         - Get a specific post
POST   /api/v1/posts              - Create a new post
PUT    /api/v1/posts/{id}         - Update a post
DELETE /api/v1/posts/{id}         - Delete a post
```

### Sub-resource: Comments (belonging to Posts)
```
GET    /api/v1/posts/{postId}/comments              - Get all comments for a post
GET    /api/v1/posts/{postId}/comments/{commentId}  - Get a specific comment
POST   /api/v1/posts/{postId}/comments              - Create a comment on a post
PUT    /api/v1/posts/{postId}/comments/{commentId}  - Update a comment
DELETE /api/v1/posts/{postId}/comments/{commentId}  - Delete a comment
```

### Resource: Authors
```
GET    /api/v1/authors            - Get all authors
GET    /api/v1/authors/{id}       - Get a specific author
GET    /api/v1/authors/{id}/posts - Get all posts by an author
```

**NOTE:** Don't use URLs like `/api/v1/getAllPosts` or `/api/v1/createPost` etc.

## Writing HTTP Request Handlers with Spring MVC
To build REST APIs with Spring MVC, add the following Spring Boot starter dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
```

Spring MVC provides annotations to handle HTTP requests in a clean, declarative way.

### @Controller vs @RestController

**@Controller**: Used for traditional MVC applications that return views (HTML pages). 
Methods typically return view names.

```java
@Controller
public class WebController {
    @GetMapping("/home")
    public String home(Model model) {
        return "home"; // Returns view name
    }
}
```

**@RestController**: A convenience annotation that combines `@Controller` and `@ResponseBody`. 
Used for REST APIs that return data (JSON/XML). All methods automatically serialize return values to the response body.

```java
@RestController
public class ApiController {
    @GetMapping("/api/v1/posts")
    public List<Post> getPosts() {
        return postService.getAllPosts(); // Returns JSON
    }
}
```

## Mapping HTTP Methods

Spring provides specialized annotations for different HTTP methods:

### @GetMapping
Used to retrieve resources:

```java
@RestController
@RequestMapping("/api/v1/posts")
public class PostController {

    @GetMapping
    public List<Post> getAllPosts() {
        return postService.getAllPosts();
    }

    @GetMapping("/{id}")
    public Post getPost(@PathVariable Long id) {
        return postService.getPostById(id);
    }
}
```

### @PostMapping
Used to create new resources:

```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public Post createPost(@RequestBody Post post) {
    return postService.createPost(post);
}
```

### @PutMapping
Used to update existing resources:

```java
@PutMapping("/{id}")
public Post updatePost(@PathVariable Long id, @RequestBody Post post) {
    return postService.updatePost(id, post);
}
```

### @DeleteMapping
Used to delete resources:

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deletePost(@PathVariable Long id) {
    postService.deletePost(id);
}
```

### @PatchMapping
Used for partial updates:

```java
@PatchMapping("/{id}")
public Post patchPost(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    return postService.partialUpdate(id, updates);
}
```

## Spring MVC Annotations

### @PathVariable
Extracts values from the URL path:

```java
@GetMapping("/posts/{postId}/comments/{commentId}")
public Comment getComment(
    @PathVariable Long postId,
    @PathVariable Long commentId
) {
    return commentService.getComment(postId, commentId);
}

// Using custom name
@GetMapping("/authors/{authorId}")
public Author getAuthor(@PathVariable("authorId") Long id) {
    return authorService.getAuthorById(id);
}
```

### @RequestParam
Extracts query parameters from the URL:

```java
// GET /api/v1/posts?page=2&size=10&sort=title
@GetMapping("/posts")
public List<Post> getPosts(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(required = false) String sort
) {
    return postService.getPosts(page, size, sort);
}
```

### @RequestBody
Binds the HTTP request body to a method parameter (deserializes JSON to Java object):

```java
@PostMapping("/posts")
public Post createPost(@RequestBody Post post) {
    // Spring automatically converts JSON to Post object
    return postService.createPost(post);
}
```

### @ResponseStatus
Sets the HTTP status code for the response:

```java
@PostMapping("/posts")
@ResponseStatus(HttpStatus.CREATED) // Returns 201
public Post createPost(@RequestBody Post post) {
    return postService.createPost(post);
}

@DeleteMapping("/posts/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT) // Returns 204
public void deletePost(@PathVariable Long id) {
    postService.deletePost(id);
}
```

## Using ResponseEntity

`ResponseEntity` provides full control over the HTTP response, including status codes, headers, and body. 
It's more flexible than `@ResponseStatus`.

### Basic Usage

```java
@GetMapping("/{id}")
public ResponseEntity<Post> getPost(@PathVariable Long id) {
    Post post = postService.getPostById(id);
    if (post == null) {
        return ResponseEntity.notFound().build(); // 404
    }
    return ResponseEntity.ok(post); // 200
}
```

### Creating Resources with Location Header

```java
@PostMapping
public ResponseEntity<Post> createPost(@RequestBody Post post) {
    Post created = postService.createPost(post);

    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(created.getId())
        .toUri();

    return ResponseEntity.created(location).body(created); // 201 with Location header
}
```

### Custom Headers

```java
@GetMapping("/{id}")
public ResponseEntity<Post> getPost(@PathVariable Long id) {
    Post post = postService.getPostById(id);

    return ResponseEntity.ok()
        .header("X-Custom-Header", "CustomValue")
        .header("X-Post-Author", post.getAuthor())
        .body(post);
}
```

### Different Status Codes

```java
@PutMapping("/{id}")
public ResponseEntity<Post> updatePost(
    @PathVariable Long id,
    @RequestBody Post post
) {
    try {
        Post updated = postService.updatePost(id, post);
        return ResponseEntity.ok(updated); // 200
    } catch (PostNotFoundException e) {
        return ResponseEntity.notFound().build(); // 404
    }
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> deletePost(@PathVariable Long id) {
    boolean deleted = postService.deletePost(id);
    if (deleted) {
        return ResponseEntity.noContent().build(); // 204
    }
    return ResponseEntity.notFound().build(); // 404
}
```

### Conditional Responses

```java
@GetMapping("/posts")
public ResponseEntity<List<Post>> getPosts(@RequestParam(required = false) String author) {
    List<Post> posts = postService.getPostsByAuthor(author);

    if (posts.isEmpty()) {
        return ResponseEntity.noContent().build(); // 204
    }

    return ResponseEntity.ok()
        .cacheControl(CacheControl.maxAge(60, TimeUnit.SECONDS))
        .body(posts);
}
```

## Validation
Jakarta Validation is a framework for validating Java beans.
To use Jakarta Validation, add the following Spring Boot starter dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Now add the necessary annotations to the request body POJO:

```java
class CreatePostRequest {
    @NotBlank(message = "Title is required")
    @Size(min = 5, max = 100, message = "Title must be between 5 and 100 characters")
    private String title;
    
    @NotBlank(message = "Content is required")
    private String content;
}
```

Now we can use `@Valid` annotation to validate the request body:

```java
@PostMapping
public Post createPost(@Valid @RequestBody CreatePostRequest request) {
    //...
}
```

## Error Handling
If a request handling method in a controller throws an Exception, 
then Spring Boot will handle it and return the response using its default Exception Handling mechanism.

We can handle exceptions in different ways, and depending on your use-case, you can choose one of the approaches that fit best for you.

- Using Controller level @ExceptionHandler
- GlobalExceptionHandler using @RestControllerAdvice
- Spring's ProblemDetails for HTTP APIs (RFC 9457).

### Using Controller level @ExceptionHandler

```java
@RestController
@RequestMapping("/api/v1/posts")
class PostController {

    //...
    //...

    @PostMapping
    ResponseEntity<Post> create(@RequestBody @Valid CreatePostRequest request) {
        //logic that might throw PostSlugAlreadyExistsException
    }

    @PutMapping("/{id}")
    ResponseEntity<Post> update(@PathVariable Long id, @RequestBody @Valid UpdatePostRequest request) {
        //logic that might throw PostSlugAlreadyExistsException
    }
    
    @ExceptionHandler(PostSlugAlreadyExistsException.class)
    public ResponseEntity<ApiError> handle(PostSlugAlreadyExistsException e) {
        ApiError error = new ApiError(e.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

In this approach, you don't have to duplicate the exception handling logic in multiple handler methods in the controller. 
If `PostSlugAlreadyExistsException` is thrown from `create(…)` or `update(…)` methods, 
they will be handled by the respective `@ExceptionHandler` methods.

### GlobalExceptionHandler using @RestControllerAdvice
What if the same type of exceptions may occur in different Controllers, and we want to handle those Exceptions in the same way? 
In such cases, we can use the Global Exception Handling approach by using `@RestControllerAdvice`.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(PostNotFoundException.class)
    public ResponseEntity<ApiError> handle(PostNotFoundException e) {
        ApiError error = new ApiError(e.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(PostSlugAlreadyExistsException.class)
    public ResponseEntity<ApiError> handle(PostSlugAlreadyExistsException e) {
        ApiError error = new ApiError(e.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

By using ControllerAdvice approach, we don't have to duplicate the same `@ExceptionHandler` logic in multiple Controllers.

**IMPORTANT:** If you have an `@ExceptionHandler` handling the same Exception in both Controller 
and `GlobalExceptionHandler` then Controller level `@ExceptionHandler` method takes priority.

### Using ProblemDetails API
Spring Framework 6 implemented the Problem Details for HTTP APIs specification, RFC 9457.

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
    @ExceptionHandler(PostNotFoundException.class)
    ProblemDetail handle(PostNotFoundException e) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, e.getMessage());
        problemDetail.setTitle("Post Not Found");
        problemDetail.setType(URI.create("https://api.jblogger.com/errors/not-found"));
        return problemDetail;
    }

    @ExceptionHandler(PostSlugAlreadyExistsException.class)
    ProblemDetail handle(PostSlugAlreadyExistsException e) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, e.getMessage());
        problemDetail.setTitle("Post Slug Already Exists");
        problemDetail.setType(URI.create("https://api.jblogger.com/errors/bad-request"));
        return problemDetail;
    }
}
```

Now when you make the API call to fetch post with non-existing id then you will get the following response:

```json
{
  "type": "https://api.jblogger.com/errors/not-found",
  "title": "Post Not Found",
  "status": 404,
  "detail": "Post with id: 111 not found",
  "instance": "/api/posts/111"
}
```

In addition to the standard fields type, title, status, detail, instance we can also include custom properties as follows:

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(PostNotFoundException.class)
    ProblemDetail handle(PostNotFoundException e) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, e.getMessage());
        problemDetail.setTitle("Post Not Found");
        problemDetail.setType(URI.create("https://api.jblogger.com/errors/not-found"));
        problemDetail.setProperty("errorCategory", "Generic");
        problemDetail.setProperty("timestamp", Instant.now());
        return problemDetail;
    }
}
```

**NOTE:** It is a good practice to always handle all types of exceptions and return a standard response using Problem Details API.

## Assignment
Implement the following REST API endpoints for the Blog application using Spring MVC.

- Get all posts
- Get a post by id
- Create a new post
- Update a post
- Delete a post

### Post
Use the following Post class for the data model:

```java
class Post {
    private Long id;
    private String slug;
    private String title;
    private String content;
    
    // Getters and setters
}
```

### PostRepository
Use the following PostRepository using InMemory HashMap as data store:

```java
class PostRepository {
    private static final AtomicLong counter = new AtomicLong();
    private Map<Long, Post> posts = new HashMap<>();
    
    public List<Post> getAllPosts() {
        return new ArrayList<>(posts.values());
    }
    
    public Post getPostById(Long id) {
        return posts.get(id);
    }
    
    public Post createPost(Post post) {
        post.setId(counter.incrementAndGet());
        posts.put(post.getId(), post);
        return post;
    }
    
    public Post updatePost(Long id, Post post) {
        post.setId(id);
        posts.put(id, post);
    }
    
    public boolean deletePost(Long id) {
        return posts.remove(id) != null;
    }
}
```


# Spring JdbcClient

## Table of Contents
- [Overview](#overview)
- [Working with H2 Database](#working-with-h2-database)
- [Database Initialization](#database-initialization)
- [Using Spring JdbcClient](#using-spring-jdbcclient)
- [Adding PostgreSQL Database Support](#adding-postgresql-database-support)
- [Assignment](#assignment)

## Overview

Spring's JDBC support simplifies working with SQL databases by eliminating boilerplate code commonly required with traditional JDBC. 
Instead of manually managing database connections, creating statements, handling result sets, and dealing with exception handling, 
Spring provides higher-level abstractions like `JdbcClient` (introduced in Spring Framework 6.1) 
and `JdbcTemplate` that handle these concerns automatically. 
This allows developers to focus on writing SQL queries and mapping results to domain objects, 
while Spring manages resource management, exception translation, and connection pooling behind the scenes.

## Working with H2 Database

H2 is an embedded in-memory database that's good enough for development and testing. 
Spring Boot automatically configures all necessary beans to interact with H2 when it detects the H2 dependency on the classpath.

### Adding H2 Dependency
Add the JDBC starter and H2 database driver dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-h2console</artifactId>
</dependency>
```

### Auto-Configuration

Spring Boot automatically configures the following beans when JDBC starter and H2 are on the classpath:

- **DataSource**: Connection pool for managing database connections
- **JdbcClient/JdbcTemplate**: For executing SQL queries
- **DataSourceTransactionManager**: For transaction management
- **H2 Console**: Web-based database console (accessible at `/h2-console` when enabled)

### Configuration Properties

Configure H2 in your `application.properties`:

```properties
# Enable H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

With this configuration, you can access the H2 console at `http://localhost:8080/h2-console` and connect using the JDBC URL `jdbc:h2:mem:testdb`.

## Database Initialization

Spring Boot provides automatic database initialization using SQL script files. 
This is useful for creating schema and loading initial data during application startup.

### Using schema.sql

Create a `schema.sql` file in `src/main/resources` to define your database schema:

```sql
DROP TABLE IF EXISTS bookmarks;

create table bookmarks
(
    id         bigserial primary key,
    title      varchar   not null,
    url        varchar   not null,
    created_at timestamp default now()
);
```

### Using data.sql

Create a `data.sql` file in `src/main/resources` to populate initial data:

```sql
DELETE FROM bookmarks;

INSERT INTO bookmarks(title, url) VALUES
('How (not) to ask for Technical Help?','https://teste.com/how-to-not-to-ask-for-technical-help'),
('Getting Started with Kubernetes','https://teste.com/getting-started-with-kubernetes'),
('Few Things I learned in the HardWay in 15 years of my career','https://teste.com/few-things-i-learned-the-hardway-in-15-years-of-my-career'),
('All the resources you ever need as a Java & Spring application developer','https://teste.com/all-the-resources-you-ever-need-as-a-java-spring-application-developer'),
('SpringBoot Integration Testing using Testcontainers Starter','https://teste.com/spring-boot-integration-testing-using-testcontainers-starter'),
('Testing SpringBoot Applications','https://teste.com/spring-boot-testing')
;
```

### Configuration

Control database initialization behavior in `application.properties`:

```properties
# Database initialization mode
# always: Always initialize (default for embedded databases)
# never: Never initialize
# embedded: Only initialize embedded databases
spring.sql.init.mode=always
```

For platform-specific initialization, you can create files like `schema-h2.sql`, `schema-postgresql.sql`, `data-h2.sql`, etc.

### Using Spring JdbcClient
Spring framework 6.1 introduced a new **JdbcClient** API, which is a wrapper on top of **JdbcTemplate**, 
for performing database operations using a fluent API.

**NOTE:** **JdbcClient** is recommended over **JdbcTemplate** for new projects.

Let's explore how we can use **JdbcClient** to perform various database operations.

Create a Bookmark domain class.

```java
import java.time.Instant;

public record Bookmark(
        Long id, 
        String title, 
        String url, 
        Instant createdAt
) {}
```

Let's implement CRUD operations on Bookmark domain class using JdbcClient API.

```java
@Repository
public class BookmarkRepository {
    private final JdbcClient jdbcClient;

    public BookmarkRepository(JdbcClient jdbcClient) {
        this.jdbcClient = jdbcClient;
    }
    //...
    // ...
}
```

**Fetch all bookmarks:**

```java
public List<Bookmark> findAll() {
    String sql = "select id, title, url, created_at from bookmarks";
    return jdbcClient.sql(sql).query(new BookmarkRowMapper()).list();
}

static class BookmarkRowMapper implements RowMapper<Bookmark> {
    @Override
    public Bookmark mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Bookmark(
                rs.getLong("id"),
                rs.getString("title"),
                rs.getString("url"),
                rs.getTimestamp("created_at").toInstant()
        );
    }
}     
```

We can also simplify this as follows:

```java 
public List<Bookmark> findAll() {
    String sql = "select id, title, url, created_at from bookmarks";
    return jdbcClient.sql(sql).query(Bookmark.class).list();
}
```

The **JdbcClient** API will take care of dynamically creating a **RowMapper** by using **SimplePropertyRowMapper**. 
It will perform the mapping between bean property names to table column names by converting camelCase to underscore notation.

**Find bookmark By ID:**

We can fetch a bookmark by id using JdbcClient as follows:

```java
public Optional<Bookmark> findById(Long id) {
    String sql = "select id, title, url, created_at from bookmarks where id = :id";
    return jdbcClient.sql(sql).param("id", id).query(Bookmark.class).optional();
    
    // If you want to use your own RowMapper
    //return jdbcClient.sql(sql).param("id", id).query(new BookmarkRowMapper()).optional();
}
```

**Insert a bookmark:**

We can insert a new row into the bookmarks table and get the generated primary key value as follows:

```java
public Long save(Bookmark bookmark) {
    String sql = "insert into bookmarks(title, url) values(:title,:url)";
    KeyHolder keyHolder = new GeneratedKeyHolder();
    jdbcClient.sql(sql)
                .param("title", bookmark.title())
                .param("url", bookmark.url())
                .update(keyHolder);
    return (Long) keyHolder.getKeys().get("ID");
}
```

**Update a bookmark:**

We can update a bookmark as follows:

```java
public void update(Bookmark bookmark) {
    String sql = "update bookmarks set title = ?, url = ? where id = ?";
    int count = jdbcClient.sql(sql)
            .param(1, bookmark.title())
            .param(2, bookmark.url())
            .param(3, bookmark.id())
            .update();
    if (count == 0) {
        throw new RuntimeException("Bookmark not found");
    }
}
```

**Delete a bookmark:**

We can delete a bookmark as follows:

```java
public void delete(Long id) {
    String sql = "delete from bookmarks where id = ?";
    int count = jdbcClient.sql(sql).param(1, id).update();
    if (count == 0) {
        throw new RuntimeException("Bookmark not found");
    }
}
```

## Adding PostgreSQL Database Support

For production environments, you'll typically want to use a more robust database like PostgreSQL or MySQL.

Let's replace H2 with PostgreSQL in this guide.

### Adding PostgreSQL Dependency

Add the PostgreSQL driver to your `pom.xml`:

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

### PostgreSQL Configuration

In the **Getting Started** section, we have started the PostgreSQL server using Docker.

Configure PostgreSQL connection in `application.properties`:

```properties
# PostgreSQL Configuration
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=postgres
```

When we explicitly configure the JDBC DataSource properties, 
Spring Boot will automatically configure DataSource using the provided properties instead of using the H2 in-memory database.

## Assignment

- In the previous section, we have implemented REST API endpoints but stored data in **HashMap**.
- Let's store data in a relational database using Spring **JdbcClient**.
- Create **schema.sql** and **data.sql** scripts to create and initialize **posts** table.
- Update **PostRepository** to use **JdbcClient** and persist data in the database instead of HashMap.


# Database Transaction Management

## Table of Contents
- [Transaction Management using plain JDBC](#transaction-management-using-plain-jdbc)
- [Declarative Transaction Management using @Transactional annotation](#declarative-transaction-management-using-transactional-annotation)
- [Programmatic Transaction Management using TransactionTemplate](#programmatic-transaction-management-using-transactiontemplate)
- [Assignment](#assignment)

A database transaction is a single unit of work, which either completes fully or does not complete at all and leaves the database in a consistent state. 
While implementing database transactions, you need to take ACID (Atomicity, Consistency, Isolation, Durability) properties into consideration.

Let's understand how we can handle database transactions in our Spring Boot applications.

## Transaction Management using plain JDBC

First, let's take a quick look at how we usually handle database transactions in plain JDBC.

```java
class UserService {
    
    void register(User user) {
        String sql = "...";
        Connection conn = dataSource.getConnection(); // <1>
        try(conn) {  // <6>
            conn.setAutoCommit(false);  // <2>
            PreparedStatement pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, user.getEmail());
            pstmt.setString(2, user.getPassword());
            pstmt.executeUpdate();  // <3>
            conn.commit();  // <4>
        } catch(SQLException e) {
            conn.rollback();  // <5>
        }
    }
}
```

In the above code snippet:

* (1) We got a database connection from the DataSource connection pool
* (2) By default, every database operation will be running in its own transaction automatically. 
      To take control of defining our transaction scope, we set AutoCommit to false.
* (3) Then we performed the necessary database operations.
* (4) If everything works fine, then we are committing the transaction.
* (5) If any exception occurred, then we roll back the transaction.
* (6) As we are using try-with-resources syntax, the connection will be automatically closed.

In this implementation there is a lot of boilerplate code to handle database transaction.

Spring simplifies the database transaction handling mechanism by using Aspect Oriented Programming (AOP) behind the scenes. 

We can implement database transaction handling in a declarative approach using **@Transactional** annotation 
or programmatically using **TransactionTemplate**.

## Declarative Transaction Management using @Transactional annotation
Typically, in a 3-tier layered architecture, we will have a Controller in the web layer, 
Service in the business logic layer and Repository in the data access layer.

Generally, a "Unit Of Work" is modeled as a Service method and is considered to be a "transactional boundary".

We can add Spring's **@Transactional** annotation on a Service layer method to define the transaction scope. 
All the database operations in that method will be running in a single transaction. 

If the method is executed successfully, then the transaction will be committed. 
If any **RuntimeException** occurs during the method execution, then the transaction will be rolled back.

```java
@Service
class UserService {
    private final JdbcClient jdbcClient;
    
    @Transactional
    void register(User user) {
        String insertSql = "...";
        jdbcClient.sql(insertSql).update();
        String updateSql = "...";
        jdbcClient.sql(updateSql).update();
    }
}
```

- You can add **@Transactional** annotation on a method level to make that particular method transactional, 
  or at class level to make all the public methods in that class as transactional.
- When you add **@Transactional** annotation on a class or on any of its methods, 
- Spring creates a Proxy for that class and applies the transaction handling logic using Spring AOP behind the scenes.

When you add **@Transactional** annotation, by default:

- The propagation level is set to REQUIRED which participates in the current transaction if it exists or starts a new transaction.
- Rolls back the transaction if any RuntimeException is thrown by the method.
- Does NOT roll back the transaction if the method throws any Checked Exception.

You can customize this behavior by specifying the **@Transactional** attributes as follows:

```java
@Service
class UserService {
    private final JdbcClient jdbcClient;
    
    @Transactional(
            propagation = Propagation.REQUIRES_NEW,
            rollbackFor = DuplicateUserException.class,
            noRollbackFor = IllegalArgumentException.class)
    void register(User user) throws DuplicateUserException {
        String sql = "...";
        jdbcClient.sql(sql).update();
    }
}

class DuplicateUserException extends Exception {}
```

In the above code snippet, we specified propagation level to be **REQUIRES_NEW** instead of the default **REQUIRED**. 
The **REQUIRES_NEW** propagation level suspends the current transaction if one exists, starts a new transaction, 
executes the database operations, commits the transaction and then resumes the previously suspended transaction.

Also, we specified to rollback the transaction if **DuplicateUserException** occurs even though it is a Checked Exception. 

And, we specified not to rollback the transaction if **IllegalArgumentException** occurs even though it is a RuntimeException.

## Programmatic Transaction Management using TransactionTemplate
Spring also provides a programmatic approach to handling database transactions using **TransactionTemplate** as follows:

```java
@Service
class UserService {
    private final JdbcClient jdbcClient;
    private final TransactionTemplate transactionTemplate;

    void register(User user) {
        transactionTemplate.execute(status -> {
            String sql = "...";
            jdbcClient.sql(sql).update();
            
            // If anything goes wrong, rollback
            //status.setRollbackOnly();

            return result;
        });
    }
}
```

## Assignment
- Add transactional handling support to **PostRepository** using **@Transactional** annotation.
- In the **PostRepository** class **createPost()** method first insert the post record and check if post content has any swear words and throw a **RuntimeException** if it does.
  Verify that the post is not inserted in the database if the RuntimeException is thrown.


# Flyway Database Migrations

## Table of Contents
- [Create Flyway Migration Scripts](#create-flyway-migration-scripts)
- [Customizing Flyway Configuration](#customizing-flyway-configuration)
- [Java based Flyway Migrations](#java-based-flyway-migrations)
- [Assignment](#assignment)

In the JdbcClient section, we have seen how to initialize the database using **schema.sql** and **data.sql** scripts.
This may be useful for demos and quick prototypes, but for real-world applications we should use a database migration tool.

[Flyway](https://flywaydb.org/) is one of the most popular Java-based database migration libraries.
Spring Boot provides out-of-the-box support for Flyway database migrations.

**Why should we use a database migration tool?**

- Versioned, immutable migrations
- Each migration runs exactly once
- Schema history stored in the database
- Safe locking and fail-fast behavior
- CI/CD and production-ready by design

Let us see how we can use Flyway for implementing database migrations in a Spring Boot application that uses PostgreSQL database.

Add the following dependencies to `pom.xml`:

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-flyway</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

## Create Flyway Migration Scripts
Flyway follows the `V<VERSION>__<DESCRIPTION>.sql` (Capital V, version number, 2 underscores, description) naming convention for its versioned migration scripts.

Let's add the following two migration scripts under **src/main/resources/db/migration** folder.

**V1__create_tables.sql**
```sql
create table bookmarks
(
    id         bigserial not null,
    title      varchar   not null,
    url        varchar   not null,
    created_at timestamp,
    primary key (id)
);
```

**V2__create_bookmarks_indexes.sql**
```sql
CREATE INDEX idx_bookmarks_title ON bookmarks(title);
```

Now when you start the application, you should notice the following Flyway execution related logs as follows:

```shell
INFO 4009 --- [           main] o.f.c.i.database.base.BaseDatabaseType   : Database: jdbc:postgresql://localhost:5432/postgres (PostgreSQL 18)
INFO 4009 --- [           main] o.f.c.i.s.JdbcTableSchemaHistory         : Schema history table "public"."flyway_schema_history" does not exist yet
INFO 4009 --- [           main] o.f.core.internal.command.DbValidate     : Successfully validated 2 migrations (execution time 00:00.010s)
INFO 4009 --- [           main] o.f.c.i.s.JdbcTableSchemaHistory         : Creating Schema History table "public"."flyway_schema_history" ...
INFO 4009 --- [           main] o.f.core.internal.command.DbMigrate      : Current version of schema "public": << Empty Schema >>
INFO 4009 --- [           main] o.f.core.internal.command.DbMigrate      : Migrating schema "public" to version "1 - create tables"
INFO 4009 --- [           main] o.f.core.internal.command.DbMigrate      : Migrating schema "public" to version "2 - create bookmarks indexes"
INFO 4009 --- [           main] o.f.core.internal.command.DbMigrate      : Successfully applied 2 migrations to schema "public", now at version v2 (execution time 00:00.041s)
```

Flyway keeps track of all the applied migrations history in **flyway_schema_history** table by default.
If you check the data in **flyway_schema_history** table now, you can see the following rows:

```shell
| installed_rank | version | description              | type | script                           | checksum   | installed_by | installed_on               | execution_time | success |
|:---------------|:--------|:-------------------------|:-----|:---------------------------------|:-----------|:-------------|:---------------------------|:---------------|:--------|
| 1              | 1       | create tables            | SQL  | V1__create_tables.sql            | 1020037327 | test         | 2026-01-06 09:13:04.439012 | 6              | true    |
| 2              | 2       | create bookmarks indexes | SQL  | V2__create_bookmarks_indexes.sql | 732086927  | test         | 2026-01-06 09:13:04.456876 | 4              | true    |
```

If you keep the same database instance running and restart the application, then Flyway doesn't re-run the already applied migrations.
If you have added any new migrations, then only those migration scripts will be executed.


**Important Flyway Rules:**

The following rules must be followed, otherwise Flyway will throw error while applying migrations:

* There should be no duplicate version numbers in flyway migration script file names.

  Ex: **V1__init.sql**, **V1__indexes.sql**. Here version number 1 is used multiple times.
* Once a migration is applied, you should not change its content.


## Customizing Flyway Configuration
Spring Boot provides sensible defaults for Flyway migrations,
but you can configure various Flyway configuration properties
using **spring.flyway.{property-name}** properties in **application.properties** file.

### Flyway migrations for different databases
If you are building a product which can be used with different databases, then you can configure flyway migration locations as follows:

```properties
spring.flyway.locations=classpath:db/migration/{vendor}
```

Then you can place **mysql** specific scripts under **src/main/resources/db/migration/mysql** directory,
**postgresql** specific scripts under **src/main/resources/db/migration/postgresql**, etc.
You can check the vendor names in **org.springframework.boot.jdbc.DatabaseDriver** class.

In addition to this, some interesting flyway configuration properties are:

```properties
# disable flyway execution
spring.flyway.enabled=false

# If you have an existing database and start using Flyway for new database changes.
spring.flyway.baseline-on-migrate=true

# To customize flyway migrations tracking table name
spring.flyway.table=db_migrations
```

## Java based Flyway Migrations
In addition to SQL scripts, you can also write database migrations using Java classes.

You can create **V3__InsertSampleData.java** in **db.migration** package as follows:

```java
package db.migration;

import org.flywaydb.core.api.migration.BaseJavaMigration;
import org.flywaydb.core.api.migration.Context;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.SingleConnectionDataSource;

public class V3__InsertSampleData extends BaseJavaMigration {

  public void migrate(Context context) {
      JdbcClient jdbcClient = JdbcClient.create(
              new SingleConnectionDataSource(context.getConnection(), true));

      jdbcClient.sql("...").update();
  }
}
```

If the database changes involve complex logic which is difficult to write using plain SQL,
then Java-based migrations come very handy.

You can use Flyway Database Migrations with different persistence technologies
such as **JdbcClient**, **Spring Data Jdbc**, **Spring Data JPA**, **jOOQ**, etc.

## Assignment
- Remove the **schema.sql** and **data.sql** scripts from the application.
- Use Flyway database migrations to create **users**, **posts**, **comments** tables and insert sample data.

# Spring Data JPA

## Table of Contents
- [Main Features of Spring Data JPA](#main-features-of-spring-data-jpa)
- [Setup](#setup)
- [Creating the Entity](#creating-the-entity)
- [CRUD Operations with Spring Data JPA](#crud-operations-with-spring-data-jpa)
- [Sorting and Pagination](#sorting-and-pagination)
- [Custom Finder Methods](#custom-finder-methods)
- [Spring Data JPA Projections](#spring-data-jpa-projections)
- [Spring Data JPA Auditing](#spring-data-jpa-auditing)
- [Summary](#summary)
- [Assignment](#assignment)

**Spring Data JPA** is a part of the Spring Data family that simplifies data access using the Java Persistence API (JPA). 
It reduces boilerplate code by providing repository abstractions and eliminates the need to write repetitive CRUD operations manually.

## Main Features of Spring Data JPA

- **Automatic CRUD Operations**: Provides built-in methods like `save()`, `findById()`, `findAll()`, `delete()` without writing implementation code
- **Query Methods**: Automatically generates queries from method names (e.g., `findByTitle()`, `findByCreatedAtBefore()`)
- **Pagination and Sorting**: Built-in support for paginating and sorting query results
- **Custom Queries**: Write custom queries using JPQL or native SQL with `@Query` annotation
- **Auditing**: Automatically track creation and modification timestamps and users
- **Projections**: Fetch only specific fields instead of entire entities

In this guide, we'll use the **bookmarks** example from the Flyway section to demonstrate Spring Data JPA features.

## Setup

Add the Spring Data JPA dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

Configure database connection in `application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=postgres

spring.jpa.show-sql=true
```

## Creating the Entity

Create a `Bookmark` entity class that maps to the `bookmarks` table:

```java
package com.dgw.jblogger.domain;

import jakarta.persistence.*;
import java.time.Instant;

@Entity
@Table(name = "bookmarks")
public class Bookmark {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String url;

    @Column(name = "created_at")
    private Instant createdAt;

    // Constructors
    protected Bookmark() {
    }

    public Bookmark(String title, String url) {
        this.title = title;
        this.url = url;
        this.createdAt = Instant.now();
    }

    // Getters and Setters
}
```

**Key Annotations:**
- `@Entity`: Marks the class as a JPA entity
- `@Table`: Specifies the table name (optional if table name matches class name)
- `@Id`: Marks the primary key field
- `@GeneratedValue`: Specifies how the primary key is generated
- `@Column`: Maps the field to a database column

## CRUD Operations with Spring Data JPA

Create a repository interface by extending `JpaRepository`:

```java
package com.dgw.jblogger.repository;

import com.dgw.jblogger.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {
    // No code needed! Spring Data JPA provides implementations automatically
}
```

That's it! Spring Data JPA automatically provides the following methods:

- `save(entity)` - Create or update
- `findById(id)` - Find by ID
- `findAll()` - Find all entities
- `deleteById(id)` - Delete by ID
- `count()` - Count all entities
- And many more...

### Using the Repository

Create a service class to demonstrate CRUD operations:

```java
package com.dgw.jblogger.service;

import com.dgw.jblogger.domain.Bookmark;
import com.dgw.jblogger.repository.BookmarkRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@Transactional
public class BookmarkService {

    private final BookmarkRepository bookmarkRepository;

    public BookmarkService(BookmarkRepository bookmarkRepository) {
        this.bookmarkRepository = bookmarkRepository;
    }

    // Create
    public Bookmark createBookmark(Bookmark bookmark) {
        return bookmarkRepository.save(bookmark);
    }

    // Read - Find by ID
    public Optional<Bookmark> getBookmarkById(Long id) {
        return bookmarkRepository.findById(id);
    }

    // Read - Find all
    public List<Bookmark> getAllBookmarks() {
        return bookmarkRepository.findAll();
    }

    // Update
    public Bookmark updateBookmark(Long id, Bookmark updatedBookmark) {
        return bookmarkRepository.findById(id)
            .map(bookmark -> {
                bookmark.setTitle(updatedBookmark.getTitle());
                bookmark.setUrl(updatedBookmark.getUrl());
                return bookmarkRepository.save(bookmark);
            })
            .orElseThrow(() -> new RuntimeException("Bookmark not found with id: " + id));
    }

    // Delete
    public void deleteBookmark(Long id) {
        bookmarkRepository.deleteById(id);
    }

    // Count
    public long countBookmarks() {
        return bookmarkRepository.count();
    }
}
```

## Sorting and Pagination

Spring Data JPA provides excellent support for sorting and pagination.

### Sorting

```java
import org.springframework.data.domain.Sort;

@Service
@Transactional
public class BookmarkService {

    private final BookmarkRepository bookmarkRepository;

    // ... constructor

    // Sort by title ascending
    public List<Bookmark> getAllBookmarksSortedByTitle() {
        return bookmarkRepository.findAll(Sort.by(Sort.Direction.ASC, "title"));
    }

    // Sort by created date descending
    public List<Bookmark> getAllBookmarksSortedByDate() {
        return bookmarkRepository.findAll(Sort.by(Sort.Direction.DESC, "createdAt"));
    }

    // Sort by multiple fields
    public List<Bookmark> getAllBookmarksSortedByMultipleFields() {
        Sort sort = Sort.by(
            Sort.Order.desc("createdAt"),
            Sort.Order.asc("title")
        );
        return bookmarkRepository.findAll(sort);
    }
}
```

### Pagination

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

@Service
@Transactional
public class BookmarkService {

    private final BookmarkRepository bookmarkRepository;

    // ... constructor

    // Get paginated results
    public Page<Bookmark> getBookmarksPaginated(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return bookmarkRepository.findAll(pageable);
    }

    // Pagination with sorting
    public Page<Bookmark> getBookmarksPaginatedAndSorted(int page, int size) {
        Pageable pageable = PageRequest.of(
            page,
            size,
            Sort.by(Sort.Direction.DESC, "createdAt")
        );
        return bookmarkRepository.findAll(pageable);
    }
}
```

The `Page` object contains useful information:
- `getContent()` - List of entities on current page
- `getTotalElements()` - Total number of elements
- `getTotalPages()` - Total number of pages
- `getNumber()` - Current page number
- `hasNext()` - Whether there is a next page
- `hasPrevious()` - Whether there is a previous page

### Using Pagination in a Controller

```java
package com.dgw.jblogger.controller;

import com.dgw.jblogger.domain.Bookmark;
import com.dgw.jblogger.service.BookmarkService;
import org.springframework.data.domain.Page;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/bookmarks")
public class BookmarkController {

    private final BookmarkService bookmarkService;

    public BookmarkController(BookmarkService bookmarkService) {
        this.bookmarkService = bookmarkService;
    }

    @GetMapping
    public Page<Bookmark> getBookmarks(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return bookmarkService.getBookmarksPaginatedAndSorted(page, size);
    }
}
```

Access the endpoint: `http://localhost:8080/api/v1/bookmarks?page=0&size=10`

**NOTE:** Instead of directly returning a `Page<T>` object, map it to a custom DTO by exposing only the necessary fields.

## Custom Finder Methods

Spring Data JPA can automatically generate queries from method names.

### Using findBy...() Methods

Add these methods to your `BookmarkRepository`:

```java
package com.dgw.jblogger.repository;

import com.dgw.jblogger.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;

import java.time.LocalDateTime;
import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // Find by title (exact match)
    List<Bookmark> findByTitle(String title);

    // Find by title containing (case-insensitive)
    List<Bookmark> findByTitleContainingIgnoreCase(String title);

    // Find by URL
    List<Bookmark> findByUrl(String url);

    // Find by created date after
    List<Bookmark> findByCreatedAtAfter(LocalDateTime date);

    // Find by created date between
    List<Bookmark> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    // Find by title and URL
    List<Bookmark> findByTitleAndUrl(String title, String url);

    // Find by title or URL
    List<Bookmark> findByTitleContainingOrUrlContaining(String title, String url);

    // Find first 5 by title containing, ordered by created date descending
    List<Bookmark> findTop5ByTitleContainingOrderByCreatedAtDesc(String title);

    // Check if exists by title
    boolean existsByTitle(String title);

    // Count by title containing
    long countByTitleContaining(String title);
}
```

**Common Keywords:**
- `findBy`, `readBy`, `getBy`, `queryBy`
- `countBy`, `existsBy`, `deleteBy`
- `Containing`, `StartingWith`, `EndingWith`
- `IgnoreCase`, `AllIgnoreCase`
- `And`, `Or`
- `GreaterThan`, `LessThan`, `Between`
- `OrderBy...Asc`, `OrderBy...Desc`
- `First`, `Top`, `Distinct`

### Using @Query Annotation

For complex queries, use the `@Query` annotation with JPQL:

```java
package com.dgw.jblogger.repository;

import com.dgw.jblogger.domain.Bookmark;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDateTime;
import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // JPQL query with positional parameter
    @Query("SELECT b FROM Bookmark b WHERE b.title LIKE %?1%")
    List<Bookmark> searchByTitle(String title);

    // JPQL query with named parameter
    @Query("SELECT b FROM Bookmark b WHERE b.title LIKE %:keyword% OR b.url LIKE %:keyword%")
    List<Bookmark> searchByKeyword(@Param("keyword") String keyword);

    // JPQL query with pagination
    @Query("SELECT b FROM Bookmark b WHERE b.createdAt >= :date")
    Page<Bookmark> findRecentBookmarks(@Param("date") LocalDateTime date, Pageable pageable);

    // JPQL query with multiple parameters
    @Query("SELECT b FROM Bookmark b WHERE b.title = :title AND b.createdAt > :date")
    List<Bookmark> findByTitleAndDateAfter(
        @Param("title") String title,
        @Param("date") LocalDateTime date
    );
}
```

### Using Native SQL Queries

For database-specific queries, use native SQL:

```java
package com.dgw.jblogger.repository;

import com.dgw.jblogger.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // Native SQL query
    @Query(value = "SELECT * FROM bookmarks WHERE title ILIKE %:keyword%",
           nativeQuery = true)
    List<Bookmark> searchByTitleNative(@Param("keyword") String keyword);

    // Native SQL with pagination (PostgreSQL specific)
    @Query(value = "SELECT * FROM bookmarks ORDER BY created_at DESC LIMIT :limit OFFSET :offset",
           nativeQuery = true)
    List<Bookmark> findRecentBookmarksNative(@Param("limit") int limit, @Param("offset") int offset);

    // Native SQL for aggregation
    @Query(value = "SELECT COUNT(*) FROM bookmarks WHERE created_at >= CURRENT_DATE",
           nativeQuery = true)
    long countTodayBookmarks();
}
```

**Note:** Use `nativeQuery = true` for native SQL queries.

### Data Modifying Queries with @Modifying

For any data modification such as INSERT/UPDATE/DELETE operations, use `@Modifying` annotation:

```java
package com.dgw.jblogger.repository;

import com.dgw.jblogger.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDateTime;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // Update query
    @Modifying
    @Query("UPDATE Bookmark b SET b.title = :title WHERE b.id = :id")
    int updateTitle(@Param("id") Long id, @Param("title") String title);

    // Update multiple fields
    @Modifying
    @Query("UPDATE Bookmark b SET b.title = :title, b.url = :url WHERE b.id = :id")
    int updateBookmark(@Param("id") Long id, @Param("title") String title, @Param("url") String url);

    // Delete query
    @Modifying
    @Query("DELETE FROM Bookmark b WHERE b.createdAt < :date")
    int deleteOldBookmarks(@Param("date") LocalDateTime date);

    // Native update query
    @Modifying
    @Query(value = "UPDATE bookmarks SET title = UPPER(title) WHERE id = :id",
           nativeQuery = true)
    int uppercaseTitle(@Param("id") Long id);
}
```

## Spring Data JPA Projections

Projections allow you to fetch only specific fields instead of entire entities, improving performance.

### Interface-based Projections

Create an interface with getter methods for the fields you want:

```java
package com.dgw.jblogger.projection;

import java.time.LocalDateTime;

public interface BookmarkSummary {
    Long getId();
    String getTitle();
    Instant getCreatedAt();
    // URL is excluded
}
```

Use the projection in your repository:

```java
package com.dgw.jblogger.repository;

import com.dgw.jblogger.domain.Bookmark;
import com.dgw.jblogger.projection.BookmarkSummary;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // Returns only id, title, and createdAt
    List<BookmarkSummary> findAllBy();

    List<BookmarkSummary> findByTitleContaining(String title);
}
```

### Class-based Projections (DTOs)

Create a DTO class:

```java
package com.dgw.jblogger.dto;

public class BookmarkDTO {
    private Long id;
    private String title;

    public BookmarkDTO(Long id, String title) {
        this.id = id;
        this.title = title;
    }

    // Getters and setters
}
```

Use the DTO in a custom query:

```java
package com.dgw.jblogger.repository;

import com.dgw.jblogger.domain.Bookmark;
import com.dgw.jblogger.dto.BookmarkDTO;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    @Query("SELECT new com.dgw.jblogger.dto.BookmarkDTO(b.id, b.title) FROM Bookmark b")
    List<BookmarkDTO> findAllBookmarkDTOs();
}
```

### Dynamic Projections

Use generic type parameter to choose a projection at runtime:

```java
package com.dgw.jblogger.repository;

import com.dgw.jblogger.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    <T> List<T> findByTitleContaining(String title, Class<T> type);
}
```

Use it in your service:

```java
// Get full entities
List<Bookmark> fullBookmarks = bookmarkRepository.findByTitleContaining("Java", Bookmark.class);

// Get projections
List<BookmarkSummary> summaries = bookmarkRepository.findByTitleContaining("Java", BookmarkSummary.class);
```

## Spring Data JPA Auditing

Auditing automatically tracks who created/modified an entity and when.

### Enable Auditing

Add `@EnableJpaAuditing` to your main application class or configuration class:

```java
package com.dgw.jblogger.config;

import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

### Update the Entity

Add auditing fields to your entity:

```java
package com.dgw.jblogger.domain;

import jakarta.persistence.*;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.Instant;

@Entity
@Table(name = "bookmarks")
@EntityListeners(AuditingEntityListener.class)
public class Bookmark {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String url;

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private Instant createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private Instant updatedAt;

    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;

    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;

    // Constructors, getters, and setters...
}
```

**Key Annotations:**
- `@EntityListeners(AuditingEntityListener.class)` - Enables auditing for this entity
- `@CreatedDate` - Automatically set when entity is created
- `@LastModifiedDate` - Automatically updated when entity is modified
- `@CreatedBy` - User who created the entity
- `@LastModifiedBy` - User who last modified the entity

### Create Migration for Audit Fields

Create a new Flyway migration `V3__add_audit_columns.sql`:

```sql
ALTER TABLE bookmarks ADD COLUMN updated_at TIMESTAMP;
ALTER TABLE bookmarks ADD COLUMN created_by VARCHAR(255);
ALTER TABLE bookmarks ADD COLUMN updated_by VARCHAR(255);
```

### Provide AuditorAware Implementation

To track the user, implement `AuditorAware`:

```java
package com.dgw.jblogger.config;

import org.springframework.data.domain.AuditorAware;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class AuditorAwareImpl implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        // In a real application, get the current user from SecurityContext
        // For now, return a hardcoded value
        return Optional.of("system");

        // With Spring Security, you would do:
        // return Optional.ofNullable(SecurityContextHolder.getContext())
        //     .map(SecurityContext::getAuthentication)
        //     .filter(Authentication::isAuthenticated)
        //     .map(Authentication::getName);
    }
}
```

Now, whenever you save a bookmark, the audit fields will be automatically populated:

```java
Bookmark bookmark = new Bookmark("Spring Boot", "https://spring.io");
bookmarkRepository.save(bookmark);
// createdAt, updatedAt, createdBy, and updatedBy are automatically set
```

## Summary

Spring Data JPA simplifies database operations significantly:

1. **No boilerplate code**: Extend `JpaRepository` to get CRUD operations automatically
2. **Query methods**: Create queries just by defining method names
3. **Custom queries**: Use `@Query` for complex JPQL or native SQL
4. **Pagination and sorting**: Built-in support with `Pageable` and `Sort`
5. **Projections**: Fetch only the fields you need
6. **Auditing**: Automatically track creation and modification metadata

With Spring Data JPA, you can focus on business logic instead of writing repetitive data access code!

## Assignment
* Implement database interactions in jblogger application using Spring Data JPA
* Create a Flyway migration for audit fields
* Enable auditing in your application
* Provide an implementation of `AuditorAware`
* Verify that audit fields are automatically populated when saving or updating entities
* Add pagination to the find all bookmarks API endpoint


# Spring Security Basics

## Table of Contents
- [Web Application Security Overview](#web-application-security-overview)
- [Java Web Application Security](#java-web-application-security)
- [Spring Security Architecture](#spring-security-architecture)
- [Spring Security Key Components](#spring-security-key-components)
- [Spring Security in Web Applications](#spring-security-in-web-applications)
- [Method-Level Security](#method-level-security)
- [Assignments](#assignments)

## Web Application Security Overview

Security is a critical aspect of web applications. You need to protect your resources from unauthorized access 
and ensure that only authenticated users can access private resources.

### Public vs Private Resources

In most web applications, you have two types of resources:

**1. Public Resources** - Available to everyone without authentication
- Homepage
- About page
- Contact page
- Product listing pages
- Login page
- Registration page

**2. Private Resources** - Available only to authenticated users
- User profile
- Dashboard
- Shopping cart
- Order history
- Admin panel

**Real-World Example:**
Think of a shopping website like Amazon:
- Anyone can browse products (public)
- Only logged-in users can add to cart or make purchases (private)
- Only admins can manage products and orders (private + role-based)

```
┌─────────────────────────────────────────┐
│         Web Application                 │
├─────────────────────────────────────────┤
│  Public Resources                       │
│  ✓ /home        (Everyone)              │
│  ✓ /products    (Everyone)              │
│  ✓ /login       (Everyone)              │
├─────────────────────────────────────────┤
│  Private Resources                      │
│  ✓ /profile     (Authenticated Users)   │
│  ✓ /cart        (Authenticated Users)   │
│  ✓ /admin       (Admin Role Only)       │
└─────────────────────────────────────────┘
```

### Role-Based Access Control (RBAC)

RBAC is a method to control access to resources based on the user's role in the organization.

**Common Roles:**
- **USER** - Regular users with basic access
- **ADMIN** - Administrators with full access
- **MANAGER** - Managers with specific permissions
- **GUEST** - Limited access

**Example Scenario:**

```
User: John (Role: USER)
- Can view products ✓
- Can place orders ✓
- Can view own profile ✓
- Cannot access admin panel ✗

User: Sarah (Role: ADMIN)
- Can view products ✓
- Can place orders ✓
- Can view own profile ✓
- Can access admin panel ✓
- Can manage all users ✓
```

**RBAC Benefits:**
1. **Simplified Management** - Assign roles instead of individual permissions
2. **Scalability** - Easy to add new users with predefined roles
3. **Security** - Principle of least privilege (users get only what they need)
4. **Compliance** - Meet regulatory requirements

## Java Web Application Security

Before diving into Spring Security, let's understand how security works in traditional Java web applications.

### Servlets to Handle HTTP Requests

Servlets are Java classes that handle HTTP requests and responses.

**Example Servlet:**
```java
@WebServlet("/profile")
public class ProfileServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        // Check if user is logged in
        HttpSession session = request.getSession(false);
        if (session == null || session.getAttribute("user") == null) {
            response.sendRedirect("/login");
            return;
        }

        // User is authenticated, show profile
        response.getWriter().println("Welcome to your profile!");
    }
}
```

**Problems with Manual Security Checks:**
- Code duplication in every servlet
- Easy to forget security checks
- Hard to maintain
- No centralized security logic

### Filters to Intercept Requests

Filters are Java components that intercept requests before they reach servlets. They're perfect for implementing security checks.

**Filter Lifecycle:**
```
Browser → Filter → Servlet → Response
           ↓
    Security Check
```

**Example Security Filter:**
```java
@WebFilter("/secure/*")
public class AuthenticationFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        // Check if user is authenticated
        HttpSession session = httpRequest.getSession(false);
        if (session == null || session.getAttribute("user") == null) {
            // Not authenticated - redirect to login
            httpResponse.sendRedirect("/login");
            return;
        }

        // Authenticated - continue to the requested resource
        chain.doFilter(request, response);
    }
}
```

**How Filters Work:**

1. **Request arrives** at `/secure/profile`
2. **Filter intercepts** the request
3. **Filter checks** if user is authenticated
4. **If authenticated**: Pass request to servlet
5. **If not authenticated**: Redirect to login page

**Filter Pattern Matching:**
```java
@WebFilter("/admin/*")    // Protects all admin URLs
@WebFilter("/secure/*")   // Protects all secure URLs
@WebFilter("/*")          // Intercepts ALL requests
```

### Authentication and Authorization

**Authentication** - "Who are you?"
- Verifies the identity of the user
- Checks username and password
- Confirms user is who they claim to be

**Authorization** - "What can you do?"
- Determines what resources the user can access
- Checks user's roles and permissions
- Enforces access control rules

**Example Flow:**

```
┌──────────────────────────────────────────────────────┐
│ 1. User Login Request                                │
│    Username: john@example.com                        │
│    Password: secret123                               │
└────────────────┬─────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────┐
│ 2. Authentication (Who are you?)                     │
│    - Verify username exists                          │
│    - Check password matches                          │
│    - Load user roles                                 │
│    Result: ✓ Authenticated (Roles: USER, MANAGER)    │
└────────────────┬─────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────┐
│ 3. User tries to access /admin/users                 │
└────────────────┬─────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────┐
│ 4. Authorization (What can you do?)                  │
│    Required Role: ADMIN                              │
│    User Roles: USER, MANAGER                         │
│    Result: ✗ Access Denied (Missing ADMIN role)      │
└──────────────────────────────────────────────────────┘
```

**Traditional Java Implementation:**
```java
public class SecurityHelper {

    // Authentication
    public User authenticate(String username, String password) {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new AuthenticationException("User not found");
        }

        if (!passwordMatches(password, user.getPassword())) {
            throw new AuthenticationException("Invalid password");
        }

        return user;
    }

    // Authorization
    public boolean authorize(User user, String requiredRole) {
        return user.getRoles().contains(requiredRole);
    }
}
```

**Limitations of Manual Implementation:**
- Complex code to maintain
- Security vulnerabilities if not done correctly
- Need to handle sessions, cookies, CSRF tokens manually
- Password encryption and hashing
- Remember-me functionality
- Account lockout after failed attempts

**This is where Spring Security comes in!** It provides all these features out of the box.

## Spring Security Architecture

Spring Security is a powerful and highly customizable authentication and access-control framework. 
It's the de-facto standard for securing Spring-based applications.

### Spring Security Filter Chain

Spring Security uses a chain of filters to handle security concerns. 
When a request comes in, it passes through multiple security filters before reaching your controller.

**Filter Chain Flow:**

```
HTTP Request
     ↓
┌────────────────────────────────────────────┐
│  Spring Security Filter Chain              │
├────────────────────────────────────────────┤
│  1. SecurityContextPersistenceFilter       │
│     (Load security context from session)   │
├────────────────────────────────────────────┤
│  2. LogoutFilter                           │
│     (Handle logout requests)               │
├────────────────────────────────────────────┤
│  3. UsernamePasswordAuthenticationFilter   │
│     (Process login form submission)        │
├────────────────────────────────────────────┤
│  4. BasicAuthenticationFilter              │
│     (Handle HTTP Basic authentication)     │
├────────────────────────────────────────────┤
│  5. ExceptionTranslationFilter             │
│     (Handle security exceptions)           │
├────────────────────────────────────────────┤
│  6. FilterSecurityInterceptor              │
│     (Check authorization/access control)   │
└──────────────────┬─────────────────────────┘
                   ↓
           Your Controller
```

**Key Filters Explained:**

1. **SecurityContextPersistenceFilter**
   - Loads the security context (user info) from session
   - Makes it available throughout the request

2. **UsernamePasswordAuthenticationFilter**
   - Processes login form submissions
   - Authenticates username and password
   - Creates authentication token

3. **ExceptionTranslationFilter**
   - Catches security exceptions
   - Redirects to login page if not authenticated
   - Shows access denied page if not authorized

4. **FilterSecurityInterceptor**
   - Final filter in the chain
   - Checks if user has required roles/permissions
   - Either allows access or throws AccessDeniedException

**Example Request Flow:**

```
User submits login form
  ↓
UsernamePasswordAuthenticationFilter intercepts
  ↓
Extracts username="john" password="secret123"
  ↓
Calls AuthenticationManager.authenticate()
  ↓
UserDetailsService loads user from database
  ↓
PasswordEncoder compares passwords
  ↓
If successful: Create Authentication object
  ↓
Store in SecurityContext
  ↓
Redirect to homepage
```

### Spring Security Configuration

Spring Security can be configured in multiple ways:

**1. Default Configuration (Zero Configuration)**
- Spring Boot auto-configures security
- All endpoints protected by default
- Single user with generated password

**2. Properties-based Configuration**
- Configure username/password in application.properties
- Simple but limited customization

**3. Java-based Configuration**
- Create a configuration class
- Full control over security behavior
- Most flexible approach

**Configuration Class Structure:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutSuccessUrl("/")
            );

        return http.build();
    }
}
```

### Spring Security Authentication Methods

Spring Security supports multiple authentication methods:

#### 1. Basic Authentication

Sends credentials with each request in HTTP headers (Base64 encoded).

```
GET /api/users HTTP/1.1
Authorization: Basic am9objpzZWNyZXQxMjM=
```

**Configuration:**
```java
http.httpBasic(Customizer.withDefaults());
```

**Use Case:** REST APIs, simple integrations

**Pros:**
- Simple to implement
- Stateless (no session)

**Cons:**
- Credentials sent with every request
- Must use HTTPS
- No logout mechanism

#### 2. Form-Based Login

Traditional HTML form for login.

```html
<form action="/login" method="POST">
    <input type="text" name="username" />
    <input type="password" name="password" />
    <button type="submit">Login</button>
</form>
```

**Configuration:**
```java
http.formLogin(form -> form
    .loginPage("/login")
    .defaultSuccessUrl("/home")
    .permitAll()
);
```

**Use Case:** Web applications with UI

**Pros:**
- User-friendly
- Session-based
- Supports remember-me

**Cons:**
- Requires session management
- CSRF protection needed

#### 3. JWT (JSON Web Token)

Token-based authentication for stateless applications.

```
GET /api/users HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Use Case:** Modern SPAs, microservices

**Pros:**
- Stateless
- Scalable
- Works across domains

**Cons:**
- Token revocation complexity
- Larger payload size

#### 4. OAuth2

Delegated authentication using third-party providers.

**Configuration:**
```java
http.oauth2Login(oauth -> oauth
    .loginPage("/login")
    .defaultSuccessUrl("/home")
);
```

**Use Case:** Social login (Google, Facebook, GitHub)

**Pros:**
- No password management
- Trusted providers
- Better user experience

**Cons:**
- Depends on external service
- Complex setup

---

## Spring Security Key Components

Understanding these core components is essential for working with Spring Security.

### 1. UserDetails

`UserDetails` is an interface that represents a user in Spring Security. It provides core user information.

**Interface Definition:**
```java
public interface UserDetails {
    String getUsername();
    String getPassword();
    Collection<? extends GrantedAuthority> getAuthorities();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

**Implementation Example:**
```java
public class CustomUserDetails implements UserDetails {

    private String username;
    private String password;
    private List<GrantedAuthority> authorities;
    private boolean enabled;

    public CustomUserDetails(User user) {
        this.username = user.getEmail();
        this.password = user.getPassword();
        this.authorities = user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
            .collect(Collectors.toList());
        this.enabled = user.isActive();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
}
```

**Spring's Default Implementation:**
```java
// Quick way to create UserDetails
UserDetails user = org.springframework.security.core.userdetails.User
    .withUsername("john")
    .password("{bcrypt}$2a$10$...")
    .roles("USER", "ADMIN")
    .build();
```

**Key Points:**
- Represents the authenticated user
- Contains credentials and authorities
- Can be customized to include additional fields (email, phone, etc.)

### 2. UserDetailsService

`UserDetailsService` is responsible for loading user-specific data. It has one method: `loadUserByUsername()`.

**Interface Definition:**
```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

**Implementation Example:**
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        // Load user from database
        User user = userRepository.findByEmail(username)
            .orElseThrow(() -> new UsernameNotFoundException(
                "User not found: " + username));

        // Convert to UserDetails
        return new CustomUserDetails(user);
    }
}
```

**Real-World Example with JPA:**
```java
// Entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;

    @ElementCollection(fetch = FetchType.EAGER)
    private Set<String> roles = new HashSet<>();

    private boolean active = true;

    // Getters and setters
}

// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

// UserDetailsService
@Service
public class DatabaseUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {

        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException(
                "User not found with email: " + email));

        return org.springframework.security.core.userdetails.User
            .withUsername(user.getEmail())
            .password(user.getPassword())
            .roles(user.getRoles().toArray(new String[0]))
            .disabled(!user.isActive())
            .build();
    }
}
```

**Key Points:**
- Called by Spring Security during authentication
- Should throw `UsernameNotFoundException` if user not found
- Typically loads user from database, LDAP, or external API

### 3. PasswordEncoder

`PasswordEncoder` is used to encode passwords and verify them during authentication.

**Interface Definition:**
```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
}
```

**Common Encoders:**

1. **BCryptPasswordEncoder** (Recommended)
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

2. **Pbkdf2PasswordEncoder**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new Pbkdf2PasswordEncoder();
}
```

3. **SCryptPasswordEncoder**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new SCryptPasswordEncoder();
}
```

**Usage Example:**
```java
@Service
public class UserService {
    
    private final PasswordEncoder passwordEncoder;
    private final UserRepository userRepository;
    
    //constructor

    public void registerUser(String email, String rawPassword) {
        // Encode password before saving
        String encodedPassword = passwordEncoder.encode(rawPassword);

        User user = new User();
        user.setEmail(email);
        user.setPassword(encodedPassword);
        user.getRoles().add("USER");

        userRepository.save(user);
    }

    public boolean checkPassword(String email, String rawPassword) {
        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        // Verify password
        return passwordEncoder.matches(rawPassword, user.getPassword());
    }
}
```

**BCrypt Example:**
```java
PasswordEncoder encoder = new BCryptPasswordEncoder();

// Encoding
String rawPassword = "myPassword123";
String encoded = encoder.encode(rawPassword);
// Output: $2a$10$xjfHKf7d8YhKJF9fJK8f.eW...

// Verification
boolean matches = encoder.matches("myPassword123", encoded);
// Output: true

boolean matches2 = encoder.matches("wrongPassword", encoded);
// Output: false
```

**Password Storage Format:**
```
{algorithm}encodedPassword

Examples:
{bcrypt}$2a$10$xjfHKf7d8YhKJF9fJK8f.eW...
{pbkdf2}$2a$10$xjfHKf7d8YhKJF9fJK8f.eW...
{noop}plainTextPassword  // No encoding (NOT RECOMMENDED)
```

**Key Points:**
- **Never store plain text passwords**
- BCrypt is recommended (adaptive, includes salt)
- Same password produces different encoded values (due to salt)
- One-way encoding (cannot decode back to original)

### 4. AuthenticationManager

`AuthenticationManager` is the main interface for authentication in Spring Security.

**Interface Definition:**
```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication)
        throws AuthenticationException;
}
```

**Default Implementation: ProviderManager**

`ProviderManager` delegates to a list of `AuthenticationProvider` instances.

**Authentication Flow:**

```
1. User submits credentials
         ↓
2. Create Authentication object (unauthenticated)
         ↓
3. AuthenticationManager.authenticate(auth)
         ↓
4. ProviderManager loops through AuthenticationProviders
         ↓
5. DaoAuthenticationProvider:
   - Calls UserDetailsService.loadUserByUsername()
   - Loads UserDetails from database
   - Calls PasswordEncoder.matches()
   - Compares passwords
         ↓
6. If successful: Return Authentication object (authenticated)
   If failed: Throw AuthenticationException
         ↓
7. Store Authentication in SecurityContext
```

**Configuration Example:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    AuthenticationManager authenticationManager(UserDetailsService uds, PasswordEncoder pe) {
        var authProvider = new DaoAuthenticationProvider(uds);
        authProvider.setPasswordEncoder(pe);

        return new ProviderManager(authProvider);
    }
}
```

**Manual Authentication Example:**
```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @PostMapping("/login")
    public ResponseEntity<String> login(@RequestBody LoginRequest request) {
        try {
            // Create unauthenticated token
            Authentication authentication = new UsernamePasswordAuthenticationToken(
                request.getUsername(),
                request.getPassword()
            );

            // Authenticate
            Authentication authenticated =
                authenticationManager.authenticate(authentication);

            // Store in security context
            SecurityContextHolder.getContext().setAuthentication(authenticated);

            return ResponseEntity.ok("Login successful");

        } catch (AuthenticationException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body("Invalid credentials");
        }
    }
}
```

**Key Components Working Together:**

```
┌─────────────────────────────────────────────────────┐
│         AuthenticationManager                       │
│           (ProviderManager)                         │
└────────────────┬────────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────────┐
│      DaoAuthenticationProvider                      │
├─────────────────────────────────────────────────────┤
│  - Uses UserDetailsService                          │
│  - Uses PasswordEncoder                             │
└────────────┬────────────────────┬───────────────────┘
             ↓                    ↓
┌────────────────────┐  ┌─────────────────────┐
│ UserDetailsService │  │  PasswordEncoder    │
│                    │  │                     │
│ loadUserByUsername │  │  encode()           │
│                    │  │  matches()          │
└────────────────────┘  └─────────────────────┘
```

## Spring Security in Web Applications

Let's explore different ways to configure Spring Security in a Spring Boot application, from simplest to most advanced.

### 1. Default Spring Security Configuration

When you add Spring Security dependency, Spring Boot automatically configures security.

**Add Dependency:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**What Happens Automatically:**

1. **All endpoints/urls are protected** - Requires authentication for every URL
2. **Default user created** - Username: `user`
3. **Random password generated** - Printed in console logs
4. **Default login page** - Auto-generated at `/login`
5. **HTTP Basic authentication** - Enabled by default
6. **CSRF protection** - Enabled by default

**Console Output:**
```
Using generated security password: 8e4e5c3a-6f5b-4d8e-9a3f-2c1d5e6f7a8b
```

To use Thymeleaf views, add the following dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Create thymeleaf templates in `src/main/resources/templates` directory.

**Testing Default Security:**
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

@RestController
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "Welcome Home!";
    }

    @GetMapping("/user")
    public String user() {
        return "User Page";
    }
}
```

**Accessing the Application:**
- Navigate to `http://localhost:8080/`
- Redirected to `/login`
- Enter username: `user`
- Enter password: (from console)
- Access granted

**Pros:**
- Zero configuration required
- Quick setup for testing

**Cons:**
- Not suitable for production
- Password changes on every restart
- Single user only
- No customization

### 2. Security Configuration with Properties

Configure basic security settings using `application.properties` or `application.yml`.

**application.properties:**
```properties
# Single user configuration
spring.security.user.name=admin
spring.security.user.password=admin123
spring.security.user.roles=ADMIN
```

**Testing:**
```bash
curl -u admin:admin123 http://localhost:8080/user
```

**Limitations:**
- Only one user can be configured
- No password encryption
- Cannot configure URL-specific security
- Not suitable for production

### 3. In-Memory Authentication

Configure multiple users in memory using Java configuration.

**SecurityConfig.java:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/home", "/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/home")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutSuccessUrl("/")
                .permitAll()
            );

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder.encode("user123"))
            .roles("USER")
            .build();

        UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder.encode("admin123"))
            .roles("ADMIN", "USER")
            .build();

        UserDetails manager = User.builder()
            .username("manager")
            .password(passwordEncoder.encode("manager123"))
            .roles("MANAGER")
            .build();

        return new InMemoryUserDetailsManager(user, admin, manager);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**URL Access Control Explained:**

```java
.requestMatchers("/", "/home", "/public/**").permitAll()
// Everyone can access: /, /home, /public/anything

.requestMatchers(HttpMethod.GET, "/api/products").permitAll()
// Everyone can access: /api/products using only HTTP GET method

.requestMatchers("/admin/**").hasRole("ADMIN")
// Only ADMIN role: /admin/users, /admin/settings

.requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
// USER or ADMIN role: /user/profile, /user/orders

.anyRequest().authenticated()
// All other URLs require authentication
```

**Testing Different Users:**

```java
@RestController
public class TestController {

    @GetMapping("/")
    public String home() {
        return "Public Home Page";
    }

    @GetMapping("/user/profile")
    public String userProfile() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        return "User Profile: " + auth.getName();
    }

    @GetMapping("/admin/dashboard")
    public String adminDashboard() {
        return "Admin Dashboard";
    }
}
```

**Access Control Results:**

| URL                 | user | admin | manager | Anonymous |
|---------------------|------|-------|---------|-----------|
| /                   | ✓    | ✓     | ✓       | ✓         |
| /user/profile       | ✓    | ✓     | ✗       | ✗         |
| /admin/dashboard    | ✗    | ✓     | ✗       | ✗         |

**Pros:**
- Multiple users with different roles
- Passwords encrypted
- URL-specific security
- Good for testing and development

**Cons:**
- Users hardcoded (restart required to change)
- Not suitable for production

### 4. JDBC Authentication

Store users in a database and authenticate against it.

**Step 1: Database Schema**

Spring Security provides default schema:

```sql
-- Users table
CREATE TABLE users (
    username VARCHAR(50) NOT NULL PRIMARY KEY,
    password VARCHAR(100) NOT NULL,
    enabled BOOLEAN NOT NULL
);

-- Authorities table
CREATE TABLE authorities (
    username VARCHAR(50) NOT NULL,
    authority VARCHAR(50) NOT NULL,
    FOREIGN KEY (username) REFERENCES users(username)
);

-- Optional: Create unique index
CREATE UNIQUE INDEX ix_auth_username ON authorities (username, authority);
```

**Step 2: Insert Sample Data**

```sql
-- Insert users (passwords are BCrypt encoded)
-- password for 'john' is 'john123'
INSERT INTO users (username, password, enabled)
VALUES ('john', '$2a$10$xjfHKf7d8YhKJF9fJK8f.eW5J3F5fJf8f.eW5J3F5fJf', true);

-- password for 'sarah' is 'sarah123'
INSERT INTO users (username, password, enabled)
VALUES ('sarah', '$2a$10$aK9fJK8f.eW5J3F5fJf8f.eW5J3F5fJf8f.eW5J3F5f', true);

-- Insert authorities
INSERT INTO authorities (username, authority) VALUES ('john', 'ROLE_USER');
INSERT INTO authorities (username, authority) VALUES ('sarah', 'ROLE_ADMIN');
INSERT INTO authorities (username, authority) VALUES ('sarah', 'ROLE_USER');
```

**Step 3: Security Configuration**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/home").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())
            .logout(Customizer.withDefaults());

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);
        return manager;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**Step 4: Custom Schema (Optional)**

If you have your own user table structure:

```sql
CREATE TABLE app_users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(100) NOT NULL,
    full_name VARCHAR(100),
    active BOOLEAN DEFAULT true
);

CREATE TABLE user_roles (
    user_id BIGINT NOT NULL,
    role VARCHAR(50) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES app_users(id)
);
```

**Custom JDBC Configuration:**

```java
@Bean
public UserDetailsService userDetailsService() {
    JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);

    // Custom query to load user
    manager.setUsersByUsernameQuery(
        "SELECT email, password, active FROM app_users WHERE email = ?"
    );

    // Custom query to load authorities
    manager.setAuthoritiesByUsernameQuery(
        "SELECT u.email, r.role " +
        "FROM app_users u " +
        "JOIN user_roles r ON u.id = r.user_id " +
        "WHERE u.email = ?"
    );

    return manager;
}
```

**Creating Users Programmatically:**

```java
@Service
public class UserInitService {

    @Autowired
    private JdbcUserDetailsManager userDetailsManager;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @PostConstruct
    public void initUsers() {
        if (!userDetailsManager.userExists("john")) {
            UserDetails user = User.builder()
                .username("john")
                .password(passwordEncoder.encode("john123"))
                .roles("USER")
                .build();
            userDetailsManager.createUser(user);
        }
    }
}
```

**Pros:**
- Users stored in database
- Can add/remove users without restarting
- Standard Spring Security tables

**Cons:**
- Fixed table structure
- Limited flexibility
- Difficult to add custom user fields

### 5. Custom UserDetailsService

The most flexible approach - implement your own `UserDetailsService` with custom entity structure.

**Step 1: Create User Entity**

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private String fullName;

    private boolean enabled = true;

    private boolean accountNonLocked = true;

    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "role")
    private Set<String> roles = new HashSet<>();

    @CreationTimestamp
    private Instant createdAt;

    @UpdateTimestamp
    private Instant updatedAt;

    // Constructors, getters, setters
}
```

**Step 2: Create Repository**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

**Step 3: Implement Custom UserDetails**

```java
public class CustomUserDetails implements UserDetails {

    private final User user;

    public CustomUserDetails(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
            .collect(Collectors.toList());
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getEmail();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return user.isAccountNonLocked();
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return user.isEnabled();
    }

    // Additional methods to access User entity
    public String getFullName() {
        return user.getFullName();
    }

    public Long getId() {
        return user.getId();
    }
}
```

**Step 4: Implement UserDetailsService**

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;
    
    //constructor

    @Override
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {

        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException(
                "User not found with email: " + email));

        return new CustomUserDetails(user);
    }
}
```

**Step 5: Security Configuration**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/register", "/css/**", "/js/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/perform-login")
                .defaultSuccessUrl("/dashboard", true)
                .failureUrl("/login?error=true")
                .usernameParameter("email")  // Custom field name
                .passwordParameter("password")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout=true")
                .invalidateHttpSession(true)
                .deleteCookies("JSESSIONID")
                .permitAll()
            )
            .exceptionHandling(ex -> ex
                .accessDeniedPage("/access-denied")
            );

        return http.build();
    }

    
}
```

**Step 6: User Registration Service**

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    public User registerNewUser(UserRegistrationDto dto) {
        // Check if user already exists
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new RuntimeException("Email already exists");
        }

        // Create new user
        User user = new User();
        user.setEmail(dto.getEmail());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setFullName(dto.getFullName());
        user.setEnabled(true);
        user.getRoles().add("USER");

        return userRepository.save(user);
    }

    public User getCurrentUser() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()) {
            return null;
        }

        CustomUserDetails userDetails = (CustomUserDetails) auth.getPrincipal();
        return userRepository.findById(userDetails.getId())
            .orElse(null);
    }
}
```

**Step 7: Controller Example**

```java
@Controller
public class AuthController {

    @Autowired
    private UserService userService;

    @GetMapping("/login")
    public String loginPage() {
        return "login";
    }

    @GetMapping("/register")
    public String registerPage(Model model) {
        model.addAttribute("user", new UserRegistrationDto());
        return "register";
    }

    @PostMapping("/register")
    public String registerUser(@Valid @ModelAttribute UserRegistrationDto dto,
                              BindingResult result) {
        if (result.hasErrors()) {
            return "register";
        }

        try {
            userService.registerNewUser(dto);
            return "redirect:/login?registered=true";
        } catch (Exception e) {
            result.rejectValue("email", "error.email", e.getMessage());
            return "register";
        }
    }

    @GetMapping("/dashboard")
    public String dashboard(Model model) {
        User user = userService.getCurrentUser();
        model.addAttribute("user", user);
        return "dashboard";
    }
}
```

**Accessing Current User in Controllers:**

```java
@RestController
@RequestMapping("/api")
public class ApiController {

    // Method 1: Using SecurityContextHolder
    @GetMapping("/user/info")
    public Map<String, Object> getUserInfo() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        CustomUserDetails userDetails = (CustomUserDetails) auth.getPrincipal();

        return Map.of(
            "email", userDetails.getUsername(),
            "fullName", userDetails.getFullName(),
            "roles", userDetails.getAuthorities()
        );
    }

    // Method 2: Using @AuthenticationPrincipal annotation
    @GetMapping("/user/profile")
    public Map<String, Object> getProfile(
            @AuthenticationPrincipal CustomUserDetails userDetails) {

        return Map.of(
            "email", userDetails.getUsername(),
            "fullName", userDetails.getFullName()
        );
    }

    // Method 3: Using Authentication parameter
    @GetMapping("/user/details")
    public Map<String, Object> getDetails(Authentication authentication) {
        CustomUserDetails userDetails = (CustomUserDetails) authentication.getPrincipal();

        return Map.of(
            "name", authentication.getName(),
            "authorities", authentication.getAuthorities()
        );
    }
}
```

**Pros:**
- Complete flexibility
- Custom user fields
- Production-ready
- Can integrate with any data source

**Cons:**
- More code to write
- Need to implement all security logic

### 6. Spring Security Logout

Spring Security provides built-in logout functionality.

**Basic Logout Configuration:**

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .logout(logout -> logout
            .logoutUrl("/logout")                    // URL to trigger logout
            .logoutSuccessUrl("/login?logout=true")   // Redirect after logout
            .invalidateHttpSession(true)              // Invalidate session
            .deleteCookies("JSESSIONID")              // Delete cookies
            .clearAuthentication(true)                // Clear authentication
            .permitAll()
        );

    return http.build();
}
```

**Logout Form (GET request):**

```html
<form action="/logout" method="post">
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
    <button type="submit">Logout</button>
</form>
```

**Logout Link (using Thymeleaf):**

```html
<form th:action="@{/logout}" method="post">
    <button type="submit">Logout</button>
</form>
```

**Custom Logout Handler:**

```java
@Component
public class CustomLogoutHandler implements LogoutHandler {

    @Override
    public void logout(HttpServletRequest request,
                      HttpServletResponse response,
                      Authentication authentication) {

        // Custom logout logic
        if (authentication != null) {
            String username = authentication.getName();
            System.out.println("User " + username + " logged out");

            // Log to database, send notification, etc.
        }
    }
}
```

**Custom Logout Success Handler:**

```java
@Component
public class CustomLogoutSuccessHandler implements LogoutSuccessHandler {

    @Override
    public void onLogoutSuccess(HttpServletRequest request,
                               HttpServletResponse response,
                               Authentication authentication)
            throws IOException, ServletException {

        if (authentication != null) {
            String username = authentication.getName();
            System.out.println("User " + username + " successfully logged out");
        }

        response.setStatus(HttpStatus.OK.value());
        response.sendRedirect("/login?logout=true");
    }
}
```

**Using Custom Handlers:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private CustomLogoutHandler customLogoutHandler;

    @Autowired
    private CustomLogoutSuccessHandler customLogoutSuccessHandler;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .logout(logout -> logout
                .logoutUrl("/logout")
                .addLogoutHandler(customLogoutHandler)
                .logoutSuccessHandler(customLogoutSuccessHandler)
                .permitAll()
            );

        return http.build();
    }
}
```

**Logout Best Practices:**

1. **Always use POST** for logout (prevent CSRF attacks)
2. **Invalidate session** to free server resources
3. **Clear authentication** from SecurityContext
4. **Delete cookies** to remove client-side data
5. **Log logout events** for audit trails
6. **Redirect appropriately** based on user type


## Method-Level Security

Method-level security allows you to protect individual methods in your application using annotations. 
This provides fine-grained access control at the service or controller layer.

### Enabling Method Security

To use method-level security, you need to enable it in your configuration:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // Enable method-level security
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults());

        return http.build();
    }
}
```

### Method Security Annotations

Spring Security provides several annotations for method-level security:

#### 1. @PreAuthorize

Checks authorization **before** method execution. Most commonly used annotation.

**Basic Example:**
```java
@Service
public class UserService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) {
        // Only admins can delete users
        userRepository.deleteById(userId);
    }

    @PreAuthorize("hasRole('USER')")
    public User getUserProfile(Long userId) {
        // Only authenticated users with USER role can access
        return userRepository.findById(userId).orElseThrow();
    }

    @PreAuthorize("hasAnyRole('USER', 'ADMIN')")
    public List<User> getAllUsers() {
        // Users with either USER or ADMIN role can access
        return userRepository.findAll();
    }
}
```

**Advanced Example with SpEL (Spring Expression Language):**
```java
@Service
public class PostService {

    @Autowired
    private PostRepository postRepository;

    // Allow access only to the post owner or admin
    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public void updatePost(Long postId, Long userId, String content) {
        Post post = postRepository.findById(postId).orElseThrow();
        post.setContent(content);
        postRepository.save(post);
    }

    // Check if user is owner of the post
    @PreAuthorize("@postSecurityService.isOwner(#postId, authentication.principal.id)")
    public void deletePost(Long postId) {
        postRepository.deleteById(postId);
    }

    // Multiple conditions
    @PreAuthorize("hasRole('PREMIUM_USER') and #userId == authentication.principal.id")
    public void accessPremiumContent(Long userId) {
        // Only premium users can access their own premium content
    }
}
```

**Custom Security Service:**
```java
@Service("postSecurityService")
public class PostSecurityService {

    @Autowired
    private PostRepository postRepository;

    public boolean isOwner(Long postId, Long userId) {
        return postRepository.findById(postId)
            .map(post -> post.getUserId().equals(userId))
            .orElse(false);
    }

    public boolean canEdit(Long postId, Long userId) {
        Post post = postRepository.findById(postId).orElse(null);
        if (post == null) return false;

        // Owner can edit, or admin, or if post is in draft status
        return post.getUserId().equals(userId) ||
               post.getStatus().equals("DRAFT");
    }
}
```

#### 2. @PostAuthorize

Checks authorization **after** method execution. Useful when you need to check the return value.

```java
@Service
public class DocumentService {

    // Only allow if returned document belongs to current user
    @PostAuthorize("returnObject.userId == authentication.principal.id")
    public Document getDocument(Long documentId) {
        return documentRepository.findById(documentId).orElseThrow();
    }

    // Allow if user is admin or owns the returned document
    @PostAuthorize("hasRole('ADMIN') or returnObject.userId == authentication.principal.id")
    public Document getDocumentDetails(Long documentId) {
        return documentRepository.findById(documentId).orElseThrow();
    }
}
```

#### 3. @PreFilter

Filters collection parameters **before** method execution.

```java
@Service
public class BatchService {

    // Filter the list, keeping only items where userId matches authenticated user
    @PreFilter("filterObject.userId == authentication.principal.id")
    public void processOrders(List<Order> orders) {
        // Only process orders that belong to the current user
        orders.forEach(order -> orderRepository.save(order));
    }

    // Another example
    @PreFilter("hasRole('ADMIN') or filterObject.ownerId == authentication.principal.id")
    public void deleteMultipleDocuments(List<Document> documents) {
        documents.forEach(doc -> documentRepository.delete(doc));
    }
}
```

#### 4. @PostFilter

Filters collection return values **after** method execution.

```java
@Service
public class PostService {

    // Filter results to only show posts from user's friends or public posts
    @PostFilter("filterObject.isPublic == true or filterObject.userId == authentication.principal.id")
    public List<Post> getAllPosts() {
        return postRepository.findAll();
    }

    // Only return documents owned by current user
    @PostFilter("filterObject.ownerId == authentication.principal.id")
    public List<Document> getAllDocuments() {
        return documentRepository.findAll();
    }

    // Admin sees all, users see only their own
    @PostFilter("hasRole('ADMIN') or filterObject.userId == authentication.principal.id")
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }
}
```

### Performance Considerations

```java
@Service
public class PerformanceExampleService {

    // ❌ Bad: Loads all records then filters
    @PostFilter("filterObject.userId == authentication.principal.id")
    public List<Order> getAllOrders() {
        return orderRepository.findAll(); // Loads all orders
    }

    // ✅ Good: Filters at database level
    @PreAuthorize("isAuthenticated()")
    public List<Order> getUserOrders(Long userId) {
        return orderRepository.findByUserId(userId); // Filters in SQL
    }
}
```

### Combining URL-Level and Method-Level Security

You can use both URL-level and method-level security together for defense in depth:

```java
@RestController
@RequestMapping("/api/posts")
public class PostController {

    @Autowired
    private PostService postService;

    // URL-level: Requires authentication
    // Method-level: Requires ADMIN role
    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletePost(@PathVariable Long id) {
        postService.deletePost(id);
        return ResponseEntity.ok().build();
    }

    // Only post owner or admin can update
    @PreAuthorize("@postSecurityService.isOwner(#id, authentication.principal.id) or hasRole('ADMIN')")
    @PutMapping("/{id}")
    public ResponseEntity<Post> updatePost(
            @PathVariable Long id,
            @RequestBody PostUpdateDto dto) {

        Post updated = postService.updatePost(id, dto);
        return ResponseEntity.ok(updated);
    }

    // Anyone authenticated can view
    @GetMapping("/{id}")
    public ResponseEntity<Post> getPost(@PathVariable Long id) {
        Post post = postService.getPost(id);
        return ResponseEntity.ok(post);
    }
}
```

### SpEL Expressions in Method Security

Spring Expression Language (SpEL) provides powerful expressions for complex security rules:

#### Common SpEL Variables:

| Variable         | Description                          | Example                                |
|------------------|--------------------------------------|----------------------------------------|
| `authentication` | Current Authentication object        | `authentication.principal.id`          |
| `principal`      | Current user principal               | `principal.username`                   |
| `#paramName`     | Method parameter                     | `#userId == principal.id`              |
| `returnObject`   | Method return value (@PostAuthorize) | `returnObject.userId == principal.id`  |
| `filterObject`   | Current item in collection (filters) | `filterObject.ownerId == principal.id` |

#### SpEL Examples:

```java
@Service
public class SecurityExamplesService {

    // Check if parameter matches current user
    @PreAuthorize("#userId == authentication.principal.id")
    public void updateProfile(Long userId, ProfileDto dto) { }

    // Check if user has specific authority
    @PreAuthorize("hasAuthority('WRITE_PRIVILEGE')")
    public void writeData() { }

    // Multiple conditions with AND
    @PreAuthorize("hasRole('USER') and #userId == authentication.principal.id")
    public void updateUserData(Long userId) { }

    // Multiple conditions with OR
    @PreAuthorize("hasRole('ADMIN') or #ownerId == authentication.principal.id")
    public void deleteResource(Long ownerId) { }

    // Check authentication status
    @PreAuthorize("isAuthenticated()")
    public void authenticatedOnly() { }

    // Allow anonymous access
    @PreAuthorize("permitAll()")
    public void publicAccess() { }

    // Deny all access
    @PreAuthorize("denyAll()")
    public void noAccess() { }

    // Complex expression
    @PreAuthorize("hasRole('ADMIN') or (hasRole('USER') and #userId == authentication.principal.id)")
    public void complexCheck(Long userId) { }

    // Call custom bean method
    @PreAuthorize("@customSecurityService.hasAccess(#resourceId, authentication.principal.id)")
    public void checkCustomAccess(Long resourceId) { }
}
```

### Method Security with Custom Annotations

Create custom security annotations for reusability:

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("hasRole('ADMIN')")
public @interface IsAdmin {
}

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
public @interface IsOwnerOrAdmin {
}

// Usage
@Service
public class CustomAnnotationService {

    @IsAdmin
    public void adminOnlyMethod() {
        // Only admins can access
    }

    @IsOwnerOrAdmin
    public void ownerOrAdminMethod(Long userId) {
        // Owner or admin can access
    }
}
```

### Exception Handling

When authorization fails, Spring Security throws `AccessDeniedException`. 
Handle it with a global exception handler:

```java
@RestControllerAdvice
public class SecurityExceptionHandler {

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.FORBIDDEN.value(),
            "Access Denied",
            "You don't have permission to access this resource"
        );
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }

    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuthentication(AuthenticationException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.UNAUTHORIZED.value(),
            "Authentication Failed",
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(error);
    }
}
```

## Assignments

### Assignment 1: Basic Security Setup

**Objective:** Set up basic Spring Security with in-memory authentication

**Tasks:**
1. Create a new Spring Boot application with Spring Security
2. Configure three users: user, admin, manager with different roles
3. Create the following endpoints:
   - `/public/home` - accessible to everyone
   - `/user/profile` - accessible to USER and ADMIN
   - `/admin/dashboard` - accessible only to ADMIN
4. Test with different users and verify access control

**Expected Output:**
- Anonymous users can access `/public/home`
- USER can access `/user/profile` but not `/admin/dashboard`
- ADMIN can access both `/user/profile` and `/admin/dashboard`

### Assignment 2: Custom Login Page

**Objective:** Create a custom login page with styling

**Tasks:**
1. Create a custom login page (HTML/Thymeleaf)
2. Add CSS styling to make it attractive
3. Display error messages for failed login attempts
4. Show success message after registration

**Expected Output:**
- Custom styled login page
- Error messages displayed properly


### Assignment 3: Database Authentication

**Objective:** Implement JDBC-based authentication

**Tasks:**
1. Create database tables for users and authorities
2. Configure Spring Security to use JDBC authentication
3. Insert sample users via SQL scripts
4. Create a registration page to add new users
5. Hash passwords using BCrypt

**Expected Output:**
- Users stored in database
- New users can register
- Passwords are encrypted
- Login works with database users

### Assignment 4: Custom UserDetailsService

**Objective:** Implement custom user entity with JPA

**Tasks:**
1. Create a User entity with fields: id, email, password, fullName, enabled, roles
2. Create UserRepository with JPA
3. Implement CustomUserDetailsService
4. Create registration and login functionality
5. Add user profile page showing user details

**Expected Output:**
- Complete user management system
- Registration with validation
- Login with custom user entity
- Profile page showing user information

### Assignment 5: Role-Based Authorization

**Objective:** Implement complex role-based access control

**Tasks:**
1. Create roles: USER, PREMIUM_USER, MANAGER, ADMIN
2. Create the following endpoints:
   - `/free/content` - accessible to all authenticated users
   - `/premium/content` - accessible to PREMIUM_USER and above
   - `/manager/reports` - accessible to MANAGER and ADMIN
   - `/admin/settings` - accessible only to ADMIN

**Expected Output:**
- Different content for different roles
- Access denied page for unauthorized access

### Assignment 6: Logout and Session Management

**Objective:** Implement custom logout functionality

**Tasks:**
1. Create a custom logout handler to log logout events
2. Save login/logout timestamp in database
3. Display login history on user profile
4. Add session timeout configuration

**Expected Output:**
- Logout events logged in database
- User can see login history
- Session expires after inactivity

### Assignment 7: Password Management

**Objective:** Implement password change and reset functionality

**Tasks:**
1. Create "Change Password" page
2. Validate old password before changing
3. Implement password strength requirements
4. Create "Forgot Password" functionality (without email for now)

**Expected Output:**
- Users can change password
- Password validation working
- Password reset functionality


# Spring Security JWT-based Authentication

## Table of Contents
- [Web Application Security vs API Security](#web-application-security-vs-api-security)
- [How JWT-Based Authentication Works](#how-jwt-based-authentication-works)
- [JWT Structure and Components](#jwt-structure-and-components)
- [JWT Authentication in Spring Security](#jwt-authentication-in-spring-security)
- [Generating RSA Keys with OpenSSL](#generating-rsa-keys-with-openssl)
- [OAuth2 Resource Server Configuration](#oauth2-resource-server-configuration)
- [Implementing JWT Token Provider](#implementing-jwt-token-provider)
- [Configuring Security Filters and CORS](#configuring-security-filters-and-cors)
- [Implementing Login Endpoint](#implementing-login-endpoint)
- [Retrieving Current User from JWT](#retrieving-current-user-from-jwt)
- [Best Practices and Security Considerations](#best-practices-and-security-considerations)
- [Assignments](#assignments)

## Web Application Security vs API Security

Understanding the difference between traditional web application security and 
API security is crucial for choosing the right authentication mechanism.

### Traditional Web Application Security

Traditional web applications are **stateful** and use server-side sessions.

**Characteristics:**

1. **Session-Based Authentication**
   - Server creates and stores session after login
   - Session ID stored in a cookie
   - Session data stored on server (in-memory or database)

2. **Cookie-Based**
   - JSESSIONID cookie sent with each request
   - Cookies are domain-specific
   - Automatic cookie management by browser

### REST API Security

Modern REST APIs are **stateless** and use token-based authentication.

**Characteristics:**

1. **Token-Based Authentication**
   - Server generates token after login
   - Token contains all user information
   - No server-side session storage

2. **Stateless**
   - Each request contains all necessary information
   - Server doesn't store client state
   - Easy to scale horizontally

3. **JSON Communication**
   - Data exchanged in JSON format
   - API consumed by multiple clients (web, mobile, third-party)

### When to Use What?

**Use Session-Based (Traditional):**
- Building a monolithic web application
- Server-side rendered pages
- Simple authentication requirements
- Need immediate session invalidation
- Single domain application

**Use Token-Based (JWT):**
- Building REST APIs
- Microservices architecture
- Mobile applications
- Single Page Applications (SPAs)
- Cross-domain authentication
- Need to scale horizontally

## How JWT-Based Authentication Works

JWT (JSON Web Token) is an open standard (RFC 7519) for securely transmitting information between parties as a JSON object.

### JWT Authentication Flow

Here's a complete authentication flow using JWT:

```
┌─────────┐                                        ┌─────────┐
│         │                                        │         │
│ Client  │                                        │  Server │
│         │                                        │         │
└────┬────┘                                        └────┬────┘
     │                                                  │
     │  1. POST /auth/login                             │
     │     { username, password }                       │
     ├─────────────────────────────────────────────────>│
     │                                                  │
     │                                          2. Validate credentials
     │                                                  │
     │                                          3. Generate JWT token
     │                                             (sign with secret key)
     │                                                  │
     │  4. Return JWT token                             │
     │     { token: "eyJhbGc..." }                      │
     │<─────────────────────────────────────────────────┤
     │                                                  │
5. Store token                                          │
   (localStorage/memory)                                │
     │                                                  │
     │  6. GET /api/users                               │
     │     Authorization: Bearer eyJhbGc...             │
     ├─────────────────────────────────────────────────>│
     │                                                  │
     │                                          7. Extract token
     │                                                  │
     │                                          8. Validate signature
     │                                                  │
     │                                          9. Verify expiration
     │                                                  │
     │                                          10. Extract user info
     │                                                  │
     │  11. Return protected resource                   │
     │      { users: [...] }                            │
     │<─────────────────────────────────────────────────┤
     │                                                  │
```

**Step-by-Step Explanation:**

1. **Client sends credentials** - User submits username and password to login endpoint

2. **Server validates credentials** - Server checks if username and password are correct

3. **Server generates JWT** - If valid, server creates a JWT token containing:
   - User ID
   - Username
   - Roles/Authorities
   - Expiration time
   - Other claims

4. **Server returns JWT** - Token sent back to client in response body

5. **Client stores token** - Client saves token (localStorage, sessionStorage, or in-memory)

6. **Client sends token** - For subsequent requests, client includes token in Authorization header

7. **Server extracts token** - Server reads the token from Authorization header

8. **Server validates signature** - Server verifies token hasn't been tampered with

9. **Server checks expiration** - Ensures token hasn't expired

10. **Server extracts user info** - Decodes token to get user details

11. **Server responds** - Returns requested resource if authorized

## JWT Structure and Components

A JWT consists of three parts separated by dots (`.`):

```
eyJhbGciOiJSUzI1NXXXX.eyJzdWIiOiIxjIzOTXXXX.signature_here

└──── Header ───────┘ └───── Payload ─────┘ └── Signature ──┘
```

### 1. Header

Contains metadata about the token:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

**Fields:**
- `alg` - Algorithm used for signing (RS256, HS256, etc.)
- `typ` - Type of token (JWT)

**Encoded:** `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9`

### 2. Payload

Contains the claims (user information):

```json
{
  "sub": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "roles": ["USER", "ADMIN"],
  "iat": 1516239022,
  "exp": 1516242622
}
```

**Standard Claims:**

| Claim | Name               | Description                          |
|-------|--------------------|--------------------------------------|
| `sub` | Subject            | User identifier                      |
| `iss` | Issuer             | Who issued the token                 |
| `aud` | Audience           | Who the token is intended for        |
| `exp` | Expiration Time    | When token expires (Unix timestamp)  |
| `iat` | Issued At          | When token was created               |
| `nbf` | Not Before         | Token not valid before this time     |
| `jti` | JWT ID             | Unique identifier for the token      |

**Custom Claims:**

You can add custom claims:
```json
{
  "userId": 123,
  "username": "johndoe",
  "roles": ["USER", "ADMIN"],
  "permissions": ["read", "write"]
}
```

**Important:** Don't store sensitive data in payload (it's base64 encoded, not encrypted!)

**Encoded:** `eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0`

### 3. Signature

Ensures token hasn't been tampered with:

```
RSASHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  privateKey
)
```

**How it works:**

1. Take encoded header and payload
2. Combine them with a dot
3. Sign with private key (RS256) or secret (HS256)
4. Append signature to token

**Verification:**

```
publicKey.verify(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  signature
)
```

Server uses public key (RS256) or the same secret (HS256) to verify signature.

## JWT Authentication in Spring Security

Spring Security provides two approaches to implement JWT authentication:

### 1. Custom JWT Filter (Manual Approach)

Using libraries like **jjwt** (Java JWT):

**Dependency:**

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

**Pros:**
- Full control over JWT generation and validation
- Can customize claims easily
- Lightweight

**Cons:**
- More code to write
- Manual filter configuration
- Need to handle edge cases

### 2. OAuth2 Resource Server (Spring Security's Built-in)

Using **Spring Security OAuth2 Resource Server**:

**Dependency:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

**Pros:**
- Built-in Spring Security support
- Less code to write
- Well-tested and secure
- Follows OAuth2 standards
- Easy configuration

**Cons:**
- Requires understanding of OAuth2 concepts

**In this guide, we'll use OAuth2 Resource Server approach** as it's the recommended way.

## Generating RSA Keys with OpenSSL

For RS256 algorithm, we need a public-private key pair. Let's generate them using OpenSSL.

### Step 1: Generate Private Key

```bash
# Generate 2048-bit RSA private key
openssl genrsa -out private.pem 2048
```

**Output:**
```
Generating RSA private key, 2048 bit long modulus
.......+++
...........................................+++
e is 65537 (0x010001)
```

**private.pem:**
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA0Z5V3QJ5K... (many lines)
-----END RSA PRIVATE KEY-----
```

### Step 2: Generate Public Key

```bash
# Extract public key from private key
openssl rsa -in private.pem -pubout -out public.pem
```

**Output:**
```
writing RSA key
```

**public.pem:**
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0Z5V... (many lines)
-----END PUBLIC KEY-----
```

### Step 3: Convert to PKCS8 Format (Required for Java)

```bash
# Convert private key to PKCS8 format
openssl pkcs8 -topk8 -inform PEM -outform PEM -in private.pem -out private_key.pem -nocrypt
```

**private_key.pem:**
```
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASC... (many lines)
-----END PRIVATE KEY-----
```

### Step 4: Place Keys in Spring Boot Project

Create a directory structure:

```
src/main/resources/
└── keys/
    ├── private_key.pem
    └── public.pem
```

### Alternative: One-Line Commands

```bash
# Generate private key in PKCS8 format directly
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048

# Generate public key
openssl rsa -pubout -in private_key.pem -out public.pem
```

## OAuth2 Resource Server Configuration

Now let's configure Spring Security to use OAuth2 Resource Server with JWT.

### Step 1: Add Dependencies

**pom.xml:**

```xml
<dependencies>
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- OAuth2 Resource Server -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    </dependency>

    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

</dependencies>
```

### Step 2: Configure Application Properties

**application.properties:**

```properties

# JWT Configuration
jwt.private.key=classpath:keys/private_key.pem
jwt.public.key=classpath:keys/public.pem
jwt.expiration=3600000
```

### Step 3: Configure JwtDecoder and JwtEncoder

Create a configuration class to set up JWT encoding and decoding:

**RSAKeyProperties.java:**

```java
package com.example.security.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

@ConfigurationProperties(prefix = "jwt")
public record RSAKeyProperties(
    RSAPublicKey publicKey,
    RSAPrivateKey privateKey,
    Long expiration
) {
}
```

**JwtConfig.java:**

```java
package com.example.security.config;

import com.nimbusds.jose.jwk.JWK;
import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import com.nimbusds.jose.jwk.source.ImmutableJWKSet;
import com.nimbusds.jose.jwk.source.JWKSource;
import com.nimbusds.jose.proc.SecurityContext;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.oauth2.jwt.NimbusJwtEncoder;

@Configuration
@EnableConfigurationProperties(RSAKeyProperties.class)
public class JwtConfig {

    private final RSAKeyProperties rsaKeys;

    public JwtConfig(RSAKeyProperties rsaKeys) {
        this.rsaKeys = rsaKeys;
    }

    /**
     * JwtDecoder: Used to decode and validate JWT tokens
     * Uses the public key to verify the signature
     */
    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withPublicKey(rsaKeys.publicKey()).build();
    }

    /**
     * JwtEncoder: Used to encode/generate JWT tokens
     * Uses the private key to sign the token
     */
    @Bean
    public JwtEncoder jwtEncoder() {
        JWK jwk = new RSAKey.Builder(rsaKeys.publicKey())
                .privateKey(rsaKeys.privateKey())
                .build();

        JWKSource<SecurityContext> jwks = new ImmutableJWKSet<>(new JWKSet(jwk));
        return new NimbusJwtEncoder(jwks);
    }
}
```

### Step 4: Configure JwtAuthenticationConverter

The `JwtAuthenticationConverter` converts a JWT token into an Authentication object with proper authorities.

Register a `JwtAuthenticationConverter` bean as follows:

```java
package com.example.security.config;

@Configuration
@EnableConfigurationProperties(RSAKeyProperties.class)
public class JwtConfig {

    //...
    
    @Bean
    JwtAuthenticationConverter jwtAuthenticationConverter() {
        var grantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("");

        var jwtAuthenticationConverter = new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
        return jwtAuthenticationConverter;
    }
}
```

## Implementing JWT Token Provider

Create a service to generate and validate JWT tokens.

**JwtTokenProvider.java:**

```java
package com.example.security.jwt;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.oauth2.jwt.JwtClaimsSet;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.JwtEncoderParameters;
import org.springframework.stereotype.Service;

import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.stream.Collectors;

@Service
public class JwtTokenProvider {

    private final JwtEncoder jwtEncoder;
    private final Long jwtExpiration;

    public JwtTokenProvider(JwtEncoder jwtEncoder) {
        this.jwtEncoder = jwtEncoder;
        this.jwtExpiration = 3600000L; // read from config
    }

    /**
     * Generate JWT token from Authentication object
     */
    public String generateToken(Authentication authentication) {
        Instant now = Instant.now();

        // Extract roles/authorities from authentication
        String roles = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(" "));

        // Build JWT claims
        JwtClaimsSet claims = JwtClaimsSet.builder()
                .issuer("self")                                    // Token issuer
                .issuedAt(now)                                     // Issued time
                .expiresAt(now.plus(jwtExpiration, ChronoUnit.MILLIS))  // Expiration time
                .subject(authentication.getName())                 // Username
                .claim("roles", roles)                            // User roles
                .build();

        // Encode and return token
        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }

    /**
     * Generate token with custom user details
     */
    public String generateToken(String username, String roles, Long userId) {
        Instant now = Instant.now();

        JwtClaimsSet claims = JwtClaimsSet.builder()
                .issuer("self")
                .issuedAt(now)
                .expiresAt(now.plus(jwtExpiration, ChronoUnit.MILLIS))
                .subject(username)
                .claim("roles", roles)
                .claim("userId", userId)
                .build();

        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }

    /**
     * Generate refresh token (longer expiration)
     */
    public String generateRefreshToken(Authentication authentication) {
        Instant now = Instant.now();
        Long refreshExpiration = 604800000L; // 7 days

        JwtClaimsSet claims = JwtClaimsSet.builder()
                .issuer("self")
                .issuedAt(now)
                .expiresAt(now.plus(refreshExpiration, ChronoUnit.MILLIS))
                .subject(authentication.getName())
                .claim("type", "refresh")
                .build();

        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }
}
```

## Configuring Security Filters and CORS

Configure Spring Security with JWT authentication, session management, CSRF, and CORS.

**SecurityConfig.java:**

```java
package com.example.security.config;

import com.example.security.jwt.JwtToAuthenticationConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationProvider;
import org.springframework.security.oauth2.server.resource.web.BearerTokenAuthenticationEntryPoint;
import org.springframework.security.oauth2.server.resource.web.access.BearerTokenAccessDeniedHandler;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.List;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                // Disable CSRF (not needed for stateless JWT authentication)
                .csrf(AbstractHttpConfigurer::disable)

                // Configure CORS
                .cors(cors -> cors.configurationSource(corsConfigurationSource()))

                // Configure URL authorization
                .authorizeHttpRequests(auth -> auth
                        // Public endpoints
                        .requestMatchers("/auth/**").permitAll()
                        .requestMatchers("/public/**").permitAll()
                        .requestMatchers(HttpMethod.GET, "/api/posts").permitAll()

                        // Admin endpoints
                        .requestMatchers("/admin/**").hasRole("ADMIN")

                        // All other endpoints require authentication
                        .anyRequest().authenticated()
                )

                // Configure OAuth2 Resource Server
                .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))

                // Configure session management (stateless)
                .sessionManagement(session -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                )

                // Configure exception handling
                .exceptionHandling(ex -> ex
                        .authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint())
                        .accessDeniedHandler(new BearerTokenAccessDeniedHandler())
                );

        return http.build();
    }

    /**
     * Configure CORS
     */
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();

        // Allow specific origins (change for production) //make it configurable
        configuration.setAllowedOrigins(List.of("http://localhost:3000", "http://localhost:4200"));

        // Allow specific HTTP methods
        configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));

        // Allow specific headers
        configuration.setAllowedHeaders(List.of("*"));

        // Allow credentials (cookies, authorization headers)
        configuration.setAllowCredentials(true);

        // Expose headers to client
        configuration.setExposedHeaders(List.of("Authorization"));

        // Max age for preflight requests
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);

        return source;
    }

    /**
     * Password encoder bean
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * Authentication Manager for username/password authentication
     */
    @Bean
    public AuthenticationManager authenticationManager(UserDetailsService uds, PasswordEncoder pe) {
        var authProvider = new DaoAuthenticationProvider(uds);
        authProvider.setPasswordEncoder(pe);
        return new ProviderManager(authProvider);
    }
}
```

### Configuration Explained

**1. CSRF Disabled:**
```java
.csrf(AbstractHttpConfigurer::disable)
```
- Not needed for stateless JWT authentication
- CSRF protects against cookie-based attacks
- JWT sent in Authorization header, not cookies

**2. Session Management:**
```java
.sessionManagement(session -> session
    .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
)
```
- `STATELESS` - No session created
- Each request must contain JWT token
- Server doesn't store any session data

**3. OAuth2 Resource Server:**
```java
.oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
```

- Configures JWT validation
- Uses `JwtDecoder` to validate tokens
- Converts JWT to Authentication using the registered JwtAuthenticationConverter

**4. Exception Handling:**
```java
.exceptionHandling(ex -> ex
    .authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint())
    .accessDeniedHandler(new BearerTokenAccessDeniedHandler())
)
```
- `BearerTokenAuthenticationEntryPoint` - Returns 401 Unauthorized
- `BearerTokenAccessDeniedHandler` - Returns 403 Forbidden

**5. CORS Configuration:**
- Allows cross-origin requests from specified origins
- Required for SPAs (React, Angular, Vue) on different ports
- Allows specific HTTP methods and headers

## Implementing Login Endpoint

Create an authentication controller to handle login requests.

### Step 1: Create User Entity

**User.java:**

```java
package com.example.security.entity;

import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(nullable = false)
    private String password;

    @Column(unique = true, nullable = false)
    private String email;

    private boolean enabled = true;

    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "role")
    private Set<String> roles = new HashSet<>();

    // Constructors, getters, setters
    public User() {}

    public User(String username, String password, String email) {
        this.username = username;
        this.password = password;
        this.email = email;
    }

    // Getters and Setters
}
```

### Step 2: Create Repository

**UserRepository.java:**

```java
package com.example.security.repository;

import com.example.security.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}
```

### Step 3: Implement UserDetailsService

**CustomUserDetailsService.java:**

```java
package com.example.security.service;

import com.example.security.entity.User;
import com.example.security.repository.UserRepository;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Collection;
import java.util.stream.Collectors;

@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));

        return org.springframework.security.core.userdetails.User
                .withUsername(user.getUsername())
                .password(user.getPassword())
                .authorities(getAuthorities(user))
                .disabled(!user.isEnabled())
                .build();
    }

    private Collection<? extends GrantedAuthority> getAuthorities(User user) {
        return user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .toList();
    }
}
```

### Step 4: Create DTOs

**LoginRequest.java:**

```java
package com.example.security.dto;

import jakarta.validation.constraints.NotBlank;

public record LoginRequest(
    @NotBlank(message = "Username is required")
    String username,

    @NotBlank(message = "Password is required")
    String password){
}
```

**LoginResponse.java:**

```java
package com.example.security.dto;

public record LoginResponse(
    String accessToken,
    String refreshToken,
    Long expiresIn){
}
```

**RegisterRequest.java:**

```java
package com.example.security.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record RegisterRequest(
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    String username,

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    String email,

    @NotBlank(message = "Password is required")
    @Size(min = 6, max = 100, message = "Password must be at least 6 characters")
    String password){
}
```

### Step 5: Create Authentication Controller

**AuthController.java:**

```java
package com.example.security.controller;

import com.example.security.dto.LoginRequest;
import com.example.security.dto.LoginResponse;
import com.example.security.dto.RegisterRequest;
import com.example.security.entity.User;
import com.example.security.jwt.JwtTokenProvider;
import com.example.security.repository.UserRepository;
import jakarta.validation.Valid;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider jwtTokenProvider;

    public AuthController(AuthenticationManager authenticationManager,
                          UserRepository userRepository,
                          PasswordEncoder passwordEncoder,
                          JwtTokenProvider jwtTokenProvider) {
        this.authenticationManager = authenticationManager;
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.jwtTokenProvider = jwtTokenProvider;
    }

    /**
     * Login endpoint - Returns JWT token
     */
    @PostMapping("/login")
    public ResponseEntity<?> login(@Valid @RequestBody LoginRequest loginRequest) {
        try {
            // Authenticate user
            Authentication authentication = authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(
                            loginRequest.username(),
                            loginRequest.password()
                    )
            );

            // Set authentication in security context
            SecurityContextHolder.getContext().setAuthentication(authentication);

            // Generate JWT tokens
            String accessToken = jwtTokenProvider.generateToken(authentication);
            String refreshToken = jwtTokenProvider.generateRefreshToken(authentication);

            // Return tokens
            LoginResponse response = new LoginResponse(accessToken, refreshToken, 3600000L);
            return ResponseEntity.ok(response);

        } catch (Exception e) {
            Map<String, String> error = new HashMap<>();
            error.put("error", "Invalid username or password");
            return ResponseEntity.badRequest().body(error);
        }
    }

    /**
     * Register endpoint - Creates new user
     */
    @PostMapping("/register")
    public ResponseEntity<?> register(@Valid @RequestBody RegisterRequest registerRequest) {
        // Check if username exists
        if (userRepository.existsByUsername(registerRequest.username())) {
            Map<String, String> error = new HashMap<>();
            error.put("error", "Username already exists");
            return ResponseEntity.badRequest().body(error);
        }

        // Check if email exists
        if (userRepository.existsByEmail(registerRequest.email())) {
            Map<String, String> error = new HashMap<>();
            error.put("error", "Email already exists");
            return ResponseEntity.badRequest().body(error);
        }

        // Create new user
        User user = new User();
        user.setUsername(registerRequest.username());
        user.setEmail(registerRequest.email());
        user.setPassword(passwordEncoder.encode(registerRequest.password()));
        user.getRoles().add("USER");

        userRepository.save(user);

        Map<String, String> response = new HashMap<>();
        response.put("message", "User registered successfully");
        return ResponseEntity.ok(response);
    }
}
```

### Step 6: Test with cURL

**Register a new user:**

```bash
curl -X POST http://localhost:8080/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john",
    "email": "john@example.com",
    "password": "password123"
  }'
```

**Response:**
```json
{
  "message": "User registered successfully"
}
```

**Login:**

```bash
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john",
    "password": "password123"
  }'
```

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600000
}
```

## Retrieving Current User from JWT

Now let's implement endpoints to retrieve the current authenticated user's details from the JWT token.

### Step 1: Create UserController

**UserController.java:**

```java
package com.example.security.controller;

import com.example.security.entity.User;
import com.example.security.repository.UserRepository;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/user")
public class UserController {

    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    /**
     * Get current user details from JWT token
     * Method 1: Using SecurityContextHolder
     */
    @GetMapping("/me")
    public ResponseEntity<?> getCurrentUser() {
        // Get authentication from security context
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication == null || !authentication.isAuthenticated()) {
            return ResponseEntity.status(401).body("Not authenticated");
        }

        // Get JWT from authentication
        Jwt jwt = (Jwt) authentication.getPrincipal();

        // Extract user details from JWT
        String username = jwt.getSubject();
        String roles = jwt.getClaimAsString("roles");

        // Fetch user from database
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new RuntimeException("User not found"));

        // Create response
        Map<String, Object> response = new HashMap<>();
        response.put("id", user.getId());
        response.put("username", user.getUsername());
        response.put("email", user.getEmail());
        response.put("roles", user.getRoles());

        return ResponseEntity.ok(response);
    }

    /**
     * Method 2: Using Authentication parameter
     */
    @GetMapping("/profile")
    public ResponseEntity<?> getUserProfile(Authentication authentication) {
        if (authentication == null) {
            return ResponseEntity.status(401).body("Not authenticated");
        }

        Jwt jwt = (Jwt) authentication.getPrincipal();
        String username = jwt.getSubject();

        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new RuntimeException("User not found"));

        Map<String, Object> response = new HashMap<>();
        response.put("id", user.getId());
        response.put("username", user.getUsername());
        response.put("email", user.getEmail());
        response.put("roles", user.getRoles());
        response.put("enabled", user.isEnabled());

        return ResponseEntity.ok(response);
    }

    /**
     * Get all JWT claims
     */
    @GetMapping("/claims")
    public ResponseEntity<?> getAllClaims(Authentication authentication) {
        Jwt jwt = (Jwt) authentication.getPrincipal();

        Map<String, Object> claims = new HashMap<>();
        claims.put("subject", jwt.getSubject());
        claims.put("issuer", jwt.getIssuer());
        claims.put("issuedAt", jwt.getIssuedAt());
        claims.put("expiresAt", jwt.getExpiresAt());
        claims.put("roles", jwt.getClaimAsString("roles"));
        claims.put("allClaims", jwt.getClaims());

        return ResponseEntity.ok(claims);
    }
}
```

### Step 2: Test with cURL

**Get current user (requires JWT):**

```bash
curl -X GET http://localhost:8080/api/user/me \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "id": 1,
  "username": "john",
  "email": "john@example.com",
  "roles": ["USER"]
}
```

**Get JWT claims:**

```bash
curl -X GET http://localhost:8080/api/user/claims \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "subject": "john",
  "issuer": "self",
  "issuedAt": "2024-01-15T10:30:00Z",
  "expiresAt": "2024-01-15T11:30:00Z",
  "roles": "ROLE_USER",
  "allClaims": {
    "sub": "john",
    "iss": "self",
    "iat": 1705318200,
    "exp": 1705321800,
    "roles": "ROLE_USER"
  }
}
```

## Best Practices and Security Considerations

### 1. Token Expiration

**Keep access tokens short-lived:**

```java
// Access token: 15-60 minutes
jwt.expiration=900000  // 15 minutes

// Refresh token: 7-30 days
jwt.refresh.expiration=604800000  // 7 days
```

**Implement refresh token mechanism:**

```java
@PostMapping("/refresh")
public ResponseEntity<?> refreshToken(@RequestBody Map<String, String> request) {
    String refreshToken = request.get("refreshToken");

    // Validate refresh token
    // Generate new access token

    return ResponseEntity.ok(new LoginResponse(newAccessToken, refreshToken, 3600000L));
}
```

### 2. Token Storage

**Client-side storage options:**

```javascript
// ✅ Good: In-memory (most secure)
let token = null;

function setToken(value) {
    token = value;
}

// ❌ Bad: localStorage (vulnerable to XSS)
localStorage.setItem('token', token);

// ⚠️ Okay: sessionStorage (better than localStorage)
sessionStorage.setItem('token', token);

// ✅ Best: httpOnly cookie (if using same domain)
// Set cookie from backend with httpOnly flag
```

### 3. Secure Key Management

**Never hardcode keys:**

```java
// ❌ Bad
String secret = "mySecretKey123";

// ✅ Good - Use environment variables
@Value("${jwt.private.key}")
private String privateKeyPath;

// ✅ Better - Use secret management services
// AWS Secrets Manager, Azure Key Vault, etc.
```

## Assignments

### Assignment 1: Basic JWT Authentication

**Objective:** Implement JWT authentication from scratch

**Tasks:**
1. Generate RSA keys using OpenSSL
2. Configure Spring Security with OAuth2 Resource Server
3. Implement login endpoint that returns JWT
4. Create a protected endpoint that requires JWT
5. Test with Postman or cURL

**Expected Output:**
- User can register and login
- JWT token returned on successful login
- Protected endpoints accessible only with valid JWT

### Assignment 2: User Profile Management

**Objective:** Implement user profile CRUD operations

**Tasks:**
1. Create endpoints to get current user profile
2. Implement update profile endpoint (only own profile)
3. Add change password functionality
4. Implement delete account endpoint

**Expected Output:**
- Users can view and update their profile
- Users can change their password
- Users can delete their account

### Assignment 3: Role-Based Access Control

**Objective:** Implement role-based authorization with JWT

**Tasks:**
1. Create USER and ADMIN roles
2. Implement admin-only endpoints
3. Use @PreAuthorize for method-level security
4. Create endpoints that require specific roles

**Expected Output:**
- Different access levels based on roles
- Admin can access all endpoints
- Regular users have limited access

### Assignment 4: Refresh Token Implementation

**Objective:** Implement refresh token mechanism

**Tasks:**
1. Generate refresh tokens (longer expiration)
2. Store refresh tokens in database
3. Implement /auth/refresh endpoint
4. Validate refresh tokens
5. Return new access token

**Expected Output:**
- Users get both access and refresh tokens on login
- Access token can be refreshed without re-login
- Refresh tokens can be invalidated

### Assignment 5: Post Management with JWT

**Objective:** Apply JWT security to jblogger REST API endpoints

**Tasks:**
1. Generate RSA keys using OpenSSL
2. Configure Spring Security with OAuth2 Resource Server
3. Secure endpoints with `SecurityFilterChain` and/or `@PreAuthorize`
4. Test JWT security with Postman

**Expected Output:**
- JWT security applied to jblogger endpoints
- Access to protected endpoints requires valid JWT


# Testing Spring Boot Applications

## Table of Contents
- [Introduction](#introduction)
- [Spring Boot Testing Support](#spring-boot-testing-support)
- [Unit Testing](#unit-testing)
- [Slice Testing](#slice-testing)
- [Integration Testing](#integration-testing)
- [Testing Secured REST API Endpoints](#testing-secured-rest-api-endpoints)
- [Best Practices](#best-practices)
- [Assignment](#assignment)

## Introduction

Testing is crucial for ensuring the reliability and maintainability of Spring Boot applications. 
Spring Boot provides comprehensive testing support through various testing annotations and utilities 
that make it easy to write unit tests, slice tests, and integration tests.

## Spring Boot Testing Support

Spring Boot provides a wide range of tools and technologies for testing Spring Boot applications.

The `spring-boot-starter-test` transitively includes several commonly used testing libraries:

- **JUnit 5/6**: The standard testing framework for Java applications
- **Spring Test**: Spring Framework's testing support
- **AssertJ**: Fluent assertion library
- **Hamcrest**: Matcher library for assertions
- **Mockito**: Mocking framework
- **JSONassert**: JSON assertion library
- **JsonPath**: XPath for JSON

### Adding Spring Boot Test Dependencies

Add the Spring Boot Test starter to your `pom.xml`:

```xml
<!--  Upto Spring Boot 3.x  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!--  From Spring Boot 4.x, we need to add test starters for specific technologies  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa-test</artifactId>
    <scope>test</scope>
</dependency>
```

## Unit Testing

Unit tests focus on testing individual components in isolation without loading the Spring context.

### Testing Service Layer without Spring
While writing unit tests for service layer components, we should be able to instantiate the component 
and invoke its methods without loading the Spring context.
If the subject under test depends on other components, we can use Mockito mocks as dependencies with stubbed behaviors.

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private PaymentService paymentService;

    @Mock
    private NotificationService notificationService;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldProcessOrderSuccessfully() {
        // Given
        Order order = new Order(1L, "ORDER-001", 100.0);
        when(orderRepository.save(any(Order.class))).thenReturn(order);
        when(paymentService.processPayment(anyDouble())).thenReturn(true);
        doNothing().when(notificationService).sendConfirmation(any(Order.class));

        // When
        Order result = orderService.placeOrder(order);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
    }

    @Test
    void shouldHandlePaymentFailure() {
        // Given
        Order order = new Order(1L, "ORDER-002", 200.0);
        when(paymentService.processPayment(anyDouble())).thenReturn(false);

        // When
        assertThatThrownBy(() -> orderService.placeOrder(order))
            .isInstanceOf(PaymentFailedException.class);

        // Then
        verify(paymentService).processPayment(200.0);
        verify(notificationService, never()).sendConfirmation(any());
    }
}
```

## Slice Testing

Slice tests focus on testing a specific "slice" of your application with minimal context loading.

### Testing Web Layer with @WebMvcTest

The `@WebMvcTest` based slice tests loads only the web layer components (controllers, filters, etc.) without the full application context.
So, if the controller depends on other components, we need to register them as mock beans using `@MockitoBean`.

```java
@WebMvcTest(BookController.class)
class BookControllerTest {

    @Autowired
    private MockMvcTester mockMvcTester;

    @MockitoBean
    private BookService bookService;

    @Test
    void shouldReturnBookWhenGetById() throws Exception {
        // Given
        Book book = new Book(1L, "Spring Boot in Action", "Craig Walls", "1234567890");
        when(bookService.getBookById(1L)).thenReturn(book);

        MvcTestResult testResult = mockMvcTester
                .post()
                .uri("/api/books/{id}", 1L)
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestBody)
                .exchange();
        
        assertThat(testResult)
                .hasStatusOk()
                .bodyJson()
                .isLenientlyEqualTo("""
                          {
                             "id": 1,
                             "title": "Spring Boot in Action",
                             "author": "Craig Walls"
                          }
                        """);
    }

    @Test
    void shouldReturnNotFoundWhenBookDoesNotExist() throws Exception {
        // Given
        when(bookService.getBookById(999L))
            .thenThrow(new BookNotFoundException("Book not found with id: 999"));

        // When & Then
        mockMvcTester
                .get()
                .uri("/api/books/{id}", 999L)
                .contentType(MediaType.APPLICATION_JSON)
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.NOT_FOUND);
    }
}
```

### Testing Data Layer with @DataJpaTest

`@DataJpaTest` configures an in-memory database if you have any in-memory database driver and loads only JPA-related components.

```java
@DataJpaTest
class BookRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldSaveAndFindBook() {
        // Given
        Book book = new Book(null, "Spring Boot Guide", "John Doe", "1234567890");
        entityManager.persist(book);
        entityManager.flush();

        // When
        Optional<Book> found = bookRepository.findById(book.getId());

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getTitle()).isEqualTo("Spring Boot Guide");
    }

    @Test
    void shouldFindBooksByAuthor() {
        // Given
        Book book1 = new Book(null, "Book 1", "Jane Smith", "111");
        Book book2 = new Book(null, "Book 2", "Jane Smith", "222");
        Book book3 = new Book(null, "Book 3", "John Doe", "333");

        entityManager.persist(book1);
        entityManager.persist(book2);
        entityManager.persist(book3);
        entityManager.flush();

        // When
        List<Book> books = bookRepository.findByAuthor("Jane Smith");

        // Then
        assertThat(books).hasSize(2);
        assertThat(books).extracting(Book::getAuthor)
            .containsOnly("Jane Smith");
    }
}
```

There many other slice testing annotations available such as `@JsonTest`, `@DataMongoTest`, `@JdbcTest`, etc. 

## Integration Testing

Integration tests load the complete application context and test the application as a whole.

### Using @SpringBootTest

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@AutoConfigureRestTestClient
class BookIntegrationTest {

    @Autowired
    protected MockMvcTester mockMvcTester;

    @Autowired
    protected RestTestClient restTestClient;
    
    // NOTE: You can use mockMvcTester to invoke API endpoints similar to the @WebMvcTest example.
    // However, I prefer to use restTestClient for invoking API endpoints.

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldCreateAndRetrieveBook() {
        // Given
        BookDTO newBook = new BookDTO("Integration Test Book", "Test Author", "INT-123");

        // When - Create book
        BookDTO bookDTO = restTestClient
                .post()
                .uri("/api/books")
                .contentType(MediaType.APPLICATION_JSON)
                .body(newBook)
                .exchange()
                .expectStatus().isCreated()
                .returnResult(BookDTO.class)
                .getResponseBody();
        
        // Then - Verify creation
        assertThat(bookDTO).isNotNull();
        Long bookId = bookDTO.getId();

        // When - Retrieve book
        BookDTO savedBook = restTestClient
                .get()
                .uri("/api/books/{id}", bookId)
                .exchange()
                .expectStatus().isOk()
                .returnResult(BookDTO.class)
                .getResponseBody();

        // Then - Verify retrieval
        assertThat(savedBook).isNotNull();
        assertThat(savedBook.getTitle()).isEqualTo("Integration Test Book");
    }
}
```

### Using Testcontainers for Database Testing
[Testcontainers](https://testcontainers.com/) help us to spin up application dependencies (such as databases, message brokers, and more) 
as docker containers in a controlled environment during tests. 
This is particularly useful for integration testing where the application needs to interact with external services.

Add Testcontainers dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>

<!-- Upto Spring Boot 3.x (Testcontainers 1.x) -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>

<!-- From Spring Boot 4.x (Testcontainers 2.x) -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers-postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

Create a test configuration:

```java
@TestConfiguration(proxyBeanMethods = false)
public class TestContainersConfiguration {

    @Bean
    @ServiceConnection
    PostgreSQLContainer postgresContainer() {
        return new PostgreSQLContainer("postgres:18-alpine");
    }
}
```

Use Testcontainers in integration tests:

```java
import org.springframework.context.annotation.Import;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Import(TestContainersConfiguration.class)
class BookIntegrationWithTestcontainersTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldPersistBookInPostgres() {
        //...
    }

    @Test
    void shouldHandleTransactions() {
        //...
    }
}
```

Now when you run the tests, Testcontainers will spin up a PostgreSQL database for you and configure the application to use it.

**NOTE:** In `@SpringBootTest` integration tests also you can use `@MockitoBean` to mock a Spring bean.

## Testing Secured REST API Endpoints

Add security test dependency:

```xml
<!-- Upto Spring Boot 3.x (Testcontainers 1.x) -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- From Spring Boot 4.x (Testcontainers 2.x) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

### Testing with Spring Security

You can use `@WithMockUser`, `@WithUserDetails` annotations to simulate authenticated user.

You can also simulate authenticated user request using `MockMvcTester.with(user(...))` as well.

```java
@SpringBootTest
@AutoConfigureMockMvc
class SecuredBookControllerTest {

    @Autowired
    private MockMvcTester mockMvcTester;

    @Test
    void shouldDenyAccessWithoutAuthentication() throws Exception {
        mockMvcTester
                .get()
                .uri("/api/books")
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.UNAUTHORIZED);
    }

    @Test
    @WithMockUser(username = "user", roles = {"USER"})
    void shouldAllowAccessWithAuthentication() throws Exception {
        mockMvcTester
                .get()
                .uri("/api/books")
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.OK);
    }

    @Test
    void shouldAuthenticateUsingWithUser() throws Exception {
        mockMvcTester
                .get()
                .uri("/api/books")
                .with(user("user")) // <--- note this
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.OK);
    }

    @Test
    @WithUserDetails("admin@example.com")
    void shouldUseActualUserFromDatabase() throws Exception {
        mockMvcTester
                .get()
                .uri("/api/books")
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.OK);
    }
}
```

### Testing with a Real JWT Token

Instead of mocking authentication, you can generate a real JWT token using `JwtTokenProvider` and use it in your tests.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureRestTestClient
class JwtSecuredEndpointTest {

    @Autowired
    private MockMvcTester mockMvcTester;
    
    @Autowired
    private RestTestClient restTestClient;
    
    @Autowired
    private JwtTokenProvider tokenProvider;

    @Test
    void shouldAccessProtectedEndpointWithValidToken() {
        // Given
        String token = tokenProvider.generateToken("user@example.com", List.of("ROLE_USER"));

        // When
        
        //Using MockMvcTester
        assertThat(mockMvcTester.get().uri("/api/books")
                .header("Authorization", "Bearer " + token))
                .hasStatusOk();
        
        // Using RestTestClient        
        BookDTO[] books = restTestClient
                .get()
                .uri("/api/books")
                .header("Authorization", "Bearer " + token)
                .exchange()
                .expectStatus()
                .isOk()
                .returnResult(BookDTO[].class)
                .getResponseBody();
        
        // Then
        assertThat(books).isNotNull();
    }
}
```

## Best Practices

1. **Use the Right Test Type**:
   - Unit tests for business logic
   - Slice tests for specific layers
   - Integration tests for end-to-end flows

2. **Follow AAA Pattern**: Arrange, Act, Assert
   ```java
   @Test
   void testMethod() {
       // Arrange (Given)
       // Act (When)
       // Assert (Then)
   }
   ```

3. **Use Descriptive Test Names**:
   ```java
   shouldReturnBookWhenValidIdProvided()
   shouldThrowExceptionWhenBookNotFound()
   ```

4. **Isolate Tests**: Each test should be independent
   ```java
   @BeforeEach
   void setUp() {
       bookRepository.deleteAll();
   }
   ```

5. **Use Test Fixtures**: Create reusable test data
   ```java
   class BookFixtures {
       static Book createDefaultBook() {
           return new Book(null, "Default Book", "Default Author", "123");
       }
   }
   ```

6. **External Dependencies**: Use Testcontainers or WireMock for external services

7. **Test Edge Cases**: Include boundary conditions, null values, and error scenarios

8. **Use AssertJ for Readable Assertions**:
   ```java
   assertThat(book)
       .isNotNull()
       .extracting(Book::getTitle)
       .isEqualTo("Expected Title");
   ```

## Assignment

Create a comprehensive test suite for a jblogger with the following requirements:

1. **Unit Tests**:
   - Test `PostService` with mocked `PostRepository`
   - Test `UserService` with mocked dependencies
   - Include edge cases and error scenarios

2. **Slice Tests**:
   - `@WebMvcTest` for `PostController`
   - `@DataJpaTest` for `PostRepository` custom queries
   - Test validation constraints

3. **Integration Tests**:
   - Use `@SpringBootTest` with Testcontainers (PostgreSQL)
   - Test all API endpoints using `TestRestTemplate` or `MockMvcTester` 

4. **Security Tests**:
   - Test authenticated and unauthenticated access
   - Test role-based authorization (USER, ADMIN)
   - Test JWT token validation

5. **Coverage Goals**:
   - Aim for >80% code coverage
   - Include happy path and error scenarios


# Docker Compose Deployment

## Table of Contents
- [Introduction](#introduction)
- [Dockerizing a Spring Boot Application](#dockerizing-a-spring-boot-application)
- [Using Paketo Buildpacks](#using-paketo-buildpacks)
- [Deploying with Docker Compose](#deploying-with-docker-compose)
- [Assignments](#assignments)

## Introduction

Docker enables you to package your Spring Boot application with all its dependencies into a standardized container 
that can run consistently across different environments. Docker Compose simplifies the orchestration of multi-container applications, 
making it ideal for deploying Spring Boot applications with databases and other services.

In this guide, you'll learn multiple approaches to containerize your Spring Boot application and 
deploy it alongside a PostgreSQL database using Docker Compose.

## Dockerizing a Spring Boot Application

### Using Dockerfile

The simplest approach is to create a Dockerfile that copies your JAR file and runs it.

#### Basic Dockerfile

```dockerfile
# Use an official OpenJDK runtime as base image
FROM eclipse-temurin:25-jre

# Set the working directory
WORKDIR /app

# Copy the JAR file into the container
COPY target/*.jar app.jar

# Expose the application port
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### Building and Running

```bash
# Build your Spring Boot application
./mvnw clean package

# Build the Docker image
docker build -t my-spring-app:1.0 .

# Run the container
docker run -p 8080:8080 my-spring-app:1.0
```

### Using Multistage Dockerfile with Layer Caching

Multistage builds optimize your Docker images by separating the build environment from the runtime environment. 
They also enable better layer caching, reducing build times.

#### Multistage Dockerfile

```dockerfile
# Stage 1: Build stage
FROM eclipse-temurin:25-jdk-alpine AS builder

WORKDIR /workspace/app

# Copy Maven wrapper and pom.xml first (for dependency caching)
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .

# Download dependencies (this layer will be cached)
RUN ./mvnw dependency:go-offline -B

# Copy source code
COPY src src

# Build the application
RUN ./mvnw package -DskipTests

# Extract layers for better caching
RUN java -Djarmode=layertools -jar target/*.jar extract

# Stage 2: Runtime stage
FROM eclipse-temurin:25-jre

WORKDIR /app

# Copy extracted layers from builder stage
COPY --from=builder /app/dependencies/ ./
COPY --from=builder /app/spring-boot-loader/ ./
COPY --from=builder /app/snapshot-dependencies/ ./
COPY --from=builder /app/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
```

#### Benefits of Layered Approach

1. **Faster Builds**: Dependencies are cached separately from application code
2. **Smaller Images**: Only necessary runtime components are included
3. **Optimal Caching**: Each layer is cached independently

#### Building with Multistage Dockerfile

```bash
# Build the image (no need to run mvnw separately)
docker build -t my-spring-app:1.0 .

# The build happens inside Docker, ensuring consistency
```

### Using Paketo Buildpacks

Paketo Buildpacks provide a modern, cloud-native approach to building container images without writing Dockerfiles. 
They automatically detect your application type and apply best practices.

#### What are Buildpacks?

Buildpacks are tools that transform your application source code into container images without requiring a Dockerfile. They:
- Automatically detect application type
- Install required dependencies
- Apply security patches
- Optimize for production
- Follow best practices

#### Using Spring Boot Maven Plugin with Buildpacks

The Spring Boot Maven plugin includes built-in support for Buildpacks.

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

Now we can build the image using the `spring-boot:build-image` goal:

```bash
# Build image using Buildpacks
./mvnw spring-boot:build-image

# The image name defaults to: docker.io/library/${project.artifactId}:${project.version}
```

#### Customizing Buildpack Configuration in pom.xml

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <image>
                    <name>my-registry.com/${project.artifactId}:${project.version}</name>
                    <env>
                        <BP_JVM_VERSION>25</BP_JVM_VERSION>
                        <BPE_JAVA_TOOL_OPTIONS>-XX:MaxRAMPercentage=75.0</BPE_JAVA_TOOL_OPTIONS>
                    </env>
                    <buildpacks>
                        <buildpack>paketo-buildpacks/java</buildpack>
                    </buildpacks>
                </image>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### Running the Buildpack-Created Image

```bash
# Run the image
docker run -p 8080:8080 my-spring-app:1.0
```

#### Benefits of Paketo Buildpacks

1. **No Dockerfile Required**: Automated image creation
2. **Security**: Regular security updates and patches
3. **Optimization**: Automatic layer optimization and caching
4. **Consistency**: Standard images across teams
5. **Best Practices**: Built-in production-ready configurations

## Deploying with Docker Compose

Docker Compose allows you to define and run multi-container applications. 
It's perfect for deploying Spring Boot applications with databases and other services.

### Creating docker-compose.yml

Create a `docker-compose.yml` file in your project root:

```yaml
services:
  # PostgreSQL Database
  postgres:
    image: postgres:18-alpine
    environment:
      POSTGRES_DB: myappdb
      POSTGRES_USER: dbuser
      POSTGRES_PASSWORD: dbpassword
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      # If you want to run initialization scripts to initialize the database,
      # keep the SQL scripts in init-scripts directory in the same directory as docker-compose.yml
      # - ./init-scripts:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dbuser -d myappdb"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Spring Boot Application
  app:
    image: my-spring-app:1.0
    container_name: spring-boot-app
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/myappdb
      SPRING_DATASOURCE_USERNAME: dbuser
      SPRING_DATASOURCE_PASSWORD: dbpassword
    ports:
      - "8080:8080"
    restart: unless-stopped

volumes:
  postgres-data:
    driver: local
```

### Running the Application

#### Start All Services

```bash
# Build and start all services in detached mode
docker compose up -d --build

# View logs
docker compose logs -f

# View logs for specific service
docker compose logs -f app
```

Now you can access the application at http://localhost:8080.

#### Common Docker Compose Commands

```bash
# Stop all services
docker compose stop

# Stop and remove containers, networks
docker compose down

# Stop and remove containers, networks, and volumes
docker compose down -v

# Restart a specific service
docker compose restart app

# View running services
docker compose ps

# Execute command in running container
docker compose exec app bash

# View resource usage
docker compose stats

# Rebuild specific service
docker compose up -d --build app
```


## Assignments

### Assignment 1: Basic Dockerization
**Objective**: Create a basic Dockerfile for your Spring Boot application.

**Tasks**:
1. Create a simple Dockerfile that packages your Spring Boot JAR
2. Build the Docker image
3. Run the container and test the application

### Assignment 2: Multistage Dockerfile
**Objective**: Optimize your Docker image using multistage builds.

**Tasks**:
1. Convert your basic Dockerfile to a multistage build
2. Implement layer caching for dependencies
3. Compare the image sizes before and after

### Assignment 3: Paketo Buildpacks
**Objective**: Build your application using Paketo Buildpacks.

**Tasks**:
1. Configure the Spring Boot Maven plugin
2. Build the image using `./mvnw spring-boot:build-image`
3. Customize environment variables for JVM settings
4. Compare build time and image size with Dockerfile approach
5. Document the differences and benefits

### Assignment 4: Docker Compose Deployment
**Objective**: Deploy your application with PostgreSQL using Docker Compose.

**Tasks**:
1. Create a `docker-compose.yml` with jblogger app and PostgreSQL
2. Configure environment variables for database connection
3. Add a volume for database persistence
4. Test the complete setup


# Conclusion

Congratulations on completing this Spring Boot self-learning guide! 

Throughout this comprehensive journey, you've built a solid foundation in Spring Boot development 
and gained practical experience through hands-on assignments.

## What You've Learned

This guide covered the complete lifecycle of building a Spring Boot application:

**Foundation & Core Concepts**
- Understanding Spring Boot's fundamentals, auto-configuration, and dependency injection
- Working with Spring Boot starters and the application structure
- Implementing dependency injection and component scanning

**Building REST APIs**
- Creating RESTful web services using Spring MVC
- Implementing CRUD operations with proper HTTP methods
- Handling request/response mapping, validation, and error handling
- Working with different HTTP status codes and response entities

**Data Persistence**
- Using Spring JdbcClient for direct database interactions
- Managing database transactions with `@Transactional`
- Implementing database migrations using Flyway
- Leveraging Spring Data JPA for simplified data access
- Understanding JPA repositories, query methods, etc

**Security**
- Implementing authentication and authorization with Spring Security
- Configuring security filters, authentication managers, and password encoding
- Building JWT-based authentication systems
- Applying method-level security with annotations and SpEL expressions

**Testing & Quality**
- Writing unit tests for services and components
- Creating slice tests for web, JPA, and security layers
- Implementing integration tests for end-to-end scenarios
- Using TestContainers for realistic database testing
- Testing secured endpoints and authentication flows

**Deployment**
- Containerizing Spring Boot applications with Docker
- Orchestrating multi-container applications using Docker Compose

## Final Thoughts

Spring Boot's convention-over-configuration approach and rich ecosystem make it an excellent choice for building modern Java applications. 
The hands-on experience you've gained through the jblogger application provides a strong foundation for building your own projects.

Remember that learning is an ongoing process. Continue practicing, explore the official Spring documentation, 
and engage with the Spring community. Build real projects, experiment with different features, 
and don't hesitate to dive deeper into areas that interest you most.

Happy coding, and best of luck with your Spring Boot journey!

## Resources

- [Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/redirect.html)

