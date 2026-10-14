<p align="center">
  <img src="docs/branding/ember-runner.svg" width="250" alt="EMBER usage runner logo"/>
  <h3 align="center">Runner for the EMBER-DANDI Usage stats.</h3>
</p>

GitHub Action-controlled CRON jobs that extract the EMBER-DANDI S3 access logs and publish the access summaries
from the self-hosted Typhon runner (moved from [CodyCBakerPhD/self-hosted-crons](https://github.com/CodyCBakerPhD/self-hosted-crons)).
They mirror the DANDI workflows in [brain-bbqs-runners/dandi-usage-runner](https://github.com/brain-bbqs-runners/dandi-usage-runner),
pointed at the EMBER-DANDI API and the `ember-dandi-access-summaries` repository.
The commands and the upload run through [CodyCBakerPhD/dandi-s3-logs-action](https://github.com/CodyCBakerPhD/dandi-s3-logs-action) `@v1`,
and the pre-run check through [CodyCBakerPhD/check-runner-action](https://github.com/CodyCBakerPhD/check-runner-action) `@v1`.

| Workflow | Schedule |
| --- | --- |
| Run EMBER log extraction | Daily, 10:00 UTC |
| Update EMBER summaries | Sundays, Wednesdays and Saturdays, 16:00 UTC |
| Update unassociated EMBER summaries | 1st of each month, 13:00 UTC |

## Configuration

A self-hosted runner with the labels `typhon` and `ember` must be registered to this repository.

Repository variables:

- `EMBER_S3_LOG_BASE_PATH` (holds `uv`, the environment, `extraction_cache`, `flocks` and the `ember-dandi-access-summaries` clone)
- `S3LOGS_ENV_NAME` (the environment under that path, with `dandi_s3_log_extraction` installed)
- `CONTENT_ID_TO_USAGE_DANDISET_PATH_URL` (cache mapping EMBER content IDs to Dandisets; DANDI's default holds no EMBER content)

Repository secrets:

- `S3_LOG_EXTRACTION_PASSWORD`
- `MAXMIND_ACCOUNT_ID` / `MAXMIND_LICENSE_KEY`
- `IPS_TO_SKIP` (optional)
- `_GITHUB_API_KEY` (token able to list this repository's runners, for the pre-run check)
- `MAIL_USERNAME` / `MAIL_PASSWORD` (failure notifications)
