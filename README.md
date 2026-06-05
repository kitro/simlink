### Issue in resolve_root_symlink in frankenphp

In this repo I tested to reproduce the bug I was encounter when using symlinks in deployment.

The the docker compase contains 3 services:

- frankenphp
- caddy
- php-fpm


To demo the issue I simulate a deployment by creating two versions of the app and then symlinking the current version to the new one.

First let's up the services:

```bash
docker compose up -d
```

Access to the caddy container and symlink the current version to the version one:

```bash
docker compose exec caddy sh
```

```bash
ln -sfn /srv/version_1 /srv/current
```

Access to the frankenphp and symlink the current version to the version one (just to make sure the symlink works in both service)

```bash
docker compose exec frankenphp bash
```

```bash
ln -sfn /srv/version_1 /srv/current
```

To test the response:

For caddy+fpm
```bash
curl localhost
```

For frankenphp

```bash
curl localhost:81
```

### The output for both looks good : 

```
From version 1%
```

Now let's switch the symlink to version 2 :

Access to the caddy container and symlink the current version to the version 2:

```bash
docker compose exec caddy sh
```

```bash
ln -sfn /srv/version_2 /srv/current
```

Access to the frankenphp and symlink the current version to the version 2 :

```bash
docker compose exec frankenphp bash
```

```bash
ln -sfn /srv/version_2 /srv/current
```


### The outputs:

For caddy+fpm
```bash
curl localhost
```
output: `From version 2` (good)

For frankenphp

```bash
curl localhost:81
```

output: `From version 1` (issue)

In frankenphp i tried to reload the config `frankenphp reload -c /etc/frankenphp/Caddyfile -f` with no luck. still pointing to 
old symlink.

Only when I restart the container `docker compose restart frankenphp` than I got the good output `From version 2`.

In frankenphp, it seems that the symlink is resolved only at startup and not on each request, which is why the change in the symlink does not reflect until the container is restarted. This behavior can be problematic in deployment scenarios where you want to switch between different versions of the application without downtime.

## Why using symlinks in deployment?

Using symlinks in deployment allows for zero-downtime deployments. By creating a new version of the application and then updating the symlink to point to the new version, you can ensure that users experience no interruption in service. This approach also allows for easy rollbacks, as you can simply update the symlink to point back to the previous version if something goes wrong with the new deployment.

Open source tools used this deployment strategy : https://capistranorb.com/, https://ansistrano.com/ https://deployer.org/
