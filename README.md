# LaraDocker

This is a docker/docker-compose template project meant to simplify running an NGINX, PHP, MARIADB fullstack application.

While implicitly it's meant to work with [Laravel](https://laravel.com/), this is optional. The only strict dependency on the [./src](./src) directory is that there be a [./src/public/index.php](./src/public/index.php) file, because the [./nginx/default.conf](./nginx/default.conf) file is hard-coded to proxy there.

This consists of the following services...

| Service    | Purpose            | Implementation |
|------------|--------------------|----------------|
| `web`      | Web Server         | nginx          |
| `app`      | Application Server | php            |
| `database` | Database Server    | mariadb        |
| `mail`     | Mail Server        | mailpit        |

...including some utility services...

- [webgrind](https://github.com/jokkedk/webgrind)
  - "Webgrind is an Xdebug profiling web frontend in PHP."
- [composer](https://getcomposer.org/)
  - "A Dependency Manager for PHP"
- [npm](https://www.npmjs.com/)
  - "npm is the package manager for Node.js."
- [artisan](https://laravel.com/docs/master/artisan)
  - "Artisan is the command line interface included with [Laravel](https://laravel.com/)."

# User Permissions in Containers

It's important to have `USER_ID` and `GROUP_ID` environment variables declared which will enable `docker-compose` to share file permissions between the local system and the containers.

```shell
export USER_ID=$(id --user)
export GROUP_ID=$(id --group)
```

# Setting up HTTPS Certificates

For local development, probably the easiest way to simulate HTTPS is using [mkcert](https://mkcert.dev/), "a simple tool for making locally-trusted development certificates. It requires no configuration."

Don't forget to run `mkcert -i` to register mkcert's Certificate Authority so your machine considers the certificates valid. Then, to actually generate the certificates, simply execute...

```shell
mkcert \
  -cert-file ./nginx/crt.pem \
  -key-file ./certs/key.pem \
  localhost.test *.localhost.test
```

...[docker-compose.yml](./docker-compose.yml) is hard-coded to expect both a `crt.pem` and a `key.pem` in the [./nginx/certs](./nginx/certs) directory. It is mounting those files where the [./nginx/default.conf](./nginx/default.conf) is hard-coded to find them. The Nginx configuration file is also hard-coded to work with the domain `localhost.test`, which is why we generate certificates for both `localhost.test` and any of its subdomain.

# Running the Application

```shell
docker-compose up web app database mail
```

The above command instructs `docker-compose` to start just those services we need to be actively running for the application to function. The order is not important; the services list their dependencies to ensure `docker-compose` boots them in the correct order. Simply executing `docker-compose up` would achieve a similar effect, but also causing the utility containers starting, executing their default commands, and immediately exiting. This is harmless but unnecessary.

The only exception to this is the `webgrind` service which will keep running as it's a separate, persistent application.

Assuming you've pointed `localhost.test` to `127.0.0.1` in your local DNS hosts file, then after starting the app, with the certificates installed, navigating to [localhost.test](https://localhost.test) should cause nginx to proxy php-fpm and invoke [./src/public/index.php](./src/public/index.php) (if it exists).

# Running the Utilities

### PHP Dependencies

The `composer` services is a direct alias of PHP's `composer`, except that it's built on top of the `app` container. `docker-compose` opens a shell in the [./src](./src) directory. `docker-compose` is configured to use [./composer](./composer) as a cache directory for `composer`.

To create a new Laravel project you can simply run...

```shell
docker-compose run composer create-project laravel/laravel .
```

Running any other `composer` command is the same.

```shell
docker-compose run composer require laravel/pint --dev
```

### JS Dependencies

Like composer above, the `npm` service is an alias to Node's `npm`, with the shell defaulting to the [./src](./src) directory and using [./npm](./npm) as a cache directory.


### Artisan

Because this project is implicitly for laravel, there's a service which executes `artisan` in an image based off what the `app` service is using, with the same packages and extensions installed, etc.

### Webgrind

As per the [./docker-compose.yml](./docker-compose.yml) file, after running `docker-compose up webgrind`, [http://localhost:8081](https://localhost:8081) should render a GUI for parsing cachegrind files generated by Xdebug.

The `docker-compose.yml` file hard-codes the `app` and `webgrind` containers to rely on the same [./xdebug](./xdebug) directory so that when `xdebug` running in the `app` container generates cachegrind files, `webgrind` can be aware of them (on refresh).
