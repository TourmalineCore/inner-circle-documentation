# 006: Sensitive Logs Naming Convention

## Status

Accepted (2026-07-27)

## Context

We need the sensitive pipeline logs to be stored on our server. To do this, we need to determine how they will be stored.

## Desicion

The file path will include the project and log type, and the log file itself will contain the creation date, service name, run ID, and job ID.

Example: `/var/log/inner-circle/e2e/2026-07-22-inner-circle-time-api-run-27459153975-job-81169369662.log`

Where `inner-circle` is the project

- `e2e` is the log type
- `2026-07-22` is the launch date
- `inner-circle-time-api` is the service name
- `run-27459153975` is the pipeline run ID
- `job-81169369662` is the job ID

### Advantages

- Files are conveniently sorted by date and service, and thanks to unique identifiers, logs will not be overwritten.

### Disadvantages

- After a while, there will be a lot of files stored in a single directory, which can make it difficult to find the logs you need.

## Alternatives

### 1. The file name contains only the date and time

Example: `/var/log/inner-circle/e2e/inner-circle-time-api-2026-07-22T09-44-55.log`

#### Advantages

- The file name is always unique.
- It's fairly easy to match the pipeline run time to the corresponding file.

#### Disadvantages

- GitHub does not provide a way to retrieve information about when a pipeline was launched, so you'll need to capture the time while the pipeline is running. Because of this, there may be cases where a pipeline starts right at the minute mark, causing the actual start time and the time in the filename to differ by one minute.
- It's hard to determine which job of pipeline is related to this log.

### 2. The date is treated as a directory, and the name contains only the run ID and the job ID

Example: `/var/log/inner-circle/2026-07-22/e2e/inner-circle-time-api-run-27459153975-job-81169369662.log`

#### Advantages

- The logs are grouped by date, so there won't be a long list of files.

#### Disadvantages

- It's inconvenient to sort logs by date and service.
