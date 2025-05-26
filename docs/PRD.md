# Launch Kit PRD - FastHTML SaaS Utilities

## Product Overview

**Vision:** Provide transparent, composable utilities that help developers build production-ready SaaS applications with FastHTML without hiding the framework or adding magic.

**Philosophy:** Like MonsterUI adds UI components to FastHTML, Launch Kit adds SaaS functionality through simple utilities you can understand, customize, and control.

**Target Users:**
- FastHTML developers building SaaS applications
- Teams who want SaaS features without framework lock-in
- Developers who value transparency and control

## Core Principles

1. **No Magic** - Every utility is inspectable and understandable
2. **Composable** - Import only what you need, ignore the rest
3. **Transparent** - See exactly what's happening, no hidden behavior
4. **FastHTML-First** - Enhance FastHTML patterns, don't replace them
5. **Override-Friendly** - Sensible defaults with full customization

## Success Metrics

- Developers can add auth to FastHTML app in <5 minutes
- 100% of utilities are self-contained and optional
- Zero wrapper functions or framework abstractions
- Documentation shows real FastHTML patterns throughout
- Every utility can be customized or replaced

## Package Structure

### Phase 1: Core Authentication & Admin (Weeks 1-2)

**Deliverables:**
- Authentication utilities (password hashing, verification)
- Pre-built auth routes (login, logout, register) 
- Session-based auth beforeware
- Admin panel utilities
- CSRF protection
- Basic UI components

**Structure:**
```
launch_kit/
├── auth.py          # hash_password(), verify_password(), user_auth_before()
├── auth_routes.py   # setup_auth_routes() with customizable templates
├── permissions.py   # require_auth(), require_role() checks
├── admin.py         # AdminPanel utilities, generate_list_view()
├── middleware.py    # setup_csrf_protection(), RateLimiter
├── components.py    # LoginForm, AdminTable, other UI components
```

### Phase 2: Teams & Billing (Week 3)

**Deliverables:**
- Team management utilities
- Billing integration helpers (Stripe, Paddle)
- Subscription management
- Team invitation system

**Structure:**
```
launch_kit/
├── teams.py         # Team model, invitation utilities
├── billing.py       # StripeClient, subscription helpers
├── billing_ui.py    # PricingTable, BillingSettings components
```

### Phase 3: SaaS Features (Week 4)

**Deliverables:**
- Feature flags with percentage rollouts
- Audit logging for compliance
- Background job queue
- Email utilities
- Search functionality
- Data export (CSV/JSON)

**Structure:**
```
launch_kit/
├── saas_features.py # FeatureFlags, audit_log(), JobQueue
├── search.py        # SearchIndex for full-text search
├── exports.py       # export_to_csv(), create_export_response()
├── api.py           # API key management, validation
```

## Key Features & Implementation

### 1. Authentication System

**NOT This:**
```python
# ❌ Hidden magic
app = create_saas_app(features=["auth"])
```

**But This:**
```python
# ✅ Transparent utilities
from fasthtml.common import *
from launch_kit.auth import user_auth_before, hash_password
from launch_kit.auth_routes import setup_auth_routes

app, rt = fast_app()  # Standard FastHTML

# Add auth beforeware
beforeware = Beforeware(
    user_auth_before,
    skip=['/auth/login', '/auth/signup', '/static/.*']
)

# Add auth routes (optional - build your own if preferred)
setup_auth_routes(rt)
```

**Key Utilities:**
- `hash_password(password)` - Returns bcrypt hash
- `verify_password(password, hash)` - Timing-safe verification
- `user_auth_before(req, sess)` - Beforeware that adds user to session
- `setup_auth_routes(rt, login_template=None)` - Optional pre-built routes

### 2. Admin Panel

**Utilities, not framework:**
```python
from launch_kit.admin import AdminPanel, setup_admin_routes
from launch_kit.permissions import require_role

# Option 1: Use pre-built admin routes
setup_admin_routes(rt, models=[User, Team])

# Option 2: Build your own with utilities
@rt("/admin/users")
def get(req, sess):
    if not require_role("admin", req, sess):
        return RedirectResponse('/auth/login')
    
    admin = AdminPanel()
    return Title("Users"), admin.generate_list_view(User)
```

### 3. Feature Flags

**Simple, powerful, transparent:**
```python
from launch_kit.saas_features import FeatureFlags

flags = FeatureFlags({
    "new_ui": {"percentage": 50},      # 50% rollout
    "billing": True,                   # Everyone
    "beta": {"users": [1, 2, 3]}      # Specific users
})

@rt("/")
def get(req, sess):
    user = sess.get('user')
    if flags.is_enabled('new_ui', user):
        return NewDashboard()
    return OldDashboard()
```

### 4. CSRF Protection

**Works with FastHTML forms:**
```python
from launch_kit.middleware import setup_csrf_protection

csrf_input, verify_csrf = setup_csrf_protection(app, secret_key)

# In your forms
def LoginForm(session):
    return Form(
        csrf_input(session),  # Adds hidden CSRF token
        Input(name="username"),
        Input(name="password", type="password"),
        Button("Login")
    )
```

### 5. Audit Logging

**For compliance and debugging:**
```python
from launch_kit.saas_features import audit_log

@rt("/admin/delete-user/{id}")
def post(req, sess, id: int):
    user = sess['user']
    audit_log(user['id'], 'delete_user', f'user:{id}', 
              details={'ip': req.client.host})
    # Delete user...
```

## Development Approach

### nbdev-Based Development

Each module developed as a notebook with:
- Inline documentation
- Visual examples
- Integrated tests
- Export markers

Example notebook structure:
```python
#| default_exp auth

#| export
def hash_password(password: str) -> str:
    """Hash password using bcrypt"""
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

#| test
password = "test123!"
hashed = hash_password(password)
assert verify_password(password, hashed)
```

### Testing Philosophy

Tests are simple and explicit:
```python
def test_feature_flags():
    flags = FeatureFlags({"test": {"percentage": 100}})
    assert flags.is_enabled("test", {"id": 1})
    
def test_rate_limiter():
    limiter = RateLimiter(5, 60)
    for _ in range(5):
        assert limiter.is_allowed("key")
    assert not limiter.is_allowed("key")
```

## Documentation

### Examples First

Every utility includes working examples:
```python
# Simple auth example
from fasthtml.common import *
from launch_kit.auth import setup_auth_routes

app, rt = fast_app()
setup_auth_routes(rt)
serve()  # Working app with auth!
```

### Tutorial Notebooks

1. **Minimal App** - Auth in 10 lines
2. **Custom Auth** - Replace any component
3. **Full SaaS** - All features integrated
4. **Production Deploy** - Real-world setup

## Success Criteria

### Phase 1 Complete When:
- [x] Developers can add auth without reading source code
- [x] All utilities work independently
- [x] No wrapper functions or hidden behavior
- [x] Examples show real FastHTML patterns
- [x] 90%+ test coverage

### Phase 2 Complete When:
- [x] Team management works with standard FastHTML
- [x] Billing integration is copy-paste simple
- [x] All UI components use HTMX properly
- [x] Documentation includes production examples

### Phase 3 Complete When:
- [x] Feature flags enable A/B testing
- [x] Audit logs meet compliance needs
- [x] Search works with any FastHTML app
- [x] All utilities are production-tested

## Technical Requirements

### Dependencies
- FastHTML (latest)
- bcrypt (auth)
- python-dotenv (config)
- stripe (optional - billing)
- Optional: Redis for production rate limiting

### Performance
- Utilities add <10ms overhead
- Rate limiting uses Redis in production
- CSRF tokens are cryptographically secure
- Password hashing uses bcrypt factor 12

### Security
- Timing-safe password comparison
- Secure session handling
- CSRF protection on state changes  
- Audit logging for compliance
- Rate limiting on all endpoints

## Out of Scope

These violate our transparency principle:
- ❌ Wrapper functions hiding FastHTML
- ❌ Magic configuration files
- ❌ Automatic route registration
- ❌ Framework abstractions
- ❌ Hidden middleware chains

## Development Process

1. **Start with use case** - What does developer need?
2. **Write simplest utility** - One function, one purpose
3. **Add to notebook** - Document with examples
4. **Test in real app** - Ensure FastHTML compatibility
5. **Get feedback** - Is it transparent enough?

## Risk Mitigation

**Risk:** Developers expect magic framework
**Mitigation:** Clear docs showing this is utilities, not framework

**Risk:** Feature creep toward abstraction
**Mitigation:** Every PR must maintain transparency principle

**Risk:** Utilities become interdependent  
**Mitigation:** Each utility must work standalone

## Success Examples

### Good Launch Kit Code:
```python
# Transparent - developer sees everything
from launch_kit.auth import hash_password
hashed = hash_password("user_password")
```

### Bad Launch Kit Code:
```python
# Hidden magic - violates principles
from launch_kit import SaaSApp
app = SaaSApp(features=["all"])  # What's happening??
```

## Summary

Launch Kit provides **utilities, not a framework**. Every function is:
- **Transparent** - You can read the source
- **Optional** - Use only what you need
- **Customizable** - Override anything
- **FastHTML-native** - Uses standard patterns

This approach gives developers SaaS building blocks without sacrificing control or understanding of their application.