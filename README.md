<p align="center">
  <img src="docs/branding/ember-runner.svg" width="250" alt="EMBER usage runner logo"/>

  <h1 align="center">EMBER Usage (Runner)</h1>
</p>

GitHub Action-controlled CRON jobs that extract the EMBER-DANDI S3 access logs and publish the access summaries
from the self-hosted Typhon runner (moved from [CodyCBakerPhD/self-hosted-crons](https://github.com/CodyCBakerPhD/self-hosted-crons)).
They mirror the DANDI workflows in [brain-bbqs-runners/dandi-usage-runner](https://github.com/brain-bbqs-runners/dandi-usage-runner),
pointed at the EMBER-DANDI API and the `ember-dandi-access-summaries` repository.
The install, the commands and the upload run through [CodyCBakerPhD/dandi-s3-logs-action](https://github.com/CodyCBakerPhD/dandi-s3-logs-action) `@v2`,
and the pre-run check through [CodyCBakerPhD/check-runner-action](https://github.com/CodyCBakerPhD/check-runner-action) `@v1`.

| Workflow | Schedule |
| --- | --- |
| Run EMBER log extraction | Daily, 10:00 UTC |
| Update EMBER summaries | Sundays, Wednesdays and Saturdays, 16:00 UTC |
| Update unassociated EMBER summaries | 1st of each month, 13:00 UTC |
| Install EMBER extraction packages (creates the environment on Python 3.14 if needed; `recreate` rebuilds it) | Manual dispatch only |

## Configuration

A self-hosted runner with the labels `typhon` and `ember` must be registered to this repository.

The workflows work under `/mnt/s3-logs/ember-s3-logs` on the runner's host, which holds `uv`, the environment (`env`,
created and filled by the install workflow), `extraction_cache`, `flocks` and the `ember-dandi-access-summaries` clone.
The path is written into the workflows, so no repository variables are needed.

Repository secrets:

- `S3_LOG_EXTRACTION_PASSWORD`
- `MAXMIND_ACCOUNT_ID` / `MAXMIND_LICENSE_KEY`
- `IPS_TO_SKIP` (required by the extraction): a regex of requester IPs to leave out. The extractor treats an empty value
  as matching every IP and would record nothing, so with none to leave out, set it to `^$`.
- `_GITHUB_API_KEY` (token able to list this repository's runners, for the pre-run check)
- `MAIL_USERNAME` / `MAIL_PASSWORD` (failure notifications)

The summaries map EMBER content IDs to Dandisets through
[ember-cache/content-id-to-usage-dandiset-path](https://github.com/ember-cache/content-id-to-usage-dandiset-path)
(its `dist` branch), named in the summaries workflows. The plugin's default is DANDI's cache, which holds no EMBER content.
