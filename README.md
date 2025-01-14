# Scalingo Nginx Buildpack

This buildpack aims at installing a nginx instance and let you configure it at
your convenance.

## Defining the Version

By default we're installing the latest available version of Nginx, but if you
want to use a specific version, you can define the environment variable `NGINX_VERSION`

```console
$ scalingo env-set NGINX_VERSION=1.8.0
```

## Configuration

The buildpack is expecting a configuration file at the root of the project
which can be:

* `nginx.conf`: Simple configuration file
* `nginx.conf.erb`: Template to generate the configuration file
* `servers.conf.erb`: (optional) Let you configure your nginx instance at the `http` level if required

If the template is found, it will be rendered as configuration file, it let you use environment
variables as in the following examples.

## Discouraged Directives

The following directives should not be used in you configuration file: `listen`, `access_log`, `error_log` and `server_name`.

## Configuration Examples (`nginx.conf`)

### Split Traffic to 2 APIs

```
location /api/v1 {
  proxy_pass https://api-v1-app.scalingo.io;
}

location /api/v2 {
  proxy_pass https://api-v2-app.scalingo.io;
}
```

Using a template to give the names of the app from the environment: `nginx.conf.erb`

```
location /api/v1 {
  proxy_pass <%= ENV["API_V1_BACKEND"] %>;
}

location /api/v2 {
  proxy_pass <%= ENV["API_V2_BACKEND"] %>;
}
```

Use nginx configuration:
[https://nginx.org/en/docs/](https://nginx.org/en/docs/) to get details about
how to configure your app.

## Configuration Examples (`servers.conf.erb`)

When using this configuration method, the previous one won't be considered,
they are exclusive.


###  Setup throttling with a `limit_req_zone`

```
# instruction at the http level like
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

server {
    server_name localhost;
    listen <%= ENV['PORT'] %>;

    charset utf-8;
    location {
        limit_req zone=one burst=5;
        proxy_pass http://<%= ENV["API_V1_BACKEND"] %>;
    }
}
```

### Multiple domains configuration

```
server {
    server_name front.example.com;
    listen <%= ENV['PORT'] %>;

    charset utf-8;
    location {
        proxy_pass http://<%= ENV["FRONT_BACKEND"] %>;
    }
}

server {
    server_name api.example.com;
    listen <%= ENV['PORT'] %>;

    charset utf-8;
    location {
        proxy_pass http://<%= ENV["API_BACKEND"] %>;
    }
}
```

## Using Nginx as WAF with ModSecurity

You can easily enable the WAF feature of nginx bundled with ModSecurity by
adding the following environment variable to your application.

```
scalingo env-set ENABLE_MODSECURITY=true
```

At the next deployment, several additional actions will be done:

1. ModSecurity and its dependencies will be installed
2. Default configuration for ModSecurity will be enabled

### Customizing configuration

A few environment variables can be tweaked in order to configure ModSecurity

* `MODSECURITY_DEBUG_LOG_LEVEL` (default `0`): from `0` to `9` (no log to super verbose)
* `MODSECURITY_AUDIT_LOG_LEVEL` (default `Off`): Either `On` (all requests), or `RelevantOnly` (requests returning 4XX and 5XX status code)
