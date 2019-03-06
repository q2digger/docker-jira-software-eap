## Atlassian Jira Software EAP (Early Access Preview)
[Atlassian Developer blog](https://checkup.atlassian.com/server/jira/platform/)

An Early Access Program (EAP) release is a public development release leading up to the official release of a Jira version.
Development releases are a snapshot of our work in progress, primarily focused on allowing Jira users to see the new features in advance and provide us with some useful feedback. It also gives app developers an opportunity to test and fix their apps in advance of an official release.

What are the features of this container?
1. I added a driver to connect to MySQL databases
2. I updated the driver to connect to the Postgresql databases.
3. The container contains an automatic database connection setup script. Thus, when you first start the application, this dialog will be omitted.(example below)

### Simple line to start: 

```
    docker run -d -p 8080:8080 --rm --name jira q2digger/jira-software-eap:latest
```

Line to start with ENV VARIABLES (and declare frontend proxy-server): 
```
    docker run -d -p 8080:8080 --rm --name jira-eap \
      -e JVM_MINIMUM_MEMORY=2048m \
      -e JVM_MAXIMUM_MEMORY=4096m \
      -e CATALINA_CONNECTOR_PROXYNAME='jira.local.net' \
      -e CATALINA_CONNECTOR_PROXYPORT='443' \
      -e CATALINA_CONNECTOR_SCHEME='https' \
      q2digger/jira-software-eap:latest
```
or you can use docker-compose.yml (example below)

### Setup database during first run of the container

You can use **jira.config** to setup database config. 
It need only first run of the container (dbconfig.xml not exist)

Options for file ```jira.config```

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

### Example **jira.config**

```
    [database]
    user = atlassian
    password = atlassian
    host = jiradb
    port = 5432
    name = jira
    type = postgresql
    schema = public
```

### Example **docker-compose.yml**
```
version: '2'

services:
  proxy:
    image: jwilder/nginx-proxy
    ports:
      - 0.0.0.0:80:80
      - 0.0.0.0:443:443
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock
      - ./certs/:/etc/nginx/certs:ro
      - ./jira-nginx.conf:/etc/nginx/vhost.d/jira.local.net
    networks:
      - proxy
  jiradb:
    image: blacklabelops/postgres
    volumes:
      - _jiradb:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=atlassian
      - POSTGRES_DB=jira
      - POSTGRES_USER=atlassian
      - POSTGRES_ENCODING=UNICODE
      - POSTGRES_COLLATE=C
      - POSTGRES_COLLATE_TYPE=C
    networks:
      - jira
  jira:
    image: q2digger/jira-software-eap:latest
    ports:
      - 0.0.0.0:8080:8080
    volumes:
      - _jiradata:/var/atlassian/jira
      - _jiralogs:/opt/atlassian/jira/logs
      - ./jira.config:/opt/atlassian/jira/conf/jira.config
      - ./certs/:/ssl/root
    networks:
      - jira
      - proxy
    environment:
      JVM_MINIMUM_MEMORY: '2048m'
      JVM_MAXIMUM_MEMORY: '4096m'
      CATALINA_CONNECTOR_PROXYNAME: 'jira.local.net'
      CATALINA_CONNECTOR_PROXYPORT: '443'
      CATALINA_CONNECTOR_SCHEME: 'https'
      VIRTUAL_HOST: 'jira.local.net'
      VIRTUAL_PORT: '8080'

volumes:
  _jiradb:
  _jiradata:
  _jiralogs:

networks:
  jira:
  proxy:
```

### SSL
I prepare self-signed certs with command:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout jira.local.net.key -out jira.local.net.crt
```

> Written with [StackEdit](https://stackedit.io/).

