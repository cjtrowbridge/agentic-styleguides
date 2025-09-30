# CLI-First Java Development Style Guide

This document describes a simple, command-line–first approach to writing, building, and distributing Java software. The focus is on keeping projects transparent, reproducible, and free from heavy build systems. For logging and interpretability rules that apply to all CLI-based projects (including Java), please refer to `cli-styleguide.md` in this repository.

---

## 1. Philosophy of CLI-First Java Development

Most modern Java developers use heavy IDEs (Eclipse, IntelliJ, NetBeans). While powerful, these tools can:

* Introduce hidden complexity.
* Enforce project conventions that are hard to escape.
* Add dependencies on large ecosystems (Maven, Gradle, IDE plugins).
* Make automation and reproducibility harder.

A CLI-first style instead:

* Uses the **JDK directly** (`javac`, `jar`, `java`, `jlink`, `jpackage`).
* Relies on **text editors** and **scripts** (Makefiles, batch files, shell scripts).
* Keeps projects **transparent and reproducible**.
* Emphasizes **self-contained applications** that include everything they need (JRE, binaries like Tor/IPFS, configs).

---

## 2. Basic Project Structure

A typical CLI-first Java project:

```
project/
  src/
    com/example/app/Main.java
  resources/
    bin/      # external binaries to embed (e.g., tor, ipfs)
    etc/      # configuration files
  build/      # compiled classes (ignored by VCS)
```

* `src/` → Java sources
* `resources/` → static files to package into the JAR
* `build/` → created by compile script

---

## 3. Compiling and Running with No Dependencies

Compile directly:

```bash
# compile sources into build/
javac -d build $(find src -name "*.java")

# add resources to build/
cp -r resources/* build/

# create manifest for entry point
echo "Main-Class: com.example.app.Main" > build/MANIFEST.MF

# build runnable jar
jar --create --file app.jar --manifest build/MANIFEST.MF -C build .

# run
java -jar app.jar
```

This produces a **single runnable JAR** that runs with one command on any system with Java.

---

## 4. Shipping Self-Contained Apps

Instead of requiring users to install Java, you can ship your own minimal runtime:

```bash
jlink --add-modules java.base,java.desktop \
      --strip-debug --no-header-files --no-man-pages \
      --output runtime

jpackage --name MyApp --input . --main-jar app.jar --runtime-image runtime
```

* `jlink` → trimmed JRE with only needed modules
* `jpackage` → native installer (MSI, PKG, DEB, etc.)

---

## 5. Including External Tools (Tor, IPFS, etc.)

Bundle third-party tools without system dependencies:

1. Place binaries (e.g., `tor.exe`, `ipfs.exe`) into `resources/bin/`.
2. Extract them at runtime to your app’s private folder.
3. Launch with `ProcessBuilder`.
4. Communicate via sockets/APIs.

```java
Process tor = new ProcessBuilder(
  "bin/tor.exe", "-f", "etc/tor/torrc"
).start();
```

This way, users don’t install Tor/IPFS system-wide—you control versions/configs.

---

## 6. Why Not Maven or Gradle?

Maven/Gradle are powerful, but not always necessary.

**You don’t need them if:**

* Few/no external libraries
* Shipping self-contained apps
* Prefer Makefiles/shell scripts

**You may want them if:**

* Many dependencies
* Multi-module projects
* Complex packaging/publishing pipelines

---

## 7. Benefits of This Style

* **Simplicity:** Just the JDK + your code
* **Transparency:** Every step is a command you understand
* **Portability:** One JAR runs anywhere Java runs
* **Control:** Choose exactly what to bundle
* **Reproducibility:** Your build script *is* the pipeline

---

## 8. Example Workflow

1. Write code in any text editor.
2. Run `make` (or script) to:

   * Compile sources
   * Copy resources
   * Package JAR
3. Test with `java -jar app.jar`.
4. Ship JAR or build installers with `jpackage`.

---

By following this style, your Java projects stay minimal, reproducible, and fully CLI-driven. For interpretability and logging rules, always cross-reference `cli-styleguide.md`.
