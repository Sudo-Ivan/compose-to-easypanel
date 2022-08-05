# Docker Compose to Easypanel

[compose-to-easypanel](https://npmjs.com/package/compose-to-easypanel)

Easypanel is a Beautiful, Easy to use Server Control Panel based on Docker [easypanel.io](https://easypanel.io).

with this package you are able to create an Easypanel Schema based on your docker-compose file

## Quickstart

### Via Npx

The CLI can be easily run via npx:

```sh
npx compose-to-easypanel <projectName> -i <docker-compose.yml> -o <output-file.json>
```

### Via Npm

```sh
npm install -g compose-to-easypanel
```

```sh
compose-to-easypanel <projectName> -i <docker-compose.yml> -o <output-file.json>
```

## Example

Creating an Mysql application with Adminer as Dashboard

### Write The Compose

`./docker-compose.yml`

```yml
version: "3"
services:
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080

  db:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: examplePassword
      MYSQL_PASSWORD: examplePasswordNonRoot
```

### Run the CLI

```sh
npx <mysql-adminer> -i ./docker-compose.yml -o ./mysql-adminer-schema.json
```

### Copy the Schema

after running this command you should be able to view the json schema under `./mysql-adminer-schema.json`

Generated Schema:

```json
{
  "services": [
    {
      "type": "app",
      "data": {
        "projectName": "test",
        "serviceName": "adminer",
        "source": { "type": "image", "image": "adminer" },
        "ports": [{ "published": 8080, "target": 8080 }]
      }
    },
    {
      "type": "mysql",
      "data": {
        "projectName": "test",
        "serviceName": "db",
        "rootPassword": "examplePassword",
        "password": "examplePasswordNonRoot"
      }
    }
  ]
}
```

### Create The Services

after you copied the schema you can go to your easypanel Dashboard. Go to `your-project`, select `templates` and scroll all the way down to **developer** an then click `Create from Schema`

Please note:
Domains and Proxys are currently not supported, you need to add these manually via the Easypanel dashboard

## Supported Docker Compose Properties

- `image`
- `container_name`
- `ports`
- `environment`
- `volumes`
- `command`

### Image

Docker Compose File

```yml
# ...
image: easypanel/easypanel
```

Generated Schema

```json
{
  "source": {
    "type": "image",
    "image": "easypanel/easypanel"
  }
}
```

### Container Name

The property `container_name` represents the `serviceName`, the default `serviceName` is the `key` of the `service` object in the docker-compose file

```yml
#...
service050:
  container_name: "my-super-container-name" # This is the serviceName
  image: easypanel/easypanel
  #...
```

```yml
#...
service050: # This is the serviceName
  image: easypanel/easypanel
  # ...
```

### Ports

docker-compose file:

```yml
# ...
ports:
  # outside:inside
  - 3000:3000
  - 8000:5000
```

Generated Schema:

```json
{
  "ports": [
    {
      "published": 3000,
      "target": 3000
    },
    {
      "published": 8000,
      "target": 5000
    }
  ]
}
```

### Environment

docker-compose file:

```yml
# ...
environment:
  SECRET: my-super-secret
  ANOTHER_SECRET: ohooho-secret
```

Generated Schema:

```json
{
  "env": "SECRET=my-super-secret\nANOTHER_SCRET=ohooho-secret"
}
```

**Its currently not supported to load your env variables through an .env file**

```yml
# ...
environment:
  SECRET: ${SECRET} # that won't work !
```

### Volumes

docker-compose file:

```yml
# ...
volumes:
  # outside:inside
  - ./my/bind:/etc/my/bind
  - my-volume:/etc/my/volume
```

Generated Schema:

```json
{
  "volumes": [
    {
      "type": "bind",
      "hostPath": "./my/bind",
      "mountPath": "/etc/my/bind"
    },
    {
      "type": "volume",
      "name": "my-volume",
      "mountPath": "/etc/my/volume"
    }
  ]
}
```

### Command

docker-compose file:

please specify the command as string!

```yml
# ...
command: yarn start # that works
# command: ["yarn", "start"] won't work !
```

Generated Schema:

```json
{
  "deploy": {
    "command": "yarn start"
  }
}
```

## Database Services

Easypanel supports all popular databases out of the box:

- `postgres`
- `mysql`
- `mongo`
- `redis`

The Cli automatically creates an database service if the official image is used, however sometimes you need to provide an custom image, to do that you need to provide and env Variable in your `docker-compose` named `EASYPANEL_DATABASE` with the value of one database service (`postgres`, `mysql`, `mongo`,`redis`)

you also need to provide an Password env variable, otherwise a new one is created

- Mongo: `MONGO_INITDB_ROOT_PASSWORD: <password>`
- Postgres: `POSTGRES_PASSWORD: <password>`
- Redis: `REDIS_PASSWORD: <password>`
- MySql: `MYSQL_ROOT_PASSWORD: <root-pw>; MYSQL_PASSWORD: <password>`

docker-compose file

```yml
db:
  image: "postgres"
  environment:
    POSTGRES_PASSWORD: "super-password"
```

Generated Schema

```json
{
  "type": "postgres",
  "data": {
    "password": "super-password"
  }
}
```

With custom image

```yml
db:
  image: "myuser/postgres"
  environment:
    EASYPANEL_DATABASE: postgres
    PASSWORD: "super-password"
```

Generated Schema

```json
{
  "type": "postgres",
  "data": {
    "image": "myuser/postgres",
    "password": "super-password"
  }
}
```

## Contribution

Contribution is always welcome :-)

1. Fork the Repo
2. Create a Branch from `master`
3. Edit the Source Code
4. Submit a PR
