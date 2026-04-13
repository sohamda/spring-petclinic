# Spring PetClinic – Copilot Instructions

## Project Overview
Spring Boot 4.0 web app (Java 17) using Spring MVC, Spring Data JPA, Thymeleaf, and Bean Validation. Supports H2 (default), MySQL, and PostgreSQL via Spring profiles.

## Architecture
- **No service layer** — controllers inject repositories directly.
- Domain is organized by feature: `owner/`, `vet/`, `system/`, `model/`.
- Each feature package contains its entity, repository, controller, and tests.
- Use package-private visibility for controllers and repositories (no `public`).

## Entities & Data
- Extend `BaseEntity` (auto-generated `Integer` id) or `NamedEntity` / `Person`.
- Use `@Table(name = "snake_case")` — Hibernate `PhysicalNamingStrategySnakeCaseImpl` is active.
- DDL is managed by SQL scripts in `src/main/resources/db/{h2,mysql,postgres}/`, not Hibernate auto-DDL.
- Validate with Jakarta Bean Validation annotations (`@NotBlank`, `@Pattern`, etc.) or custom `Validator` implementations registered via `@InitBinder`.

## Repositories
- Extend `JpaRepository<Entity, Integer>`.
- Derive queries from method names (no `@Query` unless necessary).

## Controllers
- Use `@Controller` with `@GetMapping` / `@PostMapping`.
- Bind forms with `@Valid` + `BindingResult`; use `@InitBinder` with `setAllowedFields` to prevent mass-assignment.
- Return Thymeleaf view names (e.g., `"owners/ownerDetails"`).
- For JSON endpoints, add `@ResponseBody` to the handler method.

## Thymeleaf Templates
- Fragment-based layout defined in `templates/fragments/layout.html`.
- Use `th:text="#{key}"` for i18n (messages in `src/main/resources/messages/`).
- Forms use `th:field="*{property}"` binding.

## Testing
- **Unit tests**: `@WebMvcTest` + `@MockitoBean` for controller slices. BDD-style mocking (`given().willReturn()`), Hamcrest matchers.
- **Integration tests**: `@SpringBootTest(webEnvironment = RANDOM_PORT)` with real database. Use `RestTemplate` for HTTP assertions.
- Add `@DisabledInNativeImage` and `@DisabledInAotMode` to tests that don't support GraalVM native.

## Build & Formatting
- **Maven**: `./mvnw spring-boot:run` &nbsp;|&nbsp; **Gradle**: `./gradlew bootRun`
- Spring Java Format plugin enforces code style — run `./mvnw spring-javaformat:apply` or `./gradlew format` before committing.
- NoHttp checkstyle rule: never use `http://` URLs in source; use `https://`.

## Database Profiles
- Default: H2 in-memory. Switch with `spring.profiles.active=mysql` or `postgres`.
- MySQL/Postgres credentials come from environment variables (`MYSQL_URL`, `POSTGRES_USER`, etc.).
- Docker Compose support via `docker-compose.yml` for local database containers.
