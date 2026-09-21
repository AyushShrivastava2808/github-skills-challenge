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

