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

This project uses MonsterUI as the default UI framework. When developing:

1. **Use MonsterUI components** instead of raw HTML:
   - `Button` instead of `<button>`
   - `Card` instead of `<div>` containers
   - `Form`, `FormField`, `Input` for forms
   - `DataTable` for tables
   - `Alert` for notifications
   - `Badge`, `Chip` for status indicators

2. **MonsterUI patterns**:
   - All components support HTMX attributes
   - Built-in dark mode support
   - Responsive by default
   - Accessible (WCAG compliant)
   
3. **Example**:
   ```python
   # Instead of:
   Div(H1("Title", cls="text-2xl"), Form(...))
   
   # Use:
   Card(
       PageHeader("Title"),
       Form(FormField(Label("Email"), Input(type="email")))
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

## Memories

- Use context7 to access FastHtml and MonsterUI documentation
- Use fastai coding style which can be found [here](https://docs.fast.ai/dev/style.html)
- Follow nbdev best practices from [here](https://nbdev.fast.ai/tutorials/best_practices.html)