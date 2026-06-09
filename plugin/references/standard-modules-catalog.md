# Bootique Standard Modules Catalog

The modules below are the ones whose versions are managed by the **`bootique-bom`** Bill of Materials. This catalog reflects the **`4.0-M3`** tag — re-check the BOM if you're on a different release.

```xml
<!-- import once, in <dependencyManagement> -->
<dependency>
    <groupId>io.bootique.bom</groupId>
    <artifactId>bootique-bom</artifactId>
    <version>4.0-M3</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

With the BOM imported, declare any module below **without a `<version>`** — the BOM pins it. Group ids vary per module family (e.g. `io.bootique.jersey`, `io.bootique.jdbc`); the artifact id is what's listed here.

Conventions in this catalog:

- **Runtime** — an auto-loadable `BQModule` you add to an app's runtime classpath.
- **Testing** — a `test`-scoped helper, almost always JUnit 5 based (`*-junit5`), `testcontainers`, or `wiremock`.
- **`-instrumented`** variants add `bootique-metrics` timers/health checks to the base module.

## Core

| Artifact (`io.bootique`) | Kind | Description |
|---|---|---|
| `bootique` | Runtime | The core: DI container, module loading, YAML config, CLI parsing, command dispatch. Every app depends on it. |
| `bootique-junit5` | Testing | JUnit 5 extensions for booting a `BQRuntime` inside tests and asserting on it. The base for most other `*-junit5` modules. |

## Web — Jetty (`io.bootique.jetty`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-jetty` | Runtime | Embedded Jetty server. Contributes the `--server` command, servlet/filter/listener injection. |
| `bootique-jetty-cors` | Runtime | CORS filter support for the Jetty server. |
| `bootique-jetty-websocket` | Runtime | JSR-356 WebSocket endpoint support on top of Jetty. |
| `bootique-jetty-instrumented` | Runtime | Jetty with request timing, thread-pool, and connection metrics. |
| `bootique-jetty-junit5` | Testing | Plugs Jetty server on an ephemeral port into a test BQRuntime for integration tests. Provides client to access Jetty |

## Web — JAX-RS / Jersey (`io.bootique.jersey`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-jersey` | Runtime | Jersey JAX-RS server integration; register resources and features via DI. |
| `bootique-jersey-jackson` | Runtime | Jackson JSON (de)serialization for Jersey endpoints. |
| `bootique-jersey-beanvalidation` | Runtime | Bean Validation (JSR-380) for JAX-RS resource parameters. |
| `bootique-jersey-client` | Runtime | Configurable JAX-RS `Client` (connection pools, timeouts, auth) injected from YAML. |
| `bootique-jersey-client-instrumented` | Runtime | The JAX-RS client with per-target request metrics. |
| `bootique-jersey-client-junit5-wiremock` | Testing | WireMock-backed stubs for testing code that uses the Jersey client. |

## Web — other

| Artifact | Group | Kind | Description |
|---|---|---|---|
| `bootique-mvc` | `io.bootique.mvc` | Runtime | Server-side MVC: maps JAX-RS resources to templated views. |
| `bootique-mvc-freemarker` | `io.bootique.mvc` | Runtime | FreeMarker template renderer for `bootique-mvc`. |
| `bootique-mvc-mustache` | `io.bootique.mvc` | Runtime | Mustache template renderer for `bootique-mvc`. |
| `bootique-cxf-core` | `io.bootique.cxf` | Runtime | Apache CXF core integration. |
| `bootique-cxf-jaxrs` | `io.bootique.cxf` | Runtime | CXF-based JAX-RS support. |
| `bootique-cxf-jaxws-server` | `io.bootique.cxf` | Runtime | SOAP/JAX-WS server endpoints via CXF. |
| `bootique-cxf-jaxws-client` | `io.bootique.cxf` | Runtime | SOAP/JAX-WS client proxies via CXF. |
| `bootique-tapestry59` | `io.bootique.tapestry` | Runtime | Apache Tapestry 5.9 web framework integration. |

## REST APIs — Agrest (`io.bootique.agrest`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-agrest5` | Runtime | Agrest 5 — generates REST endpoints directly from an ORM model (Cayenne). |
| `bootique-agrest5-swagger` | Runtime | OpenAPI/Swagger document generation for Agrest endpoints. |
| `bootique-agrest-junit5` | Testing | Test helpers for asserting on Agrest endpoint responses. |

## API docs — Swagger (`io.bootique.swagger`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-swagger` | Runtime | Serves an OpenAPI spec generated from JAX-RS resources. |
| `bootique-swagger-ui` | Runtime | Bundles Swagger UI to browse the served spec. |

## Persistence — JDBC (`io.bootique.jdbc`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-jdbc` | Runtime | `DataSource` management and YAML-configured connection pools. |
| `bootique-jdbc-hikaricp` | Runtime | HikariCP connection-pool implementation. |
| `bootique-jdbc-hikaricp-instrumented` | Runtime | HikariCP with pool metrics and health checks. |
| `bootique-jdbc-junit5` | Testing | DB test fixtures — table data load/inspect helpers. |
| `bootique-jdbc-junit5-derby` | Testing | Embedded Apache Derby for tests. |
| `bootique-jdbc-junit5-testcontainers` | Testing | Spins up real databases in Docker via Testcontainers for tests. |

## Persistence — ORM / SQL

| Artifact | Group | Kind | Description |
|---|---|---|---|
| `bootique-cayenne50` | `io.bootique.cayenne` | Runtime | Apache Cayenne 5.0 ORM integration. |
| `bootique-cayenne50-jcache` | `io.bootique.cayenne` | Runtime | JCache-backed query caching for Cayenne 5.0. |
| `bootique-cayenne50-junit5` | `io.bootique.cayenne` | Testing | Cayenne 5.0 test helpers (schema setup, data managers). |
| `bootique-cayenne42` | `io.bootique.cayenne` | Runtime | Apache Cayenne 4.2 ORM integration (legacy line). |
| `bootique-cayenne42-jcache` | `io.bootique.cayenne` | Runtime | JCache query caching for Cayenne 4.2. |
| `bootique-cayenne42-junit5` | `io.bootique.cayenne` | Testing | Cayenne 4.2 test helpers. |
| `bootique-jooq` | `io.bootique.jooq` | Runtime | jOOQ typesafe SQL integration. |
| `bootique-mybatis` | `io.bootique.mybatis` | Runtime | MyBatis SQL mapper integration. |

## Persistence — schema migrations

| Artifact | Group | Kind | Description |
|---|---|---|---|
| `bootique-liquibase` | `io.bootique.liquibase` | Runtime | Liquibase migrations; contributes `--lb-*` commands. |
| `bootique-flyway` | `io.bootique.flyway` | Runtime | Flyway migrations; contributes migration commands. |

## NoSQL / caching

| Artifact | Group | Kind | Description |
|---|---|---|---|
| `bootique-mongo-client` | `io.bootique.mongo` | Runtime | MongoDB client, configured from YAML. |
| `bootique-mongo-morphia` | `io.bootique.mongo` | Runtime | Morphia object-document mapper on top of the Mongo client. |
| `bootique-mongo-junit5` | `io.bootique.mongo` | Testing | MongoDB test fixtures. |
| `bootique-jcache` | `io.bootique.jcache` | Runtime | JSR-107 (JCache) caching integration. |

## Messaging

| Artifact | Group | Kind | Description |
|---|---|---|---|
| `bootique-kafka-client` | `io.bootique.kafka` | Runtime | Kafka producer/consumer clients from YAML config. |
| `bootique-kafka-streams` | `io.bootique.kafka` | Runtime | Kafka Streams topology integration. |
| `bootique-rabbitmq-client` | `io.bootique.rabbitmq` | Runtime | RabbitMQ connection, channel, and endpoint management. |
| `bootique-rabbitmq-junit5` | `io.bootique.rabbitmq` | Testing | RabbitMQ test helpers. |

## Jobs & scheduling (`io.bootique.job`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-job` | Runtime | Named jobs, scheduling, and the `--exec`/scheduler commands. |
| `bootique-job-instrumented` | Runtime | Job execution metrics. |
| `bootique-job-zookeeper` | Runtime | ZooKeeper-based cluster locks so a job runs on one node only. |
| `bootique-job-consul` | Runtime | Consul-based distributed job locking. |

## Security — Shiro (`io.bootique.shiro`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-shiro` | Runtime | Apache Shiro core: realms, subjects, authentication/authorization. |
| `bootique-shiro-jdbc` | Runtime | JDBC-backed Shiro realm. |
| `bootique-shiro-jwt` | Runtime | JWT token authentication. |
| `bootique-shiro-web` | Runtime | Shiro web integration (filters, session handling). |
| `bootique-shiro-web-jwt` | Runtime | JWT authentication for web apps. |
| `bootique-shiro-web-oidc` | Runtime | OpenID Connect authentication for web apps. |
| `bootique-shiro-web-mdc` | Runtime | Pushes the authenticated principal into the SLF4J MDC for logging. |

## Cloud — AWS (`io.bootique.aws`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-aws2` | Runtime | AWS SDK v2 core — region/credentials configured from YAML. |
| `bootique-aws2-s3` | Runtime | S3 client integration. |
| `bootique-aws2-s3-junit5` | Testing | S3 test helpers. |
| `bootique-aws2-secrets` | Runtime | AWS Secrets Manager integration. |
| `bootique-aws2-junit5` | Testing | AWS test helpers. |

## ETL — LinkMove (`io.bootique.linkmove`)

| Artifact | Kind | Description |
|---|---|---|
| `bootique-linkmove3` | Runtime | LinkMove 3 data-synchronization/ETL engine. |
| `bootique-linkmove3-json` | Runtime | JSON source connector for LinkMove. |
| `bootique-linkmove3-rest` | Runtime | REST source connector for LinkMove. |

## Observability

| Artifact | Group | Kind | Description |
|---|---|---|---|
| `bootique-logback` | `io.bootique.logback` | Runtime | Logback logging integration, configured from YAML. |
| `bootique-logback-smtp` | `io.bootique.logback` | Runtime | SMTP appender to email log events. |
| `bootique-metrics` | `io.bootique.metrics` | Runtime | Dropwizard Metrics integration (timers, meters, gauges). |
| `bootique-metrics-healthchecks` | `io.bootique.metrics` | Runtime | Health-check framework and reporting. |

## Infrastructure & misc

| Artifact | Group | Kind | Description |
|---|---|---|---|
| `bootique-curator` | `io.bootique.curator` | Runtime | Apache Curator / ZooKeeper client management. Used for job clustering among other things|
| `bootique-docker` | `io.bootique.docker` | Runtime | Docker client integration. |
| `bootique-simplejavamail` | `io.bootique.simplejavamail` | Runtime | Email sending via the Simple Java Mail library. |
