# Migration Plan: java (Java 7 → Java 21)

## Overview

This migration plan provides a comprehensive, step-by-step guide to upgrade the Java codebase from Java 7 to Java 21. It covers all layers: bootstrap/build system, model/entity, data/repository, service, web/controller, configuration, utilities, and test cases. Each task specifies exact file paths, detailed instructions, and dependencies to ensure a smooth, automated migration. Key milestones include updating dependencies, refactoring deprecated APIs, modernizing configuration, and validating with updated test suites.

---

## Migration Tasks

---

### Task 0: Update Build System and Project Bootstrap

**Source Files:**
- /downloaded_code/pom.xml
- /downloaded_code/build.gradle
- /downloaded_code/build.xml
- /downloaded_code/src/main/java/com/example/MainApp.java
- /downloaded_code/src/main/resources/application.properties
- /downloaded_code/src/main/resources/beans.xml

**Target Files:**
- /downloaded_code/pom.xml
- /downloaded_code/build.gradle
- /downloaded_code/src/main/java/com/example/MainApp.java
- /downloaded_code/src/main/resources/application.yml

**Instructions:**
1. **Update Maven/Gradle Java Version:**
   - In `/downloaded_code/pom.xml`, find `<maven-compiler-plugin>` configuration.
     - Change `<source>7</source>` and `<target>7</target>` to `<source>21</source>` and `<target>21</target>`.
     - Update plugin versions:
       - `maven-compiler-plugin` to `3.10.1`
       - `maven-surefire-plugin` to `3.0.0-M7`
     - Add Spring Boot BOM:
       ```xml
       <dependencyManagement>
         <dependencies>
           <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-dependencies</artifactId>
             <version>3.2.5</version>
             <type>pom</type>
             <scope>import</scope>
           </dependency>
         </dependencies>
       </dependencyManagement>
       ```
   - In `/downloaded_code/build.gradle`, set `sourceCompatibility = '21'` and `targetCompatibility = '21'`. Update plugins:
     - `id 'org.springframework.boot' version '3.2.5'`
     - `id 'io.spring.dependency-management' version '1.1.4'`
2. **Remove Ant Build Scripts:**
   - Delete `/downloaded_code/build.xml`.
   - Migrate any custom build tasks to Maven/Gradle equivalents.
3. **Refactor Bootstrap Class:**
   - In `/downloaded_code/src/main/java/com/example/MainApp.java`, replace:
     ```java
     public class MainApp {
         public static void main(String[] args) {
             ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
             MyService service = (MyService) ctx.getBean("myService");
             service.run();
         }
     }
     ```
     with:
     ```java
     @SpringBootApplication
     public class MainApp {
         public static void main(String[] args) {
             SpringApplication.run(MainApp.class, args);
         }
     }
     ```
     - Add import: `import org.springframework.boot.SpringApplication;`
     - Add import: `import org.springframework.boot.autoconfigure.SpringBootApplication;`
   - Remove all references to `beans.xml` and manual bean loading.
4. **Migrate Properties to YAML:**
   - Convert `/downloaded_code/src/main/resources/application.properties` to `/downloaded_code/src/main/resources/application.yml`.
     - For each property, convert:
       ```
       db.driver=com.mysql.jdbc.Driver
       db.url=jdbc:mysql://localhost:3306/mydb
       db.username=root
       db.password=pass
       ```
       to:
       ```yaml
       db:
         driver: com.mysql.jdbc.Driver
         url: jdbc:mysql://localhost:3306/mydb
         username: root
         password: pass
       ```
   - Update all references in code/config to use `application.yml`.
5. **Remove beans.xml:**
   - Delete `/downloaded_code/src/main/resources/beans.xml`.
   - Ensure all bean definitions are migrated to annotation-based configuration.
6. **Validation:**
   - Run `mvn clean install` or `gradle build`.
   - Application should start via `SpringApplication.run(MainApp.class, args);` without XML bean config.

**Dependencies:** None

---

### Task 1: Migrate Model/Entity Layer to Java 21

**Source Files:**
- /downloaded_code/src/main/java/com/example/entity/UserEntity.java
- /downloaded_code/src/main/java/com/example/entity/OrderEntity.java
- /downloaded_code/src/main/java/com/example/entity/ProductEntity.java
- /downloaded_code/src/main/java/com/example/dto/UserDTO.java
- /downloaded_code/src/main/java/com/example/dto/OrderDTO.java
- /downloaded_code/src/main/java/com/example/model/Address.java
- /downloaded_code/src/main/java/com/example/model/Customer.java
- /downloaded_code/src/main/resources/hibernate.cfg.xml
- /downloaded_code/src/main/resources/persistence.xml

**Target Files:**
- /downloaded_code/src/main/java/com/example/entity/UserEntity.java
- /downloaded_code/src/main/java/com/example/entity/OrderEntity.java
- /downloaded_code/src/main/java/com/example/entity/ProductEntity.java
- /downloaded_code/src/main/java/com/example/dto/UserDTO.java
- /downloaded_code/src/main/java/com/example/dto/OrderDTO.java
- /downloaded_code/src/main/java/com/example/model/Address.java
- /downloaded_code/src/main/java/com/example/model/Customer.java
- /downloaded_code/src/main/java/com/example/config/HibernateConfig.java

**Instructions:**
1. **Refactor Date/Calendar Fields:**
   - In each entity class (e.g., `/downloaded_code/src/main/java/com/example/entity/UserEntity.java`), find:
     ```java
     private Date createdDate;
     ```
     Replace with:
     ```java
     private LocalDateTime createdDate;
     ```
     - Add import: `import java.time.LocalDateTime;`
     - Update getter/setter methods to use `LocalDateTime`.
     - Update Hibernate mapping annotations if necessary (`@Convert` or custom converter).
2. **Update DTOs to Records:**
   - In `/downloaded_code/src/main/java/com/example/dto/UserDTO.java`, replace:
     ```java
     public class UserDTO {
         private Long id;
         private String name;
         // constructor, getters
     }
     ```
     with:
     ```java
     public record UserDTO(Long id, String name) {}
     ```
   - Remove manual getter/setter and constructor code.
3. **Update Hibernate Configuration:**
   - In `/downloaded_code/src/main/resources/hibernate.cfg.xml` and `/downloaded_code/src/main/resources/persistence.xml`, update schema to latest Hibernate 6.x.
   - Remove deprecated properties (e.g., `hibernate.dialect` if outdated).
   - Migrate configuration to Java-based config:
     - Create `/downloaded_code/src/main/java/com/example/config/HibernateConfig.java`:
       ```java
       @Configuration
       public class HibernateConfig {
           @Bean
           public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) {
               LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
               emf.setDataSource(dataSource);
               emf.setPackagesToScan("com.example.entity");
               emf.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
               return emf;
           }
       }
       ```
     - Add necessary imports.
4. **Replace SimpleDateFormat:**
   - In all model/entity classes, find usages of `SimpleDateFormat`.
     - Replace with `DateTimeFormatter`:
       ```java
       DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
       ```
       - Add import: `import java.time.format.DateTimeFormatter;`
5. **Validation:**
   - Run all entity-related unit tests.
   - Verify database schema compatibility and data migration.

**Dependencies:** Task 0

---

### Task 2: Migrate Data/Repository Layer to Spring Data JPA

**Source Files:**
- /downloaded_code/src/main/java/com/example/dao/UserDaoImpl.java
- /downloaded_code/src/main/java/com/example/dao/OrderDaoImpl.java
- /downloaded_code/src/main/java/com/example/dao/ProductDaoImpl.java
- /downloaded_code/src/main/java/com/example/dao/HibernateUtil.java
- /downloaded_code/src/main/java/com/example/repository/UserRepository.java
- /downloaded_code/src/main/java/com/example/repository/OrderRepository.java
- /downloaded_code/src/main/java/com/example/repository/ProductRepository.java
- /downloaded_code/src/main/resources/hibernate/User.hbm.xml
- /downloaded_code/src/main/resources/hibernate/Order.hbm.xml
- /downloaded_code/src/main/resources/hibernate/Product.hbm.xml
- /downloaded_code/src/main/resources/applicationContext.xml
- /downloaded_code/src/main/resources/jdbc.properties

**Target Files:**
- /downloaded_code/src/main/java/com/example/repository/UserRepository.java
- /downloaded_code/src/main/java/com/example/repository/OrderRepository.java
- /downloaded_code/src/main/java/com/example/repository/ProductRepository.java
- /downloaded_code/src/main/java/com/example/config/DatabaseConfig.java

**Instructions:**
1. **Refactor DAOs to Repositories:**
   - For each DAO (e.g., `/downloaded_code/src/main/java/com/example/dao/UserDaoImpl.java`), extract interface methods and create repository:
     - In `/downloaded_code/src/main/java/com/example/repository/UserRepository.java`:
       ```java
       @Repository
       public interface UserRepository extends JpaRepository<User, Long> {
           Optional<User> findByUsername(String username);
       }
       ```
       - Add import: `import org.springframework.data.jpa.repository.JpaRepository;`
       - Add import: `import org.springframework.stereotype.Repository;`
   - Remove DAO implementation classes.
2. **Remove HibernateUtil and SessionFactory:**
   - Delete `/downloaded_code/src/main/java/com/example/dao/HibernateUtil.java`.
   - Remove all usages of `SessionFactory` and manual session management.
3. **Convert Hibernate XML Mappings to Annotations:**
   - In `/downloaded_code/src/main/resources/hibernate/User.hbm.xml`, `/downloaded_code/src/main/resources/hibernate/Order.hbm.xml`, `/downloaded_code/src/main/resources/hibernate/Product.hbm.xml`, migrate all mappings to JPA annotations in entity classes.
     - Example: Add `@Entity`, `@Table`, `@Id`, `@Column` annotations in `/downloaded_code/src/main/java/com/example/entity/UserEntity.java`.
   - Delete XML mapping files after migration.
4. **Migrate Configuration to Java:**
   - Create `/downloaded_code/src/main/java/com/example/config/DatabaseConfig.java`:
     ```java
     @Configuration
     @EnableJpaRepositories(basePackages = "com.example.repository")
     public class DatabaseConfig {
         @Bean
         public DataSource dataSource() {
             // Use properties from application.yml
         }
     }
     ```
     - Add import: `import org.springframework.context.annotation.Configuration;`
     - Add import: `import org.springframework.data.jpa.repository.config.EnableJpaRepositories;`
5. **Update Transaction Management:**
   - Annotate service/repository classes with `@Transactional`.
   - Remove manual transaction code from DAOs.
6. **Validation:**
   - Run integration tests for all repository methods.
   - Verify data access and transaction boundaries.

**Dependencies:** Task 1

---

### Task 3: Refactor Service Layer to Modern Spring Patterns

**Source Files:**
- /downloaded_code/src/main/java/com/example/service/UserService.java
- /downloaded_code/src/main/java/com/example/service/UserServiceImpl.java
- /downloaded_code/src/main/java/com/example/service/OrderService.java
- /downloaded_code/src/main/java/com/example/service/OrderServiceImpl.java
- /downloaded_code/src/main/resources/applicationContext.xml

**Target Files:**
- /downloaded_code/src/main/java/com/example/service/UserServiceImpl.java
- /downloaded_code/src/main/java/com/example/service/OrderServiceImpl.java
- /downloaded_code/src/main/java/com/example/config/ServiceConfig.java

**Instructions:**
1. **Annotate Service Classes:**
   - In `/downloaded_code/src/main/java/com/example/service/UserServiceImpl.java` and `/downloaded_code/src/main/java/com/example/service/OrderServiceImpl.java`, add:
     ```java
     @Service
     public class UserServiceImpl implements UserService { ... }
     ```
     - Add import: `import org.springframework.stereotype.Service;`
2. **Refactor to Constructor Injection:**
   - Replace manual instantiation:
     ```java
     private UserRepository userRepository = new UserRepositoryImpl();
     ```
     with:
     ```java
     private final UserRepository userRepository;
     @Autowired
     public UserServiceImpl(UserRepository userRepository) {
         this.userRepository = userRepository;
     }
     ```
     - Add import: `import org.springframework.beans.factory.annotation.Autowired;`
3. **Replace Custom Transaction Management:**
   - Remove all usages of `TransactionTemplate` and manual commit/rollback.
   - Annotate methods with `@Transactional`:
     ```java
     @Transactional
     public void createUser(User user) { ... }
     ```
     - Add import: `import org.springframework.transaction.annotation.Transactional;`
4. **Remove XML Bean Definitions:**
   - Delete service bean definitions from `/downloaded_code/src/main/resources/applicationContext.xml`.
   - Ensure component scanning is enabled in main application class.
5. **Validation:**
   - Run unit and integration tests for all service methods.
   - Verify transaction boundaries and bean lifecycle.

**Dependencies:** Task 2

---

### Task 4: Migrate Web/Controller Layer to Java 21 and Jakarta EE

**Source Files:**
- /downloaded_code/src/main/java/com/example/web/UserServlet.java
- /downloaded_code/src/main/java/com/example/web/OrderServlet.java
- /downloaded_code/src/main/java/com/example/action/UserAction.java
- /downloaded_code/src/main/java/com/example/action/OrderAction.java
- /downloaded_code/src/main/webapp/WEB-INF/web.xml
- /downloaded_code/src/main/webapp/views/user.jsp
- /downloaded_code/src/main/webapp/views/order.jsp

**Target Files:**
- /downloaded_code/src/main/java/com/example/web/UserServlet.java
- /downloaded_code/src/main/java/com/example/web/OrderServlet.java
- /downloaded_code/src/main/java/com/example/action/UserAction.java
- /downloaded_code/src/main/java/com/example/action/OrderAction.java
- /downloaded_code/src/main/webapp/WEB-INF/web.xml
- /downloaded_code/src/main/webapp/views/user.jsp
- /downloaded_code/src/main/webapp/views/order.jsp

**Instructions:**
1. **Update Servlet Imports:**
   - In `/downloaded_code/src/main/java/com/example/web/UserServlet.java` and `/downloaded_code/src/main/java/com/example/web/OrderServlet.java`, replace:
     ```java
     import javax.servlet.http.HttpServlet;
     import javax.servlet.http.HttpServletRequest;
     import javax.servlet.http.HttpServletResponse;
     ```
     with:
     ```java
     import jakarta.servlet.http.HttpServlet;
     import jakarta.servlet.http.HttpServletRequest;
     import jakarta.servlet.http.HttpServletResponse;
     ```
2. **Add Annotation-Based Mappings:**
   - Add `@WebServlet("/user")` and `@WebServlet("/order")` annotations to servlet classes.
     - Add import: `import jakarta.servlet.annotation.WebServlet;`
   - Remove corresponding servlet mappings from `/downloaded_code/src/main/webapp/WEB-INF/web.xml`.
3. **Refactor Raw Types and Enumeration Usage:**
   - Replace:
     ```java
     List users = new ArrayList();
     Enumeration names = req.getParameterNames();
     while (names.hasMoreElements()) {
         String name = (String) names.nextElement();
         users.add(name);
     }
     ```
     with:
     ```java
     List<String> users = Collections.list(req.getParameterNames());
     ```
     - Add import: `import java.util.Collections;`
     - Use generics throughout.
4. **Refactor Anonymous Inner Classes to Lambdas:**
   - Replace anonymous inner classes in controller logic with lambda expressions.
5. **Update JSPs:**
   - In `/downloaded_code/src/main/webapp/views/user.jsp` and `/downloaded_code/src/main/webapp/views/order.jsp`, remove all scriptlets (`<% ... %>`) and replace with JSTL/EL expressions.
     - Add JSTL taglib:
       ```jsp
       <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
       ```
     - Replace `${user.name}` scriptlet with EL: `${user.name}`
6. **Update web.xml Schema:**
   - Update `/downloaded_code/src/main/webapp/WEB-INF/web.xml` to Servlet 5.0 schema.
     - Remove deprecated elements.
     - Ensure compatibility with Jakarta EE 10.
7. **Validation:**
   - Deploy application to Jakarta EE 10-compatible servlet container.
   - Access `/user` and `/order` endpoints; verify correct behavior.

**Dependencies:** Task 3

---

### Task 5: Migrate Configuration to Spring Boot Standards

**Source Files:**
- /downloaded_code/src/main/resources/spring/applicationContext.xml
- /downloaded_code/src/main/resources/spring/datasource.xml
- /downloaded_code/src/main/resources/spring/services.xml
- /downloaded_code/src/main/resources/config.properties
- /downloaded_code/src/main/resources/db.properties
- /downloaded_code/src/main/resources/spring/beans.xml
- /downloaded_code/src/main/resources/spring/context.xml

**Target Files:**
- /downloaded_code/src/main/java/com/example/config/AppConfig.java
- /downloaded_code/src/main/java/com/example/config/DataSourceConfig.java
- /downloaded_code/src/main/resources/application.yml

**Instructions:**
1. **Migrate XML to Java Config:**
   - For each XML config (e.g., `/downloaded_code/src/main/resources/spring/applicationContext.xml`), create corresponding Java config class:
     - `/downloaded_code/src/main/java/com/example/config/AppConfig.java`:
       ```java
       @Configuration
       public class AppConfig { }
       ```
     - `/downloaded_code/src/main/java/com/example/config/DataSourceConfig.java`:
       ```java
       @Configuration
       public class DataSourceConfig {
           @Value("${db.driver}")
           private String driverClassName;
           @Value("${db.url}")
           private String url;
           @Value("${db.username}")
           private String username;
           @Value("${db.password}")
           private String password;

           @Bean
           public DataSource dataSource() {
               BasicDataSource ds = new BasicDataSource();
               ds.setDriverClassName(driverClassName);
               ds.setUrl(url);
               ds.setUsername(username);
               ds.setPassword(password);
               return ds;
           }
       }
       ```
   - Add necessary imports.
2. **Merge Properties Files:**
   - Consolidate `/downloaded_code/src/main/resources/config.properties` and `/downloaded_code/src/main/resources/db.properties` into `/downloaded_code/src/main/resources/application.yml`.
     - Convert all properties to YAML format.
3. **Remove PropertyPlaceholderConfigurer:**
   - Delete all usages in XML and Java.
   - Use `@Value` or `@ConfigurationProperties` for property injection.
4. **Refactor Bean Definitions:**
   - Replace `<bean>` XML definitions with `@Bean` methods in Java config.
   - Remove all XML bean files after migration.
5. **Update Resource Management:**
   - Refactor manual resource management in config classes to use try-with-resources.
6. **Validation:**
   - Run application and verify all beans are loaded via Java config.
   - Check property injection from `application.yml`.

**Dependencies:** Task 4

---

### Task 6: Refactor Utility & Support Files to Java 21 APIs

**Source Files:**
- /downloaded_code/src/main/java/com/example/util/StringUtils.java
- /downloaded_code/src/main/java/com/example/util/DateUtils.java
- /downloaded_code/src/main/java/com/example/util/ConfigLoader.java
- /downloaded_code/src/main/java/com/example/helper/FileHelper.java
- /downloaded_code/src/main/java/com/example/validator/EmailValidator.java
- /downloaded_code/src/main/java/com/example/validator/NullCheckValidator.java
- /downloaded_code/src/main/java/com/example/exception/ValidationException.java
- /downloaded_code/src/main/java/com/example/exception/ConfigException.java

**Target Files:**
- /downloaded_code/src/main/java/com/example/util/StringUtils.java
- /downloaded_code/src/main/java/com/example/util/DateUtils.java
- /downloaded_code/src/main/java/com/example/util/ConfigLoader.java
- /downloaded_code/src/main/java/com/example/helper/FileHelper.java
- /downloaded_code/src/main/java/com/example/validator/EmailValidator.java
- /downloaded_code/src/main/java/com/example/validator/NullCheckValidator.java
- /downloaded_code/src/main/java/com/example/exception/ValidationException.java
- /downloaded_code/src/main/java/com/example/exception/ConfigException.java

**Instructions:**
1. **Refactor Date/Time APIs:**
   - In `/downloaded_code/src/main/java/com/example/util/DateUtils.java`, replace:
     ```java
     public static Date getCurrentDate() { return new Date(); }
     ```
     with:
     ```java
     public static LocalDateTime getCurrentDateTime() { return LocalDateTime.now(); }
     ```
     - Add import: `import java.time.LocalDateTime;`
   - Update all usages of `Date` to `LocalDateTime`.
2. **Update File I/O to NIO:**
   - In `/downloaded_code/src/main/java/com/example/helper/FileHelper.java`, replace:
     ```java
     try (FileInputStream fis = new FileInputStream(path)) {
         return new String(fis.readAllBytes(), "UTF-8");
     }
     ```
     with:
     ```java
     return Files.readString(Path.of(path), StandardCharsets.UTF_8);
     ```
     - Add import: `import java.nio.file.Files;`
     - Add import: `import java.nio.file.Path;`
     - Add import: `import java.nio.charset.StandardCharsets;`
3. **Refactor Custom Exceptions:**
   - In `/downloaded_code/src/main/java/com/example/exception/ValidationException.java`, replace:
     ```java
     public class ValidationException extends Exception {
         public ValidationException(String message) { super(message); }
     }
     ```
     with:
     ```java
     public record ValidationException(String message) extends Exception {}
     ```
   - Consider sealed classes for exception hierarchies.
4. **Refactor Validators for Null-Safety:**
   - In `/downloaded_code/src/main/java/com/example/validator/EmailValidator.java`, replace manual null checks with `Optional`:
     ```java
     public boolean isValid(String email) {
         return Optional.ofNullable(email).filter(e -> e.matches(REGEX)).isPresent();
     }
     ```
     - Add import: `import java.util.Optional;`
5. **Update Try-With-Resources and Multi-Catch:**
   - Refactor all try-with-resources blocks to use enhanced syntax.
   - Update exception handling to use multi-catch where applicable.
6. **Validation:**
   - Run all utility and validator unit tests.
   - Verify correct behavior and compatibility.

**Dependencies:** Task 5

---

### Task 7: Migrate Test Cases to JUnit 5 and Spring Boot Test

**Source Files:**
- /downloaded_code/src/test/java/com/example/UserServiceTest.java
- /downloaded_code/src/test/java/com/example/OrderControllerTest.java
- /downloaded_code/src/test/java/com/example/OrderServiceTest.java
- /downloaded_code/src/test/java/com/example/ProductRepositoryTest.java
- /downloaded_code/src/test/resources/test-context.xml
- /downloaded_code/pom.xml
- /downloaded_code/build.gradle

**Target Files:**
- /downloaded_code/src/test/java/com/example/UserServiceTest.java
- /downloaded_code/src/test/java/com/example/OrderControllerTest.java
- /downloaded_code/src/test/java/com/example/OrderServiceTest.java
- /downloaded_code/src/test/java/com/example/ProductRepositoryTest.java
- /downloaded_code/pom.xml
- /downloaded_code/build.gradle

**Instructions:**
1. **Update Build Files for JUnit 5:**
   - In `/downloaded_code/pom.xml`, add:
     ```xml
     <dependency>
       <groupId>org.junit.jupiter</groupId>
       <artifactId>junit-jupiter</artifactId>
       <version>5.10.0</version>
       <scope>test</scope>
     </dependency>
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-test</artifactId>
       <version>3.2.5</version>
       <scope>test</scope>
     </dependency>
     <dependency>
       <groupId>org.mockito</groupId>
       <artifactId>mockito-core</artifactId>
       <version>5.2.0</version>
       <scope>test</scope>
     </dependency>
     ```
   - In `/downloaded_code/build.gradle`, add:
     ```groovy
     testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
     testImplementation 'org.springframework.boot:spring-boot-starter-test:3.2.5'
     testImplementation 'org.mockito:mockito-core:5.2.0'
     ```
   - Remove PowerMock dependencies.
2. **Refactor Test Classes to JUnit 5:**
   - In each test class (e.g., `/downloaded_code/src/test/java/com/example/UserServiceTest.java`), replace:
     ```java
     @RunWith(SpringJUnit4ClassRunner.class)
     @ContextConfiguration(locations = {"classpath:test-context.xml"})
     public class UserServiceTest {
         @Before
         public void setUp() { ... }
         @Test
         public void testFindUser() { ... }
     }
     ```
     with:
     ```java
     @SpringBootTest
     @AutoConfigureMockMvc
     class UserServiceTest {
         @BeforeEach
         void setUp() { ... }
         @Test
         void testFindUser() { ... }
     }
     ```
     - Add import: `import org.springframework.boot.test.context.SpringBootTest;`
     - Add import: `import org.springframework.test.web.servlet.MockMvc;`
     - Add import: `import org.junit.jupiter.api.Test;`
     - Add import: `import org.junit.jupiter.api.BeforeEach;`
   - Remove all usages of `@RunWith`, `@ContextConfiguration`, and XML context files.
3. **Refactor Mocking Logic:**
   - Replace PowerMock usages with Mockito:
     - Use `@Mock`, `@InjectMocks`, and Mockito methods.
     - Add import: `import org.mockito.Mock;`
     - Add import: `import org.mockito.InjectMocks;`
     - Add import: `import org.mockito.Mockito;`
4. **Update Lifecycle Annotations:**
   - Replace `@Before` with `@BeforeEach`, `@After` with `@AfterEach`.
5. **Remove Unused XML Configs:**
   - Delete `/downloaded_code/src/test/resources/test-context.xml`.
6. **Validation:**
   - Run `mvn test` or `gradle test`.
   - Ensure all tests pass and coverage is maintained.

**Dependencies:** Task 6

---

## Library and API Mappings

| Old Library/API                        | New Library/API                              | Notes                                                      |
|----------------------------------------|----------------------------------------------|------------------------------------------------------------|
| javax.servlet.*                        | jakarta.servlet.*                            | Update all servlet imports and dependencies                |
| org.springframework.context.ApplicationContext | org.springframework.boot.SpringApplication | Use Spring Boot for application startup                    |
| org.springframework.context.support.ClassPathXmlApplicationContext | @SpringBootApplication + component scanning | Remove XML context, use annotation-based config            |
| java.util.Date                         | java.time.LocalDateTime                      | Update all date fields, methods, and mappings              |
| java.util.Calendar                     | java.time.LocalDate/LocalDateTime            | Use modern Java time API                                   |
| java.text.SimpleDateFormat             | java.time.format.DateTimeFormatter           | Update date formatting logic                               |
| org.hibernate.SessionFactory           | org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean | Use Spring-managed JPA                                     |
| org.springframework.beans.factory.config.PropertyPlaceholderConfigurer | @Value/@ConfigurationProperties             | Use Spring Boot property injection                         |
| org.apache.commons.dbcp.BasicDataSource| org.apache.commons.dbcp2.BasicDataSource     | Use updated DBCP version                                   |
| JUnit 4 (`org.junit.*`)                | JUnit 5 (`org.junit.jupiter.*`)              | Update all test annotations and imports                    |
| PowerMock                              | Mockito 5.x                                  | Refactor mocking logic, remove PowerMock                   |
| Manual transaction management          | @Transactional                               | Use annotation-based transaction management                |
| Struts 1.x/2.x                         | Spring MVC or Jakarta EE                     | Migrate actions to controllers if feasible                 |
| Manual file I/O (`FileInputStream`)    | java.nio.file.Files, java.nio.file.Path       | Use NIO for file operations                                |
| Manual null checks                     | java.util.Optional, Objects.requireNonNull    | Use modern null-safety patterns                            |
| XML-based Spring config                | Java-based @Configuration classes            | Migrate all XML configs to Java                            |
| Manual bean definitions                | @Component, @Service, @Repository annotations| Use annotation-based bean registration                     |

---

## Migration Sequence

1. **Foundation and Setup**
   - Task 0: Update build system, bootstrap, dependencies, and configuration files.
2. **Data Model Updates**
   - Task 1: Refactor model/entity classes, update date/time APIs, convert DTOs to records.
3. **Data Access Layer Changes**
   - Task 2: Migrate DAOs to Spring Data JPA repositories, update mappings and configuration.
4. **Business Logic Updates**
   - Task 3: Refactor service layer to use dependency injection, annotation-based transactions.
5. **Presentation Layer Changes**
   - Task 4: Update web/controller layer to Jakarta EE, refactor servlets, JSPs, and Struts actions.
6. **Configuration and Deployment**
   - Task 5: Migrate all configuration to Java-based and YAML, consolidate properties, remove XML.
7. **Utility and Support Files**
   - Task 6: Refactor utility classes to use Java 21 APIs, update exception and validation logic.
8. **Testing and Validation**
   - Task 7: Migrate all test cases to JUnit 5 and Spring Boot Test, update build/test plugins, remove deprecated frameworks.

---

**End of Migration Plan**