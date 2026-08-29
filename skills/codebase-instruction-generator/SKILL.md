---
name: 'codebase-instruction-generator'
description: 'Skill for discovering, analyzing, and generating comprehensive modular architecture blueprints and AI instructions for Spring Boot, Angular, and Fullstack codebases.'
---

# Codebase Instruction Generator Skill

Use this skill when asked to:
- *"Generate instructions for this project"*
- *"Document the codebase architecture"*
- *"Create a blueprint for new team members or AI agents"*
- *"Extract rules, security configs, and API contracts from Spring Boot / Angular"*

---

## 🛠️ Step-by-Step Skill Workflow

### Step 1: Detect Project Boundaries
Inspect the current workspace directory for:
- Spring Boot backend indicators: `pom.xml`, `build.gradle`, `src/main/java`, `application.yml`, `application.properties`.
- Angular frontend indicators: `angular.json`, `package.json`, `src/app`.
- If split across separate directories, ask user for the exact paths.

### Step 2: Determine Deployment Topology
Clarify or detect:
- **Bundled Static Hosting**: Angular `dist/` is copied into Spring Boot's `src/main/resources/static/`.
- **Decoupled Hosting**: Angular and Spring Boot are deployed independently.

### Step 3: Deep Extraction
Extract:
1. **Backend**: Controllers, Services, Repositories, Entities, DTOs, `SecurityFilterChain`, JWT logic, CSP headers, Swagger/OpenAPI config, Actuator/Prometheus metrics, `@RestControllerAdvice` error envelopes.
2. **Frontend**: Standalone vs NgModule components, Signals vs RxJS, Functional Guards, HTTP Interceptors, Environment variables, SCSS/Tailwind styling.
3. **Fullstack Glue**: API endpoints, CORS policies, JWT attachment, token refresh lifecycle.

### Step 4: Generate Modular Document Structure
Write the following files into the workspace:
1. `PROJECT_BLUEPRINT.md` (Master index and executive summary)
2. `docs/architecture/01-system-overview.md`
3. `docs/architecture/02-backend-spring-boot.md`
4. `docs/architecture/03-frontend-angular.md`
5. `docs/architecture/04-security-and-csp.md`
6. `docs/architecture/05-api-contracts-swagger.md`
7. `docs/architecture/06-observability-metrics.md`
8. `docs/architecture/07-build-and-deployment.md`
9. `docs/architecture/08-ai-replication-recipe.md`
