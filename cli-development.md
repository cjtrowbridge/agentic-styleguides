# CLI Logging and Interpretability Style Guide

This style guide outlines best practices for command‑line interface (CLI) development tools. Its goal is to make every action that your tool performs visible from the CLI so that both humans and large language models (LLMs) can understand and debug the workflow. The guidance in this document is language‑agnostic and can be applied to any project that exposes a command‑line surface, even if the project also ships a graphical user interface.

## 1. Philosophy

* **CLI first.** The CLI should be the primary interface for your tool. Even if a GUI exists, every operation must be executable from the command line so that automated agents and scripts can drive the tool without a GUI.
* **Transparency.** Everything that happens in your tool should be visible in the CLI. Hiding behaviour behind GUI elements or implicit side effects makes it impossible for developers and LLMs to reason about what the tool is doing.
* **Reproducibility.** A user (or agent) following the documented CLI commands should be able to reproduce any run of your tool. Avoid hidden state or reliance on external environment configuration when possible.
* **Cross-platform reliability.** All code must run correctly on both Windows and Linux without requiring manual tweaks or OS-specific forks.

## 2. Verbose logging

To support LLMs and developers in understanding your tool’s behaviour, every CLI‑based project must create a root‑level log directory (for example, `logs/`). Each run of the application must write to a brand‑new log file inside that directory named with the timestamp of when the run started using a sortable pattern such as `log.YYYY-MM-DD-HH-mm-ss-ffffff.txt`. Every per-run log file must contain:

* **User inputs and actions.** Log the exact command arguments or interactive input received.
* **Outputs and results.** Log what the tool prints to stdout/stderr and any side effects it performs (e.g. files written, network calls made, database queries executed). Include timestamps to aid debugging.
* **Contextual metadata.** Provide information about the component generating the log (module name, function, or class) and the phase of the operation.

Logs should be written in a plain‑text, append‑only format. The goal is to give agents complete visibility into what happened during execution, so do not reuse log files across runs—always start a fresh, timestamped file when the process begins. If multiple processes are spawned (for example, launching Tor or IPFS as sub‑processes), their stdout/stderr should also be captured and appended to that run’s log file.

## 3. Test mode and simulation

LLMs often need to verify that your tool behaves correctly without modifying real data. Provide a dedicated **test mode** (for example, via a `--test` or `-test` flag) that:

* Simulates a broad range of typical user actions.
* Produces the same verbose logs as a normal run.
* Does **not** modify any persistent user data (such as real databases or on‑disk files). Use temporary directories or in‑memory data structures in test mode.

When running in test mode, the tool should exercise enough code paths to make automated testing effective. This allows LLMs to verify functionality without incurring the overhead of running integration tests during every normal startup.

## 4. README.md and style guide references

Every repository that contains CLI‑based tools must include a comprehensive **`README.md`** file. The styleguide references must be listed at the top of the README.md so that agents will always see them first. At a minimum, the README.md should:

* List relevant styleguide files at the top with brief descriptions
* Describe the high‑level purpose of the project and its architecture.
* Explain where logs live (for example, the root‑level `logs/` directory and the timestamped files it contains) and how to run the project in normal and test modes.

Agents and developers should read `README.md` first to understand how to apply the various style guides in the repository.

## 5. Project roadmap and specification requirements

Every CLI‑based project must include:

* **`project-roadmap.md`** - Comprehensive development roadmap with all tasks and features
* **`project-specification.md`** - Detailed technical specification and requirements
* **`social-context.md`** - Social context, organization details, partnerships, and impact

### Roadmap requirements

The roadmap must include:

* **Status legend** at the top explaining checkbox meanings:
  - `[ ]` - Not started
  - `[?]` - In progress / Testing / Development  
  - `[x]` - Completed and tested
* **All major and minor tasks** broken down by phases
* **Current status** clearly marked with appropriate checkboxes
* **Update instructions** requiring roadmap updates whenever tasks change status

### Specification requirements

The specification must include:

* **Complete technical requirements** for all features
* **Architecture details** and component descriptions
* **Configuration schemas** and data formats
* **Success criteria** and acceptance tests
* **Security and privacy** requirements

### Social context requirements

The social context document must include:

* **Organization details** and mission statement
* **Partnerships and collaborations** with other organizations
* **Use cases and user stories** showing how the project helps people
* **Community impact** and social benefits
* **Future opportunities** and potential for related projects
* **Stakeholder information** and target audiences
* **Cultural and social considerations** relevant to the project
* **Accessibility and inclusion** aspects
* **Ethical considerations** and responsible development practices

All three documents must be kept current whenever project details change, features are added, or development status updates.

## 6. Logging best practices

The following guidelines apply across languages:

* **Consistent format.** Use a structured logging format (e.g. timestamps, log level, module name, message). Consistency makes parsing and analysis easier for tools.
* **Appropriate log levels.** Use `INFO` for normal operations, `WARN` for recoverable issues, and `ERROR` for serious problems. Do not hide exceptions—log stack traces at the `ERROR` level.
* **Context in messages.** Include enough detail in each log entry to understand what was happening. For example, log input parameters, intermediate results, and the outcome of operations.
* **Cross‑language adherence.** When your project consists of components in multiple languages, ensure that each component writes to the same per-run log file in the same format.

## 7. Integration with large language models (LLMs)

LLMs cannot see your GUI or internal state; they rely entirely on textual output. To make your tool LLM‑friendly:

* **Expose state via CLI.** Provide commands or flags that output the current configuration, status, or internal metrics of your tool. Avoid requiring an API or GUI for this.
* **Descriptive errors.** Write error messages that explain what went wrong and how to fix it. Avoid cryptic messages or silent failures.
* **Deterministic output.** When possible, avoid non‑deterministic ordering of logs (e.g. due to concurrency) that could confuse automated analysis. If concurrency is necessary, clearly label log lines with thread or process identifiers.

## 8. Example CLI workflow

Document a typical usage scenario for your tool. For example:

```bash
# run the tool normally and capture logs
./mytool --input data.txt --output results.txt

# inspect the newest log file
ls logs/
cat "$(ls -t logs/log.* | head -n 1)"

# run in test mode
./mytool --test

# run a command to dump current status
./mytool --status
```

## 9. Language-Specific Supplementary Styleguides
For development in these langauges, read and follow these additional styles:
- java-cli-development.md
- python-cli-development.md

By following this style guide, your CLI‑based development tools will remain transparent, debuggable, and compatible with automated agents as your project evolves.
