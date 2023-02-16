user ?= example_user
database ?= example_database
password ?= example_password
sslmode ?= verify-ca
postgres ?= postgres:14.1
percona ?= perconalab/pmm-server-fb:PR-3053-5388b36
disable-inner-db ?= true

default: help

help:                                                                           ## Display this help message.
	@echo "Please use \`make <target>\` where <target> is one of:"
	@grep '^[a-zA-Z]' $(MAKEFILE_LIST) | \
	    awk -F ':.*?## ' 'NF==2 {printf "  %-26s%s\n", $$1, $$2}'

ip = $(shell docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' example-percona-server)

up: gen compose-up                                                              ## Compose up external postgres and percona-server.
	@echo http://$(ip)

compose-up:
	docker compose up --detach --renew-anon-volumes --remove-orphans
	
down:                                                                           ## Compose down up external postgres and percona-server.
	docker compose down --remove-orphans --rmi local

all: up                                                                         ## Initialize and compose up external postgres and percona-server. `make all user=<user>|example_user database=<database_name>|example_database password=<password>|example_password sslmode=<sslmode>|verify-ca postgres=<version>|postgres:14.1 postgres ?= postgres:14.1 percona=<percona-server-image>|perconalab/pmm-server-fb:PR-3053-5388b36 disable-inner-db=<1,0>|1`

gen: init certificate-authority external-postgres percona-server remove-srl     ## Regenerates all variables needed to up.
clean: down                                                                     ## Clean-up project to initial state.
	rm -rf ./certificate-configurations \
		./external-postgres-certificates \
		./percona-server-certificates \
		./queries \
		./docker-compose.yml \
		./external-postgres-configuration \
		./Dockerfile.Postgres \
		./Dockerfile.Percona

certificate-authority:
	openssl req -new -sha256 -nodes -newkey rsa:2048 \
		-config ./certificate-configurations/certificate_authority.cnf \
		-keyout /tmp/certificate_authority.key \
		-out /tmp/certificate_authority.csr
	openssl x509 -req -days 365 -sha256 \
		-in /tmp/certificate_authority.csr  \
		-extfile /etc/ssl/openssl.cnf -extensions v3_ca \
		-signkey /tmp/certificate_authority.key \
		-out ./external-postgres-certificates/certificate_authority.crt 
	cp ./external-postgres-certificates/certificate_authority.crt ./percona-server-certificates/certificate_authority.crt

external-postgres:
	openssl req -new -sha256 -nodes -newkey rsa:2048 \
		-config ./certificate-configurations/external_postgres.cnf \
		-keyout ./external-postgres-certificates/external_postgres.key \
		-out /tmp/external_postgres.csr
	openssl x509 -req -days 365 -sha256 \
		-extfile ./certificate-configurations/external_postgres.cnf -extensions req_ext \
		-CA ./external-postgres-certificates/certificate_authority.crt -CAkey /tmp/certificate_authority.key -CAcreateserial \
		-in /tmp/external_postgres.csr \
		-out ./external-postgres-certificates/external_postgres.crt

percona-server:
	openssl req -new -sha256 -nodes -newkey rsa:2048 \
		-config ./certificate-configurations/percona_server.cnf \
		-keyout ./percona-server-certificates/percona_server.key \
		-out /tmp/percona_server.csr
	openssl x509 -req -days 365 -sha256 \
		-CA ./percona-server-certificates/certificate_authority.crt -CAkey /tmp/certificate_authority.key -CAcreateserial \
		-in /tmp/percona_server.csr \
		-out ./percona-server-certificates/percona_server.crt

ifneq ($(sslmode), disable)
  withHBADocker = COPY ./external-postgres-configuration/pg_hba.conf /external-postgres-configuration/pg_hba.conf
  withHBACompose = -c hba_file=/external-postgres-configuration/pg_hba.conf
  withSSL= -c ssl=on
endif

init:
	rm -rf ./external-postgres-certificates ./percona-server-certificates ./queries ./external-postgres-configuration ./certificate-configurations
	mkdir ./external-postgres-certificates ./percona-server-certificates ./queries ./external-postgres-configuration ./certificate-configurations
	cp ./templates/docker-compose.yml ./docker-compose.yml
	cp ./templates/percona_server.cnf ./certificate-configurations/percona_server.cnf
	cp ./templates/certificate_authority.cnf ./certificate-configurations/certificate_authority.cnf
	cp ./templates/external_postgres.cnf ./certificate-configurations/external_postgres.cnf
	cp ./templates/pg_hba.conf ./external-postgres-configuration/pg_hba.conf
	cp ./templates/init.sql ./queries/init.sql
	cp ./templates/Dockerfile.Postgres ./Dockerfile.Postgres
	cp ./templates/Dockerfile.Percona ./Dockerfile.Percona
	sed -i "s,__USER__,$(user),g" ./docker-compose.yml \
		./certificate-configurations/percona_server.cnf \
		./external-postgres-configuration/pg_hba.conf \
		./queries/init.sql
	sed -i "s,__DATABASE__,$(database),g" ./docker-compose.yml \
		./queries/init.sql
	sed -i "s,__PASSWORD__,$(password),g" ./docker-compose.yml \
		./queries/init.sql
	sed -i "s,__SSL_MODE__,$(sslmode),g" ./docker-compose.yml
	sed -i "s,__WITH_HBA__,$(withHBADocker),g" ./Dockerfile.Postgres
	sed -i "s,__WITH_SSL__,$(withSSL),g" ./docker-compose.yml
	sed -i "s,__WITH_HBA__,$(withHBACompose),g" ./docker-compose.yml
	sed -i "s,__PERCONA_SERVER__,$(percona),g" ./Dockerfile.Percona
	sed -i "s,__POSTGRES_VERSION__,$(postgres),g" ./Dockerfile.Postgres
	sed -i "s,__PMM_DISABLE_BUILTIN_POSTGRES__,$(disable-inner-db),g" ./docker-compose.yml

remove-srl:
	rm ./external-postgres-certificates/certificate_authority.srl ./percona-server-certificates/certificate_authority.srl
