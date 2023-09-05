# 🐘 Mastodon maintenance

Here are some scripts that i using to manage manstodon instance

## Install

To install on docker run following lines

```bash
git clone https://github.com/tootsuite/mastodon  
cd mastodon
cp .env.production.sample .env.production
cp docker-compose.yml prod-docker-compose.yml
docker-compose -f prod-docker-compose.yml build
```

Next you need to configure some keys

create secret key base and otp secret

```bash
# base
docker-compose -f prod-docker-compose.yml run --rm mastodon-web bundle exec rake secret
# otp
docker-compose -f prod-docker-compose.yml run --rm mastodon-web bundle exec rake secret
```
vappid key

```shell
docker-compose -f prod-docker-compose.yml run --rm mastodon-web bundle exec rake mastodon:webpush:generate_vapid_key
```

Rebuild image after update .env file

```shell
docker-compose -f prod-docker-compose.yml build
```

That is all on installation on docker

## Update

When you moving instance to another server go bottom to Restore

Stop mastodon

```bash
docker-compose -f prod-docker-compose.yml down -v
```

This will be used when updating  and of course clone from github and rebuild image

Migrate database

```bash
docker-compose -f applications/mastodon/docker-compose.yml run --rm -e SKIP_POST_DEPLOYMENT_MIGRATIONS=true mastodon-web rails db:migrate
```

Restart all services

```bash
docker-compose -f applications/mastodon/docker-compose.yml run --rm  mastodon-web rails db:migrate
```
Restart all services


## 📦 Backup

To backup database run

```bash
docker exec -t mastodon_mastodon-postgres_1 pg_dump -c -U postgres > mastodon_`date +%d-%m-%Y"_"%H_%M_%S`.sql mastodon_production
```

Next you have to backup mastodon data. If you using S3 storage then you can skip this because fileas are backed up by your provider.

## 🔃 Restore (optionlal)

Our server is backed up daily with restic. To restore backup run following command

```bash
restic restore <id-of-backup> --target /tmp/mastodon-restore
```

Restoring database

```bash
docker-compose -f prod-docker-compose.yml up -d mastodon-postgres
```

create user

```bash
docker exec -it mastodon_mastodon-postgres_1 psql --username postgres
CREATE DATABASE mastodon_production;
CREATE USER mastodon WITH PASSWORD '<your password>';
GRANT ALL PRIVILEGES ON DATABASE mastodon_production TO mastodon;
```

restore from backup

```bash
docker exec -i mastodon_mastodon-postgres_1 psql --username mastodon mastodon_production < /path/to/backup.sql
```

or

```bash
cat /path/to/backup.sql | docker exec -i mastodon_mastodon-postgres_1 psql --username mastodon mastodon_production
```

continue as on update. So rebuild image run migration and precompile packages.

## Create new admin (optional)

new admin

```shell
docker-compose -f prod-docker-compose.yml run --rm mastodon-web bin/tootctl accounts create emma --email emma@themaymeow.com --confirmed --role admin
```

## 😎 Add Mutatnt Emojis (optional)

Download emojis

```bash
wget https://mutant.tech/dl/2020.04/mtnt_2020.04_masto.tgz -O /path/to/emoji
```

Run following command on your docker container

```bash
docker-compose -f prod-docker-compose.yml run --rm -v /pat/to/emoji/:/mutant mastodon-web  bin/tootctl emoji import /mutant/mtnt_2020.04_masto.tgz
```