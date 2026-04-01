*The Python Testing Script: A Master Guide to Mocking and Patching — Built for engineers who want to write tests that actually prove something.*


# 🎬 Python Mocking & Patching: The Ultimate Master Document

### A Complete, Professional-Grade Guide to Test Isolation in Python

> **“A test is only as good as its isolation. Master the mock, master the test.”**

-----

## 📋 Table of Contents

- [Introduction: The Movie Set Philosophy](#introduction-the-movie-set-philosophy)
- [The Test Doubles Taxonomy](#the-test-doubles-taxonomy)
- [Module 1: Mock vs. MagicMock — The Basics](#module-1-mock-vs-magicmock--the-basics)
- [Module 2: The mocker Fixture — pytest-mock’s Power Tool](#module-2-the-mocker-fixture--pytest-mocks-power-tool)
- [Module 3: Strictness Control — spec & autospec](#module-3-strictness-control--spec--autospec)
- [Module 4: AsyncMock — Testing the Async World](#module-4-asyncmock--testing-the-async-world)
- [Module 5: The Patching Namespace — Where to Patch](#module-5-the-patching-namespace--where-to-patch)
- [Module 6: Scripting Behavior — return_value vs side_effect](#module-6-scripting-behavior--return_value-vs-side_effect)
- [Module 7: Multiple Return Values & side_effect Exhaustion](#module-7-multiple-return-values--side_effect-exhaustion)
- [Module 8: Assert Methods — Verifying Your Mocks](#module-8-assert-methods--verifying-your-mocks)
- [Module 9: Spying with wraps](#module-9-spying-with-wraps)
- [Module 10: Monkeypatching — The pytest Surgical Tool](#module-10-monkeypatching--the-pytest-surgical-tool)
- [Module 11: mock_open — Testing File I/O](#module-11-mock_open--testing-file-io)
- [Module 12: Mocking Third-Party Services](#module-12-mocking-third-party-services)
- [Module 13: Mock Hell — Anti-Patterns & How to Escape](#module-13-mock-hell--anti-patterns--how-to-escape)
- [Module 14: Dependency Injection for Testability](#module-14-dependency-injection-for-testability)
- [Module 15: Comparison Tables](#module-15-comparison-tables)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

-----

## Introduction: The Movie Set Philosophy

Before writing a single line of code, we need a mental model. Welcome to **The Movie Set Analogy** — the framework that makes every mocking concept click.

|Role                         |Testing Equivalent                |What It Does                                   |
|-----------------------------|----------------------------------|-----------------------------------------------|
|🎬 **Director**               |`patch()` / `mocker.patch()`      |Controls *what* gets replaced and *when*       |
|🦸 **Lead Actor**             |Real production code              |The thing you’re actually testing              |
|🥋 **Stunt Double**           |`Mock` / `MagicMock` / `AsyncMock`|A safe stand-in that won’t break the scene     |
|📝 **Script Revision**        |`return_value` / `side_effect`    |Tells the stunt double how to perform          |
|🔍 **On-Set Documentary Crew**|`wraps` parameter                 |Watches without replacing real performance     |
|✂️ **Quick Script Change**    |`monkeypatch` fixture             |A last-minute line change while filming is live|
|🕵️ **Continuity Supervisor**  |Assert methods                    |Checks every call was made as scripted         |

-----

## The Test Doubles Taxonomy

> **This is the section most guides skip — and it causes the most confusion.**

“Mock” is actually an umbrella term. There are five distinct types of test doubles, and knowing which one to reach for is the foundation of good test design. Using a Mock when you need a Fake, or a Stub when you need a Spy, leads directly to fragile, over-coupled tests.

|Double   |What It Is                                             |When to Use                                                             |
|---------|-------------------------------------------------------|------------------------------------------------------------------------|
|**Dummy**|An object passed around but never used                 |Filling required parameters that don’t affect the test                  |
|**Stub** |Returns pre-programmed responses; does nothing else    |Replacing a dependency to control what it returns (e.g. `return_value`) |
|**Spy**  |Records calls while delegating to the real impl        |Observing interactions without replacing behavior (`wraps=`)            |
|**Mock** |A stub that also *verifies* it was called correctly    |When the interaction itself (not just the result) is what you’re testing|
|**Fake** |A simplified working implementation (e.g. in-memory DB)|When you want realistic behavior without real infrastructure            |


> **💡 Expert Tip:** When you use `patch()` with `return_value`, you’re technically creating a **Stub** (controlling outputs). When you then use `assert_called_with()`, you’ve made it a **Mock** (verifying interactions). Python’s `Mock` class can play any role — but being intentional about which role you’re casting it in keeps your tests clean.

> **⚠️ Danger Zone: Mocks are NOT Stubs** — This is the single most important distinction in test design. A stub makes tests pass by controlling inputs. A mock makes tests *meaningful* by verifying behavior. Conflating them is the root cause of most “Mock Hell” scenarios (covered in Module 13).

-----

## Module 1: Mock vs. MagicMock — The Basics

### 📖 Plain English Definition

A **`Mock`** is a blank, programmable impersonator. You hand it to your code in place of a real object, and it silently accepts any attribute access or method call — recording everything for inspection. A **`MagicMock`** is the same thing, but pre-wired with support for Python’s special “dunder” methods (`__len__`, `__iter__`, `__getitem__`, `__enter__`, `__exit__`), making it compatible with containers, context managers, and other protocol-heavy objects.

### 🛠️ Ways to Use This

- **Mocking a simple service call**: Replace a `PaymentService` with a `Mock` to verify `.charge()` is called without charging anyone.
- **Mocking a database result set**: Use `MagicMock` when your code does `for row in db_cursor:` — the `__iter__` protocol works out of the box.
- **Mocking a context manager**: `MagicMock` supports `with open(...) as f:` automatically via `__enter__` and `__exit__`.

### 🎬 Movie Set Analogy

A `Mock` is a **generic stunt double** — they can stand on set and take direction, but if the scene requires juggling or speaking fluent French, you need a specialist. A `MagicMock` is the **full-package performer**, pre-trained in every “special skill” Python’s data model requires.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module1_mock_vs_magicmock.py
# PURPOSE: Understand when to reach for Mock vs. MagicMock
# ============================================================

from unittest.mock import Mock, MagicMock


# --- SECTION 1: The Simple Mock ---
# Use Mock when the real object has no dunder/protocol magic.
# Think: service clients, API wrappers, simple utility classes.

def send_welcome_email(email_service, user_email):
    """A simple function that depends on an external email service."""
    email_service.send(to=user_email, subject="Welcome!")


def test_send_welcome_email_calls_service():
    # A blank impersonator — no special skills needed.
    mock_service = Mock()

    send_welcome_email(mock_service, "user@example.com")

    # .assert_called_once_with() is the key payoff: it proves our function
    # called .send() with the RIGHT arguments, not just any arguments.
    mock_service.send.assert_called_once_with(to="user@example.com", subject="Welcome!")


# --- SECTION 2: Why MagicMock? Enter the Dunder Methods ---
# Dunder methods (Double UNDERscore) are Python's protocol hooks:
# __len__ → len(), __iter__ → for loops, __getitem__ → []
# A plain Mock does NOT pre-configure these. MagicMock does.

def count_active_users(user_repository):
    """Returns how many users are in the repository."""
    # len() internally calls user_repository.__len__()
    # Plain Mock raises: TypeError: object of type 'Mock' has no len()
    return len(user_repository)


def test_count_with_magicmock():
    mock_repo = MagicMock()

    # Tell the mock what __len__ should return.
    mock_repo.__len__.return_value = 42

    result = count_active_users(mock_repo)
    assert result == 42


# --- SECTION 3: MagicMock as a Context Manager ---
# `with some_object as x:` requires __enter__ and __exit__.
# MagicMock handles both automatically — crucial for file I/O.

def read_config(file_path):
    """Reads a config file and returns its content."""
    with open(file_path) as f:
        return f.read()


def test_read_config(mocker):
    # patch builtins.open — MagicMock supports the context manager protocol.
    mock_open = mocker.patch("builtins.open", MagicMock())

    # `mock_open.return_value.__enter__.return_value` is the `f` in `with open() as f`
    mock_open.return_value.__enter__.return_value.read.return_value = "theme: dark"

    result = read_config("config.yaml")
    assert result == "theme: dark"


# --- SECTION 4: Plain Mock FAILS on Dunder Methods ---
# This is the most important distinction. Knowing it saves hours of debugging.

def test_why_plain_mock_fails_on_len():
    plain_mock = Mock()

    try:
        # TypeError: Plain Mock doesn't pre-configure __len__
        length = len(plain_mock)
    except TypeError as e:
        print(f"Confirmed: Plain Mock fails with: {e}")

    # THE FIX: Use MagicMock
    magic_mock = MagicMock()
    magic_mock.__len__.return_value = 5
    assert len(magic_mock) == 5


# --- SECTION 5: pytest-mock defaults to MagicMock ---
# When you use mocker.patch(), it returns a MagicMock by default.
# You can override this with new_callable= if you need a different type.

def test_mocker_patch_default_type(mocker):
    # mocker.patch() returns MagicMock unless you specify new_callable
    mock_obj = mocker.patch("os.path.exists")
    assert isinstance(mock_obj, MagicMock)  # ✅ Confirmed: it's a MagicMock
```

> **💡 Expert Tip:** Default to `MagicMock` — it’s a strict superset of `Mock`. The only reason to use plain `Mock` is when you *want* to explicitly signal that no magic method protocols are needed, making your test intent clearer to future readers.

> **⚠️ Danger Zone:** Never rely on a `Mock` silently swallowing an unexpected attribute access as confirmation everything is fine. `mock.any_attr_ever` always returns another `Mock` without error. This is why `spec`/`autospec` (Module 3) exists.

-----

## Module 2: The `mocker` Fixture — pytest-mock’s Power Tool

### 📖 Plain English Definition

The **`mocker`** fixture is provided by the `pytest-mock` plugin. It wraps Python’s `unittest.mock` library with a cleaner, pytest-native interface. Instead of manually starting and stopping patches (or managing context managers), `mocker` handles all cleanup automatically when the test ends. It’s injected into your test via pytest’s dependency injection — just add `mocker` as a parameter.

### 🛠️ Ways to Use This

- **`mocker.patch()`**: Replace any importable name with a `MagicMock` for the duration of the test.
- **`mocker.patch.object()`**: Replace a method on a specific object instance rather than a whole module-level name.
- **`mocker.MagicMock()` / `mocker.Mock()`**: Create standalone mock objects without patching anything.

### 🎬 Movie Set Analogy

The `mocker` fixture is your **on-set props manager**. You don’t have to personally track every prop swap and restore it after filming. You hand the list to the props manager at the start of the scene, they make all the substitutions, and they put everything back exactly as it was the moment the director yells “Cut!” — automatically, every time.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module2_mocker_fixture.py
# PURPOSE: Master the mocker fixture — pytest's mocking interface
# REQUIRES: pip install pytest-mock
# ============================================================

import pytest
import requests


# If you get "fixture 'mocker' not found" — the fix is almost always:
#   pip install pytest-mock
# The mocker fixture comes from the pytest-mock plugin. It does NOT
# need to be imported. pytest injects it automatically once installed.


# --- SECTION 1: mocker.patch() — The Workhorse ---

def get_user_data(user_id: int) -> dict:
    """Calls an external API to get user data."""
    response = requests.get(f"https://api.example.com/users/{user_id}")
    return response.json()


def test_get_user_data(mocker):
    # mocker.patch() patches the name, returns a MagicMock,
    # AND automatically restores the original after the test.
    # No `with` block, no manual cleanup — mocker handles it.
    mock_get = mocker.patch("requests.get")

    # Configure the mock's response chain: .get() → .json() → our data
    mock_get.return_value.json.return_value = {"id": 1, "name": "Alice"}

    result = get_user_data(1)

    # Verify our function used the API correctly
    mock_get.assert_called_once_with("https://api.example.com/users/1")
    assert result == {"id": 1, "name": "Alice"}


# --- SECTION 2: mocker.patch.object() — Patching a Specific Instance ---
# Use this when you have an actual object instance and want to patch
# one specific method on it, leaving the rest of the object intact.

class EmailService:
    def send(self, to: str, subject: str, body: str) -> bool:
        """Real implementation would call an SMTP server."""
        pass

    def validate_address(self, email: str) -> bool:
        """Real implementation would validate the format."""
        return "@" in email


def test_patch_object_single_method(mocker):
    service = EmailService()

    # Patch ONLY the `send` method on this specific instance.
    # `validate_address` remains the real implementation.
    mock_send = mocker.patch.object(service, "send", return_value=True)

    service.send(to="alice@example.com", subject="Hi", body="Hello")

    # Verify the mock was called correctly
    mock_send.assert_called_once_with(
        to="alice@example.com", subject="Hi", body="Hello"
    )

    # The real validate_address still works — we didn't replace it
    assert service.validate_address("alice@example.com") is True


# --- SECTION 3: mocker.patch.multiple() — Patch Several Names at Once ---
# More efficient than stacking multiple @patch decorators.

import os

def get_system_info() -> dict:
    """Reads system information from environment and OS."""
    return {
        "user": os.environ.get("USER", "unknown"),
        "platform": os.name,
        "cwd": os.getcwd(),
    }


def test_patch_multiple(mocker):
    # Patch multiple names in one call.
    # Values are applied as keyword arguments: name=mock_value
    mocker.patch.multiple(
        "os",
        name="nt",       # Simulates Windows
        getcwd=mocker.MagicMock(return_value="/mock/path"),
    )

    result = get_system_info()
    assert result["cwd"] == "/mock/path"


# --- SECTION 4: mocker.patch.dict() — Patching Dictionaries ---
# Safe way to temporarily modify a dictionary, like os.environ.

def get_debug_flag() -> bool:
    """Reads the DEBUG flag from environment variables."""
    return os.environ.get("DEBUG", "false").lower() == "true"


def test_patch_dict_env(mocker):
    # Temporarily inject DEBUG=true into os.environ for this test.
    # After the test, os.environ is restored to its original state.
    mocker.patch.dict(os.environ, {"DEBUG": "true"})

    assert get_debug_flag() is True


# --- SECTION 5: mocker.MagicMock() — Creating Standalone Mocks ---
# Sometimes you don't need to patch anything — you just need a mock
# object to pass as an argument to the function under test.

def process_order(payment_gateway, order: dict) -> str:
    """Processes an order using the provided payment gateway."""
    result = payment_gateway.charge(
        amount=order["total"],
        currency=order["currency"]
    )
    return "success" if result else "failed"


def test_process_order(mocker):
    # Create a standalone mock — no patching needed.
    # We're testing process_order's logic, and we control the gateway.
    mock_gateway = mocker.MagicMock()
    mock_gateway.charge.return_value = True

    result = process_order(mock_gateway, {"total": 99.99, "currency": "USD"})

    assert result == "success"
    mock_gateway.charge.assert_called_once_with(amount=99.99, currency="USD")


# --- SECTION 6: new_callable — Specifying the Mock Type ---
# mocker.patch() defaults to MagicMock. Use new_callable to override.

import asyncio

async def fetch_data():
    """An async function that we want to mock."""
    pass


def test_patch_with_new_callable(mocker):
    from unittest.mock import AsyncMock

    # Without new_callable=AsyncMock, patching an async function
    # gives you a MagicMock that can't be awaited.
    mock_fetch = mocker.patch(
        "__main__.fetch_data",
        new_callable=AsyncMock,
        return_value={"data": "mocked"}
    )

    # Now it returns a proper coroutine when called.
    assert isinstance(mock_fetch, AsyncMock)
```

> **💡 Expert Tip:** `mocker.patch()` is functionally identical to `unittest.mock.patch()` but with automatic cleanup — you never need to call `.stop()` or worry about teardown. This is the primary reason to prefer `mocker` over `patch()` in pytest projects.

> **⚠️ Danger Zone:** The `fixture 'mocker' not found` error has exactly one root cause 95% of the time: `pytest-mock` is not installed. Run `pip install pytest-mock` and add it to your `requirements.txt`. Other causes include: misspelling `mocker` as `mock`, trying to explicitly import it (don’t — pytest injects it), or running tests from a virtual environment that doesn’t have the plugin installed.

-----

## Module 3: Strictness Control — spec & autospec

### 📖 Plain English Definition

By default, a `Mock` responds to *any* attribute or method call — even ones that don’t exist on the real object. **`spec`** and **`autospec`** are guardrails that lock a mock to the actual interface of the real class. If your test code accidentally calls a method that doesn’t exist in production, the mock raises `AttributeError` immediately — catching the bug at test-time, not in production.

### 🛠️ Ways to Use This

- **Preventing typo-induced silent failures**: If your real class has `.connect()` but you call `.conect()`, `autospec` catches it instantly. Plain `Mock` silently accepts it.
- **Enforcing correct method signatures**: `autospec` validates that mocked methods are called with the correct number and names of arguments, mirroring the real function’s signature.
- **Documenting the interface boundary**: `spec=RealClass` communicates clearly: “this mock is supposed to behave like this specific class.”

### 🎬 Movie Set Analogy

A plain `Mock` is a stunt double with **no script constraints** — they’ll do anything you ask, even things physically impossible for the character. A `spec`’d mock has **read the full script**: they know what the character can and cannot do, and will refuse loudly if you ask for something out-of-character.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module3_spec_autospec.py
# PURPOSE: Use spec and autospec to prevent silent test failures
# ============================================================

from unittest.mock import Mock, create_autospec, patch
import pytest


class DatabaseConnection:
    """A real class with a defined interface."""

    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port

    def connect(self) -> bool:
        pass

    def query(self, sql: str, params: tuple = ()) -> list:
        pass

    def disconnect(self):
        pass


# --- SECTION 1: The "Silent Failure" Problem ---
# This test PASSES but is WRONG — it gives false confidence.

def test_demonstrates_silent_failure():
    plain_mock = Mock()

    # BUG: The real method is `connect()`, not `conect()`.
    # Plain Mock silently creates `conect` as a new fake attribute.
    plain_mock.conect()  # Should error — does NOT!

    # This passes, but it's a lie. Production code would blow up.
    plain_mock.conect.assert_called_once()


# --- SECTION 2: spec= Catches Attribute Errors ---

def test_spec_catches_typos():
    # spec=DatabaseConnection: only allow attributes that actually exist.
    specced_mock = Mock(spec=DatabaseConnection)

    # Typo immediately raises AttributeError — caught at test-time.
    with pytest.raises(AttributeError):
        specced_mock.conect()

    # The correct method works fine.
    specced_mock.connect()


# --- SECTION 3: create_autospec — The Gold Standard ---
# spec= checks attribute NAMES but NOT method SIGNATURES.
# create_autospec validates both names AND call signatures.

def test_autospec_validates_signatures():
    # instance=True: mocking an instance, not the class itself.
    auto_mock = create_autospec(DatabaseConnection, instance=True)

    # Correct call — passes.
    auto_mock.query("SELECT * FROM users", params=())

    # Wrong call — `query()` doesn't accept a `limit` keyword.
    # create_autospec enforces the real signature.
    with pytest.raises(TypeError):
        auto_mock.query("SELECT *", limit=10)


# --- SECTION 4: autospec=True via patch() — The Idiomatic Pattern ---

class UserService:
    def get_user(self, db: DatabaseConnection, user_id: int):
        db.connect()
        rows = db.query("SELECT * FROM users WHERE id = ?", params=(user_id,))
        db.disconnect()
        return rows[0] if rows else None


def test_get_user_with_autospec(mocker):
    # autospec=True on patch() applies full signature validation automatically.
    # This is the most idiomatic, professional pattern.
    mock_db = mocker.patch(
        "__main__.DatabaseConnection",
        autospec=True
    )

    mock_db.return_value.query.return_value = [{"id": 1, "name": "Alice"}]

    service = UserService()
    db_instance = DatabaseConnection("localhost", 5432)
    result = service.get_user(db_instance, user_id=1)

    # Verify method call order and arguments
    db_instance.connect.assert_called_once()
    db_instance.query.assert_called_once_with(
        "SELECT * FROM users WHERE id = ?", params=(1,)
    )
    db_instance.disconnect.assert_called_once()
    assert result == {"id": 1, "name": "Alice"}
```

> **💡 Expert Tip:** Make `autospec=True` your default. The tiny extra setup is worth catching interface drift — when your production class changes but your mocks don’t. This is one of the most insidious sources of “tests pass, production breaks” bugs.

> **⚠️ Danger Zone:** `spec=` only validates attribute *names*. It does **not** validate call signatures. Only `create_autospec` / `autospec=True` validates both. Don’t mistake one for the other — `spec=` gives you about 50% of the protection you think it does.

-----

## Module 4: AsyncMock — Testing the Async World

### 📖 Plain English Definition

When Python runs `await some_function()`, it expects a **coroutine** — a special object the event loop can pause and resume. A regular `Mock()` returns a plain value, not a coroutine, causing `TypeError` the moment your `async` function tries to `await` it. **`AsyncMock`** is specifically designed to return a coroutine when called, making it fully compatible with `async/await` code.

### 🛠️ Ways to Use This

- **Mocking async API clients**: `async def fetch_weather(client):` awaits `client.get(url)` — use `AsyncMock` so the `await` resolves cleanly.
- **Simulating async database failures**: `AsyncMock(side_effect=TimeoutError())` tests your retry logic without a real database.
- **Testing async context managers**: `AsyncMock` supports `__aenter__` and `__aexit__` for `async with` blocks.

### 🎬 Movie Set Analogy

Regular `Mock` is a stunt double trained for **synchronous scenes**. But async code is a **wire-work flying scene**: the stunt double needs special rigging (the event loop) to even be on set. `AsyncMock` comes pre-rigged — they plug directly into the event loop without breaking the scene.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module4_asyncmock.py
# PURPOSE: Test async/await functions correctly
# REQUIRES: pip install pytest-asyncio
# ============================================================

import pytest
from unittest.mock import AsyncMock, MagicMock, patch


class AsyncWeatherClient:
    async def get_temperature(self, city: str) -> dict:
        pass  # Real async HTTP call in production.


async def get_weather_report(client: AsyncWeatherClient, city: str) -> str:
    """Our real async function under test."""
    data = await client.get_temperature(city)
    temp = data.get("temp_c", "unknown")
    return f"The temperature in {city} is {temp}°C"


# --- SECTION 1: Why Regular Mock FAILS in Async Code ---

@pytest.mark.asyncio
async def test_why_regular_mock_fails():
    """Documents the failure mode — confirms our understanding."""
    regular_mock = MagicMock()

    try:
        # MagicMock().get_temperature() returns a MagicMock, NOT a coroutine.
        # `await` on a non-coroutine raises TypeError.
        await get_weather_report(regular_mock, "Paris")
    except TypeError as e:
        # TypeError: object MagicMock can't be used in 'await' expression
        print(f"Confirmed failure: {e}")


# --- SECTION 2: The Correct Fix — AsyncMock ---

@pytest.mark.asyncio
async def test_get_weather_report_success():
    # AsyncMock returns a coroutine when called — fully event-loop compatible.
    mock_client = AsyncMock()
    mock_client.get_temperature.return_value = {"temp_c": 22, "condition": "sunny"}

    result = await get_weather_report(mock_client, "Paris")

    assert result == "The temperature in Paris is 22°C"

    # Use assert_awaited_once_with(), NOT assert_called_once_with().
    # `awaited` confirms the coroutine actually ran, not just that it was invoked.
    # An un-awaited coroutine is a silent bug Python warns about but won't stop.
    mock_client.get_temperature.assert_awaited_once_with("Paris")


# --- SECTION 3: AsyncMock with side_effect for Error Testing ---

@pytest.mark.asyncio
async def test_handles_network_error():
    mock_client = AsyncMock()
    # side_effect on AsyncMock causes the coroutine to RAISE when awaited.
    mock_client.get_temperature.side_effect = ConnectionError("Network unreachable")

    with pytest.raises(ConnectionError, match="Network unreachable"):
        await get_weather_report(mock_client, "London")


# --- SECTION 4: Patching Async Methods with new_callable=AsyncMock ---

@pytest.mark.asyncio
async def test_patching_async_with_patch_object():
    # Always use new_callable=AsyncMock when patching async methods.
    # Python 3.8+ auto-detects async functions, but being explicit is safer.
    with patch.object(
        AsyncWeatherClient,
        "get_temperature",
        new_callable=AsyncMock,
        return_value={"temp_c": -5}
    ) as mock_method:
        client = AsyncWeatherClient()
        result = await get_weather_report(client, "Oslo")

        assert result == "The temperature in Oslo is -5°C"
        mock_method.assert_awaited_once_with("Oslo")


# --- SECTION 5: AsyncMock as a Context Manager (async with) ---

@pytest.mark.asyncio
async def test_async_context_manager():
    """Test code that uses `async with` — e.g., an async DB session."""
    mock_session = AsyncMock()

    # __aenter__ returns what `session` is bound to in `async with ... as session`
    mock_session.__aenter__.return_value = mock_session
    mock_session.execute.return_value = [{"id": 1}]

    async with mock_session as session:
        rows = await session.execute("SELECT 1")
        assert rows == [{"id": 1}]

    # Verify the context manager lifecycle was respected
    mock_session.__aenter__.assert_awaited_once()
    mock_session.__aexit__.assert_awaited_once()


# --- SECTION 6: Mocking Celery Tasks (Async Task Queues) ---
# Celery uses .delay() and .apply_async() to submit tasks to a broker.
# We mock these methods so tests don't need RabbitMQ/Redis running.

# tasks.py (production code):
# @app.task
# def reverse_string(s: str) -> str:
#     return s[::-1]

def test_mock_celery_delay(mocker):
    """Test that a Celery task is dispatched with the correct argument."""
    # Patch the .delay() method — the mechanism that sends to the broker.
    # This eliminates the need for a running message broker in tests.
    mock_delay = mocker.patch("tasks.reverse_string.delay", return_value="olleh")

    # Simulate what your application code does when it triggers the task.
    from tasks import reverse_string
    result = reverse_string.delay("hello")

    # Verify the task was queued with the correct argument
    mock_delay.assert_called_once_with("hello")
    assert result == "olleh"


def test_mock_celery_apply_async(mocker):
    """Test apply_async for tasks that use advanced calling options."""
    # apply_async offers more control (countdown, eta, queue selection).
    mock_apply = mocker.patch("tasks.add.apply_async", autospec=True)

    from tasks import add
    add.apply_async(args=[5, 7])

    # Verify the method was called with the expected positional args
    mock_apply.assert_called_once_with(args=[5, 7])
```

> **💡 Expert Tip:** Use `assert_awaited_once_with()` (not `assert_called_once_with()`) for async mocks. An un-awaited coroutine is a silent bug — Python will warn about it but won’t stop your test from passing. The `awaited` assertions are the only way to confirm the coroutine actually executed.

> **⚠️ Danger Zone:** Forgetting `@pytest.mark.asyncio` causes async tests to silently succeed without executing the async body — every assertion passes vacuously. If your async tests seem to pass too easily, check for this first.

-----

## Module 5: The Patching Namespace — Where to Patch

### 📖 Plain English Definition

Patching works by replacing a **name** in a **namespace**. When you `import requests` in your module, Python creates the name `requests` in *your module’s* namespace — a local alias. When patching, you must replace that local alias, not the original in the `requests` library. The rule: **patch where the name is used, not where it is defined**.

### 🛠️ Ways to Use This

- **Patching a top-level import**: Module does `import os` → patch `your_module.os`.
- **Patching a direct import**: Module does `from datetime import datetime` → patch `your_module.datetime`.
- **Patching a class method**: Module does `from mylib import MyClass` → patch `your_module.MyClass.method_name`.

### 🎬 Movie Set Analogy

Python’s import system is a **film crew directory**. When the production office (your module) hires a stunt driver (imports `requests`), they get that driver’s number in *their own contact list*. To swap in a different driver, the Director must update **the production office’s contact list**, not the talent agency’s database. Patching the source is like calling the agency — the production office still has the original number.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: payment_processor.py (Production Code)
# ============================================================

import requests
from datetime import datetime


def charge_customer(amount: float, card_token: str) -> dict:
    response = requests.post(
        "https://api.payments.com/charge",
        json={"amount": amount, "token": card_token}
    )
    return response.json()


def get_current_timestamp() -> str:
    return datetime.now().isoformat()
```

```python
# ============================================================
# FILE: test_module5_patching_namespace.py
# ============================================================

from unittest.mock import patch, MagicMock


# --- SECTION 1: The WRONG Way (Common Beginner Mistake) ---

def test_WRONG_patching_the_source():
    # ❌ WRONG: Patching at the SOURCE module.
    # payment_processor.py already has its own reference to `requests`.
    # Patching the source library doesn't affect the reference in our module.
    with patch("requests.post") as mock_post:
        # This OFTEN won't intercept calls in payment_processor.py
        pass


# --- SECTION 2: The RIGHT Way — Patch the Destination ---

def test_CORRECT_patching_the_destination():
    # ✅ CORRECT: Patch `requests` as it exists in payment_processor's namespace.
    # Format: "module_where_name_is_USED.name_to_replace"
    with patch("payment_processor.requests") as mock_requests:
        mock_response = MagicMock()
        mock_response.json.return_value = {"status": "charged", "id": "ch_123"}
        mock_requests.post.return_value = mock_response

        from payment_processor import charge_customer
        result = charge_customer(29.99, "tok_visa_abc")

        mock_requests.post.assert_called_once_with(
            "https://api.payments.com/charge",
            json={"amount": 29.99, "token": "tok_visa_abc"}
        )
        assert result == {"status": "charged", "id": "ch_123"}


# --- SECTION 3: Patching a `from X import Y` Style Import ---
# `from datetime import datetime` creates a NEW local name `datetime`
# in payment_processor's namespace. Patch THAT local name.

def test_patching_from_import():
    # ✅ CORRECT: Target the local name inside payment_processor.
    with patch("payment_processor.datetime") as mock_datetime:
        # Frozen clock — deterministic time for tests.
        mock_datetime.now.return_value.isoformat.return_value = "2024-01-15T10:30:00"

        from payment_processor import get_current_timestamp
        result = get_current_timestamp()

        assert result == "2024-01-15T10:30:00"


# --- SECTION 4: Stacking @patch Decorators ---
# Applied bottom-up. Arguments arrive in the REVERSE decorator order.

@patch("payment_processor.requests")   # Applied second (outer) → arg 2nd
@patch("payment_processor.datetime")   # Applied first (inner) → arg 1st
def test_multiple_patches(mock_datetime, mock_requests):
    # ⚠️ Bottom decorator's mock is the FIRST function argument.
    # This trips up many developers — always match order carefully.

    mock_datetime.now.return_value.isoformat.return_value = "2024-06-01T12:00:00"
    mock_requests.post.return_value.json.return_value = {"status": "ok"}

    from payment_processor import charge_customer
    result = charge_customer(9.99, "tok_amex_xyz")

    assert result == {"status": "ok"}


# --- SECTION 5: patch.object — Patching a Specific Method on an Object ---
# Use this when you have an actual instance and only want to replace
# one method, leaving the rest of the object intact.

class PaymentProcessor:
    def process_payment(self, amount, currency="USD"):
        return True

    def validate_card(self, token: str) -> bool:
        return len(token) > 5


def test_patch_object_method(mocker):
    processor = PaymentProcessor()

    # Patch ONLY process_payment — validate_card remains real.
    mock_process = mocker.patch.object(
        processor, "process_payment", return_value=False
    )

    result = processor.process_payment(100)
    assert result is False  # Mock returned False

    # validate_card is NOT mocked — real implementation still runs
    assert processor.validate_card("tok_abc123") is True
```

> **💡 Expert Tip:** When in doubt, inspect `your_module.__dict__` in a Python shell to see the exact names in that module’s namespace. Whatever name you see there is exactly what to patch. This removes all guesswork.

> **⚠️ Danger Zone:** The single most common mocking bug is patching `library.function` instead of `your_module.function`. If mocks mysteriously don’t take effect, this is the *first* thing to check — before anything else.

-----

## Module 6: Scripting Behavior — `return_value` vs `side_effect`

### 📖 Plain English Definition

**`return_value`** is the simplest directive: “always return this exact thing.” It’s static. **`side_effect`** is a power tool: it can raise exceptions, call a real function, or return different values on each successive call. When `side_effect` is set, it **overrides** `return_value` completely.

### 🛠️ Ways to Use This

- **Static API response** (`return_value`): Always return the same mocked response.
- **Simulating failure** (`side_effect=Exception`): Test your retry and fallback logic.
- **Paginated responses** (`side_effect=[page1, page2, page3]`): Different data on each call.
- **Argument-dependent responses** (`side_effect=callable`): Return different data based on what the mock was called with.

### 🎬 Movie Set Analogy

`return_value` is a **pre-filmed cutaway shot**: the same clip every time. `side_effect` is a **live improv actor**: each cue can produce something completely different — deliver a line, throw a punch, or collapse dramatically.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module6_return_value_vs_side_effect.py
# ============================================================

import pytest
from unittest.mock import Mock, call


# --- SECTION 1: return_value — The Static Response ---

def test_return_value_static():
    mock_api = Mock()
    mock_api.get_user.return_value = {"user_id": 42, "name": "Alice"}

    # Same result every call — it's a fixed pre-filmed shot.
    assert mock_api.get_user(42) == {"user_id": 42, "name": "Alice"}
    assert mock_api.get_user(42) == {"user_id": 42, "name": "Alice"}


# --- SECTION 2: side_effect with an Exception ---
# Assigning an Exception INSTANCE raises it when the mock is called.
# This is the primary way to test error-handling paths.

class DataFetcher:
    def fetch(self, url: str) -> dict:
        pass

    def fetch_with_retry(self, url: str, retries: int = 3) -> dict:
        for attempt in range(retries):
            try:
                return self.fetch(url)
            except ConnectionError:
                if attempt == retries - 1:
                    raise
        return {}


def test_side_effect_raises_exception():
    fetcher = DataFetcher()
    mock_fetch = Mock()
    mock_fetch.side_effect = ConnectionError("Timeout after 30s")
    fetcher.fetch = mock_fetch

    with pytest.raises(ConnectionError, match="Timeout after 30s"):
        fetcher.fetch_with_retry("https://api.example.com/data", retries=1)


# --- SECTION 3: Exception Class vs Instance in side_effect ---
# You can assign either an exception CLASS or an INSTANCE.
# Class: Mock raises it without a message.
# Instance: Mock raises it WITH your custom message.

def test_exception_class_vs_instance():
    mock_a = Mock(side_effect=ValueError)        # Raises ValueError()
    mock_b = Mock(side_effect=ValueError("bad")) # Raises ValueError("bad")

    with pytest.raises(ValueError):
        mock_a()

    with pytest.raises(ValueError, match="bad"):
        mock_b()


# --- SECTION 4: side_effect with a List — Cycling Through Values ---
# Each call consumes the next item. Exceptions in the list are raised.
# Once exhausted, further calls raise StopIteration.

def test_side_effect_list_for_retry():
    fetcher = DataFetcher()
    mock_fetch = Mock()

    # First call fails, second call succeeds — simulates transient failure.
    mock_fetch.side_effect = [
        ConnectionError("First attempt failed"),  # Call 1: raises
        {"data": "success"}                        # Call 2: returns
    ]

    fetcher.fetch = mock_fetch
    result = fetcher.fetch_with_retry("https://api.example.com/data", retries=2)

    assert result == {"data": "success"}
    assert mock_fetch.call_count == 2  # Confirms retry actually happened


# --- SECTION 5: side_effect Exhaustion → StopIteration ---
# This is critical to understand for robust test writing.

def test_side_effect_exhaustion(mocker):
    mock_iter = mocker.MagicMock()
    mock_iter.side_effect = [1, 2, 3]

    assert mock_iter() == 1
    assert mock_iter() == 2
    assert mock_iter() == 3

    # All values consumed — next call raises StopIteration.
    # Make sure your list is at least as long as the expected call count.
    with pytest.raises(StopIteration):
        mock_iter()


# --- SECTION 6: side_effect with a Callable — Argument-Dependent Responses ---
# The callable receives the same arguments as the mock.
# Return DEFAULT (the sentinel) to fall through to return_value.

def test_side_effect_callable_dynamic():
    mock_db = Mock()

    def dynamic_query(sql: str, params: tuple = ()):
        """Returns different results based on which table is queried."""
        if "users" in sql:
            return [{"id": 1, "name": "Alice"}]
        elif "orders" in sql:
            return [{"order_id": 101, "total": 99.99}]
        return []  # Unknown table

    mock_db.query.side_effect = dynamic_query

    # Lambda shorthand for the same pattern:
    # mock_db.query.side_effect = lambda sql, params=(): {"users": [...]}
    #   .get(next(k for k in {...} if k in sql), [])

    users = mock_db.query("SELECT * FROM users")
    orders = mock_db.query("SELECT * FROM orders")
    unknown = mock_db.query("SELECT * FROM unknown")

    assert users == [{"id": 1, "name": "Alice"}]
    assert orders == [{"order_id": 101, "total": 99.99}]
    assert unknown == []


# --- SECTION 7: side_effect OVERRIDES return_value ---
# When both are set, side_effect always wins.

def test_side_effect_overrides_return_value():
    mock = Mock()
    mock.return_value = "ignored"
    mock.side_effect = ValueError("side_effect wins!")

    with pytest.raises(ValueError, match="side_effect wins!"):
        mock()  # return_value is completely bypassed
```

> **💡 Expert Tip:** For testing multiple return values in sequence, `side_effect=[val1, val2, val3]` is cleaner than anything else. But always ensure the list is at least as long as your test’s call count — a surprise `StopIteration` is hard to debug.

> **⚠️ Danger Zone:** `side_effect` with a lambda that uses a dict lookup (`{"saving": 0.02, "current": 0.01}.get(account_type, 0.005)`) is an elegant pattern for argument-dependent responses. But don’t over-engineer it — if the logic gets complex, consider a proper `Fake` object instead (see Module 13).

-----

## Module 7: Multiple Return Values & side_effect Exhaustion

### 📖 Plain English Definition

Real systems rarely return the same response every time. **Multiple return values** let you simulate sequences — a paginated API, a service that fails once then succeeds, or a function that behaves differently based on what’s called. This module goes deeper on `side_effect` patterns and the exhaustion behavior you must understand for robust tests.

### 🛠️ Ways to Use This

- **Paginated APIs**: Return `page_1_data`, then `page_2_data`, then `page_3_data` on successive calls.
- **Retry logic**: Fail twice, then succeed — verify retry count and final result.
- **State machine simulation**: Different responses at each stage of a workflow.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module7_multiple_return_values.py
# ============================================================

import pytest
from unittest.mock import Mock


# --- SECTION 1: Argument-Based Return Values with a Lambda ---
# Use when the return value should depend on the input argument.

def define_interest_rate(account_type: str) -> float:
    """Returns the interest rate for a given account type."""
    if account_type == "saving":
        return 0.02
    elif account_type == "current":
        return 0.01
    return 0.005


def calculate_interest(account_type: str, balance: float) -> float:
    rate = define_interest_rate(account_type)
    return balance * rate


def test_argument_dependent_side_effect(mocker):
    # Patch define_interest_rate with a lambda that mirrors the real logic
    # but uses a controlled lookup — we're testing calculate_interest, not the rate logic.
    mocker.patch(
        "test_module7_multiple_return_values.define_interest_rate",
        side_effect=lambda account_type: {
            "saving": 0.02,
            "current": 0.01,
        }.get(account_type, 0.005),  # 0.5% default for unknown types
    )

    assert calculate_interest("saving", 1000) == 20.0
    assert calculate_interest("current", 1000) == 10.0
    assert calculate_interest("unknown", 1000) == 5.0


# --- SECTION 2: Sequential Values via List ---
# Each call returns the next item in the list.
# Mixing return values and exceptions in one list is powerful.

def test_mixed_sequential_side_effect():
    mock_service = Mock()

    # Simulates: first call fails, second returns data, third fails again.
    mock_service.fetch.side_effect = [
        ConnectionError("Timeout"),     # Call 1: raises
        {"data": [1, 2, 3]},            # Call 2: returns data
        ConnectionError("Server down"), # Call 3: raises
    ]

    with pytest.raises(ConnectionError, match="Timeout"):
        mock_service.fetch("url")

    assert mock_service.fetch("url") == {"data": [1, 2, 3]}

    with pytest.raises(ConnectionError, match="Server down"):
        mock_service.fetch("url")


# --- SECTION 3: Exhaustion Behavior — What Happens After the List Runs Out ---
# This is a critical edge case. After all values are consumed,
# the next call raises StopIteration. Tests that don't account for
# this will fail with a confusing error.

def test_side_effect_exhaustion_detail():
    mock = Mock()
    mock.side_effect = [10, 20]  # Only 2 values

    assert mock() == 10
    assert mock() == 20

    # 3rd call: StopIteration — the list is exhausted.
    # Make your list match your expected call count exactly.
    with pytest.raises(StopIteration):
        mock()


# --- SECTION 4: Preventing Exhaustion with a Callable ---
# If you need infinite cycling behavior, use itertools.cycle
# or a callable that manages its own state.

from itertools import cycle


def test_infinite_cycling_values():
    mock_sensor = Mock()

    # itertools.cycle repeats the sequence forever — never exhausts.
    # Use when your code may call the mock an indeterminate number of times.
    values = cycle([0.1, 0.5, 0.9])
    mock_sensor.read.side_effect = lambda: next(values)

    # Cycles through 0.1, 0.5, 0.9, 0.1, 0.5, 0.9, ...
    assert mock_sensor.read() == 0.1
    assert mock_sensor.read() == 0.5
    assert mock_sensor.read() == 0.9
    assert mock_sensor.read() == 0.1  # Wraps around
    assert mock_sensor.read() == 0.5


# --- SECTION 5: Weather API Pagination Example ---
# A practical simulation of paginated API responses.

def fetch_all_pages(api_client, endpoint: str) -> list:
    """Fetches all pages from a paginated API."""
    results = []
    page = 1
    while True:
        data = api_client.get(f"{endpoint}?page={page}")
        if not data:
            break
        results.extend(data)
        page += 1
    return results


def test_paginated_api_responses():
    mock_client = Mock()

    # Three pages of data, then empty list to signal end of pagination.
    mock_client.get.side_effect = [
        [{"id": 1}, {"id": 2}],   # Page 1
        [{"id": 3}, {"id": 4}],   # Page 2
        [{"id": 5}],               # Page 3
        [],                        # No more pages — loop terminates
    ]

    results = fetch_all_pages(mock_client, "https://api.example.com/items")

    assert len(results) == 5
    assert results[0] == {"id": 1}
    assert results[4] == {"id": 5}
    assert mock_client.get.call_count == 4
```

> **💡 Expert Tip:** When testing retry logic, a list like `[NetworkError(), NetworkError(), success_data]` precisely documents your test intent: “it should fail twice, then succeed on the third attempt.” This is far more readable than three separate tests.

> **⚠️ Danger Zone:** A silent test failure caused by `StopIteration` propagating into pytest’s machinery can look like a test *passing* in some configurations, rather than erroring. Always count your expected calls and make your `side_effect` list match exactly.

-----

## Module 8: Assert Methods — Verifying Your Mocks

### 📖 Plain English Definition

Creating a mock and calling your code is only half the job. The **assert methods** on mock objects let you verify *how* the mock was used — was it called? How many times? With what arguments? In what order? Without assertions, a mock is just a passive stand-in that proves nothing.

### 🛠️ Ways to Use This

- **`assert_called_once_with()`**: The single most-used assertion — verifies one call with exact arguments.
- **`assert_has_calls()`**: Verifies a sequence of calls in a specific order (critical for workflow tests).
- **`assert_not_called()`**: Verifies a code path was NOT taken (negative testing).

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module8_assert_methods.py
# PURPOSE: Master all mock assertion methods
# ============================================================

import pytest
from unittest.mock import Mock, MagicMock, call


# --- FULL TAXONOMY OF ASSERTION METHODS ---

def test_all_assertion_methods():
    mock = Mock()

    # Make some calls
    mock.method("first_call")
    mock.method("second_call")
    mock.method("third_call")

    # assert_called() — was called at least once (any number, any args)
    mock.method.assert_called()

    # assert_called_once() — was called EXACTLY once (fails here — called 3x)
    # mock.method.assert_called_once()  # Would fail

    # assert_called_with() — checks ONLY the MOST RECENT call
    mock.method.assert_called_with("third_call")  # Last call was "third_call"

    # assert_any_call() — checks that THIS call happened at some point
    mock.method.assert_any_call("first_call")   # Happened at some point ✅
    mock.method.assert_any_call("second_call")  # Happened at some point ✅

    # call_count — total number of calls
    assert mock.method.call_count == 3

    # called — boolean: was it called at all?
    assert mock.method.called is True


def test_assert_not_called():
    mock = Mock()
    # assert_not_called() — verifies the method was NEVER invoked
    mock.some_method.assert_not_called()

    # Use case: verify a cache hit bypasses the database call
    # mock_db.query.assert_not_called()


def test_assert_called_once_with():
    mock = Mock()
    mock.send_email(to="alice@example.com", subject="Hello")

    # assert_called_once_with() — verifies BOTH that it was called exactly
    # once AND that the arguments match. The workhorse assertion.
    mock.send_email.assert_called_once_with(to="alice@example.com", subject="Hello")


# --- SECTION: The Critical Difference Between assert_called_with and assert_any_call ---

def test_called_with_vs_any_call():
    mock = Mock()
    mock.notify("alice@example.com", "Welcome!")
    mock.notify("bob@example.com", "Update!")
    mock.notify("carol@example.com", "Reminder!")

    # assert_called_with() checks the LAST call ONLY:
    mock.notify.assert_called_with("carol@example.com", "Reminder!")
    # This would FAIL even though alice was notified:
    # mock.notify.assert_called_with("alice@example.com", "Welcome!")  # ❌

    # assert_any_call() checks any call in the entire history:
    mock.notify.assert_any_call("alice@example.com", "Welcome!")  # ✅


# --- SECTION: assert_has_calls — Ordered Sequence Verification ---

def test_assert_has_calls_order():
    mock_workflow = Mock()

    # Simulate a multi-step workflow
    mock_workflow.step_one("init")
    mock_workflow.step_two("process")
    mock_workflow.step_three("finalize")

    # assert_has_calls verifies a sequence occurred in ORDER.
    # any_order=False (default): strict sequence required.
    mock_workflow.assert_has_calls([
        call.step_one("init"),
        call.step_two("process"),
        call.step_three("finalize"),
    ], any_order=False)

    # any_order=True: all calls must exist, but order doesn't matter.
    mock_workflow.assert_has_calls([
        call.step_three("finalize"),
        call.step_one("init"),
    ], any_order=True)


# --- SECTION: Inspecting call_args_list Manually ---
# For complex cases where the assert methods aren't expressive enough.

def test_manual_call_inspection():
    mock = Mock()

    mock.process({"id": 1, "status": "pending"})
    mock.process({"id": 2, "status": "active"})
    mock.process({"id": 3, "status": "pending"})

    # Extract all arguments from all calls
    all_args = [c.args[0] for c in mock.process.call_args_list]

    # Find all pending items that were processed
    pending_calls = [a for a in all_args if a["status"] == "pending"]
    assert len(pending_calls) == 2


# --- SECTION: Neglecting Call Order — A Common Mistake ---
# If your code has a required execution sequence (e.g., connect before query),
# not asserting order gives you false confidence.

def test_call_order_matters():
    mock_db = Mock()

    # Simulate what our code should do: connect, then query, then disconnect
    mock_db.connect()
    mock_db.query("SELECT 1")
    mock_db.disconnect()

    # Without call order assertion, this test passes even if the order was wrong:
    mock_db.connect.assert_called_once()
    mock_db.query.assert_called_once_with("SELECT 1")
    mock_db.disconnect.assert_called_once()

    # WITH call order assertion — catches ordering bugs:
    mock_db.assert_has_calls([
        call.connect(),
        call.query("SELECT 1"),
        call.disconnect(),
    ])


# --- SECTION: reset_mock() — Resetting State Between Reuses ---
# If you reuse a mock across multiple logical sections of one test,
# reset it to clear call history. Better practice: use separate mocks.

def test_reset_mock():
    mock = Mock()

    # First phase
    mock.action("phase_1")
    assert mock.action.call_count == 1

    # Reset — clears call history, return_value, side_effect, children
    mock.reset_mock()

    # Second phase — starts fresh
    assert mock.action.call_count == 0
    mock.action("phase_2")
    mock.action.assert_called_once_with("phase_2")
```

> **💡 Expert Tip:** `assert_called_with()` only checks the **last** call. If you call a mock 5 times and care about the 3rd, use `assert_any_call()` or `call_args_list[2]`. Using `assert_called_with()` on a multi-call mock is a subtle trap that can let bugs through.

> **⚠️ Danger Zone:** Neglecting call order assertions is one of the most common ways to write tests that pass despite incorrect behavior. If your code has a required execution sequence, always use `assert_has_calls()` with `any_order=False`.

-----

## Module 9: Spying with `wraps`

### 📖 Plain English Definition

The `wraps` parameter creates a mock that **delegates to the real implementation** while still recording all call activity. The actual function executes and returns its real result, but you gain full introspection: call count, arguments, exceptions. It’s pure observation — no interference.

### 🛠️ Ways to Use This

- **Verifying a utility function is called correctly**: Confirm `sorted()` is called with the right `key=` argument without replacing its behavior.
- **Auditing internal calls**: Verify how many times an internal helper is called during a complex operation.
- **Characterization tests**: Capture and lock down what existing code actually does before refactoring.

### 🎬 Movie Set Analogy

`wraps` is your **on-set documentary camera**. The Lead Actor performs for real — no stunt double. But a camera crew shadows every move. The performance is authentic; every frame is captured.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module9_wraps.py
# ============================================================

from unittest.mock import Mock, MagicMock, patch, call
import pytest


def calculate_discount(price: float, discount_percent: float) -> float:
    """Real utility function with real business logic."""
    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("Discount must be between 0 and 100.")
    return round(price * (1 - discount_percent / 100), 2)


def apply_bulk_discounts(prices: list, discount_percent: float) -> list:
    return [calculate_discount(p, discount_percent) for p in prices]


# --- SECTION 1: Basic wraps — Observe Without Replacing ---

def test_spy_on_calculate_discount():
    # The real function runs. The spy records the call.
    spy = Mock(wraps=calculate_discount)

    result = spy(100.0, 20.0)

    # Real logic ran — result is accurate
    assert result == 80.0

    # Spy recorded the call — we can assert on it
    spy.assert_called_once_with(100.0, 20.0)


# --- SECTION 2: Real Exceptions Still Propagate Through the Spy ---

def test_spy_propagates_real_exceptions():
    spy = Mock(wraps=calculate_discount)

    with pytest.raises(ValueError, match="Discount must be between 0 and 100"):
        spy(100.0, -10.0)

    # The failed call was still recorded
    spy.assert_called_once_with(100.0, -10.0)


# --- SECTION 3: Spying on Internal Calls via patch() ---
# The most powerful pattern: understand what happens INSIDE a function.

def test_spy_on_internal_calls():
    with patch(
        "__main__.calculate_discount",
        wraps=calculate_discount  # Real function still runs
    ) as spy:
        prices = [100.0, 200.0, 50.0]
        results = apply_bulk_discounts(prices, 10.0)

        # Real calculations happened
        assert results == [90.0, 180.0, 45.0]

        # Now inspect internal behavior
        assert spy.call_count == 3  # Called once per price

        spy.assert_has_calls([
            call(100.0, 10.0),
            call(200.0, 10.0),
            call(50.0, 10.0),
        ])


# --- SECTION 4: wraps Ignores return_value ---
# When wraps is set, return_value is bypassed — the real function's
# return value is always used.

def test_wraps_ignores_return_value():
    spy = Mock(wraps=calculate_discount)
    spy.return_value = 9999  # This will be completely ignored

    result = spy(100.0, 50.0)

    # Real: 100 - 50% = 50.0 — NOT 9999
    assert result == 50.0


# --- SECTION 5: Partial Spy — Wrap Some, Stub Others ---

class PricingEngine:
    def base_price(self, product_id: int) -> float:
        return 100.0  # DB lookup in reality

    def apply_tax(self, price: float, rate: float) -> float:
        return round(price * (1 + rate), 2)  # Real calculation


def test_partial_spy():
    real_engine = PricingEngine()
    spy_engine = MagicMock(wraps=real_engine)

    # Override base_price (stubbing the DB lookup)
    # Setting return_value on a wrapped method overrides the wrap for THAT method.
    spy_engine.base_price.return_value = 200.0

    base = spy_engine.base_price(product_id=42)
    final = spy_engine.apply_tax(base, rate=0.1)

    assert base == 200.0         # Stubbed
    assert final == 220.0        # Real logic: 200 * 1.1
    spy_engine.apply_tax.assert_called_once_with(200.0, rate=0.1)
```

> **💡 Expert Tip:** `wraps` is ideal for “characterization tests” — tests that document and lock down what existing code actually does, before you refactor it. Observe the real behavior, capture it in assertions, then safely change the internals.

> **⚠️ Danger Zone:** Don’t use `wraps` when the real function has network calls, file I/O, or expensive side effects. You *want* a pure mock in those cases. `wraps` is for lightweight, pure-function spying only.

-----

## Module 10: Monkeypatching — The pytest Surgical Tool

### 📖 Plain English Definition

`monkeypatch` is a pytest fixture that provides a **safe, scoped mechanism to directly modify attributes, environment variables, and dictionary entries** during a single test. Unlike `patch()` which replaces names with mock objects, `monkeypatch` directly sets or removes values on real objects. Every change is **automatically reversed** after the test — no manual cleanup.

### 🎬 Movie Set Analogy: The Quick Script Change

You’re mid-scene and the director shouts “Quick — change that line!” The script supervisor (monkeypatch) steps in, makes a surgical edit to the page the actor is holding right now, and the scene continues. After the take, the supervisor quietly restores the original page.

### 🛠️ Ways to Use This

- **`setattr`**: Replace `os.path.exists` with a lambda, testing file-existence branches without touching the filesystem.
- **`setenv`**: Inject `DATABASE_URL=sqlite:///:memory:` without modifying the real OS environment.
- **`delattr`**: Remove a feature flag attribute to test the fallback path.
- **`syspath_prepend`**: Add a path to `sys.path` for import testing.
- **`chdir`**: Change the working directory for tests involving relative paths.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module10_monkeypatch.py
# ============================================================

import os
import sys
import pytest


# --- Production Code Under Test ---

def load_database_url() -> str:
    url = os.environ.get("DATABASE_URL")
    if not url:
        raise EnvironmentError("DATABASE_URL environment variable is not set.")
    return url


def check_file_exists(path: str) -> str:
    return f"File found: {path}" if os.path.exists(path) else f"File missing: {path}"


class FeatureFlags:
    NEW_DASHBOARD = True


def is_new_dashboard_enabled() -> bool:
    return getattr(FeatureFlags, "NEW_DASHBOARD", False)


APP_CONFIG = {"debug_mode": False, "max_connections": 10}


# --- SECTION 1: setenv — Injecting Environment Variables ---

def test_load_db_url_with_setenv(monkeypatch):
    # Sets env var for THIS TEST ONLY. Restored automatically after.
    monkeypatch.setenv("DATABASE_URL", "postgresql://localhost/testdb")
    assert load_database_url() == "postgresql://localhost/testdb"


def test_load_db_url_missing(monkeypatch):
    # raising=False: no error if the var didn't exist to begin with.
    monkeypatch.delenv("DATABASE_URL", raising=False)
    with pytest.raises(EnvironmentError):
        load_database_url()


# --- SECTION 2: setattr — Replacing Functions on Objects ---

def test_file_exists_present(monkeypatch):
    monkeypatch.setattr(os.path, "exists", lambda path: True)
    assert check_file_exists("/any/path.txt") == "File found: /any/path.txt"


def test_file_exists_absent(monkeypatch):
    monkeypatch.setattr(os.path, "exists", lambda path: False)
    assert check_file_exists("/missing.log") == "File missing: /missing.log"


# --- SECTION 3: setattr with Module-Level Function via String Path ---
# Use the string form when you want to target a name in a specific module.

def send_email(to: str, subject: str, body: str):
    pass  # Real SMTP in production


def notify_user(user_email: str, event_name: str):
    send_email(
        to=user_email,
        subject=f"Event: {event_name}",
        body=f"Your event '{event_name}' was triggered."
    )


def test_notify_user_calls_send_email(monkeypatch):
    captured = []

    def fake_send(to, subject, body):
        captured.append({"to": to, "subject": subject})

    # String form: "module_path.attribute_name"
    monkeypatch.setattr("test_module10_monkeypatch.send_email", fake_send)

    notify_user("alice@example.com", "UserSignup")

    assert len(captured) == 1
    assert captured[0]["to"] == "alice@example.com"
    assert captured[0]["subject"] == "Event: UserSignup"


# --- SECTION 4: delattr — Testing Missing Attribute Paths ---

def test_feature_flag_missing(monkeypatch):
    monkeypatch.delattr(FeatureFlags, "NEW_DASHBOARD")
    # getattr(..., False) provides the fallback when attribute is absent
    assert is_new_dashboard_enabled() is False


def test_feature_flag_present(monkeypatch):
    monkeypatch.setattr(FeatureFlags, "NEW_DASHBOARD", True)
    assert is_new_dashboard_enabled() is True


# --- SECTION 5: setitem / delitem — Modifying Dictionaries ---

def is_debug_enabled() -> bool:
    return APP_CONFIG.get("debug_mode", False)


def test_debug_via_setitem(monkeypatch):
    # Safely modifies a dict key for this test, then restores it.
    monkeypatch.setitem(APP_CONFIG, "debug_mode", True)
    assert is_debug_enabled() is True


def test_debug_key_deleted(monkeypatch):
    monkeypatch.delitem(APP_CONFIG, "debug_mode")
    assert is_debug_enabled() is False


# --- SECTION 6: syspath_prepend and chdir ---

def test_syspath_prepend(monkeypatch, tmp_path):
    # Temporarily add a path to sys.path — useful for testing imports
    # from non-standard locations. Restored after test.
    monkeypatch.syspath_prepend(str(tmp_path))
    assert str(tmp_path) == sys.path[0]  # It's at the front


def test_chdir(monkeypatch, tmp_path):
    # Change the current working directory for this test.
    # Useful for functions that use relative paths or os.getcwd().
    monkeypatch.chdir(tmp_path)
    assert os.getcwd() == str(tmp_path)


# --- SECTION 7: Global Monkeypatching — Blocking HTTP Requests ---
# Use autouse=True in conftest.py to apply a monkeypatch across ALL tests.
# This is powerful for ensuring no test accidentally makes real HTTP calls.

# In conftest.py:
# @pytest.fixture(autouse=True)
# def no_requests(monkeypatch):
#     """Block ALL HTTP requests in the entire test suite."""
#     monkeypatch.delattr("requests.sessions.Session.request")
#
# Any test that tries to use requests will now raise AttributeError,
# forcing developers to mock their HTTP calls explicitly.
```

> **💡 Expert Tip:** `monkeypatch` excels at modifying *configuration* (env vars, config dicts, feature flags). `patch()` excels at replacing *dependencies* (services, databases, external I/O). When you’re changing what a value *is*, use `monkeypatch`. When you’re replacing what your code *calls*, use `patch()`.

> **⚠️ Danger Zone:** Avoid patching core Python builtins globally (like `open` or `compile`) with `autouse=True` — it can destabilize pytest’s own internals. Use the global pattern sparingly, and only for clear cross-cutting concerns like blocking HTTP requests.

-----

## Module 11: `mock_open` — Testing File I/O

### 📖 Plain English Definition

`mock_open` is a helper that creates a pre-configured `MagicMock` designed specifically to replace Python’s built-in `open()` function. It correctly simulates the context manager protocol (`with open() as f:`) and the `read()`, `write()`, and `readline()` methods, saving you from manually wiring up `__enter__` and `__exit__` yourself.

### 🛠️ Ways to Use This

- **Testing file reads**: Simulate `f.read()` returning specific content without touching the filesystem.
- **Testing file writes**: Verify `f.write()` was called with the correct data.
- **Testing config loading**: Mock reading a YAML or JSON config file.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module11_mock_open.py
# ============================================================

import pytest
from unittest.mock import mock_open, patch, call


# --- Production Code Under Test ---

def read_config_file(path: str) -> str:
    """Reads a config file and returns its content."""
    with open(path, "r") as f:
        return f.read()


def write_report(path: str, content: str) -> None:
    """Writes content to a report file."""
    with open(path, "w") as f:
        f.write(content)


def count_lines(path: str) -> int:
    """Counts the number of lines in a file."""
    with open(path, "r") as f:
        return len(f.readlines())


# --- SECTION 1: Basic mock_open for Reading ---

def test_read_config_file(mocker):
    # mock_open() creates a MagicMock pre-configured for file operations.
    # The `read_data` parameter is what f.read() will return.
    mock_file = mock_open(read_data="database_url: postgresql://localhost/mydb")

    # Patch builtins.open with our mock
    mocker.patch("builtins.open", mock_file)

    result = read_config_file("config.yaml")

    # Verify the content was returned
    assert result == "database_url: postgresql://localhost/mydb"

    # Verify open() was called with the right path and mode
    mock_file.assert_called_once_with("config.yaml", "r")


# --- SECTION 2: Testing File Writes ---

def test_write_report(mocker):
    mock_file = mock_open()
    mocker.patch("builtins.open", mock_file)

    write_report("report.txt", "Monthly Summary\nTotal: $9,999")

    # Verify file was opened for writing
    mock_file.assert_called_once_with("report.txt", "w")

    # Verify the correct content was written
    # mock_open().write is tracked on the handle returned by __enter__
    mock_file().write.assert_called_once_with("Monthly Summary\nTotal: $9,999")


# --- SECTION 3: Testing readline() ---

def test_count_lines(mocker):
    # For readlines(), configure the mock to return a list of lines
    mock_file = mock_open(read_data="line1\nline2\nline3\n")
    mocker.patch("builtins.open", mock_file)

    # readlines() is pre-configured by mock_open to split on newlines
    result = count_lines("data.txt")
    assert result == 3


# --- SECTION 4: File Not Found — Testing Error Handling ---

def safe_read(path: str) -> str:
    """Reads a file, returning empty string if not found."""
    try:
        with open(path, "r") as f:
            return f.read()
    except FileNotFoundError:
        return ""


def test_file_not_found(mocker):
    # Make open() raise FileNotFoundError to test the error path
    mocker.patch("builtins.open", side_effect=FileNotFoundError)

    result = safe_read("nonexistent.txt")
    assert result == ""
```

> **💡 Expert Tip:** `mock_open` is the cleanest way to test file I/O. The alternative — creating real temporary files — is slower and leaves cleanup concerns. Reserve real file I/O for integration tests; unit tests should mock it.

-----

## Module 12: Mocking Third-Party Services

### 📖 Plain English Definition

Real-world code depends on external services — REST APIs, AWS S3, databases, message queues. Testing against real infrastructure is slow, expensive, non-deterministic, and often impossible in CI/CD. This module covers patterns for mocking the most common third-party dependencies.

### 🛠️ Ways to Use This

- **REST APIs**: Mock `requests.get` / `requests.post` with a `MockResponse` class.
- **AWS Services**: Use the `moto` library to simulate S3, DynamoDB, SQS, and more.
- **Background Tasks (Celery)**: Mock `.delay()` and `.apply_async()` to test task dispatch without a broker.

### 💻 Annotated Code Block

```python
# ============================================================
# FILE: test_module12_third_party_services.py
# ============================================================

import pytest
from unittest.mock import MagicMock, patch


# ============================================================
# PATTERN 1: Mocking REST APIs with a MockResponse Class
# ============================================================
# The MockResponse pattern is cleaner than chaining .return_value
# attributes — it's readable, reusable, and self-documenting.

import requests


def get_weather(city: str) -> dict:
    """Calls an external weather API."""
    response = requests.get(f"https://goweather.herokuapp.com/weather/{city}")
    return response.json()


class MockResponse:
    """Reusable mock HTTP response object."""

    def __init__(self, data: dict, status_code: int = 200):
        self._data = data
        self.status_code = status_code

    def json(self):
        return self._data

    def raise_for_status(self):
        if self.status_code >= 400:
            raise requests.HTTPError(f"HTTP {self.status_code}")


def test_get_weather_mocked(mocker):
    mock_data = {
        "temperature": "+7 °C",
        "wind": "13 km/h",
        "description": "Partly cloudy",
    }

    # Patch requests.get at the point of use in this module.
    mocker.patch(
        "test_module12_third_party_services.requests.get",
        return_value=MockResponse(mock_data)
    )

    result = get_weather("London")

    assert result["temperature"] == "+7 °C"
    assert result["description"] == "Partly cloudy"


def test_get_weather_handles_server_error(mocker):
    mocker.patch(
        "test_module12_third_party_services.requests.get",
        return_value=MockResponse({}, status_code=503)
    )

    response = requests.get("https://goweather.herokuapp.com/weather/London")
    with pytest.raises(requests.HTTPError):
        response.raise_for_status()


# ============================================================
# PATTERN 2: Reusable Mock via Fixture
# ============================================================
# When the same mock is needed across multiple tests, move it
# into a fixture in conftest.py for DRY, maintainable setup.

@pytest.fixture
def mock_weather_api(mocker):
    """Reusable fixture for the weather API mock."""
    return mocker.patch(
        "test_module12_third_party_services.requests.get",
        return_value=MockResponse({"temperature": "+15 °C", "wind": "5 km/h"})
    )


def test_weather_fixture_usage(mock_weather_api):
    result = get_weather("Tokyo")
    assert result["temperature"] == "+15 °C"
    mock_weather_api.assert_called_once()


# ============================================================
# PATTERN 3: Mocking AWS S3 with moto
# ============================================================
# moto intercepts boto3 calls and simulates AWS infrastructure.
# No real AWS account, no real costs, no real latency.
#
# REQUIRES: pip install moto boto3

# from moto import mock_aws
# import boto3
#
# def get_s3_object(bucket: str, key: str) -> bytes:
#     """Fetches an object from S3."""
#     client = boto3.client("s3")
#     response = client.get_object(Bucket=bucket, Key=key)
#     return response["Body"].read()
#
#
# @pytest.fixture(scope="function")
# def aws_credentials(monkeypatch):
#     """Prevents real AWS calls by injecting fake credentials."""
#     monkeypatch.setenv("AWS_ACCESS_KEY_ID", "testing")
#     monkeypatch.setenv("AWS_SECRET_ACCESS_KEY", "testing")
#     monkeypatch.setenv("AWS_DEFAULT_REGION", "us-east-1")
#
#
# @pytest.fixture(scope="function")
# def s3_client(aws_credentials):
#     """Provides a moto-backed S3 client for tests."""
#     with mock_aws():
#         yield boto3.client("s3", region_name="us-east-1")
#
#
# @mock_aws
# def test_get_s3_object(s3_client):
#     # Create the bucket and object in the moto-simulated environment
#     s3_client.create_bucket(Bucket="test-bucket")
#     s3_client.put_object(Bucket="test-bucket", Key="data.json", Body=b'{"key": "val"}')
#
#     result = get_s3_object("test-bucket", "data.json")
#     assert result == b'{"key": "val"}'


# ============================================================
# PATTERN 4: Mocking Celery Task Dispatch
# ============================================================
# Mock .delay() to test that your code dispatches tasks correctly
# without needing a running RabbitMQ/Redis broker.

# tasks.py:
# from celery import Celery
# app = Celery("tasks", broker="pyamqp://")
#
# @app.task
# def send_notification(user_id: int, message: str):
#     pass

def trigger_notification(user_id: int, message: str):
    """Application code that dispatches a Celery task."""
    from tasks import send_notification
    send_notification.delay(user_id, message)


@pytest.fixture
def mock_celery_task(mocker):
    """Patches Celery's delay method — no broker needed."""
    return mocker.patch("tasks.send_notification.delay")


def test_trigger_notification_dispatches_task(mock_celery_task):
    trigger_notification(user_id=42, message="Your order shipped!")

    # Verify the task was dispatched with the correct arguments
    mock_celery_task.assert_called_once_with(42, "Your order shipped!")
```

> **💡 Expert Tip:** The `MockResponse` class pattern (instead of chaining `.return_value.json.return_value`) is dramatically more readable and reusable. Define it once in `conftest.py` or a `test_helpers.py` module and share it across your test suite.

-----

## Module 13: Mock Hell — Anti-Patterns & How to Escape

### 📖 Plain English Definition

**“Mock Hell”** is the state you reach when your tests are so deeply coupled to implementation details — via excessive mocking — that every refactoring breaks dozens of tests, even when the actual behavior is unchanged. This is the most important module in the guide for long-term test maintainability.

### The Five Deadly Sins of Mocking

#### Sin 1: Mocking Internal Implementation Details

```python
# ❌ BAD: This test breaks if you refactor the internals,
# even if get_full_name() still works correctly.

class User:
    def get_full_name(self):
        first = self.get_first_name()   # Calls two methods internally
        last = self.get_last_name()
        return f"{first} {last}"

def test_get_full_name_BAD():
    user = User()
    user.get_first_name = Mock(return_value="Jane")
    user.get_last_name = Mock(return_value="Smith")
    assert user.get_full_name() == "Jane Smith"
    # If someone refactors to a single DB call, this test breaks
    # even though the external behavior (returning a full name) is unchanged.


# ✅ GOOD: Test the interface, not the implementation.
# This test survives any internal refactoring.

def test_get_full_name_GOOD():
    user = User()
    result = user.get_full_name()
    assert result == "John Doe"  # Whatever the real implementation returns
```

#### Sin 2: Deep / Recursive Mocks (Mocks-in-Mocks)

```python
# ❌ BAD: Three levels of mocking. Fragile, unreadable.
# Any change to WeatherClient or WeatherData breaks this test.

def test_weather_BAD():
    mock_client = Mock()
    mock_data = Mock()                              # Mock of a mock's return value
    mock_data.get_temperature.return_value = 72
    mock_data.get_humidity.return_value = 55
    mock_client.fetch_weather.return_value = mock_data  # Recursive mock!

    service = WeatherService(mock_client)
    report = service.get_weather_report("New York")

    assert "72" in report
    # If WeatherData ever changes its API, ALL these assertions break.


# ✅ GOOD: Use a Fake — a simple, real implementation.
# It's resilient to internal changes and tests actual behavior.

class FakeWeatherClient:
    """An in-memory implementation for testing. No external calls."""
    def fetch_weather(self, location: str):
        return WeatherData(temperature=72, humidity=55)

def test_weather_GOOD():
    service = WeatherService(FakeWeatherClient())
    report = service.get_weather_report("New York")
    assert "72" in report
```

#### Sin 3: Mocking Low-Level Architecture

```python
# ❌ BAD: Tightly coupled to the SQL implementation.
# Switching from SQLite to PostgreSQL breaks ALL of these.

def test_get_user_by_id_BAD():
    mock_conn = Mock()
    mock_cursor = mock_conn.cursor.return_value
    mock_cursor.fetchone.return_value = {"id": 1, "name": "Alice"}

    service = UserService(mock_conn)
    user = service.get_user_by_id(1)

    # If you rename the table, or switch ORMs, this breaks.
    mock_cursor.execute.assert_called_once_with(
        "SELECT * FROM users WHERE id = ?", (1,)
    )


# ✅ GOOD: Mock the repository interface, not the SQL cursor.
# Your tests survive storage layer changes.

class FakeUserRepository:
    def __init__(self):
        self._users = {1: {"id": 1, "name": "Alice"}}

    def get_by_id(self, user_id: int):
        return self._users.get(user_id)

def test_get_user_by_id_GOOD():
    service = UserService(FakeUserRepository())
    user = service.get_user_by_id(1)
    assert user["name"] == "Alice"
```

#### Sin 4: Tests Coupled to Call Count (Not Behavior)

```python
# ❌ BAD: Tests HOW the code works, not WHAT it produces.

def test_process_order_BAD():
    mock_inventory = Mock()
    mock_payment = Mock()
    mock_inventory.check_inventory.return_value = True
    mock_payment.process_payment.return_value = True

    processor = OrderProcessor(mock_payment, mock_inventory)
    result = processor.process_order("item1", 2, 50.0)

    # These break if implementation changes (e.g. cache avoids a re-check)
    mock_inventory.check_inventory.assert_called_once()
    mock_payment.process_payment.assert_called_once()


# ✅ GOOD: Mock Roles, Not Objects.
# Test that the right interactions happen for the right ROLES.
# Verify that the order was processed, and that the key interactions
# were correct — but don't over-specify the how.

def test_process_order_GOOD():
    payment_role = Mock()
    inventory_role = Mock()
    inventory_role.check_inventory.return_value = True
    payment_role.process_payment.return_value = True

    processor = OrderProcessor(payment_role, inventory_role)
    result = processor.process_order("item1", 2, 50.0)

    assert result == "Order processed"
    inventory_role.check_inventory.assert_called_with("item1", 2)
    payment_role.process_payment.assert_called_with(50.0)
```

#### Sin 5: Not Having Integration Tests to Back Up Unit Tests

Mocked unit tests cannot catch:

- Breaking API schema changes from a third party
- SQL query bugs (you mocked the cursor)
- Network/auth configuration issues
- Emergent behavior between components

**The Rule**: Unit tests (with mocks) for speed and isolation. Integration tests (with real dependencies) for correctness. You need both.

```python
# ✅ PROFESSIONAL STRUCTURE:

# Unit test — fast, isolated, mocked:
def test_charge_customer_unit(mocker):
    mocker.patch("payment.requests.post", return_value=MockResponse({"status": "ok"}))
    result = charge_customer(9.99, "tok_visa")
    assert result["status"] == "ok"

# Integration test — slow, real, not mocked:
@pytest.mark.integration
def test_charge_customer_integration():
    # Uses the real test payment gateway
    result = charge_customer(1.00, "tok_test_visa")
    assert result["status"] == "ok"
```

### When IS Mocking the Right Choice?

```
✅ Mock when you want to:
  - Eliminate expensive resources (network, DB, compute)
  - Make non-deterministic code deterministic (randomness, time, network errors)
  - Test objects whose state can't or shouldn't be exposed
  - Speed up a CI/CD pipeline significantly
  - Test error conditions that are hard to trigger in real systems

❌ Don't mock when:
  - You're mocking internal methods of the class under test
  - The mock is more complex than the real thing
  - You end up with 3+ levels of nested mock configuration
  - Refactoring breaks the test even though behavior is unchanged
```

> **💡 Expert Tip:** Ask yourself: “If I completely rewrote the internals but kept the public interface identical, would this test still pass?” If the answer is no, you’re over-mocking. Pull back to the interface boundary.

> **⚠️ Danger Zone:** Mock Hell is insidious — it builds gradually. The warning signs are: tests that take more lines to set up than to assert, tests that break when you rename a private method, and tests where you can’t tell what behavior is being verified without reading every line.

-----

## Module 14: Dependency Injection for Testability

### 📖 Plain English Definition

**Dependency Injection (DI)** is a design pattern where objects receive their dependencies from the outside rather than creating them internally. It’s not just a testing technique — it’s the design decision that makes mocking easy, clean, and not a sign of bad tests.

### 📖 The Core Problem DI Solves

```python
# ❌ WITHOUT DI: Tightly coupled. Impossible to test without patching internals.

class ReportGenerator:
    def __init__(self):
        self.db = DatabaseConnection("prod-host", 5432)  # Hard-coded dependency
        self.emailer = SMTPEmailer("smtp.company.com")   # Another hard-coded dep

    def generate_and_send(self, report_id: int):
        data = self.db.query(f"SELECT * FROM reports WHERE id={report_id}")
        self.emailer.send("manager@company.com", data)


# To test this, you MUST patch DatabaseConnection and SMTPEmailer globally.
# The test is fragile, coupled to implementation details, and hard to read.
```

```python
# ✅ WITH DI: Loosely coupled. Trivially testable. No patching needed at all.

class ReportGenerator:
    def __init__(self, db, emailer):  # Dependencies come from OUTSIDE
        self.db = db
        self.emailer = emailer

    def generate_and_send(self, report_id: int):
        data = self.db.query(f"SELECT * FROM reports WHERE id={report_id}")
        self.emailer.send("manager@company.com", data)


# In production:
generator = ReportGenerator(
    db=DatabaseConnection("prod-host", 5432),
    emailer=SMTPEmailer("smtp.company.com")
)

# In tests — inject Fakes or Mocks, zero patching required:
def test_generate_and_send(mocker):
    mock_db = mocker.MagicMock()
    mock_emailer = mocker.MagicMock()
    mock_db.query.return_value = {"id": 1, "content": "Q4 Summary"}

    generator = ReportGenerator(db=mock_db, emailer=mock_emailer)
    generator.generate_and_send(report_id=1)

    mock_emailer.send.assert_called_once_with(
        "manager@company.com",
        {"id": 1, "content": "Q4 Summary"}
    )
```

### The Adapter Pattern for External Libraries

When you `import requests` directly in your code, you’ve created a hard dependency on a specific library. The Adapter pattern wraps it:

```python
# ============================================================
# FILE: test_module14_dependency_injection.py
# ============================================================

# --- The Adapter Pattern ---
# Wrap external libraries in your own interface.
# Benefits: swap the underlying library anytime, mock the adapter (not requests).

from abc import ABC, abstractmethod


class HttpClientInterface(ABC):
    """Define YOUR interface, independent of any library."""
    @abstractmethod
    def get(self, url: str) -> dict:
        pass

    @abstractmethod
    def post(self, url: str, payload: dict) -> dict:
        pass


class RequestsHttpClient(HttpClientInterface):
    """The real implementation — wraps requests."""
    def get(self, url: str) -> dict:
        import requests
        return requests.get(url).json()

    def post(self, url: str, payload: dict) -> dict:
        import requests
        return requests.post(url, json=payload).json()


class FakeHttpClient(HttpClientInterface):
    """A Fake for testing — no network calls, predictable responses."""
    def __init__(self, responses: dict):
        self._responses = responses  # url → response data

    def get(self, url: str) -> dict:
        return self._responses.get(url, {})

    def post(self, url: str, payload: dict) -> dict:
        return self._responses.get(url, {"status": "ok"})


# --- Service Using DI ---

class WeatherService:
    def __init__(self, http_client: HttpClientInterface):
        self._client = http_client

    def get_weather(self, city: str) -> dict:
        return self._client.get(f"https://weather.api.com/{city}")


# --- Tests: Clean, No Patching Needed ---

def test_get_weather_with_fake():
    fake_client = FakeHttpClient({
        "https://weather.api.com/London": {"temp": 15, "condition": "cloudy"}
    })

    service = WeatherService(fake_client)
    result = service.get_weather("London")

    assert result["temp"] == 15
    assert result["condition"] == "cloudy"


def test_get_weather_city_not_found():
    fake_client = FakeHttpClient({})  # No configured responses

    service = WeatherService(fake_client)
    result = service.get_weather("UnknownCity")

    assert result == {}  # Graceful handling of missing data


# --- Centralize Mocks in Fixtures ---
# Define all mocks in one place — fixtures.
# This avoids inconsistencies and makes updates a single-point change.

import pytest

@pytest.fixture
def weather_service(mocker):
    """Provides a WeatherService with a mocked HTTP client."""
    mock_client = mocker.MagicMock(spec=HttpClientInterface)
    mock_client.get.return_value = {"temp": 20, "condition": "sunny"}
    return WeatherService(mock_client)


def test_weather_sunny(weather_service):
    result = weather_service.get_weather("Paris")
    assert result["condition"] == "sunny"


def test_weather_temperature(weather_service):
    result = weather_service.get_weather("Rome")
    assert result["temp"] == 20
```

> **💡 Expert Tip:** The single best thing you can do to improve testability across an entire codebase is to audit all `__init__` methods for hard-coded instantiations (`self.x = SomeClass()`). Each one is a hidden hard dependency. Make them injectable parameters and watch your test setup complexity collapse.

-----

## Module 15: Comparison Tables

### Table 1: `unittest.mock.patch` vs. `pytest.monkeypatch`

|Dimension                       |`unittest.mock.patch`                           |`pytest.monkeypatch`                             |
|--------------------------------|------------------------------------------------|-------------------------------------------------|
|**Primary Use**                 |Replacing objects/functions with Mock instances |Directly modifying attributes, env vars, dicts   |
|**Interface**                   |Decorator, context manager, or manual start/stop|pytest fixture injected as a parameter           |
|**Cleanup**                     |✅ Automatic (context manager / decorator)       |✅ Automatic (fixture teardown)                   |
|**Environment Variables**       |`patch.dict(os.environ, {...})`                 |`monkeypatch.setenv()` / `delenv()` — cleaner API|
|**Dictionary Modification**     |`patch.dict(my_dict, {"key": "val"})`           |`monkeypatch.setitem()` / `delitem()`            |
|**Attribute Replacement**       |`patch.object(obj, "attr", Mock())`             |`monkeypatch.setattr(obj, "attr", value)`        |
|**Works with unittest.TestCase**|✅ Yes                                           |❌ No — pytest fixtures only                      |
|**Create Mock Objects**         |✅ Primary purpose                               |⚠️ Possible but not idiomatic                     |
|**Async Support**               |✅ Via `AsyncMock` + `new_callable`              |✅ Can patch async functions via `setattr`        |
|**Best For**                    |Mocking service dependencies, external I/O      |Config, env vars, feature flags, global state    |
|**Typical Syntax**              |`@patch("module.Class.method")`                 |`monkeypatch.setattr(module, "attr", value)`     |
|**String-based target**         |✅ Required — `"module.attribute"`               |✅ Supported but object form also works           |
|**sys.path modification**       |❌ Not built-in                                  |✅ `monkeypatch.syspath_prepend()`                |
|**chdir support**               |❌ Not built-in                                  |✅ `monkeypatch.chdir()`                          |

-----

### Table 2: `Mock` vs. `MagicMock` vs. `AsyncMock`

|Dimension                        |`Mock`                            |`MagicMock`                                         |`AsyncMock`                                                |
|---------------------------------|----------------------------------|----------------------------------------------------|-----------------------------------------------------------|
|**Best For**                     |Simple objects, no protocol magic |Containers, context managers, protocol-heavy objects|Any `async def` function or method                         |
|**Dunder Methods Pre-configured**|❌ None                            |✅ 20+ (`__len__`, `__iter__`, `__enter__`, etc.)    |✅ All of MagicMock + `__aenter__`, `__aexit__`, `__aiter__`|
|**`await mock()` works**         |❌ TypeError                       |❌ TypeError                                         |✅ Yes                                                      |
|**`for x in mock:` works**       |❌ TypeError                       |✅ Yes                                               |✅ Yes                                                      |
|**`len(mock)` works**            |❌ TypeError                       |✅ Yes                                               |✅ Yes                                                      |
|**`with mock as x:` works**      |❌ TypeError                       |✅ Yes                                               |✅ Yes                                                      |
|**`async with mock:` works**     |❌ No                              |❌ No                                                |✅ Yes                                                      |
|**`async for x in mock:` works** |❌ No                              |❌ No                                                |✅ Yes                                                      |
|**pytest-mock default**          |❌                                 |✅ `mocker.patch()` returns MagicMock                |✅ Auto-detected for async funcs                            |
|**Async assert methods**         |❌                                 |❌                                                   |✅ `assert_awaited_with()`, `assert_awaited_once()`         |
|**`spec=` Compatible**           |✅                                 |✅                                                   |✅                                                          |
|**`autospec=` Compatible**       |✅                                 |✅                                                   |✅ Auto-detects async                                       |
|**When to Prefer**               |Explicit: “no magic needed” signal|Default for most cases                              |Any code with `async def` or `await`                       |

-----

### Table 3: Mock vs. Stub vs. Fake vs. Spy (Test Doubles)

|Double   |Implementation                   |Verifies Calls?|Real Logic? |Best For                                          |
|---------|---------------------------------|---------------|------------|--------------------------------------------------|
|**Dummy**|`Mock()` (unused)                |❌              |❌           |Satisfying required parameters                    |
|**Stub** |`Mock(return_value=x)`           |❌              |❌           |Controlling what a dependency returns             |
|**Mock** |`Mock()` + `assert_called_with()`|✅              |❌           |Verifying interactions with dependencies          |
|**Spy**  |`Mock(wraps=real_fn)`            |✅              |✅           |Observing calls without replacing behavior        |
|**Fake** |Custom class (e.g. `FakeDB`)     |❌              |✅ Simplified|Replacing infrastructure with in-memory equivalent|

-----

## Quick Reference Cheat Sheet

```python
# ── INSTALLATION ────────────────────────────────────────────
# pip install pytest pytest-mock pytest-asyncio

# ── CREATING MOCKS ──────────────────────────────────────────
from unittest.mock import Mock, MagicMock, AsyncMock, create_autospec, mock_open

m = Mock()                                  # Blank mock, no dunder methods
m = MagicMock()                             # Mock + all dunder methods (DEFAULT)
m = AsyncMock()                             # Mock for async/await code
m = create_autospec(MyClass, instance=True) # Spec-validated — validates signatures
m = mock_open(read_data="file content")     # Pre-configured for file I/O

# ── THE MOCKER FIXTURE (pytest-mock) ────────────────────────
def test_example(mocker):
    mocker.patch("module.name")                          # Returns MagicMock
    mocker.patch("module.name", return_value=42)         # With return value
    mocker.patch("module.name", new_callable=AsyncMock)  # Specific type
    mocker.patch.object(obj, "method")                   # Patch one method
    mocker.patch.multiple("module", attr1=val1)          # Patch several
    mocker.patch.dict(os.environ, {"KEY": "value"})      # Patch dict
    mocker.MagicMock()                                   # Standalone mock

# ── CONFIGURING BEHAVIOR ────────────────────────────────────
m.method.return_value = 42                  # Always return 42
m.method.side_effect = ValueError("bad")   # Raise this exception
m.method.side_effect = ValueError          # Raise exception class
m.method.side_effect = [1, 2, 3]          # Return 1, then 2, then 3
m.method.side_effect = my_function         # Call fn with same args
m.method.side_effect = cycle([1, 2, 3])   # Infinite cycling (itertools)

# ── ASSERTIONS ──────────────────────────────────────────────
m.method.assert_called()                          # Called ≥ once
m.method.assert_called_once()                     # Called exactly once
m.method.assert_called_with(a, b=c)               # Last call had these args
m.method.assert_called_once_with(a, b=c)          # Only call, these args
m.method.assert_any_call(a)                       # Any call had this arg
m.method.assert_not_called()                      # Never called
m.method.assert_has_calls([call(a), call(b)])     # Ordered sequence
m.method.assert_has_calls([...], any_order=True)  # Unordered sequence
m.method.call_count                               # Number of calls (int)
m.method.call_args_list                           # Full call history

# ASYNC-SPECIFIC (AsyncMock)
await_mock.assert_awaited()
await_mock.assert_awaited_once_with(arg)
await_mock.assert_not_awaited()

# RESET
m.reset_mock()  # Clears call history, children, return_value, side_effect

# ── PATCHING ────────────────────────────────────────────────
from unittest.mock import patch

with patch("mymodule.ClassName") as mock:             # Context manager
    ...

@patch("mymodule.ClassName")                          # Decorator
def test(mock): ...

with patch.object(MyClass, "method") as mock: ...     # Specific method
with patch.dict(os.environ, {"KEY": "val"}): ...      # Dict patching
with patch("builtins.open", mock_open(read_data="x")) as m: ...  # File I/O

@patch("mymodule.B")   # ← arg 1st (inner)
@patch("mymodule.A")   # ← arg 2nd (outer) — BOTTOM arg = FIRST parameter
def test(mock_a, mock_b): ...  # Reverse order!

# ── MONKEYPATCH (pytest fixture) ────────────────────────────
def test(monkeypatch):
    monkeypatch.setattr(obj, "attr", new_value)       # Replace attribute
    monkeypatch.setattr("module.func", fake_func)     # Replace by path
    monkeypatch.delattr(obj, "attr")                  # Remove attribute
    monkeypatch.setenv("VAR", "value")                # Set env var
    monkeypatch.delenv("VAR", raising=False)          # Remove env var
    monkeypatch.setitem(my_dict, "key", "val")        # Set dict item
    monkeypatch.delitem(my_dict, "key")               # Remove dict item
    monkeypatch.syspath_prepend("/path/to/module")    # Modify sys.path
    monkeypatch.chdir("/some/directory")              # Change cwd

# ── THE GOLDEN RULES ────────────────────────────────────────
# 1. Patch WHERE the name is USED, not where it's defined.
#    ✅ patch("mymodule.requests")   ❌ patch("requests.get")
#
# 2. Use MagicMock by default. Use Mock only when explicitly signaling "no magic."
#
# 3. Use AsyncMock for anything with `async def` or `await`.
#
# 4. Use autospec=True / create_autospec to catch interface drift.
#
# 5. Use wraps= to spy without replacing real behavior.
#
# 6. Use monkeypatch for env vars, config, feature flags.
#    Use patch() for service dependencies and external I/O.
#
# 7. Don't mock internals. Test interfaces, not implementation.
#
# 8. When mocks get complex, use a Fake instead.
#
# 9. Use Dependency Injection to make mocking clean and optional.
#
# 10. Always back up unit tests with integration tests.
```

-----

*Built for mastery. Every module grounded in real-world sources. Mock confidently. Test fearlessly.*

