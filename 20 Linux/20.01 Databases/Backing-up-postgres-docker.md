# Backing up (and restore) postgres

Following snippets you can use as follows or you can use them as part of scripts for automatic backups.

## I running postgres in Docker

To backup Postgres running on Docker image use

```bash
## to dump all databases
docker exec -t <container-name> pg_dumpall -c -U postgres > dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql

## or for single database
docker exec -t <container-name>  pg_dump -c -U ${PG_USER} > dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql
```

To restore backup you can run


``` bash
cat /path/to/backup.sql | docker exec -i mastodon_mastodon-postgres_1 psql --username mastodon mastodon_production

# or
docker exec -i <container-name> psql --username mastodon mastodon_production < /path/to/backup.sql
```

## I don't running postgres in Docker

Simirally you can backup and restore without docker too:

```bash
## to dump all databases
pg_dumpall -c -U postgres > dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql

## or for single database
pg_dump -c -U ${PG_USER} > dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql
```

To restore backup you can run


``` bash
cat /path/to/backup.sql | psql --username mastodon mastodon_production

# or
psql --username mastodon mastodon_production < /path/to/backup.sql
```