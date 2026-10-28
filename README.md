<p align="center">
  <img src="docs/branding/ember-runner.svg" width="250" alt="EMBER usage runner logo"/>

  <h1 align="center">EMBER Usage (Runner)</h1>
</p>

GitHub Action-controlled CRON jobs that extract the EMBER-DANDI S3 access logs and publish the access summaries
from the self-hosted Typhon runner (moved from [CodyCBakerPhD/self-hosted-crons](https://github.com/CodyCBakerPhD/self-hosted-crons)).
They mirror the DANDI workflows in [brain-bbqs-runners/dandi-usage-runner](https://github.com/brain-bbqs-runners/dandi-usage-runner),
pointed at the EMBER-DANDI API and the `ember-dandi-access-summaries` repository.
The install, the commands and the upload run through [CodyCBakerPhD/dandi-s3-logs-action](https://github.com/CodyCBakerPhD/dandi-s3-logs-action) `@v3`,
and the pre-run check through [CodyCBakerPhD/check-runner-action](https://github.com/CodyCBakerPhD/check-runner-action) `@v1`.

| Workflow | Schedule |
| --- | --- |
| Run EMBER log extraction | Daily, 10:00 UTC |
| Update EMBER summaries | Sundays, Wednesdays and Saturdays, 16:00 UTC |
| Update unassociated EMBER summaries | 1st of each month, 13:00 UTC |
| Install EMBER extraction packages (creates the environment on Python 3.14 if needed; `recreate` rebuilds it; sets up restic) | Manual dispatch only |
| Scan EMBER extraction cache for excluded IPs (`validate excluded`, which fails, and so emails, if the cache still holds their requests) | Manual dispatch only |
| Remove excluded IPs from EMBER extraction cache (restic snapshot, then `reset excluded`, then `validate excluded`, which must come back clean; the removal is permanent, so un-excluding an IP later needs a full re-extraction) | Manual dispatch only |

## Configuration

A self-hosted runner with the labels `typhon` and `ember` must be registered to this repository.

The workflows work under `/mnt/s3-logs/ember-s3-logs` on the runner's host, which holds `uv`, the environment (`env`),
`extraction_cache`, `flocks` and the `ember-dandi-access-summaries` clone. The path is written into the workflows, so
no repository variables are needed. The remove workflow snapshots the cache with restic first, as the DANDI runner
does, using `snapshots/restic`, its password file `snapshots/.restic-password`, the repository `snapshots/backups`, and
`tmp` as its temporary directory.

On a fresh runner, dispatching the install workflow downloads `uv` and creates the environment. It also sets up restic
for the snapshots, each part only when it is missing: a pinned, checksum-verified binary at `snapshots/restic`, the
password file `snapshots/.restic-password`, and the repository `snapshots/backups`. An existing binary, password file or
repository, with its snapshots, is never replaced or deleted. A new password file takes the optional `RESTIC_PASSWORD`
secret, or else the placeholder `123`, as on the DANDI runner, since the snapshots never leave the host.

The first summaries run clones
[aplbrain/ember-dandi-access-summaries](https://github.com/aplbrain/ember-dandi-access-summaries) over HTTPS.
The clone and its pushes authenticate with the `SUMMARIES_REPOSITORY_TOKEN` secret, so the runner user needs no git
credentials of its own. The token reaches git through the step's environment (which needs git 2.31 or newer on the
runner's host) and is never written to the clone.

Repository secrets:

- `S3_LOG_EXTRACTION_PASSWORD`
- `RESTIC_PASSWORD` (optional): the password for a new restic repository, in place of the placeholder
- `MAXMIND_ACCOUNT_ID` / `MAXMIND_LICENSE_KEY`
- `SUMMARIES_REPOSITORY_TOKEN` (required by the summaries upload): a fine-grained token with the resource owner
  `aplbrain`, access to only `aplbrain/ember-dandi-access-summaries`, and the permission `Contents: Read and write`.
  Fine-grained tokens expire, and an expired one fails the upload, so renew it and update the secret.
- `_GITHUB_API_KEY` (token able to list this repository's runners, for the pre-run check)
- `MAIL_USERNAME` / `MAIL_PASSWORD` (failure notifications)

Requester IPs to leave out are listed on the runner's host in the runner user's
`~/.s3-log-extraction/excluded_ips.txt`, one address per line, with `#` comments. This file is the only list of
excluded addresses. The extraction drops their requests and every summaries workflow leaves them out, so no repository
secret is needed. If the DANDI runner runs as the same user on the host, the two runners share this one file.

Only exact addresses are accepted. Networks in CIDR notation and patterns are refused with an error. An absent or
empty file leaves no one out.

The scan workflow checks the cache for any of their requests extracted before they were listed, and fails if it finds
any. The remove workflow then deletes them from the cache.

Exclusion only works in one direction over time. Adding an address drops its requests from future extraction and,
once the remove workflow has run, from the cache. Removing an address from the file does not bring its requests back,
since they were never extracted or were deleted. To include an address again, the extraction has to be rerun from
scratch (`reset extraction`, then a full `extract`).

The summaries map EMBER content IDs to Dandisets through
[ember-cache/content-id-to-usage-dandiset-path](https://github.com/ember-cache/content-id-to-usage-dandiset-path)
(its `dist` branch), named in the summaries workflows. The plugin's default is DANDI's cache, which holds no EMBER content.
