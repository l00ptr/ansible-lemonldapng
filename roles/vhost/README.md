# VHost

A basic role to handle vhost creation and configuration on LemonLDAP.
Currently we only handle vhosts creation on nginx, the idea behind that
role is also to create a generic way to configure vhosts on nginx side
and LemonLDAP manager (that part is not yet implemented).

## Role Variables

### `vhost_vhosts` item attributes

#### `vhostname`

- **Type:** `str`
- **Required:** `yes`

The hostname of the virtual host.

Used as the Nginx `server_name`.

``` yaml
vhostname: example.dalibo.com
```

#### `port`

- **Type:** `int`
- **Required:** `no`
- **Default:** `__nginx_default_https_port` (`443` by default)

The HTTPS port on which Nginx listens.

The same port is used when redirecting HTTP traffic to HTTPS.

``` yaml
port: 443
```

#### `certificate_location`

- **Type:** `str`
- **Required:** `no`
- **Default:** `vhost_global_certificate_location`

Path to the SSL/TLS certificate used by the virtual host.

``` yaml
certificate_location: /etc/ssl/certs/example.pem
```

#### `certificate_key_location`

- **Type:** `str`
- **Required:** `no`
- **Default:** `vhost_global_certificate_key_location`

Path to the SSL/TLS private key used by the virtual host.

``` yaml
certificate_key_location: /etc/ssl/private/example.key
```

#### `client_max_body_size`

- **Type:** `str`
- **Required:** `no`
- **Default:** not configured

Sets the maximum allowed size of the client request body.

The value is passed directly to Nginx's `client_max_body_size`
directive.

``` yaml
client_max_body_size: 20M
```

#### `configure_default_location`

- **Type:** `bool`
- **Required:** `no`
- **Default:** `true`

Whether to configure the default Nginx `location /` for the virtual
host.

When enabled, the location:

- authenticates requests through LemonLDAP::NG;
- optionally proxies requests to `backend`;
- optionally configures WebSocket support;
- passes LemonLDAP::NG headers to the application.

Set this to `false` when the virtual host only requires custom
locations.

``` yaml
configure_default_location: false
```

#### `backend`

- **Type:** `str`
- **Required:** `no`

Backend used by the default `location /`.

The value is passed directly to Nginx's `proxy_pass` directive.

``` yaml
backend: http://{{ ip_prefix }}.4:3000
```

For an HTTPS backend:

``` yaml
backend: https://{{ ip_prefix }}.15:443
```

If `backend` is not defined, no `proxy_pass` directive is generated for
the default location.

#### `use_websocket`

- **Type:** `bool`
- **Required:** `no`
- **Default:** `false`

Enables WebSocket support for the default `location /`.

When enabled, the following Nginx configuration is generated:

``` nginx
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection $connection_upgrade;
proxy_read_timeout 86400s;
proxy_send_timeout 86400s;
```

Example:

``` yaml
use_websocket: true
```

#### `protection_type`

- **Type:** `str`
- **Required:** `no`

Controls authentication handling for the default `location /`.

When set to `authbasic`, the LemonLDAP::NG `401` response is not
redirected using `$lmlocation`.

``` yaml
protection_type: authbasic
```

For normal LemonLDAP::NG authentication, leave this unset.

#### `vhost_protection_type`

- **Type:** `str`
- **Required:** `no`

Controls authentication handling for custom locations.

When set to `authbasic`, the `401` response is not redirected using
`$lmlocation`.

``` yaml
vhost_protection_type: authbasic
```

#### `custom_locations`

- **Type:** `list`
- **Required:** `no`
- **Default:** `[]`

Defines additional Nginx locations for the virtual host.

Each entry must contain the following attributes.

##### `custom_locations[].path`

- **Type:** `str`
- **Required:** `yes`

Path used in the Nginx `location` directive.

``` yaml
custom_locations:
  - path: /api/
```

##### `custom_locations[].server_proto`

- **Type:** `str`
- **Required:** `yes`

Protocol used to connect to the backend.

Typical values are:

- `http`
- `https`

``` yaml
server_proto: http
```

##### `custom_locations[].backend`

- **Type:** `str`
- **Required:** `yes`

Backend hostname and port.

The protocol is provided separately by `server_proto`.

``` yaml
backend: 10.0.0.20:8080
```

This generates:

``` nginx
proxy_pass http://10.0.0.20:8080;
```

## Example

``` yaml
vhost_vhosts:
  - vhostname: grafana.dalibo.info
    configure_default_location: true
    backend: http://{{ ip_prefix }}.4:3000

  - vhostname: example.dalibo.com
    client_max_body_size: 20M
    backend: http://{{ ip_prefix }}.10:8080
    use_websocket: true

  - vhostname: api.dalibo.com
    configure_default_location: false
    custom_locations:
      - path: /api/
        server_proto: http
        backend: '{{ ip_prefix }}.20:8080'
      - path: /websocket/
        server_proto: http
        backend: '{{ ip_prefix }}.21:8080'
```

## License

MIT
