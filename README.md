# ember-usage-runner
Runner for the EMBER-DANDI Usage stats.

GitHub Action-controlled CRON job that extracts the EMBER-DANDI S3 access logs and publishes the access summaries
from the self-hosted Typhon runner
(moved from [CodyCBakerPhD/self-hosted-crons](https://github.com/CodyCBakerPhD/self-hosted-crons)).
The extraction commands and the upload run through
[CodyCBakerPhD/dandi-s3-logs-action](https://github.com/CodyCBakerPhD/dandi-s3-logs-action) `@v1`.

| Workflow | Schedule |
| --- | --- |
| Process EMBER logs (extraction, summaries, totals, upload) | Daily, 10:00 UTC |

## Configuration

A self-hosted runner with the labels `typhon` and `ember` must be registered to this repository.

Repository variables:

- `S3LOGS_ENV_PATH` (Python environment holding the extraction packages and `duct`)
- `S3LOGS_UV_PATH` (`uv` executable)

Repository secrets:

- `S3_LOG_EXTRACTION_PASSWORD`
- `MAXMIND_ACCOUNT_ID` / `MAXMIND_LICENSE_KEY`
- `_GITHUB_API_KEY` (token able to list this repository's runners, for the pre-run check)
- `MAIL_USERNAME` / `MAIL_PASSWORD` (failure notifications)
