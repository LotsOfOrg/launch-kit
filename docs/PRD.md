# Launch Kit PRD - FastHTML SaaS Toolkit

## Product Overview

**Goal:** Create a comprehensive nbdev-based toolkit that enables developers to build production-ready SaaS applications with FastHTML in under 50 lines of code.

**Target Users:**
- Solo developers building MVPs
- Small teams creating B2B SaaS products
- FastHTML developers who want to skip boilerplate

## Success Metrics
- Developer can create authenticated SaaS app in <10 minutes
- Package has >1000 GitHub stars within 6 months
- 90%+ test coverage maintained
- Documentation satisfaction >4.5/5

## MVP Requirements (Phase 1)

### Epic 1: Core Foundation
**Timeline:** Week 1-2

**User Story:** As a developer, I want to create a basic SaaS app with authentication so I can focus on my business logic.

**Acceptance Criteria:**
- [ ] `create_saas_app()` function creates FastHTML app with single import:
  ```python
  from launch_kit import create_saas_app, SaaSConfig
  app = create_saas_app(SaaSConfig(app_name="MyApp"))
  ```
- [ ] Password authentication with bcrypt hashing (cost factor 12)
- [ ] User registration with email validation:
  - Registration at `/auth/register`
  - Login at `/auth/login`
  - Logout at `/auth/logout`
  - Email verification tokens expire in 24 hours
- [ ] Session management using secure cookies:
  - HTTPOnly, Secure, SameSite=Lax flags
  - Session timeout configurable (default 30 days)
  - Automatic session refresh on activity
- [ ] User dashboard at `/dashboard` with:
  - Welcome message with username
  - Last login timestamp
  - Account settings link
- [ ] SQLite database with migrations:
  - Users table with id, email, username, password_hash, email_verified, created_at
  - Sessions table with id, user_id, token, expires_at
  - Automatic migration on app startup
- [ ] All code in nbdev notebooks with >90% test coverage

**Implementation Tasks:**
1. Set up nbdev project structure:
   - Initialize with `nbdev new` 
   - Configure settings.ini with project metadata
   - Set up GitHub Actions for CI/CD
   - Create development notebook template

2. Create `00_core.ipynb` with base application factory:
   - Define `create_saas_app()` function
   - Implement feature registration system
   - Add middleware setup (CORS, security headers)
   - Include error handling and logging setup

3. Create `01_config.ipynb` with SaaSConfig class:
   - Dataclass with type hints and defaults
   - Environment variable loading with python-decouple
   - Configuration validation on instantiation
   - Feature flags dictionary with type checking

4. Create `02_database.ipynb` with schema and models:
   - SQLAlchemy or fastsql models
   - Migration system using Alembic or custom
   - Connection pooling configuration
   - Query helper functions
   - Seed data for development

5. Create `03_auth.ipynb` with complete auth system:
   - Password hashing with bcrypt
   - Login/logout/register routes
   - Session management middleware
   - Rate limiting (5 attempts per 15 minutes)
   - Password strength validation
   - "Remember me" functionality

6. Create UI components in `04_ui_base.ipynb`:
   - Base layout with navigation
   - Form components with CSRF protection
   - Flash message system
   - Responsive design with Tailwind CSS
   - Loading states and error handling

7. Write tests achieving >90% coverage:
   - Unit tests for each function
   - Integration tests for auth flows
   - UI component rendering tests
   - Security tests (SQL injection, XSS)
   - Performance benchmarks

8. Generate and deploy documentation:
   - API reference from docstrings
   - Interactive examples
   - Getting started guide
   - Deploy to GitHub Pages

### Epic 2: Admin Dashboard
**Timeline:** Week 3

**User Story:** As a SaaS operator, I want an admin dashboard to manage users and view analytics.

**Acceptance Criteria:**
- [ ] Admin dashboard at `/admin` with role-based access:
  - Only users with `is_admin=True` can access
  - Automatic redirect for non-admins
  - Admin middleware checks on all admin routes

- [ ] User management interface with:
  - Sortable table with columns: Username, Email, Status, Created, Last Login, Actions
  - Quick actions: Enable/Disable, Reset Password, Make Admin
  - Bulk actions: Export CSV, Bulk Delete, Bulk Email
  - User detail modal with full profile and activity log

- [ ] Search and filtering capabilities:
  - Real-time search by username/email
  - Filters: Status (active/inactive), Verified (yes/no), Admin (yes/no)
  - Date range filters for registration and last login
  - Results update via HTMX without page reload

- [ ] Analytics dashboard showing:
  - Total users, active today/week/month
  - Registration chart (daily for 30 days)
  - User retention cohort analysis
  - Top referral sources
  - Geographic distribution (by timezone)

- [ ] Comprehensive audit logging:
  - Log all admin actions with timestamp, admin user, action type, target user
  - Searchable audit log interface
  - Export audit logs for compliance
  - Automatic cleanup of logs older than 90 days

### Epic 3: CLI and Templates  
**Timeline:** Week 4

**User Story:** As a developer, I want CLI tools to scaffold new projects quickly.

**Acceptance Criteria:**
- [ ] CLI installation and commands:
  ```bash
  pip install launch-kit
  launch-kit --version  # Shows version
  launch-kit --help     # Shows all commands
  ```

- [ ] Project creation with templates:
  ```bash
  launch-kit new myapp --template=basic  # Creates in ./myapp/
  # Interactive prompts for:
  # - App name
  # - Database (SQLite/PostgreSQL/MySQL)
  # - Authentication providers
  # - Initial admin email
  ```

- [ ] Template specifications:
  - **Basic**: Auth, user dashboard, settings (< 500 lines)
  - **Advanced**: +Teams, billing, admin panel (< 2000 lines)
  - **Enterprise**: +API, webhooks, multi-tenancy (< 5000 lines)
  - All templates include tests and documentation

- [ ] Feature addition commands:
  ```bash
  launch-kit add auth --providers=google,github
  launch-kit add billing --provider=stripe --mode=subscriptions
  launch-kit add teams --features=invites,roles
  launch-kit add api --features=keys,rate-limiting
  ```

- [ ] Database management:
  ```bash
  launch-kit db init      # Create database and tables
  launch-kit db migrate   # Run pending migrations
  launch-kit db rollback  # Rollback last migration
  launch-kit db seed      # Load sample data
  launch-kit db reset     # Drop and recreate
  ```

- [ ] Development tools:
  ```bash
  launch-kit dev          # Start with hot reload, opens browser
  launch-kit dev --port=8000 --no-open  # Custom options
  launch-kit test         # Run test suite with coverage
  launch-kit lint         # Run code quality checks
  ```

## Phase 2: Enhanced Features

### Epic 4: Team Management
- Multi-user teams
- Role-based permissions
- Team switching interface
- Invitation system

### Epic 5: Billing Integration
- Stripe integration
- Subscription management
- Usage-based billing
- Pricing table component

### Epic 6: API Management
- API key generation
- Rate limiting
- API documentation
- Webhook handling

## Technical Requirements

### Development Standards
- All code must be in nbdev notebooks
- 90%+ test coverage required
- All functions must have docstrings and examples
- Documentation auto-generated from notebooks
- Follow FastHTML conventions and patterns

### Performance Requirements
- App startup time <2 seconds:
  - Lazy loading of optional features
  - Precompiled templates
  - Connection pool pre-warming
  
- Page load times <500ms:
  - HTMX for partial page updates
  - Aggressive caching headers for static assets
  - Gzip compression enabled
  - Database query optimization (N+1 prevention)
  
- Concurrent user support:
  - Handle 1000+ concurrent connections
  - Horizontal scaling ready with Redis sessions
  - Database connection pooling (min=5, max=20)
  - Rate limiting: 100 requests/minute per user
  
- Database optimization:
  - Indexes on all foreign keys and commonly queried fields
  - Query execution time logging (warn >100ms)
  - Prepared statements for all queries
  - Automatic EXPLAIN on slow queries in development

### Security Requirements
- OWASP Top 10 compliance:
  - SQL injection prevention via parameterized queries
  - XSS protection with auto-escaping templates
  - CSRF tokens on all state-changing operations
  - Secure authentication with timing-safe comparisons
  - XML external entity (XXE) prevention
  - Security headers: CSP, X-Frame-Options, X-Content-Type-Options
  
- Authentication security:
  - Passwords: min 8 chars, complexity requirements
  - Account lockout after 5 failed attempts (15 min)
  - Password reset tokens expire in 1 hour
  - Two-factor authentication support (TOTP)
  
- Session security:
  - Cryptographically secure session tokens (32 bytes)
  - Session fixation prevention
  - Automatic session invalidation on password change
  - Optional IP address validation
  
- Input validation:
  - All inputs validated with pydantic models
  - SQL injection prevention via ORM
  - File upload restrictions (types, size)
  - Rate limiting on all endpoints
  
- Data protection:
  - Sensitive data encryption at rest
  - HTTPS enforcement in production
  - Security event logging
  - Regular dependency vulnerability scanning

## Dependencies
- FastHTML (latest)
- nbdev (latest)
- fastsql and fastlite or similar ORM
- Bcrypt for password hashing
- Click for CLI

## Out of Scope (Future Versions)
- Advanced analytics
- Multi-language support
- Mobile app integration
- Advanced compliance features (SOC2, HIPAA)

## Development Process

### Workflow
1. Each feature starts as a notebook exploration
2. Code, tests, and docs developed together in notebooks
3. Use `nbdev_preview` for live documentation updates
4. Export to Python modules with `nbdev_export`
5. Continuous integration with GitHub Actions

### Quality Gates
- All notebooks must run without errors
- Tests must pass in CI
- Documentation must build successfully
- Code review required for all changes
- Performance benchmarks must pass

## Risk Mitigation
- **Complexity creep:** Start with MVP, resist feature additions
- **nbdev learning curve:** Provide comprehensive examples
- **FastHTML compatibility:** Pin versions, test regularly
- **Community adoption:** Focus on developer experience and docs