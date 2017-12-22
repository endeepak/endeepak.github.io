---
layout: post
title: "To cron or not to cron"
date: 2017-12-22 12:16:01 +0530
comments: true
categories: monitoring devops jenkins
---

Using crontab for scheduling background jobs like database backup, purging old data is a common practice in lot of IT organizations. As these jobs are running in background, failures can get missed easily without proper monitoring. In most cases, failures occur due to an error in job script and some times due to mistakes in cron configuration

<!-- More -->

Monitoring short lived cron jobs are not straight forward compared to monitoring long running services like web services. These are few well-known mechanisms for alerting cron job failures

* Cron job writes to a file on success. Monitoring system regularly reads this file and alerts based on last modified time-stamp of the file -[[Refer](https://serverfault.com/questions/33145/techniques-to-monitor-cron-tasks)]
* Cron job pings a [monitoring service](https://deadmanssnitch.com/) on success. This monitoring service acts as a [dead man's switch](https://en.wikipedia.org/wiki/Dead_man%27s_switch) and alerts if there was no call made by the job within expected time interval


### Pragmatic alternative for crontab

If you have a CI server like jenkins in your infrastructure, one of the approach that has worked well for us is - create a scheduled job in jenkins with slack/email notifications on failures.

Advantages of this approach

* Simpler to setup and effective for catching job script failures
* UI to check job output and history of previous runs
* Ability to manually trigger the job when needed

### Important note

Whether you use scheduled jobs in crontab or jenkins, you shouldn't depend only on the job's exit status for determining success - [[Refer](https://about.gitlab.com/2017/02/01/gitlab-dot-com-database-incident/)]. It is important to have alerting based on expected state of the system after job execution. *Example*: Timestamp of latest backup file uploaded to backup storage, minimum size of the backup file etc
