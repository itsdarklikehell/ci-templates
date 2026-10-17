# systemd — CI runner user units

This directory contains systemd *user* unit templates for self-hosted CI
runner services, plus the runner script they execute.

## Units

- `systemd/ci-runner@.service` — template for a self-hosted CI runner
  instance. Instantiate as `ci-runner@1.service`, `ci-runner@2.service`, …
  Each instance runs `/home/rizzo/bin/ci-runner <instance>`, which polls a
  job queue and executes dropped shell scripts.

## Deployment (this host)

1. Install the unit into the user systemd directory:

       cp systemd/ci-runner@.service ~/.config/systemd/user/

2. Enable at boot. User services need linger enabled (already done on this
   host), then the instance is enabled into default.target:

       loginctl enable-linger $USER          # already set: Linger=yes
       systemctl --user enable ci-runner@1.service

3. Start the runner:

       systemctl --user start ci-runner@1.service

4. Logs are kept by journald (configured via the unit file):

       journalctl --user -u ci-runner@1.service
       journalctl --user -u ci-runner@1.service --since "1 hour ago"

## Restart policy

- `Restart=on-failure` — the runner restarts if it crashes or exits non-zero
- `RestartSec=10` — wait 10 seconds between restart attempts
- `StartLimitBurst=5` / `StartLimitIntervalSec=300` — at most 5 restarts
  per 5 minutes, preventing a tight restart loop on a broken runner
- `KillMode=control-group` / `TimeoutStopSec=30` — orderly shutdown on stop

## Logging

- `StandardOutput=journal` and `StandardError=journal` send all runner and
  job output to the user journal
- `SyslogIdentifier=ci-runner-%i` tags each instance separately in the log
- Job output is also captured to `$CI_RUNNER_JOBS_DIR/done/<ok|fail>/`

## Adding a job

Drop an executable shell script into the instance's queue:

       echo '#!/usr/bin/env bash' > ~/.local/state/ci-runner/1/queue/hello.sh
       echo 'echo "CI runner works"; exit 0' >> ~/.local/state/ci-runner/1/queue/hello.sh
       chmod +x ~/.local/state/ci-runner/1/queue/hello.sh
