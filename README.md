# Employee Management System — Spring Boot REST API with React Frontend

A full-stack CRUD application: a Spring Boot REST API backed by MySQL, and a separate React single-page frontend that consumes it. Create, list, view, update, and delete employee records.

## Why this exists

The smallest honest example of a **decoupled** full-stack application — not a server-rendered monolith, but a REST backend and a browser client that are developed, built, and deployed independently and only agree on a JSON contract.

That split is where most of the interesting decisions live: DTOs so the API contract is not your database schema, a mapper between entity and DTO, a dedicated exception type that turns a missing record into a proper 404, and CORS configuration so the frontend on one port can call the backend on another. All of those are present here, once each, in about two hundred lines of Java.

## Architecture

```
React SPA (Vite, port 5173)
        |
        |  axios --> http://localhost:8080/api/employees
        v
EmployeeController          @RestController, CORS enabled
        |
        v
EmployeeService / Impl      business logic
        |                   throws ResourceNotFoundException -> 404
        v
EmployeeRepository          Spring Data JPA
        |
        v
Employee entity  <-- EmployeeMapper --> EmployeeDto
        |
        v
MySQL (schema: ems)
```

## API

Base path `/api/employees`:

| Method | Path | Purpose | Response |
| --- | --- | --- | --- |
| `POST` | `/` | Create an employee | `201 Created` with the saved record |
| `GET` | `/{id}` | Fetch one employee | `200`, or `404` if not found |
| `GET` | `/` | List all employees | `200` |
| `PUT` | `/{id}` | Update an employee | `200`, or `404` if not found |
| `DELETE` | `/{id}` | Delete an employee | `200` |

## Tech stack

**Backend** — Java, Spring Boot, Spring Data JPA, Lombok, MySQL, Maven
**Frontend** — React 18, Vite, React Router, axios, Bootstrap 5

## Setup

### 1. Database

```sql
CREATE DATABASE ems;
```

### 2. Backend

Configuration lives in `ems-backend/src/main/resources/application.properties` and reads from environment variables:

```properties
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/ems
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

Set them before starting:

```bash
export DB_USERNAME=root
export DB_PASSWORD=your-password       # PowerShell: $env:DB_PASSWORD="your-password"

cd ems-backend
./mvnw spring-boot:run
```

The API comes up on <http://localhost:8080>. `spring.jpa.hibernate.ddl-auto=update` creates the `employees` table on first start.

### 3. Frontend

```bash
cd ems-frontend
npm install
npm run dev
```

Open <http://localhost:5173>. The backend allows all origins (`@CrossOrigin("*")`), so no proxy configuration is needed in development.

## Build for production

```bash
cd ems-backend  && ./mvnw clean package      # -> target/*.jar
cd ems-frontend && npm run build             # -> dist/
```

Serve `dist/` from any static host and point it at wherever the API is deployed.

## Notes and limits

- **Credentials come from environment variables.** An earlier revision of this repository had the database password committed in `application.properties`. It has been parameterised — if you are reading this from a clone of that history, that password should be treated as compromised and rotated.
- **`@CrossOrigin("*")` allows every origin.** Convenient in development, wrong in production. Restrict it to your actual frontend origin before deploying.
- **No authentication.** Every endpoint is open to anyone who can reach it.
- **No validation on the DTO.** `EmployeeDto` accepts whatever is posted. Add `@Valid` and Bean Validation annotations before trusting client input.
- **`ddl-auto=update` is a development convenience** — it never drops or renames anything, so schema changes accumulate silently. Move to Flyway or Liquibase for anything with real data.
- Build output (`ems-frontend/dist/`, `ems-backend/target/`) and `.idea/` are committed. They belong in `.gitignore`.
