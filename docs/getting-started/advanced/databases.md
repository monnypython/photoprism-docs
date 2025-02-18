# Advanced Database Setup

PhotoPrism is compatible with [MariaDB 10.5.10+](https://mariadb.org/) (earlier versions 
see [MDEV-25362](https://jira.mariadb.org/browse/MDEV-25362)), [MySQL 8](https://www.mysql.com/), 
and [SQLite 3](https://www.sqlite.org/).

!!! info
    Our [docker-compose.yml](https://dl.photoprism.org/docker/) examples include
    tested and working database configurations. These docs are for advanced users only.

## MariaDB ##

When creating a new database, make sure to set the charset and collation as follows:

```sql
CREATE DATABASE photoprism
CHARACTER SET = 'utf8mb4'
COLLATE = 'utf8mb4_unicode_ci';
```

Now create a user and grant privileges for this new database:

```sql
CREATE USER 'photoprism'@'%' IDENTIFIED BY 'insecure';
GRANT ALL PRIVILEGES ON photoprism.* to 'photoprism'@'%';
FLUSH PRIVILEGES;
```

Set the database environment variables for PhotoPrism as follows:

```yaml
services:
  photoprism:
    environment:
      PHOTOPRISM_DATABASE_DRIVER: "mysql"
      PHOTOPRISM_DATABASE_SERVER: "mariadb:3306"
      PHOTOPRISM_DATABASE_NAME: "photoprism"
      PHOTOPRISM_DATABASE_USER: "photoprism"
      PHOTOPRISM_DATABASE_PASSWORD: "insecure"
```

!!! note
    `mariadb:3306` needs to be replaced with the actual database server host and port, 
    unless you're using our [docker-compose.yml](https://dl.photoprism.org/docker/docker-compose.yml)
    example without modifications.

## Database Migrations ##

### MariaDB to Sqlite ###

- Install <https://github.com/techouse/mysql-to-sqlite3> on your host.
- Stop Photoprism: `docker-compose stop photoprism`
- Add the port to the MariaDB service
- On the **host** now run `sudo mysql2sqlite -f <PATH_TO_STORAGE_MOUNT>/storage/index.db -d photoprism -u root --mysql-password 'please_change'`
- Shutdown your current stack fully: `docker-compose down`
- Edit your `docker-compose.yml`:
  - Remove the MariaDB service.
  - Change in the Photoprism settings to use the sqlite driver and remove the other database settings
- Start your stack again with `docker-compose up -d`
- If this worked you may want to delete the old mountpoint for the MariaDB database.

### Sqlite to MariaDB ###

- Install <https://github.com/techouse/sqlite3-to-mysql> on your host. (openSUSE: `zypper in python-sqlite3-to-mysql`)
- Shutdown your current stack: `docker-compose down`
- Add the current snippet of the MariaDB to your Sqlite Photoprism Docker-Compose with the addition of the extra `ports`
  section where you expose port 3306 to the Host. In my case this looked like attachment 1.
- Start the stack again: `docker-compose up -d`
- Stop Photoprism: `docker-compose stop photoprism`
- On the **host** now run `sudo sqlite3mysql -f <PATH_TO_STORAGE_MOUNT>/storage/index.db -d photoprism -u root --mysql-password 'please_change'`
- Shutdown your current stack again: `docker-compose down`
- Edit your `docker-compose.yml` so it uses the MariaDB database you added before. Don't forget to remove the `ports`
  section of the MariaDB Container.
- Start your stack again with `docker-compose up -d`
- If this worked you may want to delete the file `index.db` in the `storage` mount since it contains out of date
  information.

Attachment 1:

```yml
mariadb:
    image: mariadb:10.5
    container_name: mariadb
    restart: unless-stopped
    ports:
      - 3306:3306
    security_opt:
      - seccomp:unconfined
      - apparmor:unconfined
    command: mysqld --transaction-isolation=READ-COMMITTED --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --max-connections=512 --innodb-rollback-on-timeout=OFF --innodb-lock-wait-timeout=50
    volumes: # Don't remove permanent storage for index database files!
      - "./database:/var/lib/mysql"
    environment:
      MYSQL_ROOT_PASSWORD: please_change
      MYSQL_DATABASE: photoprism
      MYSQL_USER: photoprism
      MYSQL_PASSWORD: insecure
```
