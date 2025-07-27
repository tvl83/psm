# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Provider Screening Module (PSM) is a Java EE Enterprise Application designed for U.S. state Medicare/Medicaid Information Systems (MMIS) to provide a portal for screening service providers as part of the provider enrollment process. The application uses a multi-module Gradle build system with Spring Framework, Hibernate ORM, jBPM workflow engine, and Drools business rules.

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

## Architecture

### Multi-Module Structure
The PSM is organized into several Gradle subprojects within `psm-app/`:

**Core Business Modules:**
- `cms-business-model` - JAXB-generated data types from XML schemas, used by Drools rules
- `cms-business-process` - jBPM workflow definitions and callbacks
- `services` - Service layer with Hibernate entities, business logic, and data access
- `cms-web` - Spring MVC web controllers, JSP/Handlebars templates, REST API

**Supporting Modules:**
- `cms-portal-services` - EAR packaging module that combines all components
- `frontend` - JavaScript management with Node.js dependencies
- `userhelp` - Sphinx-based user documentation
- `integration-tests` - Serenity BDD integration tests
- `db` - Liquibase database migration scripts

### Technology Stack
- **Application Server**: WildFly 11 (Java EE 7)
- **Framework**: Spring Framework 4.3.x with Spring Security 4.2.x
- **ORM**: Hibernate 5.1.x with JPA 2.1
- **Workflow**: jBPM 5.4.0.Final with Drools business rules
- **Database**: PostgreSQL 9.6.x/10.x with Liquibase migrations
- **Frontend**: JavaScript with jQuery, Node.js for dependency management
- **Templates**: Mix of JSP and Handlebars templates
- **Testing**: Spock (Groovy), JUnit, Serenity BDD for integration tests

### Key Architecture Patterns
- **MVC Pattern**: Spring MVC controllers in `cms-web` handle HTTP requests
- **Service Layer**: Business logic encapsulated in services with EJB-style interfaces
- **Data Access**: Hibernate entities with DAO pattern for database operations
- **Workflow Engine**: jBPM processes defined in BPMN files for enrollment workflows
- **Business Rules**: Drools rules for validation and screening logic
- **API**: FHIR-compliant REST API for external integrations

### User Roles and Access Control
1. **Provider** - Create and manage their own enrollments
2. **Service Agent** - Create, view, and edit enrollments for provider clients
3. **Service Admin** - Manage enrollments, provider types, and screening schedules
4. **System Admin** - User management and system administration

### Configuration Files
- `gradle.properties` - Database connection settings (create from template)
- `psm-app/services/src/main/resources/cms.properties` - Application configuration
- `psm-app/cms-web/WebContent/WEB-INF/web.xml` - Web application settings
- `psm-app/cms-web/WebContent/WEB-INF/spring-security.xml` - Security configuration

### Development Workflow
1. **Local Development**: Use WildFly 11 with PostgreSQL database
2. **Database**: Run Liquibase migrations + jBPM SQL script for full schema
3. **Testing**: Unit tests with Spock/JUnit, integration tests with Serenity
4. **Code Quality**: Checkstyle for Java, JSCS for JavaScript
5. **API Documentation**: Generated from Javadoc annotations

### Common Development Tasks
- **Adding new provider types**: Modify provider type entities and forms
- **Business rule changes**: Update Drools rule files in `cms-business-process`
- **UI changes**: Edit JSP templates or Handlebars templates in `cms-web`
- **Database changes**: Create new Liquibase changesets in `db/changelog/`
- **API endpoints**: Add controllers in `cms-web` with FHIR transformers

### External Dependencies
- CAV ETL application for external data source integration (LEIE, DMF, PECOS)
- SMTP server for email notifications
- LDAP server for authentication (optional)