# Full Setup Instructions

### Configuration File

**The configuration file needs to be provided by you!**

Don't worry, this is easy to do.

The app requires a configuration file to let it know what database you're using.

Here's an example configuration for `mysql` (or mariadb) that is compatible with the docker-compose example below:

```json
{
  "database": {
    "engine": "mysql",
    "host": "db",
    "name": "npm",
    "user": "npm",
    "password": "npm",
    "port": 3306
  }
}
```

Once you've created your configuration file it's easy to mount it in the docker container.

**Note:** After the first run of the application, the config file will be altered to include generated encryption keys unique to your installation. These keys
affect the login and session management of the application. If these keys change for any reason, all users will be logged out.


### Database

This app doesn't come with a database, you have to provide one yourself. Currently only `mysql/mariadb` is supported for the minimum versions:

- MySQL v5.7.8+
- MariaDB v10.2.7+

It's easy to use another docker container for your database also and link it as part of the docker stack, so that's what the following examples
are going to use.

::: warning

When using a `mariadb` database, the NPM configuration file should still use the `mysql` engine!

:::


### Running the App

Via `docker-compose`:

```yml
version: "3"
services:
  app:
    image: jc21/nginx-proxy-manager:2
    restart: always
    ports:
      # Public HTTP Port:
      - '80:80'
      # Public HTTPS Port:
      - '443:443'
      # Admin Web Port:
      - '81:81'
    volumes:
      # Make sure this config.json file exists as per instructions above:
      - ./config.json:/app/config/production.json
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db
  db:
    image: jc21/mariadb-aria:10.4
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "npm"
      MYSQL_DATABASE: "npm"
      MYSQL_USER: "npm"
      MYSQL_PASSWORD: "npm"
    volumes:
      - ./data/mysql:/var/lib/mysql
```

Then:

```bash
docker-compose up -d
```


### Running on Raspberry PI / ARM devices

The docker images support the following architectures:
- amd64
- arm64
- armv7

The docker images are a manifest of all the architecture docker builds supported, so this means
you don't have to worry about doing anything special and you can follow the common instructions above.

Check out the [dockerhub tags](https://cloud.docker.com/repository/registry-1.docker.io/jc21/nginx-proxy-manager/tags)
for a list of supported architectures and if you want one that doesn't exist,
[create a feature request](https://github.com/jc21/nginx-proxy-manager/issues/new?assignees=&labels=enhancement&template=feature_request.md&title=).

Also, if you don't know how to already, follow [this guide to install docker and docker-compose](https://manre-universe.net/how-to-run-docker-and-docker-compose-on-raspbian/)
on Raspbian.


### Initial Run

After the app is running for the first time, the following will happen:

1. The database will initialize with table structures
2. GPG keys will be generated and saved in the configuration file
3. A default admin user will be created

This process can take a couple of minutes depending on your machine.


### Default Administrator User

```
Email:    admin@example.com
Password: changeme
```

Immediately after logging in with this default user you will be asked to modify your details and change your password.
