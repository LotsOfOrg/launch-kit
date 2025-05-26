# Launch Kit Phase 1 Implementation Plan

## Phase 1 Overview: Core Authentication & Admin (Weeks 1-2)

### Deliverables
- Authentication utilities (password hashing, verification)
- Pre-built auth routes (login, logout, register)
- Session-based auth beforeware
- Admin panel utilities
- CSRF protection
- Basic UI components

### Sprint Breakdown

## Sprint 1: Core Authentication (Week 1, Days 1-3)

### Goals
- Implement core authentication utilities
- Create session-based auth system
- Build pre-built auth routes
- Integrate with FastHTML patterns

### Issues

#### Issue 1: Core Authentication Utilities Module
**File**: `nbs/00_auth.ipynb`
**Description**: Create the foundational authentication utilities module

**Tasks**:
- [ ] Set up nbdev notebook structure with proper exports
- [ ] Implement `hash_password()` using bcrypt
- [ ] Implement `verify_password()` with timing-safe comparison
- [ ] Create `user_auth_before()` beforeware function
- [ ] Add session management utilities
- [ ] Write comprehensive tests for all functions
- [ ] Add Markdown documentation cells explaining usage

**Technical Considerations**:
- Research bcrypt best practices (cost factor 12)
- Ensure timing-safe string comparison for security
- Study FastHTML session handling patterns
- Reference: FastHTML docs on beforeware and sessions

**Potential Challenges**:
- Ensuring thread safety for session handling
- Proper error handling for invalid sessions
- Making beforeware skipping patterns flexible

#### Issue 2: Pre-built Auth Routes Module
**File**: `nbs/01_auth_routes.ipynb`
**Description**: Create customizable pre-built authentication routes

**Tasks**:
- [ ] Implement `setup_auth_routes()` function
- [ ] Create login route (GET and POST)
- [ ] Create signup/register route (GET and POST)
- [ ] Create logout route
- [ ] Add password reset flow (optional for Phase 1)
- [ ] Support custom templates for each route
- [ ] Add HTMX support for all forms
- [ ] Create example usage in notebook

**Technical Considerations**:
- Study FastHTML routing patterns (`rt` object)
- Research HTMX form submission patterns
- Look into MonsterUI form components
- Reference: FastHTML docs on routing and forms

**Potential Challenges**:
- Making routes customizable without complexity
- Handling form validation elegantly
- Ensuring proper redirect flows (303 status codes)

#### Issue 3: User Model and Database Integration
**File**: `nbs/09_database.ipynb` (partial)
**Description**: Create basic user model and database utilities

**Tasks**:
- [ ] Create simple User model structure
- [ ] Add database initialization utilities
- [ ] Create user CRUD operations
- [ ] Add migration helpers (if needed)
- [ ] Document database schema
- [ ] Create test fixtures

**Technical Considerations**:
- Research FastHTML database patterns
- Study SQLite best practices
- Look into database migration strategies

**Potential Challenges**:
- Making database layer pluggable
- Handling different database backends
- Ensuring proper indexing for performance

## Sprint 2: Admin Panel & Permissions (Week 1, Days 4-5)

### Goals
- Build permission system with role-based access control
- Create admin panel utilities
- Implement admin routes with CRUD operations

### Issues

#### Issue 4: Permissions System Module
**File**: `nbs/02_permissions.ipynb`
**Description**: Create role-based access control decorators and utilities

**Tasks**:
- [ ] Implement `require_auth()` check function
- [ ] Implement `require_role()` check function
- [ ] Implement `require_permission()` check function
- [ ] Create `auth_required` decorator
- [ ] Add role hierarchy support
- [ ] Create permission mapping utilities
- [ ] Write comprehensive tests
- [ ] Document permission patterns

**Technical Considerations**:
- Study FastHTML request/session patterns
- Research RBAC best practices
- Look into decorator patterns in Python

**Potential Challenges**:
- Making decorators work with FastHTML routes
- Handling async routes properly
- Creating flexible permission hierarchies

#### Issue 5: Admin Panel Utilities Module
**File**: `nbs/03_admin.ipynb`
**Description**: Create utilities for building admin panels

**Tasks**:
- [ ] Create `AdminPanel` utility class
- [ ] Implement `generate_list_view()` for models
- [ ] Implement `generate_edit_form()` for models
- [ ] Add pagination utilities
- [ ] Create filtering/search helpers
- [ ] Add bulk action support
- [ ] Document admin patterns
- [ ] Create example admin dashboard

**Technical Considerations**:
- Research MonsterUI table components
- Study HTMX patterns for dynamic updates
- Look into form generation patterns

**Potential Challenges**:
- Making admin utilities flexible enough
- Handling different model types
- Creating intuitive filtering UI

#### Issue 6: Admin Routes Module
**File**: `nbs/04_admin_routes.ipynb`
**Description**: Create pre-built admin routes

**Tasks**:
- [ ] Implement `setup_admin_routes()` function
- [ ] Create model list routes
- [ ] Create model create/edit routes
- [ ] Create model delete routes
- [ ] Add admin dashboard route
- [ ] Support custom admin templates
- [ ] Add HTMX enhancements
- [ ] Create usage examples

**Technical Considerations**:
- Study RESTful route patterns
- Research HTMX modal patterns
- Look into bulk operations

**Potential Challenges**:
- Handling nested resources
- Making routes customizable
- Ensuring proper authorization

## Sprint 3: Middleware, UI Components & Integration (Week 2, Days 1-3)

### Goals
- Implement CSRF protection
- Create rate limiting utilities
- Build reusable UI components
- Complete integration and testing

### Issues

#### Issue 7: Middleware Module
**File**: `nbs/07_middleware.ipynb`
**Description**: Create security middleware utilities

**Tasks**:
- [ ] Implement `setup_csrf_protection()` function
- [ ] Create CSRF token generation/validation
- [ ] Implement `RateLimiter` class
- [ ] Add IP-based rate limiting
- [ ] Create middleware chaining helpers
- [ ] Write security tests
- [ ] Document middleware usage
- [ ] Create example integrations

**Technical Considerations**:
- Research CSRF protection patterns
- Study rate limiting algorithms
- Look into Redis integration for production

**Potential Challenges**:
- Making CSRF work with HTMX
- Handling distributed rate limiting
- Ensuring minimal performance impact

#### Issue 8: UI Components Module
**File**: `nbs/10_components.ipynb`
**Description**: Create reusable UI components using MonsterUI

**Tasks**:
- [ ] Create `LoginForm` component
- [ ] Create `SignupForm` component
- [ ] Create `AdminTable` component
- [ ] Create `UserMenu` component
- [ ] Create `PermissionBadge` component
- [ ] Add form validation display
- [ ] Ensure HTMX integration
- [ ] Document component usage

**Technical Considerations**:
- Deep dive into MonsterUI components
- Study HTMX form patterns
- Research accessible form design

**Potential Challenges**:
- Maintaining MonsterUI consistency
- Handling form validation elegantly
- Creating flexible components

#### Issue 9: Integration Testing & Examples
**File**: `nbs/examples/minimal_app.ipynb`
**Description**: Create comprehensive examples and integration tests

**Tasks**:
- [ ] Create minimal auth example
- [ ] Create custom auth example
- [ ] Create admin panel example
- [ ] Write integration tests
- [ ] Test all components together
- [ ] Create troubleshooting guide
- [ ] Document common patterns
- [ ] Performance benchmarks

**Technical Considerations**:
- Study FastHTML testing patterns
- Research integration testing strategies
- Look into performance profiling

**Potential Challenges**:
- Testing session-based auth
- Simulating HTMX requests
- Ensuring examples are realistic

## Technical Research Required

### Week 0 Preparation
1. **FastHTML Deep Dive**
   - Session handling mechanisms
   - Beforeware patterns and middleware
   - Database integration patterns
   - Testing strategies

2. **MonsterUI Study**
   - Component library overview
   - Form components and validation
   - Table and data display patterns
   - HTMX integration examples

3. **Security Research**
   - bcrypt best practices
   - CSRF protection in AJAX apps
   - Session security patterns
   - Rate limiting strategies

4. **nbdev Workflow**
   - Project setup and configuration
   - Export patterns
   - Testing in notebooks
   - Documentation generation

## Success Metrics

### Sprint 1 Complete When:
- [ ] Basic auth flow works end-to-end
- [ ] All auth utilities have 95%+ test coverage
- [ ] Documentation shows clear usage patterns
- [ ] No hidden magic or wrapped functions

### Sprint 2 Complete When:
- [ ] Admin panel can perform CRUD on any model
- [ ] Permission system is flexible and clear
- [ ] All utilities work independently
- [ ] Examples show real FastHTML patterns

### Sprint 3 Complete When:
- [ ] CSRF protection works with all forms
- [ ] Rate limiting is production-ready
- [ ] UI components are consistent with MonsterUI
- [ ] Full integration example runs successfully

## Risk Mitigation

### Technical Risks
1. **FastHTML Pattern Misalignment**
   - Mitigation: Continuous testing with real FastHTML apps
   - Weekly review of patterns with FastHTML examples

2. **MonsterUI Integration Issues**
   - Mitigation: Early prototyping of all components
   - Direct communication with MonsterUI maintainers

3. **Performance Concerns**
   - Mitigation: Benchmark from day 1
   - Profile all middleware and beforeware

### Process Risks
1. **Scope Creep**
   - Mitigation: Strict adherence to "utilities, not framework"
   - Regular reviews against core principles

2. **Over-Engineering**
   - Mitigation: Start with simplest implementation
   - Refactor only when needed

## Definition of Done

Each module is complete when:
1. [ ] All functions are exported and documented
2. [ ] Tests achieve 90%+ coverage
3. [ ] Examples demonstrate real usage
4. [ ] No hidden behavior or magic
5. [ ] Works with standard FastHTML patterns
6. [ ] Performance impact is <10ms
7. [ ] Security best practices are followed
8. [ ] Documentation is clear and complete