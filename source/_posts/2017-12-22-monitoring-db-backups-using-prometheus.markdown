---
layout: post
title: "Monitoring DB backups using prometheus"
date: 2017-12-22 14:19:02 +0530
comments: true
categories: devops prometheus monitoring
---

Ensuring correct database backups taken at regular interval is very critical for disaster recovery. Recent [gitlab database incident](https://about.gitlab.com/2017/02/01/gitlab-dot-com-database-incident/) re-emphasizes this fact. [Gitlab](https://about.gitlab.com/) was very transparent about this and [documented](https://gitlab.com/gitlab-com/www-gitlab-com/issues/1108) approaches for preventing these failures

The preventive measures include, monitoring -

* *Backup file is created in every x interval*: Catches backups not being uploaded due to backup script error or scheduling error
* *Size of latest backup file is at-least y bytes*: Catches erroneous backup file uploaded due to script error

<!-- More -->

In our case, DB backups are uploaded to Azure blob storage(similar to AWS S3) and [prometheus](https://prometheus.io/) is used for monitoring

High level design

* Run an exporter which can expose metrics such as `latest_file_timestamp` and `latest_file_size` for each blob container where backup files are uploaded
* Alert if `current_time - latest_file_timestamp > backup_interval` or `latest_file_size < expected_backup_file_size`

As we couldn't find any existing exporter, we wrote [prometheus-azure-blob-exporter](https://github.com/project-sunbird/prometheus-azure-blob-exporter) to capture following metrics

```
# Last modified timestamp(milliseconds) for latest file in container
azure_blob_latest_file_timestamp{container="postgresql-backup"} 1508707802000.0
# Size in bytes for latest file in container
azure_blob_latest_file_size{container="postgresql-backup"} 5606443.0
```

Alerts are defined as

* Check backup is created every day

```
# 24 hours + 1 hour slack time for backup process
# threshold_interval_in_milliseconds => 25 * 60 * 60 * 1000 => 90000000
ALERT backup_is_too_old
  IF (time() * 1000) - azure_blob_latest_file_timestamp > 90000000
  FOR 5m
  ANNOTATIONS {
      summary = "Backup is too old",
      description = "There is no backup file created for a day in {{ $labels.container }}",
  }
```

* Check latest backup file created has minimum size of 1MB

```
# threshold_size_in_bytes => 1MB =~ 1000000
ALERT backup_size_is_too_small
  IF azure_blob_latest_file_size < 1000000
  FOR 5m
  ANNOTATIONS {
      summary = "Backup size is too small",
      description = "Latest backup file is smaller than 1MB in {{ $labels.container }}",
  }
```

Please checkout the [github repo](https://github.com/project-sunbird/prometheus-azure-blob-exporter) for more details
