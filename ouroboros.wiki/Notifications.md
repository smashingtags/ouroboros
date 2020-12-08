Ouroboros uses [apprise](https://github.com/caronc/apprise) to support a large variety of notification platforms.

### Example email/webhook usage

```shell
docker run -d --name ouroboros \
  -v /var/run/docker.sock:/var/run/docker.sock \
  pyouroboros/ouroboros -N 'mailtos://myUsername:myPassword@gmail.com?to=receivingAddress@gmail.com'\ 
  'jsons://webhook.site/something'
```

### Sample integrations

Some examples:
- Email
- Webhooks
- Discord
- Pushover

### More Integrations

See all notification integrations [supported by apprise](https://github.com/caronc/apprise#supported-notifications)

### Startup notifications

If notifications are enabled, all notification platforms will be fired with a body stating that ouroboros has started with the current time set in the container and the timestamp of when ouroboros will next check for updates.