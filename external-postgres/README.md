# External-PostgreSQL
[Task description](https://jira.percona.com/browse/PMM-10913)
## How to use
* `make` - prints help message.
* `make init user=<test_user>|default_user` means that by default will be `default_user` if `user` flag is not provided. You can modify `user`, `sslmode`.. etc, by printing `make init sslmode=disable` separated by space.