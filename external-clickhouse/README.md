# External-Clickhouse
[Task description](https://jira.percona.com/browse/PMM-10391)
## How to use
* `make` - prints help message.
* `make init with-volume=<0,1>|1` means that by default will be `1 or true` if `with-volume` flag is not provided. You can modify `with-volume`, `percona`.. etc, by printing `make init percona=<percona-server-image>` separated by space.