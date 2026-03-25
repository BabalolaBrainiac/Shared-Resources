# Python + Flask + asyncio Masterclass
## For TypeScript Developers — 24-Hour Interview Prep Edition
## Earlybird AI | Boris Bambo (CTO)

Every concept is explained relative to TypeScript. Every example uses the Earlybird domain.
Boris specifically mentioned asyncio — that gets the most depth.
By the end you build a production-ready Case Notes API in Python/Flask.

---

# HOW TO USE THIS IN 24 HOURS

**Suggested time split:**
- Module 1 (Python syntax): 2 hours
- Module 2 (Types + Pydantic): 2 hours
- Module 3 (Flask): 2.5 hours
- Module 4 (asyncio — read twice): 4 hours
- Module 5 (Async Flask): 2 hours
- Module 6 (Architecture): 1.5 hours
- Module 7 (Testing): 1 hour
- Module 8 (Production): 1 hour
- Final Project: remaining time

**[WHY this order]** asyncio is Boris's stated focus — it gets 4 hours. Flask is the
container. Python syntax is fast to learn for a TS dev. Don't spend more than 2 hours
on syntax — you already know how to program.

---

# MODULE 1: PYTHON FOR TYPESCRIPT DEVELOPERS

## 1.1 The Mental Model Shift

TypeScript compiles to JavaScript. Python is interpreted directly.
TypeScript has static types that are erased at runtime. Python has type hints that are
optional and mostly ignored at runtime (unless you use Pydantic or mypy).

| TypeScript concept | Python equivalent |
|---|---|
| `npm install` | `pip install` or `poetry add` |
| `package.json` | `pyproject.toml` or `requirements.txt` |
| `node_modules/` | virtual environment (`venv`) |
| `tsconfig.json` | `mypy.ini` or `pyproject.toml [tool.mypy]` |
| `interface` / `type` | `TypedDict`, `dataclass`, Pydantic `BaseModel` |
| `class` | `class` (similar, fewer features) |
| `async/await` | `async/await` (same syntax, different engine) |
| `Promise.all` | `asyncio.gather` |
| `Promise.allSettled` | `asyncio.gather(return_exceptions=True)` |
| Express | Flask |
| Express Router | Flask `Blueprint` |
| Zod | Pydantic |
| Jest | pytest |
| `console.log` | `print()` or `logging.getLogger()` |
| `process.env` | `os.environ` or `python-dotenv` |
| `try/catch` | `try/except` |
| `throw new Error` | `raise ValueError(...)` |
| `?.` optional chain | `getattr(obj, "field", None)` or `obj.field if obj else None` |
| `??` nullish coalescing | `value or default` (careful — falsy, not null-only) |

---

## 1.2 Setup — The Python Equivalent of `npm init`

```bash
# Create project
mkdir casenotes-api && cd casenotes-api

# Create virtual environment — equivalent of node_modules scoped to this project
python3 -m venv venv

# Activate it — must do this in every new terminal session
source venv/bin/activate        # Mac/Linux
# venv\Scripts\activate         # Windows

# You'll see (venv) in your prompt — means it's active
# Install packages into this isolated environment
pip install flask pydantic python-dotenv pytest

# Save dependencies — equivalent of package.json dependencies
pip freeze > requirements.txt
```

**[WHY virtual environments?]** Python installs packages globally by default. If Project A
needs `flask==2.0` and Project B needs `flask==3.0`, they conflict. A virtual environment
is an isolated Python + packages folder per project — same reason Node uses `node_modules`
per project. Always activate the venv before working.

**[WHY `pip freeze > requirements.txt`?]** This captures the exact installed versions.
Someone else running `pip install -r requirements.txt` gets identical versions. It is
`package-lock.json` equivalent. Commit it to git.

---

## 1.3 Syntax — What Changes From TypeScript

**Variables and types:**
```python
# Python — no let/const distinction. Variables are always mutable.
name = "Babalola Opeyemi"   # inferred as str
age = 31                     # inferred as int
is_active = True             # bool — capitalised True/False, not true/false
nothing = None               # None, not null or undefined

# Type hints — optional annotations (like TS but not enforced at runtime)
name: str = "Babalola Opeyemi"
age: int = 31
is_active: bool = True
note_id: str | None = None   # Python 3.10+ union syntax — same as TypeScript's string | null
```

**[WHY `None` not `null`?]** Python has one concept for "no value" — `None`. TypeScript
has two (`null` and `undefined`) because JavaScript has two. Python simplified this.

**[WHY `True` capitalised?]** Python convention. The booleans are `True` and `False`,
not `true` and `false`. You will forget this in the interview — remember it now.

---

**Strings:**
```python
# f-strings — equivalent of template literals
name = "Opeyemi"
greeting = f"Hello, {name}"         # "Hello, Opeyemi"
note_ref = f"Note-{note_id:04d}"    # zero-padded number

# Multi-line strings — triple quotes
sql = """
    SELECT * FROM notes
    WHERE case_id = %s
    AND is_deleted = false
"""
```

---

**Functions:**
```python
# TypeScript                          # Python
# function greet(name: string): str  # def greet(name: str) -> str:
def greet(name: str) -> str:
    return f"Hello, {name}"

# Default parameters
def create_note(
    case_id: int,
    content: str,
    created_by: str,
    is_urgent: bool = False          # default — same as TypeScript
) -> dict:
    return {
        "case_id": case_id,
        "content": content,
        "created_by": created_by,
        "is_urgent": is_urgent,
    }

# Keyword arguments — Python-specific, very useful
create_note(case_id=1, content="hello", created_by="ope@earlybird.ai")
create_note(1, "hello", "ope@earlybird.ai", is_urgent=True)
```

**[WHY keyword arguments?]** In TypeScript you pass an options object: `createNote({ caseId: 1, content: "hello" })`. Python functions support named arguments natively — you can call `create_note(is_urgent=True, case_id=1, content="hello", created_by="ope@earlybird.ai")` in any order by naming them. Same readability benefit as TS options objects, built into the language.

---

**Indentation — Python's most important syntax rule:**
```python
# Python uses indentation instead of curly braces
# A block is ANYTHING indented under a colon
if adviser is not None:
    adviser.caseload += 1       # inside the if
    print(adviser.name)         # still inside
print("done")                   # outside — back to zero indent

# TypeScript equivalent:
# if (adviser !== null) {
#   adviser.caseload += 1;
#   console.log(adviser.name);
# }
# console.log("done");
```

**[WHY indentation?]** This is Python's defining characteristic. It enforces readable
code structurally — you cannot write deeply nested spaghetti and hide it with braces.
The number of spaces must be consistent (4 spaces is the convention — use your editor's
auto-indent).

---

**Comparison operators:**
```python
# Python           # TypeScript
x == y             # x === y  (Python has no == vs === — always value equality)
x != y             # x !== y
x is None          # x === null (identity check — use for None specifically)
x is not None      # x !== null
not x              # !x
x and y            # x && y
x or y             # x || y
```

**[WHY `is None` not `== None`?]** `is` checks object identity (same memory address).
`None` is a singleton in Python — there is only one `None` object. `x is None` is the
correct, idiomatic way to check for null. Using `== None` works but triggers linting warnings.

---

**Lists, dicts, loops:**
```python
# List — equivalent of Array
notes: list[str] = ["note-1", "note-2", "note-3"]
notes.append("note-4")           # push
first = notes[0]                 # index
last = notes[-1]                 # last item — Python-specific negative index

# Dict — equivalent of object / Map
adviser: dict[str, str | int] = {
    "id": "ADV-001",
    "name": "Opeyemi",
    "caseload": 45,
}
adviser["region"] = "North West" # add key
name = adviser.get("name", "unknown")  # safe get with default — like ?? in TS

# For loop
for note in notes:
    print(note)

# Enumerate — when you need the index too
for i, note in enumerate(notes):
    print(f"{i}: {note}")

# Dict iteration
for key, value in adviser.items():
    print(f"{key}: {value}")

# List comprehension — map + filter in one line
urgent_notes = [n for n in notes if n.is_urgent]  # filter
note_ids = [n.id for n in notes]                   # map
```

**[WHY list comprehension?]** Python's equivalent of `.filter()` and `.map()`. It is the
idiomatic Python way — you will see it everywhere in a Python codebase. Read it as:
`[expression for item in iterable if condition]`.

---

**Error handling:**
```python
# TypeScript                  # Python
# try { } catch (e) { }       # try: except ExceptionType as e:

try:
    note = find_note(note_id)
except ValueError as e:
    print(f"Validation error: {e}")
except NotFoundError as e:
    print(f"Not found: {e}")
except Exception as e:         # catch-all — like catch (e: unknown)
    print(f"Unexpected: {e}")
finally:
    cleanup()                  # always runs — same as TypeScript finally

# Raising errors
raise ValueError("Content too short")
raise NotFoundError(f"Note {note_id} not found")
```

---

**Classes:**
```python
class CaseNote:
    # __init__ is the constructor
    def __init__(
        self,
        id: str,
        case_id: int,
        content: str,
        created_by: str,
        is_urgent: bool = False,
    ) -> None:
        self.id = id
        self.case_id = case_id
        self.content = content
        self.created_by = created_by
        self.is_urgent = is_urgent
        self.is_deleted = False

    def soft_delete(self) -> None:
        self.is_deleted = True

    def __repr__(self) -> str:          # like toString() / __repr__
        return f"CaseNote(id={self.id}, case_id={self.case_id})"
```

**[WHY `self`?]** Every instance method in Python must take `self` as its first parameter.
`self` is the equivalent of `this` in TypeScript. Python requires it explicitly — there is
no implicit `this`. If you forget `self`, the method call will fail with a confusing error.

---

# MODULE 2: TYPE SYSTEM — PYDANTIC IS YOUR ZOD

## 2.1 Why Python's Type Hints Alone Aren't Enough

```python
# Type hints are just annotations — Python does NOT enforce them at runtime
def add(a: int, b: int) -> int:
    return a + b

add("hello", "world")  # NO ERROR at runtime — returns "helloworld"
# Python ignores the hints unless you run mypy or use Pydantic
```

**[WHY this matters]** In TypeScript, the compiler enforces types before the code runs.
In Python, type hints are documentation for tools and developers — they don't stop bad
data from entering your functions at runtime. For a web API receiving user input, you
NEED runtime validation. That is what Pydantic does.

---

## 2.2 Pydantic — Python's Zod

```bash
pip install pydantic pydantic[email]
```

```python
from pydantic import BaseModel, Field, EmailStr
from datetime import datetime
from typing import Optional

# This is your Zod schema + TypeScript interface in one
class CreateNoteRequest(BaseModel):
    case_id: int
    content: str = Field(min_length=5, max_length=50000)
    created_by: EmailStr                    # validates email format
    is_urgent: bool = False                 # default value

# Using it
data = {
    "case_id": 1,
    "content": "Participant attended session",
    "created_by": "ope@earlybird.ai",
}

note_request = CreateNoteRequest(**data)    # validates on construction
print(note_request.case_id)                # 1
print(note_request.is_urgent)              # False — default applied
```

**[WHY `**data`?]** The `**` unpacks a dictionary into keyword arguments. It is the
Python equivalent of spreading an object: `CreateNoteRequest(**data)` is like
`new CreateNoteRequest({ ...data })` in TypeScript.

---

**Pydantic validation errors:**
```python
from pydantic import ValidationError

try:
    bad = CreateNoteRequest(
        case_id="not-a-number",       # wrong type
        content="hi",                  # too short
        created_by="not-an-email",     # invalid email
    )
except ValidationError as e:
    print(e.errors())
    # [
    #   {"loc": ["case_id"], "msg": "value is not a valid integer", ...},
    #   {"loc": ["content"], "msg": "ensure this value has at least 5 characters", ...},
    #   {"loc": ["created_by"], "msg": "value is not a valid email address", ...},
    # ]
```

**[WHY Pydantic over manual validation?]** One class gives you: type coercion, field
validation, error messages with field locations, JSON serialisation, and documentation.
The equivalent in plain Python would be 50+ lines of manual checks. In Flask, Pydantic is
your first line of defence against bad input.

---

**Pydantic models as DTOs:**
```python
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional
import uuid

# Domain response model
class CaseNoteResponse(BaseModel):
    id: str
    case_id: int
    content: str
    created_by: str
    is_urgent: bool
    created_at: datetime
    updated_at: datetime

    class Config:
        # Allows creating from ORM/SQLAlchemy objects
        from_attributes = True   # Pydantic v2 — was orm_mode = True in v1

# Create DTO — no id, timestamps (server generates them)
class CreateNoteRequest(BaseModel):
    case_id: int
    content: str = Field(min_length=5, max_length=50000)
    created_by: EmailStr
    is_urgent: bool = False

# Update DTO — all optional, only send what changes
class UpdateNoteRequest(BaseModel):
    content: Optional[str] = Field(default=None, min_length=5, max_length=50000)
    is_urgent: Optional[bool] = None

    def has_changes(self) -> bool:
        return any(v is not None for v in self.model_dump().values())
```

**[WHY `Optional[str]`?]** `Optional[str]` is `str | None` in Python's type system.
In Pydantic it means the field can be absent from the input or explicitly `None`.
Use it for PATCH fields — same pattern as TypeScript's `Partial<T>`.

---

**Serialising to dict/JSON:**
```python
note = CreateNoteRequest(case_id=1, content="hello world", created_by="ope@earlybird.ai")

# To dict — equivalent of toJSON()
note.model_dump()
# {"case_id": 1, "content": "hello world", "created_by": "ope@earlybird.ai", "is_urgent": False}

# To JSON string
note.model_dump_json()
# '{"case_id":1,"content":"hello world",...}'

# Exclude None values — useful for PATCH responses
update = UpdateNoteRequest(is_urgent=True)
update.model_dump(exclude_none=True)
# {"is_urgent": True}  — content not included because it's None
```

---

## 2.3 dataclasses — Lightweight Models Without Validation

```python
from dataclasses import dataclass, field
from datetime import datetime

# Equivalent of a TypeScript interface + class combined
# No validation — just structure and defaults
@dataclass
class Adviser:
    id: str
    name: str
    email: str
    caseload: int = 0
    is_active: bool = True

    def assign_case(self) -> None:
        self.caseload += 1

adviser = Adviser(id="ADV-001", name="Opeyemi", email="ope@earlybird.ai")
adviser.assign_case()
print(adviser.caseload)  # 1
```

**[WHY dataclass over plain class?]** `@dataclass` auto-generates `__init__`, `__repr__`,
and `__eq__` — the same boilerplate you'd write manually. It is the Python equivalent of
TypeScript's class with constructor shorthand (`public readonly id: string`). Use
dataclasses for internal domain objects that don't touch external input. Use Pydantic for
anything receiving user/API data.

---

# MODULE 3: FLASK — BUILDING APIs

## 3.1 Flask vs Express — The Mental Model

| Express (Node.js) | Flask (Python) |
|---|---|
| `app.get("/path", handler)` | `@app.route("/path", methods=["GET"])` |
| `req.body` | `request.get_json()` |
| `req.params.id` | URL variable: `<id>` → function parameter |
| `req.query.page` | `request.args.get("page")` |
| `res.json(data)` | `jsonify(data)` |
| `res.status(404).json(...)` | `jsonify(...), 404` |
| Express Router | Flask `Blueprint` |
| `app.use(middleware)` | `@app.before_request` or Blueprint `before_request` |
| `createApp()` factory | `create_app()` factory |

---

## 3.2 Basic Flask App

```bash
pip install flask
```

```python
# app.py
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route("/health")
def health():
    return jsonify({"status": "ok"})

@app.route("/notes", methods=["GET"])
def get_notes():
    notes = [{"id": "1", "content": "Test note"}]
    return jsonify(notes)

@app.route("/notes/<note_id>", methods=["GET"])
def get_note(note_id: str):               # URL variable injected as parameter
    return jsonify({"id": note_id})

@app.route("/notes", methods=["POST"])
def create_note():
    data = request.get_json()             # req.body equivalent
    return jsonify(data), 201            # status code as second return value

if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

**[WHY decorator routes?]** `@app.route("/notes")` is a Python decorator — it wraps the
function and registers it with Flask's URL routing system. The decorator syntax is the
same idea as TypeScript decorators (`@Controller`, `@Get`) in NestJS, but Flask uses them
for all route registration. The function below the decorator IS the route handler.

---

## 3.3 Request and Response

```python
from flask import request, jsonify, abort

@app.route("/notes", methods=["POST"])
def create_note():
    # Get JSON body — equivalent of req.body in Express
    data = request.get_json()
    if data is None:
        return jsonify({"error": "Request body must be JSON"}), 400

    # Query parameters — equivalent of req.query
    page = request.args.get("page", 1, type=int)      # default=1, cast to int
    limit = request.args.get("limit", 20, type=int)

    # Headers
    auth_header = request.headers.get("Authorization")

    # Response with status code — tuple: (body, status_code)
    return jsonify({"id": "new-note-id"}), 201

# abort() — immediately returns an error response
@app.route("/notes/<note_id>")
def get_note(note_id: str):
    note = find_note(note_id)
    if note is None:
        abort(404)                  # like res.status(404).send() but immediate
    return jsonify(note)
```

---

## 3.4 Blueprints — Flask's Express Router

```python
# routes/notes.py — equivalent of an Express Router file
from flask import Blueprint, jsonify, request

notes_bp = Blueprint("notes", __name__, url_prefix="/api/notes")

@notes_bp.route("/", methods=["GET"])
def list_notes():
    return jsonify([])

@notes_bp.route("/<note_id>", methods=["GET"])
def get_note(note_id: str):
    return jsonify({"id": note_id})

@notes_bp.route("/", methods=["POST"])
def create_note():
    data = request.get_json()
    return jsonify(data), 201

@notes_bp.route("/<note_id>", methods=["PATCH"])
def update_note(note_id: str):
    data = request.get_json()
    return jsonify({"id": note_id, **data})

@notes_bp.route("/<note_id>", methods=["DELETE"])
def delete_note(note_id: str):
    return jsonify({"deleted": True}), 200
```

```python
# app.py — register blueprints (equivalent of app.use("/api/notes", notesRouter))
from flask import Flask
from routes.notes import notes_bp
from routes.advisers import advisers_bp

def create_app() -> Flask:
    app = Flask(__name__)
    app.register_blueprint(notes_bp)
    app.register_blueprint(advisers_bp)
    return app

app = create_app()

if __name__ == "__main__":
    app.run(debug=True)
```

**[WHY the `create_app()` factory?]** Same reason as Express — separating app creation
from server startup. Tests can call `create_app()` to get a fresh app instance without
starting a real server. The `url_prefix="/api/notes"` on the Blueprint means every route
defined inside it is automatically prefixed. You don't repeat the prefix on each route.

---

## 3.5 Error Handlers

```python
# app.py
from flask import Flask, jsonify
from werkzeug.exceptions import HTTPException

def create_app() -> Flask:
    app = Flask(__name__)

    # Handle 404 — equivalent of Express's notFoundHandler
    @app.errorhandler(404)
    def not_found(e):
        return jsonify({"error": "Resource not found"}), 404

    # Handle 422 — validation error
    @app.errorhandler(422)
    def unprocessable(e):
        return jsonify({"error": "Invalid request data"}), 422

    # Handle all HTTP exceptions
    @app.errorhandler(HTTPException)
    def handle_http_exception(e):
        return jsonify({"error": e.description, "status": e.code}), e.code

    # Handle unexpected exceptions — equivalent of Express error middleware
    @app.errorhandler(Exception)
    def handle_exception(e):
        app.logger.error(f"Unhandled exception: {e}", exc_info=True)
        return jsonify({"error": "Internal server error"}), 500

    app.register_blueprint(notes_bp)
    return app
```

---

## 3.6 Middleware — before_request and after_request

```python
# Equivalent of Express middleware
from flask import Flask, request, g
import uuid
import time

def create_app() -> Flask:
    app = Flask(__name__)

    @app.before_request
    def attach_request_id():
        # g is Flask's request-scoped storage — like res.locals in Express
        g.request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
        g.start_time = time.time()

    @app.after_request
    def add_headers(response):
        response.headers["X-Request-ID"] = g.request_id
        duration = (time.time() - g.start_time) * 1000
        response.headers["X-Response-Time"] = f"{duration:.2f}ms"
        return response

    return app
```

**[WHY `g`?]** Flask's `g` object is request-scoped — it lives for the duration of one
request and is cleaned up automatically. It is equivalent to `res.locals` in Express.
Store per-request data (request ID, authenticated user, DB connection) on `g`.

---

# MODULE 4: asyncio — BORIS'S FOCUS

This is the most important module. Read it twice.

## 4.1 The Fundamental Difference From JavaScript

JavaScript has ONE event loop and it is always running. Every async operation in Node.js
uses it automatically. You never think about the event loop directly.

Python is different. Python has:
1. Threads — OS-level parallel execution (blocked by the GIL for CPU work)
2. `asyncio` — a cooperative concurrency system you must explicitly opt into
3. Multiprocessing — true parallelism for CPU-bound work

**asyncio is Python's version of Node.js's event loop — but you have to start it yourself
and use it explicitly.**

```
JavaScript/Node.js:
  Event loop runs always → all async I/O is non-blocking automatically

Python without asyncio:
  Code runs synchronously → I/O blocks the process → one request at a time

Python with asyncio:
  You start an event loop → async functions can yield control during I/O →
  multiple operations can be in-flight simultaneously
```

---

## 4.2 The GIL — Why asyncio Exists in Python

Python has the Global Interpreter Lock (GIL). Only ONE thread can execute Python
bytecode at a time. This means:

- **Threading in Python does NOT give you true CPU parallelism** — threads take turns,
  they don't run in parallel (for CPU-bound work)
- **For I/O-bound work (waiting for network, disk, database)** — threads help because the
  GIL is released during I/O waits
- **asyncio sidesteps the GIL** — instead of using threads, it uses cooperative yielding.
  When one coroutine is waiting for I/O, the event loop runs another coroutine.

**[WHY Boris cares about this]** Earlybird's backend does a lot of I/O: calling AI APIs
(transcription, note generation), writing to databases, sending notifications. All of
that is network I/O. asyncio makes it possible to handle many of those concurrently
without spawning threads — which is faster and uses less memory.

---

## 4.3 async/await in Python — Syntax and Rules

```python
import asyncio

# async def defines a coroutine — equivalent of async function in TS
async def fetch_note(note_id: str) -> dict:
    # await suspends THIS coroutine and lets the event loop run others
    # Only works on coroutines and awaitables
    await asyncio.sleep(0.1)    # simulates async I/O (like an API call)
    return {"id": note_id, "content": "Test note"}

# You cannot call a coroutine like a regular function
result = fetch_note("note-1")   # This returns a coroutine OBJECT, not the result
                                 # Equivalent of calling an async fn without await in TS

# You must await it — or run it with asyncio.run()
async def main():
    result = await fetch_note("note-1")   # now it runs and returns the dict
    print(result)

# Entry point — starts the event loop
asyncio.run(main())
```

**[WHY `asyncio.run(main())`?]** JavaScript starts its event loop automatically when the
process starts. Python requires you to explicitly start it. `asyncio.run()` creates an
event loop, runs the coroutine you pass, and closes the loop when done. In a Flask/FastAPI
app, the framework handles this for you — but understanding it is essential for debugging.

---

## 4.4 asyncio.gather — Running Things in Parallel

This is the single most important asyncio function to know.

```python
import asyncio
import aiohttp     # pip install aiohttp — async HTTP client

# Equivalent of Promise.all in TypeScript
async def get_note_with_adviser(note_id: str) -> dict:
    # WRONG — sequential, slow
    note = await fetch_note(note_id)          # waits ~100ms
    adviser = await fetch_adviser(note.adviser_id)  # waits ANOTHER ~100ms
    # Total: ~200ms

    # RIGHT — parallel, fast
    note, adviser = await asyncio.gather(
        fetch_note(note_id),
        fetch_adviser(note.adviser_id),
    )
    # Total: ~100ms — both run simultaneously
    return {"note": note, "adviser": adviser}
```

**[WHY `asyncio.gather` maps to `Promise.all`]** Both take multiple awaitables/Promises
and run them concurrently. Both return when ALL complete. Both fail fast — if any one
fails, the others are cancelled and the error propagates.

```python
# TypeScript equivalent:
# const [note, adviser] = await Promise.all([
#   fetchNote(noteId),
#   fetchAdviser(adviserId),
# ]);
```

---

## 4.5 asyncio.gather with Exception Handling

```python
import asyncio

# Equivalent of Promise.allSettled — don't fail fast, collect all results
async def get_multiple_notes(note_ids: list[str]) -> list:
    tasks = [fetch_note(note_id) for note_id in note_ids]

    # return_exceptions=True — equivalent of Promise.allSettled
    results = await asyncio.gather(*tasks, return_exceptions=True)

    successful = []
    failed = []

    for note_id, result in zip(note_ids, results):
        if isinstance(result, Exception):
            failed.append({"id": note_id, "error": str(result)})
        else:
            successful.append(result)

    return {"notes": successful, "errors": failed}
```

**[WHY `*tasks`?]** The `*` unpacks the list into individual arguments.
`asyncio.gather(*tasks)` is the same as `asyncio.gather(tasks[0], tasks[1], tasks[2])`.
`gather` takes varargs, not a list — `*` converts the list. Equivalent of spreading:
`Promise.all([...tasks])`.

---

## 4.6 asyncio.create_task — Fire and Don't Wait

```python
import asyncio

# Equivalent of not-awaiting a Promise (fire and forget)
async def save_note_and_notify(note: dict) -> dict:
    # Save the note — we need this done before responding
    saved = await save_to_db(note)

    # Send notification — we don't need to wait for this
    # create_task schedules it to run "in the background"
    asyncio.create_task(send_adviser_notification(saved["id"]))

    # Return immediately — notification will complete later
    return saved
```

**[WHY create_task over fire-and-forget in TS?]** In TypeScript you can just not await
a Promise: `sendAdviserNotification(id)` — it runs in the background. Python's asyncio
requires you to explicitly wrap unwaited coroutines in `create_task()`. Otherwise the
coroutine is created but never actually scheduled to run.

**[IMPORTANT caveat]** If the process dies before the task completes, you lose it. For
reliable background work, use a proper queue (Celery + Redis) not `create_task`. Use
`create_task` for non-critical async work like logging or metrics.

---

## 4.7 async Context Managers and Async Iteration

```python
import asyncio
import aiohttp

# Async context manager — equivalent of async using() in C# or try-with-resources
async def call_ai_api(prompt: str) -> str:
    async with aiohttp.ClientSession() as session:          # opens and closes automatically
        async with session.post(
            "https://api.example.com/generate",
            json={"prompt": prompt}
        ) as response:
            data = await response.json()
            return data["result"]
    # session and response are automatically closed here — even on exception

# Async iteration — streaming responses
async def stream_transcription(audio_url: str):
    async with aiohttp.ClientSession() as session:
        async with session.get(audio_url) as response:
            async for chunk in response.content:            # reads chunks as they arrive
                process_chunk(chunk)
```

**[WHY async context managers matter for Earlybird]** WorkScribe transcribes audio and
calls AI APIs. Both are long-running I/O operations. Async context managers ensure the
HTTP session and response stream are always properly closed — no connection leaks.

---

## 4.8 asyncio Timeouts

```python
import asyncio

# Never trust external APIs — always set timeouts
async def call_with_timeout(note_id: str) -> dict:
    try:
        # Wait at most 5 seconds
        result = await asyncio.wait_for(
            fetch_note(note_id),
            timeout=5.0
        )
        return result
    except asyncio.TimeoutError:
        raise TimeoutError(f"Fetching note {note_id} timed out after 5s")
```

---

## 4.9 asyncio Concurrency Limiting — Semaphores

```python
import asyncio

# What if you have 1000 notes to fetch but the API has rate limits?
# Without limiting — you fire 1000 requests simultaneously and get rate-limited
# With semaphore — max 10 requests at a time

async def fetch_all_notes_safely(note_ids: list[str]) -> list[dict]:
    semaphore = asyncio.Semaphore(10)   # max 10 concurrent

    async def fetch_one(note_id: str) -> dict:
        async with semaphore:            # blocks if 10 are already running
            return await fetch_note(note_id)

    tasks = [fetch_one(note_id) for note_id in note_ids]
    return await asyncio.gather(*tasks)
```

**[WHY semaphores?]** This is the `p-limit` pattern from Node.js, built into asyncio.
Firing unlimited concurrent requests to an external API is how you get your IP rate-limited
or banned. Semaphores are the production solution.

---

## 4.10 The asyncio Event Loop — What's Actually Happening

```
Coroutine A runs
  ↓ await some_io()  ← A suspends, event loop picks up next runnable task
Coroutine B runs
  ↓ await some_io()  ← B suspends
Coroutine C runs
  ↓ completes        ← C finishes
some_io() for A completes  ← event loop resumes A
A continues...
```

This is cooperative concurrency — each coroutine voluntarily yields at `await` points.
Nothing preempts a running coroutine. If you write a coroutine that does heavy CPU work
without any `await`, it will block the entire event loop — all other coroutines starve.

**[SENIOR DECISION — CPU-bound work]**
```python
import asyncio

# WRONG — heavy computation blocks the event loop
async def process_transcript(text: str) -> str:
    return heavy_nlp_processing(text)   # blocks for 2 seconds — nothing else runs

# RIGHT — offload CPU-bound work to a thread pool
async def process_transcript(text: str) -> str:
    loop = asyncio.get_event_loop()
    # run_in_executor runs the function in a thread, returns awaitable
    result = await loop.run_in_executor(None, heavy_nlp_processing, text)
    return result
```

**[WHY `run_in_executor`?]** CPU-bound work cannot be made async — it holds the GIL and
blocks everything. `run_in_executor` moves it to a thread pool, which releases the main
thread back to the event loop. For Earlybird: audio transcription processing, PDF
generation, heavy data transformation — these should all use `run_in_executor`.

---

# MODULE 5: ASYNC FLASK

## 5.1 Flask 2.0+ Native Async Support

Flask 2.0 added native async/await support. Install the async extra:

```bash
pip install flask[async]
# This installs asgiref which powers async route handling in Flask
```

```python
from flask import Flask, jsonify, request
import asyncio
import aiohttp

app = Flask(__name__)

# async route handler — just add async def
@app.route("/notes/<note_id>")
async def get_note(note_id: str):
    note = await fetch_note_from_db(note_id)      # non-blocking DB call
    adviser = await fetch_adviser(note.adviser_id) # non-blocking HTTP call
    return jsonify({"note": note, "adviser": adviser})

# Parallel fetching in a route
@app.route("/dashboard")
async def get_dashboard():
    note_count, adviser_count, recent_notes = await asyncio.gather(
        count_notes(),
        count_advisers(),
        get_recent_notes(limit=5),
    )
    return jsonify({
        "note_count": note_count,
        "adviser_count": adviser_count,
        "recent_notes": recent_notes,
    })
```

**[WHY Flask over FastAPI for async?]** FastAPI is purpose-built for async and uses
ASGI natively. Flask 2.0 added async support but is fundamentally WSGI. If Boris's team
is using Flask, it likely predates FastAPI's popularity. Knowing how to use async in
Flask correctly is exactly what will impress him — it shows you understand the limitations
and can work within them.

---

## 5.2 Async HTTP Client — aiohttp

```python
import aiohttp
import asyncio

# Never use the `requests` library in async code — it is synchronous and blocks
# Use aiohttp for async HTTP requests

async def call_ai_transcription(audio_url: str) -> dict:
    async with aiohttp.ClientSession() as session:
        async with session.post(
            "https://api.earlybird.ai/transcribe",
            json={"audio_url": audio_url},
            headers={"Authorization": f"Bearer {API_KEY}"},
            timeout=aiohttp.ClientTimeout(total=30),  # 30 second timeout
        ) as response:
            if response.status != 200:
                raise RuntimeError(f"Transcription API error: {response.status}")
            return await response.json()

# In a Flask route
@app.route("/transcribe", methods=["POST"])
async def transcribe():
    data = request.get_json()
    result = await call_ai_transcription(data["audio_url"])
    return jsonify(result)
```

**[WHY NOT `requests`?]** The `requests` library (Python's most popular HTTP library) is
synchronous — it blocks the thread while waiting for the response. In an async Flask
route, using `requests` will block the event loop for the duration of the HTTP call,
defeating the entire purpose of async. `aiohttp` is non-blocking — it yields control
to the event loop while waiting for the response.

---

## 5.3 Async Database Access

```python
# For truly async database access, use an async driver
# SQLAlchemy 1.4+ supports async with asyncpg (PostgreSQL)

pip install sqlalchemy asyncpg

from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine(
    "postgresql+asyncpg://user:password@localhost/casenotes",
    echo=True,
)

AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

async def get_note_by_id(note_id: str) -> dict | None:
    async with AsyncSessionLocal() as session:
        result = await session.execute(
            select(NoteModel).where(NoteModel.id == note_id)
        )
        note = result.scalar_one_or_none()
        return note
```

**[WHY asyncpg over psycopg2?]** `psycopg2` is synchronous. `asyncpg` is a PostgreSQL
driver built from scratch for async — it is also significantly faster than psycopg2 for
high-throughput workloads. For an async Flask/FastAPI backend, the database driver must
also be async, or you've blocked the event loop at the most expensive point.

---

# MODULE 6: ARCHITECTURE PATTERNS IN PYTHON

## 6.1 Project Structure — Flask Equivalent of Your TypeScript Layout

```
casenotes-api/
  src/
    __init__.py           ← makes src a Python package
    app.py                ← create_app() factory
    config.py             ← environment config (Pydantic Settings)
    routes/
      __init__.py
      notes.py            ← Blueprint: /api/notes
      advisers.py         ← Blueprint: /api/advisers
    domain/
      __init__.py
      models.py           ← Pydantic schemas (CreateNoteRequest, etc.)
      errors.py           ← custom exception hierarchy
    services/
      __init__.py
      note_service.py     ← business logic
      adviser_service.py
    repositories/
      __init__.py
      note_repository.py  ← data access (in-memory or DB)
    middleware/
      __init__.py
      request_id.py
      auth.py
  tests/
    test_note_service.py
    test_notes_routes.py
  Dockerfile
  requirements.txt
  .env.example
```

**[WHY `__init__.py`?]** Python treats a directory as a package (importable module) only
if it contains an `__init__.py` file. It can be empty. Without it, Python cannot import
from that directory. Every folder in your project that you want to import from needs one.

---

## 6.2 Repository Pattern — Same as TypeScript, Python Syntax

```python
# domain/errors.py
class AppError(Exception):
    def __init__(self, message: str, status_code: int = 500):
        super().__init__(message)
        self.status_code = status_code

class NotFoundError(AppError):
    def __init__(self, resource: str, id: str):
        super().__init__(f"{resource} with id '{id}' not found", 404)

class ValidationError(AppError):
    def __init__(self, message: str):
        super().__init__(message, 422)

class ConflictError(AppError):
    def __init__(self, message: str):
        super().__init__(message, 409)
```

```python
# repositories/note_repository.py
from abc import ABC, abstractmethod     # ABC = Abstract Base Class
from domain.models import CaseNote
from typing import Optional
import uuid
from datetime import datetime

# Abstract repository — Python equivalent of a TypeScript interface
class INoteRepository(ABC):
    @abstractmethod
    def find_by_id(self, note_id: str) -> Optional[CaseNote]:
        pass

    @abstractmethod
    def find_all(self) -> list[CaseNote]:
        pass

    @abstractmethod
    def create(self, data: dict) -> CaseNote:
        pass

    @abstractmethod
    def update(self, note_id: str, updates: dict) -> CaseNote:
        pass

    @abstractmethod
    def soft_delete(self, note_id: str) -> CaseNote:
        pass


# In-memory implementation — use for the interview project
class InMemoryNoteRepository(INoteRepository):
    def __init__(self):
        self._store: dict[str, CaseNote] = {}

    def find_by_id(self, note_id: str) -> Optional[CaseNote]:
        return self._store.get(note_id)

    def find_all(self) -> list[CaseNote]:
        return [n for n in self._store.values() if not n.is_deleted]

    def create(self, data: dict) -> CaseNote:
        note = CaseNote(
            id=str(uuid.uuid4()),
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow(),
            **data,
        )
        self._store[note.id] = note
        return note

    def update(self, note_id: str, updates: dict) -> CaseNote:
        note = self.find_by_id(note_id)
        if note is None:
            raise NotFoundError("CaseNote", note_id)
        for key, value in updates.items():
            if value is not None:
                setattr(note, key, value)
        note.updated_at = datetime.utcnow()
        return note

    def soft_delete(self, note_id: str) -> CaseNote:
        note = self.find_by_id(note_id)
        if note is None:
            raise NotFoundError("CaseNote", note_id)
        if note.is_deleted:
            raise ConflictError(f"Note {note_id} is already deleted")
        note.is_deleted = True
        note.updated_at = datetime.utcnow()
        return note
```

**[WHY `ABC` and `@abstractmethod`?]** Python doesn't have interfaces. `ABC` (Abstract
Base Class) is the closest equivalent. A class inheriting from `ABC` with
`@abstractmethod` methods cannot be instantiated unless all abstract methods are
implemented. It is Python's way of enforcing the interface contract — if you forget to
implement `find_by_id` on `InMemoryNoteRepository`, Python raises `TypeError` at startup.

---

## 6.3 Service Layer

```python
# services/note_service.py
from repositories.note_repository import INoteRepository
from domain.models import CreateNoteRequest, UpdateNoteRequest
from domain.errors import NotFoundError

class NoteService:
    def __init__(self, repo: INoteRepository):
        self._repo = repo             # depends on the interface, not the implementation

    def get_note(self, note_id: str) -> dict:
        note = self._repo.find_by_id(note_id)
        if note is None:
            raise NotFoundError("CaseNote", note_id)
        return note.__dict__

    def list_notes(self) -> list[dict]:
        return [n.__dict__ for n in self._repo.find_all()]

    def create_note(self, payload: CreateNoteRequest) -> dict:
        note = self._repo.create(payload.model_dump())
        return note.__dict__

    def update_note(self, note_id: str, payload: UpdateNoteRequest) -> dict:
        updates = payload.model_dump(exclude_none=True)
        if not updates:
            raise ValidationError("No fields to update")
        note = self._repo.update(note_id, updates)
        return note.__dict__

    def delete_note(self, note_id: str) -> dict:
        note = self._repo.soft_delete(note_id)
        return note.__dict__
```

---

## 6.4 Wiring It Together — Dependency Injection Without a Framework

```python
# app.py
from flask import Flask, jsonify
from repositories.note_repository import InMemoryNoteRepository
from services.note_service import NoteService
from routes.notes import create_notes_blueprint
from domain.errors import AppError

def create_app() -> Flask:
    app = Flask(__name__)

    # Compose dependencies
    note_repo = InMemoryNoteRepository()
    note_service = NoteService(note_repo)

    # Pass service into the Blueprint factory
    notes_bp = create_notes_blueprint(note_service)
    app.register_blueprint(notes_bp)

    # Error handlers
    @app.errorhandler(AppError)
    def handle_app_error(e: AppError):
        return jsonify({"error": str(e)}), e.status_code

    @app.errorhandler(Exception)
    def handle_exception(e: Exception):
        app.logger.error(f"Unhandled: {e}", exc_info=True)
        return jsonify({"error": "Internal server error"}), 500

    return app
```

```python
# routes/notes.py — Blueprint factory pattern
from flask import Blueprint, jsonify, request
from pydantic import ValidationError as PydanticValidationError
from services.note_service import NoteService
from domain.models import CreateNoteRequest, UpdateNoteRequest

def create_notes_blueprint(note_service: NoteService) -> Blueprint:
    bp = Blueprint("notes", __name__, url_prefix="/api/notes")

    @bp.route("/", methods=["GET"])
    def list_notes():
        return jsonify(note_service.list_notes())

    @bp.route("/<note_id>", methods=["GET"])
    def get_note(note_id: str):
        note = note_service.get_note(note_id)
        return jsonify(note)

    @bp.route("/", methods=["POST"])
    def create_note():
        try:
            payload = CreateNoteRequest(**request.get_json(force=True))
        except PydanticValidationError as e:
            return jsonify({"error": e.errors()}), 422
        note = note_service.create_note(payload)
        return jsonify(note), 201

    @bp.route("/<note_id>", methods=["PATCH"])
    def update_note(note_id: str):
        try:
            payload = UpdateNoteRequest(**request.get_json(force=True))
        except PydanticValidationError as e:
            return jsonify({"error": e.errors()}), 422
        note = note_service.update_note(note_id, payload)
        return jsonify(note)

    @bp.route("/<note_id>", methods=["DELETE"])
    def delete_note(note_id: str):
        note = note_service.delete_note(note_id)
        return jsonify(note)

    return bp
```

---

# MODULE 7: TESTING WITH PYTEST

## 7.1 pytest — Python's Jest Equivalent

```bash
pip install pytest pytest-asyncio
```

```python
# tests/test_note_service.py
import pytest
from unittest.mock import MagicMock, patch
from services.note_service import NoteService
from repositories.note_repository import InMemoryNoteRepository
from domain.models import CreateNoteRequest
from domain.errors import NotFoundError

# Fixtures — equivalent of Jest beforeEach setup
@pytest.fixture
def repo():
    return InMemoryNoteRepository()

@pytest.fixture
def service(repo):
    return NoteService(repo)

@pytest.fixture
def created_note(service):
    payload = CreateNoteRequest(
        case_id=1,
        content="This is a test note content",
        created_by="test@earlybird.ai",
    )
    return service.create_note(payload)

# Tests — no describe blocks, just functions prefixed with test_
def test_create_note_returns_note_with_id(service):
    payload = CreateNoteRequest(
        case_id=1,
        content="Participant attended session and discussed job goals",
        created_by="adviser@earlybird.ai",
    )
    result = service.create_note(payload)
    assert result["id"] is not None
    assert result["case_id"] == 1
    assert result["is_urgent"] is False

def test_get_note_raises_not_found_for_unknown_id(service):
    with pytest.raises(NotFoundError):
        service.get_note("non-existent-id")

def test_soft_delete_prevents_note_appearing_in_list(service, created_note):
    service.delete_note(created_note["id"])
    notes = service.list_notes()
    assert all(n["id"] != created_note["id"] for n in notes)

def test_delete_already_deleted_note_raises_conflict(service, created_note):
    service.delete_note(created_note["id"])
    from domain.errors import ConflictError
    with pytest.raises(ConflictError):
        service.delete_note(created_note["id"])

def test_update_with_no_fields_raises_validation_error(service, created_note):
    from domain.models import UpdateNoteRequest
    from domain.errors import ValidationError
    with pytest.raises(ValidationError):
        service.update_note(created_note["id"], UpdateNoteRequest())
```

```python
# tests/test_notes_routes.py — integration tests with Flask test client
import pytest
import json
from app import create_app

@pytest.fixture
def client():
    app = create_app()
    app.config["TESTING"] = True
    with app.test_client() as client:
        yield client

def test_create_note_returns_201(client):
    response = client.post(
        "/api/notes/",
        data=json.dumps({
            "case_id": 1,
            "content": "Participant attended first session today",
            "created_by": "adviser@earlybird.ai",
        }),
        content_type="application/json",
    )
    assert response.status_code == 201
    data = response.get_json()
    assert data["id"] is not None

def test_create_note_with_short_content_returns_422(client):
    response = client.post(
        "/api/notes/",
        data=json.dumps({
            "case_id": 1,
            "content": "hi",              # too short — min 5
            "created_by": "adviser@earlybird.ai",
        }),
        content_type="application/json",
    )
    assert response.status_code == 422

def test_get_nonexistent_note_returns_404(client):
    response = client.get("/api/notes/does-not-exist")
    assert response.status_code == 404
```

**[WHY fixtures?]** pytest fixtures are the equivalent of Jest's `beforeEach` setup.
A fixture is a function decorated with `@pytest.fixture`. When a test function lists a
fixture as a parameter, pytest calls it automatically and injects the result. This avoids
repeating setup code across tests. Fixtures can depend on other fixtures — `service(repo)`
depends on the `repo` fixture.

---

# MODULE 8: PRODUCTION — CONFIG, LOGGING, DOCKER

## 8.1 Environment Config with Pydantic Settings

```bash
pip install pydantic-settings python-dotenv
```

```python
# config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # Reads from environment variables automatically
    flask_env: str = "development"
    secret_key: str = "dev-secret-change-in-production"
    database_url: str = "sqlite:///dev.db"
    log_level: str = "INFO"
    allowed_origins: list[str] = ["http://localhost:3000"]

    class Config:
        env_file = ".env"         # also reads from .env file
        case_sensitive = False    # DATABASE_URL or database_url both work

# Singleton — validated at import time
settings = Settings()
# If DATABASE_URL is missing and has no default → ValidationError at startup
# NEVER reaches production with a bad config — same as TypeScript's startup validation
```

**[WHY validate config at startup?]** If `DATABASE_URL` is wrong, you want to know
immediately when the server starts — not when the first request hits a database function.
Pydantic Settings crashes with a clear error message if required config is missing.
This is the same pattern as the TypeScript masterclass's startup config validation.

---

## 8.2 Structured Logging

```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record: logging.LogRecord) -> str:
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "message": record.getMessage(),
            "module": record.module,
        }
        if hasattr(record, "request_id"):
            log_entry["request_id"] = record.request_id
        if record.exc_info:
            log_entry["exception"] = self.formatException(record.exc_info)
        return json.dumps(log_entry)

def setup_logging(level: str = "INFO") -> None:
    handler = logging.StreamHandler()
    handler.setFormatter(JSONFormatter())
    logging.basicConfig(level=level, handlers=[handler])

# Usage
logger = logging.getLogger(__name__)
logger.info("Note created", extra={"request_id": "req-123", "note_id": "note-456"})
```

---

## 8.3 Dockerfile — Multi-Stage Build

```dockerfile
# Stage 1 — build dependencies
FROM python:3.12-slim AS builder

WORKDIR /app

# Install build tools (some packages need compilation)
RUN apt-get update && apt-get install -y --no-install-recommends gcc

COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2 — production image
FROM python:3.12-slim AS production

WORKDIR /app

# Non-root user — security best practice
RUN addgroup --system appgroup && adduser --system --group appuser

# Copy installed packages from builder stage
COPY --from=builder /root/.local /home/appuser/.local

# Copy application code
COPY src/ ./src/

# Set ownership
RUN chown -R appuser:appgroup /app

USER appuser

ENV PATH=/home/appuser/.local/bin:$PATH
ENV PYTHONPATH=/app

# gunicorn is production-grade WSGI server — equivalent of pm2 for Node
CMD ["gunicorn", "--workers=4", "--bind=0.0.0.0:8000", "src.app:create_app()"]
```

**[WHY gunicorn?]** Flask's built-in server (`app.run()`) is for development only — it's
single-threaded and not hardened for production. `gunicorn` is the production-grade WSGI
server — multiple worker processes, graceful shutdown, battle-tested. Equivalent of using
`pm2` with Node.js. The `--workers=4` spawns 4 Python processes to handle requests in
parallel (workers bypass the GIL because they're separate processes, not threads).

**[WHY multi-stage?]** Build dependencies (compilers, headers) make images large and are
security liabilities. Multi-stage builds install packages in a builder stage, then copy
only the installed packages to a clean final image. The final image has no compiler —
smaller and safer.

---

# FINAL PROJECT: CASE NOTES API — BUILD AND SUBMIT

## Project Specification

Build a production-ready REST API in Python + Flask for the Earlybird case notes domain.

**Endpoints:**
```
GET    /api/notes                     list all non-deleted notes
GET    /api/notes/<id>                get one note
POST   /api/notes                     create a note
PATCH  /api/notes/<id>                update content or urgency
DELETE /api/notes/<id>                soft delete

GET    /api/advisers                  list advisers
POST   /api/advisers/<id>/assign/<participant_id>   assign participant to adviser

GET    /health                        liveness check
```

**Domain models:**
```python
# All fields from your previous Python round — you already know these
CaseNote: id, case_id, content, created_by (email), is_urgent, is_deleted, created_at, updated_at
Adviser:  id, name, email, caseload, region, is_active
Participant: id, first_name, last_name, email, assigned_adviser_id
```

**Technical requirements:**
- Pydantic validation on all inputs — never trust request body directly
- Custom error hierarchy (AppError → NotFoundError, ValidationError, ConflictError)
- Repository interface (ABC) with in-memory implementation
- Services depend on interfaces, not implementations
- At least one async route using `asyncio.gather` (e.g. dashboard endpoint)
- Structured JSON logging with request IDs
- Environment config via Pydantic Settings (validated at startup)
- pytest tests: unit tests for service layer + integration tests for at least 2 routes
- Multi-stage Dockerfile
- `.env.example`
- README with one architectural decision explained (e.g. why the async dashboard route)

---

## Grading Scale (Boris's Lens)

**Junior:**
- Logic in route handlers
- No Pydantic — raw `request.get_json()` dict access
- No error handling — KeyError crashes return 500
- No tests

**Mid:**
- Services exist but import Flask's `request` directly (wrong layer)
- Pydantic on some endpoints
- `except Exception` catching everything the same way
- A few happy-path tests

**Senior:**
- Clean layers: routes → service → repository. Routes know nothing about data storage.
- Every route validates input with Pydantic before the service sees it
- Custom error hierarchy — each error type has the right status code
- At least one async route that uses `asyncio.gather` correctly
- Tests cover: missing fields, too-short content, not found, double-delete
- Structured logging with request IDs on every request
- Config validated at startup

**Principal:**
- Background task pattern (POST returns 202 + job ID, GET /jobs/<id> returns result)
- Semaphore-controlled concurrency for bulk operations
- Health check that verifies actual dependencies (not just returns 200)
- README explains WHY decisions were made, not just what they are

---

## What To Say When Boris Asks About Async

The question Boris will ask: "Walk me through how you'd handle a request that needs to
call three different services — transcription API, database, and notification service."

The answer:

"I'd use `asyncio.gather` to run the independent calls concurrently. The transcription
result and the database write are independent — I don't need the DB result to start
transcription. So I'd kick both off in parallel and await them together. The notification
can be fired as a `create_task` after the response is returned — the user doesn't need
to wait for it. The critical thing is making sure I'm using async-native libraries:
`aiohttp` for HTTP calls, `asyncpg` for the database — not `requests` or `psycopg2`,
because those block the event loop and destroy the concurrency benefit."

That answer demonstrates: gather vs sequential, task vs awaited, async library selection.
Boris will follow up on each of those — know them cold.
