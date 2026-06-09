---
name: bootique-run
description: "Use this skill whenever the user wants to start, launch, or execute a Bootique 4.0 Java application — whether to run a webserver, execute DB migrations, execute a single named job, kick off the scheduler, or just inspect output from `--help` or `--help-config`. Trigger on phrases like 'start the app', 'run my Bootique app', 'execute the X job', 'kick off the scheduler', 'what does --help show', 'what config does this expose', 'dump --help-config', 'run the server', or any request that involves running a Bootique-built jar. The skill locates the runnable jar in a Maven multi-module repo (handling both 'lib folder' and 'jar with dependencies' recipes), builds the jar if needed, and invokes `java -jar` with the right arguments. For long-running processes (servers, schedulers) it backgrounds the process; for short-running commands (jobs, help) it streams output inline."
---

# bootique-run

Build (if needed) and run a Bootique 4.0 Java app. Three common shapes of invocation:

- **Long-running**: server, scheduler, anything that doesn't terminate on its own.
- **Short-running migration**: `--lb-update`.
- **Short-running job**: `--exec -j <jobName>` or a custom command that terminates.
- **Inspection**: `--help`, `--help-config`, or a custom command whose only effect is printing.

## Required reading

- `${CLAUDE_PLUGIN_ROOT}/references/bootique-overview.md` — CLI flags and how commands are contributed.
- `${CLAUDE_PLUGIN_ROOT}/references/runnable-jar-discovery.md` — locating the runnable jar in Maven.

## Step 1 — Locate the runnable jar

Follow `runnable-jar-discovery.md`. Cache the result for the rest of the session. If multiple runnable modules exist and the user's request is ambiguous, ask which one.

## Step 2 — Ensure the jar is built

Check whether `<module>/target/<finalName>.jar` exists. If it doesn't, or if the user has clearly edited code since the last build:

```bash
mvn -pl <module-path> -am install -DskipTests
```

`-DskipTests` is fine for this skill's purposes (running, not validating). If the user is running a job to verify behavior end-to-end, ask before skipping tests.

If the build fails, surface the failure in full and stop. Don't try to run a stale jar.

## Step 3 — Pick the invocation shape

| User intent | Command |
|---|---|
| Print available commands and flags | `java -jar <jar> --help` |
| Print the merged config schema | `java -jar <jar> --help-config` |
| Run a single named job | `java -jar <jar> --exec -j <jobName>` (add more `-j <name>` for multiple) |
| Start a webserver | `java -jar <jar> --server` (when `bootique-jetty` is on the classpath) |
| Run a custom command | `java -jar <jar> <command-name>` (discover via `--help` first if unsure) |

A few clarifications:

- **`--server` is not universal.** It exists only when the app loads `bootique-jetty`. If `--help` shows it, use it; if not, the app may have a different entry command. Read `--help`, don't assume.
- **`--exec` without `-j`** runs every job declared by the app. That's almost never what the user wants — always pair `--exec` with `-j <name>` unless the user explicitly says "run all jobs."
- **`--config=<path>`** is layered with each later file overriding earlier ones. If the user mentions a config file or environment overlay, pass it; if they don't and there's a default the app already loads from the classpath, you don't need to specify anything.

## Step 4 — Run it

### Short-running (help, single job that terminates, custom command)

Run in the foreground and surface output to the user:

```bash
java -jar <module>/target/<finalName>.jar --help-config
```

For `--help-config` specifically, the output can be hundreds of lines. Show the user enough that they can navigate, and offer to focus on a subtree if it's overwhelming.

### Long-running (server, scheduler, never-terminating job)

Run in the background and report the PID + log location. Use the harness's background-execution facility — don't fork with `&` and lose the handle. After starting, give the process 2–5 seconds to either fail-fast (port collision, config error) or settle into "started" output, then report.

If the app exposes a startup log line ("Jetty started", "Scheduler running"), wait for it. If the app exits early, surface stderr/stdout and stop.

A reasonable pattern:

```bash
java -jar <module>/target/<finalName>.jar --server --config=<overlay.yml>
```

Include `--config=` only if the user mentioned a specific config or there's an overlay convention in the repo (e.g., `dev.yml`).

### Stopping a backgrounded app

If the user later says "stop it" or "kill it," kill by PID. Don't `pkill -f java` — there may be other JVMs running.

## Step 5 — Report cleanly

What the user wants to see depends on the invocation shape:

- **Help / config inspection**: the relevant slice of output. Trim it to the section the user asked about; offer the rest if they want more.
- **Job**: exit code plus the last meaningful log lines. If the job failed, surface the stack trace.
- **Server**: "Started on port X. Logs at <path>. PID: <pid>. Tell me when you want to stop it." Don't tail logs forever — that wastes the conversation. Tail only on request.

## Common failure modes

- **`Error: Unable to access jarfile <path>`** — wrong path or jar wasn't built. Re-check Step 2.
- **`No suitable command found`** — the user asked for a command (e.g., `--server`) that this app doesn't load. Run `--help` and report what is available.
- **Port already in use** — surface the message and suggest either freeing the port or overriding via the relevant config path (which you can find via the bootique-config skill or `--help-config`).
- **`--exec` runs more than expected** — they forgot `-j`. Tell them and re-run with the specific job name.
- **Long-running app exits immediately** — almost always config or classpath. Show stderr in full; don't paraphrase.
