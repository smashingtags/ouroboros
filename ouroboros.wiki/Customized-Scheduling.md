# Scheduling

Ouroboros does not have a native scheduling implementation other than using `--interval`. This is due to there being more robust/customizable job schedulers being available such as:

- Cron
  - [Cron Tutorial](https://www.ostechnix.com/a-beginners-guide-to-cron-jobs/)
  - [Cron Expression Creator](https://crontab.guru/)
- Systemd Timers
  - [Documentation](https://wiki.archlinux.org/index.php/Systemd/Timers)

Example using ouroboros to update containers every Monday at 5AM:

**Docker**

```bash
* 5 * * 1 docker run --rm -d --name ouroboros -v /var/run/docker.sock:/var/run/docker.sock pyouroboros/ouroboros --interval 1 --run-once
```

**Pip installed CLI**

```bash
* 5 * * 1 ouroboros --interval 1 --run-once
```

Using the `--run-once` arg tells ouroboros to make one pass updating all/specified containers and then exit.
