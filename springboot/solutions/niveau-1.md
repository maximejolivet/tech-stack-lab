# Spring Boot — Solutions niveau 1

## Exercice 1

```java
@RestController
@RequestMapping("/api")
public class GreetingController {

    @GetMapping("/hello")
    public Map<String, String> hello() {
        return Map.of("message", "Hello, World!");
    }

    @GetMapping("/hello/{name}")
    public Map<String, String> helloName(@PathVariable String name) {
        return Map.of("message", "Hello, " + name + "!");
    }
}
```

## Exercice 2

```java
@GetMapping("/greet")
public ResponseEntity<Map<String, String>> greet(
        @RequestParam String name,
        @RequestParam(defaultValue = "fr") String lang) {

    String greeting = switch (lang) {
        case "fr" -> "Bonjour";
        case "en" -> "Hello";
        default -> null;
    };

    if (greeting == null) {
        return ResponseEntity.badRequest().body(Map.of("error", "Unsupported lang: " + lang));
    }
    return ResponseEntity.ok(Map.of("message", greeting + ", " + name + "!"));
}
```

## Exercice 3

```properties
# application.properties
server.port=9090
```

```yaml
# application.yml
server:
  port: 9090
```

Différence : `.properties` est un format clé=valeur plat (une ligne par clé), `.yml` est hiérarchique et évite la répétition des préfixes communs (`server.*`) — plus lisible pour une configuration riche, mais sensible à l'indentation.

## Exercice 4

```java
public record EchoRequest(String text) {}

@PostMapping("/echo")
@ResponseStatus(HttpStatus.CREATED)
public EchoRequest echo(@RequestBody EchoRequest request) {
    return request;
}
```
