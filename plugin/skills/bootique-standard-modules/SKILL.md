---
name: bootique-standard-modules
description: "Use this skill whenever the user wants to know which standard modules Bootique provides, or to pick the right one for a need — runtime integrations (web, persistence, messaging, jobs, security, cloud) and their testing counterparts. Trigger on phrases like 'what modules does Bootique have', 'is there a Bootique module for X', 'how do I add a web server / database / Kafka / scheduler to my Bootique app', 'which bootique-* artifact do I need for JDBC / Jersey / Cayenne / Shiro', 'what testing helpers does Bootique ship', 'add Bootique to talk to PostgreSQL / MongoDB / RabbitMQ / S3', or any mention of a `bootique-*` artifact or the `bootique-bom`. Also trigger when the user describes a capability (REST API, OpenAPI docs, migrations, metrics, health checks, email, ETL) and asks how to get it in Bootique. The skill maps the stated need to the right module via a static catalog and explains how to declare it via the BOM without pinning versions."
---

# bootique-standard-modules

Help the user discover and choose among the standard modules managed by the `bootique-bom` Bill of Materials, and explain how to declare them.

## Required reading

- `${CLAUDE_PLUGIN_ROOT}/references/standard-modules-catalog.md` — the full catalog (grouped by capability, with a one-line description and runtime/testing marker for each module). This is the source of truth; read it before answering.

## How to use it

1. **Map the need to a module.** The user usually describes a capability ("I need to expose a REST API", "talk to Postgres", "run scheduled jobs", "send email") rather than an artifact name. Find the matching group in the catalog and name the specific artifact(s). If several fit, list them with the one-line distinction (e.g. `bootique-jersey` vs `bootique-mvc`, HikariCP vs base JDBC).

2. **Separate runtime from testing.** Many families ship a runtime module plus a `*-junit5` / `testcontainers` / `wiremock` testing companion. When recommending a runtime module, mention its test helper if one exists, and note that it's `test`-scoped.

3. **Explain how to declare it.** The version is managed by `bootique-bom`, so:
   - Import the BOM once in `<dependencyManagement>` (see the catalog header for the snippet and the current version — `4.0-M3` as captured).
   - Add the module dependency **without a `<version>`**. Use the group id shown in the catalog and the listed artifact id.

4. **Don't invent modules.** If nothing in the catalog matches, say so plainly — the capability may live in a third-party module outside the BOM, or not exist. Don't guess an artifact name.

## Notes

- The catalog is pinned to the **`4.0-M3`** tag of `bootique-bom`. If the user is on a different Bootique release, the set may differ — offer to re-check the BOM for their version rather than trusting the snapshot.
- `-instrumented` variants layer `bootique-metrics` onto a base module; recommend them only when the user wants metrics/health checks.
- For *editing* an app's config surface or *running* it, defer to the `bootique-config` and `bootique-run` skills — this skill is about module selection only.
