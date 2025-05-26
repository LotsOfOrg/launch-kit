# Launch Kit Technical Vision v2

## Philosophy: Utilities, Not Frameworks

Following Answer.ai's principles of simplicity and transparency, Launch Kit provides composable utilities for FastHTML applications without hiding the underlying framework. Like MonsterUI adds UI components to FastHTML, Launch Kit adds SaaS functionality.

## Core Principles

1. **No Magic**: Everything is explicit and inspectable
2. **Composable**: Import only what you need
3. **Transparent**: You can see exactly what's happening
4. **FastHTML-First**: Enhances FastHTML, doesn't wrap it
5. **Override-Friendly**: Sensible defaults, full customization

## Package Structure (nbdev-based)

```
launch-kit/
├── nbs/
│   ├── index.ipynb              # Package overview
│   ├── 00_auth.ipynb            # Authentication utilities
│   ├── 01_auth_routes.ipynb     # Pre-built auth routes
│   ├── 02_permissions.ipynb     # RBAC decorators
│   ├── 03_admin.ipynb           # Admin panel utilities
│   ├── 04_admin_routes.ipynb    # Pre-built admin routes
│   ├── 05_teams.ipynb           # Team management
│   ├── 06_billing.ipynb         # Billing utilities
│   ├── 07_middleware.ipynb      # CSRF, rate limiting
│   ├── 08_validation.ipynb      # Form validation
│   ├── 09_database.ipynb        # DB utilities
│   ├── 10_components.ipynb      # UI components
│   ├── 11_saas_features.ipynb  # Email, jobs, audit, flags
│   ├── 12_api.ipynb            # API key management
│   ├── 13_search.ipynb         # Search functionality
│   ├── 14_exports.ipynb        # Data export utilities
│   ├── examples/
│   │   ├── minimal_app.ipynb   # Bare minimum example
│   │   ├── custom_auth.ipynb   # Customizing auth
│   │   └── full_saas.ipynb     # Complete example
│   └── tutorials/
│       └── ...
└── launch_kit/
    ├── auth.py                  # Authentication utilities
    ├── permissions.py           # RBAC system
    ├── admin.py                 # Admin utilities
    ├── teams.py                 # Team management
    ├── billing.py               # Billing integration
    ├── middleware.py            # Middleware helpers
    ├── validation.py            # Validation utilities
    ├── components.py            # UI components
    ├── saas_features.py         # Additional SaaS utilities
    ├── api.py                   # API management
    ├── search.py                # Search functionality
    └── exports.py               # Export utilities
```

## Usage Pattern: Explicit and Composable

### Basic FastHTML App with Auth

```python
from fasthtml.common import *
from launch_kit.auth import create_auth_routes, user_auth_before
from launch_kit.middleware import csrf_middleware, rate_limit_middleware

# Standard FastHTML app - no wrapping!
app, rt = fast_app()

# Add beforeware for authentication  
beforeware = Beforeware(
    user_auth_before,
    skip=['/auth/login', '/auth/signup', '/static/.*']
)

# Add pre-built auth routes (or build your own!)
auth_routes = create_auth_routes()
for route in auth_routes:
    rt(route.path, methods=[route.method])(route.handler)

# Your routes - with access to auth utilities
@rt("/")
def get(req, sess):
    user = sess.get('user')
    if user:
        return Title("Home"), Main(f"Welcome {user['username']}!")
    return Title("Welcome"), Main("Welcome! Please ", A("login", href="/auth/login"), ".")
```

### Adding Admin Panel

```python
from launch_kit.admin import create_admin_routes
from launch_kit.permissions import require_role

# Add admin routes with default UI
setup_admin_routes(rt, models=[User, Team, Subscription])

# Or create your own admin routes
@rt("/custom-admin")
def get(req, sess):
    if not require_role("admin", req, sess):
        return RedirectResponse('/auth/login')
    return Title("Admin"), AdminDashboard(
        users=get_all_users(),
        stats=get_admin_stats()
    )
```

## Key Components

### 1. Authentication Utilities (00_auth.ipynb)

```python
#| export
def hash_password(password: str) -> str:
    """Hash a password using bcrypt"""
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

#| export
def verify_password(password: str, hashed: str) -> bool:
    """Verify a password against a hash"""
    return bcrypt.checkpw(password.encode(), hashed.encode())

#| export
def user_auth_before(req, sess):
    """Beforeware for authentication - you control the behavior"""
    auth = sess.get('auth')
    if auth:
        user = get_user(auth)
        if user:
            # Store in session for easy access
            sess['user'] = user
            sess['permissions'] = get_permissions(user.get('role', 'user'))
```

### 2. Pre-built Routes (01_auth_routes.ipynb)

```python
#| export
def setup_auth_routes(rt, login_template=None, signup_template=None, verify_email=True):
    """Setup auth routes on your app's router
    
    Every aspect can be customized or you can build your own.
    """
    login_form = login_template or LoginForm
    signup_form = signup_template or SignupForm
    
    @rt("/auth/login")
    def get(req, sess):
        return Title("Login"), PageLayout(
            login_form(session=sess)
        )
    
    @rt("/auth/login", methods=["POST"])
    async def post(req, sess):
        # Transparent login logic you can inspect
        form = await req.form()
        username = form.get('username')
        password = form.get('password')
        
        if verify_user(username, password):
            sess['auth'] = username
            return RedirectResponse('/', status_code=303)
        
        return Title("Login"), PageLayout(
            login_form(errors={"password": "Invalid credentials"}, session=sess)
        )
    
    # Add other routes similarly...
    @rt("/auth/logout")
    def get(sess):
        sess.pop('auth', None)
        sess.pop('user', None)
        return RedirectResponse('/auth/login', status_code=303)
```

### 3. Permission Decorators (02_permissions.ipynb)

```python
#| export
def require_auth(req, sess):
    """Check if user is authenticated"""
    return sess.get('auth') is not None

#| export
def require_role(role, req, sess):
    """Check if user has specific role"""
    user = sess.get('user')
    return user and user.get('role') == role

#| export
def require_permission(permission, req, sess):
    """Check if user has specific permission"""
    perms = sess.get('permissions', [])
    return permission in perms

#| export
def auth_required(func):
    """Decorator for routes requiring authentication (FastHTML style)"""
    def wrapper(req, sess, *args, **kwargs):
        if not sess.get('auth'):
            return RedirectResponse('/auth/login', status_code=303)
        return func(req, sess, *args, **kwargs)
    return wrapper
```

### 4. Admin Utilities (03_admin.ipynb)

```python
#| export
class AdminPanel:
    """Utility class for building admin panels
    
    Not a framework - just helpful utilities!
    """
    def __init__(self, models=None):
        self.models = models or []
    
    def generate_list_view(self, model, filters=None):
        """Generate a list view for a model"""
        # Returns FastHTML components, not magic
        return Table(
            TableHeader(model._fields),
            TableBody(model.all()),
            cls="admin-table"
        )
    
    def generate_edit_form(self, model, instance=None):
        """Generate an edit form for a model"""
        # Explicit form generation you can customize
        return Form(
            *[Input(name=field, value=getattr(instance, field, ''))
              for field in model._fields],
            Button("Save", type="submit")
        )
```

### 5. Middleware Helpers (07_middleware.ipynb)

```python
#| export
def setup_csrf_protection(app, secret_key, exclude_paths=None):
    """Setup CSRF protection for FastHTML app
    
    Adds CSRF token generation and validation.
    """
    exclude = exclude_paths or []
    
    def generate_csrf_token(sess):
        """Generate and store CSRF token in session"""
        import secrets
        token = secrets.token_urlsafe(32)
        sess['_csrf_token'] = token
        return token
    
    def verify_csrf_token(token, sess):
        """Verify CSRF token"""
        return token and token == sess.get('_csrf_token')
    
    # Add CSRF token to all forms automatically
    def csrf_input(sess):
        token = sess.get('_csrf_token') or generate_csrf_token(sess)
        return Input(type="hidden", name="_csrf_token", value=token)
    
    return csrf_input, verify_csrf_token

#| export
class RateLimiter:
    """Simple rate limiter you can understand and modify"""
    def __init__(self, max_requests, window_seconds):
        self.max_requests = max_requests
        self.window = window_seconds
        self.requests = {}  # In production, use Redis
    
    def is_allowed(self, key):
        # Simple, transparent rate limiting logic
        now = time.time()
        # ... rate limit check
        return True
```

### 6. Validation Utilities (08_validation.ipynb)

```python
#| export
def validate_email(email):
    """Simple email validation"""
    pattern = r'^[\w\.-]+@[\w\.-]+\.\w+$'
    return bool(re.match(pattern, email))

#| export
def validate_password(password, min_length=8, require_special=True):
    """Password validation with clear rules"""
    errors = []
    if len(password) < min_length:
        errors.append(f"Password must be at least {min_length} characters")
    if require_special and not re.search(r'[!@#$%^&*]', password):
        errors.append("Password must contain a special character")
    return errors

#| export
class FormValidator:
    """Simple form validation helper"""
    def __init__(self, rules):
        self.rules = rules
    
    def validate(self, data):
        errors = {}
        for field, validators in self.rules.items():
            for validator in validators:
                error = validator(data.get(field))
                if error:
                    errors[field] = error
                    break
        return errors
```

### 7. UI Components (10_components.ipynb)

```python
#| export
def LoginForm(errors=None, providers=None, session=None, csrf_input=None):
    """Pre-built login form - customize or replace entirely"""
    return Card(
        H2("Login"),
        Form(
            csrf_input(session) if csrf_input else None,
            Input(name="username", placeholder="Username", required=True),
            Input(name="password", type="password", placeholder="Password", required=True),
            errors and Div(*[P(e, cls="error") for e in errors.values()]),
            Button("Login", type="submit", cls="primary"),
            method="POST",
            action="/auth/login",
            hx_post="/auth/login",  # HTMX enhancement
            hx_target="body"
        ),
        providers and OAuthProviders(providers),
        cls="login-card"
    )

#| export
def AdminTable(model, data, actions=None):
    """Reusable admin table with HTMX enhancements"""
    return Table(
        Thead(
            Tr(*[Th(field.replace('_', ' ').title()) for field in model._fields])
        ),
        Tbody(
            *[Tr(
                *[Td(getattr(item, f)) for f in model._fields],
                Td(
                    Button("Edit", hx_get=f"/admin/edit/{item.id}", hx_target="#modal"),
                    Button("Delete", hx_delete=f"/admin/delete/{item.id}", 
                           hx_confirm="Are you sure?")
                ) if actions else None
              ) for item in data],
            id="admin-tbody"
        ),
        cls="admin-table"
    )
```

### 8. Additional SaaS Features (11_saas_features.ipynb)

```python
#| export
# Email utilities
def send_verification_email(email, token, base_url):
    """Send email verification"""
    link = f"{base_url}/auth/verify?token={token}"
    # Email sending logic here
    
#| export  
# Background job utilities
class JobQueue:
    """Simple job queue for background tasks"""
    def __init__(self):
        self.jobs = []
    
    def enqueue(self, func, *args, **kwargs):
        """Add job to queue"""
        self.jobs.append((func, args, kwargs))
    
    def process(self):
        """Process jobs - run in background thread"""
        while self.jobs:
            func, args, kwargs = self.jobs.pop(0)
            func(*args, **kwargs)

#| export
# Audit logging
def audit_log(user_id, action, resource, details=None):
    """Log user actions for compliance"""
    log_entry = {
        "user_id": user_id,
        "action": action,
        "resource": resource,
        "details": details,
        "timestamp": datetime.now(),
        "ip_address": get_client_ip()
    }
    # Save to database
    
#| export
# Feature flags
class FeatureFlags:
    """Simple feature flag system"""
    def __init__(self, flags=None):
        self.flags = flags or {}
    
    def is_enabled(self, flag, user=None):
        """Check if feature is enabled"""
        if flag not in self.flags:
            return False
        
        flag_config = self.flags[flag]
        if isinstance(flag_config, bool):
            return flag_config
        
        # User-specific flags
        if user and 'users' in flag_config:
            return user['id'] in flag_config['users']
        
        # Percentage rollout
        if 'percentage' in flag_config:
            import hashlib
            user_hash = int(hashlib.md5(str(user['id']).encode()).hexdigest(), 16)
            return (user_hash % 100) < flag_config['percentage']
        
        return flag_config.get('enabled', False)
```

## Example: Building a Complete SaaS App

```python
from fasthtml.common import *
from launch_kit.auth import setup_auth_routes, user_auth_before, hash_password
from launch_kit.admin import setup_admin_routes
from launch_kit.permissions import auth_required, require_role
from launch_kit.teams import Team, setup_team_routes
from launch_kit.billing import setup_billing_routes, StripeClient
from launch_kit.components import LoginForm, AdminTable, TeamSelector
from launch_kit.saas_features import FeatureFlags, audit_log

# Standard FastHTML app with database
app, rt = fast_app(
    db_file='myapp.db',
    hdrs=[
        Script(src="https://unpkg.com/htmx.org"),
        Link(rel="stylesheet", href="/static/app.css")
    ]
)

# Setup CSRF protection
csrf_input, verify_csrf = setup_csrf_protection(app, secret_key="your-secret")

# Add beforeware for authentication
beforeware = Beforeware(
    user_auth_before,
    skip=[r'/auth/.*', r'/static/.*', r'/favicon\.ico']
)

# Setup feature flags
flags = FeatureFlags({
    "new_dashboard": {"percentage": 50},  # 50% rollout
    "billing": True,
    "teams": {"users": [1, 2, 3]}  # Specific users
})

# Setup routes
setup_auth_routes(rt, login_template=lambda **k: LoginForm(csrf_input=csrf_input, **k))
setup_admin_routes(rt, models=[User, Team, Subscription])
setup_team_routes(rt)
setup_billing_routes(rt, provider=StripeClient())

# Your custom routes using Launch Kit utilities
@rt("/")
def get(req, sess):
    if not sess.get('auth'):
        return RedirectResponse('/auth/login', status_code=303)
    
    user = sess['user']
    teams = Team.get_user_teams(user['id'])
    
    # Audit the access
    audit_log(user['id'], 'view', 'dashboard')
    
    # Check feature flag
    new_dash = flags.is_enabled('new_dashboard', user)
    
    return Title("Dashboard"), Main(
        H1(f"Welcome {user['name']}!"),
        TeamSelector(teams) if flags.is_enabled('teams', user) else None,
        DashboardV2(user) if new_dash else DashboardV1(user)
    )

@rt("/settings")
def get(req, sess):
    if not auth_required(req, sess):
        return RedirectResponse('/auth/login', status_code=303)
    
    user = sess['user']
    return Title("Settings"), Main(
        H1("Account Settings"),
        Tabs(
            Tab("Profile", ProfileForm(user)),
            Tab("Security", SecuritySettings(user)),
            Tab("Billing", BillingSettings(user)) if flags.is_enabled('billing') else None,
            Tab("API Keys", ApiKeyManager(user))
        )
    )

@rt("/admin/users")
def get(req, sess):
    if not require_role("admin", req, sess):
        return Title("Forbidden"), P("Access denied")
    
    users = User.all()
    return Title("User Management"), Main(
        H1("Users"),
        AdminTable(User, users, actions=True)
    )

# API endpoint with rate limiting
@rt("/api/data")
def get(req, sess, api_key: str = None):
    # Validate API key
    if not validate_api_key(api_key):
        return JSONResponse({"error": "Invalid API key"}, status_code=401)
    
    # Rate limit check
    if not rate_limiter.is_allowed(api_key):
        return JSONResponse({"error": "Rate limit exceeded"}, status_code=429)
    
    return JSONResponse({"data": "Your data here"})

# Run with uvicorn for production
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=5001)
```

## Key Differences from v1

1. **No App Wrapping**: Create a standard FastHTML app, add Launch Kit features
2. **Explicit Imports**: Import only what you need
3. **Transparent Utilities**: Every function is inspectable and customizable
4. **Composable Routes**: Pre-built routes you can mount, modify, or ignore
5. **Simple Decorators**: Clear permission decorators, not magic
6. **Override Everything**: Every component and utility can be replaced

## Testing Approach

```python
# Tests are simple and explicit
def test_password_hashing():
    password = "test123!"
    hashed = hash_password(password)
    assert verify_password(password, hashed)
    assert not verify_password("wrong", hashed)

def test_rate_limiter():
    limiter = RateLimiter(max_requests=5, window_seconds=60)
    for i in range(5):
        assert limiter.is_allowed("user1")
    assert not limiter.is_allowed("user1")  # 6th request blocked
```

## Additional SaaS Utilities

### 9. Search Functionality (13_search.ipynb)
```python
#| export
class SearchIndex:
    """Simple full-text search for FastHTML apps"""
    def __init__(self, db):
        self.db = db
        
    def index_record(self, table, record_id, text):
        """Add record to search index"""
        # Simple implementation - upgrade to MeiliSearch/Typesense for production
        
    def search(self, query, tables=None, limit=10):
        """Search across indexed content"""
        # Returns list of (table, id, snippet) tuples
```

### 10. Data Export (14_exports.ipynb)
```python
#| export
def export_to_csv(data, columns):
    """Export data to CSV format"""
    import csv
    from io import StringIO
    
    output = StringIO()
    writer = csv.DictWriter(output, fieldnames=columns)
    writer.writeheader()
    writer.writerows(data)
    return output.getvalue()

#| export
def create_export_response(data, filename, format='csv'):
    """Create file download response"""
    if format == 'csv':
        content = export_to_csv(data, data[0].keys())
        media_type = 'text/csv'
    elif format == 'json':
        content = json.dumps(data, default=str)
        media_type = 'application/json'
    
    return Response(
        content=content,
        media_type=media_type,
        headers={
            'Content-Disposition': f'attachment; filename="{filename}"'
        }
    )
```

## Summary

Launch Kit v2 follows the Answer.ai philosophy:
- **Simple**: Just utilities and components, no framework
- **Transparent**: You can read and understand every line
- **Composable**: Use what you need, ignore the rest
- **FastHTML-Native**: Uses FastHTML patterns (rt, fast_app, HTMX)
- **No Magic**: Everything is explicit and debuggable

Key improvements in v2:
- Fixed FastAPI patterns → proper FastHTML patterns
- Added HTMX integration throughout
- Added crucial SaaS features (audit logs, feature flags, search, exports)
- Proper session-based auth aligned with FastHTML
- CSRF protection that works with FastHTML forms
- Better examples showing real FastHTML usage

This approach gives developers the benefits of pre-built SaaS functionality while maintaining full control and understanding of their application.