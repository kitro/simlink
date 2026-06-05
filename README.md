### Issue 503 Service Unavailable after reload frankenphp

The demo simulate the issue in reload frankenphp service throw 503 during handling requests.

First let's up the services:

```bash
docker compose up -d
```

In your terminal to test http requets 

```bash
count=0; while true; do ((count++)); echo -ne "Requests sent: $count\r"; http_status=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:81); if [ "$http_status" != "200" ]; then echo ""; echo "Loop broken at request $count"; echo "Server returned HTTP status $http_status"; break; fi; done
```

Access to the frankenphp and reload the service

```bash
docker compose exec frankenphp bash
```

```bash
frankenphp reload -c /etc/frankenphp/Caddyfile -f
```

Now the command to send request stop and throw `Server returned HTTP status 503`.
