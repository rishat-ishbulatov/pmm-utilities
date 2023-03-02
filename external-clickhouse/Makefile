with-volume ?= 1
clickhouse ?= clickhouse/clickhouse-server:22.6.9.11-alpine
percona ?= perconalab/pmm-server-fb:PR-3101-72c36d8

default: help

help:                                                                           ## Display this help message.
	@echo "Please use \`make <target>\` where <target> is one of:"
	@grep '^[a-zA-Z]' $(MAKEFILE_LIST) | \
	    awk -F ':.*?## ' 'NF==2 {printf "  %-26s%s\n", $$1, $$2}'

ip = $(shell docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' example-percona-server)

up: gen compose-up                                                              ## Compose up external clickhouse and percona-server.
	@echo http://$(ip)

compose-up:
	docker compose up --detach --renew-anon-volumes --remove-orphans

down:                                                                           ## Compose down up external clickhouse and percona-server.
	docker compose down --remove-orphans --rmi local

all: up                                                                         ## Initialize and compose up external clickhouse and percona-server. `make all clickhouse=<clickhouse-image>|clickhouse/clickhouse-server:22.6.9.11-alpine percona=<percona-server-image>|perconalab/pmm-server-fb:PR-3101-72c36d8 with-volume=<0,1>|1`

gen: init                                                                       ## Regenerates all variables needed to up.
clean: down                                                                     ## Clean-up project to initial state.
	rm -rf ./docker-compose.yml
	rm -rf ./db

ifeq ($(with-volume), 1)
  volume = volumes: [./db:/var/lib/clickhouse]
endif

init:
	cp ./templates/docker-compose.yml ./docker-compose.yml
	sed -i "s,__PERCONA_SERVER_VERSION__,$(percona),g" ./docker-compose.yml
	sed -i "s,__CLICKHOUSE_VERSION__,$(clickhouse),g" ./docker-compose.yml
	sed -i "s,__WITH_VOLUME__,$(volume),g" ./docker-compose.yml
