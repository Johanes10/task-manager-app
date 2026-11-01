# Task Manager App — Plan de Implementación

> Referencia: [spec.md](./spec.md)
> **Estado:** 🔴 Pendiente

---

## Fase 1 — Backend: Dominio + Aplicación

> Objetivo: construir el núcleo de la app sin dependencias externas. Testeable con JUnit puro.

- [x] **T-01** Crear proyecto Spring Boot con Maven (Java 21, `spring-boot-starter-parent` 3.3.5)
- [ ] **T-02** Crear enum `TaskStatus` (`TODO`, `IN_PROGRESS`, `DONE`)
- [ ] **T-03** Crear enum `TaskPriority` (`LOW`, `MEDIUM`, `HIGH`)
- [ ] **T-04** Crear value object `TaskId` (record con `UUID`)
- [ ] **T-05** Crear entidad de dominio `Task` (sin anotaciones JPA/Spring)
- [ ] **T-06** Definir puerto de entrada `CreateTaskUseCase`
- [ ] **T-07** Definir puerto de entrada `GetTaskUseCase`
- [ ] **T-08** Definir puerto de entrada `UpdateTaskUseCase`
- [ ] **T-09** Definir puerto de entrada `DeleteTaskUseCase`
- [ ] **T-10** Definir puerto de entrada `ListTasksUseCase`
- [ ] **T-11** Definir puerto de salida `TaskRepository`
- [ ] **T-12** Crear `TaskNotFoundException` (dominio)
- [ ] **T-13** Crear command objects: `CreateTaskCommand`, `UpdateTaskCommand`, `TaskFilter`
- [ ] **T-14** Implementar `TaskService` (implementa todos los puertos `in/`)

---

## Fase 2 — Backend: Infraestructura

> Objetivo: conectar el dominio con JPA, HTTP y configuración de Spring.

- [ ] **T-15** Crear `TaskJpaEntity` con anotaciones `@Entity`, `@Table`, etc.
- [ ] **T-16** Crear `TaskJpaRepository` (interfaz `JpaRepository` + custom queries para filtros)
- [ ] **T-17** Crear `TaskPersistenceMapper` con MapStruct (`Task` ↔ `TaskJpaEntity`)
- [ ] **T-18** Implementar `TaskPersistenceAdapter` (implementa puerto `out/TaskRepository`)
- [ ] **T-19** Crear DTO `CreateTaskRequest` con Bean Validation (`@NotBlank`, `@Size`, etc.)
- [ ] **T-20** Crear DTO `UpdateTaskRequest`
- [ ] **T-21** Crear DTO `TaskResponse`
- [ ] **T-22** Crear `TaskRestMapper` con MapStruct (`Task` ↔ DTOs REST)
- [ ] **T-23** Implementar `TaskController` (`@RestController`, rutas `/api/v1/tasks`)
  - `POST /tasks`
  - `GET /tasks` (paginado + filtros `status`, `priority`)
  - `GET /tasks/{id}`
  - `PUT /tasks/{id}`
  - `PATCH /tasks/{id}/status`
  - `DELETE /tasks/{id}`
- [ ] **T-24** Implementar `GlobalExceptionHandler` (`@RestControllerAdvice`)
  - Manejar `TaskNotFoundException` → 404
  - Manejar `MethodArgumentNotValidException` → 400
  - Manejar excepciones genéricas → 500
- [ ] **T-25** Configurar CORS en `WebConfig.java` (permitir `http://localhost:4200`)
- [ ] **T-26** Configurar `OpenApiConfig.java` (SpringDoc, info del API)
- [ ] **T-27** Crear `BeanConfig.java` para wiring de ports → adapters
- [ ] **T-28** Configurar `application.yml` para perfil dev (H2, JPA DDL, H2 console)
- [ ] **T-29** Verificar arranque de la aplicación y Swagger en `http://localhost:8080/swagger-ui.html`

---

## Fase 3 — Frontend: Angular Standalone

> Objetivo: SPA funcional que consume todos los endpoints del backend.

- [ ] **T-30** Crear proyecto Angular 18 standalone (`ng new task-manager-frontend --standalone`)
- [ ] **T-31** Configurar `app.config.ts` (`provideRouter`, `provideHttpClient(withInterceptors(...))`, `provideAnimationsAsync`)
- [ ] **T-32** Crear `environment.ts` y `environment.prod.ts` con `apiUrl`
- [ ] **T-33** Crear modelos TypeScript en `core/models/task.model.ts` (`Task`, `CreateTaskDto`, `UpdateTaskDto`, `PagedResult`, `TaskStatus`, `TaskPriority`)
- [ ] **T-34** Implementar `TaskService` en `core/services/` con todos los métodos HTTP
- [ ] **T-35** Implementar `errorInterceptor` funcional en `core/interceptors/`
- [ ] **T-36** Implementar `TaskCardComponent` (standalone) — muestra título, estado, prioridad, fecha límite
- [ ] **T-37** Implementar `TaskListComponent` (standalone) — lista paginada, filtros por estado y prioridad
- [ ] **T-38** Implementar `TaskFormComponent` (standalone, ReactiveFormsModule) — crear y editar tarea
- [ ] **T-39** Implementar `BadgeComponent` compartido para estado y prioridad
- [ ] **T-40** Implementar `ConfirmDialogComponent` para confirmar eliminación
- [ ] **T-41** Configurar rutas lazy-loaded en `app.routes.ts`
  - `/tasks` → `TaskListComponent`
  - `/tasks/new` → `TaskFormComponent`
  - `/tasks/:id/edit` → `TaskFormComponent`
- [ ] **T-42** Implementar navegación principal (header con link a lista de tareas)

---

## Fase 4 — Integración y Verificación

> Objetivo: asegurar que backend y frontend funcionan correctamente juntos.

- [ ] **T-43** Verificar comunicación CORS: Angular `localhost:4200` → Spring `localhost:8080`
- [ ] **T-44** Smoke test manual: crear tarea desde Angular, verificar persiste en H2
- [ ] **T-45** Smoke test manual: actualizar estado de una tarea
- [ ] **T-46** Smoke test manual: eliminar tarea con confirmación
- [ ] **T-47** Smoke test manual: filtrar tareas por estado y prioridad
- [ ] **T-48** Smoke test manual: paginación en lista de tareas
- [ ] **T-49** Escribir pruebas unitarias de dominio con JUnit 5 (sin Spring context)
- [ ] **T-50** Escribir prueba de integración con `@SpringBootTest` para `TaskController`

---

## Resumen de Progreso

| Fase | Tareas | Completadas | Estado |
|------|--------|-------------|--------|
| Fase 1 — Dominio | 14 | 1 | � En progreso |
| Fase 2 — Infraestructura | 15 | 0 | 🔴 Pendiente |
| Fase 3 — Frontend | 13 | 0 | 🔴 Pendiente |
| Fase 4 — Integración | 8 | 0 | 🔴 Pendiente |
| **Total** | **50** | **0** | **0%** |
