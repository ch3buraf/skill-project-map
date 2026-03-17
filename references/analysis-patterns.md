
# Language-Specific Analysis Patterns

## Python

### Imports
```
import module_name
from package import thing
from . import sibling
from ..parent import thing
from package.sub.module import Class
```

### Key patterns
- `render_template('name.html')` → dependency on template file
- `open('path')` / `Path('path')` → file I/O dependency
- `@app.route('/path')` → defines API endpoint
- `db.session`, `cursor.execute` → database dependency
- `Blueprint('name', ...)` → Flask module boundary

### Exports to capture
- Top-level functions and classes
- `__all__` list if present
- Route decorators (`@app.route`, `@router.get`)

## JavaScript / TypeScript

### Imports
```
import x from './file'
import { x } from '../file'
import * as x from './file'
const x = require('./file')
require('./file.css')
```

### Key patterns
- `fetch('/api/...')` or `axios.get('/api/...')` → API dependency
- `<Component />` in JSX → component dependency  
- `Router` / `Route` definitions → routing structure
- `mongoose.model` / `prisma.` → database dependency
- `module.exports` / `export default` → defines what file provides

### Framework-specific
- **React**: `import Component from './Component'` + JSX usage
- **Next.js**: `pages/` directory = routes, `api/` = endpoints
- **Express**: `app.use()`, `router.get()` define routes

## Go

### Imports
```
import "project/internal/package"
import (
    "project/pkg/util"
)
```

### Key patterns
- Internal imports (same module path prefix) = project dependencies
- `http.HandleFunc` / `mux.Handle` → route definitions
- `sql.Open` / `db.Query` → database dependency
- Interface implementations (implicit) → check method signatures

## HTML / Templates

### Dependencies
```html
<link href="style.css">
<script src="app.js">
<img src="logo.png">
{% include 'partial.html' %}
{% extends 'base.html' %}
{{ url_for('static', filename='...') }}
```

### Key patterns
- Jinja2/Django: `{% extends %}`, `{% include %}`, `{% block %}`
- EJS: `<%- include('partial') %>`
- Handlebars: `{{> partial}}`
- Form `action` attributes → API endpoint dependency

## Docker / Infrastructure

### Dockerfile
```dockerfile
COPY src/ /app/src/        # depends on src/ directory
COPY requirements.txt .    # depends on requirements file
ENTRYPOINT ["./start.sh"]  # depends on entrypoint script
```

### docker-compose.yml
```yaml
build: ./service-dir       # depends on that directory's Dockerfile
volumes:
  - ./data:/data           # depends on data directory
depends_on:
  - postgres               # service dependency
```

### Makefile / Scripts
- Target dependencies: `build: compile link`
- File references: paths mentioned in commands
- Source/include directives

## Shell Scripts

### Dependencies
```bash
source ./config.sh
. ./env.sh
python app.py
docker-compose -f compose.yml up
```

## YAML / JSON Config

### Key patterns
- File path references
- Service names that match directory names
- Environment variable references that connect to .env files
- Schema references (`$ref`)

## Rust

### Dependencies  
```rust
mod module_name;           // imports src/module_name.rs or src/module_name/mod.rs
use crate::module::Thing;  // internal dependency
```

### Cargo.toml
- `[dependencies]` with `path = "../local-crate"` → internal dependency

## Java / Kotlin

### Imports
```java
import com.project.package.Class;
```
- Match package path to directory structure
- `@Autowired`, `@Inject` → dependency injection connections

## C / C++

### Includes
```c
#include "local_header.h"    // project file
#include <system_header.h>   // external - skip
```
- Header files define interfaces between modules
- `.c` / `.cpp` implementing a `.h` → strong coupling