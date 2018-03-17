# Docker Zabbix Stack

1. Set localhost variable

    ```shell
    LOCALHOST="127.0.0.1"
    ```

1. Pull all resources

    ```shell
    docker pull mysql/mysql-server:latest
    docker pull zabbix/zabbix-server-mysql:latest
    docker pull zabbix/zabbix-web-nginx-mysql:latest
    ```

1. Start mysql server for zabbix database

    ```shell
    docker run \
      --name=zabbix-database \
      -p "${LOCALHOST}:3306:3306" \
      -p "${LOCALHOST}:33060:33060" \
      -d \
      mysql/mysql-server:latest
    ```

1. Enter mysql shell

    ```shell
    docker inspect zabbix-database | grep GENERATED
    docker exec -it zabbix-database mysql -uroot -p
    ```

1. Set root users's mysql password to something else and create zabbix user

    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'REDACTED'
    CREATE USER 'zabbix'@'%' IDENTIFIED BY '6a61d727e3d0b418a2e2d56142294b756a1dbebc';
    GRANT ALL PRIVILEGES ON *.* TO 'zabbix'@'%';
    quit;
    ```

1. Find docker-IP address of mysql server

    ```shell
    docker inspect zabbix-database | awk '/IPAddress/ {print $2}'
    ```

1. Start zabbix server

    ```shell
    docker run \
      --name=zabbix-server \
      -p "0.0.0.0:10051:10051" \
      -p "0.0.0.0:10050:10050" \
      -e DB_SERVER_HOST="172.17.0.2" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="6a61d727e3d0b418a2e2d56142294b756a1dbebc" \
      -d \
      zabbix/zabbix-server-mysql:latest
    ```

1. Find docker-IP address of zabbix-server

    ```shell
    docker inspect zabbix-server | awk '/IPAddress/ {print $2}'
    ```

1. Start zabbix webserver

    ```shell
    docker run --name zabbix-webserver \
      -p "0.0.0.0:26443:443" \
      -p "0.0.0.0:26080:80" \
      -e DB_SERVER_HOST="172.17.0.2" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="6a61d727e3d0b418a2e2d56142294b756a1dbebc" \
      -e ZBX_SERVER_HOST="172.17.0.3" \
      -e PHP_TZ="EST" \
      -d \
      zabbix/zabbix-web-nginx-mysql
    ```