# 🗺️ Project Map Skill

**Generate interactive code maps for any software project — Markdown for AI, HTML for humans.**

[English](#english) · [Русский](#русский)

---

<a id="english"></a>

## English

### What is this?

A Claude skill that analyzes your codebase and generates two outputs:

| Output | Format | For whom | Purpose |
|--------|--------|----------|---------|
| `PROJECT_MAP.md` | Markdown | AI / LLMs | Architecture context, file change guide, dependency graph |
| `PROJECT_MAP.html` | Standalone HTML | Humans | Interactive visual graph with drag, zoom, filtering |

### The Problem

You open an unfamiliar project. Dozens of files, unclear connections. You need to make a change but don't know which files to touch or what will break.

An AI assistant faces the same problem — it needs to understand project structure before making accurate edits.

This skill solves both problems at once.

### Features

**Analysis:**
- Reads all source files and extracts real dependencies (imports, renders, API calls, configs)
- Groups files into logical modules automatically
- Supports Python, JavaScript/TypeScript, Go, Java, Rust, C/C++, HTML, Docker, Shell, YAML, and more

**Markdown output (`PROJECT_MAP.md`):**
- Architecture overview
- Module breakdown with file roles and exports
- Full dependency graph (what imports what)
- **File Change Guide** — "to change X, modify these files because Y"
- Quick reference (entry point, config, database, tests)

**HTML output (`PROJECT_MAP.html`):**
- Canvas-based force-directed graph (not SVG — handles 100+ nodes)
- Nodes never overlap (collision detection + strong linear repulsion)
- Drag nodes, zoom with scroll wheel, pan by dragging background
- Hover highlights connected subgraph, fades everything else
- Filter by modules via sidebar checkboxes
- Search modules by name
- File table + dependency list tabs
- Color coding: blue = modules, orange = files, green = shared files, red dashed = dependencies
- Fully standalone — single HTML file, no internet required, works offline

### Installation

#### Claude Code (per-project)

```bash
mkdir -p .claude/skills
# Copy the project-map/ folder into .claude/skills/
cp -r project-map/ .claude/skills/project-map/
```

#### Claude Code (global)

```bash
mkdir -p ~/.claude/skills
cp -r project-map/ ~/.claude/skills/project-map/
```

#### File structure

```
project-map/
├── SKILL.md                        # Main skill instructions
├── README.md                       # This file
├── evals/
│   └── evals.json                  # Test cases for skill evaluation
├── references/
│   ├── analysis-patterns.md        # Language-specific extraction patterns
│   └── dependency-extractors.md    # Dependency resolution algorithms
└── assets/
    └── map-template.html           # Interactive graph template
```

### Usage

Just ask Claude in natural language:

```
"Create a code map of this project"
"Map out the architecture"
"Show me how files connect to each other"
"I need to understand this codebase before making changes"
"What files do I need to modify to change the authentication?"
"Draw the dependency graph"
```

The skill triggers automatically on these kinds of requests.

### How It Works

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│  Read all    │────▶│  Extract     │────▶│  Group into      │
│  source files│     │  imports &   │     │  modules         │
│              │     │  dependencies│     │  (3-8 groups)    │
└─────────────┘     └──────────────┘     └──────────────────┘
                                                  │
                    ┌──────────────┐     ┌────────▼─────────┐
                    │  Inject data │◀────│  Map all         │
                    │  into HTML   │     │  dependencies    │
                    │  template    │     │  with labels     │
                    └──────┬───────┘     └──────────────────┘
                           │
              ┌────────────┼────────────┐
              ▼                         ▼
    ┌──────────────────┐    ┌──────────────────┐
    │  PROJECT_MAP.md  │    │  PROJECT_MAP.html │
    │  (for AI)        │    │  (for humans)     │
    └──────────────────┘    └──────────────────┘
```

**Step 1: Analyze** — Claude reads every file and extracts what it imports, exports, renders, calls, and connects to.

**Step 2: Group** — Files are grouped into logical modules based on directory structure, naming, and dependency clusters.

**Step 3: Map** — Every file-to-file connection is recorded with a label: `imports`, `renders`, `calls`, `builds`, `reads`, `configures`, etc.

**Step 4: Generate** — Two outputs are created. The Markdown is structured for AI consumption (especially the File Change Guide). The HTML injects the data into a pre-built interactive template.

### Graph Physics

The HTML visualization uses a custom force-directed layout engine designed to prevent the most common problem with graph visualizations — **overlapping nodes**.

| Parameter | Value | Why |
|-----------|-------|-----|
| Repulsion force | `F = 3000/d` (linear) | Inverse-linear is much stronger at close range than inverse-square |
| Collision boundary | `r1 + r2 + 35px` | Hard minimum distance, enforced every frame |
| Alpha decay | `×0.9975` | ~2000 frames before settling — nodes have time to spread |
| Velocity damping | `×0.68` | Enough momentum to escape local minima, not so much it oscillates |
| Speed cap | `25 px/frame` | Prevents explosive launches from high-force situations |
| Link rest length | `130px` (link) / `200px` (dep) | Dependencies are longer to visually separate layers |
| Center gravity | `0.004 × alpha` | Gentle pull to prevent graph from drifting off-screen |

### Dependency Types Detected

| Type | Example | Edge Label |
|------|---------|------------|
| Import | `from app import db` | `imports` |
| Template render | `render_template('index.html')` | `renders` |
| API call | `fetch('/api/users')` → route handler | `calls` |
| Config reference | `docker-compose.yml` → `Dockerfile` | `builds` |
| Data flow | Model writes DB ← Query reads DB | `reads` / `writes` |
| Build chain | `Makefile` → source files | `builds` |
| Script execution | `entrypoint.sh` → `python app.py` | `executes` |

### Evaluation

Test cases are in `evals/evals.json`. Run them through the skill-creator evaluation loop:

```
"Run the project-map skill evals"
```

### Supported Languages

Python · JavaScript · TypeScript · JSX · TSX · Go · Java · Kotlin · Rust · C · C++ · HTML · CSS · Shell · Dockerfile · docker-compose · Makefile · YAML · JSON · TOML · XML · SQL · GraphQL · Protocol Buffers · Vue · Ruby · PHP

### Tips

- **Large projects**: The skill works best with up to ~100 files. For monorepos, point it at a specific service/package.
- **Accuracy**: The map is only as good as the analysis. If dependencies look wrong, ask Claude to re-read specific files.
- **Updating**: Re-run the skill after significant refactoring to keep the map current.
- **AI context**: Include `PROJECT_MAP.md` in your Claude conversation context for better-targeted edits.

---

<a id="русский"></a>

## Русский

### Что это?

Скилл для Claude, который анализирует кодовую базу и генерирует два выхода:

| Выход | Формат | Для кого | Назначение |
|-------|--------|----------|------------|
| `PROJECT_MAP.md` | Markdown | AI / LLM | Архитектурный контекст, гид по изменениям, граф зависимостей |
| `PROJECT_MAP.html` | HTML-страница | Люди | Интерактивный визуальный граф с перетаскиванием, зумом, фильтрацией |

### Проблема

Вы открываете незнакомый проект. Десятки файлов, непонятные связи. Нужно внести изменение, но непонятно какие файлы трогать и что сломается.

AI-ассистент сталкивается с той же проблемой — ему нужно понять структуру проекта прежде чем делать точные правки.

Этот скилл решает обе проблемы одновременно.

### Возможности

**Анализ:**
- Читает все исходные файлы и извлекает реальные зависимости (импорты, рендеры, API-вызовы, конфиги)
- Автоматически группирует файлы в логические модули
- Поддерживает Python, JavaScript/TypeScript, Go, Java, Rust, C/C++, HTML, Docker, Shell, YAML и другие

**Markdown-выход (`PROJECT_MAP.md`):**
- Обзор архитектуры
- Разбивка по модулям с ролями файлов и экспортами
- Полный граф зависимостей (что импортирует что)
- **Гид по изменениям** — «чтобы изменить X, правьте эти файлы, потому что Y»
- Быстрая справка (точка входа, конфиг, БД, тесты)

**HTML-выход (`PROJECT_MAP.html`):**
- Граф на Canvas с силовой раскладкой (не SVG — держит 100+ узлов)
- Узлы никогда не накладываются (детекция коллизий + сильное линейное отталкивание)
- Перетаскивание узлов, зум колесом мыши, панорамирование фона
- При наведении подсвечивается связанный подграф, остальное затухает
- Фильтрация по модулям через чекбоксы в боковой панели
- Поиск модулей по названию
- Вкладки: таблица файлов + список зависимостей
- Цветовая кодировка: синий = модули, оранжевый = файлы, зелёный = общие файлы, красный пунктир = зависимости
- Полностью автономный — один HTML-файл, интернет не нужен, работает офлайн

### Установка

#### Claude Code (для проекта)

```bash
mkdir -p .claude/skills
# Скопируйте папку project-map/ в .claude/skills/
cp -r project-map/ .claude/skills/project-map/
```

#### Claude Code (глобально)

```bash
mkdir -p ~/.claude/skills
cp -r project-map/ ~/.claude/skills/project-map/
```

#### Структура файлов

```
project-map/
├── SKILL.md                        # Основные инструкции скилла
├── README.md                       # Этот файл
├── evals/
│   └── evals.json                  # Тест-кейсы для проверки скилла
├── references/
│   ├── analysis-patterns.md        # Паттерны извлечения по языкам
│   └── dependency-extractors.md    # Алгоритмы разрешения зависимостей
└── assets/
    └── map-template.html           # Шаблон интерактивного графа
```

### Использование

Просто попросите Claude на естественном языке:

```
"Создай карту кода этого проекта"
"Покажи архитектуру"
"Как файлы связаны между собой?"
"Мне нужно разобраться в этой кодовой базе перед изменениями"
"Какие файлы нужно править чтобы изменить авторизацию?"
"Нарисуй граф зависимостей"
```

Скилл срабатывает автоматически на подобные запросы.

### Как это работает

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│  Прочитать   │────▶│  Извлечь     │────▶│  Сгруппировать   │
│  все файлы   │     │  импорты и   │     │  в модули        │
│  проекта     │     │  зависимости │     │  (3-8 групп)     │
└──────────────┘     └──────────────┘     └──────────────────┘
                                                   │
                     ┌──────────────┐     ┌────────▼─────────┐
                     │  Подставить  │◀────│  Составить карту  │
                     │  данные в    │     │  всех зависимостей│
                     │  HTML-шаблон │     │  с метками        │
                     └──────┬───────┘     └──────────────────┘
                            │
               ┌────────────┼────────────┐
               ▼                         ▼
     ┌──────────────────┐    ┌──────────────────┐
     │  PROJECT_MAP.md  │    │  PROJECT_MAP.html │
     │  (для AI)        │    │  (для людей)      │
     └──────────────────┘    └──────────────────┘
```

**Шаг 1: Анализ** — Claude читает каждый файл и извлекает что он импортирует, экспортирует, рендерит, вызывает и к чему подключается.

**Шаг 2: Группировка** — Файлы объединяются в логические модули на основе структуры директорий, именования и кластеров зависимостей.

**Шаг 3: Картирование** — Каждая связь файл-файл записывается с меткой: `imports`, `renders`, `calls`, `builds`, `reads`, `configures` и т.д.

**Шаг 4: Генерация** — Создаются два выхода. Markdown структурирован для AI (особенно Гид по изменениям). HTML подставляет данные в готовый интерактивный шаблон.

### Физика графа

HTML-визуализация использует собственный движок силовой раскладки, спроектированный чтобы предотвратить главную проблему графовых визуализаций — **наложение узлов**.

| Параметр | Значение | Зачем |
|----------|----------|-------|
| Сила отталкивания | `F = 3000/d` (линейная) | Обратно-линейная гораздо сильнее на малых расстояниях чем обратно-квадратичная |
| Граница коллизии | `r1 + r2 + 35px` | Жёсткое минимальное расстояние, проверяется каждый кадр |
| Затухание альфы | `×0.9975` | ~2000 кадров до стабилизации — узлы успевают разойтись |
| Гашение скорости | `×0.68` | Достаточно инерции чтобы выйти из локальных минимумов, не так много чтобы осциллировать |
| Ограничение скорости | `25 px/кадр` | Предотвращает взрывные запуски при высоких силах |
| Длина покоя связи | `130px` (связь) / `200px` (зависимость) | Зависимости длиннее для визуального разделения слоёв |
| Гравитация к центру | `0.004 × alpha` | Мягкое притяжение чтобы граф не уплывал за экран |

### Типы обнаруживаемых зависимостей

| Тип | Пример | Метка ребра |
|-----|--------|-------------|
| Импорт | `from app import db` | `imports` |
| Рендер шаблона | `render_template('index.html')` | `renders` |
| API-вызов | `fetch('/api/users')` → обработчик маршрута | `calls` |
| Ссылка конфига | `docker-compose.yml` → `Dockerfile` | `builds` |
| Поток данных | Модель пишет в БД ← Запрос читает из БД | `reads` / `writes` |
| Цепочка сборки | `Makefile` → исходные файлы | `builds` |
| Запуск скрипта | `entrypoint.sh` → `python app.py` | `executes` |

### Оценка

Тест-кейсы находятся в `evals/evals.json`. Запустите их через цикл оценки skill-creator:

```
"Запусти тесты скилла project-map"
```

### Поддерживаемые языки

Python · JavaScript · TypeScript · JSX · TSX · Go · Java · Kotlin · Rust · C · C++ · HTML · CSS · Shell · Dockerfile · docker-compose · Makefile · YAML · JSON · TOML · XML · SQL · GraphQL · Protocol Buffers · Vue · Ruby · PHP

### Советы

- **Большие проекты**: Скилл лучше всего работает с проектами до ~100 файлов. Для монорепо укажите конкретный сервис/пакет.
- **Точность**: Карта настолько хороша, насколько хорош анализ. Если зависимости выглядят неправильно, попросите Claude перечитать конкретные файлы.
- **Обновление**: Перезапускайте скилл после значительного рефакторинга чтобы карта была актуальной.
- **AI-контекст**: Включите `PROJECT_MAP.md` в контекст разговора с Claude для более точных правок.

---

## License

MIT

## Contributing

Issues and PRs welcome. Key areas for improvement:
- Additional language support
- Smarter module grouping heuristics
- Performance optimization for large codebases (500+ files)
- Export graph as PNG/SVG
