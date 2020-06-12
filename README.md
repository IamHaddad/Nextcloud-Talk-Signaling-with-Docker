# Janus-Nats-Coturn
Installs Janus Nats and Cotrun for the Nextcloud Talk backend
### Running with Docker

#### Docker Compose

You will likely have to adjust the Janus command line options depending on the exact network configuration on your server. Refer to [Setup of Janus](#setup-of-janus) and the Janus documentation for how to configure your Janus server.

Copy `server.conf.in` to `server.conf` and adjust it to your liking. 

If you're using the [docker-compose.yml](docker-compose.yml) configuration as is, the MCU Url must be set to `ws://localhost:8188`, the NATS Url must be set to `nats://localhost:4222`, and TURN Servers must be set to `turn:localhost:3478?transport=udp,turn:localhost:3478?transport=tcp`.

```bash
docker-compose build
docker-compose up -d
```
