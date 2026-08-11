# Production Runtime Profile

## Scope

This profile records the verified BCC runtime deployed on the backup host as of
the `maintenance-20260811` recovery run. It is intentionally separate from
the bootstrap installer: copying a repository checkout to a host is not a
deployment procedure and must not automatically replace a live backup path.

## Components

- `scripts/atmed-bcc-run-backup`: scheduled BCC orchestration with safe
  informational command handling, fail-closed database-export integration,
  dual-provider snapshots, repository checks, and a bounded mini-restore.
- `scripts/atmed-bcc-backup-health`: post-run health evaluation.
- `scripts/atmed-bcc-maintenance-prune`: defined retention maintenance.
- `systemd/atmed-bcc-backup*`: scheduled backup and health units.
- `systemd/atmed-bcc-maintenance-prune.service`: an explicit maintenance unit.

## Deployment Boundary

The active host uses `/usr/local/bin` and `/etc/systemd/system`, while the
bootstrap installer uses a separate prefix. A future deployment change must:

1. compare the selected commit with the active host hashes;
2. make an independent backup of scripts and units;
3. validate shell syntax and systemd units;
4. install only explicitly selected files;
5. reload systemd without starting a new backup concurrently;
6. run the safe health and restore-verification path; and
7. retain a documented rollback point.

## Retention Safety

The recorded policy is 14 daily, 8 weekly, 12 monthly, and 3 yearly snapshots
per endpoint and provider. It must not be run before a fresh backup and restore
gate passes. The captured legacy prune implementation contains a broad
`restic unlock || true` path and an endpoint list that must be reconciled with
the active inventory before enabling or executing it. This repository record
does not authorize that operation.

## Secrets

The scripts contain references to protected local secret sources but no secret
values. Secret files, provider configuration, and runtime database state remain
outside Git.
