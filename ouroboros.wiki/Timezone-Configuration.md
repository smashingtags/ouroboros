To more closely monitor ouroboros' actions and for accurate log ingestion, you can change the timezone of the container from UTC by setting the [`TZ`](http://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html) environment variable like so:

```
docker run -d --name ouroboros \
  -e TZ=America/Chicago \
  -v /var/run/docker.sock:/var/run/docker.sock \
  pyouroboros/ouroboros
```