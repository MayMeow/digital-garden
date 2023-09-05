# Docker logs

Docker By default using logs in json format but it doesnt cycle it, and from time to time you will need to delete theme because theu can take a lot of storage. You can use command as follows

```bash
sudo sh -c "truncate -s 0 /var/lib/docker/containers/**/*-json.log"
```