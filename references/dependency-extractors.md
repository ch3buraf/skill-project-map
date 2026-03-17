
# Dependency Extraction Algorithms

## Core Algorithm

For each source file in the project:

1. Read the file content
2. Identify the language/type from extension
3. Apply language-specific regex patterns to find:
   - Direct imports of other project files
   - String literals that match other project file paths
   - Framework-specific connection patterns
4. Resolve relative paths to absolute project paths
5. Verify the target file exists in the project
6. Record: `{from, to, label}` where label is the dependency type

## Path Resolution

When a file at `src/api/routes.py` imports `from ..models import User`:

1. Start at `src/api/`
2. `..` → go up to `src/`
3. `models` → `src/models.py` or `src/models/__init__.py`
4. Check which exists in the project

For JS/TS without extensions:
- Try `.js`, `.ts`, `.tsx`, `.jsx`
- Try `/index.js`, `/index.ts`
- Try exact match

## Implicit Dependencies

Some dependencies aren't in import statements:

### Template rendering
```python
# In routes.py
render_template('dashboard.html')
```
→ Dependency: `routes.py` → `templates/dashboard.html` (renders)

### API consumption
```javascript
// In frontend/api.js  
fetch('/api/users')
```
```python
# In backend/routes.py
@app.route('/api/users')
```
→ Dependency: `frontend/api.js` → `backend/routes.py` (calls)

Match by URL path — find route definitions and their consumers.

### Database tables
```python
# models.py
class User(db.Model):
    __tablename__ = 'users'

# queries.py  
db.session.query(User)
```
→ Dependency: `queries.py` → `models.py` (queries User model)

### Config files
```python
# app.py
load_dotenv('.env')
config = yaml.load('config.yml')
```
→ Dependency: `app.py` → `.env` (reads config)
→ Dependency: `app.py` → `config.yml` (reads config)

## Dependency Weight

For graph visualization, assign weights to control edge emphasis:

| Type | Weight | Visual |
|------|--------|--------|
| Direct import | 3 | Thick solid line |
| Template render | 2 | Medium solid line |
| API call | 2 | Medium dashed line |
| Config read | 1 | Thin dotted line |
| Build reference | 1 | Thin dashed line |
| Data flow (DB) | 2 | Medium dotted line |