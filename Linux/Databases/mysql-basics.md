# MySQL Basics

Basic commands for MySQL and Mariadb

## Backing up database

Single database

```bash
mysqldump -u root -p database_name > database_name.sql
```

Multiple databases with `--databases`

```bash
mysqldump -u root -p --databases db_name1 db_name2 ...  > multi_database.sql
```

all databases with `--all-databases`

```bash
mysqldump -u root -p --all-databases > all-databases.sql
```

If you want to compress backup use command as follows


```bash
mysqldump -u root -p database_name | gzip > database_name.sql.gz
```

If you want to have time in file name you can add

```bash
`date +"%Y-%B-%d_%R"`

# `date +"%Y-%B-%d_%R"`.sql
```

## Restoring database

single database

> you need to create database before restoring
{.is-info}


```bash
mysql -u root -p database_name < database_name.sql
```

all databases is similart

> This will create your database when running
{.is-info}


```bash
mysql -u root -p < all-databases.sql
```
