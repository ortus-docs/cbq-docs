# Upgrade Guide

## v2 to v3

In v3, the `cbq_failed_jobs` table migrates the `failedDate` column type from a timestamp to a unix timestamp.

A migration file is included in `resources/database/migrations/2000_01_01_000006_use_unix_timestamp_for_failed_job_log_failedDate.cfc`. To run this migration, your failed jobs table will need to be empty. Alternatively, you can write your own migration that converts the timestamp to a unix timestamp. (The logic is different for each database grammar.)
