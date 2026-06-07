# AGENTS.md — project-batch (Spring Boot Batch Processor)

## Project

- **Name:** batch-processor
- **Purpose:** Background job processing. Order fulfillment, email dispatch, report generation, data cleanup.
- **Repo:** github.com/panomete/batch-processor
- **Trigger:** RabbitMQ messages + cron schedules

## Stack

- **Language:** Java 21
- **Framework:** Spring Boot 3.3 + Spring Batch 5
- **Messaging:** RabbitMQ (job queue)
- **Database:** PostgreSQL 16 (job repository + business data)
- **Scheduling:** Cron (Spring `@Scheduled`)
- **Container:** Docker

## Project Map

```
src/main/java/com/panomete/batch/
├── BatchApplication.java         — Entrypoint (not a web server)
├── config/
│   ├── BatchConfig.java          — JobRepository, TaskExecutor
│   └── RabbitConfig.java         — Queue bindings
├── job/                           — Batch jobs
│   ├── OrderFulfillmentJob.java
│   ├── ReportGenerationJob.java
│   └── DataCleanupJob.java
├── step/                          — Individual steps within jobs
│   ├── ValidateOrderStep.java
│   ├── ProcessPaymentStep.java
│   └── SendNotificationStep.java
├── listener/
│   └── JobExecutionListener.java — Log start/end, alert on failure
└── scheduler/
    └── JobScheduler.java          — Cron triggers for periodic jobs

src/main/resources/
├── application.yml
└── db/migration/                  — Batch schema + business tables
```

## Commands

```bash
./mvnw clean package -DskipTests
./mvnw test
./mvnw spring-boot:run     # Starts listening for messages
```

## Key Concepts

### Job Structure

```
Job
├── Step 1: Validate     (skip on validation error)
├── Step 2: Process      (retry 3x on transient failure)
└── Step 3: Notify       (always runs, even if previous steps failed)
```

### Idempotency

Every job must be idempotent. Duplicate messages happen. Use:

- `message_id` deduplication in Redis
- Optimistic locking on entity version
- `WHERE status = 'PENDING'` in update queries

### Error Handling

| Failure Type | Strategy |
|-------------|----------|
| Transient (timeout, deadlock) | Retry 3x with exponential backoff |
| Validation (bad data) | Skip + log + dead-letter queue |
| Permanent (broken config) | Fail job + alert on-call |
| Poison message | Move to DLQ after 3 retries |

## Constraints

- No REST endpoints. This is a worker, not a web server.
- No long-running transactions. Chunk-based processing (100 items/chunk).
- Memory: never load entire dataset. Stream or paginate.
- Jobs must report progress (items processed, failures, duration).
