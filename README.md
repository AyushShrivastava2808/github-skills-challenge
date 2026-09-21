# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

## Task 1: Set Up and Understand the Environment

### AIOps Scenario

This assessment monitors a synthetic `payment-service` that produces operational metrics and
logs. The operations team needs to identify slow payment requests, high resource usage, and
service failures before they affect users.

AIOps is used to analyse the service telemetry, detect abnormal behaviour, create anomaly
events, and pass those events through the simulated producer, topic, and consumer workflow for
downstream processing.

### Repository Components

- `data/service_data.json`: synthetic operational data for the monitored service.
- `src/anomaly_detector.py`: checks metrics and log levels to identify anomalies.
- `src/event_producer.py`: publishes detected anomaly events.
- `src/event_topic.py`: provides the in-memory event topic used by the simulation.
- `src/event_consumer.py`: receives events from the topic for processing.
- `src/aiops_pipeline.py`: coordinates data loading, detection, event publishing, consumption,
  and final AIOps output.
- `tests/`: contains validation tests for the project components and workflow.

The repository is being used from the GitHub fork `AyushShrivastava2808/github-skills-challenge`
inside a GitHub Codespace with the default development-container configuration.

## Task 2: Analyse Logs and Metrics

The supplied `data/service_data.json` file contains 10 observations for `payment-service` from
`2026-09-20T10:00:00` through `2026-09-20T10:09:00`. Each observation is one minute apart, so
the timestamps establish the order and duration of the service behaviour being monitored.

### Data Fields

- Metrics: `response_time_ms` measures request latency, `cpu_percent` measures CPU usage, and
    `memory_percent` measures memory usage.
- Log information: `log_level` identifies the severity (`INFO` or `ERROR`) and `message`
    describes the request result or failure.
- Context fields: `timestamp` identifies when the observation occurred, and `service` identifies
    the monitored application.

### Observations

The eight records from 10:00–10:04 and 10:07–10:09 appear normal. They have `INFO` logs with
successful payment messages, response times from 120–150 ms, CPU usage from 42–50%, and memory
usage from 51–57%.

The two records from 10:05–10:06 appear unusual:

| Timestamp | Metrics | Log information | Observation |
| --- | --- | --- | --- |
| 10:05 | 610 ms response, 75% CPU, 70% memory | `ERROR`: Payment service timeout | Response time is much higher than the normal range and the request timed out. |
| 10:06 | 640 ms response, 94% CPU, 91% memory | `ERROR`: Database connection timeout | Response time remains high, CPU and memory are elevated, and the database connection timed out. |

The data therefore shows a short incident between 10:05 and 10:06, followed by a return to
normal-looking values at 10:07.

## Task 3: Identify Anomalies

The provided `AnomalyDetector` was run against all 10 records using its existing threshold-based
architecture. It uses thresholds of 500 ms for response time, 80% for CPU, and 80% for memory.
The detector was corrected to treat `WARNING`, `ERROR`, and `CRITICAL` as concerning log levels;
the supplied incident records use `ERROR`.

### Detection Report

Two anomalies were detected and eight normal observations were not flagged:

| Timestamp | Detected reasons | Evidence |
| --- | --- | --- |
| 10:05 | High response time; Error log detected | 610 ms response time and `ERROR` message: Payment service timeout. |
| 10:06 | High response time; High CPU utilization; High memory utilization; Error log detected | 640 ms response time, 94% CPU, 91% memory, and `ERROR` message: Database connection timeout. |

The result is readable because every anomaly event includes its timestamp, service, anomaly type,
reasons, and original source record. No expected anomaly was missed, and no normal `INFO` event
was incorrectly flagged in the supplied data.

One limitation is that the detector uses fixed thresholds rather than learning a service-specific
baseline. A rolling baseline or adaptive thresholds could reduce false positives when normal
traffic patterns change and could identify gradual degradation below the fixed limits.

## Task 4: Verify the AIOps Event Flow

The event-processing workflow uses the existing in-memory architecture:

- Event: the detector creates a structured `ANOMALY` message containing the timestamp, service,
    reasons, and original source record.
- Producer: `EventProducer.publish()` accepts the anomaly event and publishes it.
- Topic: `EventTopic("anomaly-events")` stores the published messages in memory.
- Consumer: `EventConsumer.consume()` reads the messages from the same topic.
- AIOps processing: `EventConsumer.process()` converts each consumed event into a downstream
    status, service, timestamp, and issue summary.

The producer and consumer were initially connected to different topics, so the consumer received
no events. They were corrected to share the `anomaly-events` topic. Package-compatible imports
were also added so the workflow can run from the repository root with `python3 -m`.

### Task 4 Execution Result

Running `python3 -m src.aiops_pipeline` produced this result:

```text
Records processed: 10
Anomalies detected: 2
Events consumed: 2
AIOps Output:
processed: payment-service at 2026-09-20T10:05:00
processed: payment-service at 2026-09-20T10:06:00
```

This confirms the complete flow: operational data -> anomaly event -> producer -> shared topic
-> consumer -> downstream AIOps output. The event-flow tests also pass with `5 passed`.

## Task 5: Investigate and Correct the Workflow

The following problems were found by tracing the provided components and rerunning the affected
workflow after each correction:

| Component | Cause | Correction | Verification |
| --- | --- | --- | --- |
| `AnomalyDetector` | The supplied rule checked only `WARNING`, while the data uses `ERROR` for both incident records. | Recognize `WARNING`, `ERROR`, and `CRITICAL` log levels as concerning events. | The detector report includes `Error log detected` for both incident timestamps and leaves all eight `INFO` records unflagged. |
| `EventProducer` / `EventConsumer` / `EventTopic` | The producer published to `service-events`, but the consumer listened to a separate `anomaly-events` topic. | Create one shared `anomaly-events` topic and pass it to both producer and consumer. | The pipeline now reports 2 detected events and 2 consumed events. |
| `src` imports | Modules used top-level imports that failed when the pipeline was run as the package module from the repository root. | Use package-relative imports with a script-execution fallback. | `python3 -m src.aiops_pipeline` executes successfully from the repository root. |
| `EventConsumer` / downstream output | The consumer returned messages but did not process them for the downstream AIOps stage. | Add `process()` and return `aiops_output` from the pipeline. | The final run reports two `processed` AIOps outputs and the event-flow tests pass. |

The corrected workflow was executed again after the changes. It processed 10 records, detected 2
anomalies, consumed 2 events, produced 2 downstream AIOps outputs, and passed all 5 pipeline
tests. No external Kafka, Airflow, cloud service, or replacement architecture was introduced.

## Task 6: Execute the End-to-End Pipeline

The corrected pipeline was executed after the workflow investigation. The complete path was:

`Operational Data -> Anomaly Detection -> Event -> Producer -> anomaly-events Topic -> Consumer -> AIOps Output`

The execution report verified every required stage:

| Verification | Result |
| --- | --- |
| Operational data processed | PASS: 10 records processed |
| Abnormal behaviour detected | PASS: 2 anomalies detected |
| Anomaly events generated | PASS: both events have type `ANOMALY` |
| Events published and consumed | PASS: 2 events consumed from the topic |
| Events processed successfully | PASS: 2 outputs have status `processed` |
| Final output represents the issue | PASS: outputs include timestamps, service, and anomaly reasons |

The final outputs identify the payment-service timeout at 10:05 and the database connection
timeout with high CPU and memory usage at 10:06. The complete execution finished with overall
status `SUCCESS`.

