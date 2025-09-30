# CLI Logging and Interpretability Style Guide

This style guide outlines best practices for command‑line interface (CLI) development tools. Its goal is to make every action that your tool performs visible from the CLI so that both humans and large language models (LLMs) can understand and debug the workflow. The guidance in this document is language‑agnostic and can be applied to any project that exposes a command‑line surface, even if the project also ships a graphical user interface.

## 1. Philosophy

* **CLI first.** The CLI should be the primary interface for your tool. Even if a GUI exists, every operation must be executable from the command line so that automated agents and scripts can drive the tool without a GUI.
* **Transparency.** Everything that happens in your tool should be visible in the CLI. Hiding behaviour behind GUI elements or implicit side effects makes it impossible for developers and LLMs to reason about what the tool is doing.
* **Reproducibility.** A user (or agent) following the documented CLI commands should be able to reproduce any run of your tool. Avoid hidden state or reliance on external environment configuration when possible.

## 2. Verbose logging

To support LLMs and developers in understanding your tool’s behaviour, every CLI‑based project must produce a root‑level log file (for example, `log.txt`) containing:

* **User inputs and actions.** Log the exact command arguments or interactive input received.
* **Outputs and results.** Log what the tool prints to stdout/stderr and any side effects it performs (e.g. files written, network calls made, database queries executed). Include timestamps to aid debugging.
* **Contextual metadata.** Provide information about the component generating the log (module name, function, or class) and the phase of the operation.

Logs should be written in a plain‑text, append‑only format. Avoid rotating logs or deleting them mid‑run. The goal is to give agents complete visibility into what happened during execution. If multiple processes are spawned (for example, launching Tor or IPFS as sub‑processes), their stdout/stderr should also be captured and appended to the same log file.

## 3. Test mode and simulation

LLMs often need to verify that your tool behaves correctly without modifying real data. Provide a dedicated **test mode** (for example, via a `--test` or `-test` flag) that:

* Simulates a broad range of typical user actions.
* Produces the same verbose logs as a normal run.
* Does **not** modify any persistent user data (such as real databases or on‑disk files). Use temporary directories or in‑memory data structures in test mode.

When running in test mode, the tool should exercise enough code paths to make automated testing effective. This allows LLMs to verify functionality without incurring the overhead of running integration tests during every normal startup.

## 4. AGENTS.md and style guide references

Every repository that contains CLI‑based tools must include a root‑level **`AGENTS.md`** file. The purpose of this file is to orient both human developers and agentic tools. At a minimum, it should:

* Describe the high‑level purpose of the project and its architecture.
* Point to the relevant language‑specific development style guides (for example, `java-development.md` for Java projects) and this CLI style guide for logging and interpretability rules.
* Explain where logs live (`log.txt` or similar) and how to run the project in normal and test modes.

Agents and developers should read `AGENTS.md` first to understand how to apply the various style guides in the repository.

## 5. Logging best practices

The following guidelines apply across languages:

* **Consistent format.** Use a structured logging format (e.g. timestamps, log level, module name, message). Consistency makes parsing and analysis easier for tools.
* **Appropriate log levels.** Use `INFO` for normal operations, `WARN` for recoverable issues, and `ERROR` for serious problems. Do not hide exceptions—log stack traces at the `ERROR` level.
* **Context in messages.** Include enough detail in each log entry to understand what was happening. For example, log input parameters, intermediate results, and the outcome of operations.
* **Cross‑language adherence.** When your project consists of components in multiple languages, ensure that each component writes to the same log file in the same format.

## 6. Integration with large language models (LLMs)

LLMs cannot see your GUI or internal state; they rely entirely on textual output. To make your tool LLM‑friendly:

* **Expose state via CLI.** Provide commands or flags that output the current configuration, status, or internal metrics of your tool. Avoid requiring an API or GUI for this.
* **Descriptive errors.** Write error messages that explain what went wrong and how to fix it. Avoid cryptic messages or silent failures.
* **Deterministic output.** When possible, avoid non‑deterministic ordering of logs (e.g. due to concurrency) that could confuse automated analysis. If concurrency is necessary, clearly label log lines with thread or process identifiers.

## 7. Example CLI workflow

Document a typical usage scenario for your tool. For example:

```bash
# run the tool normally and capture logs
./mytool --input data.txt --output results.txt

# inspect the log file
cat log.txt

# run in test mode
./mytool --test

# run a command to dump current status
./mytool --status
```

## 8. Language-Specific Supplementary Styleguides
For development in these langauges, read and follow these additional styles:
- java-cli-development.md
- python-cli-development.md

By following this style guide, your CLI‑based development tools will remain transparent, debuggable, and compatible with automated agents as your project evolves.
