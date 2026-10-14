# ember-usage-runner
Runner for the EMBER-DANDI Usage stats.

GitHub Action-controlled CRON job that extracts the EMBER-DANDI S3 access logs and publishes the access summaries
from the self-hosted Typhon runner
(moved from [CodyCBakerPhD/self-hosted-crons](https://github.com/CodyCBakerPhD/self-hosted-crons)).

| Workflow | Schedule |
| --- | --- |
| Process EMBER logs (extraction, summaries, totals, upload) | Daily, 10:00 UTC |

## Configuration

A self-hosted runner with the labels `typhon` and `ember` must be registered to this repository.

Repository secrets:

- `S3_LOG_EXTRACTION_PASSWORD`
- `MAXMIND_ACCOUNT_ID` / `MAXMIND_LICENSE_KEY`
- `_GITHUB_API_KEY` (token able to list this repository's runners, for the pre-run check)
- `MAIL_USERNAME` / `MAIL_PASSWORD` (failure notifications)
