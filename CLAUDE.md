# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Odoo 19.0 — an open-source modular ERP/CRM platform written in Python. The codebase consists of a core framework (`odoo/`) and ~200+ built-in addon modules (`odoo/addons/`), plus custom/tutorial addons in `addons/`.

**Requirements:** Python 3.10–3.13, PostgreSQL >= 13.
**Database:** `odoo_db` (pre-existing local PostgreSQL database).

## Common Commands

### Running the Server
```bash
python odoo-bin -c odoo.conf
# or explicitly:
python odoo-bin --addons-path=odoo/addons,addons -d odoo_db
```

### Installing/Updating Modules
```bash
python odoo-bin -d odoo_db -i <module_name>          # install
python odoo-bin -d odoo_db -u <module_name>           # update
python odoo-bin -d odoo_db -i module1,module2         # multiple modules
```

### Running Tests
```bash
# Test a specific module
python odoo-bin -d odoo_db -i <module_name> --test-enable --stop-after-init

# Using the test subcommand
python odoo-bin test <module_name>
```

### Linting
```bash
ruff check .                    # lint entire codebase
ruff check addons/my_module/    # lint a specific module
```

### Interactive Shell
```bash
python odoo-bin shell -d odoo_db
```

### Scaffold a New Module
```bash
python odoo-bin scaffold <module_name> <destination_path>
```

### Database Management
```bash
python odoo-bin db create <dbname>
python odoo-bin db drop <dbname>
```

## Architecture

### Core Framework (`odoo/`)

- **`orm/`** — Object-Relational Mapping: model definitions (`models.py`), field types (`fields_*.py`), domain expressions (`domains.py`), environment management (`environments.py`), registry (`registry.py`)
- **`http.py`** — WSGI application, HTTP routing, request/response handling, controller dispatch
- **`sql_db.py`** — Database connection pooling and cursor management
- **`api/`** — Decorators: `@api.depends`, `@api.onchange`, `@api.constrains`, `@api.model`, `@api.model_create_multi`, etc.
- **`fields/`** — Field type implementations (Char, Integer, Many2one, One2many, Many2many, etc.)
- **`modules/`** — Module discovery, loading, and registry management
- **`cli/`** — CLI subcommands (server, db, shell, scaffold, test, i18n, etc.)
- **`tools/`** — Utilities (config, safe_eval, image processing, date utils, mail, SQL helpers, profiler)
- **`service/`** — Server lifecycle, database operations, security
- **`tests/`** — Test framework: base classes, form helpers, test discovery/runner

### Addon Module Structure

Every addon follows this convention:
```
module_name/
├── __manifest__.py          # Required: metadata (name, depends, data, etc.)
├── __init__.py              # Imports models/controllers
├── models/                  # ORM model definitions
├── views/                   # XML view definitions (form, list, kanban, etc.)
├── security/
│   └── ir.model.access.csv  # Access control rules
├── data/                    # Default data (XML/CSV)
├── static/                  # Web assets (JS, CSS, images)
├── controllers/             # HTTP route handlers
└── tests/                   # unittest-based tests
```

The `__manifest__.py` must declare `depends` (list of module dependencies) and `data` (list of XML/CSV files to load).

### Model Inheritance

Odoo uses three inheritance mechanisms:
- **Classical inheritance** (`_inherit = 'parent.model'` with new `_name`): creates a new model copying the parent
- **Extension inheritance** (`_inherit = 'existing.model'` without new `_name`): extends an existing model in-place
- **Delegation inheritance** (`_inherits = {'parent.model': 'field_id'}`): links to parent via foreign key

### Test Base Classes

- `BaseCase` — no database, simple assertions
- `TransactionCase` — each test method runs in a rolled-back transaction
- `HttpCase` — HTTP request testing with `url_open()`
- `TransactionalHttpCase` — combined HTTP + transaction testing

## Linting Rules

Ruff is configured in `ruff.toml` (target: Python 3.10). Key points:
- Import order follows Odoo convention: stdlib → third-party → `odoo` (first-party) → `odoo.addons` (local-folder)
- Line length is **not** enforced (E501 ignored)
- Unused imports allowed in `__init__.py` files (F401 ignored)
- Printf-style formatting is allowed (UP031 ignored)

## Configuration

- **`odoo.conf`** — Runtime config: database connection (`odoo_db` on localhost:5432), addons_path, http_interface (127.0.0.1). The `addons_path` must include both `odoo/addons` (built-in) and any custom addon directories.
- **`setup.py`** / **`requirements.txt`** — Package definition and pinned dependencies.

## Key Conventions

- Models use dotted names (`estate.property`) which map to underscored table names (`estate_property`)
- XML IDs follow the pattern `module_name.xml_id_name`
- Security access rules are CSV files with columns: id, name, model_id/id, group_id/id, perm_read, perm_write, perm_create, perm_unlink
- Computed fields require `@api.depends(...)` decorator listing all dependency fields
- `self` in model methods is always a recordset (potentially multiple records); iterate with `for record in self:`
