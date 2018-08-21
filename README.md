﻿## Atlassian Jira Software

You can use **jira.config** to setup database config. 
It need only first run of the container (dbconfig.xml not exist)
Options for file jira.config

database type:
  - mysql
  - postgresql
  - mssql
  - oracle

all fields are mandatory!! 
  - type
  - host
  - port
  - user 
  - password
  - database
  - schema (only for postgresql, usually "public")

Simple line to start: 

    docker run -d -p 8080:8080 --rm --name jira q2digger/jira:latest

Line to start with ENV VARIABLES: 

    docker run -d -p 8080:8080 --rm --name jira \
      -e JVM_MINIMUM_MEMORY=2048m \
      -e JVM_MAXIMUM_MEMORY=4096m \
      -e CATALINA_CONNECTOR_PROXYNAME='jira.local.net' \
      -e CATALINA_CONNECTOR_PROXYPORT='443' \
      -e CATALINA_CONNECTOR_SCHEME='https' \
      q2digger/jira-software:latest

or you can use docker-compose.yml

Example **jira.config**

    [database]
    user = atlassian
    password = atlassian
    host = jiradb
    port = 5432
    name = jira
    type = postgresql
    schema = public

> Written with [StackEdit](https://stackedit.io/).

