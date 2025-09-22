# A Simple, CLI-First Style of Java Development

This document explains the style and pipeline of writing, building, and shipping Java software entirely from the command line interface (CLI). It emphasizes simplicity, minimal dependencies, and full control over the lifecycle of your software.

---

## 1. Philosophy of CLI-First Java Development

Most modern Java developers use heavy Integrated Development Environments (IDEs) such as Eclipse, IntelliJ, or NetBeans. These tools provide visual interfaces, automatic project structures, and lots of plugins. While powerful, they can also:

* Introduce hidden complexity.
* Enforce project conventions that are hard to escape.
* Add dependencies on large ecosystems (Maven, Gradle, IDE plugins).
* Make automation and reproducibility harder.

A CLI-first style instead:

* Uses the **JDK directly** (`javac`, `jar`, `java`, `jlink`, `jpackage`).
* Relies on **text editors** and **scripts** (Makefiles, batch files, or shell scripts).
* Keeps projects **transparent and reproducible**.
* Emphasizes **self-contained applications** that include everything they need (JRE, binaries like Tor/IPFS, configs).

This approach mirrors how C programmers use `gcc`/`make`—it gives you maximum control and minimal external tooling.

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

Key points:

* **`src/`** holds Java sources.
* **`resources/`** contains static files you want packaged into the JAR.
* **`build/`** is created by your compile script.

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

This produces a **single runnable JAR** containing your code and resources. On any system with a Java runtime, it runs with one command.

---

## 4. Shipping Self-Contained Apps

Instead of requiring users to install Java, you can ship your own minimal runtime:

```bash
jlink --add-modules java.base,java.desktop \
      --strip-debug --no-header-files --no-man-pages \
      --output runtime

jpackage --name MyApp --input . --main-jar app.jar --runtime-image runtime
```

* `jlink` produces a trimmed JRE with only the modules you need.
* `jpackage` bundles your app and runtime into a native installer (MSI, PKG, DEB, etc.).

---

## 5. Including External Tools (e.g., Tor, IPFS)

To bundle third-party tools without system dependencies:

1. Place their binaries (e.g., `tor.exe`, `ipfs.exe`) into `resources/bin/`.
2. Extract them at runtime to your app’s private folder.
3. Launch them with `ProcessBuilder`.
4. Communicate with them via sockets or APIs.

Example:

```java
Process tor = new ProcessBuilder(
  "bin/tor.exe", "-f", "etc/tor/torrc"
).start();
```

This way, users don’t install Tor/IPFS system-wide—you control versions and configs.

---

## 6. Why Not Maven or Gradle?

Maven and Gradle are build automation tools. They:

* Fetch and manage dependencies from online repositories.
* Handle multi-module projects.
* Integrate with CI/CD and code-quality plugins.

You don’t need them if:

* You have **few or no external libraries**.
* You’re shipping **self-contained apps**.
* You prefer **Makefiles or shell scripts** for automation.

You may want them if:

* You use many dependencies.
* You split into multiple modules.
* You require complex packaging or publishing pipelines.

---

## 7. Benefits of This Style

* **Simplicity:** Few moving parts, just the JDK and your code.
* **Transparency:** No hidden configs, every step is a command you understand.
* **Portability:** One JAR runs everywhere Java runs.
* **Control:** You choose exactly what is bundled (runtime, configs, binaries).
* **Reproducibility:** Your Makefile or batch script is the build pipeline.

---

## 8. Example Workflow

1. Write code in any text editor.
2. Run `make` (or your script) to:

   * Compile sources.
   * Copy resources.
   * Package JAR.
3. Test locally with `java -jar app.jar`.
4. Ship JAR or build per-OS installers with `jpackage`.

## 9. Logging and Integration
- The project needs its own root-level AGENTS.md which directs agents to read the styleguides from this submodule.
- The project should output verbose logs to a root level log.txt file, including all user output, user actions, and the results of user actions.
  - This enables LLMs to debug without trying to guess what's happening and because LLMs themselves are generally unable to interact with the ui directly.
- Tests should be defined in the root-level AGENTS.md file, and running the application should produce verbose enough output to facilitate testing during LLM runs.
- LLMs should use alternative run modes such as "-test" to simulate a broad range of passive user activity in order to facilitate testing without bogging down application startup by testing every feture during normal user startup modes. Test modes should not change user content such as database content.
