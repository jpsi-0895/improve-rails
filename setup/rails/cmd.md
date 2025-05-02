# for port

sudo lsof -i -P -n
sudo lsof -i -P -n | grep puma(server name)

# curl cmd

```sh
curl -v http://127.0.0.1:3000 (verbose curl - you'll see headers and connection info.)
```

# for logs

```sh
tail -f log.production.log (file name)

```
