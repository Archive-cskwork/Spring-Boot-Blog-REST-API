# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spring Boot REST API for a blog platform with JWT authentication. Stack: Spring Boot 2.1.8, MySQL, Spring Security, JPA/Hibernate, Lombok.

## Development Commands

### Build and Run

```bash
# Build the project
mvn clean install

# Run the application
mvn spring-boot:run

# Run tests
mvn test

# Package the application
mvn package
```

### Docker

```bash
# Start services (MySQL + Application)
docker-compose up -d

# Stop services
docker-compose down

# Rebuild application container
docker-compose up -d --build application
```

### Database Setup

1. Create MySQL database: `CREATE DATABASE blogapi`
2. Run initialization script: `src/main/resources/blogapi.sql`
3. Configure credentials in `src/main/resources/application.yml`:
   - `spring.datasource.username`
   - `spring.datasource.password`

Application runs on `http://localhost:8080`

## Architecture

### Layered Architecture

The codebase follows a standard Spring Boot layered architecture:

**Controller → Service → Repository → Database**

- **Controllers** (`controller/`) - REST endpoints, request validation, response mapping
- **Services** (`service/` and `service/impl/`) - Business logic, transaction management
- **Repositories** (`repository/`) - JPA repositories extending `JpaRepository`
- **Models** (`model/`) - JPA entities with relationships

### Security Architecture

- **JWT-based authentication** - Stateless session management
- **Spring Security** - Configured in `config/SecutiryConfig.java`
- **Authentication flow**:
  1. User signs in via `/api/auth/signin`
  2. `JwtTokenProvider` generates JWT token
  3. `JwtAuthenticationFilter` validates token on subsequent requests
  4. `UserPrincipal` represents authenticated user
- **Authorization**: Method-level security using `@PreAuthorize` annotations
- **Public endpoints**: GET requests and `/api/auth/**` are public; other endpoints require authentication

### Key Patterns

**DTO Pattern**: Request/Response objects in `payload/` package are separate from entities
- Use `ModelMapper` bean to convert between entities and DTOs
- Example: `Post` entity → `PostResponse` DTO

**Audit Trail**: Entities extend `UserDateAudit` or `DateAudit` (`model/audit/`)
- Automatically tracks `createdAt`, `updatedAt`, `createdBy`, `updatedBy`
- Configured via `config/AuditingConfig.java`

**Exception Handling**: Centralized in `exception/RestControllerExceptionHandler.java`
- Custom exceptions: `ResourceNotFoundException`, `BadRequestException`, `UnauthorizedException`, etc.
- Returns standardized `ExceptionResponse` payload

**Pagination**: Uses `PagedResponse` wrapper for list endpoints
- Configurable via `AppConstants.DEFAULT_PAGE_NUMBER`, `DEFAULT_PAGE_SIZE`

### Entity Relationships

- **User** (1) → (N) **Post**, **Album**, **Todo**, **Comment**
- **Post** (N) ↔ (N) **Tag** (many-to-many via `post_tag` join table)
- **Post** (N) → (1) **Category**
- **Post** (1) → (N) **Comment**
- **Album** (1) → (N) **Photo**
- **User** (N) ↔ (N) **Role** (many-to-many)
- **User** (1) → (1) **Address**, **Company** (embedded relationships)

**Important**: Most relationships use `FetchType.LAZY` to avoid N+1 queries. Be mindful when adding new entity methods.

### Technology Notes

**Lombok**: Extensively used for boilerplate reduction
- `@Data`, `@EqualsAndHashCode`, `@NoArgsConstructor`, etc.
- Models use Lombok; manually override getters/setters when needed (e.g., for defensive copies)

**Jackson**: Configured for Java 8 date/time serialization
- `@JsonIdentityInfo` on entities to handle circular references
- `@JsonIgnore` on lazy-loaded relationships

**ModelMapper**: Bean configured in `BlogApiApplication.java`
- Used throughout service layer for entity-DTO conversions

## API Structure

All APIs under `/api/**`:
- `/api/auth/**` - Authentication (signup, signin)
- `/api/users/**` - User management
- `/api/posts/**` - Posts and comments
- `/api/albums/**` - Albums and photos
- `/api/todos/**` - Todo items
- `/api/categories/**` - Post categories
- `/api/tags/**` - Post tags

Refer to README.md for complete API documentation with request/response examples.

## Configuration

- **application.yml**: Main configuration (database, JWT settings, CORS)
- **JWT settings**: `app.jwtSecret`, `app.jwtExpirationInMs`
- **CORS**: Configured via `cors.allowedOrings` (note: typo in property name)
- **Hibernate**: `ddl-auto: none` - schema managed via SQL scripts, not auto-generated

## Common Tasks

When adding a new entity:
1. Create entity class in `model/` (extend `UserDateAudit` if tracking user/date)
2. Create repository interface in `repository/`
3. Create request/response DTOs in `payload/`
4. Create service interface and implementation in `service/`
5. Create controller in `controller/`
6. Add custom exceptions if needed in `exception/`

When modifying security:
- Update `SecutiryConfig.java` for endpoint permissions
- JWT configuration in `security/JwtTokenProvider.java`
- Current user retrieval via `@CurrentUser UserPrincipal` annotation
