# What is Rapidoid?

Rapidoid is an extremely fast HTTP server and modern Java web framework / application container, with a strong focus on high productivity and high performance.

> [www.rapidoid.org](http://www.rapidoid.org)

%%LOGO%%

# How to use this image

To quickly start Rapidoid and display some basic usage help, run:

```console
$ docker run --rm %%REPO%% --help
```

Rapidoid can be used in different ways:

-	as a web tool, to quickly prototype RESTful web services from the command line
-	as a HTTP server, to serve static resources
-	as a Java web application framework/container, to deploy a web application JAR

## Quickly prototyping SQL-powered RESTful web services

To quickly prototype SQL-powered RESTful web services from the command line, you need to link the database container to the Rapidoid container. The MySQL containers should be linked under name `mysql`, and PostgreSQL containers under name `postgres`.

This example starts a new MySQL container and links it under name `mysql` in the Rapidoid container, where a RESTful service is defined by specifying SQL query for the route `GET /users`. The service returns the result (a list of MySQL users) in JSON format.

```console
docker run -d --name some-mysql -e MYSQL_ROOT_PASSWORD=db-pass mysql

docker run -it --rm \
    -p 8888:8888 \
    --link some-mysql:mysql \
    %%REPO%% \
    profiles=mysql \
    jdbc.password=db-pass \
    '/users <= SELECT user FROM mysql.user'
```

**Note:** Please wait for several seconds for the MySQL database to initialize, and then you can visit [http://localhost:8888/users](http://localhost:8888/users) (or `http://your-host:8888/users`) in your web browser.

**Syntax for the service prototyping arguments**:

```console
'[GET|POST|PUT|DELETE|PATCH] <uri> <= <sql>'
```

## Serving static files

Rapidoid will automatically serve static files from the folders: `/app/static` (or `/app/public`, which is deprecated). To serve the contents of the `/your-www-root` directory, please mount it as `/app/static`:

```console
$ docker run -it --rm \
    -p 8888:8888 \
    -v /your-www-root:/app/static \
    %%REPO%%
```

## Configuration

Rapidoid will try to read the configuration from `/app/config.yml`. The configuration can also be specified with command-line arguments or environment variables.

To configure a custom port (by default `8888`) for the default and the Admin server, run the following command. If `admin.port` is not configured, the default server is also used as Admin server, so only one port will be opened (`on.port`).

```console
$ docker run -it --rm \
    -p 4444:4444 \
    -p 9999:9999 \
    %%REPO%% \
    on.port=4444 \
    admin.port=9999 \
    app.services=ping \
    admin.services=status
```

Then you can visit [http://localhost:4444/\_ping](http://localhost:4444/_ping) (or `http://your-host:4444/_ping`) and [http://localhost:9999/\_status](http://localhost:9999/_status) (or `http://your-host:9999/_status`) in your web browser.

The same setup can be configured with environment variables:

```console
$ docker run -it --rm \
    -p 4444:4444 \
    -p 9999:9999 \
    -e ON_PORT=4444 \
    -e ADMIN_PORT=9999 \
    %%REPO%% \
    app.services=ping \
    admin.services=status
```

For more details, please see the [full list of configuration options and their default values](http://www.rapidoid.org/the-default-configuration.html).

## Security

Rapidoid's HMAC-based security token mechanism requires all containers to share the same secret key when scaling out a web application:

```console
$ docker run -it --rm \
    -p 8888:8888 \
    -e SECRET=your-secret-key \
    %%REPO%%
```

While this is an easy way to get started, for security reasons it is recommended to store the secret key in the `/app/config.yml` file, with proper permissions.

**Note:** For production use, you must replace `your-secret-key` with a real, private secret key.

**Note:** If no secret key is specified, a random secret key will be generated, which is acceptable when deploying a single container.

## Full bootstrap of Rapidoid's Admin Center

To bootstrap a full-blown Admin Center in Rapidoid, you will also need to configure a password for the built-in `admin` user:

```console
$ docker run -d \
    --restart=always \
    -p 8888:8888 \
    -e SECRET=your-secret-key \
    -e USERS_ADMIN_PASSWORD=admin-pass \
    %%REPO%% \
    admin.services=center
```

Please replace `admin-pass` with a real password for the `admin` user. Then you can login to the Admin Center by visiting [http://localhost:8888/\_](http://localhost:8888/_) (or `http://your-host:8888/_`) in your web browser.

**Note:** For production use, you must replace `your-secret-key` with a real, private secret key (please see the `Security` section).

# How to extend this image (application JAR deployment)

To use this image as base image for your web application, simply add your application JAR as `/app/app.jar`:

```dockerfile
COPY <location/of/your/webapp.jar> /app/app.jar
```
