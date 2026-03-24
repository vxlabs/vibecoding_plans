Technology Stack: React + C#/.NET + Azure | Condensed for Token Efficiency
🚨 MANDATORY PRE-ACTION CHECKLIST

STOP AND CHECK before ANY action: Search for existing files before creating new ones • Check existing import patterns in similar files before writing imports • Use absolute imports with @ aliases (NEVER relative imports like ./, ../) • Use API_ENDPOINTS.* constants (NEVER hard-coded URLs) • Use TIMING_CONFIG.* constants (NEVER hard-coded delays) • Run linting immediately after any code change • Default commands to background execution unless specified otherwise • Check environment context (dev/staging/prod) before making changes
📂 STRICT IMPORT & CONSTANTS REQUIREMENTS
Required Import Patterns

✅ @components/, @pages/, @hooks/, @services/, @utils/, @types/ • @shared/constants, @shared/config, @shared/types • @environments/ for environment-specific configurations
Forbidden Patterns

❌ ./, ../, ../../ (will cause linting errors) • Hard-coded URLs: 'https://api.example.com', 'http://localhost:3000' • Hard-coded timing: setTimeout(fn, 1000), delay: 300 • Environment-specific values in code: 'development', 'production'
Constants Management

// ✅ GOOD - Use constants
import { API_ENDPOINTS, TIMING_CONFIG, ENV_CONFIG } from '@shared/constants';
const response = await fetch(`${API_ENDPOINTS.USERS}/${id}`);
setTimeout(callback, TIMING_CONFIG.AUTH_DELAY);

// ❌ BAD - Hard-coded values
const response = await fetch(`https://api.example.com/users/${id}`);
setTimeout(callback, 1000);

🏗️ ARCHITECTURE STANDARDS
Clean Architecture Layers (Strict Enforcement)

Domain: Core business entities, value objects, domain services • Application: Use cases, application services, DTOs, interfaces • Infrastructure: External services, databases, Azure integrations, repositories • WebAPI/Presentation: Controllers, Razor pages, React components, API models
SOLID Principles (Mandatory)

Single Responsibility: Each class/component has one reason to change • Open/Closed: Open for extension, closed for modification • Liskov Substitution: Derived classes must be substitutable for base classes • Interface Segregation: Many specific interfaces over one general interface • Dependency Inversion: Depend on abstractions, not concretions
Dependency Injection Patterns

Use constructor injection for required dependencies • Use property injection sparingly for optional dependencies • Register services with appropriate lifetimes (Singleton, Scoped, Transient) • Avoid service locator anti-pattern
Code Simplicity Principles

Prefer composition over inheritance • Write self-documenting code with clear variable and method names • Keep methods small - single responsibility, max 20-30 lines • Avoid deep nesting - use early returns and guard clauses • Minimize cognitive complexity - prefer explicit over clever code
🔧 C# (.NET 8+) STANDARDS
Code Style & Structure

Use async/await for all I/O operations with CancellationToken • Add XML documentation comments on all public types and members • Use ILogger<T> with structured logging (include contextual properties) • Default access modifier: private unless otherwise specified • Use meaningful, descriptive variable and method names • Prefer record types for immutable data structures
Data Access Patterns

Simple CRUD: Entity Framework Core with proper DbContext lifecycle • Performance-critical: Dapper with parameterized queries and connection pooling • Database migrations: Use DBUp for schema changes in production environments • Always use parameterized queries to prevent SQL injection • Implement proper transaction management for multi-step operations
Error Handling & Resilience

Implement global exception handling middleware • Return standardized error responses in APIs with correlation IDs • Use custom exception types for business rule violations • Implement retry policies for transient failures • Log exceptions with full context and correlation IDs for tracing
Testing Standards

Unit Tests: MSTest for all public methods and business logic • Mocking: NSubstitute for dependency mocking • Integration Tests: Test API endpoints and database interactions • Coverage: Aim for 80%+ code coverage on business logic • Use TestContainers for database integration tests when needed • Mock external dependencies and services in tests
⚛️ REACT + TYPESCRIPT STANDARDS
Component Architecture (Stack-Wide)

Always use functional components with hooks (no class components) • Enforce strict TypeScript - avoid any type, use proper type definitions • Props interface at the top of each component file • Named exports preferred over default exports for better refactoring • Component composition over prop drilling for state management
State Management Patterns

Use React hooks for local component state (useState, useEffect) • Context API for shared state across component trees • Consider Redux Toolkit for complex global state management • Implement proper state normalization for complex data structures • Use custom hooks to encapsulate stateful logic
Styling & UI Standards

Tailwind CSS for utility-first styling approach • Responsive design with mobile-first approach • Consistent spacing using Tailwind spacing scale • Accessibility - proper ARIA labels, semantic HTML, keyboard navigation • Design system consistency across all components
Performance Optimization

Implement React.memo for expensive component renders • Use useMemo and useCallback for expensive computations • Lazy load non-critical components and routes with React.lazy • Optimize bundle size with proper tree shaking and code splitting • Implement proper error boundaries for graceful error handling
Testing Standards

Jest + React Testing Library for component testing • Test user interactions and behavior, not implementation details • Mock external dependencies and API calls • Aim for 80%+ test coverage on components and custom hooks • Use MSW (Mock Service Worker) for API mocking in tests
🌐 RAZOR PAGES / MVC STANDARDS
Page & Controller Structure

Use Razor Pages with PageModel pattern for simple CRUD operations • Use MVC Controllers for complex business logic and API endpoints • Separate business logic into services with dependency injection • Use ViewModels for complex data binding and validation • Implement proper model validation with data annotations
Security & Validation

Use Anti-Forgery tokens for all forms and state-changing operations • Validate all input data server-side with proper error handling • Implement proper authorization with policies and claims • Sanitize output to prevent XSS attacks • Use HTTPS redirects and secure headers
🧠 MEMORY BANK & PROJECT INTELLIGENCE
Memory Bank Structure (Critical for AI Context)

Maintain structured documentation files for project understanding:
Core Files (Required): memory-bank/architectureBreakdown.md - Codebase structure, entry points, patterns • memory-bank/productContext.md - Project purpose, problems solved, goals • memory-bank/activeContext.md - Current work focus, recent changes, next steps • memory-bank/systemPatterns.md - Design patterns, component relationships • memory-bank/techContext.md - Technologies, setup, constraints, dependencies • memory-bank/progress.md - Status, known issues, testing coverage
Additional Context Files: Testing strategies and coverage reports • SOLID principle implementation guides • Design pattern implementation examples • Debugging and log analysis guides
Project Intelligence Capture (.cursorrules)

Document and learn from patterns discovered during development:
What to Capture: Critical implementation paths and successful patterns • SOLID principle implementation strategies that work • Design pattern selection criteria for specific scenarios • Testing approach preferences and effective patterns • Debugging procedure preferences and log analysis patterns • Common failure modes and their proven solutions • Project-specific architectural patterns and conventions • Known architectural challenges and their resolutions • Evolution of architectural decisions and rationale

Learning Process: 1. Identify new architectural or coding patterns during development 2. Validate effectiveness with team and through testing 3. Document in .cursorrules for future AI reference 4. Update debugging procedures and memory bank 5. Apply learned patterns consistently across projects
Environment Configuration Management

Development: Local development with proper configuration isolation • Staging: Production-like environment for final validation and testing • Production: Live environment with comprehensive monitoring • Use environment-specific configuration files (appsettings.{env}.json) • Never hard-code environment values - use configuration constants • Implement proper environment promotion and validation processes
🔒 SECURITY STANDARDS
Authentication & Authorization

Use Azure AD B2C for user authentication and identity management • Implement JWT token validation for API endpoints • Use role-based authorization with proper claims and policies • Implement proper session management and timeout handling • Use secure cookie settings and CSRF protection
Data Protection & Privacy

Encrypt sensitive data at rest and in transit • Use HTTPS for all communications (TLS 1.2 minimum) • Implement proper input validation and sanitization • Follow OWASP security guidelines and best practices • Implement proper data retention and deletion policies
Security Monitoring

Log all authentication and authorization events • Monitor for suspicious activities and security threats • Implement proper security headers and content security policies • Use Azure Security Center for threat detection and monitoring
📊 DEBUGGING PROCESS GUIDE & LOG ANALYSIS
Test-Driven Debugging Workflow

Critical Process for All Development:

    Run Failing Test: dotnet test (all tests) • dotnet test --filter "FullyQualifiedName={FullyQualifiedName}" (specific test)
    Locate Log Files: Test logs stored in TestLogs/ directory • Each component has isolated log file • Log files named after components • Implementation files in BLL/ directory
    Log Analysis Process: Start from beginning of log file • Read logs sequentially - don't jump around • Examine each entry for: Timestamp, Log level, Message, Context data • Follow sequence of operations to understand flow • Look for patterns or unexpected behavior • Identify divergence from expected execution flow
    Debug-Driven Development: Make code changes based on log analysis findings • Add logging before making changes to capture new context • Verify logs after changes to confirm fixes • Run test again to validate solution • Document solution for future reference

Structured Logging Standards

Consistent log levels: DEBUG, INFO, WARN, ERROR, FATAL • Clear, descriptive messages with relevant context • Include correlation IDs for request tracing across services • Structured JSON format for log entries with contextual properties • Component isolation - each component logs to separate files • Dependency injection for logging configuration
Log Management & Organization

Organize logs by component/feature for easy navigation • Maintain consistent naming conventions across all log files • Clean up old log files periodically to manage disk space • Archive important debugging sessions for knowledge retention • Create log file automatically when tests run • Use consistent log file naming after component names
Integration with Development Process

Include debugging considerations in all design patterns • Ensure SOLID principles support testability and debuggability • Design integrations with logging in mind from the start • Create debugging documentation as part of architecture • Update debugging procedures after significant changes • Maintain troubleshooting guides with common solutions
🧪 TESTING STANDARDS
Testing Strategy

Unit Tests: Test individual components and business logic in isolation • Integration Tests: Test API endpoints, database interactions, and service integrations • End-to-End Tests: Test critical user journeys and business workflows • Performance Tests: Test system performance under expected load • Security Tests: Test for common vulnerabilities and security issues
Coverage Requirements

Aim for 80%+ code coverage on business logic and critical paths • Focus on testing behavior and outcomes, not implementation details • Mock external dependencies and services appropriately • Use test data builders and factories for consistent test setup • Implement proper test isolation and cleanup
Test Organization

Organize tests by feature or domain area • Use descriptive test names that explain the scenario being tested • Implement proper test categorization (unit, integration, e2e) • Use shared test utilities and helpers for common scenarios • Maintain test documentation and examples
🎯 IMPLEMENTATION WORKFLOW & ARCHITECTURE ANALYSIS
Detailed Analysis Phase (Before Any Implementation)

Required Steps for Every Feature/Change:

    Scan Codebase Structure: Identify main entry points and core modules/components • Map key dependencies and architectural patterns in use • Document existing patterns and note architectural violations • Evaluate SOLID principle adherence in current code • Inventory design patterns currently in use
    Analyze Core Components: Document purpose and responsibilities of each component • Map dependencies and consumers of components • Identify technical debt and architectural issues • Perform SOLID principle analysis on existing code • Generate design pattern recommendations for improvements
    Create Architecture Diagram: Show component interactions and data flows • Highlight architectural improvements needed • Document integration points and dependencies • Map service communication patterns • Identify potential bottlenecks and issues

Strategy Development Phase

Plan Before Implementation:

    Select Appropriate Design Patterns: Choose patterns based on specific problem context • Consider GoF patterns: Creational, Structural, Behavioral • Validate pattern selection against SOLID principles • Document pattern selection rationale
    Plan SOLID Principle Implementation: Single Responsibility: One reason to change per class • Open/Closed: Open for extension, closed for modification • Liskov Substitution: Derived classes substitutable for base • Interface Segregation: Many specific interfaces over general • Dependency Inversion: Depend on abstractions, not concretions
    Create Comprehensive Testing Approach: Select appropriate unit test framework (MSTest) • Design mock/stub approach using NSubstitute • Develop test data generation strategies • Plan coverage reporting and metrics (aim for 80%+) • Establish debugging and logging procedures

Implementation Sequence Standards

Follow This Order for All Development:

    Minimize Dependency Conflicts: Sort tasks by dependency order • Create self-contained, independent tasks • Define clear acceptance criteria for each task • Identify and mitigate architectural risks
    Enable Incremental Validation: Implement comprehensive logging first • Create tests before or alongside implementation • Follow software engineering best practices • Prioritize testing framework establishment • Address fundamental architectural issues first
    Validation Phase Requirements: Verify design pattern correctness and effectiveness • Confirm SOLID principle adherence in implementation • Check test coverage metrics meet 80%+ target • Run debugging process for any test failures • Verify log quality and completeness • Document architectural decisions and rationale

📚 DOCUMENTATION STANDARDS (AI-Optimized)
Code Documentation

Include inline comments for complex business logic and algorithms • Write clear XML documentation for all public APIs and methods • Document architectural decisions and design patterns used • Include examples and usage patterns in documentation • Maintain up-to-date API documentation with OpenAPI/Swagger
Project Documentation

Maintain comprehensive README.md with setup and development instructions • Document environment-specific configuration and deployment procedures • Include troubleshooting guides and common issues resolution • Document API endpoints, request/response formats, and error codes • Maintain CHANGELOG.md for significant changes and version history
AI Context Documentation

Structure documentation for future AI consumption and understanding • Include context about business domain and terminology • Document integration points and external dependencies • Maintain decision logs and architectural rationale • Include examples of common patterns and anti-patterns
🧠 MEMORY & CONTEXT MANAGEMENT
Memory Bank Update Triggers

CRITICAL: Update memory bank files when: Discovering new architectural patterns during development • After implementing significant architectural changes • When user requests memory bank updates (MUST review ALL files) • When architecture context needs clarification • After significant debugging sessions that reveal architectural insights • When design patterns prove effective or ineffective • After resolving complex technical challenges
Memory Bank Update Process

Required Workflow for Updates: Review ALL memory bank files for completeness and accuracy • Document current architecture state and recent changes • Clarify next architecture steps and priorities • Update .cursorrules with new patterns and preferences • Update testing strategy based on lessons learned • Update debugging procedures with new insights • Document architectural decisions and their rationale
Project Context Maintenance

Maintain clear project structure and organization standards • Document key architectural decisions with rationale and alternatives considered • Track external dependencies and their purposes, versions, and integration points • Maintain glossary of domain-specific terms, concepts, and business rules • Document common patterns and conventions used across the project • Keep decision logs for major architectural and technical choices
AI Interaction Optimization

Structure code and documentation for AI comprehension and context understanding • Use consistent naming conventions and patterns across all files and components • Maintain clear separation of concerns and responsibilities in code organization • Document complex business rules and logic with clear explanations and examples • Keep project context files updated and easily accessible for AI reference • Include examples of successful patterns and anti-patterns to avoid
🔄 DEVELOPMENT WORKFLOW
Version Control Standards

Use meaningful commit messages that explain the why, not just the what • Create feature branches for all development work • Use pull requests with proper code review processes • Implement proper branching strategy (GitFlow, GitHub Flow, etc.) • Tag releases with semantic versioning
Environment Promotion

Implement proper environment promotion processes • Use staging environment for final validation before production • Implement proper rollback procedures for failed deployments • Use feature flags for controlled feature rollouts • Maintain environment parity to minimize deployment issues