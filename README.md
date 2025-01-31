# Spring Boot Starter for OpenAPI Code Generation  

## Overview  

This Spring Boot Starter provides a standardized way to automatically generate OpenAPI specifications and WebClient-based API clients. It simplifies integration between microservices by enabling automatic client code generation during the build process.  

## Features  

- **Automatic OpenAPI Specification Generation**  
  - Extracts API definitions from Spring Boot controllers.  
  - Saves the generated OpenAPI spec (`api-docs.json`).  
- **Client Code Generation**  
  - Uses OpenAPI Generator to create WebClient-based API clients.  
  - Generates client classes matching service endpoints.  
- **Customizable Mustache Templates**  
  - Allows modification of generated clients.  
- **Maven Integration**  
  - Fully automated through Maven build lifecycle.  

---

## How It Works  

1. **Generates OpenAPI specification (`api-docs.json`)**  
   - Scans all `@RestController`-annotated classes.  
   - Extracts API endpoints, request/response models.  
   - Saves the specification in `src/main/resources/api-docs.json`.  

2. **Generates WebClient-based API clients**  
   - Fetches `api-docs.json` from the service module.  
   - Uses custom Mustache templates to generate client code.  
   - Places the generated code inside the `client` module.  

---

## Installation  

To use this starter in your project, add the following dependency to your `pom.xml`:  

```xml
<dependency>
    <groupId>ru.hawk</groupId>
    <artifactId>codegen-spring-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

---

## Maven Plugins Used  

### **1. `springdoc-openapi-maven-plugin`**  
- **Purpose:** Generates OpenAPI spec (`api-docs.json`) from Spring controllers.  
- **Runs during:** `compile` phase.  

### **2. `openapi-generator-maven-plugin`**  
- **Purpose:** Generates WebClient-based API clients from OpenAPI spec.  
- **Runs during:** `generate-sources` phase.  

### **3. `maven-antrun-plugin`**  
- **Purpose:** Cleans up unnecessary generated files.  
- **Runs during:** `process-sources` phase.  

### **4. `maven-resources-plugin`**  
- **Purpose:** Copies generated clients to the correct source package.  
- **Runs during:** `process-resources` phase.  

---

## Folder Structure  

```plaintext
codegen-spring-boot-starter/
│── src/
│   ├── main/
│   │   ├── resources/
│   │   │   ├── META-INF/spring.factories
│   │   │   ├── templates/
│── pom.xml
```

---

## Configuration  

To customize package names for generated clients, override the following properties in `application.yml`:  

```yaml
openapi.codegen:
  api-package: com.example.client.api
  model-package: com.example.client.model
```

---

## How to Use in a Service  

1. **Add this starter dependency** in your service’s `pom.xml`.  
2. **Expose REST controllers** as usual.  
3. **Run `mvn clean package`**, and the OpenAPI client will be generated automatically.  
4. **Use the generated WebClient clients** in your service or other applications.  

---

## Customizing Client Generation  

To modify the generated clients, edit the Mustache templates inside the `templates/` folder:  

- **`api.mustache`** → Defines the structure of generated WebClient clients.  

Place custom templates in your service module under:  
```plaintext
src/main/resources/templates/
```
This will override the default templates.  

---

## Example  

### **Service Controller (`RuleController.java`)**  

```java
@RestController
@RequestMapping("/rule-apply")
@RequiredArgsConstructor
public class RuleController {

    @GetMapping
    public String testGet() {
        return "test";
    }

    @PostMapping
    public String testPost(@RequestBody String body) {
        return body;
    }

}
```

### **Generated Client (`RuleClient.java`)**  

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class RuleClient {
    private final WebClient ruleApplyWebClient;

    public <T> Mono<T> doGetRequest(String path, Class<T> cls) {
        return ruleApplyWebClient.get()
                .uri(path)
                .retrieve()
                .bodyToMono(cls);
    }

    public Mono<String> getTest() {
        return doGetRequest("/rule-apply", String.class);
    }
}
```

