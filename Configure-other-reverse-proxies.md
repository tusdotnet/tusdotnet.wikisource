In general these settings needs to be configured in the reverse proxy. The name of the settings differ between the different proxies.

* Request buffering needs to be disabled
* Max request size needs to be increased to the max chunk size that the client is allowed to use
* Depending on the reverse proxy used it might also be required request timeouts in Kestrel. See [Configure Kestrel](https://github.com/tusdotnet/tusdotnet/wiki/Configure-Kestrel).

# Nginx

Disable request buffering: `proxy_request_buffering off;`

Max requst body size: `client_max_body_size 50M;` and replace "50M" with the number of MB to use.

# Traefik

See https://doc.traefik.io/traefik/middlewares/http/buffering/
