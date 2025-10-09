# CLI-First C++ Development Style Guide (SDL2 + Nuklear Edition)

This document outlines a lightweight, CLI-first approach to writing, building, and distributing C++ software using **SDL2** and **Nuklear** for small, fast, cross-platform GUIs. It focuses on transparent builds, minimal dependencies, and direct control over compilation and linking—without large build systems or IDE lock-in.

---

## 1. Philosophy of CLI-First C++ Development

Most modern C++ projects use heavyweight environments like Visual Studio, Xcode, or large meta-build systems (CMake, Meson, Premake). These tools can:

* Introduce unnecessary abstraction and hidden configuration.
* Enforce non-portable conventions.
* Add dependency sprawl and opaque build steps.
* Slow iteration for small, single-purpose tools.

A CLI-first C++ style instead:

* Uses **standard compiler tools** directly (`g++`, `clang++`, `x86_64-w64-mingw32-g++`).
* Builds from **Makefiles** or **shell/batch scripts**.
* Keeps source trees **transparent, minimal, and reproducible**.
* Emphasizes **small self-contained binaries** that link only what they need.

---

## 2. Goals for GUI: Why SDL2 + Nuklear

This guide standardizes on **SDL2 + Nuklear** as the preferred GUI combination for small cross-platform applications.

### Reasons

* **Tiny footprint:** SDL2 handles windows, input, and rendering; Nuklear adds GUI widgets in a single header file.
* **Pure C/C++ stack:** No CMake superprojects, XML manifests, or dynamic runtime frameworks.
* **Immediate-mode simplicity:** UI defined inline with logic, easy to modify and debug.
* **Static-link friendly:** Both compile cleanly into a single binary with no runtime dependencies.
* **Cross-platform:** Works seamlessly on Linux, Windows, and macOS.
* **Transparent builds:** You can literally see every flag and file in the build command.

### Goal

> To build small, fast, portable GUI utilities — such as RSS readers, Ollama frontends, and monitoring dashboards — that compile in seconds and distribute as single executables.

---

## 3. Basic Project Structure

A typical CLI-first C++ + SDL2 + Nuklear project:

```
project/
  src/
    main.cpp
    gui.cpp
    gui.h
  assets/
    icons/
    fonts/
  build/        # intermediate object files (ignored by VCS)
  bin/          # final executables
  Makefile
```

* `src/` → C++ source and headers
* `assets/` → static resources embedded or loaded at runtime
* `build/` → compiler outputs (.o, .obj)
* `bin/` → packaged executables

---

## 4. Compiling and Running with Minimal Dependencies

Use `g++` (Linux/macOS) or `x86_64-w64-mingw32-g++` (Windows cross-compile).

```bash
# Compile and link directly
g++ -std=c++17 -O2 -Wall \
  src/*.cpp \
  -o bin/myapp \
  $(pkg-config --cflags --libs sdl2)
```

Add Nuklear by including it locally:

```cpp
#define NK_IMPLEMENTATION
#include "nuklear.h"
#include "nuklear_sdl_renderer.h"
```

This produces a single executable — no installers, no runtime dependencies, no DLL sprawl.

---

## 5. Makefile Example

```makefile
APP = myapp
SRC = $(wildcard src/*.cpp)
OBJ = $(SRC:src/%.cpp=build/%.o)
CXX = g++
CXXFLAGS = -std=c++17 -O2 -Wall $(shell pkg-config --cflags sdl2)
LDFLAGS = $(shell pkg-config --libs sdl2)

all: bin/$(APP)

build/%.o: src/%.cpp
	mkdir -p build
	$(CXX) $(CXXFLAGS) -c $< -o $@

bin/$(APP): $(OBJ)
	mkdir -p bin
	$(CXX) $(OBJ) -o $@ $(LDFLAGS)

clean:
	rm -rf build bin
```

Build with `make` or `mingw32-make` (on Windows).

---

## 6. Static Linking for Self-Contained Apps

To avoid runtime library issues and ensure portability:

**Linux static build:**

```bash
g++ -static -static-libgcc -static-libstdc++ -O2 \
  src/*.cpp -o bin/myapp \
  $(pkg-config --cflags --libs sdl2)
```

**Windows cross-compile (from Linux):**

```bash
x86_64-w64-mingw32-g++ -O2 -std=c++17 \
  src/*.cpp -o bin/myapp.exe \
  $(x86_64-w64-mingw32-pkg-config --cflags --libs sdl2)
```

Now you can ship `bin/myapp` or `bin/myapp.exe` as a single binary.

---

## 7. Minimal GUI Example

```cpp
#include <SDL.h>
#define NK_INCLUDE_DEFAULT_ALLOCATOR
#define NK_INCLUDE_STANDARD_IO
#define NK_INCLUDE_FIXED_TYPES
#define NK_INCLUDE_DEFAULT_FONT
#define NK_IMPLEMENTATION
#include "nuklear.h"
#define NK_SDL_RENDERER_IMPLEMENTATION
#include "nuklear_sdl_renderer.h"

int main() {
    SDL_Init(SDL_INIT_VIDEO);
    SDL_Window* win = SDL_CreateWindow("Hello",
        SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED,
        640, 480, SDL_WINDOW_SHOWN);
    SDL_Renderer* ren = SDL_CreateRenderer(win, -1, SDL_RENDERER_ACCELERATED);

    struct nk_context* ctx = nk_sdl_init(win, ren);
    nk_sdl_font_stash_begin(NULL);
    nk_sdl_font_stash_end();

    bool running = true;
    while (running) {
        SDL_Event evt;
        nk_input_begin(ctx);
        while (SDL_PollEvent(&evt)) {
            if (evt.type == SDL_QUIT) running = false;
            nk_sdl_handle_event(&evt);
        }
        nk_input_end(ctx);

        if (nk_begin(ctx, "Demo", nk_rect(50, 50, 200, 150),
            NK_WINDOW_BORDER | NK_WINDOW_MOVABLE | NK_WINDOW_TITLE)) {
            nk_layout_row_dynamic(ctx, 30, 1);
            if (nk_button_label(ctx, "Exit")) running = false;
        }
        nk_end(ctx);

        SDL_SetRenderDrawColor(ren, 30, 30, 30, 255);
        SDL_RenderClear(ren);
        nk_sdl_render(NK_ANTI_ALIASING_ON);
        SDL_RenderPresent(ren);
    }

    nk_sdl_shutdown();
    SDL_DestroyRenderer(ren);
    SDL_DestroyWindow(win);
    SDL_Quit();
    return 0;
}
```

---

## 8. Why Not Qt, wxWidgets, or GTK?

* **Qt:** powerful but enormous; multi-gigabyte SDK and proprietary build tools.
* **wxWidgets:** native look, but requires CMake or bakefiles, heavier than needed.
* **GTK:** mature but brings large runtime dependencies and theming quirks on Windows.

**SDL2 + Nuklear** avoids all of that — it’s small, fast, and fully portable.

---

## 9. Benefits of This Style

* **Minimalism:** two small dependencies (SDL2 and Nuklear), both pure C.
* **Transparency:** every build step visible in a Makefile or shell script.
* **Reproducibility:** builds identical on Linux and Windows.
* **Speed:** full rebuilds in seconds; no dependency management.
* **Portability:** same source compiles anywhere with g++/clang++.
* **Control:** choose exactly what to link, bundle, or statically embed.

---

## 10. Example Workflow

1. Write code in any editor.
2. Run `make` to:

   * Compile and link sources.
   * Build a single binary in `bin/`.
3. Test locally (`./bin/myapp` or `myapp.exe`).
4. Cross-compile for Windows if needed.
5. Distribute binaries directly — no installers or external libraries.

---

By following this **CLI-first C++ style**, you’ll produce self-contained, portable GUI tools that compile in seconds, run anywhere, and depend on nothing but your own source code. SDL2 + Nuklear provides just enough GUI capability to make tools usable without sacrificing performance or simplicity—perfect for small, independent, and reproducible software projects.

---

## 11. Logging and Interpretability

CLI-first C++ tools should adopt the same verbose logging rules described in the general CLI development guide. At minimum:

* **Root‑level log files:** Create an append‑only log file in the project root capturing all user inputs, outputs, and contextual metadata. Avoid log rotation or deletion—maintain a complete history.
* **Plain text format:** Logs must be plain text so that both humans and automated agents can parse them easily.
* **Include subprocess output:** If your application spawns subprocesses, capture their `stdout` and `stderr` streams and append them to the same log.
* **Deterministic ordering:** Log entries should appear in the exact order actions occur to make replaying and auditing runs straightforward.
* **See the CLI development guide:** For details on log levels (INFO, WARN, ERROR) and cross‑language log formats, refer to `cli-development.md` in this repository.

---

## 12. Test Mode & Simulation

Implement a `--test`, `--dry-run`, or similar command‑line flag that runs the application in simulation mode. In test mode:

* **No real side effects:** Use temporary directories for configuration, storage, and outputs. Do not modify user data or external systems.
* **Same code paths:** Exercise the same code paths and produce the same output and logging as a real run so automated agents can verify behavior.
* **Reproducibility:** Because test mode writes to isolated locations and never modifies real data, results can be replayed for testing and auditing.
* **Documentation:** Explain your test mode in the README and CLI help text so users and agents know how to activate it.

---

## 13. Project Roadmap, Specification & Social Context

Every CLI-first project should include the following files at the repository root:

* **`project-roadmap.md`** – A human‑readable task list with a status legend. List upcoming features, tasks, and bug fixes. Check off items as they’re completed and keep completed tasks for historical context.
* **`project-specification.md`** – A technical specification that outlines functional requirements, architecture, data flows, error‑handling expectations, and any third‑party services or APIs.
* **`social-context.md`** – A document describing why the project exists and who benefits. Include the history of the project, any ethical or privacy considerations, the stakeholders involved, and how the tool fits into a broader ecosystem.

These documents make it easier for contributors, users, and automated agents to understand the high‑level purpose and current state of your project.

---

## 14. Including External Tools (Tor, IPFS, etc.)

Sometimes a CLI-first C++ tool needs to interface with external binaries—such as **Tor** for anonymous networking or **IPFS** for decentralized storage. To bundle and manage these tools reliably:

1. Place the external binaries in an `assets/bin/` directory within your repository.
2. At runtime, extract the binaries to a temporary directory using standard C++ filesystem utilities.
3. Launch the binaries with `std::system` or a process library (e.g., `Boost.Process`) and communicate via sockets, pipes, or command-line interfaces.
4. Include configuration files in `assets/etc/` and reference them when launching the external tools.

By vendoring external binaries you avoid relying on system installations and maintain control over versions and configurations.

---

## 15. Cross‑Referencing this Guide

This C++ guide is part of a larger style guide collection. For general logging, interpretability, and agent‑integration rules that apply to all CLI‑based projects, refer to `cli-development.md` in this repository.
