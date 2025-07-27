# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Provider Screening Module (PSM) is a Java EE Enterprise Application designed for U.S. state Medicare/Medicaid Information Systems (MMIS) to provide a portal for screening service providers as part of the provider enrollment process. This 7-year-old codebase represents a sophisticated enterprise system combining Java EE patterns, workflow engines, and business rules for healthcare provider credentialing.

## Build Commands

### Main Build Commands
- `./gradlew cms-portal-services:build` - Build the complete application (EAR file)
- `./gradlew build` - Build all modules
- `./gradlew clean` - Clean all build artifacts

### Testing Commands
- `./gradlew test` - Run unit tests for all modules
- `./gradlew integration-tests:test` - Run integration tests (requires WildFly setup)
- `./gradlew integration-tests:aggregate` - Generate Serenity test reports

### Code Quality Commands
- `./gradlew checkstyleMain` - Run Checkstyle on main source code
- `./gradlew checkstyleTest` - Run Checkstyle on test source code
- `./gradlew frontend:npm_run_lint` - Run JavaScript linting (JSCS)

### Database Commands
- `./gradlew db:update` - Run Liquibase database migrations
- Load jBPM tables: `cat psm-app/db/jbpm.sql | psql -h localhost -U psm psm`

### Documentation Commands
- `./gradlew cms-web:apiDocs` - Generate Javadoc API documentation
- `./gradlew userhelp:html` - Build user documentation in HTML format
- `./gradlew userhelp:pdf` - Build user documentation in PDF format

## Detailed Architecture Analysis

### Multi-Module Structure
The PSM follows a traditional Java EE enterprise architecture with clear separation of concerns:

**Core Business Modules:**
- `cms-business-model` - JAXB-generated data types from XML schemas, used by Drools rules
- `cms-business-process` - jBPM workflow definitions, process handlers, and business logic implementations
- `services` - Service interfaces, JPA entities, form binders, and business services (NO traditional DAO layer)
- `cms-web` - Spring MVC web controllers, security, API endpoints, JSP/Handlebars templates

**Supporting Modules:**
- `cms-portal-services` - EAR packaging with dependency aggregation
- `frontend` - JavaScript management with Node.js dependencies and jQuery-based UI
- `userhelp` - Sphinx-based user documentation system
- `integration-tests` - Serenity BDD integration tests with Cucumber
- `db` - Liquibase database migrations and jBPM schema

### Technology Stack and Patterns
- **Java EE 6/7**: Traditional EJB architecture with stateless session beans
- **Spring Framework 4.3.x**: MVC, Security 4.2.x, and dependency injection
- **JPA 2.0/Hibernate 5.1.x**: Direct EntityManager usage (no Repository pattern)
- **jBPM 5.4.0.Final**: Workflow engine with BPMN 2.0 processes
- **Drools**: Business rules engine with external Guvnor repository
- **PostgreSQL**: Primary database with Liquibase migrations
- **WildFly 11**: Java EE application server

### Service Layer Architecture

**Key Architectural Decisions:**
- **Interface-Implementation Separation**: Service interfaces in `services/` module, implementations in `cms-business-process/`
- **EJB Pattern**: All services are `@Stateless` session beans with `@Local` interfaces
- **Base Service Pattern**: Common `BaseService` class provides EntityManager, configuration, and logging
- **No DAO Layer**: Services directly use JPA EntityManager instead of Repository/DAO pattern
- **Configuration Management**: `CMSConfigurator` provides centralized property management and service location

**Transaction Management:**
- Container-managed transactions with `@TransactionAttribute(REQUIRED)`
- Some services use bean-managed transactions for complex workflows
- JTA integration for distributed transaction support

### Entity Model and Data Access

**JPA Entity Patterns:**
- **Inheritance Strategy**: `@Inheritance(JOINED)` for Person/Organization polymorphism
- **Named Entity Graphs**: Performance optimization for complex object loading
- **Audit Trail Integration**: Entities track creation/modification metadata
- **Lookup Table Pattern**: Extensive use of reference data entities (ProviderType, ApplicationStatus, etc.)

**Data Modeling Characteristics:**
- Rich domain objects with business logic embedded in entities
- Mix of direct JPA relationships and transient collections for performance
- Business process integration via `processInstanceId` foreign keys
- String-based status indicators following government system conventions

### Web Layer Architecture

**Controller Patterns:**
- **BaseController**: Common functionality including logging, caching headers, and custom date editors
- **Session Management**: Heavy use of `@SessionAttributes` for multi-step form flows
- **Complex Page Flow**: `ApplicationPageFlowController` manages multi-page provider application process
- **Form Binding**: Custom binder registry system for complex government forms
- **Error Handling**: Custom exception translation and user-friendly error messages

**Form Binding System:**
- **Namespace-Based**: Each form uses unique namespaces (`_02_`, `_03_`, etc.) for field isolation
- **Bidirectional Binding**: Separate methods for request-to-model and model-to-view binding
- **Validation Integration**: Form errors translated from Drools business rule validation results
- **PDF Generation**: Integrated document generation for compliance and audit purposes
- **Security**: Built-in XSS protection and server-side hash validation

### Business Process Integration

**jBPM Workflow Architecture:**
- **BPMN 2.0 Processes**: Complete provider screening lifecycle defined in `ApplicationProcess.bpmn`
- **Task-Based Workflow**: Validation, automatic screening, manual review, and approval/rejection paths
- **Business Rules Integration**: Drools rules engine for validation and screening logic
- **External System Integration**: Hooks for DMF, LEIE, and other federal screening databases
- **Process Monitoring**: Custom event listeners for audit trail generation

**Workflow State Management:**
- Applications linked to process instances via `processInstanceId`
- Persistent workflow state in PostgreSQL with JTA transaction management
- Task assignment and role-based workflow routing
- Business process variables for decision gateway logic

### Security Implementation

**Multi-Layer Security:**
- **Spring Security 4.2**: Role-based access control with custom authentication providers
- **Custom Principal**: `CMSPrincipal` extends standard Principal with system metadata
- **Database Authentication**: Primary authentication via `DomainDatabaseAuthenticationProvider`
- **LDAP Support**: Configurable but currently disabled LDAP integration
- **Session Management**: Traditional servlet-based sessions with configurable timeout

**Authorization Patterns:**
- **Role-Based Access**: PROVIDER, SERVICE_ADMINISTRATOR, SERVICE_AGENT, SYSTEM_ADMINISTRATOR
- **URL-Based Security**: Path-based access control with hierarchical permissions
- **Method-Level Security**: EJB security integration for fine-grained authorization
- **CSRF Protection**: Built-in CSRF token validation for form submissions

### Legacy Architecture Characteristics

This 7-year-old codebase exhibits several era-specific patterns:

**Java EE Legacy Patterns:**
- Traditional EJB 3.x patterns with annotations
- Container services heavily utilized (JNDI, JTA, etc.)
- XML-heavy configuration (Spring Security, persistence.xml)
- Monolithic deployment model (EAR files)

**Data Access Patterns:**
- Direct EntityManager usage instead of modern Repository pattern
- Manual transaction boundary management
- String-based JPQL queries embedded in service methods
- Limited use of modern JPA features (Criteria API, etc.)

**Integration Complexity:**
- Multi-technology stack integration (JPA + jBPM + Drools + Spring Security)
- External system integration designed for government compliance
- Complex state machine for application lifecycle management
- Document generation integrated into form processing

### Development Guidelines

**Common Development Tasks:**
- **Adding Provider Types**: Modify entities in `services/`, update forms in `cms-web/`, add validation rules
- **Business Rule Changes**: Update Drools DRL files, modify validation binders
- **UI Changes**: Edit JSP templates or convert to Handlebars for reusability
- **Database Changes**: Create Liquibase changesets, update entity mappings
- **API Endpoints**: Add Spring MVC controllers with FHIR transformers for external integration
- **Workflow Changes**: Modify BPMN processes, update task handlers

**Key Files and Locations:**
- Entity definitions: `psm-app/services/src/main/java/gov/medicaid/entities/`
- Service implementations: `psm-app/cms-business-process/src/main/java/gov/medicaid/services/impl/`
- Web controllers: `psm-app/cms-web/src/main/java/gov/medicaid/controllers/`
- Form binders: `psm-app/services/src/main/java/gov/medicaid/binders/`
- Business processes: `psm-app/cms-business-process/src/main/resources/`
- Database migrations: `psm-app/db/changelog/`

### External Dependencies
- CAV ETL application for external data source integration (LEIE, DMF, PECOS)
- SMTP server for email notifications
- PostgreSQL database server
- WildFly application server with full Java EE profile
- External Guvnor rules repository (optional)