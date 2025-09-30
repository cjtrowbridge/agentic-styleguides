# CLI-First Python Development Style Guide

This document describes a simple, command-line–first approach to writing, running, and distributing Python software. The focus is on making projects transparent, reproducible, and easy to integrate with automated agents. For logging and interpretability rules that apply to all CLI-based projects, see `cli-styleguide.md`.

---

## 1. Philosophy of CLI-First Python Development

Most modern Python developers rely on IDEs, virtualenv managers, and packaging systems. While useful, these tools can:

* Introduce hidden state and complexity (virtual environments, IDE configs).
* Make automation harder when agents must guess about implicit behavior.
* Encourage reliance on external services (PyPI) without reproducibility guarantees.

A CLI-first style instead:

* Uses the **Python interpreter directly** (`python`, `pip`, `venv`).
* Relies on **text editors** and **scripts** (Makefiles, batch files, shell scripts).
* Keeps projects **transparent and reproducible**.
* Emphasizes **self-contained applications** that work on any machine.

---

## 2. Basic Project Structure

A typical CLI-first Python project:

```
project/
  src/
    myapp/
      __main__.py    # entry point
      core.py        # core logic
  resources/
    etc/             # configuration files
  tests/
    test_core.py
  build/             # distribution artifacts (ignored by VCS)
```

* `src/myapp/__main__.py` → CLI entry point (`python -m myapp`)
* `resources/etc/` → static configs
* `tests/` → unit/integration tests
* `build/` → generated wheels, packages

---

## 3. Running and Packaging

Run directly from source:

```bash
# run as a module
python -m myapp --help
```

Create a **virtual environment** for reproducibility:

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Build distribution artifacts:

```bash
python -m build   # produces .whl and .tar.gz in dist/
```

Install locally:

```bash
pip install dist/myapp-*.whl
```

---

## 4. Self-Contained Apps

Instead of relying on system Python, ship your own environment:

* **PyInstaller** or **shiv** → produce a single executable.
* **Docker** → containerize Python + your code.

Example with PyInstaller:

```bash
pyinstaller --onefile src/myapp/__main__.py --name myapp
./dist/myapp --help
```

---

## 5. Including External Tools (Tor, IPFS, etc.)

Bundle third-party tools alongside Python:

1. Place binaries into `resources/bin/`.
2. Extract them at runtime into a temporary directory.
3. Launch with `subprocess.Popen`.
4. Communicate via sockets, APIs, or subprocess pipes.

```python
import subprocess

subprocess.Popen([
    "bin/tor", "-f", "etc/tor/torrc"
])
```

---

## 6. Why Not Poetry or Conda?

Tools like Poetry/Conda are powerful, but not always necessary.

**You don’t need them if:**

* You want maximum transparency.
* You’re shipping self-contained executables.
* Dependencies are few or vendored.

**You may want them if:**

* You manage many dependencies.
* You need lockfile-style reproducibility.
* You publish widely on PyPI.

---

## 7. Benefits of This Style

* **Simplicity:** Just Python + your scripts.
* **Transparency:** No hidden configs or lock-in.
* **Portability:** One wheel or binary runs everywhere.
* **Control:** Choose what to vendor and bundle.
* **Reproducibility:** Your Makefile/script is the build pipeline.

---

## 8. Example Workflow

1. Write code in any text editor.
2. Run `make` (or script) to:

   * Run tests (`pytest`).
   * Package wheel.
   * Optionally build executable.
3. Test locally with `python -m myapp`.
4. Ship wheel, source tarball, or binary.

---

👉 By following this style, your Python projects stay minimal, reproducible, and fully CLI-driven. For interpretability and logging rules, always cross-reference `cli-styleguide.md`.
