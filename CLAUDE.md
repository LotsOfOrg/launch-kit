# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a FastHTML toolkit called "launch-kit" for building production-ready SaaS applications with authentication, admin panels, billing, and team management. The project uses nbdev for development.

## Development Commands

### Setting up the development environment
```bash
# Install in development mode
pip install -e .
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
   - `monsterui`: UI components library

3. The minimum Python version is 3.9

## Important Files

- `settings.ini`: Contains all nbdev configuration including library name, version, dependencies, and documentation settings
- `nbs/index.ipynb`: The main documentation notebook that becomes README.md
- `nbs/00_core.ipynb`: Core module implementation