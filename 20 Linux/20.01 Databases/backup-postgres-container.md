# Backup postgres container

I using following script to backup my postgres containers

``` bash
#!/bin/bash

# Backup data locations
backup_date=$(date +%d-%m-%Y"_"%H_%M_%S)
TEMP_DIR="/tmp"
MC_PATH="/usr/local/bin/mcli"

# Container and adatabase information
DOCKER_PG_CONTAINER="container-postgres_1"
PG_USER="postgres"
PG_DB_NAME="database"
S3_BUCKET="home/cloud-backup"

# Run Backup
echo "Creating database backup for ${PG_DB_NAME} in ${TEMP_DIR}"
docker exec -t ${DOCKER_PG_CONTAINER}  pg_dump -c -U ${PG_USER} > ${TEMP_DIR}/${PG_DB_NAME}_${backup_date}_daily.sql ${PG_DB_NAME}
sleep 1;
tar -czpf ${TEMP_DIR}/backup_${PG_DB_NAME}_${backup_date}.tar.gz ${TEMP_DIR}/${PG_DB_NAME}_${backup_date}_daily.sql

# Upload Backup to remote location
${MC_PATH} cp ${TEMP_DIR}/backup_${PG_DB_NAME}_${backup_date}.tar.gz ${S3_BUCKET}

# Send notification to monitoring
statuscode=$?
zabbix_sender -c /etc/zabbix/zabbix_agentd.conf -s hostname -k zabbix-server.tld -o ${statuscode}
```

To run it automatically move it to `/usr/local/sbin/script_name.sh`

``` bash
# Update permission
chown root:root /usr/local/sbin/script_name.sh
chmod 700 /usr/local/sbin/script_name.sh
```

Create new Cron task

``` bash
sudo cront tab -e
```

add at the end of file

``` bash
0 0 * * * /usr/local/sbin/run_backup.sh >> /var/log/backups.log
```

Script will run each day at 00:00. In case of errors i redirect output to file.