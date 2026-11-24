# Architeezy

A simple modeling tool for IT architects and others. You can use it on <https://architeezy.com>. The current repository provides a self-hosted version.

To run it locally checkout the repository:

```bash
git clone git@github.com:metamodeldev/architeezy-selfhosted.git
cd architeezy-selfhosted
```

For the AI assistant, start Ollama on the Docker host and run
`ollama pull qwen3.5:0.8b-q8_0`. To use another model provider or disable
the assistant, edit the model settings in `.env`.

Run the application:

```bash
docker compose up
```

Open <http://localhost:8080>. Use `admin` login and password.

To manage users open <http://localhost:8181>. Use `admin` login and password. Select `Manage realms` -> `Architeezy` -> `Users`.

For production use:

- Update passwords and secrets in `.env` file
- Setup Keycloak according to your security policies or replace it by your own Keycloak instance or other OAuth 2.0 IAM system
- Hide Architeezy behind reverse proxy with HTTPS support, for example Nginx

You can use the following Nginx configuration as a template:

```nginx
# Architeezy

server {
    listen 80;
    listen [::]:80;
    server_name architeezy.com;
    return 302 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name architeezy.com;
    ssl_certificate /etc/letsencrypt/live/architeezy.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/architeezy.com/privkey.pem;

    client_max_body_size 50M;
    proxy_read_timeout 900s;

    proxy_http_version 1.1;
    proxy_set_header Host $http_host;
    proxy_set_header Connection "upgrade";
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Port $server_port;

    location / {
        proxy_pass http://localhost:8080;
    }
}

# Keycloak

server {
    listen 80;
    listen [::]:80;
    server_name auth.architeezy.com;
    return 302 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name auth.architeezy.com;
    ssl_certificate /etc/letsencrypt/live/architeezy.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/architeezy.com/privkey.pem;

    proxy_http_version 1.1;
    proxy_set_header Host $http_host;
    proxy_set_header Connection "upgrade";
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Port $server_port;

    location / {
        proxy_pass http://localhost:8181;
    }
}
```
