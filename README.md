# Traefik & Cloudflare for Dynamic DNS & SSL

This setup provides a secure and automated way to expose services running on a home server or VPS to the internet. It uses Traefik as a reverse proxy and Cloudflare for DNS and SSL certificate management via Let's Encrypt (using the DNS-01 challenge).

## Features

- **Reverse Proxy:** Traefik routes traffic from the web to your internal services (e.g., Docker containers).
- **Automated SSL:** Automatically obtains and renews free SSL certificates from Let's Encrypt.
- **DNS-01 Challenge:** No need to open port 80 on your firewall. SSL verification happens via DNS records, which is more secure.
- **Cloudflare Integration:** Manages DNS records in Cloudflare automatically.
- **Dynamic DNS (Optional):** If your server has a dynamic IP address, a simple DDNS client can keep your Cloudflare DNS records updated. This setup assumes a DDNS client is running separately if needed.

## Prerequisites

1.  **A domain name** managed by Cloudflare.
2.  **A Cloudflare API Token** with `Zone.DNS` edit permissions for your domain.
3.  **Docker and Docker Compose** installed on your server.

## Setup

1.  **Clone the repository or copy the files** into a directory on your server (e.g., `/opt/traefik`).

2.  **Create a `.env` file** in the same directory and add your Cloudflare credentials:

    ```env
    # .env
    CF_API_TOKEN=your_cloudflare_api_token
    CF_ZONE_API_TOKEN=your_cloudflare_api_token # For some setups, this is the same token
    ```
    *Note: The `CF_API_TOKEN` is the recommended and more secure method.*

3.  **Review `traefik.yml`:**

    - Update the `email` field under `certificatesResolvers.cloudflare.acme.email` with your email address for Let's Encrypt notifications.
    - This file is set up to use the Cloudflare DNS challenge.

4.  **Launch Traefik:**
    ```bash
    docker-compose up -d
    ```

## Exposing a Service

To expose a service (e.g., another Docker container), add labels to its `docker-compose.yml`. Here is an example for a simple `whoami` service:

```yaml
version: '3.8'

services:
  whoami:
    image: "traefik/whoami"
    container_name: "whoami-app"
    networks:
      - traefik_proxy # Must be on the same network as Traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.yourdomain.com`)"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.tls.certresolver=cloudflare"
      - "traefik.http.services.whoami.loadbalancer.server.port=80"

networks:
  traefik_proxy:
    external: true
```
*Make sure the `traefik_proxy` network is the one defined in the main Traefik `docker-compose.yml`.*

After launching this service, Traefik will automatically detect it, create the route, and obtain an SSL certificate for `whoami.yourdomain.com`.
