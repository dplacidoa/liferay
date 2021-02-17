FROM mysql:5.7
ENV MYSQL_DATABASE liferay
COPY ./config/my.cnf /etc/mysql/my.cnf
COPY ./config/create-mysql.sql /docker-entrypoint-initdb.d/create-mysql.sql
