# Zabbix maintenance

## Upgrade Server

> Upgrading to v6.0 LTS needs downtime because it need to upgrade database server to version 13
{.is-warning}

> Backup database before upgrade
{.is-danger}


```bash
git pull origin master
docker-compose -f applications/zabbix/docker-compose.yml pull
docker-compose -f applications/zabbix/docker-compose.yml up -d # to apply all changes
```

### Upgrade from 5.4 to 6.0

> ‼️ **Backup database before upgrade!** Server needs to upgrade from v12 to v13 so you will have to destroy old server and delete data.
{.is-danger}

```bash
git pull origin master
docker-compose -f applications/zabbix/docker-compose.yml down # take down all services
docker-compose -f applications/zabbix/docker-compose.yml pull
```

Move old database files somewhere else so you server will create empty database.

```bash
cd /var/opt/zabbix/
sudo mv data/ data-bak
```

> You can delete theese files once you server is restored and you found everything is running as intended.
{.is-info}


Start database server

```bash
docker-compose -f applications/zabbix/docker-compose.yml up -d zabbix-postgres-server
```

Start all services

```bash
docker-compose -f applications/zabbix/docker-compose.yml up -d # to apply all changes
```

## Database backup and restore

To backup run

```bash
docker exec -t zabbix_postgres_server pg_dump -c -U zabbix > zabbix_dump-5.4.sql zabbix
```

To restore run:

```bash
cat zabbix_dump-5.4.sql | docker exec -i zabbix_postgres_server psql --username zabbix zabbix
```