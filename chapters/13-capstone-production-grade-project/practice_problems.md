# Chapter 13 — System Design / Architecture Decision Exercises

The capstone isn't about small coding drills anymore — it's about the design judgment that separates a senior engineer from someone who can implement well-specified tickets. Work through these exercises in writing (a page or two each is plenty). Use TaskFlow as the concrete system to reason about unless a problem says otherwise.

---

1. **Design the database schema for multi-tenant TaskFlow.** TaskFlow currently serves one organization. Design a schema (tables, keys, indexes) to support multiple tenants (organizations) with their own users, teams, and tasks, fully isolated from each other. Decide between shared-schema-with-tenant-id, schema-per-tenant, or database-per-tenant, and justify your choice against TaskFlow's expected scale.

2. **Write an ADR: sync vs async for the task-notification path.** TaskFlow needs to notify assignees when a task is assigned to them (email/push/in-app). Write a short Architecture Decision Record choosing between handling this synchronously inside the request path vs. asynchronously (background task, queue, or event). Include context, the options considered, the decision, and the consequences (including what you're giving up).

3. **Design the API versioning scheme.** TaskFlow's API needs to evolve without breaking existing clients (e.g., a mobile app on an older version). Design a versioning scheme (URL path, header-based, or content negotiation) and specify exactly how you'd introduce a breaking change to the `Task` resource under your scheme, including a deprecation and sunset timeline.

4. **Design for horizontal scale.** Assume TaskFlow needs to handle 100x its current traffic. Identify the first three bottlenecks you'd expect to hit (be specific — not "the database," but which query pattern or which resource) and design a mitigation for each, in priority order.

5. **Write an ADR: choosing a caching strategy.** The `GET /tasks` list endpoint has become a hot path. Write an ADR deciding between no caching, in-process caching, and a shared cache (Redis), including how you'd handle cache invalidation when a task is updated, and what staleness window is acceptable.

6. **Design the authorization model.** TaskFlow needs role-based access (e.g., admin, team lead, member) plus resource-level permissions (a user can only edit tasks in teams they belong to). Design the data model and the enforcement point(s) — where in the layered architecture does an authorization check belong, and why there rather than elsewhere?

7. **Design a zero-downtime migration for a breaking schema change.** You need to split a single `assignee_name` string column into a proper foreign key to a `users` table, on a live production database that cannot go down. Lay out the sequence of migration steps (expand/contract pattern) and what each deployed version of the application code needs to tolerate at each step.

8. **Write an ADR: message queue choice for the event pipeline.** (Pairs with the Chapter 12 Microservices track, but answer it even if you didn't build that track.) Choose between RabbitMQ and Kafka for TaskFlow's task-lifecycle events, considering expected throughput, need for replay, number of independent consumers, and operational complexity your team can realistically support.

9. **Design the observability plan for an on-call rotation.** If TaskFlow were handed to an on-call engineer tonight, what dashboards, alerts, and runbook entries would need to exist for them to diagnose "the API is returning 500s for 5% of requests" without waking you up? Specify the actual alert thresholds and the first three diagnostic steps the runbook should contain.

10. **Design the disaster-recovery plan.** Define TaskFlow's RPO (recovery point objective) and RTO (recovery time objective), and design a concrete backup and restore strategy for the database that meets them. Include how you'd test that restores actually work, not just that backups are being taken.
