# Task Manager App — Especificación Técnica

> **Versión:** 1.0 · **Fecha:** 2026-11-01
> **Stack:** Java 21 + Spring Boot 3.x (Hexagonal Architecture) · Angular 18+ (Standalone Components)

---

## 1. Visión General

Aplicación web de gestión de tareas (To-Do / Task Manager) con backend REST y frontend SPA. El backend aplica **Arquitectura Hexagonal estricta** (también llamada Ports & Adapters). El frontend consume la API usando el nuevo modelo de Angular standalone con `HttpClient` funcional.

---

## 2. Funcionalidades Requeridas

| # | Feature | Descripción |
|---|---------|-------------|
| F-01 | Gestión de tareas | CRUD completo: crear, listar, obtener por ID, actualizar, eliminar |
| F-02 | Estado de tarea | `TODO`, `IN_PROGRESS`, `DONE` |
| F-03 | Prioridad | `LOW`, `MEDIUM`, `HIGH` |
| F-04 | Fechas | Fecha de creación (automática) y fecha límite (`dueDate`) opcional |
| F-05 | Filtrado | Filtrar por estado y/o prioridad |
| F-06 | Paginación | Listado paginado (page, size, sort) |
| F-07 | Validación | Bean Validation en capa de entrada; errores claros en JSON |
| F-08 | CORS | Configuración para permitir el frontend Angular (`localhost:4200`) |

---

## 3. Arquitectura Backend — Hexagonal (Ports & Adapters)

### 3.1 Principio

```
                         ┌────────────────────────────────┐
  Driving Adapters       │           APPLICATION           │       Driven Adapters
  (Primary)              │                                 │       (Secondary)
  ──────────────────►    │  ┌──────────┐  ┌────────────┐  │  ◄────────────────────
  REST Controller        │  │  Ports   │  │  Use Cases │  │       JPA Repository
  (Spring MVC)           │  │(Interfaces│  │ (Services) │  │       (Spring Data)
                         │  └──────────┘  └────────────┘  │
                         │         DOMAIN (core)           │
                         │   Entities · Value Objects      │
                         │   Domain Events · Exceptions    │
                         └────────────────────────────────┘
```

### 3.2 Estructura de Paquetes

```
backend/
└── src/main/java/com/taskmanager/
    ├── domain/
    │   ├── model/
    │   │   ├── Task.java                  ← Entidad de dominio (sin anotaciones JPA)
    │   │   ├── TaskId.java                ← Value Object (UUID)
    │   │   ├── TaskStatus.java            ← Enum: TODO | IN_PROGRESS | DONE
    │   │   └── TaskPriority.java          ← Enum: LOW | MEDIUM | HIGH
    │   ├── port/
    │   │   ├── in/                        ← Driving ports (use-case interfaces)
    │   │   │   ├── CreateTaskUseCase.java
    │   │   │   ├── GetTaskUseCase.java
    │   │   │   ├── UpdateTaskUseCase.java
    │   │   │   ├── DeleteTaskUseCase.java
    │   │   │   └── ListTasksUseCase.java
    │   │   └── out/                       ← Driven ports (persistence interfaces)
    │   │       └── TaskRepository.java
    │   └── exception/
    │       └── TaskNotFoundException.java
    │
    ├── application/
    │   └── service/
    │       └── TaskService.java           ← Implementa todos los Use Cases
    │
    └── infrastructure/
        ├── adapter/
        │   ├── in/
        │   │   └── rest/
        │   │       ├── TaskController.java
        │   │       ├── dto/
        │   │       │   ├── CreateTaskRequest.java
        │   │       │   ├── UpdateTaskRequest.java
        │   │       │   └── TaskResponse.java
        │   │       └── mapper/
        │   │           └── TaskRestMapper.java
        │   └── out/
        │       └── persistence/
        │           ├── TaskJpaRepository.java      ← Spring Data JPA interface
        │           ├── TaskPersistenceAdapter.java ← Implementa TaskRepository port
        │           ├── entity/
        │           │   └── TaskJpaEntity.java      ← @Entity JPA
        │           └── mapper/
        │               └── TaskPersistenceMapper.java
        └── config/
            ├── WebConfig.java             ← CORS
            ├── OpenApiConfig.java         ← SpringDoc / Swagger
            └── BeanConfig.java            ← @Bean wiring use-cases
```

### 3.3 Dependencias Maven (pom.xml)

```xml
<!-- Java 21 + Spring Boot 3.3.x -->
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>3.3.5</version>
</parent>

<dependencies>
  <dependency>spring-boot-starter-web</dependency>
  <dependency>spring-boot-starter-data-jpa</dependency>
  <dependency>spring-boot-starter-validation</dependency>
  <dependency>com.h2database:h2</dependency>          <!-- dev; swap con PostgreSQL en prod -->
  <dependency>org.springdoc:springdoc-openapi-starter-webmvc-ui:2.5.0</dependency>
  <dependency>org.mapstruct:mapstruct:1.5.5.Final</dependency>
  <dependency>org.projectlombok:lombok</dependency>
</dependencies>
```

---

## 4. Modelo de Dominio

### Task (entidad de dominio)

```java
// domain/model/Task.java — sin anotaciones de infraestructura
public class Task {
    private TaskId id;
    private String title;           // requerido, 1-200 chars
    private String description;     // opcional
    private TaskStatus status;      // default: TODO
    private TaskPriority priority;  // default: MEDIUM
    private LocalDateTime createdAt;
    private LocalDate dueDate;      // nullable
}
```

### TaskId (value object)

```java
public record TaskId(UUID value) {
    public static TaskId generate() { return new TaskId(UUID.randomUUID()); }
    public static TaskId of(String uuid) { return new TaskId(UUID.fromString(uuid)); }
}
```

---

## 5. Puertos (Interfaces de Dominio)

### Driving Ports (in)

```java
// Cada use-case tiene su propio puerto — Interface Segregation
public interface CreateTaskUseCase {
    Task createTask(CreateTaskCommand command);
}
public interface ListTasksUseCase {
    Page<Task> listTasks(TaskFilter filter, Pageable pageable);
}
public interface GetTaskUseCase {
    Task getTaskById(TaskId id);
}
public interface UpdateTaskUseCase {
    Task updateTask(TaskId id, UpdateTaskCommand command);
}
public interface DeleteTaskUseCase {
    void deleteTask(TaskId id);
}
```

### Driven Port (out)

```java
public interface TaskRepository {
    Task save(Task task);
    Optional<Task> findById(TaskId id);
    Page<Task> findAll(TaskFilter filter, Pageable pageable);
    void deleteById(TaskId id);
    boolean existsById(TaskId id);
}
```

---

## 6. API REST — Contratos

Base URL: `http://localhost:8080/api/v1`

| Método | Ruta | Descripción |
|--------|------|-------------|
| `POST` | `/tasks` | Crear tarea |
| `GET` | `/tasks` | Listar tareas (paginado + filtros) |
| `GET` | `/tasks/{id}` | Obtener tarea por ID |
| `PUT` | `/tasks/{id}` | Actualizar tarea completa |
| `PATCH` | `/tasks/{id}/status` | Cambiar solo el estado |
| `DELETE` | `/tasks/{id}` | Eliminar tarea |

### Query params para GET /tasks

| Param | Tipo | Ejemplo |
|-------|------|---------|
| `status` | `TaskStatus` | `?status=IN_PROGRESS` |
| `priority` | `TaskPriority` | `?priority=HIGH` |
| `page` | int | `?page=0` |
| `size` | int | `?size=10` |
| `sort` | string | `?sort=createdAt,desc` |

### Payload de ejemplo

**POST /tasks — Request**
```json
{
  "title": "Implementar login",
  "description": "Usar JWT con refresh token",
  "priority": "HIGH",
  "dueDate": "2026-11-15"
}
```

**Response 201 Created**
```json
{
  "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "title": "Implementar login",
  "description": "Usar JWT con refresh token",
  "status": "TODO",
  "priority": "HIGH",
  "createdAt": "2026-11-01T10:30:00",
  "dueDate": "2026-11-15"
}
```

### Manejo de Errores

```json
{
  "timestamp": "2026-11-01T10:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Task with id 'xyz' not found",
  "path": "/api/v1/tasks/xyz"
}
```

---

## 7. Arquitectura Frontend — Angular Standalone

### 7.1 Estructura de Carpetas

```
frontend/
└── src/
    ├── app/
    │   ├── core/
    │   │   ├── models/
    │   │   │   └── task.model.ts          ← interfaces Task, CreateTaskDto, etc.
    │   │   ├── services/
    │   │   │   └── task.service.ts        ← HttpClient calls
    │   │   └── interceptors/
    │   │       └── error.interceptor.ts   ← manejo global de errores HTTP
    │   ├── features/
    │   │   └── tasks/
    │   │       ├── task-list/
    │   │       │   ├── task-list.component.ts   ← standalone
    │   │       │   └── task-list.component.html
    │   │       ├── task-form/
    │   │       │   ├── task-form.component.ts   ← standalone, reactive forms
    │   │       │   └── task-form.component.html
    │   │       └── task-card/
    │   │           ├── task-card.component.ts   ← standalone
    │   │           └── task-card.component.html
    │   ├── shared/
    │   │   └── components/
    │   │       ├── badge/badge.component.ts
    │   │       └── confirm-dialog/confirm-dialog.component.ts
    │   ├── app.routes.ts                  ← rutas lazy-loaded
    │   └── app.config.ts                  ← provideHttpClient, provideRouter
    └── environments/
        ├── environment.ts
        └── environment.prod.ts
```

### 7.2 Configuración (app.config.ts)

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([errorInterceptor])),
    provideAnimationsAsync(),
  ],
};
```

### 7.3 Task Service

```typescript
@Injectable({ providedIn: 'root' })
export class TaskService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = `${environment.apiUrl}/tasks`;

  getAll(params?: TaskQueryParams): Observable<PagedResult<Task>> { ... }
  getById(id: string): Observable<Task> { ... }
  create(dto: CreateTaskDto): Observable<Task> { ... }
  update(id: string, dto: UpdateTaskDto): Observable<Task> { ... }
  updateStatus(id: string, status: TaskStatus): Observable<Task> { ... }
  delete(id: string): Observable<void> { ... }
}
```

### 7.4 Modelos TypeScript

```typescript
export type TaskStatus = 'TODO' | 'IN_PROGRESS' | 'DONE';
export type TaskPriority = 'LOW' | 'MEDIUM' | 'HIGH';

export interface Task {
  id: string;
  title: string;
  description?: string;
  status: TaskStatus;
  priority: TaskPriority;
  createdAt: string;
  dueDate?: string;
}

export interface CreateTaskDto {
  title: string;
  description?: string;
  priority: TaskPriority;
  dueDate?: string;
}

export interface PagedResult<T> {
  content: T[];
  totalElements: number;
  totalPages: number;
  number: number;
  size: number;
}
```

---

## 8. Reglas de Arquitectura Hexagonal (Non-Negotiable)

1. **El dominio no depende de nada externo.** `domain/` no tiene `import` de Spring, JPA, ni Jackson.
2. **Los puertos son interfaces del dominio**, no de infraestructura.
3. **La capa `application/` solo conoce el dominio.** No accede a JPA ni a HTTP directamente.
4. **Los adaptadores implementan los puertos**, no los exponen directamente.
5. **El wiring se hace en `infrastructure/config/`** mediante `@Bean` o `@Component` — nunca en el dominio.
6. **Los mappers son responsabilidad exclusiva de los adaptadores**. El dominio nunca recibe un DTO ni una entidad JPA.

---

## 9. Configuración de Entorno

| Variable | Dev | Prod |
|----------|-----|------|
| DB | H2 in-memory | PostgreSQL 16 |
| Puerto backend | 8080 | configurable |
| Puerto frontend | 4200 | Nginx / CDN |
| API URL (Angular env) | `http://localhost:8080/api/v1` | dominio real |

### application.yml (dev)

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:taskdb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
  h2:
    console:
      enabled: true

springdoc:
  swagger-ui:
    path: /swagger-ui.html
```
