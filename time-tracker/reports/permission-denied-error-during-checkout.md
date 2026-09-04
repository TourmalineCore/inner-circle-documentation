# Report: Permission Denied Error During Checkout

## Context
When we were working on support of GitHub Actions container feature by self-hosted runner we faced a problem with checkout step in [pipeline](https://github.com/TourmalineCore/inner-circle-time-ui/actions/runs/32846287537/job/98489024036). The problem was that if after running a pipeline that uses a container, you ran a pipeline from a different branch, a `File was unable to be removed Error: EACCES: permission denied, unlink '/home/runner/actions-runner/_work/inner-circle-time-ui/inner-circle-time-ui/.git/refs/heads/feature/use-self-hosted-runners-in-pipelines'` error would occur during checkout.

## Cause of error
The problem occurs because, when using a DinD self-hosted runner together with the GitHub Actions container feature, the job container is created by the host Docker daemon, while the runner itself runs inside a container as the `runner` user. The working directory therefore has to be mounted from the host into the job container.

The job container run as `root`, so files created in the mounted working directory are owned by `root` on the host. When the next pipeline starts, the GitHub Actions Runner performs the checkout and workspace cleanup as the `runner` user. Since that user does not own the files created by the previous container job, the runner can fail to remove them with EACCES: permission denied.

## Decision

After running the job, change the owner of the working directory on the runner side so the pipelines don't have to be modified.

In the GitHub self-hosted runner, there is a variable called `ACTIONS_RUNNER_HOOK_JOB_COMPLETED` that allows the execution of custom scripts after a job has finished running.

In the docker-compose.yml, added the `ACTIONS_RUNNER_HOOK_JOB_COMPLETED` variable and the path to the script.
```yaml
environment:
  ACTIONS_RUNNER_HOOK_JOB_COMPLETED: /home/runner/actions-runner/hooks/change-owner-after-job.sh
```

In the `change-owner-after-job.sh` file, we use the chown command, which changes the owner of the working directory.

```bash
#!/bin/bash

sudo chown -R "$(id -u)" /home/runner/actions-runner/_work
```