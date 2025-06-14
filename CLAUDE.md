# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a FastHTML toolkit called "launch-kit" for building production-ready SaaS applications with authentication, admin panels, billing, and team management. The project uses nbdev for development and MonsterUI as the default UI framework for beautiful, accessible components.

## Development Commands

### Setting up the development environment
```bash
# Install in development mode
pip install -e .
# Use uv for python environments eg. uv pip install -e .
```

### Core nbdev workflow
```bash
# After making changes to notebooks in nbs/ directory
nbdev_prepare  # Compiles notebooks to Python modules and prepares for commit
```

### Building and Publishing
```bash
# Build documentation
nbdev_docs

# Run tests (if any exist)
nbdev_test

# Update library code from notebooks
nbdev_export

# Clean notebooks
nbdev_clean
```

## Project Structure

- **nbs/**: Contains Jupyter notebooks that are the source of truth for the code
  - Notebooks are compiled to Python modules in the `launch_kit/` directory
  - Changes should be made in notebooks, not directly in Python files
- **launch_kit/**: Auto-generated Python modules from notebooks (DO NOT EDIT DIRECTLY)
- **_docs/**: Generated documentation output
- **settings.ini**: nbdev configuration file containing project metadata and settings

## Key Architecture Notes

1. This project uses nbdev, which means:
   - All development happens in Jupyter notebooks under `nbs/`
   - Python modules in `launch_kit/` are auto-generated
   - Never edit files in `launch_kit/` directly - they will be overwritten
   - Use `nbdev_prepare` after making changes to notebooks

2. The project dependencies include:
   - `python-fasthtml`: The FastHTML framework
   - `monsterui>=1.0.20`: MonsterUI - the default UI framework for all components

3. The minimum Python version is 3.10

## Important Files

- `settings.ini`: Contains all nbdev configuration including library name, version, dependencies, and documentation settings
- `nbs/index.ipynb`: The main documentation notebook that becomes README.md
- `nbs/00_core.ipynb`: Core module implementation

## MonsterUI Integration

This project uses MonsterUI as the default UI framework. MonsterUI provides a comprehensive set of pre-styled components that work seamlessly with FastHTML and HTMX.

### Core Principles
- **Minimal code**: Pre-styled components reduce boilerplate
- **Semantic HTML**: Components generate accessible markup
- **HTMX-ready**: All components support HTMX attributes
- **Responsive**: Mobile-first design with Tailwind integration
- **Theme support**: Built-in dark/light mode and color themes

### Component Categories

#### 1. **Form Components**
```python
# Basic form inputs
LabelInput("Name", id="name", placeholder="Enter name")
LabelInput("Email", id="email", type="email", required=True)

# Dropdown with search
Select(*Options("Option 1", "Option 2"), 
       name='choice', searchable=True, placeholder='Choose...')

# Range slider
LabelRange(label='Volume', value='50', min='0', max='100')

# Complete form example
Form(
    LabelInput("Email", id="email", type="email"),
    LabelInput("Password", id="password", type="password"),
    Checkbox("Remember me", id="remember"),
    Button("Sign In", cls=ButtonT.primary),
    hx_post="/login"
)
```

#### 2. **Layout Components**
```python
# Page structure
Container(
    Main(
        H1("Page Title"),
        Grid(
            Card("Content 1"),
            Card("Content 2"),
            Card("Content 3"),
            cols_lg=3, cols_md=2  # Responsive columns
        )
    )
)

# Card layouts
Card(
    H2("Card Title"),
    P("Card content"),
    footer=Div(Button("Action"))
)

# Stacking utilities
DivVStacked(*items)  # Vertical stack
DivHStacked(*items)  # Horizontal stack
DivCentered(content)  # Centered content
```

#### 3. **Navigation Components**
```python
# Responsive navbar (hamburger on mobile)
Navbar(
    NavContainer(
        A("Home", href="/"),
        A("About", href="/about"),
        DropDownNavContainer(
            "More",
            A("Settings", href="/settings"),
            A("Logout", href="/logout")
        )
    )
)

# Multi-step process
Steps(
    Step("Account", completed=True),
    Step("Profile", active=True),
    Step("Payment")
)

# Collapsible sections
Accordion(
    AccordionItem("Section 1", P("Content 1")),
    AccordionItem("Section 2", P("Content 2"))
)
```

#### 4. **Data Display Components**
```python
# Basic table
Table(
    Thead(Tr(Th("Name"), Th("Email"), Th("Status"))),
    Tbody(*[Tr(Td(u.name), Td(u.email), Td(Badge(u.status))) 
           for u in users])
)

# User avatars
DiceBearAvatar(name="John Doe", h=32, w=32)

# Status indicators
Badge("Active", cls="badge-success")
Chip("Tag", removable=True)
```

#### 5. **Feedback Components**
```python
# Modal dialogs
Modal(
    ModalTitle("Confirm Action"),
    P("Are you sure you want to proceed?"),
    ModalCloseButton("Cancel"),
    Button("Confirm", cls=ButtonT.primary, hx_post="/confirm")
)

# Toast notifications
Toast('Operation successful!', dur=3.0)  # Auto-dismiss after 3 seconds

# Alert messages
Alert("Important information", cls="alert-info")

# Loading states
Loading()  # Spinner
```

#### 6. **Interactive Components**
```python
# Buttons with variants
Button("Primary", cls=ButtonT.primary)
Button("Secondary", cls=ButtonT.secondary)
Button("Danger", cls="text-destructive")

# Icons
UkIcon("home", height=20)
UkIconLink('github', href='https://github.com/user/repo')

# Lightbox for images
Lightbox(
    Img(src="/image1.jpg"),
    Img(src="/image2.jpg")
)
```

### Theme Configuration
```python
from monsterui.all import *

# Available themes: blue, green, red, slate, stone, gray, zinc, amber
# Enable features: highlightjs for code, katex for math
hdrs = Theme.blue.headers(highlightjs=True, katex=True)
app, rt = fast_app(hdrs=hdrs)

# Theme customization in component
Card(content, cls="theme-green")  # Override theme locally
```

### Common SaaS Patterns

#### Authentication Pages
```python
@rt("/login")
def login():
    return Titled("Sign In",
        Container(
            DivCentered(
                Card(
                    Form(
                        H2("Welcome Back"),
                        LabelInput("Email", id="email", type="email"),
                        LabelInput("Password", id="password", type="password"),
                        DivHStacked(
                            Checkbox("Remember me"),
                            A("Forgot password?", href="/reset", 
                              cls=TextPresets.muted_sm)
                        ),
                        Button("Sign In", cls=ButtonT.primary + " w-full"),
                        hx_post="/login"
                    ),
                    cls="max-w-md"
                )
            )
        )
    )
```

#### Dashboard Layout
```python
@rt("/dashboard")
def dashboard():
    return Container(
        Grid(
            Card(
                H3("Total Users"),
                P("1,234", cls="text-3xl font-bold"),
                footer=P("+12% from last month", cls=TextPresets.muted_sm)
            ),
            Card(
                H3("Revenue"),
                P("$45,678", cls="text-3xl font-bold"),
                footer=P("+8% from last month", cls=TextPresets.muted_sm)
            ),
            Card(
                H3("Active Sessions"),
                P("89", cls="text-3xl font-bold"),
                footer=P("Live now", cls=TextPresets.muted_sm)
            ),
            cols_lg=3
        )
    )
```

#### Data Tables with Actions
```python
def user_table(users):
    return Table(
        Thead(
            Tr(Th("User"), Th("Status"), Th("Actions", cls="text-right"))
        ),
        Tbody(*[
            Tr(
                Td(
                    DivHStacked(
                        DiceBearAvatar(u.name, h=32, w=32),
                        Div(
                            P(u.name),
                            P(u.email, cls=TextPresets.muted_sm)
                        )
                    )
                ),
                Td(Badge("Active" if u.active else "Inactive",
                         cls="badge-success" if u.active else "badge-gray")),
                Td(
                    DivRAligned(
                        UkIconLink("edit", href=f"/users/{u.id}/edit"),
                        UkIconLink("trash", hx_delete=f"/users/{u.id}",
                                  hx_confirm="Delete this user?")
                    )
                )
            ) for u in users
        ])
    )
```

### Best Practices

1. **Component Selection**: Always prefer MonsterUI components over raw HTML elements
2. **Responsive Design**: Use Grid with responsive column props (cols_lg, cols_md, cols_sm)
3. **Consistent Spacing**: Use layout utilities (DivVStacked, DivHStacked) for consistent spacing
4. **Theme Consistency**: Use ButtonT variants and TextPresets for consistent styling
5. **Accessibility**: MonsterUI components include proper ARIA attributes by default
6. **HTMX Integration**: Add HTMX attributes directly to MonsterUI components
7. **Form Handling**: Use Form component with proper field grouping for better UX

### Example: Complete CRUD Interface
```python
@rt("/items")
def items_list():
    items = get_items()
    return Titled("Items",
        Container(
            DivHStacked(
                H1("Items"),
                Button("New Item", href="/items/new", cls=ButtonT.primary)
            ),
            Card(
                Table(
                    Thead(Tr(Th("Name"), Th("Status"), Th(""))),
                    Tbody(*[
                        Tr(
                            Td(item.name),
                            Td(Badge(item.status)),
                            Td(
                                DivRAligned(
                                    Button("Edit", cls="btn-sm", 
                                          href=f"/items/{item.id}/edit"),
                                    Button("Delete", cls="btn-sm text-destructive",
                                          hx_delete=f"/items/{item.id}",
                                          hx_confirm="Delete this item?",
                                          hx_target="closest tr",
                                          hx_swap="outerHTML")
                                )
                            )
                        ) for item in items
                    ]),
                    id="items-table"
                )
            )
        )
    )
```

## Repository Information

- GitHub Repository: https://github.com/LotsOfOrg/launch-kit

### MCP Servers
```bash
# Add Puppeteer MCP server for browser automation (web scraping, testing, screenshots)
claude mcp add puppeteer npx @modelcontextprotocol/server-puppeteer
# Add context7 MCP server for getting up to date documentation
claude mcp add --transport sse context7 https://mcp.context7.com/sse
```

## nbdev Best Practices

When working with notebooks in this project, follow these best practices:

### Notebook Organization
- **Start with clear titles**: Each notebook should have a descriptive title and subtitle
- **Use H2 sections** to group related functionality (e.g., "Authentication", "User Management")
- **Use H4 sections** to break up long explanations within a module
- **Keep code cells short**: Demonstrate functionality immediately after defining it

### Documentation & Code Style
- **Mix code, tests, and docs**: Write documentation, code, and tests together in context
- **Use type annotations**: Add type hints to all function parameters and return values
- **Keep docstrings concise**: Put brief descriptions in docstrings, elaborate in markdown cells
- **Show, don't tell**: Use executable code examples instead of static descriptions
- **Include rich examples**: Add plots, diagrams, and visual outputs where helpful

### Testing Approach
- **Turn examples into tests**: Add assertions to your code examples
- **Document edge cases**: Show error handling and edge cases as executable tests
- **Use `fastcore.test`**: Leverage lightweight test utilities like `test_eq`, `test_ne`, etc.
- **Test in context**: Write tests immediately after the code they test

### Code Examples
```python
# Good: Executable example with assertion
def greet(name: str) -> str:
    "Create a greeting message"
    return f"Hello, {name}!"

# Demonstrate usage
message = greet("World")
test_eq(message, "Hello, World!")  # This serves as both example and test
```

### Notebook Types (Diátaxis Framework)
- **Tutorials**: Learning-oriented, step-by-step guides for newcomers
- **How-to guides**: Problem-oriented, practical recipes for specific tasks
- **Explanations**: Understanding-oriented, conceptual discussions
- **References**: Information-oriented, technical descriptions

### Workflow Tips
- **Use `@patch`**: For adding methods to existing classes across notebooks
- **Add rich representations**: Implement `_repr_markdown_` or `__repr__` for custom classes
- **Link related content**: Use doclinks to reference other parts of the codebase
- **Export selectively**: Use `#| export` to control what gets exported to Python modules

## FastHTML Development Best Practices

When developing FastHTML applications in notebooks:

### Application Structure
- **Use `fast_app()`**: Initialize apps with sensible defaults for rapid development
- **Leverage wildcard imports**: Use `from fasthtml.common import *` for concise code
- **Organize with decorators**: Use `@rt()` decorator for clean route definitions
- **Function-based components**: Prefer functions over classes for component definitions

### Component Development
- **Create reusable components**: Define components as functions or dataclasses
- **Custom rendering**: Implement `__ft__()` method for custom component rendering
- **Raw HTML handling**: Use `NotStr()` to safely include raw HTML content
- **Type validation**: Use Python type hints for component parameter validation

### Routing and Request Handling
- **Match function names**: Name functions according to HTTP methods (get, post, etc.)
- **Type-annotated routes**: Use type annotations for URL variable validation
- **Middleware patterns**: Implement beforeware for authentication/authorization
- **Async operations**: Use async functions for non-blocking I/O operations

### State Management
- **Session handling**: Use session dictionaries for lightweight state tracking
- **Persistent data**: Implement cookies for client-side persistence
- **User notifications**: Leverage toasts for user feedback
- **Real-time updates**: Use SSE and WebSockets for live data

### HTMX Integration
- **Dynamic loading**: Use `hx_` attributes for partial page updates
- **WebSocket support**: Implement `ws_send` and `ws_connect` for real-time features
- **Progressive enhancement**: Start with server-rendered HTML, enhance with HTMX
- **Extension usage**: Leverage HTMX extensions for advanced interactivity

### Testing in Notebooks
- **Route testing**: Use Starlette's `TestClient` for testing endpoints
- **Component inspection**: Print rendered HTML to validate output
- **Form validation**: Test form submissions and data transformations
- **Event loop awareness**: Be mindful of ASGI event loop limitations in notebooks

### Notebook-Specific Tips
```python
# Testing views in notebooks
from starlette.testclient import TestClient
client = TestClient(app)

# Test GET requests
response = client.get("/")
print(response.text)

# Test POST requests
response = client.post("/submit", data={"field": "value"})
```

### Code Example Patterns
```python
# Component with MonsterUI and FastHTML
def TodoItem(todo: Todo) -> FT:
    "Render a todo item with MonsterUI components"
    return Card(
        Div(
            Checkbox(checked=todo.done, 
                    hx_patch=f"/todos/{todo.id}",
                    hx_swap="outerHTML"),
            todo.title,
            cls="flex items-center gap-2"
        ),
        id=f"todo-{todo.id}"
    )

# Route with type validation
@rt("/todos/{id:int}")
async def get(id: int):
    todo = await get_todo(id)
    return TodoItem(todo)
```

## Memories

- Use context7 to access FastHtml and MonsterUI documentation
- Use fastai coding style which can be found [here](https://docs.fast.ai/dev/style.html)
- Follow nbdev best practices from [here](https://nbdev.fast.ai/tutorials/best_practices.html)